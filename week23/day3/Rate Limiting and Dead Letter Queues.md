# Week 23, Day 3: Rate Limiting and Dead Letter Queues

By the end of today, your queue respects the rate limits of every downstream service (WhatsApp, Telegram, M-Pesa, SMS providers) without manual sleeps, dead jobs go to a "did not work" holding tank, and workers shut down cleanly on SIGTERM.

**Prior concepts:** retries and priorities (Week 23 Day 2).

**Estimated time:** 3 hours

---

## Rate Limiting A Worker

BullMQ has built-in rate limiting per worker:

```javascript
const worker = new Worker("notifications", handler, {
  connection,
  limiter: {
    max: 20,        // 20 jobs
    duration: 1000, // per 1 second
  },
});
```

The worker processes at most 20 jobs per second. Jobs above the rate are queued until the next second. No manual `setTimeout`, no hand-tuned pacing.

Rate limits per downstream service:

- **WhatsApp Cloud API**: ~80/sec for Tier 2, higher for Tier 3. Start with 20/sec for safety.
- **Telegram Bot API**: 30/sec global, 20/min per group.
- **M-Pesa Daraja STK Push**: Generous, no published limit, but 10/sec is realistic.
- **Stripe API**: 100/sec.
- **Africa's Talking SMS**: Depends on your account tier.

Pick a rate that is comfortably below the real limit. 70% is safe.

---

## Per-Group And Per-User Rate Limits

Sometimes you have a global rate limit *and* a per-recipient rate limit. Telegram caps at 20/min per group chat regardless of your total rate. BullMQ does not have first-class support for this, but you can use grouped job names:

```javascript
// Add a group rate limiter using named jobs
await notificationsQueue.add(`telegram:${chatId}`, data, {
  // ...
});
```

Then a second worker with a group limiter:

```javascript
// Hypothetical -- BullMQ does not ship this but the pattern is standard
```

The clean approach: have a custom helper that checks a Redis counter before each send, and if the counter is over the limit for that chat, reschedule the job with a delay.

```javascript
async function checkGroupRateLimit(chatId) {
  const client = await getClient();
  const key = `rl:telegram:${chatId}`;
  const count = await client.incr(key);
  if (count === 1) await client.expire(key, 60); // 1 minute window
  return count <= 20;
}

// In the worker:
if (job.name === "telegramSend") {
  if (!(await checkGroupRateLimit(job.data.chatId))) {
    // Reschedule this job for 30 seconds later
    throw new Error("Rate limited, retry");
  }
  await telegram.sendMessage(job.data.chatId, job.data.text);
}
```

The thrown error triggers BullMQ's retry with the backoff you configured. Thirty seconds later the rate window has passed and the job succeeds.

This is not elegant but it works. A cleaner pattern is a custom worker that splits jobs by group and runs one worker per group; too complex for today.

---

## Dead Letter Queues

After all retries fail, the job is "failed" in Bull Board. That is fine for debugging, but you often want failed jobs to go somewhere for manual review or automated alerting.

Pattern: a separate "dead" queue. When a job fails definitively, move it:

```javascript
const deadLetterQueue = new Queue("dead-letter", { connection });

worker.on("failed", async (job, err) => {
  if (job.attemptsMade >= job.opts.attempts) {
    // Final failure
    await deadLetterQueue.add(job.name, {
      originalQueue: "notifications",
      originalJobId: job.id,
      data: job.data,
      error: err.message,
      failedAt: new Date(),
    });

    // Alert the admin
    await whatsapp.sendTextMessage({
      to: env.ADMIN_PHONE,
      text: `Dead job: ${job.name} for ${job.data.to}`,
    });
  }
});
```

Now `dead-letter` is a queue full of "give up" messages. The admin sees them in Bull Board, investigates, decides to retry or discard.

Do not try to "auto-fix" dead jobs. They are in the dead queue because retries did not help; a human needs to look at them.

---

## Graceful Worker Shutdown

When you deploy a new version, the old worker process gets SIGTERM. If you kill it mid-job, the job is lost (or stuck in "active" state until it times out). BullMQ has a clean shutdown method:

```javascript
async function gracefulShutdown() {
  console.log("Shutting down worker...");
  await worker.close(); // waits for in-flight jobs to finish
  console.log("Worker shut down cleanly");
  process.exit(0);
}

process.on("SIGTERM", gracefulShutdown);
process.on("SIGINT", gracefulShutdown);
```

`worker.close()` stops accepting new jobs, waits for active jobs to complete, and then disconnects from Redis. Give it a timeout if your jobs can take a long time:

```javascript
await Promise.race([
  worker.close(),
  new Promise((r) => setTimeout(r, 30000)), // 30 seconds max
]);
```

If a job is still running after 30 seconds, the process exits anyway. BullMQ will re-process that job when a new worker starts (it is marked "stalled").

### Stalled job detection

If a worker dies without graceful shutdown, jobs it was processing get stuck. BullMQ has a "stalled" check that periodically looks for jobs that have been active too long and moves them back to wait:

```javascript
const worker = new Worker("notifications", handler, {
  connection,
  stalledInterval: 30000, // check every 30s
  maxStalledCount: 3,     // after 3 stalls, fail permanently
});
```

This is on by default. It catches "worker crashed with a job in hand" cases.

---

## Observability: Events

BullMQ emits events you can hook:

```javascript
const { QueueEvents } = require("bullmq");
const queueEvents = new QueueEvents("notifications", { connection });

queueEvents.on("waiting", ({ jobId }) => console.log(`Job ${jobId} waiting`));
queueEvents.on("active", ({ jobId }) => console.log(`Job ${jobId} active`));
queueEvents.on("completed", ({ jobId, returnvalue }) => console.log(`Job ${jobId} completed`));
queueEvents.on("failed", ({ jobId, failedReason }) => console.error(`Job ${jobId} failed: ${failedReason}`));
queueEvents.on("stalled", ({ jobId }) => console.warn(`Job ${jobId} stalled`));
```

Pipe these into your logging system, metrics pipeline, or an alerting channel. For the Marathon, `console.log` is fine; for production, use a real logger.

---

## Metrics

How many jobs per second is the queue processing? What is the average job duration? These are useful to know. BullMQ has built-in metrics:

```javascript
const counts = await notificationsQueue.getJobCounts();
// { waiting: 12, active: 3, completed: 1500, failed: 5, delayed: 0 }
```

Poll this every minute and you have a queue health dashboard. Plug it into the admin dashboard:

```jsx
<div className="grid grid-cols-5 gap-4">
  <Stat label="Waiting" value={counts.waiting} />
  <Stat label="Active" value={counts.active} />
  <Stat label="Completed" value={counts.completed} />
  <Stat label="Failed" value={counts.failed} color="red" />
  <Stat label="Delayed" value={counts.delayed} />
</div>
```

Admins spot growing backlog (`waiting` climbing steadily) before it becomes a problem.

---

## Checkpoint

1. The worker has `limiter: { max: 20, duration: 1000 }` and Bull Board shows jobs being processed at exactly that rate.
2. A group rate limit test: fire 30 jobs for the same chat, verify only 20 run within a minute.
3. A job that always throws ends up in the dead-letter queue after the retry count is exhausted.
4. The admin gets a WhatsApp when a job goes to dead-letter.
5. Sending SIGTERM to the worker while it has active jobs waits for them before exiting.
6. A job that is interrupted mid-execution (hard kill the worker) is retried when a new worker starts.
7. `getJobCounts()` returns accurate counts.

Commit:

```bash
git add .
git commit -m "feat: queue rate limiting dead letter queue and graceful shutdown"
```

---

## What You Learned

- BullMQ's `limiter` enforces global rate limits.
- Per-group rate limits are a Redis counter + retry.
- Dead-letter queues hold "give up" jobs for human review.
- `worker.close()` drains in-flight jobs before exit.
- Stalled detection catches crashed workers automatically.
- `getJobCounts()` is your queue health dashboard in one call.

Tomorrow: migrating your existing notifications and bulk sends to BullMQ.
