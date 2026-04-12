# Week 15 Weekend Project: Shop MVP -- Browse to Confirmation

Assemble Week 14 and Week 15 into a working shop. Customer browses, adds to cart, checks out, pays (fake button), sees confirmation. Admin logs in, sees orders, moves them through states. No real payments -- that is Week 16.

**Estimated time:** 6-8 hours.

**Deadline:** Monday morning before Week 16.

---

## Requirements

### Customer flow
- Browse at `/products` and `/products/category/[cat]`.
- See product detail at `/products/[slug]`.
- Add to cart, cart persists.
- Checkout: info -> payment -> confirmed.
- Cannot jump checkout steps.
- Confirmation page shows the order id, then reset.
- `/my-orders?phone=...` shows past orders.

### Admin flow
- Log in at `/admin/login`.
- See orders list at `/admin/orders` with status filter.
- Click an order for detail.
- Update status via the state machine.
- Cancellation restocks inventory.
- Log out.

### Technical
- Every mutation goes through a Server Action.
- Every mutation writes through a transaction where it touches more than one row.
- Every status change respects the transition table.
- Stock is never negative (use `CHECK` constraints).
- Subtotals are computed server-side, never trusted from client.
- Admin token is an HttpOnly cookie.
- No API routes except the webhook routes inherited from the CRM project.

### Content
- 15-20 real products, at least 4 categories.
- At least 3 test admin users.
- Seed script that creates all of the above in one command.

---

## Grading Rubric (100 pts)

| Area | Points |
|---|---|
| Browse + cart works smoothly | 15 |
| Checkout state machine enforces order | 15 |
| createOrder uses a real transaction with FOR UPDATE | 20 |
| Admin auth with HttpOnly cookies | 10 |
| Admin order management with status transitions | 15 |
| Cancellation restocks inventory | 10 |
| Race condition test passes | 5 |
| Seed script + README | 5 |
| No prices ever computed on the client | 5 |

---

## Submission Checklist

- [ ] `npm run db:seed` creates admins + products + sample orders.
- [ ] Browsing flow works end to end.
- [ ] Two tabs opening simultaneously and buying the last unit: one succeeds, one fails.
- [ ] Admin login -> orders dashboard -> order detail -> status update -> cancel/restock all work.
- [ ] Stock on the product list matches actual `SELECT stock FROM products`.
- [ ] No `console.error` lines in a clean run-through.
- [ ] README has a "How to test the full flow" section.

---

## Hints

**Do not add M-Pesa this weekend.** Resist. A half-working payment integration will not help your grade. A clean fake payment will. Next week we build the real thing carefully.

**Do not rebuild the admin dashboard.** The version from Day 4 is sufficient. Polish is lower priority than correctness.

**Seed data is the difference between a demo and a project.** An empty database at demo time looks bad. A `db/seed.sql` that populates 20 products, 3 admin users, and 5 sample orders (one per status) takes an hour and saves you at demo time.

**Write the race-condition test.** Open two browser windows in incognito, each with the cart loaded for the last unit of a product. Click Pay in both within a second of each other. Watch one succeed, one fail. Screenshot it for the README -- this is proof your transaction works.

**Progressive enhancement.** The checkout forms should work with JavaScript disabled. Try it: DevTools -> Settings -> Debugger -> Disable JavaScript -> refresh. The forms still submit (Server Actions handle the POST), only the Zustand store breaks. Ship forms that degrade gracefully.

---

## If You Finish Early

- **Discount codes.** Add a `discounts` table and a field on checkout. Server Action validates the code and adjusts the subtotal.
- **Delivery fee calculation.** Flat fee based on region selected in the info step.
- **Order edit on admin side.** Let an admin add or remove items from a pending order (with restock).
- **Email or SMS receipt on confirmation.** Fire-and-forget via AT's SMS API.

Good luck. Week 16 takes the fake "Pay" button and makes it open a real M-Pesa STK push.
