# Week 10, Day 2: M-Pesa Integration and PDF Receipts

By the end of today, your backend will be fully functional. It will authenticate with Safaricom, trigger STK push payments on a phone, receive webhook callbacks when payments complete, and generate branded PDF receipts. You will test the entire payment flow using the Daraja sandbox.

This is the hardest day of the capstone. The M-Pesa integration involves concepts you have not encountered before -- OAuth tokens, webhook callbacks, and asynchronous payment flows. Take it slowly. Every piece builds on things you already know.

**Prior-week concepts you will use today:**
- Promises, async/await, try/catch (Week 5)
- Fetch API and HTTP requests (Week 5) -- now from Node.js with axios
- JSON.parse and JSON.stringify (Week 5)
- Callback functions (Week 3 and 5)
- CSS positioning concepts as analogy for PDFKit (Weeks 1-2)

**Estimated time:** 4-5 hours

---

## Recap: What You Built Yesterday

On Day 1 you built:

- An Express server with middleware (CORS, JSON parsing)
- A SQLite database with `links` and `payments` tables
- REST API routes to create and retrieve payment links
- Input validation and phone number formatting

Today you connect your server to the outside world. You will make it talk to Safaricom's servers, and Safaricom will talk back to yours.

---

## Understanding the M-Pesa STK Push Flow

Before writing any code, you need to understand the full payment flow. This is a numbered sequence -- follow it step by step.

1. **Your client opens the payment link** in their browser. They see the amount and a "Pay" button.
2. **The client taps "Pay".** Your React frontend sends a POST request to your Express backend (`POST /api/pay`).
3. **Your backend calls Safaricom's API.** It sends an HTTP request saying: "Please ask phone number 254712345678 to pay KES 5,000 to shortcode 174379."
4. **Safaricom sends an STK push to the client's phone.** The client sees a green popup asking them to enter their M-Pesa PIN.
5. **Safaricom responds to your backend immediately** with a "request accepted" message. This is NOT the payment result. It just means Safaricom received your request.
6. **The client enters their PIN (or cancels).**
7. **Safaricom processes the transaction** and then **sends a POST request to your callback URL** with the result -- success or failure, including the M-Pesa receipt number.
8. **Your backend receives the callback**, updates the database, and generates a PDF receipt.
9. **Your frontend (polling every few seconds)** detects that the payment status changed to "paid" and shows a success message.

The key insight: this is a **two-step asynchronous process**. Step 3 sends the request. Step 7 receives the result. These happen at different times, in different HTTP requests. You do not get the payment result in the response to Step 3.

Remember Promises from Week 5? You learned that some operations take time -- you make a request, and the result comes later. The M-Pesa flow is the real-world version of that concept. The difference is that instead of JavaScript resolving a Promise, Safaricom calls your server back with the result.

---

## OAuth Authentication with Safaricom

Before you can trigger an STK push, you need to prove to Safaricom that you are authorized to use their API. This is done through OAuth -- a standard way for applications to authenticate with each other.

The process is straightforward:

1. You combine your Consumer Key and Consumer Secret into a single string, separated by a colon
2. You Base64-encode that string
3. You send it to Safaricom's OAuth endpoint
4. Safaricom sends back an access token that is valid for 60 minutes
5. You include that token in every subsequent API call

This is similar to what you did in Week 5 when you called APIs with `fetch()`. Some APIs require an API key in the headers. The Daraja API uses OAuth, which is a more formal version of the same idea -- you prove who you are, and you get a temporary pass.

### Building the M-Pesa Service

Create `server/services/mpesa.js`:

