# Week 23, Day 2: Retries, Priorities, and Delayed Jobs

By the end of today, your queue handles failure gracefully: transient errors retry with backoff, permanent errors die quickly, priority jobs jump the queue, and "send this in 2 hours" is a one-line config.

**Prior concepts:** BullMQ basics (Week 23 Day 1).

**Estimated time:** 3 hours

---

## Retries With Backoff

When you add a job, specify retry policy:

```javascript
await notificationsQueue.add("sendWhatsApp", data, {
  attempts: 5,
  backoff: {
    type: "exponential",
    delay: 1000, // 1 second initial delay
  },
});
```

BullMQ retries up to 5 times. First retry at 1 second, then 2, 4, 8, 16. Total of about 31 seconds if all retries fail.

You can set defaults on the queue so you do not pass it per-add:

```javascript
const notificationsQueue = new Queue("notifications", {
  connection,
  defaultJobOptions: {
    attempts: 5,
    backoff: { type: "exponential", delay: 1000 },
    removeOnComplete: 1000, // keep last 1000 completed jobs
    removeOnFail: 5000,     // keep last 5000 failed jobs
  },
});
```

`removeOnComplete` and `removeOnFail` matter for disk space. Without them, every completed job sits in Redis forever. 1000 is generous enough to debug recent issues.

---

## Distinguishing Permanent From Transient

Not all errors should be retried. If the phone number is invalid, retrying does not help. Throw a special error or set `delay: 0, attempts: 1` on such jobs, or check inside the worker:

```javascript
const worker = new Worker("notifications", async (job) => {
  try {
    await whatsapp.sendTextMessage({ to: job.data.to, text: job.data.message });
  } catch (err) {
    if (err.response?.status === 400) {
      // Permanent: bad phone number, bad template
      throw new Error(`Permanent failure: ${err.message}`, { cause: "permanent" });
    }
    // Transient: let BullMQ retry
    throw err;
  }
}, { connection });
```

BullMQ 4+ supports "do not retry" via `job.moveToFailed()` programmatically. For simplicity, just decide in the try/catch and always throw; BullMQ will retry up to `attempts`. For truly permanent errors, log them to a dead letter queue instead of hoping retries help.

---

## Concurrency

By default a worker processes one job at a time. Bump it:

```javascript
const worker = new Worker("notifications", handler, { connection, concurrency: 10 });
```

Now the worker runs up to 10 jobs in parallel. This matters for I/O-bound work (API calls, database reads). For CPU-bound work, higher concurrency on a single Node process does not help -- Node is single-threaded per process. Spin up multiple worker processes instead.

Pick concurrency based on the downstream bottleneck:

- WhatsApp/Telegram/SMS: 5-10 (rate limits are the bottleneck).
- Database writes: 20-50 (depends on pool size).
- CPU-heavy work (PDF generation, image processing): 1 per process.

---

## Priorities

Some jobs matter more than others. A payment confirmation WhatsApp should outrun a marketing blast. BullMQ supports priorities:

```javascript
// Critical: send ASAP
await notificationsQueue.add("sendWhatsApp", data, { priority: 1 });

// Normal
await notificationsQueue.add("sendWhatsApp", data, { priority: 5 });

// Marketing blast: drip out behind everything else
await notificationsQueue.add("sendWhatsApp", data, { priority: 10 });
```

Lower number = higher priority. The worker picks the lowest-priority job available.

Be careful: priority only affects ordering *within* a queue. If you have thousands of marketing jobs all priority 10, a priority 1 job added right now still waits behind any currently-executing marketing jobs (because they are already off the queue). For hard real-time needs, use a separate queue with a dedicated worker.

---

## Delayed Jobs

"Send this in 2 hours" is one line:

```javascript
await notificationsQueue.add("sendWhatsApp", data, {
  delay: 2 * 60 * 60 * 1000, // 2 hours in ms
});
```

The job sits in a delayed state in Redis until the delay passes, then moves to active. BullMQ handles the scheduling internally -- no cron job needed.

Use cases:

- **Reminder**: send a follow-up WhatsApp 2 hours after the initial confirmation.
- **Retry cooldown**: if a payment fails, wait 10 minutes before retrying.
- **Scheduled publishes**: post a social media update at 9am tomorrow.

For repeating jobs (every hour, every day), use `repeat`:

```javascript
await notificationsQueue.add("dailyDigest", data, {
  repeat: { pattern: "0 9 * * *", tz: "Africa/Nairobi" },
});
```

BullMQ supports cron patterns directly. You can replace `node-cron` for scheduled work if you want -- but a mix of BullMQ repeat + node-cron is fine. Use BullMQ repeat for work that should be retried and observed in Bull Board; use node-cron for simple "run this function" scheduling.

---

## Job Chains And Flows

"Process this order, then send confirmation WhatsApp, then send SMS receipt" -- three dependent jobs. BullMQ has `FlowProducer`:

```javascript
const { FlowProducer } = require("bullmq");
const flow = new FlowProducer({ connection });

await flow.add({
  name: "process-order",
  queueName: "payments",
  data: { orderId },
  children: [
    {
      name: "sendWhatsApp",
      queueName: "notifications",
      data: { /* ... */ },
      opts: { priority: 1 },
    },
    {
      name: "sendSMS",
      queueName: "notifications",
      data: { /* ... */ },
    },
  ],
});
```

Children run after the parent completes. Useful for "confirm payment then notify three channels".

For most cases, you do not need flows -- adding child jobs from the parent's handler is simpler. Flows matter when the shape of the dependency graph is significant (fan-out, joins).

---

## Progress Reporting

Long jobs can report progress:

```javascript
const worker = new Worker("reports", async (job) => {
  const total = 1000;
  for (let i = 0; i < total; i++) {
    // ... do one step
    if (i % 100 === 0) {
      await job.updateProgress((i / total) * 100);
    }
  }
});
```

Bull Board shows a progress bar for jobs that report. Useful for long-running reconciliation, bulk sends, exports.

---

## Checkpoint

1. A failing job retries with exponential backoff and shows each attempt in Bull Board.
2. A job with 3 successive failures ends up in the "failed" state.
3. Running the notifications worker with `concurrency: 5` processes 5 jobs at a time.
4. A priority 1 job arrives in the middle of priority 10 jobs and runs next.
5. A job added with `delay: 30000` shows as "delayed" in Bull Board for 30 seconds before running.
6. A repeat job with cron pattern fires on schedule.
7. A long-running job reports progress and Bull Board shows the bar.

Commit:

```bash
git add .
git commit -m "feat: queue retries concurrency priorities and delayed jobs"
```

---

## What You Learned

- Retries with exponential backoff are one line of config.
- Concurrency is per-worker; I/O jobs benefit more than CPU jobs.
- Priority orders jobs within a queue.
- Delayed jobs replace `setTimeout` + in-memory state.
- Repeat jobs can replace some cron use cases.

Tomorrow: rate limiting, dead letter queues, and graceful worker shutdown.
