# Week 12, Day 4: Multi-User CRM and a MongoDB Aside

By the end of today, your CRM will be a real multi-tenant tool: when Wanjiru logs in she sees the leads she owns, when Brian logs in he sees his, and when the admin logs in she sees everyone's. Agents will be able to claim unassigned leads. Admins will be able to reassign any lead to any agent. The dashboard will grow a tiny user picker and an "assigned to me" filter. In the last 45 minutes we put down the keyboard and read a MongoDB schema side by side with our Postgres one, so you know what Mongo is, when people reach for it, and why we chose otherwise.

**Prior-week concepts you will use today:**
- `users` table, `requireAuth`, `requireRole` middleware (Week 12, Day 3)
- `leads` table, status transitions, repositories and services (Week 12, Days 1-2)
- React state, forms, and the authenticated `api` helper (Week 12, Day 3)
- Transactions with `BEGIN/COMMIT` (Week 12, Day 1)

**Estimated time:** 3 hours for the code, 45 minutes for the Mongo reading.

---

## The Plan

Two product changes drive the work:

1. A lead can be **owned** by exactly zero or one user (the `assigned_to` column exists from yesterday -- today it becomes a real foreign key).
2. Permissions split two ways:
   - **Agents** see and edit only leads assigned to them, plus unassigned leads they can claim.
   - **Admins** see and edit every lead, and can reassign any lead to any agent.

This is the first time in the Marathon you have had to write *authorization* (what a logged-in user is allowed to do) on top of *authentication* (is this user logged in). They are different jobs. Authentication lives in `requireAuth`; authorization lives in the service layer, because it depends on business rules that the middleware cannot know.

---

## Step 1: Schema Change

`assigned_to` already exists as a nullable `TEXT` column on `leads`. We will promote it to a real foreign key against `users(id)`.

In `psql`:

```sql
BEGIN;

-- Drop the old loose column
ALTER TABLE leads DROP COLUMN assigned_to;

-- Add it back as a proper UUID foreign key
ALTER TABLE leads ADD COLUMN assigned_to UUID
  REFERENCES users(id) ON DELETE SET NULL;

CREATE INDEX idx_leads_assigned_to ON leads(assigned_to);

COMMIT;
```

Notice the two patterns.

**`ON DELETE SET NULL` instead of `CASCADE`.** If a user is deleted, we do not want their leads to evaporate -- we want the leads to go back to the unassigned pool so an admin can reassign them. `SET NULL` is the right policy for "this thing has an owner but the owner is optional". `CASCADE` is the right policy for "this thing cannot exist without its parent" (like messages cannot exist without a lead).

**Everything in one transaction.** If the second `ALTER TABLE` fails (say, because there are existing non-UUID values in the old column), the first one is rolled back and you are back where you started. You never want a half-migrated schema.

Update `server/db/schema.sql` to reflect the new state. Future-you setting up a new dev environment will load schema.sql from scratch and needs the final form.

---

## Step 2: Repository Changes

Open `server/repositories/leads.repo.js`. The `list` function needs a new filter: "show me the leads this user can see". Add an optional `userId` parameter:

```javascript
async function list({ status, search, assignedTo, limit = 50, offset = 0 }) {
  const conditions = [];
  const params = [];

  if (status) {
    params.push(status);
    conditions.push(`l.status = $${params.length}`);
  }
  if (search) {
    params.push(`%${search}%`);
    conditions.push(`(l.name ILIKE $${params.length} OR l.wa_phone ILIKE $${params.length})`);
  }
  if (assignedTo === "unassigned") {
    conditions.push("l.assigned_to IS NULL");
  } else if (assignedTo === "me" /* handled by caller -> user id */) {
    // caller passes a uuid instead
  } else if (assignedTo) {
    params.push(assignedTo);
    conditions.push(`l.assigned_to = $${params.length}`);
  }

  const where = conditions.length ? `WHERE ${conditions.join(" AND ")}` : "";

  params.push(limit);
  params.push(offset);

  const { rows } = await query(
    `SELECT l.*, u.name AS assigned_to_name
     FROM leads l
     LEFT JOIN users u ON u.id = l.assigned_to
     ${where}
     ORDER BY l.created_at DESC
     LIMIT $${params.length - 1} OFFSET $${params.length}`,
    params
  );
  return rows;
}
```

