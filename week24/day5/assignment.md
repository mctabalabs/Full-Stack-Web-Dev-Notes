# Week 24 - Day 5 Assignment

## Title
Local Orchestration And Git Blame

## Overview
Day 5 wires both services together with docker-compose for local dev, and teaches `git blame` / `git log -p` for answering "why is this line here?".

## Learning Objectives Assessed
- Orchestrate two services plus Redis plus Postgres with docker-compose
- Share env vars without committing secrets
- Use `git blame` and `git log -p` to trace code history

## Prerequisites
- Days 1-4 completed

## AI Usage Rules

**Ratio this week:** 45% manual / 55% AI
**Habit:** Architecture dip. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Compose file boilerplate.
- **NOT ALLOWED FOR:** Deciding what shares a network, what does not.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: docker-compose.yml

**What to do:**
Create a root `docker-compose.yml`:

```yaml
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: dev
      POSTGRES_DB: mctaba
    ports: ["5432:5432"]
  redis:
    image: redis:7
    ports: ["6379:6379"]
  api:
    build: ./services/api
    environment:
      DATABASE_URL: postgres://postgres:dev@postgres:5432/mctaba
      REDIS_URL: redis://redis:6379
    depends_on: [postgres, redis]
    ports: ["3000:3000"]
  notifications:
    build: ./services/notifications
    environment:
      REDIS_URL: redis://redis:6379
    depends_on: [redis]
```

**Expected output:**
`docker compose up` starts all four.

### Task 2: Service-to-service reach

**What to do:**
Queue a notification from the API and watch the notifications service process it. Screenshot the logs side by side.

**Expected output:**
`day5-compose.png` shows the end-to-end flow.

### Task 3: Secrets via .env

**What to do:**
Keep secrets in a root `.env` (gitignored) and reference with `${VAR}` in compose. Commit `.env.example` with placeholders.

**Expected output:**
No secrets in the repo.

### Task 4: git blame exercise

**What to do:**
Pick a file in your codebase. Run:

```bash
git blame path/to/file.js
git log -p path/to/file.js
git log -S "some string" path/to/file.js
```

In `blame-notes.md`, explain when to reach for each command:
- `git blame` -- last author per line
- `git log -p` -- full history with diffs
- `git log -S` -- "pickaxe" finds commits that added/removed the string

Your own words.

**Expected output:**
`blame-notes.md` committed.

### Task 5: Week 24 wrap

**What to do:**
Update `README.md` with a Week 24 section and a docker-compose quickstart. Push everything.

**Expected output:**
Pushed.

## Stretch Goals (Optional - Extra Credit)

- Add a `healthcheck` to each service in compose.
- Try `docker compose watch` for hot reload.
- Explore `git log --follow` across a rename.

## Submission Requirements

- **What to submit:** Repo, compose file, screenshot, notes, `AI_AUDIT.md`.
- **Deadline:** End of Day 5

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| docker-compose.yml | 30 | All four services up. |
| End-to-end flow | 20 | API -> queue -> worker. |
| Secrets handled | 15 | `.env` gitignored, example committed. |
| Blame notes | 25 | Three commands explained. |
| Clean commits | 10 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Using localhost to reach another service.** Inside compose, use the service name.
- **Committing `.env`.** Always `.env.example` + gitignore.
- **Blaming a line without reading the commit message first.** Context matters.

## Resources

- Day 5 reading: [Week 24 Recap.md](./Week%2024%20Recap.md)
- Week 24 AI boundaries: [../ai.md](../ai.md)
- git blame: https://git-scm.com/docs/git-blame
