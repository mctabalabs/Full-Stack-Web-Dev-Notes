# Week 10, Day 3: The React Frontend -- Bringing It All Together

By the end of today, you will have a complete, working application. A business owner can create payment links on a dashboard, share them with clients, and clients can pay via M-Pesa through a clean payment page. The app will detect completed payments in real time and offer a downloadable PDF receipt.

Everything you build today uses patterns you already know. Components, props, state, useEffect, controlled forms, React Router, API calls, conditional rendering, rendering lists with `.map()`. You learned all of these in Weeks 7-9. Today you assemble them into something real.

**Prior-week concepts you will use today:**
- Components, props, useState (Week 7)
- useEffect, controlled forms, lifting state up (Week 8)
- React Router: BrowserRouter, Routes, Route, useParams (Week 8)
- API service layer, fetching data, loading and error states (Week 9)
- Rendering lists with .map(), conditional rendering (Week 7)
- preventDefault and event handlers (Week 4)
- Arrow functions, destructuring, template literals (Week 3)

**Estimated time:** 3-4 hours

---

## Recap: What You Built on Days 1 and 2

On Day 1 you built an Express server with a SQLite database and REST API routes for creating and retrieving payment links.

On Day 2 you connected that server to Safaricom's Daraja API, implemented the STK push flow, handled webhook callbacks, and built PDF receipt generation.

Your backend is fully functional. Today you build the interface that users actually see and interact with. Two pages: a Dashboard for the business owner to create and manage payment links, and a Payment Page for clients to pay.

---

## React Project Setup

If you have not already created the React app, do it now from the `mpesa-paylink` root folder:

```bash
npx create-react-app client
cd client
npm install react-router-dom axios
```

You have used `create-react-app` since Week 7. The two additional packages:

- **react-router-dom** -- You learned this in Week 8. It gives you BrowserRouter, Routes, Route, useParams, and the other routing tools.
- **axios** -- A cleaner HTTP client than the browser's built-in `fetch()`. You used this in Week 9 to create a reusable API layer. Same idea here.

### Folder Structure

Organize the `client/src` folder like this:

```
client/src/
  App.js
  pages/
    Dashboard.js
    PaymentPage.js
  components/
    CreateLinkForm.js
    LinkList.js
  services/
    api.js
```

Remember from Week 8 when you learned about React folder structure patterns? The separation here follows the same logic:

