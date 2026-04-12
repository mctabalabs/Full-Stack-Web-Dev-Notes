# Week 15, Day 4: Order Management and Admin

By the end of today, the shop has a "My Orders" page for customers (identified by phone), an admin-only orders dashboard with status transitions, and one more Server Action that moves orders through the lifecycle (`pending -> paid -> shipped -> delivered`). The customer-facing dashboard reuses the auth pattern from Week 12.

**Prior-week concepts you will use today:**
- Server Actions (Week 15, Day 3)
- JWT auth middleware (Week 12, Day 3)
- Server Components with database queries (Week 14, Day 2)

**Estimated time:** 3 hours

---

## The Admin Area

Admins live in the same Postgres `users` table from Week 12 with `role = 'admin'`. We are going to reuse the password flow there -- login via a Next.js page that calls a Server Action, stores a JWT in an HttpOnly cookie, and reads it on protected routes.

### Login as a Server Action

Create `app/admin/login/page.js`:

```jsx
// app/admin/login/page.js
import LoginForm from "./LoginForm";
export const metadata = { title: "Admin Login" };
export default function LoginPage() {
  return (
    <div className="max-w-sm mx-auto p-16">
      <h1 className="text-2xl font-bold mb-6">Admin Login</h1>
      <LoginForm />
    </div>
  );
}
```

Create `app/admin/login/LoginForm.js`:

```jsx
"use client";
import { useActionState } from "react";
import { loginAction } from "@/app/actions/auth";

export default function LoginForm() {
  const [state, formAction] = useActionState(loginAction, { error: null });

  return (
    <form action={formAction} className="space-y-4">
      {state?.error && <p className="text-red-600 text-sm">{state.error}</p>}
      <input name="email" type="email" required placeholder="email" className="w-full border p-2" />
      <input name="password" type="password" required placeholder="password" className="w-full border p-2" />
      <button type="submit" className="w-full bg-brand text-white py-2">Sign in</button>
    </form>
  );
}
```

Notice `useActionState`. It is a React 19 hook designed for Server Actions: you pass the action and an initial state, and it returns the latest state (including any error returned by the action) plus a `formAction` to bind to `<form action={...}>`. It handles the pending state automatically. No `useState` for "submitting".

Create `app/actions/auth.js`:

```javascript
// app/actions/auth.js
"use server";

import { cookies } from "next/headers";
import { redirect } from "next/navigation";
import bcrypt from "bcrypt";
import jwt from "jsonwebtoken";
import { query } from "@/lib/db";

export async function loginAction(prevState, formData) {
  const email = formData.get("email")?.toString().toLowerCase();
  const password = formData.get("password")?.toString();

  if (!email || !password) {
    return { error: "Email and password required" };
  }

  const { rows } = await query(
    "SELECT id, email, role, password_hash FROM users WHERE email = $1",
    [email]
  );
  const user = rows[0];
  if (!user) {
    return { error: "Invalid credentials" };
  }

  const ok = await bcrypt.compare(password, user.password_hash);
  if (!ok) {
    return { error: "Invalid credentials" };
  }

  if (user.role !== "admin") {
    return { error: "Not an admin" };
  }

  const token = jwt.sign(
    { sub: user.id, email: user.email, role: user.role },
    process.env.JWT_SECRET,
    { expiresIn: "8h" }
  );

  cookies().set("admin_token", token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === "production",
    sameSite: "lax",
    path: "/",
    maxAge: 60 * 60 * 8,
  });

  redirect("/admin/orders");
}

export async function logoutAction() {
  cookies().delete("admin_token");
  redirect("/admin/login");
}

export async function requireAdmin() {
  const token = cookies().get("admin_token")?.value;
  if (!token) redirect("/admin/login");
  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET);
    if (payload.role !== "admin") redirect("/admin/login");
    return payload;
  } catch {
    redirect("/admin/login");
  }
}
```

Three things worth noting.

**`cookies()` from `next/headers`** is the Next.js API for reading and writing cookies inside Server Components and Server Actions. It is a Next.js function, not a browser API, and only works on the server.

**`httpOnly: true`** -- the cookie is inaccessible to client-side JavaScript. Even if your shop had an XSS vulnerability, an attacker could not steal the admin token. This is the upgrade from Week 12's `localStorage` approach: cookies are safer for session tokens when you have the option.

