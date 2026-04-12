# Week 12, Day 2: Node + Postgres and Clean Architecture

By the end of today, your Week 11 WhatsApp CRM server will talk to the Postgres database you built yesterday instead of to SQLite, and the file layout of the project will look nothing like it did on Friday. The routes will be thin, the SQL will live in a `db/` folder, business rules will live in a `services/` folder, and a single error-handling middleware will catch everything your code throws. Then you will rerun the Week 11 webhook verification end-to-end and prove the bot still works -- just against a real database now.

There is no new product today. The CRM that was working on Friday will still work on Friday-plus-two days. The value is entirely in the refactor, and in the `pg` skills you pick up on the way. In interviews, in the capstone, in every week from here on out, "can you lay out a Node server so it does not rot after three features" is worth more than any single integration you will ship this year.

**Prior-week concepts you will use today:**
- Express app, router, middleware chain, JSON body parsing (Week 10, Day 1)
- The Week 11 CRM server: `index.js`, the webhook handler, the `/api/leads` REST routes, the bot state machine (Week 11, Days 1-3)
- `dotenv` and keeping secrets out of Git (Week 10, Day 1)
- Async/await, Promises, `try/catch` in route handlers (Week 4)
- SQL, your three-table Postgres schema (Week 12, Day 1)

**Estimated time:** 4-5 hours. This is the longest day of the week -- plan for it.

---

## What We Are Not Going To Do

Two temptations to name and refuse.

**We are not installing an ORM.** Prisma, TypeORM, Sequelize are all fine tools. They are also all tools that hide the SQL from you, and you have known SQL for exactly one day. If you start your Postgres life with Prisma you will end it unable to debug any query that does not fit Prisma's query builder. The rest of this course writes raw SQL against the `pg` driver. You will see ORMs again when you have earned them -- probably in the capstone or after.

**We are not going to rewrite every line of the Week 11 server.** Refactoring is like cleaning a kitchen: it is tempting to open every cupboard and end up with nothing done by dinner. Today we will do exactly one pass over the codebase, in one direction, with a clear definition of "done". Resist the urge to improve the React client, rename anything on the frontend, or redesign the webhook -- those can wait.

---

## The Target Architecture

By the end of today the `server/` folder will look like this:

```
server/
  index.js                # app setup only -- no business logic
  config/
    db.js                 # pg pool, exported as { query, getClient }
    env.js                # reads and validates .env
  db/
    schema.sql            # yesterday's schema, checked into git
    seed.sql              # dev seed data
  routes/
    leads.routes.js       # Express routers, no SQL, no rules
    webhook.routes.js
  controllers/
    leads.controller.js   # HTTP in, HTTP out, calls services
    webhook.controller.js
  services/
    leads.service.js      # business rules, no SQL, no HTTP
    bot.service.js        # state machine (was bot.js in week 11)
  repositories/
    leads.repo.js         # ALL SQL for leads lives here
    conversations.repo.js
    messages.repo.js
  middleware/
    errorHandler.js       # the one place errors turn into responses
    asyncHandler.js       # wraps async routes
  utils/
    AppError.js           # custom error class
```

Ten folders. Twenty-odd files. It looks like a lot. Each folder has one job, and if you know what job you are doing you will know exactly which folder to open. Before you protest that this is over-engineering for a CRM, remember that the Week 11 CRM already had 600 lines of code in a single `index.js` and you were already losing your place in it by Day 4. Structure is not bureaucracy; it is how you stop losing your place.

### The layer rules (memorise these)

1. **Routes** know about Express and URLs. Nothing else.
2. **Controllers** know about `req` and `res`. They read the request, call one service, and send a response. They never touch the database.
3. **Services** know about the business rules. "A lead cannot be marked `converted` unless it was first `qualified`". Services call repositories and pure functions; they never touch `req`/`res` or SQL directly.
4. **Repositories** know about SQL. They expose functions like `findById(id)` and `insert(lead)`. They return plain objects, never Express responses.
5. **Middleware** is the glue -- authentication, error handling, logging.

The test for whether a layer is doing the right job: if you deleted Express tomorrow and rebuilt the app as a CLI tool, the services and repositories should still work without changes. Only routes and controllers would be replaced.

---

## Step 1: Install `pg` and Wire the Pool

From inside `server/`:

```bash
npm install pg
npm uninstall better-sqlite3    # we will not need it anymore
```