```javascript
const axios = require("axios");

// Sandbox URL -- in production, this changes to the live URL
const DARAJA_BASE_URL = "https://sandbox.safaricom.co.ke";

// We cache the token so we don't request a new one on every single payment
// This avoids unnecessary API calls and respects Safaricom's rate limits
let tokenCache = {
  token: null,
  expiresAt: 0,
};

/**
 * Get an OAuth access token from Safaricom.
 * Returns a cached token if it's still valid. Requests a new one if expired.
 */
async function getAccessToken() {
  const now = Date.now();

  // If we have a cached token that won't expire in the next 60 seconds, reuse it
  if (tokenCache.token && tokenCache.expiresAt > now + 60000) {
    return tokenCache.token;
  }

  // Combine Consumer Key and Secret, then Base64-encode them
  // This is how HTTP Basic Authentication works
  const auth = Buffer.from(
    `${process.env.MPESA_CONSUMER_KEY}:${process.env.MPESA_CONSUMER_SECRET}`
  ).toString("base64");

  // Request a new token from Safaricom
  const response = await axios.get(
    `${DARAJA_BASE_URL}/oauth/v1/generate?grant_type=client_credentials`,
    {
      headers: {
        Authorization: `Basic ${auth}`,
      },
    }
  );

  // Cache the token with its expiry time
  tokenCache = {
    token: response.data.access_token,
    expiresAt: now + response.data.expires_in * 1000,
  };

  return tokenCache.token;
}

/**
 * Initiate an STK Push request.
 * This triggers the M-Pesa payment prompt on the customer's phone.
 *
 * @param {string} phone   - Phone number in 254XXXXXXXXX format
 * @param {number} amount  - Amount in KES
 * @param {string} linkId  - Your internal reference (the payment link ID)
 */
async function initiateSTKPush(phone, amount, linkId) {
  const token = await getAccessToken();
  const timestamp = generateTimestamp();
  const shortcode = process.env.MPESA_SHORTCODE;
  const passkey = process.env.MPESA_PASSKEY;

  // The password is a Base64 encoding of: Shortcode + Passkey + Timestamp
  // This is Safaricom's way of verifying you are authorized for this shortcode
  const password = Buffer.from(shortcode + passkey + timestamp).toString(
    "base64"
  );

  const payload = {
    BusinessShortCode: shortcode,
    Password: password,
    Timestamp: timestamp,
    TransactionType: "CustomerPayBillOnline",
    Amount: Math.ceil(amount), // M-Pesa only accepts whole numbers, no decimals
    PartyA: phone,             // The phone number being charged (the customer)
    PartyB: shortcode,         // The business receiving the payment
    PhoneNumber: phone,
    CallBackURL: process.env.MPESA_CALLBACK_URL,
    AccountReference: linkId.slice(0, 12), // Max 12 characters -- Safaricom truncates longer strings
    TransactionDesc: `Payment for invoice ${linkId.slice(0, 8)}`,
  };

  const response = await axios.post(
    `${DARAJA_BASE_URL}/mpesa/stkpush/v1/processrequest`,
    payload,
    {
      headers: {
        Authorization: `Bearer ${token}`,
        "Content-Type": "application/json",
      },
    }
  );

  return response.data;
}

/**
 * Generate a timestamp in the exact format Safaricom expects: YYYYMMDDHHmmss
 * For example: 20260329143022 means March 29, 2026 at 2:30:22 PM
 */
function generateTimestamp() {
  const now = new Date();
  const pad = (n) => String(n).padStart(2, "0");

  return (
    now.getFullYear() +
    pad(now.getMonth() + 1) + // Months are 0-indexed in JavaScript
    pad(now.getDate()) +
    pad(now.getHours()) +
    pad(now.getMinutes()) +
    pad(now.getSeconds())
  );
}

module.exports = { initiateSTKPush };
```

### Understanding the STK Push Payload

Every field in the payload has a purpose. Safaricom will reject your request if any are wrong:

**BusinessShortCode** -- The paybill or till number receiving the payment. In sandbox, always `174379`.

**Password** -- Created by concatenating the shortcode, passkey, and timestamp, then Base64-encoding the result. This proves you are authorized to trigger payments for this shortcode. If the timestamp in the password does not match the Timestamp field, Safaricom rejects the request.

**Timestamp** -- The current date and time in `YYYYMMDDHHmmss` format. Must match the timestamp baked into the password.