- **pages/** -- Full-page components that map to routes. Dashboard is one page, PaymentPage is another.
- **components/** -- Reusable pieces of UI that live inside pages. CreateLinkForm and LinkList live inside the Dashboard page.
- **services/** -- Functions that talk to external systems (your backend API). No React code here, just plain JavaScript functions that make HTTP requests.

Create the folders:

```bash
mkdir src/pages src/components src/services
```

---

## The API Service Layer

Create `client/src/services/api.js`:

```javascript
import axios from "axios";

// Create an axios instance with the backend URL pre-configured
// This means we don't have to type "http://localhost:5000" in every function
const api = axios.create({
  baseURL: "http://localhost:5000/api",
});

// Create a new payment link
export async function createLink(data) {
  const response = await api.post("/links", data);
  return response.data;
}

// Get all payment links
export async function getLinks() {
  const response = await api.get("/links");
  return response.data;
}

// Get a single payment link by ID
export async function getLink(id) {
  const response = await api.get(`/links/${id}`);
  return response.data;
}

// Trigger an M-Pesa STK push for a payment link
export async function initiatePayment(linkId, phone) {
  const response = await api.post("/pay", { linkId, phone });
  return response.data;
}

// Check if a payment has been completed (used for polling)
export async function checkPaymentStatus(linkId) {
  const response = await api.get(`/payment-status/${linkId}`);
  return response.data;
}

// Get the URL to download a receipt PDF
export function getReceiptUrl(linkId) {
  return `http://localhost:5000/api/receipts/${linkId}`;
}
```

This is a direct application of what you learned in Week 9 about creating a reusable API layer. Remember when you created a fetch wrapper to avoid repeating the base URL and headers in every component? This is the same pattern.

`axios.create({ baseURL: "..." })` creates an instance where every request automatically prepends the base URL. When you call `api.get("/links")`, it actually sends a GET request to `http://localhost:5000/api/links`. If you deploy this app and change the backend URL, you change it in one place -- not in every component.

Every API function lives here, not scattered across components. The components import only the functions they need.

---

## App Router

Replace `client/src/App.js`:

```javascript
import { BrowserRouter, Routes, Route } from "react-router-dom";
import Dashboard from "./pages/Dashboard";
import PaymentPage from "./pages/PaymentPage";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Dashboard />} />
        <Route path="/pay/:linkId" element={<PaymentPage />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

You learned React Router in Week 8. This is the same BrowserRouter, Routes, and Route pattern. Two routes:

- `/` renders the Dashboard (for the business owner)
- `/pay/:linkId` renders the PaymentPage (for the client receiving the payment link)

The `:linkId` is a URL parameter. When someone opens `/pay/a3f8b2c1-7d4e-...`, React Router extracts `a3f8b2c1-7d4e-...` and makes it available through `useParams()`. You used this exact pattern in Week 8.

---

## The Dashboard Page

The Dashboard is what the business owner sees. It has two sections: a form to create new payment links, and a list of all existing links.

Create `client/src/pages/Dashboard.js`:

```javascript
import { useState, useEffect } from "react";
import { createLink, getLinks } from "../services/api";
import CreateLinkForm from "../components/CreateLinkForm";
import LinkList from "../components/LinkList";

function Dashboard() {
  const [links, setLinks] = useState([]);
  const [loading, setLoading] = useState(true);

  // Load all existing links when the component first mounts
  // This is the same useEffect-on-mount pattern from Week 8
  useEffect(() => {
    loadLinks();
  }, []); // Empty dependency array = run once on mount

  async function loadLinks() {
    try {
      const data = await getLinks();
      setLinks(data);
    } catch (err) {
      console.error("Failed to load links:", err);
    } finally {
      setLoading(false); // Stop showing "Loading..." whether it succeeded or failed
    }
  }

  // This function is passed as a prop to CreateLinkForm
  // When the form creates a new link, we add it to the top of our list
  async function handleCreateLink(formData) {
    const newLink = await createLink(formData);
    // Add the new link to the beginning of the array so it appears at the top
    setLinks((prev) => [newLink, ...prev]);
    return newLink;
  }

  return (
    <div style={styles.container}>
      <header style={styles.header}>
        <h1 style={styles.title}>Paylink</h1>
        <p style={styles.subtitle}>
          Create payment links. Get paid via M-Pesa. Instantly.
        </p>
      </header>

      <main style={styles.main}>
        <CreateLinkForm onSubmit={handleCreateLink} />

        <section style={styles.section}>
          <h2 style={styles.sectionTitle}>Your Links</h2>
          {loading ? (
            <p>Loading...</p>
          ) : (
            <LinkList links={links} />
          )}
        </section>
      </main>
    </div>
  );
}

const styles = {
  container: {
    maxWidth: 720,
    margin: "0 auto",
    padding: "40px 20px",
    fontFamily:
      '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif',
  },
  header: {
    marginBottom: 40,
  },
  title: {
    fontSize: 32,
    fontWeight: 700,
    color: "#153564",
    margin: 0,
  },
  subtitle: {
    fontSize: 16,
    color: "#666",
    marginTop: 8,
  },
  main: {
    display: "flex",
    flexDirection: "column",
    gap: 40,
  },
  section: {},
  sectionTitle: {
    fontSize: 18,
    fontWeight: 600,
    color: "#333",
    marginBottom: 16,
  },
};

export default Dashboard;
```

### Lifting State Up

The `links` state lives in Dashboard, not in LinkList or CreateLinkForm. Why? Because both child components need access to the same data. CreateLinkForm needs to add new links to the list. LinkList needs to display the list.

This is the "lifting state up" pattern from Week 8. When two sibling components need the same data, you move the state to their parent. The parent passes the data down as props and passes callback functions that children can use to update it.

Dashboard owns the `links` state. It passes `links` to LinkList for display. It passes `handleCreateLink` to CreateLinkForm as the `onSubmit` prop. When the form submits, it calls `onSubmit`, which calls `handleCreateLink`, which updates the `links` state, which causes both components to re-render with the new data.

### Loading State

The `loading` state starts as `true` and flips to `false` after the API call completes (in the `finally` block). While loading is true, we show "Loading..." instead of the link list. This is the same loading state pattern you practiced in Week 9 when fetching data from APIs.

---

## The CreateLinkForm Component

Create `client/src/components/CreateLinkForm.js`:

```javascript
import { useState } from "react";

function CreateLinkForm({ onSubmit }) {
  // A single state object for all form fields
  // An alternative is separate useState calls for each field
  // This approach is cleaner when you have many fields
  const [form, setForm] = useState({
    clientName: "",
    clientPhone: "",
    amount: "",
    description: "",
  });
  const [submitting, setSubmitting] = useState(false);
  const [createdLink, setCreatedLink] = useState(null);

  // One handler for all inputs, using the input's name attribute to update the right field
  function handleChange(e) {
    setForm((prev) => ({
      ...prev,                    // Keep all existing fields
      [e.target.name]: e.target.value, // Update only the field that changed
    }));
  }

  async function handleSubmit(e) {
    // Prevent the browser from refreshing the page on form submit
    // This is the same preventDefault from Week 4 DOM events
    e.preventDefault();
    setSubmitting(true);

    try {
      const link = await onSubmit({
        ...form,
        amount: parseFloat(form.amount), // Convert string to number
      });
      setCreatedLink(link);
      // Clear the form after successful creation
      setForm({ clientName: "", clientPhone: "", amount: "", description: "" });
    } catch (err) {
      alert("Failed to create link: " + (err.response?.data?.error || err.message));
    } finally {
      setSubmitting(false);
    }
  }

  function copyLink() {
    navigator.clipboard.writeText(createdLink.paymentUrl);
    alert("Link copied!");
  }

  return (
    <div>
      <form onSubmit={handleSubmit} style={styles.form}>
        <div style={styles.row}>
          <div style={styles.field}>
            <label style={styles.label}>Client Name</label>
            <input
              name="clientName"
              value={form.clientName}
              onChange={handleChange}
              required
              placeholder="e.g. John Kamau"
              style={styles.input}
            />
          </div>
          <div style={styles.field}>
            <label style={styles.label}>Phone Number</label>
            <input
              name="clientPhone"
              value={form.clientPhone}
              onChange={handleChange}
              required
              placeholder="0712 345 678"
              style={styles.input}
            />
          </div>
        </div>

        <div style={styles.row}>
          <div style={styles.field}>
            <label style={styles.label}>Amount (KES)</label>
            <input
              name="amount"
              type="number"
              min="1"
              value={form.amount}
              onChange={handleChange}
              required
              placeholder="500"
              style={styles.input}
            />
          </div>
          <div style={styles.field}>
            <label style={styles.label}>Description (optional)</label>
            <input
              name="description"
              value={form.description}
              onChange={handleChange}
              placeholder="e.g. Website design deposit"
              style={styles.input}
            />
          </div>
        </div>

        <button type="submit" disabled={submitting} style={styles.button}>
          {submitting ? "Creating..." : "Create Payment Link"}
        </button>
      </form>

      {createdLink && (
        <div style={styles.linkBox}>
          <p style={styles.linkLabel}>Share this link with your client:</p>
          <div style={styles.linkRow}>
            <code style={styles.linkUrl}>{createdLink.paymentUrl}</code>
            <button onClick={copyLink} style={styles.copyBtn}>
              Copy
            </button>
          </div>
        </div>
      )}
    </div>
  );
}

const styles = {
  form: {
    background: "#F8FAFC",
    border: "1px solid #E2E8F0",
    borderRadius: 12,
    padding: 24,
  },
  row: {
    display: "flex",
    gap: 16,
    marginBottom: 16,
  },
  field: {
    flex: 1,
  },
  label: {
    display: "block",
    fontSize: 13,
    fontWeight: 600,
    color: "#555",
    marginBottom: 6,
  },
  input: {
    width: "100%",
    padding: "10px 12px",
    border: "1px solid #D1D5DB",
    borderRadius: 8,
    fontSize: 14,
    boxSizing: "border-box",
    outline: "none",
  },
  button: {
    width: "100%",
    padding: "12px",
    background: "#153564",
    color: "white",
    border: "none",
    borderRadius: 8,
    fontSize: 15,
    fontWeight: 600,
    cursor: "pointer",
    marginTop: 8,
  },
  linkBox: {
    marginTop: 20,
    background: "#ECFDF5",
    border: "1px solid #A7F3D0",
    borderRadius: 8,
    padding: 16,
  },
  linkLabel: {
    fontSize: 13,
    color: "#065F46",
    margin: "0 0 8px 0",
  },
  linkRow: {
    display: "flex",
    gap: 8,
    alignItems: "center",
  },
  linkUrl: {
    flex: 1,
    fontSize: 13,
    background: "white",
    padding: "8px 12px",
    borderRadius: 6,
    border: "1px solid #D1FAE5",
    wordBreak: "break-all",
  },
  copyBtn: {
    padding: "8px 16px",
    background: "#059669",
    color: "white",
    border: "none",
    borderRadius: 6,
    fontSize: 13,
    cursor: "pointer",
    whiteSpace: "nowrap",
  },
};

export default CreateLinkForm;
```

### Controlled Form Pattern

Every input in this form has a `value` tied to state and an `onChange` that updates state. This is the controlled form pattern from Week 8. React owns the form data. When the user types, `handleChange` fires, updates the state, and React re-renders the input with the new value.

The `name` attribute on each input matches the key in the `form` state object. The `handleChange` function uses `[e.target.name]` (a computed property name from Week 3) to update the right field. This is cleaner than writing a separate handler for each input.

### preventDefault

`e.preventDefault()` in `handleSubmit` stops the browser from refreshing the page when the form submits. This is the same `preventDefault` you learned in Week 4 when handling DOM events. Without it, the browser would do a full page reload and you would lose all your React state.

### After Successful Creation

When the API call succeeds, we do two things: save the new link to `createdLink` state (which shows the green success box with the payment URL), and clear the form fields back to empty strings. The `copyLink` function uses `navigator.clipboard.writeText()` to copy the payment URL to the clipboard.

---

## The LinkList Component

Create `client/src/components/LinkList.js`:

```javascript
import { getReceiptUrl } from "../services/api";

function LinkList({ links }) {
  // Empty state: no links created yet
  if (links.length === 0) {
    return (
      <p style={{ color: "#999", fontSize: 14 }}>
        No payment links yet. Create your first one above.
      </p>
    );
  }

  return (
    <div style={styles.list}>
      {links.map((link) => (
        <div key={link.id} style={styles.card}>
          <div style={styles.cardTop}>
            <div>
              <p style={styles.clientName}>{link.client_name}</p>
              <p style={styles.description}>
                {link.description || "No description"}
              </p>
            </div>
            <div style={styles.right}>
              <p style={styles.amount}>
                KES {Number(link.amount).toLocaleString()}
              </p>
              {/* Dynamic badge color based on payment status */}
              <span
                style={{
                  ...styles.badge,
                  background: link.status === "paid" ? "#D1FAE5" : "#FEF3C7",
                  color: link.status === "paid" ? "#065F46" : "#92400E",
                }}
              >
                {link.status}
              </span>
            </div>
          </div>

          <div style={styles.cardBottom}>
            <span style={styles.date}>
              {new Date(link.created_at).toLocaleDateString()}
            </span>
            {/* Only show the download link if the payment is completed */}
            {link.status === "paid" && (
              <a
                href={getReceiptUrl(link.id)}
                style={styles.receiptLink}
                target="_blank"
                rel="noreferrer"
              >
                Download Receipt
              </a>
            )}
          </div>
        </div>
      ))}
    </div>
  );
}

