# Week 13, Day 3: Connecting USSD to the CRM

By the end of today, the USSD app will be talking to the Week 12 Postgres database and to Africa's Talking's SMS API. Customers will be able to dial the code, be recognised by their phone number, see their real open tickets, file a new support ticket that creates a real row in the CRM, and get an SMS confirmation. Dispatchers on the web dashboard will see USSD-created tickets appear next to WhatsApp-created ones. The two channels share one database and one customer view.

This is the payoff day for three weeks of work. You are unifying three channels -- WhatsApp (Week 11), web dashboard (Week 12), USSD (today) -- around a single source of truth. That is the actual thing SMEs pay for: "I do not care which channel a customer used, I want one inbox, one history, one place to work."

**Prior-week concepts you will use today:**
- The Week 12 layered architecture -- routes, controllers, services, repositories (Week 12, Day 2)
- The Postgres `crm` database with `leads`, `conversations`, `messages` (Week 12, Day 1)
- The USSD state machine and Redis sessions (Week 13, Day 2)
- Africa's Talking API key setup (Week 13, Day 1)

**Estimated time:** 4 hours

---

## The Plan

We are going to fold the USSD app into the Week 12 CRM `server/` project. The standalone `ussd-hello` folder from Days 1-2 was a scratch pad; today it merges in.

The merged project structure:

```
server/
  config/
    db.js        (Postgres)
    redis.js     (NEW)
    env.js       (+ AT vars)
  routes/
    auth.routes.js
    leads.routes.js
    webhook.routes.js   (WhatsApp)
    ussd.routes.js      (NEW)
  controllers/
    ussd.controller.js  (NEW)
  services/
    ussd/
      dispatcher.js
      states/
        welcome.js
        my_tickets.js
        new_ticket_category.js
        new_ticket_message.js
        new_ticket_confirm.js
      session.js
    sms.service.js      (NEW -- Africa's Talking SMS)
    tickets.service.js  (NEW -- or reuse leads.service)
  repositories/
    leads.repo.js
    tickets.repo.js     (NEW -- see below)
```

A design decision: we are going to treat "support ticket" as a specialisation of "lead". A lead from WhatsApp is someone asking about a property. A ticket from USSD is someone asking about a service issue. Under the hood they are the same shape: a phone, a conversation, a status. For today we add a `channel` column and a `category` column to `leads` and keep using one table.

### Schema addition

```sql
BEGIN;
ALTER TABLE leads ADD COLUMN channel TEXT NOT NULL DEFAULT 'whatsapp'
  CHECK (channel IN ('whatsapp', 'ussd', 'web'));
ALTER TABLE leads ADD COLUMN category TEXT;
CREATE INDEX idx_leads_channel ON leads(channel);
COMMIT;
```

Update `db/schema.sql` too so a fresh setup includes these columns.

The `channel` column is the one bit of de-denormalisation worth its weight: the dashboard filter "USSD tickets only" becomes one indexed WHERE clause instead of a join through conversations. `category` is free text we will constrain in the service.

---

## Wiring Redis Into The Main Server

Copy `config/redis.js` from the scratch pad into `server/config/redis.js`. Add to `config/env.js`:

```javascript
const required = [
  // ...existing...
  "REDIS_URL",
  "AT_USERNAME",
  "AT_API_KEY",
];

// in the exported object:
REDIS_URL: process.env.REDIS_URL,
AT_USERNAME: process.env.AT_USERNAME,
AT_API_KEY: process.env.AT_API_KEY,
AT_SMS_FROM: process.env.AT_SMS_FROM || "", // short code if you have one
```

And `.env` gains:

```env
REDIS_URL=redis://localhost:6379
AT_USERNAME=sandbox
AT_API_KEY=...
```

---

## The Tickets Repository

Create `server/repositories/tickets.repo.js`. It is a thin wrapper around the leads table that filters by `channel = 'ussd'`.

```javascript
// server/repositories/tickets.repo.js
const { query } = require("../config/db");

async function findOpenByPhone(waPhone) {
  const { rows } = await query(
    `SELECT id, category, status, notes, created_at
     FROM leads
     WHERE wa_phone = $1
       AND channel = 'ussd'
       AND status NOT IN ('converted', 'lost')
     ORDER BY created_at DESC
     LIMIT 5`,
    [waPhone]
  );
  return rows;
}

async function create({ phone, category, message }) {
  const { rows } = await query(
    `INSERT INTO leads (wa_phone, name, inquiry_type, category, notes, channel, status)
     VALUES ($1, $2, $3, $4, $5, 'ussd', 'new')
     RETURNING *`,
    [phone, `USSD caller ${phone}`, category, category, message]
  );
  return rows[0];
}

module.exports = { findOpenByPhone, create };
```

