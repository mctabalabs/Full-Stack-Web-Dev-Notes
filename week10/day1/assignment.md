# Week 10 - Day 1 Assignment

## Title
Payments Intro and the Express Backend Foundation

## Overview
Week 10 is the payments week. Today you register for an M-Pesa Daraja sandbox account, build a minimal Express server that will handle payment callbacks later in the week, and understand the payments request-response cycle before any money moves. No AI writes a single line of your backend.

## Learning Objectives Assessed
- Register for Safaricom Daraja sandbox and obtain consumer key/secret
- Scaffold an Express server by hand
- Load environment variables from `.env` and keep them out of Git
- Understand what a webhook/callback URL is
- Expose localhost via ngrok for the callback

## Prerequisites
- Week 9 completed
- A Kenyan phone number (optional but useful for sandbox tests)

## AI Usage Rules

**Ratio this week:** 50% manual / 50% AI
**Habit:** NEVER trust AI with money. Every line that touches an amount, a reference, or an idempotency key is hand-written. See [../ai.md](../ai.md).

- **ALLOWED FOR:** UI scaffolding for the React frontend later in the week. Asking AI to explain Daraja error codes after you hit them.
- **NOT ALLOWED FOR:** Writing your Express routes, amount validation, or callback handler. Writing your token-exchange code.
- **AUDIT REQUIRED:** Yes. Include a "Money code audit" section listing every file that touches money and who wrote each line.

## Tasks

### Task 1: Register for Safaricom Daraja sandbox

**What to do:**
1. Go to https://developer.safaricom.co.ke/
2. Sign up with a real email.
3. Create a new app in the sandbox. Subscribe it to "Lipa Na M-Pesa Online".
4. Note your Consumer Key and Consumer Secret. Do NOT commit them.

**Expected output:**
Sandbox app created. Consumer Key and Consumer Secret copied somewhere safe (password manager, or your `.env` file).

### Task 2: Scaffold the Express backend

**What to do:**
```bash
mkdir payments-backend
cd payments-backend
npm init -y
npm install express cors dotenv axios
npm install -D nodemon
```

Create `index.js`:

```javascript
require("dotenv").config();
const express = require("express");
const cors = require("cors");

const app = express();
app.use(cors());
app.use(express.json());

app.get("/health", (req, res) => {
  res.json({ status: "ok", timestamp: new Date().toISOString() });
});

const PORT = process.env.PORT || 3001;
app.listen(PORT, () => {
  console.log(`Server on :${PORT}`);
});
```

Add to `package.json`:
```json
"scripts": {
  "dev": "nodemon index.js"
}
```

Run `npm run dev` and visit `http://localhost:3001/health`. You should see the JSON response.

Type every line yourself. No AI.

**Expected output:**
GET /health returns `{ status: "ok", ... }`.

### Task 3: Environment variables and .gitignore

**What to do:**
Create `.env`:

```env
PORT=3001
MPESA_CONSUMER_KEY=your_key_here
MPESA_CONSUMER_SECRET=your_secret_here
MPESA_ENV=sandbox
```

Create `.gitignore`:

```
node_modules/
.env
```

Create `.env.example` (this IS committed):

```env
PORT=3001
MPESA_CONSUMER_KEY=
MPESA_CONSUMER_SECRET=
MPESA_ENV=sandbox
```

Verify your secrets are not in the repo: `git status` should NOT show `.env` as a tracked file.

**Expected output:**
`.env` ignored by Git. `.env.example` committed.

### Task 4: ngrok setup for callback URL

**What to do:**
Install ngrok: https://ngrok.com/download

Run:

```bash
ngrok http 3001
```

ngrok gives you a public HTTPS URL like `https://abc123.ngrok-free.app`. This is the URL Daraja will call when a payment completes. Save it -- you will use it tomorrow.

Visit `https://your-ngrok-url/health` from any browser. The Express health check should respond.

**Expected output:**
Screenshot `day1-ngrok.png` of the ngrok terminal and a successful `/health` call via the public URL.

### Task 5: Write the payments architecture in your own words

**What to do:**
In `day1-architecture.md`, draw (text or photograph) the flow:

```
Customer --> React frontend --> Express backend --> Daraja API --> Customer's phone
                                                 <--  callback  <-- Daraja
```

In 6-8 sentences, explain:
- Why Daraja calls your server and not the other way around
- Why you need ngrok (or a real server) for the callback
- What would happen if you lost the callback (hint: the customer would be charged but you would not know)

No AI. Your own words.

**Expected output:**
`day1-architecture.md` committed.

## Stretch Goals (Optional - Extra Credit)

- Add a second health route that tests Daraja reachability by making a simple API call (no auth yet).
- Add request logging with a simple `console.log(req.method, req.url)` middleware.
- Install a free tier of a proper tunnel service (Cloudflare Tunnel) as an alternative to ngrok.

## Submission Requirements

- **What to submit:** Repo with `payments-backend/`, `.env.example`, `day1-architecture.md`, screenshot, `AI_AUDIT.md`.
- **Deadline:** End of Day 1

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Daraja sandbox account created | 10 | Keys obtained. |
| Express backend scaffolded by hand | 20 | Every line typed yourself. /health endpoint working. |
| Environment variables and gitignore | 15 | `.env` ignored. `.env.example` committed. No secrets leaked. |
| ngrok public URL working | 15 | Public URL reaches localhost. Screenshot confirms. |
| Architecture explanation | 20 | Three questions answered in student's own words. |
| Money code audit | 15 | Every file listed with author. No AI-generated money code. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Committing `.env`.** Everyone does this once. If you ever see `.env` in `git status`, stop, add it to `.gitignore`, and use `git rm --cached .env`.
- **Sharing your Consumer Secret.** Never paste it in Discord, in a screenshot, or in an AI prompt. Regenerate if you accidentally expose.
- **Forgetting ngrok auth token.** ngrok free tier requires a token after first use. Sign up and `ngrok config add-authtoken ...`.

## Resources

- Day 1 reading: [Introduction to Payments.md](./Introduction%20to%20Payments.md)
- Day 1 reading: [The Backend Foundation.md](./The%20Backend%20Foundation.md)
- Week 10 AI boundaries: [../ai.md](../ai.md)
- Daraja API docs: https://developer.safaricom.co.ke/APIs
- ngrok quickstart: https://ngrok.com/docs/getting-started/
