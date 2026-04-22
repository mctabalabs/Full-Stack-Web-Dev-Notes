# Understanding CORS: Cross-Origin Resource Sharing

> **AI boundaries this week:** 50% manual / 50% AI. See [ai.md](../ai.md).

## Learning Goals

By the end of this lesson, you should be able to:

- Explain what the Same-Origin Policy is and why browsers enforce it
- Describe what CORS is and how it solves cross-origin communication
- Understand preflight requests and when the browser sends them
- Configure the `cors` middleware in Express with custom options
- Debug common CORS errors confidently

**Prior-week concepts you will use today:**
- HTTP methods and headers (Week 6)
- Fetch API and API calls from React (Week 9, Day 1)
- Express middleware (Week 10, Day 1 -- if you have started the capstone)

**Estimated time:** 1-1.5 hours

---

## 1. The Same-Origin Policy

Every browser enforces a security rule called the **Same-Origin Policy**. It says: JavaScript running on one origin cannot read responses from a different origin.

An "origin" is the combination of three things:

| Part | Example |
|---|---|
| **Protocol** | `http` or `https` |
| **Host** | `localhost`, `example.com` |
| **Port** | `3000`, `5000`, `80` |

Two URLs have the same origin only if ALL three parts match:

| URL A | URL B | Same Origin? | Why |
|---|---|---|---|
| `http://localhost:3000` | `http://localhost:3000/about` | Yes | Same protocol, host, and port |
| `http://localhost:3000` | `http://localhost:5000` | No | Different port |
| `http://localhost:3000` | `https://localhost:3000` | No | Different protocol |
| `https://app.example.com` | `https://api.example.com` | No | Different host (subdomain counts) |

### Why does this rule exist?

Imagine you are logged into your bank at `https://bank.com`. You have a session cookie that proves who you are. Now you visit a malicious site at `https://evil.com`. Without the Same-Origin Policy, that site's JavaScript could do this:

```javascript
// This runs on evil.com
const response = await fetch("https://bank.com/api/transfer", {
  method: "POST",
  body: JSON.stringify({ to: "attacker", amount: 10000 }),
  credentials: "include", // Sends your bank cookies!
});
const data = await response.json(); // Read your account data
```

The Same-Origin Policy blocks this. The browser will send the request, but it will refuse to let `evil.com`'s JavaScript read the response. The bank's data stays safe.

> **Key Insight:** The Same-Origin Policy protects the **user**, not the server. The server still receives the request -- it is the browser that blocks the response from reaching untrusted JavaScript.

---

## 2. The Problem: Your Own Frontend Cannot Talk to Your Own Backend

Here is the frustrating part. When you run your React app on `http://localhost:3000` and your Express server on `http://localhost:5000`, the browser sees two different origins (different ports). So when React calls your API:

```javascript
// This runs in your React app at localhost:3000
const response = await fetch("http://localhost:5000/api/links");
```

The browser blocks the response. You see this error in the console:

```
Access to fetch at 'http://localhost:5000/api/links' from origin
'http://localhost:3000' has been blocked by CORS policy: No
'Access-Control-Allow-Origin' header is present on the requested resource.
```

Your server processed the request and sent back the data. The browser received it. But it refused to hand it to your JavaScript because the origins do not match.

This is where CORS comes in.

---

## 3. What CORS Is

CORS (Cross-Origin Resource Sharing) is a system that lets the **server** tell the **browser**: "It is safe to let JavaScript from this other origin read my responses."

The server does this by including specific HTTP headers in its responses. The most important one is:

```
Access-Control-Allow-Origin: http://localhost:3000
```

This header says: "Responses from this server can be read by JavaScript running on `http://localhost:3000`." When the browser sees this header, it allows the response through.

You can also use a wildcard to allow any origin:

```
Access-Control-Allow-Origin: *
```

This is common during development but should be restricted in production.

### The CORS Dance (Simple Requests)

For simple requests (GET, POST with basic content types), the flow is:

```
1. React (localhost:3000) sends:
   GET /api/links HTTP/1.1
   Origin: http://localhost:3000          <-- Browser adds this automatically

2. Express (localhost:5000) responds:
   HTTP/1.1 200 OK
   Access-Control-Allow-Origin: *         <-- Server says "anyone can read this"
   Content-Type: application/json
   [{"id": "abc", "amount": 5000}]

3. Browser checks the header:
   "Does Access-Control-Allow-Origin include my origin?"
   Yes -> Hand the response to JavaScript
   No  -> Block the response, throw a CORS error
```

---

## 4. Preflight Requests

Not all requests are "simple." When your JavaScript sends a request with custom headers, uses methods like PUT or DELETE, or sends JSON with `Content-Type: application/json`, the browser sends an extra request first called a **preflight**.

The preflight is an `OPTIONS` request. It asks the server: "I am about to send a POST with JSON. Will you accept it?"

