# Week 12 - Day 1 Assignment

## Title
Install PostgreSQL and Migrate The CRM Schema

## Overview
Week 12 replaces SQLite with real PostgreSQL. Today you install Postgres, create a database for the CRM, write the schema with proper types (UUID, TIMESTAMPTZ, CHECK constraints), and load it. SQL is mostly a repeated pattern you have touched before -- but the types and constraints are new and important.

## Learning Objectives Assessed
- Install Postgres locally (or run it via Docker)
- Create a database and a restricted user for the app
- Write a Postgres schema with UUID, TIMESTAMPTZ, and CHECK constraints
- Use `psql` to verify tables and run queries

## Prerequisites
- Week 11 completed (monorepo layout, SQLite-backed CRM)

## AI Usage Rules

**Ratio this week:** 45% manual / 55% AI
**Habit:** Never trust AI with auth. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Generating migration SQL after you wrote the first table by hand. Postgres type reference lookups.
- **NOT ALLOWED FOR:** Designing the schema for you. Writing password hashing or JWT code (those come Day 3).
- **AUDIT REQUIRED:** Yes. Auth code audit starts this week and is hand-written only.

## Tasks

### Task 1: Install Postgres

**What to do:**
Install Postgres 15 or 16 for your OS:
- macOS: `brew install postgresql@16 && brew services start postgresql@16`
- Ubuntu: `sudo apt install postgresql postgresql-contrib`
- Windows: https://www.postgresql.org/download/windows/
- Or: `docker run -d --name crm-pg -e POSTGRES_PASSWORD=dev -p 5432:5432 postgres:16`

Verify: `psql --version` prints a version.

**Expected output:**
Postgres running on port 5432.

### Task 2: Create the database and app user

**What to do:**
In `psql` as a superuser:

```sql
CREATE USER crm_app WITH PASSWORD 'crm_dev_password';
CREATE DATABASE crm_dev OWNER crm_app;
GRANT ALL PRIVILEGES ON DATABASE crm_dev TO crm_app;
```

Connect as the app user: `psql -U crm_app -d crm_dev -h localhost`.

**Expected output:**
You can connect to `crm_dev` as `crm_app`. Screenshot `day1-psql.png`.

### Task 3: Write the schema by hand

**What to do:**
Create `packages/backend/db/schema.sql`:

```sql
CREATE TABLE leads (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  wa_phone TEXT NOT NULL UNIQUE,
  name TEXT,
  email TEXT,
  inquiry_type TEXT,
  status TEXT NOT NULL DEFAULT 'new'
    CHECK (status IN ('new', 'contacted', 'qualified', 'closed')),
  notes TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE conversations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  lead_id UUID NOT NULL REFERENCES leads(id) ON DELETE CASCADE,
  state TEXT NOT NULL,
  data JSONB,
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  lead_id UUID NOT NULL REFERENCES leads(id) ON DELETE CASCADE,
  direction TEXT NOT NULL CHECK (direction IN ('in', 'out')),
  body TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_leads_status ON leads(status);
CREATE INDEX idx_messages_lead_id ON messages(lead_id);
```

Load it: `psql -U crm_app -d crm_dev -f packages/backend/db/schema.sql`

**Expected output:**
Three tables and two indexes exist. `\dt` in psql shows them.

### Task 4: Migrate Week 11 data (optional but recommended)

**What to do:**
Write a small Node script `scripts/migrate-from-sqlite.js` that reads rows from your SQLite file and inserts them into Postgres. Handle the ID type change (SQLite TEXT IDs to UUID). If your SQLite IDs are already UUIDs, keep them; otherwise generate new ones.

**Expected output:**
Postgres has your old leads, conversations, and messages.

### Task 5: Schema notes

**What to do:**
In `schema-notes.md`, answer in your own words:
- Why `UUID` instead of auto-incrementing integers?
- Why `TIMESTAMPTZ` instead of `TIMESTAMP`?
- What does the `CHECK (status IN (...))` constraint buy you over just a comment?
- Why `ON DELETE CASCADE` on `conversations.lead_id` and `messages.lead_id`?

No AI paraphrasing.

**Expected output:**
`schema-notes.md` committed.

## Stretch Goals (Optional - Extra Credit)

- Add a `updated_at` trigger that auto-updates `updated_at` on every row update.
- Add a partial index: `CREATE INDEX ... WHERE status != 'closed'` for open leads only.
- Use `node-pg-migrate` or a migration tool instead of raw SQL files.

## Submission Requirements

- **What to submit:** Repo with `db/schema.sql`, `schema-notes.md`, screenshots, `AI_AUDIT.md`.
- **Deadline:** End of Day 1

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Postgres installed and running | 10 | Screenshot confirms. |
| crm_app user and crm_dev database | 15 | App user can connect with restricted privileges. |
| Schema with correct types | 35 | UUID, TIMESTAMPTZ, CHECK, REFERENCES all present. Loads without errors. |
| Indexes | 10 | Two indexes present. |
| Schema notes | 20 | Four questions answered in student's own words. |
| Clean commits | 10 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Using the `postgres` superuser for your app.** The app should never have DROP DATABASE privileges. Create a restricted user.
- **Storing timestamps as TEXT.** `TIMESTAMPTZ` is the correct type. It handles time zones automatically.
- **Forgetting `gen_random_uuid()`.** Postgres 13+ has it built in. Earlier versions need the `uuid-ossp` extension.

## Resources

- Day 1 reading: [PostgreSQL Fundamentals.md](./PostgreSQL%20Fundamentals.md)
- Week 12 AI boundaries: [../ai.md](../ai.md)
- Postgres data types: https://www.postgresql.org/docs/current/datatype.html
