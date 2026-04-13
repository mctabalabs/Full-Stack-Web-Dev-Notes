# Week 24 - Day 4 Assignment

## Title
Contracts, Versioning, And Observability

## Overview
Day 4 makes the notification contract enforceable (schema validation), versions it, and adds request tracing so you can follow a single order through every service that touched it.

## Learning Objectives Assessed
- Validate job data with a schema (zod)
- Version the contract and handle both versions
- Emit and propagate a trace ID
- Structure logs so you can `grep traceId`

## Prerequisites
- Day 3 completed

## AI Usage Rules

**Ratio this week:** 45% manual / 55% AI
**Habit:** Architecture dip. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Schema boilerplate.
- **NOT ALLOWED FOR:** The versioning policy (one version, two versions, breaking cutoff).
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Zod schema

**What to do:**
```bash
npm install zod
```

In `services/notifications/src/schema.js`:

```javascript
const { z } = require("zod");
const WhatsappV1 = z.object({
  version: z.literal(1),
  phone: z.string().regex(/^\+?\d{10,15}$/),
  body: z.string().min(1).max(4096),
  traceId: z.string().optional(),
});
module.exports = { WhatsappV1 };
```

In the worker, `WhatsappV1.parse(job.data)` and reject (UnrecoverableError) anything that fails.

**Expected output:**
Schema enforced.

### Task 2: Add version field to producer

**What to do:**
Everywhere the main API enqueues notifications, add `version: 1`. Ship both sides in the same commit.

**Expected output:**
All enqueues carry the version.

### Task 3: Trace ID

**What to do:**
In the main API, generate a trace ID per request (`crypto.randomUUID()`) and attach it to the request object. When enqueuing a notification, include `traceId: req.traceId`. In the worker, log it on every line.

Example:
```
api: {traceId: "abc", event: "order.created", orderId: "..."}
worker: {traceId: "abc", event: "whatsapp.send", phone: "..."}
```

**Expected output:**
`grep abc` finds both lines.

### Task 4: Version bump preview

**What to do:**
In `versioning.md`, write 5-7 sentences:
- How would you introduce WhatsappV2 without breaking WhatsappV1 producers still in flight?
- Where in the worker do you branch on `version`?
- When can you delete the V1 handler? (When no V1 jobs are in the queue AND no producers still emit V1.)

Your own words.

**Expected output:**
`versioning.md` committed.

### Task 5: Minimal tracing dashboard

**What to do:**
Create a tiny admin endpoint `GET /admin/trace/:id` that queries your log file (or a log table) and returns all events for that trace.

If you write logs to stdout only, pipe them to a file first. Or, stretch goal: use a real log aggregator.

**Expected output:**
Endpoint returns the events.

## Stretch Goals (Optional - Extra Credit)

- Adopt OpenTelemetry and export traces to Jaeger.
- Use `pino` with a `traceId` child logger.
- Add `spanId` and parent/child relationships.

## Submission Requirements

- **What to submit:** Repo, schema, notes, `AI_AUDIT.md`.
- **Deadline:** End of Day 4

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Zod schema | 20 | Validated in worker. |
| Version field on producer | 15 | Added everywhere. |
| Trace ID propagation | 25 | Visible in both services. |
| Versioning notes | 20 | Three questions answered. |
| Trace endpoint | 15 | Returns events. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Validating on the producer only.** The consumer is the trust boundary.
- **Version field as a number only.** Prefer a literal type so TS/zod can branch.
- **One log file per service with no correlation.** Trace IDs are the whole point.

## Resources

- Day 4 reading: [Contracts Versioning and Observability.md](./Contracts%20Versioning%20and%20Observability.md)
- Week 24 AI boundaries: [../ai.md](../ai.md)
