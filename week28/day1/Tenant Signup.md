# Week 28, Day 1: Tenant Signup

By the end of today, a new user can sign up through the Next.js dashboard, the system creates a tenant for them, wires them up as the owner, and lands them on an empty dashboard they can start configuring.

**Estimated time:** 4-5 hours.

---

## Flow

1. User visits `/signup`.
2. Enters company name, email, password.
3. Server:
   a. Creates a `users` row (hashed password).
   b. Creates a `tenants` row with a slug derived from the company name.
   c. Creates a `tenant_members` row linking user as `owner`.
   d. Creates a `wallets` row with zero balance.
   e. Seeds the tenant with a default product (optional, helps with empty states).
   f. Issues a JWT scoping the session to the new tenant.
4. Browser sets the auth cookie.
5. Redirect to `/dashboard`.

All of this happens in one transaction.

---

## The Server Action

```javascript
// shop/app/actions/signup.js
"use server";

import bcrypt from "bcrypt";
import jwt from "jsonwebtoken";
import { cookies } from "next/headers";
import { redirect } from "next/navigation";
import { pool } from "@/lib/db";

export async function signupAction(prevState, formData) {
  const companyName = formData.get("companyName")?.toString().trim();
  const email = formData.get("email")?.toString().toLowerCase();
  const password = formData.get("password")?.toString();

  if (!companyName || companyName.length < 2) return { error: "Company name required" };
  if (!email || !email.includes("@")) return { error: "Valid email required" };
  if (!password || password.length < 8) return { error: "Password must be 8+ chars" };

  const slug = slugify(companyName);

  const client = await pool.connect();
  try {
    await client.query("BEGIN");

    // 1. Check for duplicate email
    const { rows: existing } = await client.query("SELECT id FROM users WHERE email = $1", [email]);
    if (existing.length > 0) {
      await client.query("ROLLBACK");
      return { error: "An account with that email already exists" };
    }

    // 2. Create user
    const passwordHash = await bcrypt.hash(password, 12);
    const { rows: userRows } = await client.query(
      `INSERT INTO users (email, password_hash, name)
       VALUES ($1, $2, $3) RETURNING id`,
      [email, passwordHash, companyName]
    );
    const userId = userRows[0].id;

    // 3. Create tenant with a unique slug
    const uniqueSlug = await findUniqueSlug(client, slug);
    const { rows: tenantRows } = await client.query(
      `INSERT INTO tenants (name, slug, status)
       VALUES ($1, $2, 'active') RETURNING id`,
      [companyName, uniqueSlug]
    );
    const tenantId = tenantRows[0].id;

    // 4. Link user as owner
    await client.query(
      `INSERT INTO tenant_members (tenant_id, user_id, role)
       VALUES ($1, $2, 'owner')`,
      [tenantId, userId]
    );

    // 5. Create wallet
    await client.query(
      `INSERT INTO wallets (tenant_id, balance_cents) VALUES ($1, 0)`,
      [tenantId]
    );

    // 6. Seed a default product
    await client.query(
      `INSERT INTO products (tenant_id, sku, name, price_cents, stock)
       VALUES ($1, 'DEMO-1', 'Demo product', 100000, 10)`,
      [tenantId]
    );

    await client.query("COMMIT");

    // 7. Issue JWT
    const token = jwt.sign(
      { sub: userId, email, tenantId, role: "owner" },
      process.env.JWT_SECRET,
      { expiresIn: "8h" }
    );
    cookies().set("auth_token", token, {
      httpOnly: true,
      secure: process.env.NODE_ENV === "production",
      sameSite: "lax",
      path: "/",
      maxAge: 60 * 60 * 8,
    });

    return redirect("/dashboard");
  } catch (err) {
    await client.query("ROLLBACK");
    console.error("Signup error:", err);
    return { error: "Signup failed. Please try again." };
  } finally {
    client.release();
  }
}

function slugify(name) {
  return name.toLowerCase().replace(/[^a-z0-9]+/g, "-").replace(/^-|-$/g, "").slice(0, 32);
}

async function findUniqueSlug(client, base) {
  let slug = base || "tenant";
  let attempt = 0;
  while (true) {
    const candidate = attempt === 0 ? slug : `${slug}-${attempt}`;
    const { rows } = await client.query("SELECT id FROM tenants WHERE slug = $1", [candidate]);
    if (rows.length === 0) return candidate;
    attempt++;
    if (attempt > 100) throw new Error("Could not find unique slug");
  }
}
```

Long function. Each step is deliberate and the whole thing is transactional -- a failure at any step rolls everything back.

---

## The Signup Page

```jsx
// shop/app/signup/page.js
import { useActionState } from "react";
import { signupAction } from "@/app/actions/signup";

export default function SignupPage() {
  return (
    <div className="max-w-sm mx-auto p-16">
      <h1 className="text-3xl font-bold mb-6">Create your SME OS</h1>
      <SignupForm />
    </div>
  );
}
```

```jsx
// shop/app/signup/SignupForm.js
"use client";
import { useActionState } from "react";
import { signupAction } from "@/app/actions/signup";

export default function SignupForm() {
  const [state, formAction] = useActionState(signupAction, { error: null });

  return (
    <form action={formAction} className="space-y-4">
      {state?.error && <p className="text-red-600">{state.error}</p>}
      <input name="companyName" placeholder="Company name" required className="w-full border p-2" />
      <input name="email" type="email" placeholder="Email" required className="w-full border p-2" />
      <input name="password" type="password" placeholder="Password" required className="w-full border p-2" />
      <button type="submit" className="w-full bg-black text-white py-2">Create my SME OS</button>
    </form>
  );
}
```

Simple. The Server Action does all the work.

---

## What Happens Next

After signup, the user lands on `/dashboard`. Today that page is a placeholder:

```jsx
// shop/app/dashboard/page.js
export default async function DashboardPage() {
  const user = await requireAuth();
  return (
    <div className="p-8">
      <h1 className="text-2xl font-bold">Welcome, {user.email}</h1>
      <p>Your SME OS is ready. Tomorrow you will see your customers, orders, and products here.</p>
    </div>
  );
}
```

The placeholder is fine. We are building day-by-day.

---

## Testing

Manual test:
1. Sign up with `Test Company 1` / `test1@example.com` / `password123`.
2. Check the database: one new tenant, one user, one member, one wallet, one product.
3. Visit `/dashboard`. See the welcome message.
4. Log out (delete cookie). Visit `/dashboard` -- redirect to `/signup`.
5. Sign up with the same email -- error.
6. Sign up with a different email and `Test Company 1` -- slug should be `test-company-1-1` (suffix added).

Every test should pass. If not, debug before moving on.

---

## Checkpoint

1. `/signup` renders the form.
2. Submitting creates all five rows (user, tenant, member, wallet, product) in one transaction.
3. Cookie is set and `/dashboard` renders the welcome message.
4. Duplicate email is rejected.
5. Slug collisions are handled with a suffix.
6. A failure mid-transaction rolls everything back.

Commit:

```bash
git add .
git commit -m "feat: tenant signup with transactional setup"
```

---

## What You Learned

Today is the first real Capstone build. The shape is: one form, one Server Action, one transaction, many tables. This shape will repeat for every feature.

Tomorrow: tenant membership, auth middleware, and the first protected API endpoint.
