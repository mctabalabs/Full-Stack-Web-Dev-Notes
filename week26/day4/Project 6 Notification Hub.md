# Week 26, Day 4: Project 6 -- Multi-Channel Notification Hub

By the end of today, Project 6 is complete: a generic notification service that tries WhatsApp, falls back to SMS, handles user preferences, enforces quiet hours, and logs every attempt. The foundation was laid in Week 23's weekend; today we finish it.

**Prior concepts:** everything from Weeks 11-25. Especially the queues from Week 23 and the notification service from Week 24.

**Estimated time:** 3 hours

---

## What A Notification Hub Is

A single API your whole app uses to notify users. The caller does not care about channels; the hub picks, tries, and retries.

```javascript
await notificationHub.notify({
  userId: "user-123",
  template: "order_confirmed",
  data: { orderId: "ord-456", total: 2499 },
  priority: 2,
});
```

The hub:

1. Looks up the user's preferred channels.
2. Checks quiet hours; delays if within them.
3. Tries the first channel. If permanent fail, moves to the next.
4. Logs every attempt.
5. Fires through BullMQ for retries.

The caller gets a promise that resolves when the attempt is enqueued. It does not wait for delivery.

---

## The API

```javascript
// notification-service/hub.js
const { notificationsQueue } = require("./queues");
const { query } = require("./config/db");

async function notify({ userId, template, data, priority = 5, channels }) {
  const user = await getUserPreferences(userId);
  if (!user) throw new Error("Unknown user");

  const orderedChannels = channels || user.preferred_channels || ["whatsapp", "sms"];

  // Check quiet hours
  if (isInQuietHours(user)) {
    const delay = millisecondsUntilQuietEnd(user);
    await notificationsQueue.add("dispatch", {
      userId,
      template,
      data,
      orderedChannels,
    }, { delay, priority });
    return { status: "scheduled", scheduledFor: new Date(Date.now() + delay) };
  }

  await notificationsQueue.add("dispatch", {
    userId,
    template,
    data,
    orderedChannels,
  }, { priority });
  return { status: "queued" };
}

async function getUserPreferences(userId) {
  const { rows } = await query(
    "SELECT * FROM user_preferences WHERE user_id = $1",
    [userId]
  );
  return rows[0];
}

module.exports = { notify };
```

The hub is a thin wrapper. Real work happens in the dispatcher worker.

---

## The Dispatcher Worker

```javascript
// notification-service/workers/dispatch.worker.js
const { Worker } = require("bullmq");
const { renderTemplate } = require("../templates");
const { getUserPreferences } = require("../hub");
const { whatsappQueue, telegramQueue, smsQueue } = require("../queues");

const worker = new Worker("notifications", async (job) => {
  if (job.name !== "dispatch") return;

  const { userId, template, data, orderedChannels } = job.data;
  const user = await getUserPreferences(userId);
  if (!user) return;

  const { text, language } = renderTemplate(template, data, user.language || "en");

  for (const channel of orderedChannels) {
    try {
      await sendViaChannel(channel, user, text);
      await logAttempt({ userId, channel, status: "sent" });
      return;
    } catch (err) {
      await logAttempt({ userId, channel, status: "failed", error: err.message });
      if (isPermanent(err)) continue;
      throw err; // let BullMQ retry the same channel
    }
  }

  throw new Error("All channels failed");
});

async function sendViaChannel(channel, user, text) {
  if (channel === "whatsapp") {
    if (!user.phone) throw new Error("Permanent: no phone");
    await whatsappQueue.add("sendWhatsApp", { to: user.phone, text });
  } else if (channel === "sms") {
    if (!user.phone) throw new Error("Permanent: no phone");
    await smsQueue.add("sendSMS", { to: user.phone, text });
  } else if (channel === "telegram") {
    if (!user.telegram_chat_id) throw new Error("Permanent: no telegram");
    await telegramQueue.add("sendTelegram", { chatId: user.telegram_chat_id, text });
  } else {
    throw new Error(`Unknown channel: ${channel}`);
  }
}
```

Notice: the dispatcher itself does not call the API. It enqueues into the *channel-specific* queue. The channel workers actually talk to the APIs. Two-layer queue:

```
Caller -> notify() -> notifications queue (dispatch) -> channel queue (whatsapp/sms/telegram)
```

This layering lets you change the channel order (WhatsApp first, SMS second) without touching the channel workers. And channel rate limits apply exactly to the channel workers, not to the dispatcher.

---

## Templates

Templates live in a small registry:

```javascript
// notification-service/templates.js
const TEMPLATES = {
  order_confirmed: {
    en: ({ orderId, total }) => `Your order ${orderId} is confirmed. Total: KSh ${total}`,
    sw: ({ orderId, total }) => `Oda yako ${orderId} imethibitishwa. Jumla: KSh ${total}`,
  },
  order_shipped: {
    en: ({ orderId }) => `Your order ${orderId} has been dispatched.`,
    sw: ({ orderId }) => `Oda yako ${orderId} imetumwa.`,
  },
  appointment_reminder: {
    en: ({ when, serviceName }) => `Reminder: ${serviceName} tomorrow at ${when}.`,
    sw: ({ when, serviceName }) => `Kumbusho: ${serviceName} kesho saa ${when}.`,
  },
};

function renderTemplate(name, data, language = "en") {
  const template = TEMPLATES[name];
  if (!template) throw new Error(`Unknown template: ${name}`);
  const fn = template[language] || template.en;
  return { text: fn(data), language };
}

module.exports = { renderTemplate, TEMPLATES };
```

Every outbound message goes through the template registry. If you need to change wording, you change one function. Two languages ship by default; adding more is a new key.

---

## Quiet Hours

```javascript
function isInQuietHours(user) {
  if (!user.quiet_hours_start || !user.quiet_hours_end) return false;
  const now = new Date();
  const nowMinutes = now.getHours() * 60 + now.getMinutes();

  const start = parseTimeString(user.quiet_hours_start);
  const end = parseTimeString(user.quiet_hours_end);

  if (start < end) {
    return nowMinutes >= start && nowMinutes < end;
  } else {
    // Overnight quiet hours (e.g., 22:00 to 06:00)
    return nowMinutes >= start || nowMinutes < end;
  }
}

function millisecondsUntilQuietEnd(user) {
  const now = new Date();
  const end = new Date(now);
  const [h, m] = user.quiet_hours_end.split(":").map(Number);
  end.setHours(h, m, 0, 0);
  if (end <= now) end.setDate(end.getDate() + 1);
  return end.getTime() - now.getTime();
}
```

During quiet hours, the job is delayed in BullMQ. When the quiet window ends, the job becomes active and the dispatcher processes it. No manual clock-watching.

Exception: high-priority jobs should bypass quiet hours. Add a `urgent: true` flag:

```javascript
if (urgent || !isInQuietHours(user)) {
  // send now
}
```

Urgent messages are payment failures, critical alerts, security issues. Marketing is not urgent.

---

## Attempt Logging

```javascript
async function logAttempt({ userId, channel, status, error }) {
  await query(
    `INSERT INTO notification_attempts (user_id, channel, status, error, created_at)
     VALUES ($1, $2, $3, $4, NOW())`,
    [userId, channel, status, error || null]
  );
}
```

Every attempt on every channel gets a row. An admin page reads from this table and shows "why did my customer not get their message?".

---

## Admin Page

```jsx
// app/admin/notifications/page.js
import { query } from "@/lib/db";

export default async function NotificationsPage() {
  const { rows: summary } = await query(
    `SELECT channel, status, COUNT(*)::int AS count
     FROM notification_attempts
     WHERE created_at > NOW() - INTERVAL '24 hours'
     GROUP BY channel, status`
  );

  const { rows: failed } = await query(
    `SELECT id, user_id, channel, error, created_at
     FROM notification_attempts
     WHERE status = 'failed'
       AND created_at > NOW() - INTERVAL '24 hours'
     ORDER BY created_at DESC LIMIT 50`
  );

  return (
    <div className="max-w-4xl mx-auto p-8">
      <h1 className="text-2xl font-bold mb-6">Notifications</h1>

      <h2 className="font-semibold mb-2">Last 24h</h2>
      <table className="w-full text-sm mb-8">
        <thead><tr><th>Channel</th><th>Status</th><th>Count</th></tr></thead>
        <tbody>
          {summary.map((s) => (
            <tr key={`${s.channel}-${s.status}`} className="border-t">
              <td>{s.channel}</td><td>{s.status}</td><td>{s.count}</td>
            </tr>
          ))}
        </tbody>
      </table>

      <h2 className="font-semibold mb-2">Recent failures</h2>
      <ul>
        {failed.map((f) => (
          <li key={f.id} className="border-b py-2 text-sm">
            {new Date(f.created_at).toLocaleString("en-KE")} -
            {f.channel} -
            {f.user_id} -
            {f.error}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

Two tables. Answers the "is notification healthy?" question at a glance and "why did it fail?" below.

---

## Checkpoint

1. Calling `notify({ userId, template: "order_confirmed", data: {...} })` enqueues a job.
2. The dispatcher reads it, picks the first channel, and enqueues into the channel queue.
3. The channel worker actually sends the message.
4. A user with `["whatsapp", "sms"]` preference whose WhatsApp fails (fake the error) gets an SMS attempt next.
5. Quiet hours delay the job to after the window.
6. `/admin/notifications` shows yesterday's summary and failures.
7. Adding a new channel (`email`) is adding a function to the dispatcher and two queue entries.

Commit:

```bash
git add .
git commit -m "feat: project 6 multi-channel notification hub"
```

---

## What You Learned

- Two-layer queuing: a dispatch queue chooses a channel; channel queues send.
- Templates are pure functions in a registry.
- Quiet hours use BullMQ delays, not clock checks.
- Attempt logs turn "why did my customer not get their message?" into a SQL query.

Tomorrow is the recap for Phase 4 and the weekend polishes both projects.
