# Week 28, Day 4: Wallets and Isolation Testing

By the end of today, each tenant has a working wallet, every payment flows through wallet transactions, and you have automated tests that prove two tenants cannot see each other's data under any attack.

**Estimated time:** 3-4 hours.

---

## The Wallet Model

Each tenant has one row in `wallets` with `balance_cents`. Every change goes through `wallet_transactions`. The rule:

**Never update `wallets.balance_cents` directly. Always go through `createWalletTransaction` which is atomic.**

```javascript
// api/services/wallet.service.js
async function createWalletTransaction(client, { tenantId, amountCents, kind, ref, note }) {
  // Lock the wallet row
  const { rows: wallet } = await client.query(
    `SELECT balance_cents FROM wallets WHERE tenant_id = $1 FOR UPDATE`,
    [tenantId]
  );

  if (!wallet[0]) throw new Error("Wallet not found");
  const current = BigInt(wallet[0].balance_cents);
  const next = current + BigInt(amountCents);

  if (next < 0n) throw new Error("Insufficient balance");

  // Write the transaction
  const { rows: txnRows } = await client.query(
    `INSERT INTO wallet_transactions (tenant_id, amount_cents, kind, ref, note, balance_after_cents)
     VALUES ($1, $2, $3, $4, $5, $6)
     RETURNING *`,
    [tenantId, amountCents, kind, ref, note, next.toString()]
  );

  // Update the balance
  await client.query(
    `UPDATE wallets SET balance_cents = $1, updated_at = NOW() WHERE tenant_id = $2`,
    [next.toString(), tenantId]
  );

  return txnRows[0];
}

module.exports = { createWalletTransaction };
```

Three moves: lock, insert, update. All on the same client so they are in one transaction. `FOR UPDATE` prevents race conditions when two payments land at once. BigInt for safe integer math on large balances.

---

## Wallet Routes

```javascript
// api/routes/wallet.js
router.get("/", async (req, res) => {
  const { rows } = await req.db.query(`SELECT * FROM wallets WHERE tenant_id = $1`, [req.tenantId]);
  res.json({ data: rows[0], error: null });
});

router.get("/transactions", async (req, res) => {
  const { rows } = await req.db.query(
    `SELECT * FROM wallet_transactions ORDER BY created_at DESC LIMIT 100`
  );
  res.json({ data: rows, error: null });
});
```

Read endpoints only. Writes happen via the service.

---

## Wiring Orders -> Wallet

When a payment completes, credit the tenant's wallet:

```javascript
// In the payment callback handler:
const client = await pool.connect();
try {
  await client.query("BEGIN");
  await client.query(`SET LOCAL app.current_tenant = $1`, [order.tenant_id]);

  await client.query(
    `UPDATE orders SET status = 'paid' WHERE id = $1 AND status = 'pending'`,
    [order.id]
  );

  await walletService.createWalletTransaction(client, {
    tenantId: order.tenant_id,
    amountCents: order.total_cents,
    kind: "credit",
    ref: mpesaReceipt,
    note: `Payment for order ${order.id}`,
  });

  await client.query("COMMIT");
} catch (err) {
  await client.query("ROLLBACK");
  throw err;
} finally {
  client.release();
}
```

Order status transition and wallet credit in the same transaction. Either both happen or neither.

---

## Isolation Tests

Write a real test file that exercises the whole thing:

```javascript
// api/test/isolation.test.js
const { test } = require("node:test");
const assert = require("node:assert/strict");
const request = require("supertest");
const app = require("../index");

test("two tenants cannot see each other's data", async () => {
  // Create two tenants via signup
  const tokenA = await signup("Tenant A", "a@test.com", "password123");
  const tokenB = await signup("Tenant B", "b@test.com", "password123");

  // A creates a product
  const productA = await request(app)
    .post("/api/products")
    .set("Authorization", `Bearer ${tokenA}`)
    .send({ sku: "X1", name: "A's product", priceCents: 1000, stock: 5 });
  assert.equal(productA.status, 201);

  // B tries to read A's product
  const bList = await request(app)
    .get("/api/products")
    .set("Authorization", `Bearer ${tokenB}`);
  assert.equal(bList.body.data.length, 0); // B sees zero products

  // B tries to GET A's product directly
  const bGet = await request(app)
    .get(`/api/products/${productA.body.data.id}`)
    .set("Authorization", `Bearer ${tokenB}`);
  assert.equal(bGet.status, 404); // not 403 -- do not leak existence

  // B tries to delete A's product
  const bDelete = await request(app)
    .delete(`/api/products/${productA.body.data.id}`)
    .set("Authorization", `Bearer ${tokenB}`);
  assert.equal(bDelete.status, 404);

  // A forges a token with B's tenant id
  const forgedToken = jwt.sign(
    { sub: /* A user id */, tenantId: /* B tenant id */ },
    process.env.JWT_SECRET
  );
  const forgedGet = await request(app)
    .get("/api/products")
    .set("Authorization", `Bearer ${forgedToken}`);
  assert.equal(forgedGet.status, 403);
});
```

Five scenarios. Every one a real attack. If all pass, the multi-tenant foundation is solid.

This test should run in CI on every push. If it ever fails, the tenant boundary is broken and deploys should halt.

---

## Checkpoint

1. `GET /api/wallet` returns the current tenant's balance.
2. A payment crediting the wallet writes one `wallet_transactions` row and updates `balance_cents`.
3. Two concurrent payments race-test: both succeed, final balance is correct.
4. `GET /api/wallet/transactions` shows the history.
5. The isolation test passes.
6. Breaking the RLS policy deliberately (drop one) makes the test fail -- verifying the test is useful.

Commit:

```bash
git add .
git commit -m "feat: wallet service with transactional updates and isolation tests"
```

---

## What You Learned

- Wallets never get touched directly -- always through a service.
- BigInt for money at scale.
- `FOR UPDATE` prevents wallet race conditions.
- The isolation test is the anchor that keeps the system honest.

Tomorrow: the dashboard shell and Week 28 demo.
