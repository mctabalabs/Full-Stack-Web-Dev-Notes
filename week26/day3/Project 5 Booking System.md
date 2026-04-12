# Week 26, Day 3: Project 5 -- Booking & Appointments System

The Phase 4 roadmap calls for a Booking & Appointments System as Project 5. Today we sketch and build the core of it -- a reusable scheduling engine that handles appointments, availability, conflicts, and reminders. This is also Day 3 of Week 26, so we fit it into a single day.

**Prior-week concepts you will use today:**
- Cron jobs (Week 21) for reminders.
- Queues (Week 23) for scheduled sends.
- The notification hub groundwork (Week 23 weekend).
- Everything from Phases 2 and 3.

**Estimated time:** 4 hours

---

## Why This Project

Booking is everywhere: a doctor's office, a tutor, a beauty salon, a boda dispatch confirming pickups, a laundromat slot, a small gym. Project 5 is a generic booking engine any of these use cases can reuse.

The hard part is not the UI. It is the correctness under concurrency: two customers must not be able to book the same slot. The scheduling engine is a tight loop of "check availability, reserve slot, confirm" that has to be race-safe.

---

## The Data Model

```sql
CREATE TABLE providers (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  phone TEXT,
  timezone TEXT NOT NULL DEFAULT 'Africa/Nairobi'
);

CREATE TABLE services (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  provider_id UUID REFERENCES providers(id),
  name TEXT NOT NULL,
  duration_minutes INTEGER NOT NULL CHECK (duration_minutes > 0),
  price_cents INTEGER NOT NULL DEFAULT 0
);

CREATE TABLE availability_windows (
  id BIGSERIAL PRIMARY KEY,
  provider_id UUID REFERENCES providers(id),
  day_of_week INTEGER NOT NULL CHECK (day_of_week BETWEEN 0 AND 6),
  start_minute INTEGER NOT NULL,  -- minutes from midnight
  end_minute INTEGER NOT NULL
);

CREATE TABLE appointments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  provider_id UUID NOT NULL REFERENCES providers(id),
  service_id UUID NOT NULL REFERENCES services(id),
  customer_name TEXT NOT NULL,
  customer_phone TEXT NOT NULL,
  start_at TIMESTAMPTZ NOT NULL,
  end_at TIMESTAMPTZ NOT NULL,
  status TEXT NOT NULL DEFAULT 'pending'
    CHECK (status IN ('pending', 'confirmed', 'cancelled', 'no_show', 'completed')),
  notes TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  EXCLUDE USING gist (
    provider_id WITH =,
    tstzrange(start_at, end_at) WITH &&
  ) WHERE (status NOT IN ('cancelled', 'no_show'))
);
```

The clever part is the `EXCLUDE USING gist` constraint. Postgres enforces at the database level that two non-cancelled appointments for the same provider cannot have overlapping time ranges. A race condition that tries to double-book fails with a clean error.

The `gist` index type uses the `btree_gist` extension. Enable it once per database:

```sql
CREATE EXTENSION IF NOT EXISTS btree_gist;
```

---

## The Availability Query

"Find me all free 30-minute slots for this provider on this day":

```javascript
async function findFreeSlots(providerId, date, serviceMinutes = 30) {
  const dayOfWeek = new Date(date).getDay();

  // Get the provider's availability windows for that day
  const { rows: windows } = await query(
    `SELECT start_minute, end_minute
     FROM availability_windows
     WHERE provider_id = $1 AND day_of_week = $2`,
    [providerId, dayOfWeek]
  );

  // Get existing appointments on that day
  const dayStart = new Date(date);
  dayStart.setHours(0, 0, 0, 0);
  const dayEnd = new Date(dayStart);
  dayEnd.setDate(dayEnd.getDate() + 1);

  const { rows: appts } = await query(
    `SELECT start_at, end_at FROM appointments
     WHERE provider_id = $1
       AND start_at >= $2 AND start_at < $3
       AND status NOT IN ('cancelled', 'no_show')
     ORDER BY start_at`,
    [providerId, dayStart, dayEnd]
  );

  // Walk each window, skipping slots that overlap appointments
  const freeSlots = [];
  for (const window of windows) {
    for (let t = window.start_minute; t + serviceMinutes <= window.end_minute; t += serviceMinutes) {
      const slotStart = new Date(dayStart);
      slotStart.setMinutes(t);
      const slotEnd = new Date(slotStart);
      slotEnd.setMinutes(slotStart.getMinutes() + serviceMinutes);

      const overlaps = appts.some((a) =>
        slotStart < new Date(a.end_at) && slotEnd > new Date(a.start_at)
      );
      if (!overlaps) freeSlots.push({ start: slotStart, end: slotEnd });
    }
  }

  return freeSlots;
}
```

This generates candidate slots from availability windows, subtracts occupied slots, returns the remaining free ones. Not efficient for massive calendars but perfectly fine for a small business with dozens of appointments per day.