**TransactionType** -- Always `"CustomerPayBillOnline"` for paybill payments.

**Amount** -- The amount in KES as a whole number. M-Pesa does not handle decimals. `Math.ceil()` rounds up -- if someone owes KES 500.50, we charge KES 501. In a real app, you would likely round to the nearest shilling.

**PartyA** -- The customer's phone number. This is the phone that receives the STK push popup.

**PartyB** -- The business shortcode. The money goes here.

**CallBackURL** -- The URL where Safaricom will POST the payment result. This must be publicly accessible from the internet. Your `localhost:5000` is not publicly accessible, which is why we need ngrok (explained below).

**AccountReference** -- A reference string that appears on the customer's phone in the STK push popup. Maximum 12 characters. We use the first 12 characters of the payment link ID.

**TransactionDesc** -- A human-readable description of the payment.

### Token Caching

Notice the `tokenCache` object. Every time you call `getAccessToken()`, it checks if the cached token is still valid. If it is, it returns the cached token immediately without making an HTTP request. If it has expired (or is about to expire within 60 seconds), it requests a fresh one.

Why cache? Two reasons. First, performance: making an HTTP request to Safaricom takes time. If you requested a new token for every payment, the customer would wait longer. Second, rate limits: Safaricom limits how many times you can call their OAuth endpoint. Caching respects that limit.

---

## The Payment Route

Now you need a route that your frontend can call to trigger a payment. Replace the contents of `server/routes/payments.js`:

```javascript
const express = require("express");
const db = require("../db");
const { initiateSTKPush } = require("../services/mpesa");
const { generateReceipt } = require("../services/receipt");

const router = express.Router();

// POST /pay -- Trigger an M-Pesa STK Push
router.post("/pay", async (req, res) => {
  const { linkId, phone } = req.body;

  // Look up the payment link in the database
  const link = db.prepare("SELECT * FROM links WHERE id = ?").get(linkId);

  if (!link) {
    return res.status(404).json({ error: "Payment link not found" });
  }

  // Don't allow paying a link that's already been paid
  if (link.status === "paid") {
    return res.status(400).json({ error: "This link has already been paid" });
  }

  // Use the phone from the request, or fall back to the one stored in the link
  const payPhone = phone
    ? formatPhone(phone)
    : link.client_phone;

  try {
    // Create a pending payment record BEFORE calling Safaricom
    // This way, when the callback arrives, we have a record to match it to
    const stmt = db.prepare(`
      INSERT INTO payments (link_id, phone_number, amount, status)
      VALUES (?, ?, ?, 'pending')
    `);
    stmt.run(linkId, payPhone, link.amount);

    // Call Safaricom's STK Push API
    const result = await initiateSTKPush(payPhone, link.amount, linkId);

    // ResponseCode "0" means Safaricom accepted the request
    // This does NOT mean the payment succeeded -- it means the STK push was sent
    if (result.ResponseCode === "0") {
      res.json({
        message: "STK push sent. Check your phone.",
        checkoutRequestId: result.CheckoutRequestID,
      });
    } else {
      res.status(400).json({
        error: "Failed to initiate payment",
        details: result.ResponseDescription,
      });
    }
  } catch (error) {
    console.error("STK Push Error:", error.response?.data || error.message);
    res.status(500).json({ error: "Payment initiation failed" });
  }
});

// POST /mpesa/callback -- Safaricom calls this endpoint with the payment result
router.post("/mpesa/callback", async (req, res) => {
  // CRITICAL: Respond with 200 IMMEDIATELY, before doing any processing.
  // If you take too long or return an error, Safaricom will retry the callback,
  // and you might process the same payment twice.
  res.json({ ResultCode: 0, ResultDesc: "Accepted" });

  // Extract the callback data from Safaricom's payload
  const callback = req.body.Body?.stkCallback;

  if (!callback) {
    console.error("Invalid callback payload:", req.body);
    return;
  }

  const {
    MerchantRequestID,
    CheckoutRequestID,
    ResultCode,
    ResultDesc,
    CallbackMetadata,
  } = callback;

  console.log(`Callback received: ResultCode=${ResultCode}, ${ResultDesc}`);

  // ResultCode 0 means the payment was successful
  // Any other code means failure (cancelled, wrong PIN, insufficient funds, etc.)
  if (ResultCode !== 0) {
    console.log("Payment failed or was cancelled:", ResultDesc);
    return;
  }

  // Extract the payment details from the metadata
  // Safaricom sends metadata as an array of { Name, Value } objects
  const metadata = {};
  if (CallbackMetadata?.Item) {
    CallbackMetadata.Item.forEach((item) => {
      metadata[item.Name] = item.Value;
    });
  }

  const mpesaReceipt = metadata.MpesaReceiptNumber;
  const amount = metadata.Amount;
  const transactionDate = String(metadata.TransactionDate);
  const phoneNumber = String(metadata.PhoneNumber);

  // Find the matching pending payment in our database
  // We match by phone number and amount since AccountReference gets truncated
  const payment = db
    .prepare(
      `SELECT p.*, l.id as lid FROM payments p
       JOIN links l ON p.link_id = l.id
       WHERE p.phone_number = ? AND p.amount = ? AND p.status = 'pending'
       ORDER BY p.created_at DESC LIMIT 1`
    )
    .get(phoneNumber, amount);

  if (!payment) {
    console.error("No matching pending payment found for:", {
      phoneNumber,
      amount,
    });
    return;
  }

  // Update the payment record with the M-Pesa transaction details
  db.prepare(
    `UPDATE payments
     SET mpesa_receipt = ?, transaction_date = ?, status = 'completed',
         raw_callback = ?
     WHERE id = ?`
  ).run(
    mpesaReceipt,
    transactionDate,
    JSON.stringify(req.body), // Store the full callback as JSON for debugging
    payment.id
  );

  // Mark the payment link as paid
  db.prepare("UPDATE links SET status = 'paid' WHERE id = ?").run(
    payment.link_id
  );

  // Generate a PDF receipt
  const link = db.prepare("SELECT * FROM links WHERE id = ?").get(
    payment.link_id
  );

  try {
    await generateReceipt({
      receiptNumber: mpesaReceipt,
      clientName: link.client_name,
      amount: amount,
      phone: phoneNumber,
      description: link.description,
      date: transactionDate,
      linkId: link.id,
    });
    console.log(`Receipt generated for ${mpesaReceipt}`);
  } catch (err) {
    console.error("Receipt generation failed:", err);
  }
});

// GET /payment-status/:linkId -- Check if a payment has been completed
// The frontend polls this endpoint to know when the callback has arrived
router.get("/payment-status/:linkId", (req, res) => {
  const link = db.prepare("SELECT * FROM links WHERE id = ?").get(
    req.params.linkId
  );

  if (!link) {
    return res.status(404).json({ error: "Link not found" });
  }

  const payment = db
    .prepare(
      "SELECT * FROM payments WHERE link_id = ? ORDER BY created_at DESC LIMIT 1"
    )
    .get(req.params.linkId);

  res.json({
    linkStatus: link.status,
    payment: payment
      ? {
          status: payment.status,
          mpesaReceipt: payment.mpesa_receipt,
        }
      : null,
  });
});

// Helper: format phone to 254XXXXXXXXX (same function from links.js)
function formatPhone(phone) {
  let cleaned = phone.replace(/\s+/g, "").replace(/[^0-9+]/g, "");
  if (cleaned.startsWith("+254")) cleaned = cleaned.slice(1);
  else if (cleaned.startsWith("0")) cleaned = "254" + cleaned.slice(1);
  return cleaned;
}

module.exports = router;
```

### The POST /pay Route

When the frontend sends `{ linkId: "abc123", phone: "0712345678" }`, this route:

1. Looks up the payment link in the database
2. Checks that it has not already been paid
3. Creates a pending payment record (so we have something to match the callback to)
4. Calls `initiateSTKPush()` -- which makes the HTTP request to Safaricom
5. Returns either a success message ("STK push sent") or an error

