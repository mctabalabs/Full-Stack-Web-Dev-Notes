# Week 17 Weekend: Payments Harness

A lighter weekend after Project 3. Build an admin-only "Payments Harness" page where you can initiate test payments to each provider, watch the full round-trip, and verify your unified interface is truly unified.

**Estimated time:** 4-5 hours.

**Deadline:** Monday morning before Week 18.

---

## The Goal

An admin page at `/admin/payments-harness` that shows:

1. Three cards, one per provider (M-Pesa, Airtel, Stripe).
2. Each card has a form to initiate a test payment (amount in KSh, phone for mobile money, email for Stripe).
3. After initiating, the card shows the current status, the reference id, and a "Refresh status" button.
4. A log panel shows every request/response with that provider in the current session (client-side only, not persisted).
5. A "Compare" view that shows all three providers' responses side by side for visual inspection.

---

## Functional Requirements

- Must use the unified `initiate` / `queryStatus` endpoints from Week 17 Day 3. No provider-specific code in the harness.
- Must show the provider's full response (reference id, next step, metadata).
- Must live-update the status when a callback arrives (poll every 2 seconds until terminal state).
- Must refuse to run unless the admin is logged in.
- Must not touch real orders or the real orders table. Use a dummy "test order" with a specific flag.
- Must work with all three providers on their sandboxes.

---

## Schema Addition

Add a "test mode" flag to orders so the harness can distinguish test payments from real ones:

```sql
ALTER TABLE orders ADD COLUMN is_test BOOLEAN NOT NULL DEFAULT FALSE;
CREATE INDEX idx_orders_is_test ON orders(is_test) WHERE is_test = TRUE;
```

The harness creates orders with `is_test = true`. The regular customer-facing shop never creates test orders. The admin dashboard filters them out by default.

---

## Requirements Checklist

- [ ] `/admin/payments-harness` exists and is admin-protected.
- [ ] Initiating an M-Pesa payment from the harness triggers a sandbox STK prompt.
- [ ] Initiating an Airtel payment triggers a sandbox USSD prompt.
- [ ] Initiating a Stripe payment returns a checkout URL you can open in a new tab.
- [ ] Each card shows the status updating live.
- [ ] Test orders have `is_test = true` in the database.
- [ ] The regular `/admin/orders` page does NOT show test orders (unless a checkbox is ticked).
- [ ] The log panel shows the raw request and response for each call.
- [ ] The "Compare" view renders all three cards in one row.
- [ ] README has a GIF or screenshot of all three payments running simultaneously.

---

## Grading Rubric (50 pts -- half weight this weekend)

| Area | Points |
|---|---|
| All three providers work end-to-end | 20 |
| Test mode correctly isolated from production orders | 10 |
| Live status polling without page reloads | 10 |
| Admin auth + UI polish | 5 |
| README + demo artefacts | 5 |

---

## Hints

**Do not rewrite the interface.** Reuse everything from Day 3. If you find yourself editing a provider file to support the harness, stop -- you are doing it wrong. The harness should be a pure client of the unified API.

**Sandbox accounts matter.** Make sure your Airtel sandbox is actually working before Sunday. Airtel sandbox is the most fragile of the three; sign up early and test early.

**Do not chase prettiness.** Three rectangles and a log pane. No fancy charts. The point is "prove the abstraction works", not "win a design award".

**Screen recording beats screenshots.** A 20-second screen recording of all three providers lighting up in parallel is more convincing than any amount of text. QuickTime on Mac, OBS on Linux.

---

## If You Finish Early

- **Chaos button.** A "Simulate outage" button that marks a provider as unhealthy in Redis for 5 minutes. Use it to verify your Week 17 Day 4 fallbacks.
- **Provider latency stats.** Log how long each provider's `initiate` call takes; show a rolling average.
- **Minimal Swagger-like docs** on the harness page explaining the unified interface shape.

Good luck. Next week we harden the whole payments layer for real production use.