Two things to point at.

**We joined on `users`** so every lead row comes back with `assigned_to_name`. The dashboard will show "Claimed by Wanjiru" or "Unassigned" next to each row without a second round trip.

**`assignedTo` can be `"unassigned"`, a UUID, or absent.** Three cases. The special string `"unassigned"` maps to `IS NULL`. A UUID maps to `= $N`. Absent means "no filter". This kind of little-protocol between frontend and backend is common -- document it in a comment at the top of the file so nobody has to guess.

Add an `assign` function:

```javascript
async function assign(leadId, userId) {
  const { rows } = await query(
    `UPDATE leads SET assigned_to = $1, updated_at = NOW()
     WHERE id = $2
     RETURNING *`,
    [userId, leadId]
  );
  return rows[0] || null;
}
```

`userId` can be `null` to unassign -- `pg` will pass the JavaScript `null` through as SQL `NULL`.

---

## Step 3: Authorization in the Service

This is the heart of the day. Open `server/services/leads.service.js`.

```javascript
const leadsRepo = require("../repositories/leads.repo");
const AppError = require("../utils/AppError");

// ... existing VALID_STATUSES and VALID_TRANSITIONS ...

async function listForUser(user, filters) {
  if (user.role === "admin") {
    return leadsRepo.list(filters);
  }
  // Agents see their own + (if explicitly asked) unassigned
  if (filters.assignedTo === "unassigned") {
    return leadsRepo.list({ ...filters, assignedTo: "unassigned" });
  }
  return leadsRepo.list({ ...filters, assignedTo: user.id });
}

async function getLeadForUser(user, id) {
  const lead = await leadsRepo.findById(id);
  if (!lead) throw new AppError("Lead not found", 404);

  if (user.role !== "admin" && lead.assigned_to && lead.assigned_to !== user.id) {
    throw new AppError("Lead not found", 404);
  }
  return lead;
}

async function claimLead(user, id) {
  const lead = await leadsRepo.findById(id);
  if (!lead) throw new AppError("Lead not found", 404);
  if (lead.assigned_to) {
    throw new AppError("Lead is already assigned", 409);
  }
  return leadsRepo.assign(id, user.id);
}

async function reassignLead(user, id, newOwnerId) {
  if (user.role !== "admin") {
    throw new AppError("Only admins can reassign leads", 403);
  }
  const lead = await leadsRepo.findById(id);
  if (!lead) throw new AppError("Lead not found", 404);
  return leadsRepo.assign(id, newOwnerId); // null means unassign
}

async function changeStatus(user, id, nextStatus) {
  const lead = await getLeadForUser(user, id); // enforces visibility
  if (!VALID_STATUSES.includes(nextStatus)) {
    throw new AppError(`Invalid status: ${nextStatus}`, 400);
  }
  const allowed = VALID_TRANSITIONS[lead.status];
  if (!allowed.includes(nextStatus)) {
    throw new AppError(`Cannot move from ${lead.status} to ${nextStatus}`, 409);
  }
  return leadsRepo.updateStatus(id, nextStatus);
}

module.exports = {
  listForUser,
  getLeadForUser,
  claimLead,
  reassignLead,
  changeStatus,
  getStats: async (user) => {
    // Stats are admin-only for now. Agents see their own numbers tomorrow.
    if (user.role !== "admin") {
      throw new AppError("Admins only", 403);
    }
    const rows = await leadsRepo.statsByStatus();
    const total = rows.reduce((sum, r) => sum + r.total, 0);
    return { total, byStatus: rows };
  },
};
```

Two non-obvious patterns worth naming.

**When an agent asks for a lead they do not own, we return 404 -- not 403.** Returning 403 ("Forbidden") would tell the agent that the lead exists but they cannot see it. That is a small information leak. Returning 404 ("Not Found") means an outsider cannot even tell which lead ids are in the system. This trick is standard in production APIs; learn it early.

**Every service function takes `user` as its first argument.** The service layer cannot know who is logged in unless the caller (the controller) passes it. We could stash `user` in a request-scoped global, but passing it explicitly makes testing trivial and makes authorization decisions visible in the code. Every permission check in this file is local to one function -- no spooky action at a distance.

