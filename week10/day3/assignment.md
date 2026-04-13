# Week 10 - Day 3 Assignment

## Title
React Frontend for the Paylink and PDF Receipt Generation

## Overview
Today you build the React frontend that triggers your Day 2 STK Push endpoint, then generate a PDF receipt for successful payments. The UI is AI-assisted (repeated React pattern). The PDF generation and the amount math are manual.

## Learning Objectives Assessed
- Build a React form that POSTs to your Express backend
- Handle the three UI states during a payment (idle, pending, done)
- Generate a PDF with pdfkit
- Display a receipt on success with the real Daraja reference

## Prerequisites
- Days 1-2 completed
- Your Express backend is running and reachable via ngrok

## AI Usage Rules

**Ratio:** 50/50. **Habit:** NEVER trust AI with money. See [../ai.md](../ai.md).

- **ALLOWED FOR:** React form scaffolding, pdfkit styling, UI polish. Repeated patterns you already know.
- **NOT ALLOWED FOR:** Amount validation, the receipt's amount field, idempotency logic.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: React form for paylink

**What to do:**
In a new React project (or reuse your Week 7 `react-lab`), create `PayForm.jsx`:

```jsx
import { useState } from "react";

function PayForm() {
  const [phone, setPhone] = useState("");
  const [amount, setAmount] = useState("");
  const [status, setStatus] = useState("idle"); // idle | pending | done | error
  const [message, setMessage] = useState("");

  async function handlePay(e) {
    e.preventDefault();
    setStatus("pending");
    setMessage("Check your phone for the M-Pesa prompt...");

    try {
      const res = await fetch("http://localhost:3001/mpesa/stk", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ phone, amount: parseInt(amount, 10) }),
      });
      const data = await res.json();

      if (data.ResponseCode === "0") {
        setMessage(`STK sent. Checkout ID: ${data.CheckoutRequestID}`);
        // TODO Day 4: poll for completion
      } else {
        setStatus("error");
        setMessage(data.error || "Failed to send STK");
      }
    } catch (err) {
      setStatus("error");
      setMessage(err.message);
    }
  }

  return (
    <form onSubmit={handlePay}>
      <input
        type="tel"
        value={phone}
        onChange={(e) => setPhone(e.target.value)}
        placeholder="254..."
        required
      />
      <input
        type="number"
        value={amount}
        onChange={(e) => setAmount(e.target.value)}
        placeholder="Amount"
        required
      />
      <button type="submit" disabled={status === "pending"}>
        {status === "pending" ? "Sending..." : "Pay with M-Pesa"}
      </button>
      <p>{message}</p>
    </form>
  );
}

export default PayForm;
```

The form itself is AI-assisted territory (repeated React pattern). Use AI if it helps.

**Expected output:**
Clicking Pay sends a request to your backend and triggers an STK prompt.

### Task 2: Install pdfkit on the backend

**What to do:**
```bash
cd payments-backend
npm install pdfkit
```

Create `services/receipt.js`:

```javascript
const PDFDocument = require("pdfkit");

function generateReceipt({ phone, amount, reference, date }) {
  const doc = new PDFDocument();
  const chunks = [];

  doc.on("data", (chunk) => chunks.push(chunk));

  return new Promise((resolve, reject) => {
    doc.on("end", () => resolve(Buffer.concat(chunks)));
    doc.on("error", reject);

    doc.fontSize(20).text("Mctaba Labs Receipt", { align: "center" });
    doc.moveDown();
    doc.fontSize(14);
    doc.text(`Phone: ${phone}`);
    doc.text(`Amount: KES ${amount}`);  // amount hand-coded, no AI
    doc.text(`Reference: ${reference}`);
    doc.text(`Date: ${date}`);
    doc.moveDown();
    doc.fontSize(12).text("Thank you for your payment.", { align: "center" });

    doc.end();
  });
}

module.exports = { generateReceipt };
```

Write the PDF generator by hand. Amount and reference are money fields -- no AI.

**Expected output:**
`services/receipt.js` created.

### Task 3: Expose a receipt endpoint

**What to do:**
```javascript
const { generateReceipt } = require("./services/receipt");

app.get("/mpesa/receipt/:checkoutId", async (req, res) => {
  // TODO Day 4: look up the real payment from a store
  // For now, generate a fake receipt for testing
  const pdf = await generateReceipt({
    phone: "254708374149",
    amount: 100,
    reference: req.params.checkoutId,
    date: new Date().toISOString(),
  });

  res.setHeader("Content-Type", "application/pdf");
  res.setHeader("Content-Disposition", `inline; filename="receipt-${req.params.checkoutId}.pdf"`);
  res.send(pdf);
});
```

Visit `http://localhost:3001/mpesa/receipt/test123` in your browser. A PDF downloads or displays.

**Expected output:**
PDF receipt renders in the browser. Screenshot `day3-receipt.png`.

### Task 4: Connect the frontend to the receipt

**What to do:**
After a successful STK init, let the frontend show a "View receipt" link pointing to `http://localhost:3001/mpesa/receipt/${checkoutId}` (it will 404 gracefully if the payment has not completed -- that is tomorrow's polish).

**Expected output:**
The React form now includes a clickable link to view the receipt.

### Task 5: Update the money code audit

**What to do:**
Add a new entry to `AI_AUDIT.md`:
- `services/receipt.js` -- amount and reference lines -- hand-written
- `services/mpesa.js` -- token, initiator, callback -- hand-written
- Frontend form -- AI-assisted (repeated pattern, UI only)

For every money-related line, confirm the author. This is the most important audit entry of the week.

**Expected output:**
Updated audit.

## Stretch Goals (Optional - Extra Credit)

- Add a QR code to the receipt PDF encoding the reference.
- Save the PDF to disk as well as returning it (for your own records).
- Email the receipt automatically after a successful payment (you would need nodemailer -- stretch goal only).

## Submission Requirements

- **What to submit:** Repo with backend and frontend, screenshots, `AI_AUDIT.md`.
- **Deadline:** End of Day 3

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| React PayForm with three states | 20 | Idle, pending, done all visible. |
| PDF receipt generation | 25 | pdfkit generates valid PDF with amount and reference. |
| /mpesa/receipt endpoint | 15 | Returns PDF with correct content-type. |
| Frontend-backend wired | 15 | Pay flow works end to end in dev. |
| Money code audit updated | 20 | Every money line flagged, all hand-written. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Trusting AI to set the amount in the PDF.** Rewrite any AI-generated receipt line yourself. Amounts shown on receipts are contractually binding.
- **Letting the frontend control the amount displayed.** The PDF amount must come from the backend's trusted store, not from a URL parameter.
- **Forgetting Content-Type on the PDF response.** Without it, browsers will not render the PDF inline.

## Resources

- Day 3 reading: [The React Frontend.md](./The%20React%20Frontend.md)
- Week 10 AI boundaries: [../ai.md](../ai.md)
- pdfkit docs: http://pdfkit.org/docs/getting_started.html
