# Week 28, Day 5: Dashboard Shell and Week 28 Demo

By the end of today, the Next.js dashboard has a sidebar, a tenant header, a working Orders page, a working Products page, and a Wallet widget. Enough to demo "the SME OS core is alive".

**Estimated time:** 4 hours.

---

## The Layout

```
+---------------------------------------------------+
| Logo  | Company: Mctaba Shop        | Wanjiru v |
+-------+-------------------------------------------+
|       |                                           |
|  nav  |              main content                 |
|       |                                           |
+-------+-------------------------------------------+
```

Sidebar with nav (Orders, Products, Customers, Messages, Wallet, Settings). Main area. Top bar with tenant name and user menu.

```jsx
// shop/app/dashboard/layout.js
import Link from "next/link";
import { requireAuth } from "@/app/lib/auth";

export default async function DashboardLayout({ children }) {
  const user = await requireAuth();

  return (
    <div className="min-h-screen flex">
      <aside className="w-56 bg-gray-900 text-white p-4">
        <div className="font-bold mb-6">SME OS</div>
        <nav className="space-y-1">
          <NavLink href="/dashboard">Overview</NavLink>
          <NavLink href="/dashboard/orders">Orders</NavLink>
          <NavLink href="/dashboard/products">Products</NavLink>
          <NavLink href="/dashboard/customers">Customers</NavLink>
          <NavLink href="/dashboard/messages">Messages</NavLink>
          <NavLink href="/dashboard/wallet">Wallet</NavLink>
          <NavLink href="/dashboard/settings">Settings</NavLink>
        </nav>
      </aside>
      <div className="flex-1">
        <header className="border-b p-4 flex justify-between items-center">
          <div className="text-sm text-gray-500">{user.tenantName}</div>
          <div className="text-sm">{user.email}</div>
        </header>
        <main>{children}</main>
      </div>
    </div>
  );
}

function NavLink({ href, children }) {
  return (
    <Link href={href} className="block px-3 py-2 rounded hover:bg-gray-800">
      {children}
    </Link>
  );
}
```

Every `/dashboard/*` page inherits this shell automatically.

---

## The Overview Page

```jsx
// shop/app/dashboard/page.js
import { query } from "@/lib/db";
import { requireAuth } from "@/app/lib/auth";

export default async function OverviewPage() {
  const user = await requireAuth();

  const { rows: stats } = await query(
    `SELECT
       (SELECT COUNT(*)::int FROM orders WHERE tenant_id = $1) AS total_orders,
       (SELECT COUNT(*)::int FROM orders WHERE tenant_id = $1 AND created_at::date = CURRENT_DATE) AS today_orders,
       (SELECT COUNT(*)::int FROM customers WHERE tenant_id = $1) AS total_customers,
       (SELECT balance_cents FROM wallets WHERE tenant_id = $1) AS wallet_balance`,
    [user.tenantId]
  );
  const s = stats[0];

  return (
    <div className="p-8">
      <h1 className="text-3xl font-bold mb-6">Welcome, {user.name}</h1>
      <div className="grid grid-cols-4 gap-4">
        <Stat label="Total orders" value={s.total_orders} />
        <Stat label="Today" value={s.today_orders} />
        <Stat label="Customers" value={s.total_customers} />
        <Stat label="Wallet" value={`KSh ${(s.wallet_balance / 100).toLocaleString()}`} />
      </div>
    </div>
  );
}

function Stat({ label, value }) {
  return (
    <div className="border rounded p-4">
      <div className="text-sm text-gray-500">{label}</div>
      <div className="text-2xl font-bold mt-1">{value}</div>
    </div>
  );
}
```

Four big numbers. The single most important screen in any admin dashboard is the "did the business make money today" widget.

---

## Tenant RLS In Next.js Server Components

Next.js Server Components use their own connection. To make RLS work, wrap queries in a helper:

