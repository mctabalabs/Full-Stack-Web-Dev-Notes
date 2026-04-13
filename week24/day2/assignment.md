# Week 24 - Day 2 Assignment

## Title
Extract The Notification Service -- A Practical Split

## Overview
Today you extract the notification worker into its own repo/service with its own Dockerfile and package.json. You learn firsthand what "split" really costs.

## Learning Objectives Assessed
- Extract a service from a monolith
- Define a clean interface (the queue) as the boundary
- Keep the extracted service deployable on its own
- Feel the cost: shared types, shared config, shared secrets

## Prerequisites
- Day 1 completed
- Notification queue from Week 23

## AI Usage Rules

**Ratio this week:** 45% manual / 55% AI
**Habit:** Architecture dip. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Dockerfile and boilerplate.
- **NOT ALLOWED FOR:** The interface -- you design the job contract yourself.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Create the new service folder

**What to do:**
Inside your monorepo (or new standalone repo), create `services/notifications/`:

```
services/notifications/
  package.json
  src/
    worker.js
    senders/
      whatsapp.js
      email.js
      sms.js
  Dockerfile
  .env.example
```

Copy over only the code the worker needs. Remove any imports from the main API.

**Expected output:**
Service folder in place.

### Task 2: Dependency minimisation

**What to do:**
Start with an empty `package.json` and add only what the worker needs:

```json
{
  "name": "@mctaba/notifications",
  "private": true,
  "main": "src/worker.js",
  "dependencies": {
    "bullmq": "^5.0.0",
    "ioredis": "^5.0.0"
  }
}
```

If you find yourself adding `express`, stop and ask why.

**Expected output:**
Minimal package.json committed.

### Task 3: Job contract

**What to do:**
Define the shape of a notification job in `contracts/notification.md`:

```markdown
## Notification Job Contract v1

Job name: `whatsapp` | `email` | `sms`

Data:
- phone (string, E.164, required for whatsapp/sms)
- to (string, email, required for email)
- subject (string, required for email)
- body (string, required)
- locale (string, optional, default "en")
- traceId (string, optional)

Behavior:
- Retries: transient errors only
- DLQ: after max attempts
- Rate limit: 20/sec per worker
```

This is the contract between producer and consumer.

**Expected output:**
Contract committed.

### Task 4: Dockerfile

**What to do:**
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY src ./src
CMD ["node", "src/worker.js"]
```

Build and run it pointing at your local Redis. Queue a job from the main API and confirm the containerised worker processes it.

**Expected output:**
Worker in Docker processes a real job.

### Task 5: Split cost notes

**What to do:**
In `split-cost.md`, write 5-7 sentences:
- What did you LOSE by splitting? (Shared types, shared config, simpler deploy.)
- What did you GAIN? (Independent scaling, cleaner surface area.)
- Was it worth it for your project today? Be honest.

Your own words.

**Expected output:**
`split-cost.md` committed.

## Stretch Goals (Optional - Extra Credit)

- Publish the service as an internal Docker image in GitHub Container Registry.
- Add versioning to the contract (`schemaVersion: 1`).
- Set up a separate CI workflow for the notifications service only.

## Submission Requirements

- **What to submit:** Repo with services folder, contract, notes, `AI_AUDIT.md`.
- **Deadline:** End of Day 2

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Service folder | 15 | Isolated, no monolith imports. |
| Minimal dependencies | 15 | Only what worker needs. |
| Job contract | 25 | Fields + behavior. |
| Dockerfile + run | 25 | Real job processed. |
| Split cost notes | 15 | Honest reflection. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Sharing a `common/` folder via relative imports.** Now the split is fake.
- **Skipping the contract.** It becomes oral tradition and rots.
- **Deploying the new service with the same secrets as the old one.** Principle of least privilege.

## Resources

- Day 2 reading: [Extracting the Notification Service.md](./Extracting%20the%20Notification%20Service.md)
- Week 24 AI boundaries: [../ai.md](../ai.md)