const styles = {
  list: {
    display: "flex",
    flexDirection: "column",
    gap: 12,
  },
  card: {
    background: "white",
    border: "1px solid #E5E7EB",
    borderRadius: 10,
    padding: 16,
  },
  cardTop: {
    display: "flex",
    justifyContent: "space-between",
    alignItems: "flex-start",
  },
  clientName: {
    fontSize: 15,
    fontWeight: 600,
    color: "#111",
    margin: 0,
  },
  description: {
    fontSize: 13,
    color: "#888",
    margin: "4px 0 0",
  },
  right: {
    textAlign: "right",
  },
  amount: {
    fontSize: 16,
    fontWeight: 700,
    color: "#153564",
    margin: 0,
  },
  badge: {
    display: "inline-block",
    fontSize: 11,
    fontWeight: 600,
    padding: "3px 10px",
    borderRadius: 20,
    marginTop: 6,
    textTransform: "uppercase",
  },
  cardBottom: {
    display: "flex",
    justifyContent: "space-between",
    alignItems: "center",
    marginTop: 12,
    paddingTop: 12,
    borderTop: "1px solid #F3F4F6",
  },
  date: {
    fontSize: 12,
    color: "#AAA",
  },
  receiptLink: {
    fontSize: 13,
    color: "#FF6600",
    fontWeight: 600,
    textDecoration: "none",
  },
};

