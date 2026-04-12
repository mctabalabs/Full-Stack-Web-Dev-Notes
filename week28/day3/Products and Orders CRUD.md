# Week 28, Day 3: Products and Orders CRUD

By the end of today, every core resource (products, customers, orders) has full CRUD handlers behind the middleware chain, with basic input validation and consistent response shapes.

**Estimated time:** 4 hours.

---

## The Pattern

Each resource is a file that exports a router. Each router has the same shape:

```
GET    /            -> list
GET    /:id         -> detail
POST   /            -> create
PATCH  /:id         -> partial update
DELETE /:id         -> soft delete
```

Every handler uses `req.db.query(...)` with no explicit tenant filter (RLS handles it). Every response is `{ data, error }`.

---

## Products Router

```javascript
// api/routes/products.js
const express = require("express");
const { z } = require("zod");
const router = express.Router();

const ProductCreate = z.object({
  sku: z.string().min(1),
  name: z.string().min(1),
  priceCents: z.number().int().nonnegative(),
  stock: z.number().int().nonnegative(),
});

router.get("/", async (req, res) => {
  const { rows } = await req.db.query(
    `SELECT id, sku, name, price_cents, stock, created_at FROM products ORDER BY created_at DESC`
  );
  res.json({ data: rows, error: null });
});

router.get("/:id", async (req, res) => {
  const { rows } = await req.db.query(
    `SELECT * FROM products WHERE id = $1`,
    [req.params.id]
  );
  if (!rows[0]) return res.status(404).json({ data: null, error: { message: "Not found" } });
  res.json({ data: rows[0], error: null });
});

router.post("/", async (req, res) => {
  try {
    const input = ProductCreate.parse(req.body);
    const { rows } = await req.db.query(
      `INSERT INTO products (tenant_id, sku, name, price_cents, stock)
       VALUES ($1, $2, $3, $4, $5)
       RETURNING *`,
      [req.tenantId, input.sku, input.name, input.priceCents, input.stock]
    );
    res.status(201).json({ data: rows[0], error: null });
  } catch (err) {
    if (err.name === "ZodError") {
      return res.status(400).json({ data: null, error: { message: err.issues[0].message } });
    }
    if (err.code === "23505") {
      return res.status(409).json({ data: null, error: { message: "Duplicate SKU" } });
    }
    throw err;
  }
});

router.patch("/:id", async (req, res) => {
  const { name, priceCents, stock } = req.body;
  const { rows } = await req.db.query(
    `UPDATE products
     SET name = COALESCE($1, name),
         price_cents = COALESCE($2, price_cents),
         stock = COALESCE($3, stock)
     WHERE id = $4
     RETURNING *`,
    [name, priceCents, stock, req.params.id]
  );
  if (!rows[0]) return res.status(404).json({ data: null, error: { message: "Not found" } });
  res.json({ data: rows[0], error: null });
});

router.delete("/:id", async (req, res) => {
  const result = await req.db.query("DELETE FROM products WHERE id = $1", [req.params.id]);
  if (result.rowCount === 0) return res.status(404).json({ data: null, error: { message: "Not found" } });
  res.status(204).end();
});

module.exports = router;
```

Zod handles input validation. Standard error shapes. No explicit tenant filter anywhere.

Install zod: `npm install zod`.

---

## Orders Router

Orders are more complicated because they have items and status transitions.