```
1. Browser sends a PREFLIGHT:
   OPTIONS /api/links HTTP/1.1
   Origin: http://localhost:3000
   Access-Control-Request-Method: POST
   Access-Control-Request-Headers: Content-Type

2. Server responds:
   HTTP/1.1 204 No Content
   Access-Control-Allow-Origin: *
   Access-Control-Allow-Methods: GET, POST, PUT, DELETE
   Access-Control-Allow-Headers: Content-Type

3. Browser reads the preflight response:
   "The server allows POST with Content-Type from my origin."
   -> Sends the actual POST request

4. Server responds to the actual request:
   HTTP/1.1 201 Created
   Access-Control-Allow-Origin: *
   {"id": "abc123", "amount": 5000}
```

This is why you sometimes see two requests in the browser's Network tab for a single API call -- the first is the preflight (`OPTIONS`), the second is your actual request.

> **Common Gotcha:** If your server does not handle `OPTIONS` requests, the preflight fails and the browser never sends your actual request. The `cors` middleware handles this automatically.

---

## 5. Using the `cors` Middleware in Express

When you write `app.use(cors())` in your Express server, the `cors` package automatically:

1. Adds `Access-Control-Allow-Origin: *` to every response
2. Handles preflight `OPTIONS` requests
3. Sets all the necessary CORS headers

### Basic Usage (Development)

```javascript
const cors = require("cors");
app.use(cors()); // Allow all origins
```

This is what you use in the capstone project. It works for development because your frontend and backend are both on localhost.

### Restricted Usage (Production)

In production, you should only allow requests from your own frontend:

```javascript
const cors = require("cors");

app.use(cors({
  origin: "https://paylink.yourdomain.com",  // Only allow your frontend
  methods: ["GET", "POST"],                   // Only these HTTP methods
  credentials: true,                          // Allow cookies to be sent
}));
```

### Multiple Allowed Origins

If you need to allow several origins (e.g., your production site and a staging environment):

```javascript
const allowedOrigins = [
  "https://paylink.yourdomain.com",
  "https://staging.yourdomain.com",
];

app.use(cors({
  origin: function (origin, callback) {
    // Allow requests with no origin (like mobile apps or Postman)
    if (!origin) return callback(null, true);

    if (allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error("Not allowed by CORS"));
    }
  },
}));
```

---

## 6. Common CORS Headers Reference

| Header | Purpose | Example |
|---|---|---|
| `Access-Control-Allow-Origin` | Which origins can read the response | `*` or `http://localhost:3000` |
| `Access-Control-Allow-Methods` | Which HTTP methods are allowed | `GET, POST, PUT, DELETE` |
| `Access-Control-Allow-Headers` | Which custom headers the client can send | `Content-Type, Authorization` |
| `Access-Control-Allow-Credentials` | Whether cookies/auth headers are allowed | `true` |
| `Access-Control-Max-Age` | How long (seconds) the browser can cache the preflight | `86400` (24 hours) |

---

## 7. Debugging CORS Errors

When you see a CORS error, follow this checklist:

1. **Is your server running?** A CORS error can mask a connection error. Check that your Express server is actually responding.

2. **Open the Network tab.** Find the failed request. Check if there is a preflight (`OPTIONS`) request before it. Did the preflight succeed?

3. **Check the response headers.** Click the request and look at the Response Headers. Is `Access-Control-Allow-Origin` present? Does it match your frontend's origin?

4. **Is `cors()` middleware placed before your routes?** Middleware runs in order. If `cors()` comes after a route, that route will not have CORS headers.

```javascript
// Wrong: CORS middleware comes after the route
app.get("/api/links", (req, res) => { /* ... */ });
app.use(cors());

// Correct: CORS middleware comes first
app.use(cors());
app.get("/api/links", (req, res) => { /* ... */ });
```

5. **Are you sending credentials?** If your frontend sends `credentials: "include"` (for cookies), the server cannot use `Access-Control-Allow-Origin: *`. It must specify the exact origin and set `Access-Control-Allow-Credentials: true`.

> **Remember:** CORS is always a server-side fix. You cannot bypass it from the frontend. If you see a CORS error, the fix is on the Express side.

---

## 8. CORS Beyond Development

When you deploy your app, CORS behavior changes:

- **Same origin (recommended):** If your React build is served by the same Express server (or through a reverse proxy like Nginx), everything is the same origin. CORS is not needed at all.
- **Different origins:** If your frontend is on `https://app.paylink.com` and your API is on `https://api.paylink.com`, you need CORS configured on the API server to allow the frontend's origin.

Most production setups use a reverse proxy that makes the frontend and API appear to be on the same origin, avoiding CORS entirely.

---

## Quick Activity

1. Start your Express server from the capstone without `cors()` middleware
2. Open your React app and try to fetch data -- observe the CORS error in the console
3. Open the Network tab and inspect the failed request's response headers
4. Add `app.use(cors())` back and verify the request succeeds
5. Try configuring `cors({ origin: "http://localhost:9999" })` -- a wrong origin. Observe the error changes.

---

## Key References

- [MDN: Cross-Origin Resource Sharing (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [MDN: Same-Origin Policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)
- [Express cors middleware documentation](https://expressjs.com/en/resources/middleware/cors.html)