Two choices worth explaining.

**We use the same `leads` table** rather than creating a new `tickets` table. Adding a new table would mean the dashboard needs a new query, the stats page needs a UNION, and the admin needs to understand "leads vs tickets". One table with a `channel` column is honest: USSD and WhatsApp are different entry points to the same concept -- "a customer who needs something".

**`name` defaults to `USSD caller <phone>`.** On USSD we do not yet know the caller's name (they have not typed it and we have not looked them up in a customer table). The placeholder is ugly but visible, so the dashboard shows "USSD caller +254712..." until someone updates it. Tomorrow we add a proper name lookup; today, this is honest.

---

## The SMS Service

Create `server/services/sms.service.js`:

```javascript
// server/services/sms.service.js
const env = require("../config/env");

const AT_URL = "https://api.sandbox.africastalking.com/version1/messaging";

async function sendSMS(to, message) {
  const body = new URLSearchParams({
    username: env.AT_USERNAME,
    to,
    message,
    from: env.AT_SMS_FROM || "",
  }).toString();

  const res = await fetch(AT_URL, {
    method: "POST",
    headers: {
      apiKey: env.AT_API_KEY,
      "Content-Type": "application/x-www-form-urlencoded",
      Accept: "application/json",
    },
    body,
  });

  const data = await res.json();
  if (!res.ok) {
    console.error("SMS send failed:", data);
    throw new Error("SMS failed");
  }
  return data;
}

module.exports = { sendSMS };
```

Three things to notice.

**AT's SMS API is also form-urlencoded**, not JSON. The USSD handler receives form data from AT and sends form data back to AT. Consistency you will get used to.

**The `apiKey` header**, not `Authorization: Bearer`. AT uses a non-standard header for its SMS API. Blame telecom conventions.

**Errors throw.** On Day 2 yesterday we talked about USSD sessions needing fast responses -- so we are going to call `sendSMS` *after* we finish replying to AT, not before. The user should not wait for the SMS send in the middle of the session. More on this below.

---

## The Full USSD Flow

Create `server/routes/ussd.routes.js`:

```javascript
// server/routes/ussd.routes.js
const express = require("express");
const ussdController = require("../controllers/ussd.controller");

const router = express.Router();
router.post("/", express.urlencoded({ extended: false }), ussdController.handle);

module.exports = router;
```

And wire it in `index.js`:

```javascript
app.use("/ussd", require("./routes/ussd.routes"));
```

No `requireAuth` -- AT's servers call this endpoint, not browsers. We rely on the fact that the URL is public-but-obscure (and in Week 18 we will add IP allow-listing because then it will be routing real money).

Create `server/controllers/ussd.controller.js`:

```javascript
// server/controllers/ussd.controller.js
const dispatcher = require("../services/ussd/dispatcher");

async function handle(req, res) {
  const { sessionId, phoneNumber, text } = req.body;

  const response = await dispatcher.run({
    sessionId,
    phoneNumber,
    rawText: text || "",
  });

  res.set("Content-Type", "text/plain");
  res.send(response);
}

module.exports = { handle };
```

The controller is three lines of real logic. Everything happens in the dispatcher.

Create `server/services/ussd/dispatcher.js`:

```javascript
// server/services/ussd/dispatcher.js
const session = require("./session");
const states = require("./states");

async function run({ sessionId, phoneNumber, rawText }) {
  const parts = rawText.split("*");
  const latestInput = rawText === "" ? "" : parts[parts.length - 1];

  const current = await session.get(sessionId);
  const handlerName = current.state || "welcome";
  const handler = states[handlerName] || states.welcome;

  const result = await handler({
    input: latestInput,
    context: current.context,
    phoneNumber,
  });

  if (result.response.startsWith("END")) {
    await session.destroy(sessionId);
    if (result.postSessionTask) {
      // Fire and forget -- do not block the USSD reply.
      result.postSessionTask().catch((err) =>
        console.error("post-session task failed:", err)
      );
    }
  } else {
    await session.set(sessionId, {
      state: result.nextState,
      context: result.nextContext,
    });
  }

  return result.response;
}

module.exports = { run };
```

One addition compared to yesterday: handlers can now return a `postSessionTask`, a function that runs *after* the USSD response is sent. This is where SMS sends, database writes that are slow, and any other "fire and forget" side effects go. The user sees the `END` message immediately; the SMS fires half a second later.

Critically, the error from a `postSessionTask` does not bubble up to AT -- we `.catch` and log. If SMS fails, the USSD session still closes cleanly. This is the right trade for a feature-phone user: show them the confirmation screen, then retry the SMS in the background.

---

## State Handlers With Database Access