Create `server/config/db.js`:

```javascript
// server/config/db.js
const { Pool } = require("pg");
const env = require("./env");

const pool = new Pool({
  host: env.DB_HOST,
  port: env.DB_PORT,
  user: env.DB_USER,
  password: env.DB_PASSWORD,
  database: env.DB_NAME,
  max: 10,                      // up to 10 concurrent connections
  idleTimeoutMillis: 30000,     // close idle connections after 30s
});

pool.on("error", (err) => {
  console.error("Unexpected pg pool error", err);
  process.exit(1);
});

async function query(text, params) {
  const start = Date.now();
  const result = await pool.query(text, params);
  const ms = Date.now() - start;
  if (process.env.DB_LOG === "true") {
    console.log(`[db] ${ms}ms ${text.split("\n")[0]}`);
  }
  return result;
}

async function getClient() {
  const client = await pool.connect();
  return client;
}

module.exports = { pool, query, getClient };
```

A few things to understand.

**A pool, not a single connection.** `Pool` is a group of up to ten reusable connections. When your webhook fires and your dashboard fetches leads at the same time, they each grab a free connection from the pool, run their query, and hand it back. Without a pool you would open and close a TCP connection for every single query -- slow, and Postgres will eventually refuse connections. Pools are the only way to run `pg` in production.

**`pool.query(text, params)` parameterises automatically.** Notice that the `query` helper takes `text` and `params` separately. You will never, *ever*, use string concatenation to build SQL in this codebase. We write:

```javascript
query("SELECT * FROM leads WHERE wa_phone = $1", [phone]);
```

never

```javascript
// DO NOT DO THIS, EVER
query("SELECT * FROM leads WHERE wa_phone = '" + phone + "'");
```

The second form is a SQL injection vulnerability. If `phone` is `'; DROP TABLE leads; --`, your database is gone. Parameterised queries pass the values through a separate channel Postgres cannot be tricked by. Every junior backend engineer who has leaked a customer database in the last twenty years did it by skipping parameters. Do not be that engineer.

**`$1`, `$2`, `$3` instead of `?`.** SQLite used `?` placeholders. Postgres uses numbered ones. Get used to this now. The `params` array is one-indexed into the SQL text: `$1` = `params[0]`, `$2` = `params[1]`.

**The `query` helper logs slow queries.** Set `DB_LOG=true` in `.env` when you want to see every query scrolling by. Turn it off for normal dev so your terminal is not a firehose. This one function will save you hours of debugging when a query mysteriously takes two seconds.

**`getClient()` returns a dedicated connection** for transactions. More on that on Day 4 when we need it.

### The env module

Create `server/config/env.js`:

```javascript
// server/config/env.js
require("dotenv").config();

const required = [
  "DB_HOST",
  "DB_PORT",
  "DB_USER",
  "DB_PASSWORD",
  "DB_NAME",
  "META_VERIFY_TOKEN",
];

for (const key of required) {
  if (!process.env[key]) {
    console.error(`Missing required env var: ${key}`);
    process.exit(1);
  }
}

module.exports = {
  PORT: parseInt(process.env.PORT || "5000", 10),
  DB_HOST: process.env.DB_HOST,
  DB_PORT: parseInt(process.env.DB_PORT, 10),
  DB_USER: process.env.DB_USER,
  DB_PASSWORD: process.env.DB_PASSWORD,
  DB_NAME: process.env.DB_NAME,
  META_VERIFY_TOKEN: process.env.META_VERIFY_TOKEN,
  META_ACCESS_TOKEN: process.env.META_ACCESS_TOKEN,
  META_PHONE_NUMBER_ID: process.env.META_PHONE_NUMBER_ID,
  META_APP_SECRET: process.env.META_APP_SECRET,
};
```

Two ideas here worth naming:

**Validate env at boot.** If `DB_PASSWORD` is missing, the app should refuse to start, not die later when the first query fires. Fail loudly and early.

**One place that reads `process.env`.** Nowhere else in the codebase will you see `process.env.SOMETHING`. The rest of the app imports `env` from this one file. If tomorrow you decide to load config from AWS Secrets Manager instead, you change exactly one file.

Update `server/.env` with the database block:

```env
PORT=5000
DB_HOST=localhost
DB_PORT=5432
DB_USER=crm_user
DB_PASSWORD=crm_dev_password
DB_NAME=crm
DB_LOG=false

META_VERIFY_TOKEN=mctaba_crm_verify_2026
META_ACCESS_TOKEN=
META_PHONE_NUMBER_ID=
META_APP_SECRET=
```