export default LinkList;
```

### Rendering Lists with .map()

This is the same `.map()` pattern you have been using since Week 7. The `links` array comes in as a prop. We call `.map()` on it, and for each link object, we return a card component. The `key={link.id}` prop tells React how to track each item in the list -- without it, React cannot efficiently update the DOM when items change. You learned this in Week 7.

### Conditional Rendering

Two examples of conditional rendering here, both patterns from Week 7:

1. `link.description || "No description"` -- the `||` operator shows a fallback if description is empty
2. `{link.status === "paid" && (<a ...>Download Receipt</a>)}` -- the `&&` operator renders the download link ONLY when the status is "paid". If the status is "pending", this expression evaluates to `false`, and React renders nothing.

The badge color also changes conditionally using a ternary operator in the style object. Green background for "paid", amber for "pending".

---

## The Payment Page

This is what your client sees when they open the payment link. It is a standalone page -- clean, focused, with no dashboard elements. Just the amount, a phone input, and a pay button.

Create `client/src/pages/PaymentPage.js`:

```javascript
import { useState, useEffect, useRef } from "react";
import { useParams } from "react-router-dom";
import {
  getLink,
  initiatePayment,
  checkPaymentStatus,
  getReceiptUrl,
} from "../services/api";

function PaymentPage() {
  // Extract the linkId from the URL using useParams (Week 8 React Router)
  const { linkId } = useParams();

  const [link, setLink] = useState(null);
  const [phone, setPhone] = useState("");
  const [error, setError] = useState("");

  // Status tracks where we are in the payment flow:
  // "idle" -- waiting for user to click Pay
  // "processing" -- STK push request sent, waiting for Safaricom to respond
  // "polling" -- STK push confirmed sent, now polling for payment completion
  // "paid" -- payment confirmed
  // "failed" -- something went wrong
  const [status, setStatus] = useState("idle");

  // useRef stores the polling interval ID without causing re-renders
  // useState would trigger a re-render every time we store the interval ID,
  // which we don't need -- we just need to remember it so we can clear it later
  const pollingRef = useRef(null);

  // Load the payment link details when the component mounts
  useEffect(() => {
    loadLink();

    // Cleanup: stop polling if the user navigates away before payment completes
    // This is the useEffect cleanup function from Week 8
    return () => {
      if (pollingRef.current) clearInterval(pollingRef.current);
    };
  }, [linkId]);

  async function loadLink() {
    try {
      const data = await getLink(linkId);
      setLink(data);
      // Pre-fill the phone input with the number stored in the link
      setPhone(formatInputPhone(data.client_phone));

      // If this link was already paid, show the success state immediately
      if (data.status === "paid") {
        setStatus("paid");
      }
    } catch (err) {
      setError("This payment link is invalid or has expired.");
    }
  }

  async function handlePay() {
    setStatus("processing");
    setError("");

    try {
      // Call our backend to trigger the STK push
      await initiatePayment(linkId, phone);
      setStatus("polling");

      // Now poll the backend every 3 seconds to check if the payment completed
      // The backend receives the callback from Safaricom independently,
      // so we need to keep checking until the status changes
      let attempts = 0;
      const maxAttempts = 40; // 40 attempts x 3 seconds = 2 minutes max

      pollingRef.current = setInterval(async () => {
        attempts++;

        try {
          const result = await checkPaymentStatus(linkId);

          if (result.linkStatus === "paid") {
            // Payment confirmed! Stop polling and show success
            clearInterval(pollingRef.current);
            setStatus("paid");
          } else if (attempts >= maxAttempts) {
            // Timed out after 2 minutes
            clearInterval(pollingRef.current);
            setStatus("idle");
            setError(
              "Payment timed out. If you completed the payment, refresh the page."
            );
          }
        } catch {
          // If a single poll request fails, keep trying -- don't stop the whole flow
        }
      }, 3000);
    } catch (err) {
      setStatus("idle");
      setError(
        err.response?.data?.error || "Failed to initiate payment. Try again."
      );
    }
  }

  // Convert the stored 254XXXXXXXXX format back to 0XXX XXX XXX for display
  function formatInputPhone(p) {
    if (!p) return "";
    const s = String(p);
    if (s.startsWith("254")) {
      return "0" + s.slice(3);
    }
    return s;
  }

  // Error state: invalid or expired link
  if (error && !link) {
    return (
      <div style={styles.container}>
        <div style={styles.errorCard}>
          <p>{error}</p>
        </div>
      </div>
    );
  }

  // Loading state
  if (!link) {
    return (
      <div style={styles.container}>
        <p>Loading...</p>
      </div>
    );
  }

  return (
    <div style={styles.container}>
      <div style={styles.card}>
        {/* Header: shows the amount and payment details */}
        <div style={styles.cardHeader}>
          <p style={styles.label}>Payment Request</p>
          <h1 style={styles.amount}>
            KES {Number(link.amount).toLocaleString()}
          </h1>
          {link.description && (
            <p style={styles.description}>{link.description}</p>
          )}
          <p style={styles.meta}>To: {link.client_name}</p>
        </div>

        {/* Body: either the payment form or the success message */}
        <div style={styles.cardBody}>
          {status === "paid" ? (
            <div style={styles.successBox}>
              <div style={styles.checkmark}>&#10003;</div>
              <h2 style={styles.successTitle}>Payment Successful</h2>
              <p style={styles.successText}>
                Your M-Pesa payment has been confirmed.
              </p>
              <a
                href={getReceiptUrl(linkId)}
                style={styles.downloadBtn}
                target="_blank"
                rel="noreferrer"
              >
                Download Receipt
              </a>
            </div>
          ) : (
            <>
              <label style={styles.inputLabel}>M-Pesa Phone Number</label>
              <input
                type="tel"
                value={phone}
                onChange={(e) => setPhone(e.target.value)}
                placeholder="0712 345 678"
                style={styles.input}
                disabled={status !== "idle"}
              />

              {error && <p style={styles.errorText}>{error}</p>}

              <button
                onClick={handlePay}
                disabled={status !== "idle" || !phone}
                style={{
                  ...styles.payBtn,
                  opacity: status !== "idle" ? 0.6 : 1,
                }}
              >
                {status === "processing"
                  ? "Sending STK Push..."
                  : status === "polling"
                  ? "Waiting for confirmation..."
                  : `Pay KES ${Number(link.amount).toLocaleString()}`}
              </button>

              {status === "polling" && (
                <p style={styles.pollingText}>
                  Check your phone for the M-Pesa prompt. Enter your PIN to
                  complete the payment.
                </p>
              )}
            </>
          )}
        </div>
      </div>

      <p style={styles.footer}>Secured by M-Pesa &bull; Powered by Paylink</p>
    </div>
  );
}

