# Week 23 Weekend: Multi-Channel Notification Hub Groundwork

Start on Project 6 -- the Multi-Channel Notification Hub. This weekend's deliverable is a generic `notify()` function with channel fallback, queue integration, and a minimal admin UI. Week 26 ships the full product.

**Estimated time:** 4-6 hours.

**Deadline:** Monday before Week 24.

---

## Functional Requirements

### The `notify()` function

```javascript
await notify({
  userId: "user-123",
  message: "Your order has shipped.",
  channels: ["whatsapp", "telegram", "sms"],
  priority: 2,
});
```

- Tries channels in order.
- If the first channel fails (permanent failure), moves to the second.
- Transient failures retry via BullMQ before moving on.
- Logs every attempt to a `notification_attempts` table.

### Channels

- WhatsApp (existing).
- Telegram (existing).
- SMS via Africa's Talking (existing).
- Email via a simple SMTP sender (optional; not critical).

### User preferences

Each user has a preferred channel order stored in `user_preferences`:

```sql
CREATE TABLE user_preferences (
  user_id TEXT PRIMARY KEY,
  phone TEXT,
  telegram_chat_id BIGINT,
  email TEXT,
  preferred_channels TEXT[] DEFAULT ARRAY['whatsapp', 'telegram', 'sms'],
  quiet_hours_start TIME,
  quiet_hours_end TIME
);
```

The `notify()` function honours the user's preference list unless the caller explicitly overrides.

### Attempt logging

```sql
CREATE TABLE notification_attempts (
  id BIGSERIAL PRIMARY KEY,
  user_id TEXT NOT NULL,
  message TEXT NOT NULL,
  channel TEXT NOT NULL,
  status TEXT NOT NULL CHECK (status IN ('queued', 'sent', 'failed', 'skipped')),
  error TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

Every attempt writes a row. When one channel fails and the next is tried, you see two rows -- one `failed` and one `sent`.

### Fallback logic

```javascript
async function notify({ userId, message, channels, priority = 5 }) {
  const user = await getUserPreferences(userId);
  const ordered = channels || user.preferred_channels;

  for (const channel of ordered) {
    try {
      await sendViaChannel(channel, user, message, priority);
      return { channel, status: "sent" };
    } catch (err) {
      if (isPermanent(err)) {
        await logAttempt({ userId, message, channel, status: "failed", error: err.message });
        continue; // try next channel
      }
      throw err; // transient, let queue retry
    }
  }
  throw new Error("All channels failed");
}
```

### Quiet hours

Do not send notifications between a user's `quiet_hours_start` and `quiet_hours_end`. Instead, schedule the job for `quiet_hours_end`:

```javascript
if (isInQuietHours(user)) {
  await queue.add(jobName, data, { delay: millisecondsUntilQuietHoursEnd(user) });
  return { status: "queued", scheduledFor: end };
}
```

---

## Admin UI

A small page at `/admin/notifications` showing:
- Total attempts today by channel.
- Attempts per user (top 10).
- A search box to look up a specific user's history.
- Failed attempts in the last 24 hours.

No fancy charts; just tables.

---

## Grading Rubric (50 pts)

| Area | Points |
|---|---|
| `notify()` function with fallback | 15 |
| Every attempt logged | 10 |
| User preferences respected | 10 |
| Quiet hours delay | 5 |
| Admin UI | 5 |
| README + test instructions | 5 |

---

## Hints

**Do not try to implement all four channels perfectly.** WhatsApp + Telegram is enough. SMS is a stretch. Email is optional.

**Fake the failures for testing.** Add a query parameter or env var that forces the next send to fail, so you can verify the fallback logic without actually breaking your WhatsApp integration.

**Use existing queues.** You already have WhatsApp, Telegram, and SMS queues from Week 23 Day 4. Do not build a new one -- reuse.

**User preferences seed.** Hardcode three test users with different preferences in `db/seed.sql`. Test the fallback with each one.

Next week: microservices. Your queues and services make the split almost trivial.