The `try/catch` block around the Safaricom API call is essential. Remember from Week 5 when you learned error handling? Network requests can fail for many reasons: invalid credentials, expired token, Safaricom's servers being down, wrong phone number format. Without try/catch, an error would crash your entire server.

---

## Webhooks: The Callback Endpoint

This is the most important concept of the day. Give it the time it deserves.

### What Is a Webhook?

A webhook is the opposite of an API call. When your server calls Safaricom's API, that is a normal API call -- you ask for something. A webhook is when Safaricom calls YOUR server -- they tell you something.

Think of it this way. Without a webhook, you would need to ask Safaricom every 2 seconds: "Is the payment done yet? Is it done now? How about now?" That is called polling. It works, but it is wasteful.

With a webhook, you tell Safaricom: "When the payment is done, call this URL." Then you wait. When the payment completes, Safaricom sends an HTTP POST request to your callback URL with all the details. You process it once, and you are done.

This is event-driven architecture. Many services use webhooks: Stripe for payments, GitHub for pull request events, Slack for message events. M-Pesa uses them for payment confirmations.

### The Callback Payload

When a payment succeeds, Safaricom sends this JSON to your callback URL:

```json
{
  "Body": {
    "stkCallback": {
      "MerchantRequestID": "29115-34620561-1",
      "CheckoutRequestID": "ws_CO_191220191020363925",
      "ResultCode": 0,
      "ResultDesc": "The service request is processed successfully.",
      "CallbackMetadata": {
        "Item": [
          { "Name": "Amount", "Value": 1.00 },
          { "Name": "MpesaReceiptNumber", "Value": "NLJ7RT61SV" },
          { "Name": "TransactionDate", "Value": 20191219102115 },
          { "Name": "PhoneNumber", "Value": 254708374149 }
        ]
      }
    }
  }
}
```

When a payment fails (user cancelled, wrong PIN, insufficient funds):

```json
{
  "Body": {
    "stkCallback": {
      "MerchantRequestID": "29115-34620561-1",
      "CheckoutRequestID": "ws_CO_191220191020363925",
      "ResultCode": 1032,
      "ResultDesc": "Request cancelled by user"
    }
  }
}
```

Notice that `CallbackMetadata` only exists when the payment succeeds. On failure, you only get the `ResultCode` and `ResultDesc`.

### The Critical Rule: Respond 200 Immediately

Look at the very first line inside the callback route handler:

```javascript
res.json({ ResultCode: 0, ResultDesc: "Accepted" });
```

We respond to Safaricom BEFORE processing anything. This is not optional. If your callback takes too long to respond (because you are updating the database, generating a PDF, etc.), Safaricom assumes the request failed and retries it. If it retries, you might process the same payment twice -- charging the customer once but creating two receipt records.

So: respond immediately, then process. This is a pattern you will see in every webhook implementation, not just M-Pesa.

### Making Your Callback URL Public with ngrok

Your server runs on `localhost:5000`. Safaricom cannot reach `localhost` -- it only exists on your machine. You need a way to make your local server accessible from the internet, at least during development.

ngrok creates a tunnel: it gives you a public URL (like `https://a1b2c3d4.ngrok.io`) that forwards traffic to your local server.

```bash
# Install ngrok if you haven't already
npm install -g ngrok

# Start the tunnel (in a separate terminal)
ngrok http 5000
```

ngrok will display something like:

```
Forwarding  https://a1b2c3d4.ngrok.io -> http://localhost:5000
```

Copy that `https://` URL and update your `server/.env`:

```env
MPESA_CALLBACK_URL=https://a1b2c3d4.ngrok.io/api/mpesa/callback
```

Restart your server after changing the callback URL. Every time you restart ngrok, you get a new URL, so you will need to update `.env` again.

### The Status Polling Endpoint

The `GET /payment-status/:linkId` route is the bridge between your backend and frontend. After the frontend triggers an STK push, it needs to know when the payment completes. Since the callback is between Safaricom and your server (the frontend is not involved), the frontend polls this endpoint every few seconds to check if the status has changed from "pending" to "paid".

