# Week 28 Weekend: Core Polish

A polish weekend. The core is working. Make it solid before integrations land next week.

**Estimated time:** 4-6 hours.

---

## Tasks

- [ ] Every API endpoint has at least one happy-path test.
- [ ] Every page has a loading state.
- [ ] Empty states on every list page.
- [ ] Input validation errors surface clearly in forms.
- [ ] A deliberate Postgres outage (stop the container) yields friendly errors, not 500s.
- [ ] Signup seeds each tenant with 3 sample products and 2 sample customers for better demo first-impressions.
- [ ] `scripts/reset-db.sh` drops the database, recreates, loads schema, creates 3 demo tenants.
- [ ] `scripts/demo-data.sql` is committed and idempotent.
- [ ] The CI workflow runs `npm test` in the api and succeeds.
- [ ] Deploy to staging.

## Deliverables

- Staging URL running with demo data.
- CI green.
- Running the full isolation test locally and in CI both pass.

## Grading Rubric (50 pts)

| Area | Points |
|---|---|
| All endpoints tested | 15 |
| Loading and empty states on every page | 10 |
| Seeded demo data | 5 |
| CI passing | 10 |
| Deployed to staging | 10 |

## Hints

**Make empty states friendly.** A new tenant's first view should explain what to do next, not show an empty table. "No orders yet. Your first order will appear here."

**Do not add features.** Polishing over feature creep is the hard skill.

**Test against real Postgres in CI.** Not SQLite, not in-memory. The RLS tests only work with a real Postgres.

Next week: USSD, WhatsApp, and M-Pesa all get added to the core.
