# Week 12 Weekend Project: Lock It Down

Take the **Boda Dispatch CRM** you shipped on Week 11 weekend and harden it with everything you learned this week: PostgreSQL, layered architecture, JWT authentication, and multi-user ownership with roles. By Monday morning your dispatch system should be something you could put on the open internet without embarrassment.

This is a migration + extension project, not a green-field build. You are not starting over. You are upgrading.

**Estimated time:** 8-10 hours, split across Saturday and Sunday.

**Deadline:** Monday morning, before Week 13 Day 1 starts.

---

## The Story You Are Building For

Kevin runs Jetlink Boda, a small dispatch company in Kilimani. He has eighteen riders working shifts around a 5 km radius. Customers send a WhatsApp message to the company number -- "pick me up at Yaya Centre, drop at Valley Arcade" -- and one of Kevin's dispatchers reads the message, picks a rider based on who is free, and confirms the ride.

Kevin already has the Week 11 CRM you built -- customers message, leads land in the dashboard, dispatchers can see them. It has been working for six weeks and they love it. Two problems have turned up:

1. **Anyone with the URL can see everything.** Kevin's URL has leaked twice. Competitors read his incoming rides. He needs real logins.
2. **Two dispatchers both reach for the same ride** and riders get double-booked. Kevin wants each ride claimed by exactly one dispatcher, and for riders to be auto-assigned by the system -- round-robin, based on who has the fewest active trips -- so dispatchers can focus on reading messages.

He is paying you to fix both. Your week's work becomes his next release.

---

## Functional Requirements

### Users and Authentication

- There are **three** roles in this system: `admin`, `dispatcher`, `rider`.
- Admins and dispatchers log in via a web dashboard using email + password.
- Riders do not log in. Their WhatsApp phone number identifies them. A rider row has a phone, a name, a status (`online` / `offline`), and is created by the admin from the dashboard.
- Admins can create dispatchers and riders. Dispatchers cannot create either.
- JWTs expire in 1 hour.

### Ride Lifecycle

A ride (what the Week 11 CRM called a `lead`) moves through these states:

```
new -> assigned -> in_progress -> completed
                -> cancelled
```

- `new`: customer has messaged, no rider assigned yet.
- `assigned`: a dispatcher claimed the ride and the system has auto-picked a rider.
- `in_progress`: rider confirmed pickup.
- `completed`: rider marked the ride delivered.
- `cancelled`: dispatcher or customer backed out (any prior state can go here).

Invalid transitions (e.g. `new` -> `completed`) must be rejected by the service layer with a 409 error. This is a direct copy of the `VALID_TRANSITIONS` pattern from Day 2.

### Claiming and Auto-Assigning

- A dispatcher clicks "Claim" on a `new` ride. The server:
  1. Assigns the ride to that dispatcher.
  2. Picks a rider via **round-robin based on active load**: the rider who is currently `online` and has the fewest rides in `assigned` or `in_progress`. Ties broken by the rider who was assigned longest ago.
  3. Sets the ride state to `assigned`.
  4. Sends a WhatsApp message to both the customer ("Rider Peter is on the way, +254...") and the rider ("New ride: pick at Yaya, drop at Valley").
- If no riders are online, the claim fails with a clear error. The ride stays `new`.

This rider-picking query is the interesting part. Do it in SQL, not Node:

```sql
SELECT r.id
FROM riders r
LEFT JOIN rides ride
  ON ride.rider_id = r.id
  AND ride.status IN ('assigned', 'in_progress')
WHERE r.status = 'online'
GROUP BY r.id
ORDER BY COUNT(ride.id) ASC, MAX(ride.assigned_at) ASC NULLS FIRST
LIMIT 1
FOR UPDATE;
```

The `FOR UPDATE` locks the winning row so two simultaneous claims cannot pick the same rider. If you skip `FOR UPDATE`, race conditions *will* double-book riders on a busy Friday evening. Wrap the whole claim in a transaction (`BEGIN ... COMMIT`).

### Dashboard

- Login page, identical shape to Day 3.
- An "Active Rides" view showing `new`, `assigned`, and `in_progress` rides.
- A "Completed" view for history.
- Admin-only "Riders" page -- create, list, flip online/offline.
- Admin-only "Dispatchers" page -- create.
- A dispatcher sees only rides they claimed, plus unclaimed `new` rides. Admin sees everything.

### Bot Changes

The WhatsApp bot from Week 11 already creates leads. Two small additions:

- When a rider replies with the word "online" or "offline" (case-insensitive), flip their `status`. Authenticate by phone number.
- When a rider on an `in_progress` ride replies "done" or "delivered", move the ride to `completed`. Send a "Thank you, your ride is complete" message to the customer.

Both of these are state-machine branches inside `services/bot.service.js`. No new tables.

---

## Technical Requirements

These are non-negotiable. The project is graded on both working features *and* how they are built.