---

## Booking An Appointment

```javascript
async function bookAppointment({ providerId, serviceId, customer, startAt }) {
  const { rows: serviceRows } = await query(
    "SELECT duration_minutes FROM services WHERE id = $1 AND provider_id = $2",
    [serviceId, providerId]
  );
  const service = serviceRows[0];
  if (!service) throw new Error("Service not found");

  const endAt = new Date(new Date(startAt).getTime() + service.duration_minutes * 60000);

  try {
    const { rows } = await query(
      `INSERT INTO appointments
       (provider_id, service_id, customer_name, customer_phone, start_at, end_at, status)
       VALUES ($1, $2, $3, $4, $5, $6, 'pending')
       RETURNING *`,
      [providerId, serviceId, customer.name, customer.phone, startAt, endAt]
    );
    return rows[0];
  } catch (err) {
    // The EXCLUDE constraint throws 23P01 on overlap
    if (err.code === "23P01") {
      throw new Error("Slot no longer available");
    }
    throw err;
  }
}
```

No explicit conflict check. The database enforces it. Two customers racing for the same slot: one succeeds, the other gets a clear error. Retry-safe by construction.

---

## Reminder Cron

A job that runs every hour and sends reminders for appointments starting in 24 hours:

```javascript
async function sendAppointmentReminders() {
  const tomorrow = new Date(Date.now() + 24 * 60 * 60 * 1000);
  const tomorrowPlus1h = new Date(tomorrow.getTime() + 60 * 60 * 1000);

  const { rows: appts } = await query(
    `SELECT a.id, a.customer_name, a.customer_phone, a.start_at, s.name AS service_name, p.name AS provider_name
     FROM appointments a
     JOIN services s ON s.id = a.service_id
     JOIN providers p ON p.id = a.provider_id
     WHERE a.status IN ('pending', 'confirmed')
       AND a.start_at >= $1 AND a.start_at < $2`,
    [tomorrow, tomorrowPlus1h]
  );

  for (const appt of appts) {
    const text = `Reminder: ${appt.service_name} with ${appt.provider_name} tomorrow at ${new Date(appt.start_at).toLocaleTimeString("en-KE", { hour: "2-digit", minute: "2-digit" })}.`;
    await whatsappQueue.add("sendWhatsApp", {
      to: appt.customer_phone,
      text,
    });
  }
}
```

Enqueue, do not send directly. The WhatsApp worker handles retries. Your cron job is fast and deterministic.

Register in the cron registry as `{ name: "appointment-reminders", schedule: "0 * * * *", run: sendAppointmentReminders }`.

---

## Booking UI

For a shop owner, a simple admin page:

1. List services.
2. Pick a date -> see free slots.
3. Click a slot -> enter customer details -> book.

For a customer (if you want a public page):

1. Pick a service.
2. Pick a date.
3. See free slots.
4. Fill in name + phone.
5. Submit -> enqueue Next.js Server Action -> appointments row.

Both reuse `findFreeSlots` and `bookAppointment`. The UI is just the wrapper.

---

## Cancellations And No-Shows

```javascript
async function cancelAppointment(appointmentId, cancelledBy) {
  const result = await query(
    `UPDATE appointments SET status = 'cancelled' WHERE id = $1 AND status NOT IN ('cancelled', 'completed') RETURNING *`,
    [appointmentId]
  );
  if (result.rowCount === 0) throw new Error("Cannot cancel");

  // Notify the customer
  const appt = result.rows[0];
  await whatsappQueue.add("sendWhatsApp", {
    to: appt.customer_phone,
    text: `Your appointment has been cancelled.`,
  });

  return appt;
}
```

No-shows are a separate status. A cron job marks appointments as `no_show` 30 minutes after their scheduled start if they are still `pending`:

```javascript
async function markNoShows() {
  await query(
    `UPDATE appointments SET status = 'no_show'
     WHERE status = 'pending' AND start_at < NOW() - INTERVAL '30 minutes'`
  );
}
```

---

## Checkpoint

1. Database has `providers`, `services`, `availability_windows`, `appointments` tables with the `EXCLUDE` constraint.
2. `findFreeSlots` returns slots for a given provider and date.
3. `bookAppointment` succeeds on the first booking and fails cleanly on a conflict.
4. Two parallel calls to `bookAppointment` for the same slot: one succeeds.
5. Cron sends reminders 24 hours before the appointment.
6. Cancellation sends a WhatsApp to the customer.
7. No-shows are auto-marked after 30 minutes.

Commit:

```bash
git add .
git commit -m "feat: project 5 booking and appointments system core"
```

---

## What You Learned

- `EXCLUDE USING gist` gives you database-level conflict protection for time ranges.
- Slot generation walks availability windows and subtracts appointments.
- Reminders go through the notification queue, not directly.
- Cancellations and no-shows are transitions, not deletions.

Tomorrow: the notification hub (Project 6).
