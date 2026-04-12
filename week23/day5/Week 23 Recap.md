# Week 23, Day 5: Week Recap

Queues are the quietest upgrade in the Marathon and one of the most impactful. You cannot point at a new feature, but every existing feature is now reliable under conditions that would have broken it a week ago.

---

## What You Built

1. Three BullMQ queues: whatsapp, telegram, sms (plus chama + payments).
2. Workers for each with concurrency and rate limits.
3. Retries with exponential backoff.
4. Priorities for urgent vs marketing jobs.
5. Delayed jobs and repeat schedules.
6. Bull Board admin dashboard.
7. Dead letter queue with admin alerts.
8. Graceful SIGTERM shutdown.
9. Stalled job detection.
10. Migration of outbox, cron, and chama announcements to the queues.

---

## Self-Review Questions

1. What three problems do queues solve?
2. What does `limiter: { max: 20, duration: 1000 }` do?
3. What does `attempts: 5, backoff: exponential` do when the first attempt fails?
4. How is a delayed job different from a scheduled cron?
5. What is a dead-letter queue and when does a job end up there?
6. What does `worker.close()` do that `process.exit()` does not?
7. Why use `jobId` for daily reports?
8. Why one queue per channel instead of one big notifications queue?
9. How do per-group rate limits work in Telegram when BullMQ only has a global limiter?
10. What happens to jobs that were in flight when a worker crashes?

Target: 8/10.

---

## Peer Coding Session

### Track A: Dead-letter retry admin UI
Build an admin page that lists dead-letter jobs and has a "Retry" button that moves them back to the main queue.

### Track B: Metrics over time
Use BullMQ events to write throughput numbers to a `queue_metrics` table every minute. Plot them.

### Track C: Split the chama worker
The chama announcements currently go through the general telegram worker. Split them into a dedicated chama queue with different rate limits (chamas are slower).

### Track D: Flows for complex chains
Write a FlowProducer chain: "process payment" -> (in parallel: "send customer whatsapp", "send owner whatsapp", "send sms receipt"). Verify each child runs only after the parent succeeds.

---

## Weekend Prep

Start on Project 6 this weekend: the Multi-Channel Notification Hub. We ship it in Week 26. This weekend is the groundwork: a generic `notify(user, message, channels, priority)` function that picks a channel, falls back to another if the first fails, and queues retries.

Details in `week23/weekend/project.md`.

Next week we split the server into microservices. The queues you built today make that split easy.