---

## Step 4: Controller and Route Wiring

Update `server/controllers/leads.controller.js` to pass `req.user` into every service call:

```javascript
const leadsService = require("../services/leads.service");

async function list(req, res) {
  const leads = await leadsService.listForUser(req.user, {
    status: req.query.status,
    search: req.query.search,
    assignedTo: req.query.assignedTo,
    limit: req.query.limit ? parseInt(req.query.limit, 10) : undefined,
    offset: req.query.offset ? parseInt(req.query.offset, 10) : undefined,
  });
  res.json({ leads });
}

async function getOne(req, res) {
  const lead = await leadsService.getLeadForUser(req.user, req.params.id);
  res.json({ lead });
}

async function patchStatus(req, res) {
  const lead = await leadsService.changeStatus(req.user, req.params.id, req.body.status);
  res.json({ lead });
}

async function claim(req, res) {
  const lead = await leadsService.claimLead(req.user, req.params.id);
  res.json({ lead });
}

async function reassign(req, res) {
  const lead = await leadsService.reassignLead(req.user, req.params.id, req.body.assignedTo);
  res.json({ lead });
}

async function stats(req, res) {
  const data = await leadsService.getStats(req.user);
  res.json(data);
}

module.exports = { list, getOne, patchStatus, claim, reassign, stats };
```

And the routes:

```javascript
// server/routes/leads.routes.js
const express = require("express");
const controller = require("../controllers/leads.controller");
const asyncHandler = require("../middleware/asyncHandler");
const requireRole = require("../middleware/requireRole");

const router = express.Router();

router.get("/", asyncHandler(controller.list));
router.get("/stats", requireRole("admin"), asyncHandler(controller.stats));
router.get("/:id", asyncHandler(controller.getOne));
router.patch("/:id/status", asyncHandler(controller.patchStatus));
router.post("/:id/claim", asyncHandler(controller.claim));
router.patch("/:id/assign", requireRole("admin"), asyncHandler(controller.reassign));

module.exports = router;
```

Two routes carry the `requireRole("admin")` guard. Every other route is role-agnostic on the outside, but the service inside enforces per-row visibility. This is the layered defence we planned for: the router rejects obvious stuff; the service rejects subtle stuff.

You also need a tiny users route so the admin reassign UI can list available agents:

```javascript
// server/routes/users.routes.js
const express = require("express");
const { query } = require("../config/db");
const requireRole = require("../middleware/requireRole");
const asyncHandler = require("../middleware/asyncHandler");

const router = express.Router();

router.get("/", requireRole("admin"), asyncHandler(async (req, res) => {
  const { rows } = await query(
    "SELECT id, name, email, role FROM users ORDER BY name"
  );
  res.json({ users: rows });
}));

module.exports = router;
```

Wire it in `index.js`:

```javascript
app.use("/api/users", requireAuth, require("./routes/users.routes"));
```

This is a small cheat -- I went directly to `query` instead of building a users service. That is acceptable *because* the logic is trivial (one SELECT, admin-only, no business rules). Know when to thin out the layers for small reads. Do not thin them out for writes or for anything with a rule in it.

---

## Step 5: The Bot Needs To Know About Ownership

One subtlety: when the WhatsApp bot creates a new lead from an inbound message, who owns it? There is no logged-in user -- it is the bot creating the row.

Three reasonable answers:

1. **Leave it unassigned.** Simple and honest. Agents can claim from the unassigned pool.
2. **Round-robin across agents.** A little code.
3. **Assign to the user configured as "default owner" in env.**

We go with (1) for today because it is the simplest and least surprising. In the weekend project you will add round-robin for Boda Dispatch. The bot service needs no changes -- `assigned_to` is nullable and defaults to `NULL`.

---

## Step 6: Dashboard Updates

Open the React dashboard. Three changes.

### A filter selector in the leads page

Above the leads table, add a select:

```jsx
<select
  value={filter}
  onChange={(e) => setFilter(e.target.value)}
  className="border p-1"
>
  <option value="mine">Assigned to me</option>
  <option value="unassigned">Unassigned</option>
  {currentUser.role === "admin" && <option value="all">All leads</option>}
</select>
```

