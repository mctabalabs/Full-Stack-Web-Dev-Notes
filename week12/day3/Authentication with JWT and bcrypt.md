# Week 12, Day 3: Authentication with JWT and bcrypt

By the end of today your CRM server will have a `users` table, a signup route that hashes passwords with bcrypt, a login route that issues a signed JSON Web Token, and a `requireAuth` middleware that refuses any request to `/api/leads/*` without a valid token. You will also update the React dashboard to store the token, send it on every request, and redirect to a login page when it expires. This is the difference between a toy app and one that can sit on the public internet without being emptied by the first stranger who finds the URL.

Everything today leans on yesterday's refactor. Because the routes are wired through a single `routes/leads.routes.js` file and the business logic is in services, adding auth is a two-file change, not a twenty-file change. If you skipped the refactor, do not try to do auth on top of the Week 11 layout -- go back and finish Day 2.

**Prior-week concepts you will use today:**
- The clean `routes/controllers/services/repositories` layout (Week 12, Day 2)
- `AppError` and the error-handling middleware (Week 12, Day 2)
- Parameterised SQL queries (Week 12, Day 2)
- React fetch with headers (Week 9, Day 1)
- `localStorage` and React Router (Week 8, Day 3)
- Environment variables, `.env` discipline (Week 10, Day 1)

**Estimated time:** 3-4 hours

---

## Concepts: Passwords, Hashes, Tokens

Before we write code, get the words straight. Every mistake people make in authentication comes from confusing two of these four things.

### Password

The secret string the user types. You never, ever store it. You never log it. You never email it. When you need to compare it later you will compare the hash, not the password itself. If there is a plaintext password anywhere in your database or logs, you have failed.

### Hash

A one-way fingerprint of the password. Given a password, a hash function produces a fixed-length scrambled string. Given the hash, there is no way to work backwards to the original password except by guessing -- and a good hash function is slow enough that guessing takes years.

We use **bcrypt**, which has three properties worth naming:

1. **Slow on purpose.** bcrypt is thousands of times slower than `md5` or `sha256`. For a legitimate login this does not matter -- a few hundred milliseconds is fine. For an attacker trying a billion guesses, slowness turns "a weekend" into "a century".
2. **Salted automatically.** A salt is a random string mixed into the hash so two users with the same password end up with different hashes. Without a salt, a leaked database shows you which users picked `123456` just by looking at which hashes are identical. bcrypt salts every password by default -- you cannot forget.
3. **Tunable via rounds.** The "work factor" is a number (we use 10-12) that controls how slow the hash is. As laptops get faster you raise it. Every password you hash today is stamped with its work factor so old hashes still verify.

A hashed password looks like this:

```
$2b$12$nOUIs5kJ7naTuTFkBy1veuK0kSxUFXfuaOKdOKf9xYT0KKIGSJwFa
```

That is *one string*, not a structured record. It encodes the algorithm (`$2b`), the rounds (`$12`), the salt, and the final hash. When a user logs in, bcrypt reads the rounds and salt out of the stored string, rehashes the incoming password, and compares. You never have to manage salts manually.

### Token

A signed note the server hands the client after a successful login. Every subsequent request carries the note, the server verifies the signature, and if the signature is valid the server trusts whatever the note says ("this request is user id 42"). That is all a token is -- a note with a tamper-evident seal.

We use **JWTs** (JSON Web Tokens). A JWT is three base64 strings joined by dots:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI0MiIsImlhdCI6MTc... .nOUIs5kJ7na...
```

- The first part is the **header** -- says which algorithm was used.
- The second part is the **payload** -- the claims (user id, role, expiry).
- The third part is the **signature** -- an HMAC over header+payload using a secret only the server knows.

Anyone can base64-decode the payload and read what is inside. A JWT is **not encrypted**. Do not put secrets in it. What makes it secure is that nobody can change the payload without invalidating the signature, because they do not know the server's secret. If someone edits the payload to say `"role":"admin"` and re-base64s it, the signature will not match and `jwt.verify` will reject it.

### Session

A server-remembered fact about "who this browser is right now". There are two ways to do sessions:

- **Server-side sessions** -- the server stores a random id in a database or Redis, the client has a cookie with that id, and every request the server looks up the id. This is what Rails, Django, and classical PHP apps do.
- **Stateless sessions with JWTs** -- the server stores nothing, the client has the token, and every request the server verifies the token cryptographically. This is what we are doing today.

The JWT approach has one trade-off worth naming: you cannot easily "log out a user" from the server side, because the token is valid until it expires. If someone steals a token, it is good until expiry. Mitigations: short expiry times (we will use 1 hour), a refresh-token workflow (not today), or maintaining a denylist of revoked tokens (later). For a CRM with internal users, 1-hour tokens are plenty.

---

## Step 1: The `users` Table

In `server/db/schema.sql`, add at the bottom:

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT NOT NULL UNIQUE,
  password_hash TEXT NOT NULL,
  name TEXT NOT NULL,
  role TEXT NOT NULL DEFAULT 'agent'
    CHECK (role IN ('admin', 'agent')),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_users_email ON users(email);
```