Create `server/services/ussd/session.js` (same as yesterday's but relocated):

```javascript
// server/services/ussd/session.js
const { getClient } = require("../../config/redis");

const SESSION_TTL = 300;
const key = (id) => `ussd:session:${id}`;

async function get(sessionId) {
  const client = await getClient();
  const raw = await client.get(key(sessionId));
  return raw ? JSON.parse(raw) : { state: "welcome", context: {} };
}

async function set(sessionId, data) {
  const client = await getClient();
  await client.set(key(sessionId), JSON.stringify(data), { EX: SESSION_TTL });
}

async function destroy(sessionId) {
  const client = await getClient();
  await client.del(key(sessionId));
}

module.exports = { get, set, destroy };
```

Create `server/services/ussd/states/index.js`:

```javascript
module.exports = {
  welcome: require("./welcome"),
  my_tickets: require("./my_tickets"),
  new_ticket_category: require("./new_ticket_category"),
  new_ticket_message: require("./new_ticket_message"),
  new_ticket_confirm: require("./new_ticket_confirm"),
};
```

### `welcome.js`

```javascript
// server/services/ussd/states/welcome.js
module.exports = async function welcome({ input, context }) {
  if (input === "") {
    return {
      response:
        "CON Jetlink Support\n1. My open tickets\n2. File a new ticket\n3. Call support",
      nextState: "welcome",
      nextContext: context,
    };
  }

  if (input === "1") {
    return {
      response: "CON Loading your tickets...", // this is a dummy; my_tickets handles the first hit
      nextState: "my_tickets",
      nextContext: context,
    };
  }

  if (input === "2") {
    return {
      response:
        "CON What is the issue about?\n1. Billing\n2. Rider complaint\n3. Lost item\n4. Other\n0. Back",
      nextState: "new_ticket_category",
      nextContext: context,
    };
  }

  if (input === "3") {
    return {
      response: "END Call +254712000000 or dial again. Asante.",
      nextState: "done",
      nextContext: {},
    };
  }

  return {
    response:
      "CON Invalid. Jetlink Support\n1. My open tickets\n2. File a new ticket\n3. Call support",
    nextState: "welcome",
    nextContext: context,
  };
};
```

### `my_tickets.js` (this one touches Postgres)

```javascript
// server/services/ussd/states/my_tickets.js
const ticketsRepo = require("../../../repositories/tickets.repo");

module.exports = async function myTickets({ phoneNumber }) {
  const tickets = await ticketsRepo.findOpenByPhone(phoneNumber);

  if (tickets.length === 0) {
    return {
      response: "END You have no open tickets. Dial again to file one.",
      nextState: "done",
      nextContext: {},
    };
  }

  const lines = tickets
    .map((t, i) => `${i + 1}. ${t.category || "Ticket"} - ${t.status}`)
    .slice(0, 4); // keep under 182 chars

  return {
    response: `END Your tickets:\n${lines.join("\n")}`,
    nextState: "done",
    nextContext: {},
  };
};
```

Notice this handler reads from Postgres. The query is simple and indexed (`idx_leads_channel` + the implicit index on `wa_phone UNIQUE`), so latency is single-digit milliseconds. Well within USSD's budget.

Notice also the 4-ticket cap. On USSD you cannot "scroll"; you have one screen. Cap at what fits and accept the trade-off. A more advanced app would implement paging (`5. More...`) but for today's MVP, top-4 is the right move.

### `new_ticket_category.js`

```javascript
// server/services/ussd/states/new_ticket_category.js
const CATEGORIES = {
  "1": "billing",
  "2": "rider_complaint",
  "3": "lost_item",
  "4": "other",
};

module.exports = async function newTicketCategory({ input, context }) {
  if (input === "0") {
    return {
      response:
        "CON Jetlink Support\n1. My open tickets\n2. File a new ticket\n3. Call support",
      nextState: "welcome",
      nextContext: {},
    };
  }

  const category = CATEGORIES[input];
  if (!category) {
    return {
      response:
        "CON Invalid. What is the issue about?\n1. Billing\n2. Rider complaint\n3. Lost item\n4. Other\n0. Back",
      nextState: "new_ticket_category",
      nextContext: context,
    };
  }

  return {
    response: "CON Type a short message (max 160 chars):",
    nextState: "new_ticket_message",
    nextContext: { ...context, category },
  };
};
```

### `new_ticket_message.js`

```javascript
// server/services/ussd/states/new_ticket_message.js
module.exports = async function newTicketMessage({ input, context }) {
  if (!input || input.length < 3) {
    return {
      response: "CON Message too short. Type a short message (3-160 chars):",
      nextState: "new_ticket_message",
      nextContext: context,
    };
  }

  const truncated = input.slice(0, 160);
  return {
    response: `CON Confirm? "${truncated.slice(0, 60)}..."\n1. Yes, submit\n2. Re-type`,
    nextState: "new_ticket_confirm",
    nextContext: { ...context, message: truncated },
  };
};
```

Two tricks. **`input.slice(0, 160)`** trims to SMS-safe length in case the user's phone sends more than expected. **`truncated.slice(0, 60) + '...'`** is just for the confirmation screen so the 182-character USSD budget is not blown.

### `new_ticket_confirm.js` -- writes the ticket

```javascript
// server/services/ussd/states/new_ticket_confirm.js
const ticketsRepo = require("../../../repositories/tickets.repo");
const smsService = require("../../sms.service");

module.exports = async function newTicketConfirm({ input, context, phoneNumber }) {
  if (input === "2") {
    return {
      response: "CON Type a short message (max 160 chars):",
      nextState: "new_ticket_message",
      nextContext: { ...context, message: undefined },
    };
  }

  if (input !== "1") {
    return {
      response: `CON Invalid. Confirm?\n1. Yes, submit\n2. Re-type`,
      nextState: "new_ticket_confirm",
      nextContext: context,
    };
  }

  // Write the ticket synchronously -- it is fast.
  const ticket = await ticketsRepo.create({
    phone: phoneNumber,
    category: context.category,
    message: context.message,
  });

  const shortId = ticket.id.slice(0, 8);

  return {
    response: `END Ticket #${shortId} filed. You will get an SMS shortly.`,
    nextState: "done",
    nextContext: {},
    postSessionTask: async () => {
      await smsService.sendSMS(
        phoneNumber,
        `Jetlink: ticket #${shortId} received. Category: ${context.category}. An agent will contact you.`
      );
    },
  };
};
```

Notice the clean separation: the **write** happens inside the USSD session because it is fast and the user expects the confirmation. The **SMS** happens in `postSessionTask` because it is slow and the user should not wait. This is the rhythm of a production USSD app.

---

## End-To-End Test

1. Start Postgres, Redis, the Express server (`npm run dev`), and ngrok.
2. Update the AT callback URL to `https://<ngrok>/ussd`.
3. In the AT simulator, dial your code. See the welcome menu.
4. Pick `2` (file a new ticket), pick `1` (billing), type "I was charged twice for my ride yesterday", pick `1` (yes submit). See the confirmation.
5. Within a few seconds, the simulator's SMS log (or your real phone, if you're dialling from one) shows the SMS.
6. Open the CRM dashboard (Week 12). Filter by channel. The USSD ticket appears with the message body, category, channel = `ussd`, status = `new`.
7. Dial again, pick `1` (my open tickets). See the ticket you just filed.

