# Week 12 - Day 2 Assignment

## Title
Connect Node to Postgres and Refactor to Clean Architecture

## Overview
Yesterday you created the schema. Today you wire your Express backend to Postgres using the `pg` driver and refactor the code into routes, services, and repositories. Clean architecture pays off starting Day 3 when auth arrives -- the layers make it easier to add middleware and keep logic separated.

## Learning Objectives Assessed
- Use `pg` with a connection pool
- Write parameterised queries (never string concatenation)
- Split code into routes, services, and repositories
- Handle errors at the boundaries only

## Prerequisites
- Day 1 completed

## AI Usage Rules

**Ratio:** 45/55. **Habit:** Never trust AI with auth. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Generating the routes layer after you defined the architecture.
- **NOT ALLOWED FOR:** Writing the pool configuration or any SQL that references user tables (those come Day 3).
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Install pg and wire a connection pool

**What to do:**
```bash
cd packages/backend
npm install pg
```

Create `db/pool.js`:

```javascript
const { Pool } = require("pg");

const pool = new Pool({
  host: process.env.PG_HOST || "localhost",
  port: parseInt(process.env.PG_PORT || "5432", 10),
  user: process.env.PG_USER || "crm_app",
  password: process.env.PG_PASSWORD,
  database: process.env.PG_DATABASE || "crm_dev",
  max: 10,
  idleTimeoutMillis: 30000,
});

module.exports = pool;
```

Add the env variables to `.env` and `.env.example`.

**Expected output:**
`pool.js` working. A small test script that runs `SELECT 1` succeeds.

### Task 2: Refactor into layers

**What to do:**
Create this structure in `packages/backend/`:

```
src/
  routes/
    leads.js         (HTTP handlers, parse req/res)
  services/
    leadsService.js  (business logic, no req/res)
  repositories/
    leadsRepo.js     (SQL only, no business logic)
```

Each layer only depends on the one below. Example repository:

```javascript
// src/repositories/leadsRepo.js
const pool = require("../../db/pool");

async function list({ limit, offset, q, status }) {
  const params = [];
  const conditions = [];

  if (q) {
    params.push(`%${q}%`);
    conditions.push(`(name ILIKE $${params.length} OR email ILIKE $${params.length})`);
  }
  if (status) {
    params.push(status);
    conditions.push(`status = $${params.length}`);
  }

  const where = conditions.length ? `WHERE ${conditions.join(" AND ")}` : "";
  params.push(limit, offset);

  const sql = `
    SELECT * FROM leads
    ${where}
    ORDER BY created_at DESC
    LIMIT $${params.length - 1} OFFSET $${params.length}
  `;

  const { rows } = await pool.query(sql, params);
  return rows;
}

module.exports = { list };
```

Service layer calls the repo. Route calls the service.

**Expected output:**
One feature (list leads) working through all three layers. Parameterised SQL throughout.

### Task 3: Move all Week 11 routes to the new layout

**What to do:**
Port the rest of your Week 11 routes (`GET /api/leads`, `GET /api/leads/:id`, `PATCH /api/leads/:id`, `GET /api/stats`) to the new layered structure. Each route stays thin; logic lives in services; SQL lives in repos.

**Expected output:**
All routes still work via curl. No logic in the route handlers beyond parsing and responding.

### Task 4: Error handling middleware

**What to do:**
Add a centralised error handler at the end of your middleware chain:

```javascript
app.use((err, req, res, next) => {
  console.error(err);
  if (err.statusCode) {
    return res.status(err.statusCode).json({ error: err.message });
  }
  res.status(500).json({ error: "Internal server error" });
});
```

Throw `createError(400, "Invalid email")` (or a simple custom Error) from services when validation fails. The middleware catches them.

**Expected output:**
Triggering a validation error returns a clean 400, not a 500 stack trace.

### Task 5: Layer discipline notes

**What to do:**
In `layers-notes.md`, write 5-7 sentences:
- What is the rule for "what can each layer know about"?
- Why should repositories never format HTTP responses?
- Why should routes never write SQL directly?

Your own words.

**Expected output:**
`layers-notes.md` committed.

## Stretch Goals (Optional - Extra Credit)

- Add a simple migration runner that applies SQL files in order.
- Add request logging middleware that tracks method, path, status, and duration.
- Write tests for one repository function using a test database.

## Submission Requirements

- **What to submit:** Repo with refactored backend, `layers-notes.md`, `AI_AUDIT.md`.
- **Deadline:** End of Day 2

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| pg pool configured correctly | 15 | Env vars, max connections, working connection test. |
| Three-layer refactor | 30 | Routes, services, repositories. Each layer respects its role. |
| All Week 11 routes ported | 20 | All still working. |
| Error middleware | 15 | Validation errors become 400s, not 500s. |
| Layer discipline notes | 15 | Three questions answered honestly. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **String concatenation in SQL.** Always parameterise: `pool.query("... WHERE id = $1", [id])`.
- **Writing business logic in repositories.** Repos know SQL. Services know rules. Routes know HTTP.
- **Catching errors too early.** Let services and repos throw; catch at the route or middleware level.

## Resources

- Day 2 reading: [Node Postgres and Clean Architecture.md](./Node%20Postgres%20and%20Clean%20Architecture.md)
- Week 12 AI boundaries: [../ai.md](../ai.md)
- node-postgres docs: https://node-postgres.com/
