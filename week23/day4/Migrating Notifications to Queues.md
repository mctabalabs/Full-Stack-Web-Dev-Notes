# Week 23, Day 4: Migrating Notifications to Queues

By the end of today, the notifications that used to fire directly from your outbox and cron jobs now go through BullMQ queues. Everything has retries, rate limiting, and observability -- for free. The migration is small in code but large in correctness.

**Prior concepts:** Week 23 Days 1-3.

**Estimated time:** 2-3 hours

---

## What We Are Migrating

Three places fire notifications in your current codebase:

1. **Outbox worker** (Week 18) -- sends order confirmations when an `order.paid` event lands.
2. **Cron jobs** (Week 21) -- daily reports, overdue reminders, cycle reminders.
3. **Chama bot** (Week 22) -- contribution announcements.

Each of them currently calls `whatsapp.sendTextMessage(...)` or `telegram.sendMessage(...)` directly. That works for small volumes. Today we route all of them through queues.

---

## One Queue Per Channel

```javascript
// server/queues/index.js (extended)
const whatsappQueue = new Queue("whatsapp", {
  connection,
  defaultJobOptions: {
    attempts: 5,
    backoff: { type: "exponential", delay: 1000 },
    removeOnComplete: 1000,
    removeOnFail: 5000,
  },
});

const telegramQueue = new Queue("telegram", {
  connection,
  defaultJobOptions: {
    attempts: 5,
    backoff: { type: "exponential", delay: 1000 },
    removeOnComplete: 1000,
    removeOnFail: 5000,
  },
});

const smsQueue = new Queue("sms", {
  connection,
  defaultJobOptions: {
    attempts: 3,
    backoff: { type: "exponential", delay: 2000 },
    removeOnComplete: 1000,
    removeOnFail: 5000,
  },
});
```

Three channels, three queues. Why not one? Because their retry semantics differ (SMS costs money and should retry less), their rate limits differ, and Bull Board shows them separately.

---

## Worker Files

```javascript
// server/workers/whatsapp.worker.js
const { Worker } = require("bullmq");
const whatsapp = require("../services/whatsapp.service");

const worker = new Worker("whatsapp", async (job) => {
  const { to, text, templateName, components } = job.data;

  if (templateName) {
    await whatsapp.sendTemplateMessage({ to, templateName, components });
  } else {
    await whatsapp.sendTextMessage({ to, text });
  }
}, {
  connection,
  concurrency: 10,
  limiter: { max: 20, duration: 1000 },
});

worker.on("completed", (job) => console.log(`[whatsapp] Job ${job.id} done`));
worker.on("failed", (job, err) => console.error(`[whatsapp] Job ${job.id} failed: ${err.message}`));

process.on("SIGTERM", async () => {
  await worker.close();
  process.exit(0);
});

module.exports = worker;
```

```javascript
// server/workers/telegram.worker.js
const { Worker } = require("bullmq");
const telegram = require("../services/telegram.service");

const worker = new Worker("telegram", async (job) => {
  const { chatId, text, parseMode, replyMarkup } = job.data;
  await telegram.sendMessage(chatId, text, {
    parse_mode: parseMode,
    reply_markup: replyMarkup,
  });
}, {
  connection,
  concurrency: 5,
  limiter: { max: 25, duration: 1000 },
});

// ... same event handlers and shutdown
```

Start both as separate processes (or require them from `server/worker.js` if you prefer a single worker binary).

---

## Updating The Callers

The outbox worker currently does:

```javascript
if (row.event_type === "order.paid") {
  await orderNotifications.sendOrderConfirmation(row.payload.orderId);
}
```

Replace with a queue add:

```javascript
if (row.event_type === "order.paid") {
  await whatsappQueue.add("sendOrderConfirmation", { orderId: row.payload.orderId }, {
    priority: 2, // higher than marketing
  });
}
```

And the worker handles the send:

```javascript
const worker = new Worker("whatsapp", async (job) => {
  if (job.name === "sendOrderConfirmation") {
    await orderNotifications.sendOrderConfirmation(job.data.orderId);
    return;
  }
  // ... other job types
}, { /* ... */ });
```

This keeps `orderNotifications.sendOrderConfirmation` unchanged -- it still reads from Postgres and calls the WhatsApp API. The queue just sits in between, adding retries and rate limiting.

---

## Cron Jobs Now Add To Queues

The Week 21 daily report cron currently does:

```javascript
await whatsapp.sendTextMessage({ to: env.SHOP_OWNER_PHONE, text: message });
```

Becomes:

```javascript
await whatsappQueue.add("dailyReport", {
  to: env.SHOP_OWNER_PHONE,
  text: message,
});
```

One-line change. The cron fires, adds a job, moves on. The worker sends the message. If it fails, retries happen without the cron knowing. Next cron fire does not double-send because the cron is just adding jobs.

Idempotency in the cron: use `jobId` to prevent duplicate adds within a day:

```javascript
await whatsappQueue.add("dailyReport", data, {
  jobId: `daily-report-${new Date().toISOString().slice(0, 10)}`,
});
```

BullMQ rejects adding a job with an existing id. Two cron fires on the same day = one job. Restart safety.

---

## Chama Announcements

The chama worker currently:

```javascript
await telegramService.sendMessage(c.chama_id, "New contribution");
```

Becomes:

```javascript
await telegramQueue.add("chamaAnnouncement", {
  chatId: c.chama_id,
  text: "New contribution",
}, {
  priority: 3,
});
```

---

## Per-Recipient Rate Limiting For Telegram Groups

Telegram's 20-messages-per-minute-per-group rule needs its own check. Add a small wrapper in the worker:

```javascript
async function checkGroupLimit(chatId) {
  const client = await getClient();
  const key = `tg:rl:${chatId}`;
  const count = await client.incr(key);
  if (count === 1) await client.expire(key, 60);
  return count <= 20;
}

const worker = new Worker("telegram", async (job) => {
  if (!(await checkGroupLimit(job.data.chatId))) {
    // Will retry with the exponential backoff
    throw new Error("Telegram group rate limit");
  }
  await telegram.sendMessage(job.data.chatId, job.data.text, job.data.opts);
}, { /* ... */ });
```

A job that hits the group limit throws, BullMQ retries after 30 seconds or so, by which time the window has reset. No manual timing.

---

## Before/After

Before the migration:

- Notifications fire inline, blocking the caller.
- A dead WhatsApp API outage causes cascading failures.
- Rate limits are hand-rolled sleeps.
- No visibility into which sends failed.

After the migration:

- Notifications enqueue in milliseconds, return immediately.
- WhatsApp API outages just queue up work; retries drain when it recovers.
- Rate limits are config on the worker.
- Bull Board shows every job, its retries, and its result.

Customers never see the difference on happy paths. On bad days they still get their messages -- a few minutes later than on good days, but they come.

---

## Checkpoint

1. Placing an order on the shop now enqueues a WhatsApp job instead of sending directly; the job shows up in Bull Board and completes.
2. Stopping the WhatsApp worker (`kill` it) while the shop is taking orders: jobs pile up in the queue; restart worker and they all drain.
3. Killing the WhatsApp API (bad URL in config) causes jobs to fail, retry, and eventually end up in dead-letter.
4. Daily-report cron uses `jobId` and cannot be added twice on the same day.
5. Rate-limit test: add 50 Telegram jobs for one group at once; verify no more than 20 run in the first minute.

Commit:

```bash
git add .
git commit -m "refactor: route notifications through bullmq queues"
```

---

## What You Learned

- Migrating to queues is a small code change with large safety benefits.
- Producer code adds jobs; worker code sends messages. One-way coupling.
- `jobId` gives you add-time idempotency.
- Bull Board becomes the operational dashboard for the entire notification layer.

Tomorrow is the recap. The weekend applies this to Project 6 groundwork (Multi-Channel Notification Hub), which we finish in Week 26.