Four things to notice.

**`password_hash`, not `password`.** The column name is a form of documentation -- a future teammate reading the schema should see at a glance that this column is a hash, not a secret they can use.

**`email` is `UNIQUE`.** Two users cannot share an email. This is a common-sense product rule, but it is also a security-relevant one: if two users had the same email, a login attempt would be ambiguous.

**`role` is constrained to `admin` or `agent`.** Tomorrow we use this to give admins access to every lead and agents access only to their own. Starting with two roles is almost always right -- every system eventually needs at least two, and you can add more later.

**`password_hash TEXT`, not `VARCHAR(60)`.** A bcrypt hash is always 60 characters, but Postgres's `TEXT` and `VARCHAR` have the same performance and the same storage, so there is no reason to lock yourself in.

Load the new table:

```bash
psql -U crm_user -d crm -h localhost -c "
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT NOT NULL UNIQUE,
  password_hash TEXT NOT NULL,
  name TEXT NOT NULL,
  role TEXT NOT NULL DEFAULT 'agent' CHECK (role IN ('admin', 'agent')),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_users_email ON users(email);
"
```

And verify with `\d users`.

---

## Step 2: Install the Libraries

From `server/`:

```bash
npm install bcrypt jsonwebtoken
```

Add two new required env vars to `config/env.js`:

```javascript
const required = [
  "DB_HOST", "DB_PORT", "DB_USER", "DB_PASSWORD", "DB_NAME",
  "META_VERIFY_TOKEN",
  "JWT_SECRET",            // NEW
];

// ...in the exported object:
JWT_SECRET: process.env.JWT_SECRET,
JWT_EXPIRES_IN: process.env.JWT_EXPIRES_IN || "1h",
BCRYPT_ROUNDS: parseInt(process.env.BCRYPT_ROUNDS || "12", 10),
```

And in `.env`:

```env
JWT_SECRET=replace-me-with-a-long-random-string
JWT_EXPIRES_IN=1h
BCRYPT_ROUNDS=12
```

**Generate a real secret.** Never commit a JWT secret. Generate one with:

```bash
node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"
```

Copy the output into `JWT_SECRET`. If the secret ever leaks, every token signed with it is compromised -- rotate it and every user has to log in again.

---

## Step 3: The Users Repository

Create `server/repositories/users.repo.js`:

```javascript
// server/repositories/users.repo.js
const { query } = require("../config/db");

async function findByEmail(email) {
  const { rows } = await query(
    "SELECT * FROM users WHERE email = $1",
    [email.toLowerCase()]
  );
  return rows[0] || null;
}

async function findById(id) {
  const { rows } = await query(
    "SELECT id, email, name, role, created_at FROM users WHERE id = $1",
    [id]
  );
  return rows[0] || null;
}

async function insert({ email, passwordHash, name, role = "agent" }) {
  const { rows } = await query(
    `INSERT INTO users (email, password_hash, name, role)
     VALUES ($1, $2, $3, $4)
     RETURNING id, email, name, role, created_at`,
    [email.toLowerCase(), passwordHash, name, role]
  );
  return rows[0];
}

module.exports = { findByEmail, findById, insert };
```

Three things worth pointing at.

**`email.toLowerCase()` everywhere.** Emails are case-insensitive in practice -- users will sign up as `Wanjiru@Example.com` and log in as `wanjiru@example.com` expecting it to work. Normalising in the repository is the only way to make `UNIQUE` do the right thing.

**`findByEmail` returns the full row including `password_hash`.** This function is only called by the login service when it needs to check a password. It is tempting to never return `password_hash`, but then login cannot work. The discipline is: only the service uses `findByEmail`, and the service never includes `password_hash` in its return value to the controller.

