# Week 10, Day 1: The Backend Foundation

By the end of today, you will have a working Express server connected to a SQLite database, with REST API routes that create and retrieve payment links. You will test every endpoint using Postman or Thunder Client and see real JSON responses coming back from a server YOU built.

This is a turning point. Until now, you have been a consumer of APIs. In Week 9, you called `fetch()` to get data from someone else's server. Today, you become the person who builds the server. You write the code that listens for requests and sends back responses.

**Prior-week concepts you will use today:**
- npm, package.json, installing packages (Week 6)
- Arrow functions, destructuring, template literals (Week 3)
- JSON (Week 5)
- .gitignore and version control (Week 6)
- HTTP request-response cycle (Weeks 1-2)
- Consuming APIs with fetch (Week 9)

**Estimated time:** 3-4 hours

---

## What You Are Building This Week

You are building a payment link system called Paylink. A business owner -- a freelance web designer in Westlands, a caterer in Lavington, a tutor in Kilimani -- visits your app, enters a client's name, phone number, and the amount owed, and gets back a unique URL. When the client opens that URL, they see a clean payment page. They enter their M-Pesa phone number, tap "Pay", and your server triggers an STK push on their phone. Once they confirm with their M-Pesa PIN, Safaricom notifies your server, and your server generates a branded PDF receipt.

That is the full loop: create link, share link, collect payment, generate receipt.

This is not a toy project. M-Pesa processes billions of shillings daily. The patterns you learn this week are how real fintech systems work in Kenya and across East Africa.

### The Three-Layer Architecture

Your app has three layers:

```
Frontend (React)                    -- What the user sees
    |
    | HTTP requests (fetch/axios)
    |
Backend (Node.js + Express)         -- Your server logic
    |
    | HTTP requests (axios)          -- Talks to Safaricom
    |
Safaricom Daraja API (M-Pesa)       -- Processes real payments
    |
    | Webhook (callback)             -- Safaricom talks BACK to you
    |
Backend                              -- Receives payment confirmation
```

The React frontend talks to your Express backend through REST API calls. Your backend talks to Safaricom's Daraja API to trigger payments. Safaricom talks back to your backend through a webhook callback when the payment completes. Your backend then generates a PDF receipt.

Today you build the middle layer: the Express backend with its database and API routes.

### Why Do We Need a Backend?

Think back to Week 9 when you first learned what a "backend" is. In Weeks 7-9, your React apps fetched data from APIs that someone else built. You called `fetch('https://some-api.com/data')` and got JSON back. Someone had to write the code that received your request, queried a database, and sent back that JSON. That someone is you now.

We need a backend for three reasons:

1. **Secrets.** Your M-Pesa API credentials (Consumer Key, Consumer Secret) cannot live in React code. Anyone can open browser DevTools and read your frontend JavaScript. The backend keeps secrets safe on the server.

2. **Database access.** Browsers cannot directly read or write to databases. The backend is the gatekeeper that receives requests, validates them, and stores data.

3. **Third-party API calls.** Safaricom's Daraja API expects server-to-server communication. It sends webhook callbacks to a URL on your server. A React app running in a browser cannot receive incoming HTTP requests.

---

## Project Setup

### Folder Structure

Your project has two separate applications living side by side: a `server` folder for the backend and a `client` folder for the React frontend. They are independent -- each has its own `package.json`, its own dependencies, and runs on its own port.

Open your terminal:

```bash
mkdir mpesa-paylink
cd mpesa-paylink
mkdir server
```

We will set up the React client on Day 3. Today is all about the server.

Your server folder will eventually look like this:

```
mpesa-paylink/
  server/
    package.json
    .env
    index.js
    db.js
    routes/
      links.js
      payments.js
      receipts.js
    services/
      mpesa.js
      receipt.js
    receipts/          -- generated PDFs land here
```

### Initializing the Backend

```bash
cd server
npm init -y
npm install express cors dotenv better-sqlite3 axios uuid pdfkit
npm install --save-dev nodemon
```

You have been using `npm install` since Week 6. The difference today is that you are choosing your own packages for the first time. Here is why each one exists:

- **express** -- Your web server framework. It listens for HTTP requests and lets you define what happens when someone hits a specific URL. Without this, you would need to write hundreds of lines of raw Node.js code to handle HTTP.
- **cors** -- Allows your React app (running on port 3000) to talk to your server (running on port 5000). Without this, the browser blocks the requests. More on this shortly.
- **dotenv** -- Loads secret values from a `.env` file into your code. This keeps your M-Pesa credentials out of your source code.
- **better-sqlite3** -- A fast, simple database driver. SQLite stores your entire database in a single file. No separate database server to install or configure.
- **axios** -- An HTTP client for making requests to Safaricom's API from your server. Think of it as the server-side version of `fetch()` from Week 5.
- **uuid** -- Generates unique IDs for payment links. Each link gets an ID like `a3f8b2c1-7d4e-4f9a-b5c6-1e2d3f4a5b6c` that is practically impossible to guess.
- **pdfkit** -- Creates PDF documents with code. We will use this on Day 2 to generate receipts.
- **nodemon** (dev dependency) -- Automatically restarts your server when you save changes. Without it, you would need to manually stop and restart the server every time you edit a file.

Now open `server/package.json` and add a dev script:

```json
{
  "scripts": {
    "dev": "nodemon index.js"
  }
}
```

When you run `npm run dev`, nodemon starts your server and watches for file changes.

### Environment Variables

Create `server/.env`:

```env
PORT=5000

# Safaricom Daraja Sandbox Credentials
MPESA_CONSUMER_KEY=your_consumer_key_here
MPESA_CONSUMER_SECRET=your_consumer_secret_here
MPESA_PASSKEY=your_passkey_here
MPESA_SHORTCODE=174379
MPESA_CALLBACK_URL=https://your-ngrok-url.ngrok.io/api/mpesa/callback

# App
APP_URL=http://localhost:3000
```

Environment variables are values that change between environments. Your M-Pesa credentials on your laptop are different from the ones on a production server. By putting them in `.env`, you keep one set of code that works everywhere -- you just swap the `.env` file.

The shortcode `174379` is Safaricom's sandbox test shortcode. You will get the passkey from your Daraja sandbox dashboard at developer.safaricom.co.ke.

**Critical:** Never commit `.env` to Git. Create a `.gitignore` file in your server folder:

```
node_modules/
.env
paylink.db
receipts/
```

Remember from Week 6 when you learned about `.gitignore`? This is exactly why it exists. If you push your M-Pesa credentials to GitHub, anyone can use them. The `node_modules` folder is excluded because it is huge and can be regenerated with `npm install`. The database file and receipts are excluded because they contain data specific to your machine.

---

## The Database

### Why SQLite?

For this project, SQLite is the right choice. It stores your entire database in a single file called `paylink.db`. There is nothing to install, no server to run, no connection strings to configure. You run your Express server, and the database just works.

Could you use PostgreSQL or MongoDB? Yes. But for a small business tool used by one person or a small team, SQLite handles thousands of transactions without breaking a sweat. You can always migrate to something bigger later if the app grows. Right now, simplicity wins.

### Creating the Database

Create `server/db.js`:

```javascript
const Database = require("better-sqlite3");
const path = require("path");

// Create (or open) a database file called paylink.db in the server folder
const db = new Database(path.join(__dirname, "paylink.db"));

// Enable WAL mode for better performance when reading and writing at the same time
db.pragma("journal_mode = WAL");

// Create our two tables if they don't already exist
db.exec(`
  CREATE TABLE IF NOT EXISTS links (
    id TEXT PRIMARY KEY,
    client_name TEXT NOT NULL,
    client_phone TEXT NOT NULL,
    amount REAL NOT NULL,
    description TEXT,
    status TEXT DEFAULT 'pending',
    created_at TEXT DEFAULT (datetime('now'))
  );

  CREATE TABLE IF NOT EXISTS payments (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    link_id TEXT NOT NULL,
    mpesa_receipt TEXT,
    transaction_date TEXT,
    phone_number TEXT,
    amount REAL,
    status TEXT DEFAULT 'pending',
    raw_callback TEXT,
    created_at TEXT DEFAULT (datetime('now')),
    FOREIGN KEY (link_id) REFERENCES links(id)
  );
`);

module.exports = db;
```

### Understanding the Schema

Think back to Week 3 when you learned about arrays of objects. A database table is exactly that: an array of objects where each row is an object and each column is a key. If you had a `links` table with two rows, it would look like this as JavaScript:

```javascript
const links = [
  {
    id: "a3f8b2c1-...",
    client_name: "John Kamau",
    client_phone: "254712345678",
    amount: 5000,
    description: "Website design deposit",
    status: "pending",
    created_at: "2026-03-29 10:30:00"
  },
  {
    id: "b4e9c3d2-...",
    client_name: "Aisha Wanjiku",
    client_phone: "254798765432",
    amount: 1500,
    description: "Logo design",
    status: "paid",
    created_at: "2026-03-29 11:15:00"
  }
];
```

