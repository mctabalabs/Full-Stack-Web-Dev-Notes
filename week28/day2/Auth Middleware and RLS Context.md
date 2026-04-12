# Week 28, Day 2: Auth Middleware and RLS Context

By the end of today, every authenticated request loads the user, selects their current tenant, sets the Postgres RLS context, and runs the handler. Forgetting any of those is impossible because the middleware is the only way in.

**Estimated time:** 3 hours.

---

## The Middleware Chain

```
Request -> authenticateJWT -> resolveTenant -> setRLSContext -> handler
```

Every protected route goes through all four. Public routes (signup, login, webhooks) bypass.

---

## `authenticateJWT`

```javascript
// api/middleware/auth.js
const jwt = require("jsonwebtoken");

module.exports = async function authenticateJWT(req, res, next) {
  const token = req.headers.authorization?.split(" ")[1] || req.cookies?.auth_token;
  if (!token) return res.status(401).json({ error: { message: "Missing token" } });

  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET);
    req.user = { id: payload.sub, email: payload.email };
    req.tenantId = payload.tenantId;
    req.role = payload.role;
    next();
  } catch {
    return res.status(401).json({ error: { message: "Invalid token" } });
  }
};
```

Reads either `Authorization: Bearer ...` or the `auth_token` cookie. The JWT already contains the tenant id because signup put it there.

---

## `resolveTenant`

```javascript
// api/middleware/resolveTenant.js
module.exports = async function resolveTenant(req, res, next) {
  if (!req.tenantId) return res.status(400).json({ error: { message: "No tenant selected" } });

  const { rows } = await query(
    `SELECT t.id, t.name, t.slug, t.status, tm.role
     FROM tenants t
     JOIN tenant_members tm ON tm.tenant_id = t.id
     WHERE t.id = $1 AND tm.user_id = $2 AND t.status = 'active'`,
    [req.tenantId, req.user.id]
  );

  if (rows.length === 0) return res.status(403).json({ error: { message: "Access denied" } });
  req.tenant = rows[0];
  next();
};
```

Verifies that:
1. The tenant exists.
2. The user is a member of it.
3. The tenant is active (not suspended or deleted).

Without this middleware, a token with `tenantId` for a tenant the user was removed from would still work.

---

## `setRLSContext`

```javascript
// api/middleware/setRLSContext.js
const { pool } = require("../config/db");

module.exports = async function setRLSContext(req, res, next) {
  req.db = await pool.connect();
  try {
    await req.db.query(`SET LOCAL app.current_tenant = $1`, [req.tenantId]);
    res.on("finish", () => req.db.release());
    res.on("close", () => req.db.release());
    next();
  } catch (err) {
    req.db.release();
    next(err);
  }
};
```

This middleware is critical. It:
1. Borrows a client from the pool and attaches it to `req.db`.
2. Sets the Postgres session variable on that client.
3. Releases the client when the response is done.

Every handler in the protected chain uses `req.db.query(...)` -- not the pool directly. This ensures every query runs with the RLS context set to the current tenant.

**SET LOCAL vs SET** -- `SET LOCAL` only affects the current transaction. Since each request uses a different client from the pool, the setting is per-request and does not leak across requests. If you use `SET` instead of `SET LOCAL`, the setting persists on the client after release and another request could inherit the wrong tenant. Always `SET LOCAL`.

---

## Wiring The Chain

```javascript
// api/routes/index.js
const express = require("express");
const authenticateJWT = require("../middleware/auth");
const resolveTenant = require("../middleware/resolveTenant");
const setRLSContext = require("../middleware/setRLSContext");

const router = express.Router();

router.use(authenticateJWT);
router.use(resolveTenant);
router.use(setRLSContext);

router.use("/customers", require("./customers"));
router.use("/products", require("./products"));
router.use("/orders", require("./orders"));

module.exports = router;
```

Every route nested under `/api` goes through all three middlewares. Hard to forget.

---

## The First Handler

```javascript
// api/routes/customers.js
const express = require("express");
const router = express.Router();

router.get("/", async (req, res) => {
  const { rows } = await req.db.query(
    `SELECT id, phone, name, first_seen_at FROM customers ORDER BY first_seen_at DESC LIMIT 50`
  );
  res.json({ data: rows, error: null });
});

module.exports = router;
```

Notice no `WHERE tenant_id = ...`. RLS adds it automatically. The handler is oblivious to multi-tenancy.

**This is the reward for the RLS setup.** Every handler from now on is free from tenant filtering. The system is safer and cleaner.

---

## Verifying Isolation End-To-End

Test with two tenants:

```javascript
// Curl with tenant A's token:
curl -H "Authorization: Bearer <tokenA>" http://localhost:5000/api/customers
// Returns only A's customers.

// Curl with tenant B's token:
curl -H "Authorization: Bearer <tokenB>" http://localhost:5000/api/customers
// Returns only B's customers.

// Attempt to forge: manually craft a token with wrong tenant id:
const token = jwt.sign({ sub: userIdA, tenantId: tenantIdB }, process.env.JWT_SECRET);
curl -H "Authorization: Bearer $token" http://localhost:5000/api/customers
// Returns 403 -- the user is not a member of tenant B.
```

The forged-token case is the one to test. If your setup passes all three, tenant isolation is working.

---

## Checkpoint

1. `authenticateJWT` rejects requests without a token.
2. `resolveTenant` rejects requests where the user is not a member of the claimed tenant.
3. `setRLSContext` sets the Postgres session variable.
4. `GET /api/customers` returns only the current tenant's customers with no explicit filter.
5. Forged tokens with wrong tenant claims return 403.
6. Releasing the client on response end; no leaked connections.

Commit:

```bash
git add .
git commit -m "feat: auth middleware with rls context and tenant resolution"
```

---

## What You Learned

- Middleware chains enforce constraints at the edge.
- `SET LOCAL` keeps tenant context scoped to the request.
- RLS + middleware = handlers that never need to think about tenants.
- The forged-token test is the one that proves isolation works.

Tomorrow: products and orders CRUD using this foundation.