const styles = {
  container: {
    minHeight: "100vh",
    display: "flex",
    flexDirection: "column",
    alignItems: "center",
    justifyContent: "center",
    background: "#F1F5F9",
    padding: 20,
    fontFamily:
      '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif',
  },
  card: {
    background: "white",
    borderRadius: 16,
    boxShadow: "0 4px 24px rgba(0,0,0,0.08)",
    width: "100%",
    maxWidth: 420,
    overflow: "hidden",
  },
  cardHeader: {
    background: "#153564",
    color: "white",
    padding: "32px 24px",
    textAlign: "center",
  },
  label: {
    fontSize: 13,
    opacity: 0.7,
    margin: "0 0 8px",
    textTransform: "uppercase",
    letterSpacing: 1,
  },
  amount: {
    fontSize: 36,
    fontWeight: 700,
    margin: "0 0 8px",
  },
  description: {
    fontSize: 14,
    opacity: 0.8,
    margin: "0 0 4px",
  },
  meta: {
    fontSize: 13,
    opacity: 0.6,
    margin: 0,
  },
  cardBody: {
    padding: 24,
  },
  inputLabel: {
    display: "block",
    fontSize: 13,
    fontWeight: 600,
    color: "#555",
    marginBottom: 8,
  },
  input: {
    width: "100%",
    padding: "12px 14px",
    border: "1px solid #D1D5DB",
    borderRadius: 10,
    fontSize: 16,
    boxSizing: "border-box",
    marginBottom: 16,
    outline: "none",
  },
  payBtn: {
    width: "100%",
    padding: 14,
    background: "#22C55E",
    color: "white",
    border: "none",
    borderRadius: 10,
    fontSize: 16,
    fontWeight: 700,
    cursor: "pointer",
  },
  pollingText: {
    fontSize: 13,
    color: "#666",
    textAlign: "center",
    marginTop: 16,
    lineHeight: 1.5,
  },
  errorText: {
    fontSize: 13,
    color: "#DC2626",
    marginBottom: 12,
  },
  successBox: {
    textAlign: "center",
    padding: "20px 0",
  },
  checkmark: {
    width: 56,
    height: 56,
    borderRadius: "50%",
    background: "#D1FAE5",
    color: "#059669",
    fontSize: 28,
    display: "flex",
    alignItems: "center",
    justifyContent: "center",
    margin: "0 auto 16px",
  },
  successTitle: {
    fontSize: 20,
    fontWeight: 700,
    color: "#065F46",
    margin: "0 0 8px",
  },
  successText: {
    fontSize: 14,
    color: "#666",
    margin: "0 0 20px",
  },
  downloadBtn: {
    display: "inline-block",
    padding: "10px 24px",
    background: "#FF6600",
    color: "white",
    borderRadius: 8,
    fontSize: 14,
    fontWeight: 600,
    textDecoration: "none",
  },
  errorCard: {
    background: "#FEF2F2",
    border: "1px solid #FECACA",
    borderRadius: 12,
    padding: 24,
    maxWidth: 400,
    textAlign: "center",
    color: "#991B1B",
  },
  footer: {
    fontSize: 12,
    color: "#999",
    marginTop: 24,
  },
};

