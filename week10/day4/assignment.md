# Week 10 - Day 4 Assignment

## Title
Payment Reconciliation, Persistence, and the Week 10 Ship Checklist

## Overview
Day 4 is pre-weekend polish day. Today you add persistence (a simple JSON file or SQLite for now), tie the callback back to the original STK request with proper amount validation, poll for payment completion from the React frontend, and ship a working Paylink app end to end.

## Learning Objectives Assessed
- Persist payment intents to a local store
- Match callbacks to their original STK request via CheckoutRequestID
- Validate that the amount received equals the amount requested
- Implement frontend polling for payment completion
- Pass the end-of-week ship checklist

## Prerequisites
- Days 1-3 completed

## AI Usage Rules

**Ratio:** 50/50. **Habit:** NEVER trust AI with money. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Frontend polling UX polish, loading spinners.
- **NOT ALLOWED FOR:** Amount comparison logic, the persistence write/read for money records.
- **AUDIT REQUIRED:** Yes. Final Week 10 money code audit.

## Tasks

### Task 1: Simple JSON persistence

**What to do:**
Create `services/store.js` for a tiny file-based store. This is not production grade, but it teaches the concept without introducing a database.

```javascript
const fs = require("fs");
const path = require("path");

const FILE = path.join(__dirname, "..", "data", "payments.json");

function read() {
  if (!fs.existsSync(FILE)) return {};
  return JSON.parse(fs.readFileSync(FILE, "utf8"));
}

function write(data) {
  fs.mkdirSync(path.dirname(FILE), { recursive: true });
  fs.writeFileSync(FILE, JSON.stringify(data, null, 2));
}

function savePayment(checkoutId, payment) {
  const store = read();
  store[checkoutId] = payment;
  write(store);
}

function getPayment(checkoutId) {
  const store = read();
  return store[checkoutId];
}

module.exports = { savePayment, getPayment };
```

Write every line yourself.

**Expected output:**
Store works. `data/payments.json` is created on first save. Add `data/` to `.gitignore` so you do not commit payment records.

### Task 2: Persist STK intents and reconcile on callback

**What to do:**
Modify your /mpesa/stk route to save the intent:

```javascript
const { savePayment, getPayment } = require("./services/store");

app.post("/mpesa/stk", async (req, res) => {
  // ... validation (Day 2) ...

  try {
    const result = await initiateStkPush({ phone, amount, accountRef: "TEST", description: "Payment" });
    // Persist BEFORE returning so the callback can find this record
    savePayment(result.CheckoutRequestID, {
      phone,
      amount,
      status: "pending",
      requestedAt: new Date().toISOString(),
    });
    res.json(result);
  } catch (err) {
    // ...
  }
});
```

Modify your callback to reconcile:

```javascript
app.post("/mpesa/callback", (req, res) => {
  const callback = req.body.Body?.stkCallback;
  if (!callback) return res.status(400).end();

  const checkoutId = callback.CheckoutRequestID;
  const existing = getPayment(checkoutId);
  if (!existing) {
    console.warn("Callback for unknown checkoutId:", checkoutId);
    return res.json({ status: "unknown" });
  }
  if (existing.status !== "pending") {
    console.log("Duplicate callback:", checkoutId);
    return res.json({ status: "already processed" });
  }

  const resultCode = callback.ResultCode;
  if (resultCode === 0) {
    const metadata = callback.CallbackMetadata?.Item || [];
    const amountReceived = metadata.find((i) => i.Name === "Amount")?.Value;
    const receipt = metadata.find((i) => i.Name === "MpesaReceiptNumber")?.Value;

    // Amount validation -- CRITICAL
    if (Number(amountReceived) !== Number(existing.amount)) {
      console.error(`AMOUNT MISMATCH: expected ${existing.amount}, got ${amountReceived}`);
      savePayment(checkoutId, { ...existing, status: "mismatch", amountReceived, receipt });
    } else {
      savePayment(checkoutId, { ...existing, status: "paid", amountReceived, receipt, paidAt: new Date().toISOString() });
    }
  } else {
    savePayment(checkoutId, { ...existing, status: "failed", resultCode });
  }

  res.json({ status: "ok" });
});
```