We will build the polling logic in the React frontend on Day 3.

---

## PDF Receipt Generation

When a payment succeeds, your app generates a branded PDF receipt that the client can download. This is what turns a basic payment tool into something professional.

### What Is PDFKit?

PDFKit lets you create PDF documents with code. Instead of designing a receipt in Microsoft Word, you write JavaScript that tells PDFKit exactly where to place text, draw lines, fill rectangles, and set fonts. You are building a document the same way you built web pages with CSS -- by specifying positions, sizes, and styles.

Remember in Weeks 1-2 when you learned CSS positioning? PDFKit is similar. You tell the computer exactly where to place each element using x and y coordinates. The difference is you are building a PDF document instead of a web page. The coordinate system starts at the top-left corner of the page: x increases to the right, y increases downward. Positions are measured in points (1 point = 1/72 inch).

### Building the Receipt Generator

Create `server/services/receipt.js`:

```javascript
const PDFDocument = require("pdfkit");
const fs = require("fs");
const path = require("path");

const RECEIPTS_DIR = path.join(__dirname, "..", "receipts");

// Create the receipts directory if it doesn't exist
if (!fs.existsSync(RECEIPTS_DIR)) {
  fs.mkdirSync(RECEIPTS_DIR, { recursive: true });
}

/**
 * Generate a branded PDF receipt for a completed payment.
 */
async function generateReceipt(data) {
  return new Promise((resolve, reject) => {
    // Create a new A4-sized PDF with 50-point margins on all sides
    const doc = new PDFDocument({
      size: "A4",
      margin: 50,
    });

    // The PDF will be saved as a file named after the payment link ID
    const filePath = path.join(RECEIPTS_DIR, `${data.linkId}.pdf`);
    const stream = fs.createWriteStream(filePath);

    // Pipe the PDF content to the file
    doc.pipe(stream);

    // ---- HEADER ----
    // Large, bold title centered at the top
    doc
      .fontSize(24)
      .font("Helvetica-Bold")
      .fillColor("#153564")
      .text("PAYMENT RECEIPT", { align: "center" });

    doc.moveDown(0.5); // Move down half a line height

    // Orange horizontal line under the title
    doc
      .strokeColor("#FF6600")
      .lineWidth(2)
      .moveTo(50, doc.y)       // Start at left margin, current y position
      .lineTo(545, doc.y)      // Draw to right margin
      .stroke();               // Actually render the line

    doc.moveDown(1);

    // ---- RECEIPT DETAILS (two-column layout) ----
    const startY = doc.y;      // Remember current position for alignment
    const leftCol = 50;        // Left column x position
    const rightCol = 300;      // Right column x position

    doc.fontSize(10).font("Helvetica").fillColor("#333333");

    // Left column: receipt number, date, client name
    drawField(doc, "Receipt Number", data.receiptNumber, leftCol, startY);
    drawField(doc, "Date", formatMpesaDate(data.date), leftCol, startY + 40);
    drawField(doc, "Client", data.clientName, leftCol, startY + 80);

    // Right column: phone number, reference
    drawField(doc, "Phone", formatDisplayPhone(data.phone), rightCol, startY);
    drawField(
      doc,
      "Reference",
      data.linkId.slice(0, 8).toUpperCase(),
      rightCol,
      startY + 40
    );

    // ---- AMOUNT BOX ----
    // A light blue rectangle with the payment amount prominently displayed
    doc.y = startY + 140;

    doc.rect(50, doc.y, 495, 60).fill("#F0F4F8");

    doc
      .fontSize(12)
      .font("Helvetica")
      .fillColor("#666666")
      .text("Amount Paid", 70, doc.y + 12);

    doc
      .fontSize(28)
      .font("Helvetica-Bold")
      .fillColor("#153564")
      .text(`KES ${Number(data.amount).toLocaleString()}`, 70, doc.y + 28);

    doc.y += 80;

    // ---- DESCRIPTION (if provided) ----
    if (data.description) {
      doc
        .fontSize(10)
        .font("Helvetica-Bold")
        .fillColor("#333333")
        .text("Description", 50, doc.y);

      doc.moveDown(0.3);

      doc
        .fontSize(10)
        .font("Helvetica")
        .fillColor("#555555")
        .text(data.description, 50, doc.y, { width: 495 });

      doc.moveDown(1);
    }

    // ---- FOOTER ----
    doc.moveDown(2);

    // Thin gray line
    doc
      .strokeColor("#CCCCCC")
      .lineWidth(0.5)
      .moveTo(50, doc.y)
      .lineTo(545, doc.y)
      .stroke();

    doc.moveDown(0.5);

    doc
      .fontSize(8)
      .font("Helvetica")
      .fillColor("#999999")
      .text("This is a computer-generated receipt. No signature required.", {
        align: "center",
      });

    doc.text("Powered by Mctaba Paylink", { align: "center" });

    // ---- FINALIZE ----
    doc.end(); // Signal that we're done writing to the PDF

    // Resolve the promise when the file is fully written
    stream.on("finish", () => resolve(filePath));
    stream.on("error", reject);
  });
}

/**
 * Draw a label-value pair at a specific (x, y) position.
 * The label is small and gray. The value is larger and bold.
 */
function drawField(doc, label, value, x, y) {
  doc.fontSize(8).font("Helvetica").fillColor("#999999").text(label, x, y);

  doc
    .fontSize(11)
    .font("Helvetica-Bold")
    .fillColor("#333333")
    .text(value || "N/A", x, y + 14);
}

/**
 * Convert M-Pesa date format (YYYYMMDDHHmmss) to a human-readable string.
 * Example: "20260329143022" becomes "29 Mar 2026, 14:30"
 */
function formatMpesaDate(dateStr) {
  if (!dateStr || dateStr.length < 14) return dateStr;

  const s = String(dateStr);
  const year = s.slice(0, 4);
  const month = s.slice(4, 6);
  const day = s.slice(6, 8);
  const hour = s.slice(8, 10);
  const min = s.slice(10, 12);

  const months = [
    "Jan", "Feb", "Mar", "Apr", "May", "Jun",
    "Jul", "Aug", "Sep", "Oct", "Nov", "Dec",
  ];

  return `${parseInt(day)} ${months[parseInt(month) - 1]} ${year}, ${hour}:${min}`;
}

/**
 * Format a phone number for display on the receipt.
 * Converts 254712345678 to +254 712 345 678
 */
function formatDisplayPhone(phone) {
  const p = String(phone);
  if (p.length === 12 && p.startsWith("254")) {
    return `+254 ${p.slice(3, 6)} ${p.slice(6, 9)} ${p.slice(9)}`;
  }
  return p;
}

module.exports = { generateReceipt };
```