export default PaymentPage;
```

### The Status State Machine

Instead of using multiple boolean flags (`isLoading`, `isPaid`, `isError`, `isPolling`), this component uses a single `status` string that moves through well-defined states:

```
idle --> processing --> polling --> paid
                   \-> idle (on error)
          polling ---> idle (on timeout)
```

This is cleaner than booleans for two reasons. First, it is impossible to be in two states at once. With booleans, you could accidentally set `isLoading = true` AND `isPaid = true`, which makes no sense. With a single status string, the component is always in exactly one state. Second, the button text and UI behavior are determined by a single variable -- no complex combinations of `if (isLoading && !isPaid && !isError)`.

This is a new pattern. Think of it as an upgrade to the loading/error state pattern you learned in Week 9. Instead of two booleans, one string tracks the entire flow.

### The Polling Pattern

After the STK push is sent, the frontend needs to detect when the payment completes. The payment confirmation comes through a webhook from Safaricom to your backend. The frontend is not involved in that exchange. So the frontend polls -- it calls `checkPaymentStatus` every 3 seconds and checks if the status changed to "paid".

```javascript
pollingRef.current = setInterval(async () => {
  // Check every 3 seconds
  const result = await checkPaymentStatus(linkId);
  if (result.linkStatus === "paid") {
    clearInterval(pollingRef.current);
    setStatus("paid");
  }
}, 3000);
```

`setInterval` calls a function repeatedly at a fixed interval (3000 milliseconds = 3 seconds). It returns an interval ID that we store in `pollingRef` so we can stop it later with `clearInterval`.

### Why useRef Instead of useState?

We store the interval ID in `useRef`, not `useState`. The reason: `useState` triggers a re-render every time its value changes. We do not need the UI to update when the interval ID is stored. We just need to remember it so we can call `clearInterval` later. `useRef` gives us a mutable container that persists across renders without triggering them.

Think of `useRef` as a box you can put a value in and take it out later, without React caring. `useState` is a box that React watches -- every time you put something new in, React re-renders the component.

### Cleanup on Unmount

```javascript
useEffect(() => {
  loadLink();
  return () => {
    if (pollingRef.current) clearInterval(pollingRef.current);
  };
}, [linkId]);
```

The function returned from `useEffect` runs when the component unmounts (when the user navigates to a different page). This is the cleanup pattern from Week 8. If the user navigates away while polling is active, we stop the interval. Without this, the interval would keep running in the background, making API calls to a component that no longer exists -- a memory leak.

---

## Running the Full Application

### Start All Three Services

You need three terminals running simultaneously.

Terminal 1 -- Backend:
```bash
cd server
npm run dev
```

Terminal 2 -- Frontend:
```bash
cd client
npm start
```

Terminal 3 -- ngrok tunnel:
```bash
ngrok http 5000
```

After ngrok starts, copy the `https://` URL, update `MPESA_CALLBACK_URL` in `server/.env`, and restart the backend server.

