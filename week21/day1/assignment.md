# Week 21 - Day 1 Assignment

## Title
Cron Fundamentals -- node-cron And Your First Scheduled Task

## Overview
Week 21 is all about reliability of scheduled work. Today you install node-cron (which you briefly touched in Week 20), schedule a few jobs of different frequencies, and understand the cron expression syntax.

## Learning Objectives Assessed
- Read and write cron expressions
- Schedule recurring jobs with node-cron
- Handle timezones correctly
- Avoid the "ran on startup" trap

## Prerequisites
- Week 20 Day 4 (first node-cron exposure)

## AI Usage Rules

**Ratio this week:** 30% manual / 70% AI
**Habit:** AI writes schedules, you decide reliability. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Cron expression generation, scaffolding.
- **NOT ALLOWED FOR:** Reliability decisions (retries, overlap, timezones).
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Install and schedule

**What to do:**
In your backend, install node-cron (you likely have it). Create `jobs/daily-report.js`:

```javascript
const cron = require("node-cron");
const pool = require("../db/pool");

cron.schedule("0 8 * * *", async () => {
  console.log("[daily-report] running at", new Date().toISOString());
  const { rows } = await pool.query(
    "SELECT COUNT(*) as n FROM orders WHERE created_at > NOW() - INTERVAL '1 day'"
  );
  console.log(`[daily-report] Orders in last 24h: ${rows[0].n}`);
}, { timezone: "Africa/Nairobi" });

console.log("Daily report scheduled for 08:00 EAT");
```

Require this in your main `index.js`.

**Expected output:**
Server starts with the job scheduled.

### Task 2: Test with a short interval

**What to do:**
Temporarily change the expression to `"*/1 * * * *"` (every minute). Watch the logs. Confirm it fires 2-3 times.

Restore the real expression (`0 8 * * *`).

**Expected output:**
Log output every minute during the test.

### Task 3: Three different schedules

**What to do:**
Add three more scheduled jobs in separate files:
- `jobs/hourly-cleanup.js` -- every hour on the hour (`0 * * * *`)
- `jobs/weekly-report.js` -- Mondays at 09:00 (`0 9 * * 1`)
- `jobs/every-5-minutes.js` -- every 5 minutes (`*/5 * * * *`)

Each one just logs for now.

**Expected output:**
Four scheduled jobs in the registry.

### Task 4: Timezone matters

**What to do:**
In `timezone-notes.md`, explain:
- What timezone does node-cron use by default? (UTC.)
- Why should you pass `timezone: "Africa/Nairobi"`?
- What happens to a cron at `"0 8 * * *"` with no timezone when the server is in UTC?
- Daylight saving -- does Kenya have it? (No.)

Your own words.

**Expected output:**
`timezone-notes.md` committed.

### Task 5: "Ran on startup" trap

**What to do:**
node-cron does NOT run jobs on server startup; only on schedule. But some schedulers do. In `startup-notes.md`, write 3-4 sentences about:
- Why running "the last scheduled iteration" on startup is a dangerous assumption.
- What happens if the server crashes and restarts at 08:01? Does the 08:00 job run at 08:01?
- How would you build "catch-up" logic if you needed it?

**Expected output:**
`startup-notes.md` committed.

## Stretch Goals (Optional - Extra Credit)

- Use `toad-scheduler` as an alternative to node-cron.
- Run the same scheduler in Docker with a timezone env var.
- Add a `cron.validate(expression)` call to sanity-check user-supplied schedules.

## Submission Requirements

- **What to submit:** Repo with 4 scheduled jobs, notes, `AI_AUDIT.md`.
- **Deadline:** End of Day 1

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| First job scheduled | 20 | Logs confirm it runs. |
| Test with short interval | 10 | Fires multiple times. |
| Three more schedules | 20 | All valid expressions. |
| Timezone notes | 25 | Four questions answered. |
| Startup trap notes | 15 | Three questions answered. |
| Clean commits | 10 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Using UTC expressions when you mean local time.** Always pass the timezone.
- **Scheduling a job inside an HTTP handler.** The job is created every time the handler runs. Schedule once, at startup.
- **Expressions with too few fields.** node-cron uses 5 or 6 fields (seconds optional). POSIX cron uses 5. Double-check.

## Resources

- Day 1 reading: [Cron Jobs and Scheduled Tasks.md](./Cron%20Jobs%20and%20Scheduled%20Tasks.md)
- Week 21 AI boundaries: [../ai.md](../ai.md)
- crontab.guru (expression tester): https://crontab.guru/