```javascript
// api/routes/orders.js
const express = require("express");
const { z } = require("zod");
const router = express.Router();

const OrderCreate = z.object({
  customerPhone: z.string(),
  items: z.array(z.object({
    productId: z.string().uuid(),
    quantity: z.number().int().positive(),
  })).min(1),
  channel: z.enum(["ussd", "whatsapp", "web"]),
});

const STATUS_TRANSITIONS = {
  pending: ["paid", "cancelled"],
  paid: ["delivered", "cancelled"],
  delivered: [],
  cancelled: [],
};

router.get("/", async (req, res) => {
  const { status, channel } = req.query;
  const conditions = [];
  const params = [];
  if (status) { params.push(status); conditions.push(`status = $${params.length}`); }
  if (channel) { params.push(channel); conditions.push(`channel = $${params.length}`); }
  const where = conditions.length ? `WHERE ${conditions.join(" AND ")}` : "";
  const { rows } = await req.db.query(
    `SELECT o.*, c.phone AS customer_phone
     FROM orders o LEFT JOIN customers c ON c.id = o.customer_id
     ${where}
     ORDER BY o.created_at DESC LIMIT 50`,
    params
  );
  res.json({ data: rows, error: null });
});

router.post("/", async (req, res) => {
  const input = OrderCreate.parse(req.body);
  const client = req.db;
  try {
    await client.query("BEGIN");

    // Ensure customer exists
    const { rows: custRows } = await client.query(
      `INSERT INTO customers (tenant_id, phone) VALUES ($1, $2)
       ON CONFLICT (tenant_id, phone) DO UPDATE SET phone = EXCLUDED.phone
       RETURNING id`,
      [req.tenantId, input.customerPhone]
    );
    const customerId = custRows[0].id;

    // Fetch products
    const productIds = input.items.map((i) => i.productId);
    const { rows: products } = await client.query(
      `SELECT id, price_cents, stock FROM products WHERE id = ANY($1) FOR UPDATE`,
      [productIds]
    );

    if (products.length !== productIds.length) {
      throw new Error("Product not found");
    }

    const productMap = Object.fromEntries(products.map((p) => [p.id, p]));
    let totalCents = 0;
    for (const item of input.items) {
      const p = productMap[item.productId];
      if (p.stock < item.quantity) throw new Error(`Insufficient stock for ${item.productId}`);
      totalCents += p.price_cents * item.quantity;
    }

    // Create order
    const { rows: orderRows } = await client.query(
      `INSERT INTO orders (tenant_id, customer_id, total_cents, channel, status)
       VALUES ($1, $2, $3, $4, 'pending')
       RETURNING *`,
      [req.tenantId, customerId, totalCents, input.channel]
    );
    const order = orderRows[0];

    // Insert items, deduct stock
    for (const item of input.items) {
      const p = productMap[item.productId];
      await client.query(
        `INSERT INTO order_items (tenant_id, order_id, product_id, quantity, price_cents_at_purchase)
         VALUES ($1, $2, $3, $4, $5)`,
        [req.tenantId, order.id, item.productId, item.quantity, p.price_cents]
      );
      await client.query(
        `UPDATE products SET stock = stock - $1 WHERE id = $2`,
        [item.quantity, item.productId]
      );
    }

    await client.query("COMMIT");
    res.status(201).json({ data: order, error: null });
  } catch (err) {
    await client.query("ROLLBACK");
    throw err;
  }
});

router.patch("/:id/status", async (req, res) => {
  const { status: nextStatus } = req.body;

  const { rows: current } = await req.db.query("SELECT status FROM orders WHERE id = $1", [req.params.id]);
  if (!current[0]) return res.status(404).json({ error: { message: "Not found" } });

  if (!STATUS_TRANSITIONS[current[0].status]?.includes(nextStatus)) {
    return res.status(409).json({ error: { message: `Cannot move from ${current[0].status} to ${nextStatus}` } });
  }

  const { rows } = await req.db.query(
    `UPDATE orders SET status = $1 WHERE id = $2 RETURNING *`,
    [nextStatus, req.params.id]
  );
  res.json({ data: rows[0], error: null });
});

module.exports = router;
```

A proper orders router. Transaction around the create. Status transition enforcement. Stock deduction. All of it reuses patterns from Week 15.

Note `client.query` is inside a transaction started on the same `req.db` client from the middleware. The RLS context is already set, so every query respects the tenant boundary.

---

## Wiring

```javascript
// api/index.js
const express = require("express");
const app = express();
app.use(express.json());

app.use("/api", require("./routes/index"));

app.listen(5000, () => console.log("API on :5000"));
```

The single-line `app.use("/api", ...)` mounts everything. All routes inherit the middleware chain from Day 2.

---

## Frontend: Products List Page

```jsx
// shop/app/dashboard/products/page.js
import Link from "next/link";
import { requireAuth } from "@/app/lib/auth";
import { query } from "@/lib/db";

export default async function ProductsPage() {
  const user = await requireAuth();
  const { rows: products } = await query(
    `SELECT id, sku, name, price_cents, stock FROM products WHERE tenant_id = $1 ORDER BY created_at DESC`,
    [user.tenantId]
  );
  // NOTE: Next.js server components cannot use RLS the same way since they
  // use a different connection. Explicit tenant_id filter here. Future
  // improvement: use a helper that wraps every query with SET LOCAL.

  return (
    <div className="p-8">
      <h1 className="text-2xl font-bold mb-6">Products</h1>
      <Link href="/dashboard/products/new" className="bg-black text-white px-4 py-2 rounded mb-4 inline-block">Add product</Link>
      <table className="w-full">
        <thead><tr><th>SKU</th><th>Name</th><th>Price</th><th>Stock</th></tr></thead>
        <tbody>
          {products.map((p) => (
            <tr key={p.id} className="border-t">
              <td>{p.sku}</td>
              <td>{p.name}</td>
              <td>KSh {(p.price_cents / 100).toLocaleString()}</td>
              <td>{p.stock}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

For now, Next.js Server Components use explicit filters. RLS via `SET LOCAL` in Next.js is possible but requires a small helper to wrap every query; we will add that on Day 5 when we do the dashboard polish.

---

## Checkpoint

1. `POST /api/products` creates a product. Duplicate SKU returns 409.
2. `GET /api/products` returns only the current tenant's products.
3. `POST /api/orders` creates an order with items and deducts stock.
4. Invalid status transitions are rejected with 409.
5. Two tenants with identical SKUs coexist.
6. Frontend products page renders a table of the current tenant's products.

Commit:

```bash
git add .
git commit -m "feat: products and orders crud with transactions and status machine"
```

---

## What You Learned

- The middleware chain makes CRUD handlers almost trivial.
- Zod validates input consistently.
- RLS lets you forget about tenant filters on the backend.
- Next.js Server Components need their own tenant handling (coming Day 5).

Tomorrow: wallet, and the first end-to-end isolation test.