And `.env.example` gets the same keys with empty values. Commit both.

---

## Step 2: The Repository Layer

Create `server/repositories/leads.repo.js`:

```javascript
// server/repositories/leads.repo.js
const { query } = require("../config/db");

async function findById(id) {
  const { rows } = await query(
    "SELECT * FROM leads WHERE id = $1",
    [id]
  );
  return rows[0] || null;
}

async function findByPhone(waPhone) {
  const { rows } = await query(
    "SELECT * FROM leads WHERE wa_phone = $1",
    [waPhone]
  );
  return rows[0] || null;
}

async function list({ status, search, limit = 50, offset = 0 }) {
  const conditions = [];
  const params = [];

  if (status) {
    params.push(status);
    conditions.push(`status = $${params.length}`);
  }
  if (search) {
    params.push(`%${search}%`);
    conditions.push(`(name ILIKE $${params.length} OR wa_phone ILIKE $${params.length})`);
  }

  const where = conditions.length ? `WHERE ${conditions.join(" AND ")}` : "";

  params.push(limit);
  params.push(offset);

  const { rows } = await query(
    `SELECT * FROM leads ${where}
     ORDER BY created_at DESC
     LIMIT $${params.length - 1} OFFSET $${params.length}`,
    params
  );
  return rows;
}

async function insert({ waPhone, name, email, inquiryType }) {
  const { rows } = await query(
    `INSERT INTO leads (wa_phone, name, email, inquiry_type)
     VALUES ($1, $2, $3, $4)
     RETURNING *`,
    [waPhone, name, email, inquiryType]
  );
  return rows[0];
}

async function updateStatus(id, status) {
  const { rows } = await query(
    `UPDATE leads SET status = $1, updated_at = NOW()
     WHERE id = $2
     RETURNING *`,
    [status, id]
  );
  return rows[0] || null;
}

async function statsByStatus() {
  const { rows } = await query(
    `SELECT status, COUNT(*)::int AS total
     FROM leads
     GROUP BY status`
  );
  return rows;
}

module.exports = {
  findById,
  findByPhone,
  list,
  insert,
  updateStatus,
  statsByStatus,
};
```

Study the `list` function -- it is the one with real complexity. It builds a SQL query dynamically based on which filters the dashboard sent. The trick is building the `params` array as you go and using `params.length` to pick the right `$N` placeholder. Every other repo function is a boring one-liner, which is how you want them.

**`RETURNING *`** is a Postgres feature worth a note: after an `INSERT` or `UPDATE`, it returns the affected row(s) as if you had followed up with a `SELECT`. No round-trip, one query.

**`COUNT(*)::int`** casts the count to an integer. Postgres returns `COUNT(*)` as a `bigint`, which `pg` gives you as a string in Node (because JavaScript numbers cannot safely hold all 64-bit integers). Casting to `int` inside the query spares you from parsing it on the Node side.

Create `server/repositories/conversations.repo.js` and `server/repositories/messages.repo.js` with the same shape. I will leave the full file as an exercise so you prove you understand the pattern -- you need `findByLeadId`, `upsertState`, `appendMessage`, and `listMessagesForLead`. Use the Week 11 SQLite code as the source of truth for what each function should do; translate each SQL string to `$N` placeholders as you go.

---

## Step 3: The Service Layer

Services are where the thinking lives. Create `server/services/leads.service.js`:

```javascript
// server/services/leads.service.js
const leadsRepo = require("../repositories/leads.repo");
const AppError = require("../utils/AppError");

const VALID_STATUSES = ["new", "contacted", "qualified", "converted", "lost"];

const VALID_TRANSITIONS = {
  new: ["contacted", "lost"],
  contacted: ["qualified", "lost"],
  qualified: ["converted", "lost"],
  converted: [],
  lost: [],
};

async function getLead(id) {
  const lead = await leadsRepo.findById(id);
  if (!lead) throw new AppError("Lead not found", 404);
  return lead;
}

async function listLeads(filters) {
  return leadsRepo.list(filters);
}

async function changeStatus(id, nextStatus) {
  if (!VALID_STATUSES.includes(nextStatus)) {
    throw new AppError(`Invalid status: ${nextStatus}`, 400);
  }
  const lead = await getLead(id);
  const allowed = VALID_TRANSITIONS[lead.status];
  if (!allowed.includes(nextStatus)) {
    throw new AppError(
      `Cannot move from ${lead.status} to ${nextStatus}`,
      409
    );
  }
  return leadsRepo.updateStatus(id, nextStatus);
}

async function getStats() {
  const rows = await leadsRepo.statsByStatus();
  const total = rows.reduce((sum, r) => sum + r.total, 0);
  return { total, byStatus: rows };
}

module.exports = { getLead, listLeads, changeStatus, getStats };
```

