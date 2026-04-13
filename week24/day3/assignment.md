# Week 24 - Day 3 Assignment

## Title
Inter-Service Communication -- Sync, Async, And Event-Driven

## Overview
Day 3 compares the three main ways services talk to each other: synchronous HTTP, asynchronous queues, and event-driven pub/sub. You implement one of each in a tiny demo and feel the consequences.

## Learning Objectives Assessed
- Make a synchronous HTTP call between two local services
- Make an asynchronous queue call
- Publish and subscribe to Redis pub/sub events
- Choose the right pattern for a given interaction

## Prerequisites
- Day 2 completed

## AI Usage Rules

**Ratio this week:** 45% manual / 55% AI
**Habit:** Architecture dip. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Pattern scaffolds.
- **NOT ALLOWED FOR:** Picking the pattern for your real interactions.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Sync HTTP

**What to do:**
Main API calls notification service HTTP `POST /send` with a body. Wire it with fetch, 2 second timeout.

Experiment: kill the notification service and watch the main API fail fast on the POST.

**Expected output:**
Failure visible on the main API.

### Task 2: Async queue

**What to do:**
Same interaction but via BullMQ (you already have this from Week 23). Kill the worker. The main API succeeds; the job waits.

**Expected output:**
Success on API; job pending in Redis.

### Task 3: Event-driven pub/sub

**What to do:**
Using Redis pub/sub:

```javascript
// Publisher
const pub = new Redis();
await pub.publish("order.created", JSON.stringify({ orderId }));

// Subscriber
const sub = new Redis();
sub.subscribe("order.created");
sub.on("message", (channel, msg) => { ... });
```

Wire notifications to subscribe to `order.created` and send a confirmation.

**Expected output:**
End-to-end event fires a notification.

### Task 4: Comparison table

**What to do:**
In `communication-patterns.md`, fill:

| Criterion | Sync HTTP | Async Queue | Pub/Sub |
|-----------|-----------|-------------|---------|
| Caller waits | | | |
| Survives consumer down | | | |
| At-least-once delivery | | | |
| Response returned to caller | | | |
| Good for: | | | |

Your own words in each cell.

**Expected output:**
Table committed.

### Task 5: Pattern audit

**What to do:**
In `pattern-audit.md`, list every cross-service interaction in your project and declare the pattern used and WHY.

Example:
- Order created -> notification: pub/sub (fire and forget, user already got 200)
- Payment initiation -> M-Pesa: sync HTTP (user needs confirmation)
- Daily report -> email: queue (retry, priority)

**Expected output:**
Audit committed.

## Stretch Goals (Optional - Extra Credit)

- Try a real event bus (NATS, RabbitMQ) and compare.
- Add a trace ID that propagates through all three patterns.
- Implement a simple saga across two services.

## Submission Requirements

- **What to submit:** Repo, demo, notes, audit, `AI_AUDIT.md`.
- **Deadline:** End of Day 3

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Sync HTTP demo | 20 | Works, failure observed. |
| Async queue demo | 20 | Works, consumer-down survives. |
| Pub/sub demo | 25 | End-to-end event. |
| Comparison table | 20 | All cells filled. |
| Pattern audit | 10 | Real interactions classified. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Using sync HTTP for fire-and-forget work.** Couples the caller to the consumer's uptime.
- **Pub/sub for work that must not be lost.** Redis pub/sub does not persist. Use a queue.
- **Mixing patterns within one flow without thinking.** Each choice has consequences.

## Resources

- Day 3 reading: [Inter Service Communication.md](./Inter%20Service%20Communication.md)
- Week 24 AI boundaries: [../ai.md](../ai.md)
