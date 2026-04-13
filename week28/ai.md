# Week 28 - AI Boundaries

**Ratio this week: 40% Manual / 60% AI**
**Habit introduced: "AI cannot keep tenants safe."**
**Shift from last week: You move from pure design to building the core. But multi-tenant security is the hardest thing in the Capstone, so the manual ratio stays elevated.**

This week you build the core multi-tenant platform: signup, auth middleware, RLS contexts, tenants with products and orders, wallets, and isolation tests. The all-caps rule this week: **every line that enforces tenant isolation is hand-written, hand-reviewed, and hand-tested.** A tenant seeing another tenant's data is a Capstone-ending bug. Prevent it with discipline.

---

## Why The Ratio Is Elevated

Multi-tenant security is auth + money + architecture all at once. Miss any one of the three and tenants leak into each other. The ratio reflects the stakes.

---

## What You MUST Do Manually (40%)

### Day 1 -- Tenant signup
- Write the signup flow by hand: email verification, tenant creation, RLS context initialisation.
- Test with two tenants. Prove one cannot log in as the other.

### Day 2 -- Auth middleware and RLS context
- The middleware that sets the Postgres `app.tenant_id` session variable on every request -- manual, line by line.
- The test that proves the middleware fails closed if the tenant is missing -- manual.
- Read the Postgres RLS docs again this week. Not an AI summary. The real pages.

### Day 3 -- Products and orders CRUD (multi-tenant)
- Repeat of earlier CRUD patterns, but with RLS. Every query must be verified in isolation.
- Write the integration test: two tenants, each with their own products. Tenant A cannot see tenant B's products. Hardcoded into CI.

### Day 4 -- Wallets and isolation testing
- Wallets per tenant with double-entry bookkeeping (credits and debits must always balance).
- Full isolation test suite: every endpoint tested for cross-tenant access.
- Write one deliberate test that *tries* to break isolation (set the wrong tenant_id manually). It should fail with "row not found", not "permission denied".

### Day 5 -- Dashboard shell + demo
- Admin dashboard that shows a tenant's own data only. Reuse Week 14-16 patterns.

---

## What You CAN Use AI For (60%)

- **CRUD scaffolding** for resources after RLS is in place.
- **Dashboard UI.**
- **Test fixtures.**

Forbidden:
- Writing RLS policies.
- Writing the tenant-context middleware.
- Anything touching the tenant isolation boundary.

### Good vs bad prompts this week

**Bad:** "Build a multi-tenant orders API."
**Good:** "Here is my orders route [paste]. I have RLS enabled on the `orders` table with a policy using `current_setting('app.tenant_id')`. My middleware sets this on every request. What are three ways a malicious tenant could still see another tenant's orders?"

---

## Things AI Is Bad At This Week

- **Forgetting tenant_id in a query.** AI sometimes writes joins that bypass RLS. Every query needs manual review.
- **Session variable scope.** AI sometimes sets the session variable at connection time instead of per-transaction, which leaks across requests. Always set it per-transaction.
- **Test coverage.** AI tests the happy path; you must test the attacker path.

---

## Core Mental Models For This Week

- **RLS runs at the database.** Every query sees a filtered view, even if your application code forgot to filter.
- **Session variables must be set per-transaction.** Never per-connection.
- **Isolation is only as strong as its weakest query.** One missing filter and everything breaks.

---

## This Week's AI Audit Focus

Add a required section: **RLS coverage matrix.** For every table, list its RLS policy, its tests, and the last time the test was run.

---

## Assessment

- Live test: facilitator logs in as Tenant A, creates a product, logs in as Tenant B, tries to fetch Tenant A's product by ID. The request must fail closed.
- Walk through the RLS middleware. Explain what happens if a request has no tenant header.
- Facilitator asks: "if I manually change `app.tenant_id` in a transaction, what prevents me from reading another tenant's row?"

---

## One Sentence To Remember

"Tenants see each other's data only when engineers get lazy. Do not be lazy. Not even once."