Notice what is in here that was *not* in the Week 11 version: the `VALID_TRANSITIONS` map. Last week the dashboard let you mark any lead as `converted` from any state. This week we are saying "you cannot convert a lead you have not qualified" -- a business rule. And the business rule lives in the service, not in the controller, not in the database, not in the React form. If the same rule needs to apply to the WhatsApp bot, the bot imports the service. If it needs to apply to an admin script, the script imports the service. One rule, one place.

This is the payoff of the refactor. The Week 11 code could not have added this rule without duplicating it in three places.

---

## Step 4: Controllers and Routes

Create `server/controllers/leads.controller.js`:

```javascript
// server/controllers/leads.controller.js
const leadsService = require("../services/leads.service");

async function list(req, res) {
  const { status, search, limit, offset } = req.query;
  const leads = await leadsService.listLeads({
    status,
    search,
    limit: limit ? parseInt(limit, 10) : undefined,
    offset: offset ? parseInt(offset, 10) : undefined,
  });
  res.json({ leads });
}

async function getOne(req, res) {
  const lead = await leadsService.getLead(req.params.id);
  res.json({ lead });
}

async function patchStatus(req, res) {
  const lead = await leadsService.changeStatus(req.params.id, req.body.status);
  res.json({ lead });
}

async function stats(req, res) {
  const data = await leadsService.getStats();
  res.json(data);
}

module.exports = { list, getOne, patchStatus, stats };
```

Notice the controllers are small. They do almost nothing. That is the point -- when they do almost nothing, they are almost impossible to break. All the real work has been pushed one layer down. Notice also there is no `try/catch` -- we will let the error middleware handle that.

Create `server/routes/leads.routes.js`:

```javascript
// server/routes/leads.routes.js
const express = require("express");
const controller = require("../controllers/leads.controller");
const asyncHandler = require("../middleware/asyncHandler");

const router = express.Router();

router.get("/", asyncHandler(controller.list));
router.get("/stats", asyncHandler(controller.stats));
router.get("/:id", asyncHandler(controller.getOne));
router.patch("/:id/status", asyncHandler(controller.patchStatus));

module.exports = router;
```

---

## Step 5: Error Handling

Create `server/utils/AppError.js`:

```javascript
// server/utils/AppError.js
class AppError extends Error {
  constructor(message, statusCode = 500) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
  }
}

module.exports = AppError;
```

Create `server/middleware/asyncHandler.js`:

```javascript
// server/middleware/asyncHandler.js
module.exports = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};
```

That tiny function is why the controllers above have no `try/catch`. In Express, if an async handler throws, Express *does not* automatically pass the error to `next`. Wrapping with `asyncHandler` does it for you. Every async route goes through this wrapper. (Express 5 finally made this automatic, but we are on Express 4 for compatibility.)

Create `server/middleware/errorHandler.js`:

```javascript
// server/middleware/errorHandler.js
const AppError = require("../utils/AppError");

// eslint-disable-next-line no-unused-vars
module.exports = function errorHandler(err, req, res, next) {
  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      error: { message: err.message },
    });
  }

  // Postgres unique violation
  if (err.code === "23505") {
    return res.status(409).json({
      error: { message: "That record already exists." },
    });
  }

  // Postgres check constraint violation
  if (err.code === "23514") {
    return res.status(400).json({
      error: { message: "Invalid value for a constrained column." },
    });
  }

  console.error("Unhandled error:", err);
  res.status(500).json({ error: { message: "Internal server error" } });
};
```

One place. Every error your app throws -- validation, not-found, SQL constraint, unexpected crash -- turns into a JSON response here. Before today you had `try/catch` in every route, and each one formatted errors slightly differently. Now there is one format.

The two Postgres error codes are worth knowing by heart: `23505` (unique violation) and `23514` (check constraint violation). They will come up all the time. A full list is in the Postgres docs under "Error Codes".