Every money line hand-written.

**Expected output:**
Successful STKs result in `data/payments.json` containing paid records.

### Task 3: Status endpoint and frontend polling

**What to do:**
Add:

```javascript
app.get("/mpesa/status/:checkoutId", (req, res) => {
  const payment = getPayment(req.params.checkoutId);
  if (!payment) return res.status(404).json({ error: "Not found" });
  res.json(payment);
});
```

In the React form, after a successful STK init, poll this endpoint every 2 seconds until status is "paid" or "failed":

```jsx
async function pollStatus(checkoutId) {
  for (let i = 0; i < 30; i++) {
    await new Promise((r) => setTimeout(r, 2000));
    const res = await fetch(`http://localhost:3001/mpesa/status/${checkoutId}`);
    if (res.ok) {
      const data = await res.json();
      if (data.status === "paid") return data;
      if (data.status === "failed" || data.status === "mismatch") throw new Error(data.status);
    }
  }
  throw new Error("Timed out");
}
```

Call this after STK init. On success, show the receipt link.

**Expected output:**
Polling updates the UI when the payment completes.

### Task 4: Real receipt generation from store

**What to do:**
Update `/mpesa/receipt/:checkoutId` to read from the real store:

```javascript
app.get("/mpesa/receipt/:checkoutId", async (req, res) => {
  const payment = getPayment(req.params.checkoutId);
  if (!payment || payment.status !== "paid") {
    return res.status(404).send("Not found or not yet paid");
  }

  const pdf = await generateReceipt({
    phone: payment.phone,
    amount: payment.amount,
    reference: payment.receipt,
    date: payment.paidAt,
  });

  res.setHeader("Content-Type", "application/pdf");
  res.send(pdf);
});
```

**Expected output:**
Receipt shows real payment details, not test data.

### Task 5: Ship checklist

**What to do:**
Create `CHECKLIST.md`:

```markdown
## Week 10 Day 4 Ship Checklist

- [ ] Express backend running and exposed via ngrok
- [ ] OAuth token exchange working
- [ ] STK Push initiator working
- [ ] Callback handler with idempotency working
- [ ] Amount validation comparing callback to stored intent
- [ ] Simple JSON store persisting payments
- [ ] React form submits and polls for status
- [ ] PDF receipt generation working
- [ ] data/ folder in .gitignore
- [ ] .env NOT in the repo
- [ ] Money code audit current for all 4 days
- [ ] Repo pushed and clean
```

Tick every box honestly.

**Expected output:**
Checklist committed.

## Stretch Goals (Optional - Extra Credit)

- Replace the JSON store with SQLite using `better-sqlite3`.
- Add a retry button for failed payments.
- Deploy the backend to Render/Railway with a real URL.

## Submission Requirements

- **What to submit:** Repo, `CHECKLIST.md`, updated `AI_AUDIT.md`.
- **Deadline:** End of Day 4

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Persistence working | 20 | JSON store created. STK intents persisted. |
| Callback reconciles with store | 25 | Only known CheckoutIDs processed. Duplicate callbacks rejected. |
| Amount validation | 25 | Mismatch is caught and logged. Correct case flows to "paid". |
| Status polling from frontend | 15 | UI updates when payment completes. |
| Real receipt from store | 10 | PDF shows real payment details. |
| Ship checklist | 5 | All boxes honest. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Trusting Safaricom's amount.** Always compare to your stored intent. If there is a mismatch, flag it and do not mark as paid.
- **Committing the data folder.** `data/payments.json` has real sandbox traffic that leaks test patterns. Add `data/` to `.gitignore`.
- **Polling forever.** Cap the poll count (30 attempts = 60 seconds). After that, prompt the user to retry.

## Resources

- Prior Day 1-3 assignments
- Week 10 AI boundaries: [../ai.md](../ai.md)
- Daraja callback reference: https://developer.safaricom.co.ke/Documentation
