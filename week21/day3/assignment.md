# Week 21 - Day 3 Assignment

## Title
Long Running Jobs And Distributed Locks

## Overview
When you run two instances of your server (or scale to three), the in-memory overlap guard from Day 2 is useless -- each instance has its own flag. Today you take a real distributed lock in Postgres (or Redis) so only one instance runs the job at a time.

## Learning Objectives Assessed
- Take a Postgres advisory lock
- Release the lock on success AND failure
- Split a long job into chunks you can resume
- Decide between Postgres advisory locks and Redis SETNX

## Prerequisites
- Days 1-2 completed

## AI Usage Rules

**Ratio this week:** 30% manual / 70% AI
**Habit:** AI writes schedules, you decide reliability. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Lock helper scaffolding.
- **NOT ALLOWED FOR:** Deciding where the lock lives (that is architectural).
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Postgres advisory lock helper

**What to do:**
Create `lib/pg-lock.js`:

```javascript
const pool = require("../db/pool");

async function withLock(key, fn) {
  const client = await pool.connect();
  try {
    const { rows } = await client.query("SELECT pg_try_advisory_lock($1) AS ok", [key]);
    if (!rows[0].ok) {
      console.log(`[lock] ${key} held by another instance, skipping`);
      return;
    }
    try { return await fn(); }
    finally { await client.query("SELECT pg_advisory_unlock($1)", [key]); }
  } finally { client.release(); }
}

module.exports = { withLock };
```

`pg_try_advisory_lock` returns immediately -- if the lock is held, the caller is told to skip.

**Expected output:**
Helper committed.

### Task 2: Use the lock

**What to do:**
In `daily-report.js`:

```javascript
const { withLock } = require("../lib/pg-lock");

cron.schedule("0 8 * * *", async () => {
  await withLock(42001, async () => {
    // ...the real work
  });
}, { timezone: "Africa/Nairobi" });
```

Start two copies of the server. At 08:00 only one will run.

**Expected output:**
Only one of the two instances logs the "running" line.

### Task 3: Long job in chunks

**What to do:**
Imagine a nightly job that processes 50,000 rows. Doing it all in one SQL statement would lock the table. Instead chunk it:

```javascript
async function processAll() {
  let lastId = 0;
  while (true) {
    const { rows } = await pool.query(
      "SELECT id FROM pending_items WHERE id > $1 ORDER BY id LIMIT 500",
      [lastId]
    );
    if (rows.length === 0) break;
    for (const row of rows) await processOne(row.id);
    lastId = rows[rows.length - 1].id;
  }
}
```

Write `jobs/process-pending.js` that runs nightly under the lock and uses this pattern on any one of your existing tables (orders, leads, messages -- whichever fits).

**Expected output:**
Chunked processing works.

### Task 4: Postgres vs Redis locks

**What to do:**
In `lock-choice.md`, write 5-7 sentences:
- What does `pg_try_advisory_lock` get you for free?
- When would you prefer Redis `SET key value NX PX 60000` instead?
- What is the risk of a lock whose owner crashes and never releases?
- How does "TTL on the lock" help?

Your own words.

**Expected output:**
`lock-choice.md` committed.

### Task 5: Kill test

**What to do:**
Start the job. While it runs, `kill -9` the process. Restart. Does the advisory lock still block other instances? (No -- advisory locks are released when the Postgres session ends, so the crash releases it.)

Record the experiment in `kill-test-notes.md` with two sentences on what you saw.

**Expected output:**
`kill-test-notes.md` committed.

## Stretch Goals (Optional - Extra Credit)

- Use `redlock` for a Redis-based distributed lock.
- Record lock contention in a `lock_audit` table.
- Replace the while loop with a generator for cleaner chunking.

## Submission Requirements

- **What to submit:** Repo, notes, `AI_AUDIT.md`.
- **Deadline:** End of Day 3

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Postgres advisory lock helper | 25 | Uses try_advisory_lock, releases on finally. |
| Lock used in a real job | 20 | Two instances, only one runs. |
| Chunked long job | 25 | Pagination by id, no full-table scan. |
| Lock choice notes | 20 | Four questions answered. |
| Kill test notes | 5 | Two sentences. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Forgetting to release the lock.** Always release in a `finally`.
- **Advisory lock with a shared connection.** The lock belongs to the session. Use a dedicated client from the pool.
- **Lock TTL too short.** If a Redis lock TTL is 10 seconds but the job runs 30, you get overlap anyway.

## Resources

- Day 3 reading: [Long Running Jobs and Distributed Locks.md](./Long%20Running%20Jobs%20and%20Distributed%20Locks.md)
- Week 21 AI boundaries: [../ai.md](../ai.md)
- Postgres advisory locks: https://www.postgresql.org/docs/current/explicit-locking.html#ADVISORY-LOCKS