**`findById` explicitly lists columns and omits `password_hash`.** This is what the rest of the app uses when it wants "the current user". There is no way to accidentally leak a hash through `findById`. The narrow interface is the defence.

---

## Step 4: The Auth Service

Create `server/services/auth.service.js`:

```javascript
// server/services/auth.service.js
const bcrypt = require("bcrypt");
const jwt = require("jsonwebtoken");
const env = require("../config/env");
const usersRepo = require("../repositories/users.repo");
const AppError = require("../utils/AppError");

async function signup({ email, password, name }) {
  if (!email || !password || !name) {
    throw new AppError("email, password, and name are required", 400);
  }
  if (password.length < 8) {
    throw new AppError("Password must be at least 8 characters", 400);
  }

  const existing = await usersRepo.findByEmail(email);
  if (existing) {
    throw new AppError("An account with that email already exists", 409);
  }

  const passwordHash = await bcrypt.hash(password, env.BCRYPT_ROUNDS);
  const user = await usersRepo.insert({ email, passwordHash, name });
  const token = signToken(user);
  return { user, token };
}

async function login({ email, password }) {
  if (!email || !password) {
    throw new AppError("email and password are required", 400);
  }

  const user = await usersRepo.findByEmail(email);
  if (!user) {
    throw new AppError("Invalid credentials", 401);
  }

  const ok = await bcrypt.compare(password, user.password_hash);
  if (!ok) {
    throw new AppError("Invalid credentials", 401);
  }

  // Strip the hash before returning
  const { password_hash, ...safeUser } = user;
  const token = signToken(safeUser);
  return { user: safeUser, token };
}

function signToken(user) {
  return jwt.sign(
    { sub: user.id, email: user.email, role: user.role },
    env.JWT_SECRET,
    { expiresIn: env.JWT_EXPIRES_IN }
  );
}

function verifyToken(token) {
  try {
    return jwt.verify(token, env.JWT_SECRET);
  } catch (err) {
    throw new AppError("Invalid or expired token", 401);
  }
}

module.exports = { signup, login, verifyToken };
```

Five decisions to understand.

**The error message for login is always "Invalid credentials".** Not "User not found" and not "Wrong password". If the server told attackers which email addresses were registered, they could build a list. Same-message-for-both is standard.

**`bcrypt.compare` is the only safe way to compare hashes.** It uses a constant-time comparison -- the function always takes the same amount of time regardless of how many characters matched. A naive `===` comparison leaks information through timing.

**The JWT payload claims are `sub`, `email`, `role`.** `sub` is the JWT standard name for "subject", meaning "who this token is about". We put the user id there. We also include `email` and `role` so the middleware can answer simple authorization questions without a database lookup. We do **not** include the `password_hash` or anything else sensitive.

**The token carries enough to authorize, but the middleware will still confirm the user exists** on sensitive routes. A deleted user's token should not still work.

**`signToken` is not exported.** It is an internal helper. The outside world calls `signup` and `login`, not `signToken`.

---

## Step 5: The `requireAuth` Middleware

Create `server/middleware/requireAuth.js`:

```javascript
// server/middleware/requireAuth.js
const authService = require("../services/auth.service");
const usersRepo = require("../repositories/users.repo");
const AppError = require("../utils/AppError");

module.exports = async function requireAuth(req, res, next) {
  try {
    const header = req.headers.authorization || "";
    const [scheme, token] = header.split(" ");

    if (scheme !== "Bearer" || !token) {
      throw new AppError("Missing authentication token", 401);
    }

    const payload = authService.verifyToken(token);
    const user = await usersRepo.findById(payload.sub);
    if (!user) {
      throw new AppError("User no longer exists", 401);
    }

    req.user = user;
    next();
  } catch (err) {
    next(err);
  }
};
```

The middleware does three things in order: check the header, verify the signature, confirm the user still exists. Only then does it attach `req.user` and call `next()`. If any step fails, the error handler from Day 2 turns it into a clean 401 JSON response.

Notice that we do fetch the user from the database on every request -- that is the cost of confirming the user has not been deleted. In high-traffic systems you would cache that lookup in Redis; for a CRM with dozens of agents it is free.

### A second middleware for role-based access

Add `server/middleware/requireRole.js`:

```javascript
// server/middleware/requireRole.js
const AppError = require("../utils/AppError");

module.exports = function requireRole(...allowedRoles) {
  return (req, res, next) => {
    if (!req.user) {
      return next(new AppError("Not authenticated", 401));
    }
    if (!allowedRoles.includes(req.user.role)) {
      return next(new AppError("Forbidden", 403));
    }
    next();
  };
};
```

`requireRole("admin")` becomes a one-liner guard you can drop on any admin-only route. We will use it tomorrow to protect the "reassign a lead" endpoint.

---

## Step 6: Routes and Controllers

Create `server/controllers/auth.controller.js`:

```javascript
// server/controllers/auth.controller.js
const authService = require("../services/auth.service");

async function signup(req, res) {
  const { user, token } = await authService.signup(req.body);
  res.status(201).json({ user, token });
}

async function login(req, res) {
  const { user, token } = await authService.login(req.body);
  res.json({ user, token });
}

async function me(req, res) {
  res.json({ user: req.user });
}

module.exports = { signup, login, me };
```

Create `server/routes/auth.routes.js`:

```javascript
// server/routes/auth.routes.js
const express = require("express");
const controller = require("../controllers/auth.controller");
const asyncHandler = require("../middleware/asyncHandler");
const requireAuth = require("../middleware/requireAuth");

const router = express.Router();

router.post("/signup", asyncHandler(controller.signup));
router.post("/login", asyncHandler(controller.login));
router.get("/me", requireAuth, asyncHandler(controller.me));

module.exports = router;
```

Wire it into `index.js`:

```javascript
const authRoutes = require("./routes/auth.routes");
const requireAuth = require("./middleware/requireAuth");

// ...

app.use("/api/auth", authRoutes);
app.use("/api/leads", requireAuth, leadsRoutes);  // <-- protected!
app.use("/webhook", webhookRoutes);                // <-- still public (Meta calls it)
```

Read that carefully. We put `requireAuth` *before* `leadsRoutes` on the `app.use` line. Every route in `leads.routes.js` now requires a valid token -- with no changes inside `leads.routes.js` itself. The clean layout from yesterday paid off in one line. The webhook stays public because Meta's servers, not our users, call it. (It is still protected, just by a different mechanism: the HMAC signature check from Week 11, Day 2. Do not forget that middleware is still running.)

---

## Step 7: Test the API

Create your first user (since there is no signup UI yet, use curl):

```bash
curl -X POST http://localhost:5000/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{"email":"wanjiru@mctaba.co.ke","password":"password123","name":"Wanjiru"}'
```

You should get back a user object and a token. Copy the token.

```bash
TOKEN="eyJhbGci....the.long.string"

# This should FAIL with 401:
curl http://localhost:5000/api/leads

# This should SUCCEED:
curl http://localhost:5000/api/leads -H "Authorization: Bearer $TOKEN"

# Confirm /me works:
curl http://localhost:5000/api/auth/me -H "Authorization: Bearer $TOKEN"
```

Wrong password should 401:

```bash
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"wanjiru@mctaba.co.ke","password":"wrong"}'
```

Duplicate email should 409:

```bash
curl -X POST http://localhost:5000/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{"email":"wanjiru@mctaba.co.ke","password":"password123","name":"Wanjiru"}'
```

Each of these proves a different layer of the middleware/service/repo pipeline is doing its job.

---

## Step 8: The React Side

Open the dashboard from Week 11, Day 4. Three changes.

### Create a Login page

`client/src/pages/Login.jsx`:

```jsx
import { useState } from "react";
import { useNavigate } from "react-router-dom";

export default function Login() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState("");
  const navigate = useNavigate();

  async function handleSubmit(e) {
    e.preventDefault();
    setError("");
    try {
      const res = await fetch("http://localhost:5000/api/auth/login", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ email, password }),
      });
      const data = await res.json();
      if (!res.ok) {
        setError(data.error?.message || "Login failed");
        return;
      }
      localStorage.setItem("token", data.token);
      localStorage.setItem("user", JSON.stringify(data.user));
      navigate("/");
    } catch (err) {
      setError("Network error");
    }
  }

  return (
    <form onSubmit={handleSubmit} className="max-w-sm mx-auto mt-20 space-y-4">
      <h1 className="text-2xl font-bold">Sign in</h1>
      {error && <p className="text-red-600">{error}</p>}
      <input className="w-full border p-2" placeholder="email"
        value={email} onChange={(e) => setEmail(e.target.value)} />
      <input className="w-full border p-2" type="password" placeholder="password"
        value={password} onChange={(e) => setPassword(e.target.value)} />
      <button className="w-full bg-black text-white p-2">Sign in</button>
    </form>
  );
}
```

