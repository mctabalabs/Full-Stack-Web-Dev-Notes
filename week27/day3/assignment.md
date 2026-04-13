# Week 27 - Day 3 Assignment

## Title
Schema Design -- The Full Capstone Data Model

## Overview
Day 3 is the full data model for the capstone. Every table gets a tenant_id, the right constraints, indexes, and a seed script that creates a demo tenant so you can test end-to-end.

## Learning Objectives Assessed
- Model a full SaaS domain in one pass
- Add the right indexes for common queries
- Use constraints as invariants
- Seed a realistic demo tenant

## Prerequisites
- Days 1-2 completed

## AI Usage Rules

**Ratio this week:** 55% manual / 45% AI
**Habit:** Architecture dip. See [../ai.md](../ai.md).

- **ALLOWED FOR:** SQL boilerplate after you decided the shape.
- **NOT ALLOWED FOR:** The shape itself.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Finalise the table list

**What to do:**
`capstone/tables.md`, list every table with a one-line purpose. Expect around 10-15 tables for a sensible MVP.

**Expected output:**
Committed.

### Task 2: Migrations

**What to do:**
Write migrations for each table. Every tenant-owned table has:
- `tenant_id UUID NOT NULL REFERENCES tenants(id)`
- A composite unique constraint including tenant_id where appropriate
- RLS enabled with the same tenant_isolation policy

**Expected output:**
Migrations run clean from an empty DB.

### Task 3: Indexes

**What to do:**
For every table, add indexes for common queries. At minimum:
- `(tenant_id, created_at DESC)` for "recent items"
- `(tenant_id, status)` where status exists
- Individual indexes on foreign keys

In `indexes.md`, list every index and the query it supports.

**Expected output:**
Indexes added. Notes committed.

### Task 4: Seed demo tenant

**What to do:**
`scripts/seed-capstone.js` that creates:
- One tenant ("Acme Co", subdomain "acme")
- One owner user + one staff user + three customer users
- Five resources (slots, products, whatever fits)
- Two bookings

Use transactions. Commit all or roll back.

**Expected output:**
Seed runs, all tables have data.

### Task 5: Query exercise

**What to do:**
`capstone/queries.md`, write five real queries and explain them. Use `EXPLAIN ANALYZE` on each. Examples:
- "All bookings for tenant X in the last 7 days"
- "Top 5 resources by booking count for tenant X"
- "Outstanding payments older than 14 days per tenant"

Confirm each uses your indexes.

**Expected output:**
Committed.

## Stretch Goals (Optional - Extra Credit)

- Add soft-delete columns where appropriate.
- Add an `events` append-only log table.
- Add `updated_at` triggers.

## Submission Requirements

- **What to submit:** Repo, tables, migrations, indexes, seed, queries, `AI_AUDIT.md`.
- **Deadline:** End of Day 3

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Table list | 10 | 10-15 tables, purposes. |
| Migrations | 30 | All tables with tenant_id, RLS. |
| Indexes | 20 | Justified per query. |
| Seed | 20 | Real demo tenant. |
| Query exercise | 15 | Five queries with explain. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Sequential scans on big tables.** Index tenant_id + filter columns.
- **Seed script that fails halfway.** Wrap in a transaction.
- **Forgetting RLS on one table.** That one table is the attack surface.

## Resources

- Day 3 reading: [Schema Design.md](./Schema%20Design.md)
- Week 27 AI boundaries: [../ai.md](../ai.md)