In your `useEffect` that loads leads, translate the filter:

```javascript
const qs = filter === "mine"
  ? ""
  : filter === "unassigned"
  ? "?assignedTo=unassigned"
  : "?assignedTo=all"; // admins hit the endpoint with no filter
api(`/leads${qs}`).then((data) => setLeads(data.leads));
```

`currentUser` comes from `JSON.parse(localStorage.getItem("user"))`. A tiny `useCurrentUser` hook is worth writing so you do not sprinkle `JSON.parse` calls all over.

### A "Claim" button in the row detail panel

```jsx
{lead.assigned_to === null && (
  <button
    onClick={async () => {
      await api(`/leads/${lead.id}/claim`, { method: "POST" });
      refreshLeads();
    }}
    className="bg-black text-white px-3 py-1"
  >
    Claim lead
  </button>
)}
```

### An admin-only reassign dropdown

```jsx
{currentUser.role === "admin" && (
  <select
    value={lead.assigned_to || ""}
    onChange={async (e) => {
      await api(`/leads/${lead.id}/assign`, {
        method: "PATCH",
        body: JSON.stringify({ assignedTo: e.target.value || null }),
      });
      refreshLeads();
    }}
  >
    <option value="">Unassigned</option>
    {users.map((u) => (
      <option key={u.id} value={u.id}>{u.name}</option>
    ))}
  </select>
)}
```

Load `users` with a `GET /api/users` call inside a `useEffect` that runs only if `currentUser.role === "admin"`.

### Test it end-to-end

Create two users -- one admin (`role='admin'`, set it directly in `psql` since signup defaults to `agent`), one agent. Log in as the agent. Claim a lead. Log out. Log in as the admin. Verify the admin sees every lead. Reassign a lead. Log back in as the first agent and confirm the reassigned lead is gone from their list. Log in as a *second* agent and confirm the reassigned lead now appears. This is the full authorization story in action.

---

## A Note On What Just Happened

You just built multi-tenancy. Not the full row-level-security multi-tenancy of a SaaS product -- that is the capstone -- but the bones of it. You have a concept of "this user owns this row" that is enforced in the database (foreign key), in the service layer (authorization rules), and in the UI (what even renders). Three layers. That is how serious apps protect data.

This also quietly completed the last bullet from the Phase 2 roadmap: "multi-tenant logic (Company A cannot see Company B's data)". We did it a week early because it falls out naturally from the pattern we already built. In Week 27, when the capstone introduces real tenant companies, we will generalise "user owns lead" to "company owns everything" -- but the shape will be the same.

---

## Checkpoint

Before we look at Mongo, prove:

1. Agent A creates lead via bot -> appears in "Unassigned" -> clicks Claim -> moves to "Assigned to me".
2. Agent B cannot see that lead under any filter.
3. Admin sees the lead under "All leads", reassigns it to Agent B.
4. Agent B now sees the lead, Agent A no longer does.
5. `GET /api/leads/stats` returns 200 for admin, 403 for agent.
6. Admin deletes Agent B (`DELETE FROM users WHERE email='b@...'` in psql). The lead that was assigned to Agent B is now unassigned -- the `ON DELETE SET NULL` fired.
7. Every route you hit still runs through `requireAuth` -- try `curl http://localhost:5000/api/leads/stats` with no token, get 401.

---

## Intermission: A Short Tour of MongoDB

Put down the keyboard. What follows is reading, not code.

### What Mongo is

MongoDB is a **document database**. Instead of tables with rows and typed columns, it has collections of JSON-like documents, each of which can have any shape.

A Mongo "row" might look like this:

```json
{
  "_id": "ObjectId('65f...')",
  "waPhone": "+254712000001",
  "name": "Wanjiru Kamau",
  "email": "wanjiru@example.com",
  "inquiryType": "3-bedroom Kilimani",
  "status": "new",
  "notes": null,
  "conversation": {
    "state": "awaiting_email",
    "messages": [
      { "direction": "in", "body": "Hi, interested in Kilimani", "createdAt": "..." },
      { "direction": "out", "body": "Great! What's your email?", "createdAt": "..." }
    ]
  },
  "createdAt": "2026-04-12T08:15:00Z"
}
```

