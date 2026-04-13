# Week 21 - Day 4 Assignment

## Title
Choosing A Scheduler -- In-Process, pg_cron, Or A Queue

## Overview
Day 4 is the decision day. node-cron is fine for a single server. pg_cron lives in the database. A queue (BullMQ, next week) survives server restarts cleanly. Today you compare them with a written analysis and migrate one job to pg_cron as a proof of concept.

## Learning Objectives Assessed
- Compare in-process cron, pg_cron, and queues by criteria
- Install and use pg_cron for a real job
- Decide which scheduler fits which job
- Write an ADR (Architecture Decision Record)

## Prerequisites
- Days 1-3 completed

## AI Usage Rules

**Ratio this week:** 30% manual / 70% AI
**Habit:** AI writes schedules, you decide reliability. See [../ai.md](../ai.md).

- **ALLOWED FOR:** SQL scaffolds, comparison tables.
- **NOT ALLOWED FOR:** The decision itself -- that is an architecture call you justify.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: pg_cron setup

**What to do:**
pg_cron is a Postgres extension. In your local Postgres (or a Docker container):

```sql
CREATE EXTENSION IF NOT EXISTS pg_cron;
```

If your installation does not support it, use a Dockerised Postgres that does (e.g., `citusdata/postgres_15:15.5`) and note that in your README.

**Expected output:**
Extension installed. `SELECT * FROM cron.job;` returns empty.

### Task 2: Schedule a job in SQL

**What to do:**
Move the `daily-report` concept to pg_cron:

```sql
SELECT cron.schedule(
  'daily-order-count',
  '0 8 * * *',
  $$ INSERT INTO daily_metrics(name, value, at)
     SELECT 'orders_last_24h', COUNT(*), NOW()
     FROM orders WHERE created_at > NOW() - INTERVAL '1 day' $$
);
```

Create `daily_metrics` first if it does not exist.

**Expected output:**
Job appears in `cron.job`. Row appears in `daily_metrics` after the first run (or after `SELECT cron.schedule(...)` fires manually).

### Task 3: Comparison table

**What to do:**
In `scheduler-comparison.md`, fill this table:

| Criterion | node-cron | pg_cron | BullMQ (next week) |
|-----------|-----------|---------|--------------------|
| Lives where | | | |
| Survives app restart | | | |
| Survives DB restart | | | |
| Easy to inspect "what is scheduled right now" | | | |
| Good for long jobs | | | |
| Good for many small jobs | | | |
| Retries | | | |
| Observability | | | |

Your own words in each cell.

**Expected output:**
Table committed.

### Task 4: ADR

**What to do:**
Write `docs/adr/0001-scheduler-choice.md`:

```markdown
# ADR 0001: Scheduler choice for Mctaba backend

## Status
Accepted

## Context
We need scheduled work for daily reports, cleanup, and reminders.

## Decision
Use node-cron for in-process jobs that are quick and safe to re-run.
Use pg_cron for SQL-only jobs that should run regardless of app state.
Defer queue-based scheduling (BullMQ) to Week 22.

## Consequences
- ...
```

Fill in Consequences with at least four bullets (positive and negative).

**Expected output:**
ADR committed.

### Task 5: Pre-weekend checklist

**What to do:**
Create `CHECKLIST.md` for Week 21:

```markdown
## Week 21 Pre-Weekend Checklist
- [ ] Four jobs scheduled with node-cron
- [ ] All jobs wrapped in try/catch
- [ ] Overlap guard on long jobs
- [ ] Retry helper used in one job
- [ ] Postgres advisory lock used in one job
- [ ] One job migrated to pg_cron
- [ ] Scheduler comparison table
- [ ] ADR 0001 written
- [ ] AI_AUDIT.md current
```

Tick honestly.

**Expected output:**
Checklist committed.

## Stretch Goals (Optional - Extra Credit)

- Add `cron.job_run_details` query and print the last five runs of each pg_cron job.
- Write a tiny dashboard page listing all scheduled jobs from both node-cron and pg_cron.
- Investigate `toad-scheduler` and compare.

## Submission Requirements

- **What to submit:** Repo, comparison, ADR, checklist, `AI_AUDIT.md`.
- **Deadline:** End of Day 4

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| pg_cron installed | 15 | Extension present. |
| Job scheduled via SQL | 20 | Runs and writes to daily_metrics. |
| Comparison table | 25 | All cells filled in your own words. |
| ADR 0001 | 25 | Context, decision, and four consequences. |
| Checklist | 10 | Honest. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Choosing a scheduler because the tutorial used it.** Pick for the shape of your workload.
- **Running pg_cron against a database where the extension is not installed.** It fails silently in some distros.
- **Treating the ADR as paperwork.** The point is the reasoning, not the file.

## Resources

- Day 4 reading: [Choosing a Scheduler.md](./Choosing%20a%20Scheduler.md)
- Week 21 AI boundaries: [../ai.md](../ai.md)
- pg_cron: https://github.com/citusdata/pg_cron
