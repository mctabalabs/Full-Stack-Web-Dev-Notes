# Week 27 - Day 2 Assignment

## Title
Multi-Tenant Patterns -- Choosing Isolation

## Overview
Day 2 is the most important architectural decision of the capstone: how do you isolate tenants? Shared DB shared schema, shared DB separate schemas, or separate DBs. You pick one with a written justification and start implementing it.

## Learning Objectives Assessed
- Compare the three multi-tenant isolation models
- Pick one for your capstone with reasoning
- Understand the tradeoffs (cost, isolation, ops)
- Know when to migrate from one to another

## Prerequisites
- Day 1 completed

## AI Usage Rules

**Ratio this week:** 55% manual / 45% AI
**Habit:** Architecture dip. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Summarising approaches from articles.
- **NOT ALLOWED FOR:** Choosing for you.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Three approaches

**What to do:**
In `capstone/isolation-models.md`, fill:

| Model | Pros | Cons | Ops cost | Isolation |
|-------|------|------|----------|-----------|
| Shared DB, tenant_id column | | | | |
| Shared DB, schema per tenant | | | | |
| DB per tenant | | | | |

Your own words.

**Expected output:**
Table committed.

### Task 2: Decision

**What to do:**
`capstone/adr/0003-isolation.md`:

```
## Status
Accepted

## Context
Capstone is a multi-tenant SaaS with <N> expected tenants.
Budget, ops appetite, compliance level.

## Decision
Shared DB with tenant_id column + Postgres RLS.

## Consequences
...
```

Fill in at least five consequences, positive and negative.

**Expected output:**
ADR committed.

### Task 3: tenant_id column pattern

**What to do:**
Every tenant-owned table gets `tenant_id UUID NOT NULL REFERENCES tenants(id)`. Draft migrations for five tables (tenants, users, resources, bookings, payments):

```sql
CREATE TABLE tenants (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  subdomain TEXT UNIQUE NOT NULL
);
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  email TEXT NOT NULL,
  password_hash TEXT NOT NULL,
  role TEXT NOT NULL DEFAULT 'staff',
  UNIQUE (tenant_id, email)
);
-- ... etc
```

Note `(tenant_id, email)` is unique, NOT just email -- different tenants can share emails.

**Expected output:**
Migrations committed.

### Task 4: Row level security (preview)

**What to do:**
Enable RLS on `bookings`:

```sql
ALTER TABLE bookings ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON bookings
  USING (tenant_id = current_setting('app.tenant_id')::uuid);
```

And in your code, set the session tenant on each request:

```javascript
await client.query("SELECT set_config('app.tenant_id', $1, true)", [req.tenantId]);
```

This is a preview -- you will finish it in Week 28.

**Expected output:**
Policy in place. Query without set_config fails.

### Task 5: Scaling notes

**What to do:**
`capstone/scaling-notes.md`, 5-7 sentences:
- At what tenant count do you reconsider the model?
- What signals would push you toward schema-per-tenant?
- What is "noisy neighbour" and how does it affect your choice?

Your own words.

**Expected output:**
Committed.

## Stretch Goals (Optional - Extra Credit)

- Add a `soft_delete` pattern per tenant.
- Plan schema-per-tenant migration path.
- Add row count limits per tenant as a quota.

## Submission Requirements

- **What to submit:** Repo, models notes, ADR, migrations, scaling notes, `AI_AUDIT.md`.
- **Deadline:** End of Day 2

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Three-model table | 20 | All cells. |
| ADR 0003 | 25 | Five consequences. |
| Migrations with tenant_id | 25 | Unique constraints per tenant. |
| RLS preview | 20 | Policy active. |
| Scaling notes | 5 | Three questions answered. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Unique email globally.** Block tenants from having users with the same email.
- **Forgetting RLS on any tenant-owned table.** A single leak is the whole story.
- **Trusting application-level tenant checks only.** Defense in depth means RLS.

## Resources

- Day 2 reading: [Multi-tenant Patterns.md](./Multi-tenant%20Patterns.md)
- Week 27 AI boundaries: [../ai.md](../ai.md)
