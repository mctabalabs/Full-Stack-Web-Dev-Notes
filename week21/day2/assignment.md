# Week 21 - Day 2 Assignment

## Title
Failure Handling And Overlap Prevention

## Overview
Yesterday you scheduled jobs. Today you make them survive reality: errors inside the job, jobs that run longer than their interval, and jobs that crash the process. You add retries, a lock so two instances cannot run the same job concurrently, and structured logging.

## Learning Objectives Assessed
- Catch and log errors inside a cron callback
- Prevent overlap when a job outlives its interval
- Add bounded retries with backoff
- Make scheduled work idempotent

## Prerequisites
- Day 1 completed (jobs scheduled)

## AI Usage Rules

**Ratio this week:** 30% manual / 70% AI
**Habit:** AI writes schedules, you decide reliability. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Retry helper scaffolds, logging boilerplate.
- **NOT ALLOWED FOR:** The overlap-prevention lock -- design it yourself.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Wrap the callback in try/catch

**What to do:**
An uncaught error inside a cron callback can crash the whole process. Wrap every scheduled job:

```javascript
cron.schedule("0 * * * *", async () => {
  try {
    await runHourlyCleanup();
  } catch (err) {
    console.error("[hourly-cleanup] failed", err);
  }
});
```

Apply the same pattern to all four jobs from Day 1.

**Expected output:**
No job can crash the process.

### Task 2: Overlap guard

**What to do:**
If a job normally takes 2 minutes but occasionally takes 7, the next `*/5` tick will fire while the previous one is still running. Add an in-memory guard:

```javascript
let running = false;
cron.schedule("*/5 * * * *", async () => {
  if (running) {
    console.warn("[every-5] skipped: previous still running");
    return;
  }
  running = true;
  try { await doWork(); }
  finally { running = false; }
});
```

**Expected output:**
Second tick is skipped while the first is mid-flight.

### Task 3: Retry with backoff

**What to do:**
Create `lib/retry.js`:

```javascript
async function retry(fn, { attempts = 3, baseMs = 500 } = {}) {
  let lastErr;
  for (let i = 0; i < attempts; i++) {
    try { return await fn(); }
    catch (err) {
      lastErr = err;
      await new Promise(r => setTimeout(r, baseMs * 2 ** i));
    }
  }
  throw lastErr;
}
module.exports = { retry };
```

Use it inside `daily-report.js` to wrap the DB query. Verify the retry fires by throwing manually once.

**Expected output:**
Retry log shows three attempts before surrender.

### Task 4: Idempotency notes

**What to do:**
In `idempotency-notes.md`, answer:
- What does "idempotent" mean for a cron job?
- If `daily-report` runs twice at 08:00 because of a retry, does it matter? Why?
- Give one example of a NON-idempotent scheduled job and how you would make it idempotent (hint: a processed-ids table or an upsert).

Your own words.

**Expected output:**
`idempotency-notes.md` committed.

### Task 5: Structured logging

**What to do:**
Replace `console.log` in your jobs with a small `logJob(name, event, extra)` helper that prints JSON:

```javascript
function logJob(name, event, extra = {}) {
  console.log(JSON.stringify({ ts: new Date().toISOString(), job: name, event, ...extra }));
}
```

Emit `start`, `success`, and `error` events for each run.

**Expected output:**
Logs parse as JSON.

## Stretch Goals (Optional - Extra Credit)

- Record every run in a `job_runs` table (id, job, started_at, finished_at, status, error).
- Use `p-limit` to bound concurrency across multiple jobs sharing one DB pool.
- Page yourself via email when `daily-report` fails three times in a row.

## Submission Requirements

- **What to submit:** Repo, notes, `AI_AUDIT.md`.
- **Deadline:** End of Day 2

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Try/catch on all jobs | 20 | No crash on throw. |
| Overlap guard | 25 | Second tick skipped while first runs. |
| Retry helper | 20 | Backoff, bounded attempts. |
| Idempotency notes | 20 | Three questions answered. |
| Structured logging | 10 | JSON lines. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Swallowing errors silently.** Catch AND log; never just `catch {}`.
- **Module-level `running` flag in a clustered process.** In-memory flags only work for one process. Next task uses a real lock.
- **Retrying non-idempotent work.** Double-charges, double-emails. Check idempotency first.

## Resources

- Day 2 reading: [Failure Handling and Overlap.md](./Failure%20Handling%20and%20Overlap.md)
- Week 21 AI boundaries: [../ai.md](../ai.md)
