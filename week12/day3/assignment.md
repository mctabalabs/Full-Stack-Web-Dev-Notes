# Week 12 - Day 3 Assignment

## Title
JWT Authentication, bcrypt Password Hashing, and requireAuth Middleware

## Overview
Today the auth rule activates for the rest of the programme: **never trust AI with auth**. You will write the users table, a signup route that hashes passwords with bcrypt, a login route that issues a JWT, and a `requireAuth` middleware that protects every other route. Every line of this is yours.

## Learning Objectives Assessed
- Hash passwords with bcrypt
- Issue and verify JWT tokens
- Build a `requireAuth` middleware
- Build a `requireRole(role)` middleware
- Enforce row-scoped access (users only see their own leads)

## Prerequisites
- Days 1-2 completed

## AI Usage Rules

**Ratio:** 45/55. **Habit:** Never trust AI with auth. See [../ai.md](../ai.md).

- **ALLOWED FOR:** NOTHING in this file. Every line today is hand-written.
- **NOT ALLOWED FOR:** Every auth-related line. If AI wrote any part of it, rewrite by hand.
- **AUDIT REQUIRED:** Yes. Full "Auth manual-only proof" section showing every auth file with a "written by me" flag and a one-line explanation.

## Tasks

### Task 1: Users table

**What to do:**
Add to your schema:

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT NOT NULL UNIQUE,
  password_hash TEXT NOT NULL,
  role TEXT NOT NULL DEFAULT 'agent' CHECK (role IN ('admin', 'manager', 'agent')),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

ALTER TABLE leads ADD COLUMN assigned_to UUID REFERENCES users(id);
```

Load it.

**Expected output:**
`users` table exists. `leads.assigned_to` column exists.

### Task 2: Install bcrypt and jsonwebtoken

**What to do:**
```bash
npm install bcrypt jsonwebtoken
```

Set in `.env`:
```
JWT_SECRET=generate_a_long_random_string_here
JWT_EXPIRY=24h
BCRYPT_ROUNDS=12
```

**Expected output:**
Dependencies installed. Env vars set. JWT_SECRET at least 32 random characters.

### Task 3: Signup and login routes (manual only)

**What to do:**
Create `src/routes/auth.js`:

```javascript
const express = require("express");
const bcrypt = require("bcrypt");
const jwt = require("jsonwebtoken");
const pool = require("../../db/pool");
const router = express.Router();

router.post("/signup", async (req, res, next) => {
  try {
    const { email, password, role = "agent" } = req.body;
    if (!email || !password || password.length < 8) {
      return res.status(400).json({ error: "Invalid input" });
    }

    const hash = await bcrypt.hash(password, parseInt(process.env.BCRYPT_ROUNDS, 10));
    const { rows } = await pool.query(
      "INSERT INTO users (email, password_hash, role) VALUES ($1, $2, $3) RETURNING id, email, role",
      [email, hash, role]
    );
    res.status(201).json({ user: rows[0] });
  } catch (err) {
    if (err.code === "23505") return res.status(409).json({ error: "Email taken" });
    next(err);
  }
});

router.post("/login", async (req, res) => {
  const { email, password } = req.body;
  if (!email || !password) return res.status(400).json({ error: "Invalid input" });

  const { rows } = await pool.query("SELECT id, email, password_hash, role FROM users WHERE email = $1", [email]);
  const user = rows[0];
  if (!user) return res.status(401).json({ error: "Invalid credentials" });

  const ok = await bcrypt.compare(password, user.password_hash);
  if (!ok) return res.status(401).json({ error: "Invalid credentials" });

  const token = jwt.sign(
    { sub: user.id, email: user.email, role: user.role },
    process.env.JWT_SECRET,
    { expiresIn: process.env.JWT_EXPIRY }
  );

  res.json({ token });
});

module.exports = router;
```

Type every line yourself. Zero AI. If you accidentally tab-complete, delete and rewrite.

**Expected output:**
POST /auth/signup creates a user. POST /auth/login returns a JWT.

### Task 4: requireAuth middleware

**What to do:**
Create `src/middleware/requireAuth.js`:

```javascript
const jwt = require("jsonwebtoken");

module.exports = function requireAuth(req, res, next) {
  const header = req.headers.authorization;
  if (!header || !header.startsWith("Bearer ")) {
    return res.status(401).json({ error: "Missing token" });
  }
  const token = header.slice(7);
  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET);
    req.user = { id: payload.sub, email: payload.email, role: payload.role };
    next();
  } catch (err) {
    return res.status(401).json({ error: "Invalid token" });
  }
};
```

Apply it to your existing lead routes:

```javascript
app.use("/api/leads", requireAuth, leadsRoutes);
```

Also create `requireRole('admin')` for routes only admins can hit.

**Expected output:**
Accessing /api/leads without a token returns 401. With a valid token, it works.

### Task 5: Auth manual-only proof in the audit

**What to do:**
In `AI_AUDIT.md`, add a full "Auth manual-only proof" section listing:
- Every file that contains auth code
- For each file, confirm it was hand-written
- For each file, one sentence explaining what it does
- Your own checklist: did any AI touch any of these files?

**Expected output:**
Audit updated with explicit hand-written flags.

## Stretch Goals (Optional - Extra Credit)

- Add a `/auth/me` endpoint that returns the current user from the JWT.
- Add rate limiting on `/auth/login` to prevent brute force.
- Add refresh token support with a separate short-lived access token.

## Submission Requirements

- **What to submit:** Repo, auth files, audit, `AI_AUDIT.md` with manual-only proof.
- **Deadline:** End of Day 3

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Users table + assigned_to FK | 15 | Schema updated. Constraints correct. |
| bcrypt password hashing | 20 | Signup hashes with configured rounds. Login compares. |
| JWT issuance and verify | 20 | Token contains sub, email, role. Expires correctly. |
| requireAuth middleware | 20 | Protects routes. 401 on missing or invalid token. |
| requireRole middleware | 10 | Admin-only routes reject non-admin tokens. |
| Auth manual-only proof | 15 | Audit shows every auth file hand-written. Zero AI. Any AI-touched line costs the full 15 points. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Storing bcrypt rounds as a string.** Always `parseInt(process.env.BCRYPT_ROUNDS, 10)`.
- **Using a weak JWT_SECRET.** Generate with `openssl rand -hex 32` or similar. At least 32 random hex characters.
- **Catching all errors in the auth route.** Let unique-constraint violations propagate so you can return 409, not 500.

## Resources

- Day 3 reading: [Authentication with JWT and bcrypt.md](./Authentication%20with%20JWT%20and%20bcrypt.md)
- Week 12 AI boundaries: [../ai.md](../ai.md)
- bcrypt docs: https://github.com/kelektiv/node.bcrypt.js
- jsonwebtoken docs: https://github.com/auth0/node-jsonwebtoken
