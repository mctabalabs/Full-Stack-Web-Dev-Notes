# Week 10 - Day 2 Assignment

## Title
M-Pesa STK Push -- Initiate a Real Payment

## Overview
Today you wire up the actual M-Pesa STK Push. You will write the OAuth token exchange, the STK Push initiator, and handle the callback. Every money-related line is hand-written. By the end of today, triggering an endpoint on your Express server will ring a real phone in sandbox and ask for a PIN.

## Learning Objectives Assessed
- Exchange Consumer Key + Secret for an access token
- Construct a valid Daraja STK Push request
- Handle the callback from Daraja with idempotency
- Verify the amount Safaricom reports matches what you asked for
- Test the full loop end-to-end with sandbox credentials

## Prerequisites
- Day 1 completed (sandbox account, backend scaffolded, ngrok running)

## AI Usage Rules

**Ratio:** 50/50. **Habit:** NEVER trust AI with money. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Explaining a Daraja error code after you hit it.
- **NOT ALLOWED FOR:** Writing token exchange, STK initiator, callback handler, or amount validation. Rewrite by hand if you hit Tab accept by accident.
- **AUDIT REQUIRED:** Yes. Money code audit expanded this week.

## Tasks

### Task 1: OAuth token exchange

**What to do:**
Create `services/mpesa.js` in your backend. Write the token function by hand:

```javascript
const axios = require("axios");

async function getToken() {
  const auth = Buffer.from(
    `${process.env.MPESA_CONSUMER_KEY}:${process.env.MPESA_CONSUMER_SECRET}`
  ).toString("base64");

  const res = await axios.get(
    "https://sandbox.safaricom.co.ke/oauth/v1/generate?grant_type=client_credentials",
    {
      headers: { Authorization: `Basic ${auth}` },
    }
  );

  return res.data.access_token;
}

module.exports = { getToken };
```

Test it in a small script: `node -e "require('./services/mpesa').getToken().then(console.log)"`

**Expected output:**
A long access token prints. Screenshot `day2-token.png`.

### Task 2: STK Push initiator

**What to do:**
Add to `services/mpesa.js`:

```javascript
async function initiateStkPush({ phone, amount, accountRef, description }) {
  const token = await getToken();
  const timestamp = new Date().toISOString().replace(/[-T:\.Z]/g, "").slice(0, 14);
  const shortcode = process.env.MPESA_SHORTCODE;
  const passkey = process.env.MPESA_PASSKEY;
  const password = Buffer.from(`${shortcode}${passkey}${timestamp}`).toString("base64");

  const payload = {
    BusinessShortCode: shortcode,
    Password: password,
    Timestamp: timestamp,
    TransactionType: "CustomerPayBillOnline",
    Amount: amount,
    PartyA: phone,
    PartyB: shortcode,
    PhoneNumber: phone,
    CallBackURL: `${process.env.PUBLIC_URL}/mpesa/callback`,
    AccountReference: accountRef,
    TransactionDesc: description,
  };

  const res = await axios.post(
    "https://sandbox.safaricom.co.ke/mpesa/stkpush/v1/processrequest",
    payload,
    { headers: { Authorization: `Bearer ${token}` } }
  );

  return res.data;
}

module.exports = { getToken, initiateStkPush };
```

Add `MPESA_SHORTCODE=174379` and `MPESA_PASSKEY=bfb279f9aa9bdbcf158e97dd71a467cd2e0c893059b10f78e6b72ada1ed2c919` (these are the public sandbox test values) and `PUBLIC_URL=your-ngrok-url` to your `.env`.

Type every line yourself.

**Expected output:**
`services/mpesa.js` with token exchange and STK initiator, all hand-written.

### Task 3: Expose the STK endpoint

**What to do:**
Add a route to `index.js`:

```javascript
const { initiateStkPush } = require("./services/mpesa");

app.post("/mpesa/stk", async (req, res) => {
  const { phone, amount } = req.body;

  // Validate -- manually, never trust AI with money
  if (!phone || !/^2547\d{8}$/.test(phone)) {
    return res.status(400).json({ error: "Invalid phone" });
  }
  if (!amount || amount < 1 || amount > 150000) {
    return res.status(400).json({ error: "Invalid amount" });
  }

  try {
    const result = await initiateStkPush({
      phone,
      amount,
      accountRef: "TEST",
      description: "Test payment",
    });
    res.json(result);
  } catch (err) {
    console.error(err.response?.data || err.message);
    res.status(500).json({ error: "STK failed" });
  }
});
```