### How PDFKit Coordinates Work

The key functions you see in the receipt code:

- `doc.text(string, x, y, options)` -- Write text at position (x, y)
- `doc.rect(x, y, width, height).fill(color)` -- Draw a filled rectangle
- `doc.moveTo(x, y).lineTo(x, y).stroke()` -- Draw a line from one point to another
- `doc.fontSize(n)`, `doc.font(name)`, `doc.fillColor(hex)` -- Set text styling
- `doc.y` -- Tracks the current vertical position. After writing text, `doc.y` moves down automatically.
- `doc.moveDown(n)` -- Advance the position by `n` line heights

The receipt is built top to bottom: header first, then the details grid, then the amount box, then the description, then the footer. Each section positions its elements using exact coordinates.

### The Receipt Download Route

Replace the contents of `server/routes/receipts.js`:

```javascript
const express = require("express");
const path = require("path");
const fs = require("fs");

const router = express.Router();

const RECEIPTS_DIR = path.join(__dirname, "..", "receipts");

// GET /:linkId -- Download the PDF receipt for a payment link
router.get("/:linkId", (req, res) => {
  const filePath = path.join(RECEIPTS_DIR, `${req.params.linkId}.pdf`);

  // Check if the receipt file exists
  if (!fs.existsSync(filePath)) {
    return res.status(404).json({ error: "Receipt not found" });
  }

  // res.download() sets the right headers so the browser downloads the file
  // instead of trying to display it inline
  res.download(filePath, `receipt-${req.params.linkId.slice(0, 8)}.pdf`);
});

module.exports = router;
```