```javascript
// shop/lib/db.js
import { Pool } from "pg";

const pool = new Pool({ /* ... */ });

export async function queryAsTenant(tenantId, sql, params = []) {
  const client = await pool.connect();
  try {
    await client.query(`SET LOCAL app.current_tenant = $1`, [tenantId]);
    return await client.query(sql, params);
  } finally {
    client.release();
  }
}
```

And use it everywhere in server components:

```jsx
const { rows } = await queryAsTenant(user.tenantId,
  `SELECT id, status, total_cents FROM orders ORDER BY created_at DESC LIMIT 10`
);
```

Now Server Components and the API share the same RLS guarantee. No explicit tenant filter needed.

---

## Orders Page

```jsx
// shop/app/dashboard/orders/page.js
import Link from "next/link";
import { queryAsTenant } from "@/lib/db";
import { requireAuth } from "@/app/lib/auth";

export default async function OrdersPage({ searchParams }) {
  const user = await requireAuth();
  const status = searchParams.status;

  const conditions = [];
  const params = [];
  if (status) {
    params.push(status);
    conditions.push(`status = $${params.length}`);
  }
  const where = conditions.length ? `WHERE ${conditions.join(" AND ")}` : "";

  const { rows: orders } = await queryAsTenant(user.tenantId,
    `SELECT o.id, o.total_cents, o.status, o.channel, o.created_at, c.phone
     FROM orders o LEFT JOIN customers c ON c.id = o.customer_id
     ${where}
     ORDER BY o.created_at DESC LIMIT 100`,
    params
  );

  return (
    <div className="p-8">
      <h1 className="text-2xl font-bold mb-4">Orders</h1>
      <nav className="flex gap-3 mb-6 text-sm">
        {["", "pending", "paid", "delivered", "cancelled"].map((s) => (
          <Link
            key={s || "all"}
            href={s ? `/dashboard/orders?status=${s}` : "/dashboard/orders"}
            className={`px-3 py-1 rounded ${status === s ? "bg-black text-white" : "bg-gray-100"}`}
          >
            {s || "all"}
          </Link>
        ))}
      </nav>
      <table className="w-full text-sm">
        <thead><tr className="text-left border-b">
          <th className="p-2">Order</th><th>Customer</th><th>Channel</th><th>Total</th><th>Status</th><th>When</th>
        </tr></thead>
        <tbody>
          {orders.map((o) => (
            <tr key={o.id} className="border-b">
              <td className="p-2 font-mono text-xs">{o.id.slice(0, 8)}</td>
              <td>{o.phone}</td>
              <td>{o.channel}</td>
              <td>KSh {(o.total_cents / 100).toLocaleString()}</td>
              <td>{o.status}</td>
              <td className="text-xs text-gray-500">{new Date(o.created_at).toLocaleString("en-KE")}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

Simple table. Filter bar. Tenant-scoped via RLS.

---

## Week 28 Demo Script

At the Friday peer session or to yourself, walk through:

1. Show the signup flow. Create a new tenant "Demo Shop 1".
2. Dashboard loads; show the overview with zero stats.
3. Go to Products, add two products.
4. Go to Orders, use `curl` to create an order via the API (paste the command in a terminal visible to the audience).
5. Refresh Orders -- the new order appears.
6. Go to Wallet -- shows zero balance (payments come next week).
7. Sign up a second tenant "Demo Shop 2".
8. Show that Shop 2 sees zero orders and zero products, even though Shop 1 has data.
9. Show the isolation test passing in CI.

Ten minutes. That is Week 28 done.

---

## Checkpoint

1. Signup -> dashboard works end-to-end.
2. Orders, Products, Wallet pages render.
3. Two tenants are fully isolated in the dashboard (verified manually).
4. Isolation tests pass in CI.
5. The demo script runs smoothly.

Commit:

```bash
git add .
git commit -m "feat: dashboard shell with overview orders and wallet pages"
```

---

## What You Learned

- Next.js layouts persist across child pages.
- `queryAsTenant` makes Server Components RLS-aware.
- The demo shape: sign up, create data, show isolation, show the test.

The weekend polishes what exists. Week 29 adds the integrations.