Every line hand-typed.

**Expected output:**
POST /mpesa/stk works with a valid body.

### Task 4: First real STK test

**What to do:**
With your server and ngrok running, trigger the STK:

```bash
curl -X POST http://localhost:3001/mpesa/stk \
  -H "Content-Type: application/json" \
  -d '{"phone":"254708374149","amount":1}'
```

The sandbox test number `254708374149` will get an STK prompt. In the sandbox simulator (https://developer.safaricom.co.ke/), you can "accept" it with any 4-digit PIN.

**Expected output:**
Daraja returns a response with `ResponseCode: "0"` and `CheckoutRequestID`. Screenshot `day2-stk.png`.

### Task 5: Callback handler with idempotency

**What to do:**
Add the callback route:

```javascript
const processedCheckouts = new Set(); // in-memory for now

app.post("/mpesa/callback", (req, res) => {
  const callback = req.body.Body?.stkCallback;
  if (!callback) return res.status(400).end();

  const checkoutId = callback.CheckoutRequestID;
  if (processedCheckouts.has(checkoutId)) {
    console.log("Duplicate callback:", checkoutId);
    return res.json({ status: "already processed" });
  }

  const resultCode = callback.ResultCode;
  if (resultCode === 0) {
    const metadata = callback.CallbackMetadata?.Item || [];
    const amountItem = metadata.find((i) => i.Name === "Amount");
    const receiptItem = metadata.find((i) => i.Name === "MpesaReceiptNumber");
    const phoneItem = metadata.find((i) => i.Name === "PhoneNumber");

    // Amount validation -- hand-written
    const amountReceived = amountItem?.Value;
    // TODO: look up expected amount by CheckoutRequestID and compare
    console.log(`Payment received: ${amountReceived} from ${phoneItem?.Value}, ref ${receiptItem?.Value}`);

    processedCheckouts.add(checkoutId);
  } else {
    console.log(`Payment failed: code ${resultCode}`);
  }

  // Always return 200 so Daraja stops retrying
  res.json({ status: "ok" });
});
```

Note: this is a minimal version. Tomorrow you will persist to a real database and add proper amount validation against expected values.

**Expected output:**
When a payment completes, your Express logs show the callback details.

## Stretch Goals (Optional - Extra Credit)

- Cache the token for 55 minutes (tokens expire after 1 hour).
- Handle STK timeouts (when the user does not enter the PIN within 60 seconds).
- Add request-id tracing so you can correlate STK init with its callback.

## Submission Requirements

- **What to submit:** Repo with updated `services/mpesa.js`, `index.js`, screenshots, updated `AI_AUDIT.md`.
- **Deadline:** End of Day 2

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Token exchange by hand | 15 | Works. No AI in the code. |
| STK initiator by hand | 25 | All fields correct. ResponseCode 0 in tests. |
| Validation on /mpesa/stk | 15 | Phone and amount validated manually. Bad inputs rejected. |
| Callback handler with idempotency | 20 | Duplicate callbacks handled. ResultCode 0 vs non-zero branches. |
| First successful STK test | 15 | Screenshot confirms end-to-end loop. |
| Money code audit updated | 10 | Every line that touches money flagged as hand-written. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Hardcoding the public sandbox passkey in production code.** This key is public for testing only. In production you get a real one from Safaricom.
- **Returning 500 from the callback on errors.** Daraja retries. You should usually return 200 and log the error for later debugging.
- **Letting AI write the amount validation.** This is the single most important rule this week. Rewrite by hand if anything AI-touched slipped in.

## Resources

- Day 2 reading: [M-Pesa Integration and PDF Receipts.md](./M-Pesa%20Integration%20and%20PDF%20Receipts.md)
- Week 10 AI boundaries: [../ai.md](../ai.md)
- Daraja STK Push API: https://developer.safaricom.co.ke/APIs/MpesaExpressSimulate
- Sandbox test credentials: https://developer.safaricom.co.ke/Documentation