Notice what is different. The three tables we spent three days building in Postgres -- `leads`, `conversations`, `messages` -- are collapsed into one document. The messages live *inside* the lead. There is no foreign key because there is no separate collection to reference. You read the lead and you have everything.

That is the pitch: simple reads, natural JSON, no `JOIN` headaches.

### What it looks like in code

With Mongoose (the Node ODM):

```javascript
const { Schema, model } = require("mongoose");

const leadSchema = new Schema({
  waPhone: { type: String, required: true, unique: true },
  name: String,
  email: String,
  inquiryType: String,
  status: {
    type: String,
    enum: ["new", "contacted", "qualified", "converted", "lost"],
    default: "new",
  },
  conversation: {
    state: { type: String, default: "awaiting_name" },
    messages: [
      {
        direction: { type: String, enum: ["in", "out"] },
        body: String,
        createdAt: { type: Date, default: Date.now },
      },
    ],
  },
}, { timestamps: true });

const Lead = model("Lead", leadSchema);

// Reading the whole lead with its conversation and all messages:
const lead = await Lead.findOne({ waPhone: "+254712000001" });
// lead.conversation.messages is already populated.

// Adding a message:
lead.conversation.messages.push({ direction: "in", body: "..." });
await lead.save();
```

Compare to what you wrote in Postgres: three repositories, three separate SQL statements, a join for the detail view. Mongo collapses that.

### What the trade looks like in practice

| Dimension | PostgreSQL | MongoDB |
|---|---|---|
| Reads | JOIN when you want related data | Often one read; nested documents are already there |
| Writes | Each table writes separately; transactions tie them together | One document write for the whole nested tree |
| Relationships | Foreign keys, enforced | "References" (ids), enforced only if your code checks |
| Schema change | `ALTER TABLE` + migration | Just start writing new fields; old docs have missing fields |
| Query power | SQL -- joins, aggregates, window functions | Aggregation pipeline -- similar power, different shape |
| When two users share data | Foreign keys handle it cleanly | You denormalise and update in many places |
| When your data is logs/events | Overkill, schema feels heavy | Natural fit |

### When to pick Mongo

- **Event logs, analytics events, or anything append-only** where each document is independent and you rarely join.
- **Schemas that genuinely differ from row to row** -- a product catalog where one product has size/colour and another has voltage/wattage.
- **Rapid prototyping** when you have not decided what the shape of the data is yet.
- **Apps dominated by reading a single aggregate** -- a user profile with embedded settings, or a blog post with embedded comments.

### When to not pick Mongo

- Anything with money. Banks do not use document databases for balances for a reason -- transaction guarantees on multi-document updates are possible but harder.
- Anything where the same data is referenced from multiple places. An invoice and a customer -- you want the customer in one place and the invoice to reference them, not copies of the customer inside every invoice.
- When you want the database to enforce integrity for you.

### Why we picked Postgres for this course

Every product the Marathon builds from here on out has one of: money (payments), relationships (CRM, multi-tenant SaaS), or both. Those are the cases Postgres wins. The CRM you are building today is the cleanest example: a lead is owned by a user, sends many messages, each message links back -- you could do it in Mongo, but you would spend half your time manually keeping references in sync. Postgres makes the right thing easy.

Mongo is a good tool. If you join a team that uses it, you will be productive in a week because everything you learned here about data modelling -- unique keys, relationships, indexes, transactions -- still applies. The syntax changes; the thinking does not.

### If you want to try it on your own

Install Mongo locally with `docker run -d -p 27017:27017 --name mongo mongo:7`. Install Mongoose with `npm install mongoose`. Rewrite *just* the `leads.repo.js` from Week 12 with Mongoose instead of `pg`. You will see how much shorter it gets -- and also how many safety nets you lose. That is a good weekend exercise for anyone who finishes the main project early.

---

## End-of-Day Checklist

- Multi-user CRM works as described in steps 1-6.
- You can explain in one sentence the difference between authentication and authorization.
- You can name three cases where you would pick Postgres over Mongo and one where you would not.
- Everything is committed.

```bash
git commit -am "feat: multi-user ownership with admin and agent roles"
```

Tomorrow is Friday: recap the week, take stock of Phase 2 as a whole, and pair on the one part of the refactor you feel shakiest about before the weekend project.
