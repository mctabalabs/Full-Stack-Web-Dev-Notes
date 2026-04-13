# Week 27 - Day 4 Assignment

## Title
API Design -- Resource Shapes And Versioning

## Overview
Day 4 designs the HTTP API for the capstone: URL shapes, request/response formats, error envelope, and versioning. You commit an OpenAPI-lite markdown spec before writing any handler.

## Learning Objectives Assessed
- Design RESTful resource URLs
- Define a consistent error envelope
- Version the API from day zero
- Document endpoints in markdown

## Prerequisites
- Days 1-3 completed

## AI Usage Rules

**Ratio this week:** 55% manual / 45% AI
**Habit:** Architecture dip. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Endpoint descriptions after you defined them.
- **NOT ALLOWED FOR:** The URL shapes.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: URL shape

**What to do:**
`capstone/api.md`, list every endpoint:

```
GET    /v1/resources
POST   /v1/resources
GET    /v1/resources/:id
PATCH  /v1/resources/:id
DELETE /v1/resources/:id

GET    /v1/bookings
POST   /v1/bookings
...
```

Note the `/v1/` prefix everywhere. Tenant is derived from subdomain or header, not URL.

**Expected output:**
Committed with ~15-20 endpoints.

### Task 2: Error envelope

**What to do:**
Every error response follows one shape:

```json
{
  "error": {
    "code": "booking.slot_taken",
    "message": "That slot is already booked",
    "details": { "slotId": "..." }
  }
}
```

Document the shape in `api.md` and list the error codes you expect.

**Expected output:**
Documented.

### Task 3: Success envelope

**What to do:**
Decide: envelope or bare data?

```json
// Bare
{ "id": "...", "name": "..." }

// Envelope
{ "data": { "id": "...", "name": "..." } }
```

Pick one and justify in one paragraph. Consistency matters more than the choice.

**Expected output:**
Decision documented.

### Task 4: Pagination

**What to do:**
Decide: offset or cursor? For tenant-scoped lists, cursor is usually better (stable under inserts).

Document the shape in `api.md`:

```
GET /v1/bookings?limit=20&cursor=eyJpZCI6Li4ufQ

Response:
{
  "data": [...],
  "pageInfo": { "nextCursor": "..." }
}
```

**Expected output:**
Documented.

### Task 5: Versioning notes

**What to do:**
`api-versioning.md`, 5-7 sentences:
- Why `/v1/` from day zero?
- When do you go to `/v2/`?
- What does "additive only" mean?
- When is a response field removal a breaking change? (Always.)

Your own words.

**Expected output:**
Committed.

## Stretch Goals (Optional - Extra Credit)

- Generate a real OpenAPI YAML.
- Add rate-limit headers (`X-RateLimit-Remaining`).
- Plan HATEOAS links.

## Submission Requirements

- **What to submit:** Repo, api.md, versioning notes, `AI_AUDIT.md`.
- **Deadline:** End of Day 4

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Endpoint list | 25 | 15-20 endpoints. |
| Error envelope | 15 | Documented + codes. |
| Success envelope | 10 | Decision + justification. |
| Pagination | 20 | Cursor shape. |
| Versioning notes | 25 | Four questions answered. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Putting tenant in the URL (`/tenants/:id/bookings`).** Derive from subdomain or auth.
- **Inconsistent errors.** Every handler returns the same envelope.
- **No versioning.** You will regret it.

## Resources

- Day 4 reading: [API Design.md](./API%20Design.md)
- Week 27 AI boundaries: [../ai.md](../ai.md)