If step 7 fails, the most likely bug is that your `findOpenByPhone` query did not match -- check the exact format of `phoneNumber` from AT. On the sandbox it usually comes as `+254712...` which matches what WhatsApp sends. In production it can differ; normalise with a small helper if needed.

---

## What About Auth

USSD authenticates by phone number and trusts the carrier. That is the whole story. When AT's webhook hits you with `phoneNumber: "+254712..."`, the GSM network has already confirmed that number. No password, no JWT. This is the unique security property of USSD: the channel itself is the authentication.

The *limit* of that trust: anyone with access to the SIM can impersonate the owner. For low-stakes things (check a balance, file a ticket) that is fine. For high-stakes things (make a payment) you layer a PIN on top -- we will do that in Week 18 when we wire USSD payments.

---

## Checkpoint

1. A fresh `schema.sql` load includes the `channel` and `category` columns.
2. Dialling the code and filing a ticket creates a row in `leads` with `channel = 'ussd'`.
3. The dashboard shows USSD tickets alongside WhatsApp leads.
4. Dialling again lists the user's open tickets, filtered by their phone number.
5. The SMS arrives to the dialler's phone (or the simulator log) within a few seconds of the `END` response.
6. If you stop Redis mid-session and redial, you land on the welcome menu -- no crash, just a clean reset. (The `session.get` fallback handles this.)
7. If you stop Postgres and try to file a ticket, the session ends with an error message (you should add a top-level try/catch in the dispatcher that returns `END Temporary error, please try again.` on failure -- do this now).

Commit:

```bash
git add .
git commit -m "feat: ussd connected to crm with sms confirmations"
```

---

## What You Learned

You now have a single CRM that ingests leads from three channels (WhatsApp, web, USSD) and presents them in one dashboard. That is real unification -- not marketing unification, but "the database has one table and every channel writes to it" unification. When a dispatcher opens the dashboard on Monday, they do not care who came from where.

Tomorrow we polish: timeouts, error recovery, accessibility for feature phones, Swahili strings, and a few deeper menus. Day 5 is the recap, and the weekend project puts it all together with a three-level support flow for a real local business story.
