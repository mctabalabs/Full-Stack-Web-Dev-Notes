# Week 12 - Day 4 Assignment

## Title
Multi-User CRM -- Ownership, Roles, and Integration Tests

## Overview
Day 4 is pre-weekend polish day. Today you add row-scoped access so agents only see their own leads, integrate the auth into your React frontend (login screen + token storage), and write the integration tests that make this week's work provable. The frontend side is AI-assisted; the auth logic is still hand-written.

## Learning Objectives Assessed
- Enforce row-scoped access in repositories
- Add a login screen that stores the JWT in memory (not localStorage)
- Protect React routes with an auth wrapper
- Write integration tests that hit the real DB

## Prerequisites
- Days 1-3 completed

## AI Usage Rules

**Ratio:** 45/55. **Habit:** Never trust AI with auth. See [../ai.md](../ai.md).

- **ALLOWED FOR:** React login form styling, protected route wrapper (frontend only).
- **NOT ALLOWED FOR:** Writing the row-scoped access check, or the integration test assertions.
- **AUDIT REQUIRED:** Yes. Auth audit extended.

## Tasks

### Task 1: Row-scoped leads

**What to do:**
In your `leadsRepo.list` function, filter by `assigned_to = $user_id` unless the caller is an admin. Pass `req.user` into the service from the route handler:

```javascript
// leads route
router.get("/", requireAuth, async (req, res, next) => {
  try {
    const leads = await leadsService.list(req.query, req.user);
    res.json({ data: leads });
  } catch (err) { next(err); }
});
```

The service decides the scope:

```javascript
async function list(query, user) {
  const filters = { ...query };
  if (user.role !== "admin") {
    filters.assignedTo = user.id;
  }
  return leadsRepo.list(filters);
}
```

Write this logic by hand. It is the tenant-boundary for your CRM.

**Expected output:**
An agent token only sees their own leads. An admin token sees all.

### Task 2: Frontend login screen

**What to do:**
Build a simple login form in React that POSTs to `/auth/login` and stores the token in context (not localStorage -- we want to survive page reloads but avoid XSS footguns; for now, context + refresh = re-login is fine).

```jsx
// context/AuthContext.jsx
import { createContext, useState, useContext } from "react";

const AuthContext = createContext(null);
export function AuthProvider({ children }) {
  const [token, setToken] = useState(null);
  return <AuthContext.Provider value={{ token, setToken }}>{children}</AuthContext.Provider>;
}
export function useAuth() { return useContext(AuthContext); }
```

Login form calls `/auth/login`, sets the token via `useAuth`, then navigates to the dashboard. AI-assisted here.

**Expected output:**
A working login flow in the React dashboard.

### Task 3: ProtectedRoute wrapper

**What to do:**
```jsx
function ProtectedRoute({ children }) {
  const { token } = useAuth();
  if (!token) return <Navigate to="/login" />;
  return children;
}
```

Wrap the leads dashboard route with it. Unauthenticated visitors get redirected.

**Expected output:**
Visiting `/` without a token redirects to `/login`. With a token, you see leads.

### Task 4: Integration tests

**What to do:**
Install supertest:

```bash
npm install -D supertest
```

Write `tests/auth.test.js`:

```javascript
const request = require("supertest");
const app = require("../src/app");

describe("Auth", () => {
  it("rejects signup with short password", async () => {
    const res = await request(app).post("/auth/signup").send({ email: "t@t.com", password: "short" });
    expect(res.status).toBe(400);
  });

  it("signs up and logs in", async () => {
    const email = `test-${Date.now()}@t.com`;
    await request(app).post("/auth/signup").send({ email, password: "longenough" });
    const res = await request(app).post("/auth/login").send({ email, password: "longenough" });
    expect(res.status).toBe(200);
    expect(res.body.token).toBeTruthy();
  });

  it("rejects wrong password", async () => {
    const res = await request(app).post("/auth/login").send({ email: "never@t.com", password: "wrong" });
    expect(res.status).toBe(401);
  });
});
```

Write the test assertions yourself. These ARE auth code.

**Expected output:**
`npm test` runs and all three tests pass.

### Task 5: Pre-weekend checklist

**What to do:**
Create `CHECKLIST.md`:

```markdown
## Week 12 Day 4 Pre-Weekend Checklist

- [ ] Postgres running with crm_dev database
- [ ] Schema loaded with users and row-level access
- [ ] Signup, login, requireAuth, requireRole all working
- [ ] Frontend login screen stores token in context
- [ ] ProtectedRoute wraps dashboard
- [ ] Agents see only their own leads
- [ ] Admins see all leads
- [ ] Integration tests pass
- [ ] AI_AUDIT.md has auth manual-only proof current
- [ ] Repo pushed and clean
```

Tick honestly.

**Expected output:**
Checklist committed.

## Stretch Goals (Optional - Extra Credit)

- Add logout that clears the token from context.
- Add a "Switch to admin" demo page that shows the different views.
- Use `httpOnly` cookies instead of in-memory tokens for better XSS protection.

## Submission Requirements

- **What to submit:** Repo, tests, `CHECKLIST.md`, `AI_AUDIT.md`.
- **Deadline:** End of Day 4

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Row-scoped leads | 25 | Agents see only their own. Admins see all. Tested with two users. |
| Login screen and token context | 15 | Works end to end. |
| ProtectedRoute wrapper | 10 | Redirects unauthenticated. |
| Integration tests | 25 | All three tests passing. Assertions hand-written. |
| Pre-weekend checklist | 10 | Honest. |
| Audit current | 15 | Auth files still hand-written. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Storing JWT in localStorage by default.** It works but opens XSS exposure. In-memory + re-login is safer while you learn. Better options (httpOnly cookies) come in a later week.
- **Row-scoped access in the route instead of service.** The service is the right layer.
- **Skipping tests because "it works on my machine".** The tests are your proof in code review.

## Resources

- Day 4 reading: [Multi-user CRM and a MongoDB Aside.md](./Multi-user%20CRM%20and%20a%20MongoDB%20Aside.md)
- Week 12 AI boundaries: [../ai.md](../ai.md)
- supertest: https://github.com/ladjs/supertest