1. **PostgreSQL only.** No SQLite anywhere in the submitted project. `db/schema.sql` must be checked in and load cleanly on a fresh database.
2. **Layered architecture.** The folders must match the Day 2 structure: `routes/`, `controllers/`, `services/`, `repositories/`, `middleware/`, `config/`, `utils/`. SQL only in repositories. Business rules only in services. Grep-checkable.
3. **No raw SQL strings anywhere except repositories.** `grep -r "SELECT " src/controllers src/services` must return nothing.
4. **Parameterised queries everywhere.** `grep -r "\\${" src/repositories` should only show template literals that do not interpolate user input into SQL text.
5. **JWT auth on every route except `POST /api/auth/login`, `POST /api/auth/signup`, and `POST /webhook/whatsapp`.**
6. **Passwords hashed with bcrypt**, rounds configurable via env, default 12.
7. **Transactions around the claim flow.** The rider pick, the ride update, and the rider status update must all commit together or all roll back. Hint: use the `getClient()` helper from Day 2, run `BEGIN`, run your queries on the client, then `COMMIT` or `ROLLBACK`.
8. **`ON DELETE SET NULL` on `rides.rider_id` and `rides.dispatcher_id`.** Deleting a person should not evaporate historical rides.
9. **Seed script.** `db/seed.sql` creates 1 admin, 2 dispatchers, 4 riders (2 online, 2 offline), and 5 rides in different states. A fresh clone must be able to `npm run db:reset && npm run dev` and log in.
10. **A `README.md` in the project root** with setup instructions, env var list, how to reset the database, and a paragraph each on: how claiming works, how auto-assignment picks a rider, and how you tested the race condition.

---

## Submission Checklist

On Monday morning, demo the following to your pair partner in under ten minutes:

- [ ] Log in as admin, create two dispatchers and four riders. Two riders are online.
- [ ] Send a WhatsApp message to the bot. A new ride appears in the dispatcher's "Active Rides" page.
- [ ] Log in as dispatcher. Click Claim. Verify the ride moves to `assigned`, a rider was picked, and the customer got a confirmation WhatsApp (check the logs if the API is rate-limited).
- [ ] Flip the assigned rider to offline. Create another ride. Claim it. The other online rider is picked.
- [ ] Try to claim a ride when both riders are offline -- see the error message.
- [ ] Use `psql` to confirm rides and messages are in the correct state.
- [ ] `curl` the leads endpoint without a token -- 401.
- [ ] `curl` with an expired token (wait 61 minutes or set `JWT_EXPIRES_IN=5s` temporarily) -- 401.
- [ ] Delete a dispatcher in `psql`. Confirm the rides that dispatcher had are still in the database with `dispatcher_id = NULL`.
- [ ] Run `grep -r "SELECT " server/controllers server/services` and show it is empty.

---

## Grading Rubric (100 pts)

| Area | Points | What earns it |
|---|---|---|
| Postgres schema + migrations | 15 | All tables with correct types, constraints, indexes, FKs, cascade policies. Loads cleanly on a fresh db. |
| Layered architecture | 15 | No SQL outside repos, no business rules outside services, clean `index.js`. |
| Authentication | 15 | bcrypt, JWT, middleware, correct error messages. |
| Authorization | 15 | Dispatchers see only their claims, admin sees all, role enforcement on create-rider and create-dispatcher. |
| Claim + auto-assign | 15 | Correct SQL for rider picking, transaction + `FOR UPDATE`, works under two simultaneous claims. |
| Bot updates | 10 | `online`/`offline`/`done` handled correctly, customer notifications fire. |
| Seed script + README | 10 | Fresh clone works in one command; README is accurate. |
| Tests or race-condition proof | 5 | Written proof that the claim transaction cannot double-book, ideally a test; otherwise a careful README paragraph showing you thought about it. |

---

## Hints

**On the rider picking query.** The key insight is the `LEFT JOIN` with conditions *in the ON clause*, not in the WHERE. If you put `ride.status IN (...)` in the WHERE, you exclude riders with zero active rides -- the opposite of what you want. Put it in the `ON` and the `COUNT(ride.id)` becomes 0 for the idle rider, which sorts them to the top.

**On race conditions.** Before you trust your transaction, force a race. Open two terminal tabs, both hitting the claim endpoint at the same time with the same ride id. One should succeed, one should fail cleanly. If both succeed and both get the same rider, your `FOR UPDATE` is wrong or missing.

**On rider authentication for WhatsApp replies.** Riders do not have passwords. When the webhook fires with `"from": "+254712..."`, you look them up in the `riders` table. If found and status is `online` or `offline`, the command they typed acts on them. If the phone is not a registered rider, treat it as a customer lead (the Week 11 flow). This is a trusted-channel pattern: WhatsApp's delivery already proves ownership of the number.

**On the dispatcher UI.** The "Claim" button should disable itself the moment it is clicked, not after the response comes back. Otherwise a fast clicker fires the request twice and your transaction has to handle it. Optimistic UI is your friend here.

**On notifications.** You do not need to implement both customer and rider notifications perfectly. If you have time pressure, implement customer first, rider second, and note the gap in the README. Grading rewards honest trade-offs more than it penalises missing features.

**On what NOT to do.** Do not rewrite the WhatsApp webhook. Do not re-design the UI. Do not switch to Tailwind if you were not using it. Do not install Prisma. Do not install Redis. Every minute on those is a minute you are not building the new stuff. You are migrating and extending, not starting over.

---

## If You Finish Early

- **Add ride history timestamps.** A `ride_events` table with `(ride_id, event_type, actor_id, created_at)` so you have an audit trail: "dispatcher X claimed at 14:02, rider Y accepted at 14:03, delivered at 14:22". This is how real dispatch systems work and it is an easy extension.
- **Write one integration test.** Spin up a test database, simulate a webhook call, claim the ride, assert the rider was picked. One test beats none.
- **A simple analytics endpoint.** `GET /api/stats` -- total rides today, average time from `new` to `completed`, rides by rider. Admin only. One SQL query is all it takes.

Good luck. Pair up if you get stuck -- do not suffer alone on a Saturday night. Post in the student channel, ask your peer session partner, or flag on Sunday afternoon's open office hours. Week 13 is USSD and it starts Monday whether your weekend project is perfect or not.
