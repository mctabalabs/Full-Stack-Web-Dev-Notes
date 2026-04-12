# Week 15, Day 5: Week Recap

This week turned the Week 14 catalogue into a shoppable system. You have a cart that persists across reloads, a checkout state machine that guards against invalid jumps, Server Actions that write orders transactionally, an admin dashboard with state-machine status transitions, and a customer order lookup. No real payments yet -- that is next week.

---

## What You Built

1. **Cart store** with Zustand, persisted to `localStorage`, derived totals, selector-based subscriptions.
2. **Checkout state machine** with five states and an allowed-transitions table.
3. **Three-step checkout UI** -- info, payment, confirmed.
4. **`createOrder` Server Action** with validation, transaction, `FOR UPDATE` locking, stock deduction, and server-side price recomputation.
5. **Admin login** via Server Action with HttpOnly cookies and `requireAdmin()` guards.
6. **Admin orders dashboard** with status filter, detail page, state-machine status updater.
7. **Order cancellation with restock** inside a transaction.
8. **Customer "my orders" lookup** by phone.

---

## Self-Review Questions

**Cart**
1. Why use Zustand instead of three `useState` hooks for the cart?
2. What does `persist` do and why do we need the `hydrated` flag?
3. Why are `totalItems` and `totalCents` functions instead of stored values?
4. Why does the header counter use a selector instead of the full store?

**Checkout state machine**
5. What five states does checkout have and what are the allowed transitions?
6. Why are invalid transitions rejected by `goTo` even though the UI already enforces them?
7. What happens when a user directly navigates to `/checkout/payment` without having filled in `/checkout/info`?
8. Why does the confirmed page reset the store?

**Server Actions**
9. What does `"use server"` mean at the top of a file?
10. How does a Client Component call a Server Action? What happens on the wire?
11. Why does `createOrder` recompute subtotals on the server instead of trusting the cart?
12. What does `FOR UPDATE` do in the product SELECT inside the transaction?
13. What does `revalidatePath` do, and why do we call it after an order?

**Admin and auth**
14. Why is the admin token stored in an HttpOnly cookie instead of `localStorage`?
15. What does `requireAdmin()` do when called at the top of a Server Component?
16. Where does the status-transitions table live (hint: two places), and why is that duplication fine?
17. Why is cancellation wrapped in a transaction?

Target: 14/17 correct.

---

## Phase 3 Midpoint

Phase 3 is "Commerce, Payments & Automation". We are at the midpoint:

| Phase 3 bullet | Where |
|---|---|
| Next.js: SSR vs client-side | Week 14 full |
| State machines: cart, checkout | Week 15 Days 1-2 |
| Project 3: E-commerce Lite + WhatsApp | Week 16 (next week) |
| Multi-provider payments: Stripe + M-Pesa + Airtel | Weeks 17-18 |
| Webhook security, idempotency, refunds | Week 18 |
| Payments Service module | Week 19 |
| Telegram bots | Week 20 |
| Cron jobs | Week 21 |
| Project 4: Chama Savings Platform | Week 22 |

Three weeks down, seven to go. Next week is the biggest -- Project 3 is the first end-to-end customer-facing product of the Marathon.

---

## Peer Coding Session

### Track A: A product admin form

Build `/admin/products/new` with a Server Action that inserts a new product. Fields: slug, name, description, price (as a decimal the user types, convert to cents server-side), category, stock. Add image upload later.

### Track B: Bulk status change

Add a checkbox column to `/admin/orders` and a bar at the top with "Mark selected as shipped". Use `useTransition` and a single Server Action that accepts an array of ids. This is a real-world admin pattern.

### Track C: Order analytics

Create `/admin/analytics` (admin only) with three numbers: total orders today, revenue this month, top-selling product this week. Three SQL queries, three numbers on a page. Bonus: add a small line chart with a client-only library like `recharts`.

### Track D: Restock history audit

After today's cancellation code, there is no record of *why* stock went up. Add an `inventory_events` table with `(product_id, delta, reason, created_at)` and write an event every time stock moves. Add a "History" section to the admin product detail page.

---

## Weekend Prep

The weekend project assembles Week 14 + Week 15 into a working demo you can show a friend. No real M-Pesa -- a fake pay button is fine. The grading rubric cares about completeness and correctness, not payment integration. Focus on:

- Catalogue is polished.
- Cart and checkout work reliably.
- Admin can see, update, and cancel orders.
- The flow feels seamless from browse to confirmation.
- Database state is always consistent (transactions everywhere).

Week 16 is the real M-Pesa integration. Do not try to ship it this weekend.