### Test the Full Flow

1. Open `http://localhost:3000` in your browser. You should see the Paylink dashboard with the form and an empty links list.

2. Fill in the form: Client Name: "Aisha Wanjiku", Phone: "0712 345 678", Amount: 2500, Description: "Catering deposit for Friday event". Click "Create Payment Link".

3. You should see a green box with the payment URL. Click "Copy" to copy it.

4. Open that URL in a new browser tab. You should see the payment page with the amount (KES 2,500) and the phone number pre-filled.

5. Click the green "Pay" button. The button text changes to "Sending STK Push..." and then "Waiting for confirmation..."

6. If you are using the Daraja sandbox, the STK push simulates on the test phone number. Check your server terminal for the callback log.

7. Once the callback arrives, the payment page updates to show a green checkmark and "Payment Successful" with a "Download Receipt" button.

8. Click "Download Receipt" to download the PDF. Open it and verify it shows the correct details.

9. Go back to the Dashboard tab and refresh the page. The link should now show a green "paid" badge and a "Download Receipt" link.

---

## What You Have Built

Step back and look at what you created this week.

You built a **full-stack application** with a React frontend and a Node.js/Express backend. The frontend talks to the backend through a REST API. The backend talks to Safaricom's Daraja API to process real payments. Safaricom talks back through a webhook. Your backend generates PDF documents.