The SQL just defines the shape of that "array" -- what keys each "object" must have, what types the values are, and what defaults apply.

**Why is `id` TEXT instead of INTEGER?** Because we are using UUIDs (Universally Unique Identifiers) -- long random strings like `a3f8b2c1-7d4e-4f9a-b5c6-1e2d3f4a5b6c`. These are impossible to guess, which matters because the `id` becomes part of the payment URL. If we used sequential integers (1, 2, 3...), someone could guess other payment links just by changing the number.

**Why two tables instead of one?** The `links` table stores the payment request (who should pay, how much). The `payments` table stores the actual M-Pesa transaction data that comes back from Safaricom. One link can have multiple payment attempts -- if the first attempt fails (wrong PIN, cancelled), the client might try again. This is a one-to-many relationship: one link, many payment attempts.

**What is a FOREIGN KEY?** The line `FOREIGN KEY (link_id) REFERENCES links(id)` tells the database: "Every `link_id` value in the payments table MUST match a real `id` in the links table." This prevents orphaned records -- you cannot have a payment that belongs to a link that does not exist.

---

## The Express Server

### What Is Express?

Express is a tool that does one thing: it listens for incoming HTTP requests and lets you define what to do when each URL is hit. When someone sends a GET request to `/api/links`, Express finds the function you registered for that route and runs it. That function reads from the database and sends back JSON.

Without Express, you would need to write raw Node.js HTTP handling code -- parsing URLs manually, reading request bodies byte by byte, setting response headers by hand. Express handles all of that so you can focus on your application logic.

### Creating the Server Entry Point

Create `server/index.js`:

```javascript
require("dotenv").config(); // Load .env variables into process.env
const express = require("express");
const cors = require("cors");

const linkRoutes = require("./routes/links");
const paymentRoutes = require("./routes/payments");
const receiptRoutes = require("./routes/receipts");

const app = express();

// Middleware -- these run on EVERY request before your route handlers
app.use(cors());          // Allow React on port 3000 to call this server on port 5000
app.use(express.json());  // Parse incoming JSON request bodies so req.body works

// Mount route files at their URL paths
app.use("/api/links", linkRoutes);
app.use("/api", paymentRoutes);
app.use("/api/receipts", receiptRoutes);

// A simple health check -- hit this to confirm the server is running
app.get("/api/health", (req, res) => {
  res.json({ status: "ok", timestamp: new Date().toISOString() });
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### Middleware: What It Is and Why It Matters

Middleware is a function that runs between the moment a request arrives at your server and the moment your route handler responds. Think of it like security checkpoints at a building entrance. Before a visitor (the request) reaches the office they are looking for (your route), they pass through checkpoints that inspect, modify, or block them.

You are using two middleware functions:

**`express.json()`** reads the raw bytes of an incoming request body, parses them as JSON, and attaches the result to `req.body`. Without this, when someone sends a POST request with `{ "clientName": "John Kamau" }`, your route handler would see `req.body` as `undefined`. This middleware is the reason `req.body` works.

**`cors()`** adds special headers to every response that tell the browser: "Yes, it is okay for a different origin to read this response." Remember in Weeks 8-9 when your React app fetched data from external APIs? That worked because those APIs had CORS enabled. Your React app runs on `http://localhost:3000`. Your Express server runs on `http://localhost:5000`. These are different origins (different ports). Without CORS, the browser would block every request from React to Express. The `cors()` middleware solves this.

### Create the Routes Directory

Before we write our route files, create the folders:

```bash
mkdir routes services
```

We will also need placeholder files for routes we build on Day 2. Create empty files for now:

Create `server/routes/payments.js`:

```javascript
const express = require("express");
const router = express.Router();

// We will build these routes on Day 2
// POST /pay -- trigger STK push
// POST /mpesa/callback -- receive Safaricom webhook
// GET /payment-status/:linkId -- check if payment completed

module.exports = router;
```

Create `server/routes/receipts.js`:

```javascript
const express = require("express");
const router = express.Router();

// We will build this route on Day 2
// GET /:linkId -- download a PDF receipt

module.exports = router;
```

---

## REST API Routes: Payment Links

This is where you build the core of today's work: the ability to create payment links, list them, and retrieve individual ones.

### What Is a REST API?

In Week 9, you called APIs using `fetch()`. You sent GET requests to retrieve data and POST requests to create data. REST is the pattern behind those calls. It maps HTTP methods to actions:

| HTTP Method | Action | Example |
|---|---|---|
| GET | Read / retrieve | GET /api/links -- get all links |
| POST | Create | POST /api/links -- create a new link |
| PUT/PATCH | Update | PUT /api/links/:id -- update a link |
| DELETE | Remove | DELETE /api/links/:id -- remove a link |