---

## Testing the Backend

At this point, your entire backend is complete. Here is how to verify each piece works.

### Test the STK Push

1. Make sure your server is running: `npm run dev`
2. Make sure ngrok is running: `ngrok http 5000`
3. Update `.env` with the new ngrok URL and restart the server
4. Create a payment link first (Day 1 test): POST to `/api/links`
5. Then trigger the STK push: POST to `/api/pay`

```json
{
  "linkId": "YOUR_LINK_ID_HERE",
  "phone": "254708374149"
}
```

Use the test phone number from the Daraja sandbox documentation.

If successful, you should see:

```json
{
  "message": "STK push sent. Check your phone.",
  "checkoutRequestId": "ws_CO_..."
}
```

### Test the Callback

In the sandbox, Safaricom will send the callback automatically. Watch your server terminal for log messages:

```
Callback received: ResultCode=0, The service request is processed successfully.
Receipt generated for NLJ7RT61SV
```

### Test the Receipt Download

Send a GET request to:

```
http://localhost:5000/api/receipts/YOUR_LINK_ID_HERE
```

Your browser (or Postman) should download a PDF file. Open it and verify it shows the correct receipt details.

### Test the Payment Status

Send a GET request to:

```
http://localhost:5000/api/payment-status/YOUR_LINK_ID_HERE
```

You should see:

```json
{
  "linkStatus": "paid",
  "payment": {
    "status": "completed",
    "mpesaReceipt": "NLJ7RT61SV"
  }
}
```

---

## Common Errors

| Error | Cause | Fix |
|---|---|---|
| STK push returns `ResponseCode: 1` | Invalid credentials or expired token | Double-check your Consumer Key, Consumer Secret, and Passkey in `.env`. Make sure they match your Daraja sandbox app. |
| Callback never arrives | ngrok URL does not match, or ngrok is not running | Re-copy the ngrok URL to `.env` and restart the server. Make sure ngrok is running in a separate terminal. |
| `Cannot find module '../services/receipt'` | The receipt.js file is missing or path is wrong | Verify `server/services/receipt.js` exists |
| Phone gets STK push but callback says failed | User entered wrong PIN, cancelled, or insufficient funds | Check the `ResultCode` in the callback. `1032` = cancelled by user. `1` = insufficient balance. |
| `ECONNREFUSED` when calling Safaricom | Network issue or wrong base URL | Check your internet connection. Verify the base URL is `https://sandbox.safaricom.co.ke` |
| Receipt PDF is empty or corrupted | The `generateReceipt` function threw an error | Check the server terminal for error messages. Common cause: the `receipts/` directory was not created. |

---

## What You Built Today

You now have a fully functional backend that can:

- Authenticate with Safaricom's Daraja API using OAuth
- Trigger STK push payments on any phone number
- Receive webhook callbacks when payments complete
- Store transaction data in the database
- Generate branded PDF receipts
- Serve receipts for download

**Checkpoint:** Before moving on, verify these things work:
1. POST `/api/pay` with a valid link ID sends an STK push (you see the success response)
2. Your server logs show the callback arriving from Safaricom
3. The payment link status changes from "pending" to "paid" in the database
4. A PDF receipt file appears in the `server/receipts/` folder
5. GET `/api/receipts/:linkId` downloads the PDF
6. GET `/api/payment-status/:linkId` shows the updated status

**Tomorrow:** You build the React frontend that ties everything together. A dashboard for creating payment links, a payment page for clients, and real-time status updates using the polling endpoint. Your backend is ready. Tomorrow it gets a face.