This is not a tutorial project. This is the architecture of real fintech applications. The patterns you used -- REST APIs, OAuth authentication, webhooks, async payment flows, PDF generation, state machines, polling -- these are production patterns used by companies processing millions of transactions.

Here is every concept from the bootcamp that you applied:

| Concept | Where You Learned It | Where You Used It |
|---|---|---|
| HTML forms, inputs, buttons | Weeks 1-2 | CreateLinkForm, PaymentPage |
| CSS layout (Flexbox) | Weeks 1-2 | Every component's inline styles |
| Arrow functions | Week 3 | Every file |
| Destructuring | Week 3 | Route handlers, component props |
| Template literals | Week 3 | API URLs, receipt text |
| Array .map() | Week 3 | LinkList rendering |
| Event handling, preventDefault | Week 4 | Form submission |
| Promises, async/await | Week 5 | API calls, M-Pesa integration |
| try/catch error handling | Week 5 | Every async function |
| JSON | Week 5 | API requests and responses |
| Fetch/HTTP requests | Week 5 | axios calls (same concept, different library) |
| npm, package.json | Week 6 | Project setup |
| .gitignore | Week 6 | Protecting secrets |
| Components and props | Week 7 | Every React file |
| useState | Week 7 | Forms, loading states, link data |
| Conditional rendering | Week 7 | Status badges, success/error states |
| Rendering lists with .map() and key | Week 7 | LinkList |
| useEffect | Week 8 | Data fetching on mount |
| Controlled forms | Week 8 | CreateLinkForm, PaymentPage phone input |
| Lifting state up | Week 8 | Dashboard owning links state |
| React Router, useParams | Week 8 | App routing, PaymentPage |
| useEffect cleanup | Week 8 | Polling interval cleanup |
| API service layer | Week 9 | services/api.js |
| Loading and error states | Week 9 | Dashboard, PaymentPage |

You already knew every piece. The capstone assembled them into something real.

---

## Common Errors

| Error | Cause | Fix |
|---|---|---|
| CORS error in browser console | Backend not running, or CORS middleware not applied | Make sure the Express server is running on port 5000 and `app.use(cors())` appears before your routes in index.js |
| `Network Error` from axios | Backend server is not running | Start the backend with `npm run dev` in the server folder |
| Payment page shows "Loading..." forever | The linkId in the URL does not match any link in the database | Check that the URL is correct. Try creating a new link and using its URL. |
| STK push sent but page never updates to "paid" | Callback URL is wrong or ngrok is not running | Check that ngrok is running, the URL in .env matches, and the server was restarted after updating .env |
| Form submits but nothing happens | The onSubmit prop is not connected | Make sure Dashboard passes `handleCreateLink` as the `onSubmit` prop to CreateLinkForm |
| `Cannot read properties of undefined (reading 'data')` | The API response shape is different than expected | Check the Network tab in browser DevTools to see the actual response from your backend |
| Page shows blank white screen | A JavaScript error in a component | Open browser DevTools (F12), check the Console tab for error messages |

---

## Stretch Goals

If you want to push further, here are extensions that build on what you have:

**WhatsApp sharing.** After creating a payment link, generate a `wa.me` URL pre-filled with a message containing the payment link. This is how businesses in Kenya actually share payment links. Example: `https://wa.me/254712345678?text=Hi%20John,%20here%20is%20your%20payment%20link:%20...`

**Email receipts.** After generating the PDF, use the Nodemailer package to email the receipt to the client automatically.

**Payment link expiry.** Add an `expires_at` column to the links table. Set it to 24 hours after creation. Check it before allowing payment. Show an "Expired" message on the payment page if the link is past its expiry.

**WebSocket updates.** Replace the polling pattern with Socket.io so the payment page updates instantly when the callback arrives. No more 3-second delay.

**CSV export.** Add a "Download CSV" button to the dashboard that exports all transactions as a CSV file. Business owners need this for their records.

**Deployment.** Deploy the backend to Railway or Render, the frontend to Vercel, and switch from Daraja sandbox to production credentials. Then you have a real product, live on the internet, processing real M-Pesa payments.