---

## Step 6: The New `index.js`

Replace the old `server/index.js` with:

```javascript
// server/index.js
const express = require("express");
const cors = require("cors");
const env = require("./config/env");

const leadsRoutes = require("./routes/leads.routes");
const webhookRoutes = require("./routes/webhook.routes");
const errorHandler = require("./middleware/errorHandler");

const app = express();

app.use(cors({ origin: process.env.APP_URL || "http://localhost:3000" }));
app.use(express.json({ verify: (req, _res, buf) => { req.rawBody = buf; } }));

app.get("/health", (req, res) => res.json({ ok: true }));

app.use("/api/leads", leadsRoutes);
app.use("/webhook", webhookRoutes);

app.use(errorHandler);

app.listen(env.PORT, () => {
  console.log(`CRM server running on :${env.PORT}`);
});
```

That is the whole file. Thirty lines of pure wiring, no business logic, no SQL. When a new engineer joins the project, this is the file they open first and they can read the whole application from it: health check, two route groups, one error handler, one port.

> **The webhook routes/controller** should be refactored the same way -- move the Week 11 webhook handler into `controllers/webhook.controller.js`, the signature verification into a middleware, and the bot state machine into `services/bot.service.js`. Because the state machine now calls `conversationsRepo.upsertState(...)` instead of SQLite prepared statements, the migration is mostly search-and-replace. Give yourself an hour for this.

---

## Step 7: Run It

Load yesterday's schema into the running database from inside `server/`:

```bash
psql -U crm_user -d crm -h localhost -f db/schema.sql
```

Start the server:

```bash
npm run dev
```

You should see `CRM server running on :5000` and no other output. Test each layer:

```bash
curl http://localhost:5000/health
# {"ok":true}

curl http://localhost:5000/api/leads
# {"leads":[]}

curl -X POST http://localhost:5000/api/leads \
  -H "Content-Type: application/json" \
  -d '{"waPhone":"+254712345678","name":"Test","email":"t@t.com","inquiryType":"Demo"}'
# you will need to add a POST route + controller + service + repo.insert call
# this is the last hole in the refactor -- fix it now

curl http://localhost:5000/api/leads/stats
# {"total":1,"byStatus":[{"status":"new","total":1}]}
```

Then fire up ngrok, point the Meta webhook at it, and send a real WhatsApp message to your test number. You should see the bot reply and a new row appear in Postgres (`SELECT * FROM leads;` in `psql`). If the React dashboard from Week 11, Day 4 is still running on port 3000, refresh it -- the leads should appear. Nothing should have changed for the frontend.

That is the point. The refactor is invisible from outside the server. The value is all inside the walls.

---

## Checkpoint

Prove each of these before stopping:

1. `npm run dev` starts without errors and without a single `process.env.X` reference outside `config/env.js`.
2. `GET /api/leads` returns JSON and logs a `[db]` line if `DB_LOG=true`.
3. `PATCH /api/leads/:id/status` with body `{"status":"converted"}` against a lead that is currently `new` returns a **409** error -- proving the service layer rejected the invalid transition.
4. `PATCH /api/leads/:id/status` with a lead that is `qualified` succeeds and updates the row.
5. Sending a WhatsApp message to your bot still works -- lead and messages rows appear in Postgres.
6. No file in `controllers/` contains the word `SELECT`, `INSERT`, `UPDATE`, or `DELETE`. Grep it to prove it.
7. No file in `routes/` contains `await` except inside `asyncHandler` wrapping.
8. Stop the Postgres service (`sudo systemctl stop postgresql`), hit `GET /api/leads`, confirm you get a clean 500 with a JSON body, not a crashed server. Restart Postgres.

Commit your work as one atomic thing:

```bash
git add .
git commit -m "refactor: migrate CRM to Postgres with layered architecture"
```

---

## What You Learned

You took a 600-line `index.js` and turned it into a ten-folder application. You replaced a file-based database with a real relational one. You added one business rule (status transitions) that would have been impossible to maintain in the old layout. And every piece of the refactor is something you will do again, in every serious Node project, for the rest of your career.

Tomorrow we add users and JWT. The clean architecture pays off immediately -- we will add exactly one middleware and one service file, and suddenly every route is protected. Without today's refactor that change would have sprayed `jwt.verify(...)` calls across twenty route handlers.

Rest up. Day 3 is shorter but denser.