### Add an `api` helper that sends the token

`client/src/lib/api.js`:

```javascript
const BASE = "http://localhost:5000/api";

export async function api(path, options = {}) {
  const token = localStorage.getItem("token");
  const res = await fetch(`${BASE}${path}`, {
    ...options,
    headers: {
      "Content-Type": "application/json",
      ...(token ? { Authorization: `Bearer ${token}` } : {}),
      ...options.headers,
    },
  });

  if (res.status === 401) {
    localStorage.removeItem("token");
    localStorage.removeItem("user");
    window.location.href = "/login";
    throw new Error("Not authenticated");
  }

  const data = await res.json();
  if (!res.ok) {
    throw new Error(data.error?.message || `HTTP ${res.status}`);
  }
  return data;
}
```

Replace every `fetch("http://localhost:5000/api/leads...")` in the dashboard with `api("/leads...")`. The helper adds the token automatically and redirects to `/login` if the server says 401. This is the single chokepoint for authenticated HTTP, and it is exactly parallel to the single chokepoint for authenticated routes on the server.

### A `ProtectedRoute` wrapper

```jsx
// client/src/components/ProtectedRoute.jsx
import { Navigate } from "react-router-dom";

export default function ProtectedRoute({ children }) {
  const token = localStorage.getItem("token");
  if (!token) return <Navigate to="/login" replace />;
  return children;
}
```

In your router setup, wrap the dashboard page in `<ProtectedRoute>`. Add `<Route path="/login" element={<Login />} />` outside of it.

---

## A Note on `localStorage` vs Cookies

You are storing the JWT in `localStorage`. This is simple and works for today. It has a weakness: if your React app ever has an XSS vulnerability (a piece of untrusted HTML rendered raw, for instance), an attacker can read `localStorage` and steal the token. The alternative is to store the token in an HttpOnly cookie, which JavaScript cannot read at all -- at the cost of dealing with CSRF protection, SameSite attributes, and cookie scope.

For an internal CRM used by people you know, `localStorage` is fine. For a public-facing product with untrusted users and third-party scripts on the page, you graduate to cookies. We will revisit this when we build the payment flows in Week 18 -- money changes the risk calculation.

---

## Checkpoint

1. `POST /api/auth/signup` creates a row in `users` with a bcrypt-looking hash. Verify in `psql`: `SELECT email, substring(password_hash, 1, 7) FROM users;` -- every hash should start with `$2b$12$`.
2. `POST /api/auth/login` with the right password returns a token. With the wrong password it returns 401 with message "Invalid credentials". With a non-existent email it returns the **same** 401 with the **same** message.
3. `GET /api/leads` without a token returns 401. With a valid token it returns the leads list.
4. `GET /api/auth/me` returns the current user's id, email, name, role -- and **does not** include `password_hash`. Verify.
5. Copy a token, edit one character in the middle, send it -- 401. The signature check is working.
6. In the React dashboard, visiting `/` without logging in redirects you to `/login`. After logging in, you see the leads table. After deleting the token from localStorage and refreshing, you get redirected to login again.
7. The Meta webhook still works. Sending a WhatsApp message creates a lead row. (Webhook is public; the `requireAuth` middleware is only on `/api/leads`.)

Commit:

```bash
git add .
git commit -m "feat: add jwt auth with signup, login, and requireAuth middleware"
```

---

## What You Learned

You now have an application that enforces who-can-do-what at two layers: the database is owned by a user with restricted privileges, and the API refuses requests that cannot prove who they are. The next layer -- *which* leads a logged-in user is allowed to see -- is tomorrow's work. That is authorization on top of authentication: not just "are you logged in?" but "is this lead yours to look at?"

Also notice, again, how small the change was. Adding auth to yesterday's architecture cost you one repository, one service, one middleware, one route file, one line in `index.js`, and three client-side files. If you had tried to retrofit auth into the Week 11 server, it would have been a day of finding every route and adding a `try { jwt.verify... } catch { ... }` block. Clean layers make hard things cheap.

Day 4 tomorrow: multi-user ownership, admin vs agent permissions, and the short Mongo contrast.