When you called `fetch('https://some-api.com/users')` in React, someone on the other end had written an Express route like `router.get('/users', ...)` that queried a database and sent back JSON. That is what you are writing now. You are building the other side of the conversation.

### Creating the Links Routes

Create `server/routes/links.js`:

```javascript
const express = require("express");
const { v4: uuidv4 } = require("uuid");
const db = require("../db");

const router = express.Router();

// POST / -- Create a new payment link
// This handles POST requests to /api/links (because index.js mounts this router at /api/links)
router.post("/", (req, res) => {
  // Destructure the fields from the request body
  // This is the same destructuring you learned in Week 3
  const { clientName, clientPhone, amount, description } = req.body;

  // --- Server-side validation ---
  // Always validate on the server, even if the frontend also validates.
  // Someone could bypass your frontend entirely using Postman or curl.
  if (!clientName || !clientPhone || !amount) {
    return res.status(400).json({
      error: "clientName, clientPhone, and amount are required",
    });
  }

  if (amount <= 0) {
    return res.status(400).json({ error: "Amount must be greater than 0" });
  }

  // Normalize the phone number to 254XXXXXXXXX format
  const phone = formatPhone(clientPhone);

  // Generate a unique ID for this payment link
  const id = uuidv4();

  // Insert into the database using a prepared statement
  const stmt = db.prepare(`
    INSERT INTO links (id, client_name, client_phone, amount, description)
    VALUES (?, ?, ?, ?, ?)
  `);

  stmt.run(id, clientName, phone, amount, description || null);

  // Read back the newly created link (to get the created_at timestamp and defaults)
  const link = db.prepare("SELECT * FROM links WHERE id = ?").get(id);

  // Respond with the link data plus the shareable payment URL
  res.status(201).json({
    ...link,
    paymentUrl: `${process.env.APP_URL}/pay/${id}`,
  });
});

// GET / -- List all payment links
// This handles GET requests to /api/links
router.get("/", (req, res) => {
  const links = db.prepare(
    "SELECT * FROM links ORDER BY created_at DESC"
  ).all();
  res.json(links);
});

// GET /:id -- Get a single payment link by its ID
// The :id is a URL parameter, like useParams in React Router (Week 8)
router.get("/:id", (req, res) => {
  const link = db.prepare("SELECT * FROM links WHERE id = ?").get(
    req.params.id
  );

  if (!link) {
    return res.status(404).json({ error: "Link not found" });
  }

  res.json(link);
});

// Helper function: format any Kenyan phone number to 254XXXXXXXXX
// M-Pesa requires this exact format -- no plus sign, no leading zero
function formatPhone(phone) {
  // Remove spaces and non-numeric characters (except +)
  let cleaned = phone.replace(/\s+/g, "").replace(/[^0-9+]/g, "");

  if (cleaned.startsWith("+254")) {
    cleaned = cleaned.slice(1); // Remove the + sign, keep 254...
  } else if (cleaned.startsWith("0")) {
    cleaned = "254" + cleaned.slice(1); // Replace leading 0 with 254
  }

  return cleaned;
}

module.exports = router;
```

### Breaking Down the Route Handlers

Each route handler is a function that receives two objects: `req` (the incoming request) and `res` (your response). These are just functions -- the same kind of functions you have been writing since Week 3. The only difference is that Express calls them automatically when a matching URL is hit.

**The POST route** (`router.post("/", ...)`) handles creating a new payment link. Notice the flow:

1. Read the data from `req.body` (destructuring, just like Week 3)
2. Validate that required fields exist (defensive coding)
3. Clean up the phone number
4. Generate a unique ID
5. Insert into the database
6. Send back the created resource with a 201 status code

The `?` placeholders in the SQL (`VALUES (?, ?, ?, ?, ?)`) prevent SQL injection -- a security attack where someone sends malicious SQL code in the request body. The `?` tells the database driver to treat each value as data, not as SQL code.

**The GET routes** are simpler. `.all()` returns an array of all matching rows. `.get()` returns a single row or `undefined` if nothing matches.

### Phone Number Formatting

Users in Kenya type their phone numbers in different ways:
- `0712345678` (local format)
- `+254712345678` (international format with plus)
- `254712345678` (international format without plus)

M-Pesa requires exactly `254XXXXXXXXX`. The `formatPhone` function handles all three cases. This is defensive coding -- handling the ways real users actually type things, not just the "correct" way. This is the kind of detail that separates a demo from something you can actually ship.