**`redirect("/admin/orders")`** inside a Server Action performs a server-side redirect. The client is told to navigate, which is cleaner than throwing and handling a navigation intent on the client.

---

## The Admin Orders Page

Create `app/admin/orders/page.js`:

```jsx
// app/admin/orders/page.js
import Link from "next/link";
import { query } from "@/lib/db";
import { requireAdmin } from "@/app/actions/auth";

export const metadata = { title: "Orders | Admin" };

export default async function AdminOrdersPage({ searchParams }) {
  await requireAdmin();

  const status = searchParams.status || "";
  const params = [];
  let where = "";
  if (status) {
    params.push(status);
    where = `WHERE status = $${params.length}`;
  }

  const { rows: orders } = await query(
    `SELECT id, customer_name, customer_phone, subtotal_cents, payment_method, status, created_at
     FROM orders ${where}
     ORDER BY created_at DESC
     LIMIT 100`,
    params
  );

  return (
    <div className="max-w-5xl mx-auto p-8">
      <h1 className="text-2xl font-bold mb-4">Orders</h1>
      <nav className="flex gap-3 mb-6 text-sm">
        {["", "pending", "paid", "shipped", "delivered", "cancelled"].map((s) => (
          <Link
            key={s || "all"}
            href={s ? `/admin/orders?status=${s}` : "/admin/orders"}
            className={`px-3 py-1 rounded ${status === s ? "bg-brand text-white" : "bg-gray-100"}`}
          >
            {s || "all"}
          </Link>
        ))}
      </nav>
      <table className="w-full border-collapse">
        <thead>
          <tr className="text-left text-sm text-gray-600">
            <th className="p-2 border-b">Order</th>
            <th className="p-2 border-b">Customer</th>
            <th className="p-2 border-b">Total</th>
            <th className="p-2 border-b">Method</th>
            <th className="p-2 border-b">Status</th>
            <th className="p-2 border-b">When</th>
          </tr>
        </thead>
        <tbody>
          {orders.map((o) => (
            <tr key={o.id} className="text-sm">
              <td className="p-2 border-b">
                <Link href={`/admin/orders/${o.id}`} className="font-mono text-xs hover:underline">
                  {o.id.slice(0, 8)}
                </Link>
              </td>
              <td className="p-2 border-b">
                {o.customer_name}<br />
                <span className="text-xs text-gray-500">{o.customer_phone}</span>
              </td>
              <td className="p-2 border-b">KSh {(o.subtotal_cents / 100).toLocaleString()}</td>
              <td className="p-2 border-b">{o.payment_method}</td>
              <td className="p-2 border-b">
                <span className={`px-2 py-0.5 text-xs rounded ${statusColor(o.status)}`}>
                  {o.status}
                </span>
              </td>
              <td className="p-2 border-b text-xs text-gray-500">
                {new Date(o.created_at).toLocaleString("en-KE")}
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}

function statusColor(status) {
  return {
    pending: "bg-yellow-100 text-yellow-800",
    paid: "bg-blue-100 text-blue-800",
    shipped: "bg-purple-100 text-purple-800",
    delivered: "bg-green-100 text-green-800",
    cancelled: "bg-red-100 text-red-800",
  }[status] || "bg-gray-100";
}
```

The `await requireAdmin()` at the top of the page function is a guard: if the admin is not logged in, it redirects before any data loads. The Server Component pattern lets you do auth-then-query in a straight line.

---

## The Admin Order Detail Page

Create `app/admin/orders/[id]/page.js`:

```jsx
import { query } from "@/lib/db";
import { requireAdmin } from "@/app/actions/auth";
import StatusUpdater from "./StatusUpdater";
import { notFound } from "next/navigation";

export default async function AdminOrderDetail({ params }) {
  await requireAdmin();

  const { rows } = await query("SELECT * FROM orders WHERE id = $1", [params.id]);
  const order = rows[0];
  if (!order) notFound();

  const { rows: items } = await query(
    `SELECT oi.quantity, oi.price_cents_at_purchase, p.name, p.slug
     FROM order_items oi JOIN products p ON p.id = oi.product_id
     WHERE oi.order_id = $1`,
    [params.id]
  );

  return (
    <div className="max-w-3xl mx-auto p-8">
      <h1 className="text-2xl font-bold mb-1">Order {order.id.slice(0, 8)}</h1>
      <p className="text-sm text-gray-500 mb-6">{new Date(order.created_at).toLocaleString("en-KE")}</p>

      <div className="grid md:grid-cols-2 gap-6">
        <div>
          <h2 className="font-medium mb-2">Customer</h2>
          <p>{order.customer_name}</p>
          <p>{order.customer_phone}</p>
          <p className="text-sm text-gray-600">{order.delivery_address}</p>
        </div>
        <div>
          <h2 className="font-medium mb-2">Payment</h2>
          <p>Method: {order.payment_method}</p>
          <p>Total: KSh {(order.subtotal_cents / 100).toLocaleString()}</p>
          <p>Status: {order.status}</p>
        </div>
      </div>

      <h2 className="font-medium mt-8 mb-2">Items</h2>
      <ul className="border rounded divide-y">
        {items.map((item, i) => (
          <li key={i} className="p-3 flex justify-between">
            <span>{item.quantity} x {item.name}</span>
            <span>KSh {(item.price_cents_at_purchase * item.quantity / 100).toLocaleString()}</span>
          </li>
        ))}
      </ul>

      <div className="mt-6">
        <StatusUpdater orderId={order.id} currentStatus={order.status} />
      </div>
    </div>
  );
}
```

### The StatusUpdater client component

```jsx
// app/admin/orders/[id]/StatusUpdater.js
"use client";
import { useState, useTransition } from "react";
import { updateOrderStatus } from "@/app/actions/orders";

const TRANSITIONS = {
  pending: ["paid", "cancelled"],
  paid: ["shipped", "cancelled"],
  shipped: ["delivered"],
  delivered: [],
  cancelled: [],
};

export default function StatusUpdater({ orderId, currentStatus }) {
  const [status, setStatus] = useState(currentStatus);
  const [isPending, startTransition] = useTransition();
  const [error, setError] = useState("");

  const next = TRANSITIONS[status];

  function handleChange(nextStatus) {
    setError("");
    startTransition(async () => {
      const result = await updateOrderStatus(orderId, nextStatus);
      if (result.error) {
        setError(result.error);
      } else {
        setStatus(nextStatus);
      }
    });
  }

  if (next.length === 0) return <p className="text-sm text-gray-500">No further transitions.</p>;

  return (
    <div className="flex flex-wrap gap-2">
      {next.map((s) => (
        <button
          key={s}
          onClick={() => handleChange(s)}
          disabled={isPending}
          className="px-4 py-2 bg-brand text-white rounded disabled:opacity-50"
        >
          Move to {s}
        </button>
      ))}
      {error && <p className="text-red-600 text-sm w-full">{error}</p>}
    </div>
  );
}
```

And the corresponding action in `app/actions/orders.js`:

```javascript
// app/actions/orders.js
"use server";

import { query } from "@/lib/db";
import { revalidatePath } from "next/cache";
import { requireAdmin } from "./auth";

const TRANSITIONS = {
  pending: ["paid", "cancelled"],
  paid: ["shipped", "cancelled"],
  shipped: ["delivered"],
  delivered: [],
  cancelled: [],
};

export async function updateOrderStatus(orderId, nextStatus) {
  await requireAdmin();

  const { rows } = await query("SELECT status FROM orders WHERE id = $1", [orderId]);
  const current = rows[0]?.status;
  if (!current) return { error: "Order not found" };

  if (!TRANSITIONS[current]?.includes(nextStatus)) {
    return { error: `Cannot transition from ${current} to ${nextStatus}` };
  }

  await query(
    "UPDATE orders SET status = $1, updated_at = NOW() WHERE id = $2",
    [nextStatus, orderId]
  );

  revalidatePath(`/admin/orders/${orderId}`);
  revalidatePath("/admin/orders");
  return { ok: true };
}
```

The Server Action is the choke point for status changes. The state-machine lives in one place -- not in the button's `onClick`, not in the UI components. Validate on the server, always.

### Cancellation restocks

A cancelled order should return its items to stock. Add a side effect to the action when moving to `cancelled`:

```javascript
if (nextStatus === "cancelled" && current !== "cancelled") {
  const client = await pool.connect();
  try {
    await client.query("BEGIN");
    const { rows: items } = await client.query(
      "SELECT product_id, quantity FROM order_items WHERE order_id = $1",
      [orderId]
    );
    for (const item of items) {
      await client.query("UPDATE products SET stock = stock + $1 WHERE id = $2",
        [item.quantity, item.product_id]);
    }
    await client.query("UPDATE orders SET status = 'cancelled', updated_at = NOW() WHERE id = $1", [orderId]);
    await client.query("COMMIT");
  } catch (err) {
    await client.query("ROLLBACK");
    throw err;
  } finally {
    client.release();
  }
  revalidatePath("/products");
  return { ok: true };
}
```

Stock accounting is one of the places e-commerce bugs hurt the most. A cancellation that does not restock becomes free inventory loss. Always test it.

---

## Customer "My Orders" Page

Customers should be able to look up their orders by phone number. No password -- their phone is the identifier, same pattern as Week 13's USSD trust model. For now this is a simple lookup page; a full "customer account" with OTP is out of scope.

Create `app/my-orders/page.js`:

```jsx
import { query } from "@/lib/db";
import Link from "next/link";

export const metadata = { title: "My Orders" };

export default async function MyOrdersPage({ searchParams }) {
  const phone = searchParams.phone;
  let orders = [];
  if (phone) {
    const { rows } = await query(
      `SELECT id, subtotal_cents, status, created_at FROM orders
       WHERE customer_phone = $1 ORDER BY created_at DESC LIMIT 20`,
      [phone]
    );
    orders = rows;
  }

  return (
    <div className="max-w-xl mx-auto p-8">
      <h1 className="text-2xl font-bold mb-6">My Orders</h1>
      <form action="/my-orders" method="GET" className="flex gap-2 mb-6">
        <input
          type="tel"
          name="phone"
          defaultValue={phone}
          placeholder="+254712..."
          className="flex-1 border p-2 rounded"
        />
        <button type="submit" className="bg-brand text-white px-4 rounded">Find</button>
      </form>

      {phone && orders.length === 0 && (
        <p className="text-gray-500">No orders found for {phone}.</p>
      )}

      <ul className="space-y-3">
        {orders.map((o) => (
          <li key={o.id} className="border p-4 rounded">
            <div className="font-mono text-xs text-gray-500">{o.id.slice(0, 8)}</div>
            <div className="flex justify-between mt-1">
              <span>KSh {(o.subtotal_cents / 100).toLocaleString()}</span>
              <span className="text-sm">{o.status}</span>
            </div>
            <div className="text-xs text-gray-500">
              {new Date(o.created_at).toLocaleString("en-KE")}
            </div>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

This is again a Server Component with no client-side state. The form submits `?phone=...`, the page re-renders, the server queries. Nothing more.

There is an obvious privacy concern -- anyone who knows someone's phone number can look up their orders. For a real shop you would add OTP verification (text them a code, verify it before showing orders). Defer that to Week 18 when we formalise auth patterns. For this week, the lookup is honest-but-weak.

---

## Checkpoint

1. `/admin/login` works. Wrong password shows error inline.
2. After login, the admin cookie is set (HttpOnly -- check devtools Application tab).
3. `/admin/orders` shows all orders. Filter by status works.
4. Clicking an order opens `/admin/orders/:id` with items, customer info, status updater.
5. Moving an order from pending -> paid -> shipped -> delivered works.
6. Moving an order directly from pending -> delivered is refused with a clear error.
7. Cancelling an order restocks the items. Verify in `psql`.
8. `/my-orders?phone=+254...` shows that customer's orders.
9. Logging out via the logout action clears the cookie and sends you to `/admin/login`.

Commit:

```bash
git add .
git commit -m "feat: admin orders dashboard with status transitions and customer my-orders"
```

---

## What You Learned

- `useActionState` is the idiomatic React 19 way to handle Server Action forms.
- `cookies()` from `next/headers` reads and writes HttpOnly cookies on the server.
- `requireAdmin()` called at the top of a Server Component page redirects early.
- State-machine transitions should live in the Server Action, not the UI.
- Cancellation restocks must happen in a transaction -- half-done inventory is worse than nothing.

Tomorrow is Friday: recap, Phase 3 midpoint reflection, and the peer coding session. The weekend project puts it all together -- a working shop with catalogue, cart, checkout, orders, and admin, no real payments. Week 16 adds M-Pesa.