### Input Validation

We check that `clientName`, `clientPhone`, and `amount` all exist, and that the amount is positive. We do this on the server even though the React frontend (which we build on Day 3) will also validate.

Why both? Frontend validation is for user experience -- instant feedback without a round trip to the server. Server validation is for security. Someone could open Postman, Thunder Client, or even write a script that sends requests directly to your API, bypassing the frontend entirely. The server must never trust incoming data.

---

## Testing Your API

Start your server:

```bash
cd server
npm run dev
```

You should see `Server running on port 5000` in your terminal. If you see an error about a missing module, make sure you ran `npm install` and that all the files above are saved.

### Test 1: Health Check

Open Postman or Thunder Client. Send a GET request to:

```
http://localhost:5000/api/health
```

You should get back:

```json
{
  "status": "ok",
  "timestamp": "2026-03-29T10:30:00.000Z"
}
```

If this works, your server is running and Express is handling requests.

### Test 2: Create a Payment Link

Send a POST request to:

```
http://localhost:5000/api/links
```

Set the body to JSON:

```json
{
  "clientName": "John Kamau",
  "clientPhone": "0712345678",
  "amount": 5000,
  "description": "Website design deposit"
}
```

You should get back a 201 response with:

```json
{
  "id": "a3f8b2c1-7d4e-4f9a-b5c6-1e2d3f4a5b6c",
  "client_name": "John Kamau",
  "client_phone": "254712345678",
  "amount": 5000,
  "description": "Website design deposit",
  "status": "pending",
  "created_at": "2026-03-29 10:30:00",
  "paymentUrl": "http://localhost:3000/pay/a3f8b2c1-7d4e-4f9a-b5c6-1e2d3f4a5b6c"
}
```

Notice: the phone number was automatically converted from `0712345678` to `254712345678`. The status defaults to `"pending"`. The `paymentUrl` is what you would share with the client.

### Test 3: Validation

Send a POST request with a missing field:

```json
{
  "clientName": "Aisha Wanjiku"
}
```

You should get a 400 response:

```json
{
  "error": "clientName, clientPhone, and amount are required"
}
```

### Test 4: List All Links

Send a GET request to:

```
http://localhost:5000/api/links
```

You should get an array containing the link you created in Test 2.

### Test 5: Get a Single Link

Send a GET request to:

```
http://localhost:5000/api/links/YOUR_LINK_ID_HERE
```

Replace `YOUR_LINK_ID_HERE` with the actual `id` from Test 2.

### Test 6: Not Found

Send a GET request with a fake ID:

```
http://localhost:5000/api/links/does-not-exist
```

You should get a 404 response:

```json
{
  "error": "Link not found"
}
```

---

## Common Errors

| Error | Cause | Fix |
|---|---|---|
| `Cannot find module './routes/links'` | File path is wrong or file does not exist | Check that `server/routes/links.js` exists and the path in `require()` matches |
| `req.body is undefined` | Missing `express.json()` middleware | Make sure `app.use(express.json())` appears BEFORE your route registrations in `index.js` |
| `SQLITE_ERROR: no such table` | The database file exists but tables were not created | Delete `paylink.db` from the server folder and restart the server. The tables will be recreated automatically. |
| `Error: listen EADDRINUSE :::5000` | Port 5000 is already in use by another process | Either stop the other process or change `PORT` in your `.env` file to another number like 5001 |
| Server starts but POST returns empty body | Body is not being sent as JSON in Postman | In Postman, set the body type to "raw" and select "JSON" from the dropdown. In Thunder Client, select "JSON" as the body type. |

---

## What You Built Today

You now have:

- A Node.js project with properly organized folders and dependencies
- An `.env` file keeping secrets out of your code
- A SQLite database with two related tables (`links` and `payments`)
- An Express server with CORS and JSON parsing middleware
- Three working API endpoints: create a link, list all links, get a single link
- Input validation and phone number formatting
- All routes tested and confirmed working in Postman

**Checkpoint:** Before moving on, verify these things work:
1. `npm run dev` starts the server without errors
2. GET `/api/health` returns a JSON response
3. POST `/api/links` creates a new link and returns it with a `paymentUrl`
4. GET `/api/links` returns an array of all your created links
5. GET `/api/links/:id` returns a single link
6. POST with missing fields returns a 400 error

**Tomorrow:** You will connect this server to Safaricom's Daraja API, implement the STK push flow that triggers payments on a phone, handle webhooks (Safaricom calling YOUR server), and generate PDF receipts. The routes are ready. Tomorrow, they come alive.
