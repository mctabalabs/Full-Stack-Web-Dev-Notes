# React Routing & Multi-Page Apps

> **Stack:** React + React Router v6 (package: `react-router`)
>
> React Router is the most widely used routing solution for client-side React apps. It lets you build apps with multiple "pages" without ever leaving the browser tab.

---

## 1. Introduction to Client-Side Routing

### What Problem Does Routing Solve?

In a basic React app, everything lives inside one component tree (`App.jsx`). You show and hide different screens using conditional rendering - things like `if (page === "about") return <About />`. This works for tiny demos, but it breaks down quickly in a real product:

- URLs don't reflect what the user is looking at (bad UX, can't bookmark, can't share).
- The browser's **Back** and **Forward** buttons don't work properly.
- You can't build proper multi-step flows like `/products/42 → /cart → /checkout → /success`.
- Search engines can't crawl your pages efficiently.

### Client-Side Routing vs. Server-Side Routing

In traditional websites (think plain HTML + PHP), every time you click a link the browser sends a **new request to the server**, which responds with a completely new HTML page. This causes a full page reload.

**Client-side routing** is different. Your React app loads once, and from then on, navigation is handled entirely in the browser by JavaScript. When you click a link:

1. React Router intercepts the click.
2. It updates the URL in the address bar (using the browser's [History API](https://developer.mozilla.org/en-US/docs/Web/API/History_API)).
3. It swaps out the component being rendered - no server request, no page reload.

This makes navigation feel instant and preserves in-memory state (like cart contents).

### What Routing Enables in Real Products

Routing is the difference between a demo UI and a real, shippable application. Here's how it maps to real-world products:

| Product Type   | Example Routes                                                              |
| -------------- | --------------------------------------------------------------------------- |
| Portfolio site | `/`, `/about`, `/projects`, `/contact`                                      |
| Blog           | `/posts`, `/posts/:slug`                                                    |
| E-commerce     | `/products`, `/products/:id`, `/cart`, `/checkout`, `/checkout/success`     |
| Dashboard      | `/dashboard`, `/dashboard/settings`, `/dashboard/billing`                   |

---

## 2. Setting Up React Router

### Install

```bash
npm install react-router
```

> **Note:** You may see `react-router-dom` in older tutorials. Both are valid and kept up to date (v7.x as of early 2025). For new projects, `react-router` is the recommended import. Pick one and stay consistent.

### Wrap Your App with `<BrowserRouter>`

`<BrowserRouter>` is the **context provider** for routing. It must wrap your entire app so that all routing components and hooks have access to the current URL and navigation state.

The standard place to do this is `main.jsx` (or `index.jsx`):

```jsx
// main.jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { BrowserRouter } from "react-router";
import App from "./App";

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </StrictMode>
);
```

Everything inside `<BrowserRouter>` can now use routing features - `<Routes>`, `<Link>`, hooks like `useNavigate`, etc.

---

## 3. Core Concepts: Routes, Links, and Outlets

### `<Routes>` and `<Route>` - Mapping URLs to Components

`<Routes>` is a container that looks at the current URL and renders whichever `<Route>` matches. Each `<Route>` defines:

- **`path`** - the URL pattern to match.
- **`element`** - the component to render when that path is active.

```jsx
// App.jsx
import { Routes, Route } from "react-router";
import Home from "./pages/Home";
import About from "./pages/About";
import Products from "./pages/Products";
import NotFound from "./pages/NotFound";

export default function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
      <Route path="/products" element={<Products />} />
      <Route path="*" element={<NotFound />} />
    </Routes>
  );
}
```

> The `path="*"` route is a **wildcard** - it matches any URL that wasn't matched by the routes above it. This is how you build a 404 page. It should always be last.

### `<Link>` - Client-Side Navigation

Never use a plain `<a href="...">` tag for internal navigation in React. That triggers a full page reload, blowing away your app's state. Use `<Link>` instead - it renders as an `<a>` tag but intercepts the click and uses client-side navigation.

```jsx
import { Link } from "react-router";

function Navbar() {
  return (
    <nav>
      <Link to="/">Home</Link>
      <Link to="/about">About</Link>
      <Link to="/products">Products</Link>
    </nav>
  );
}
```

### `<NavLink>` - Active Link Styling

`<NavLink>` is just like `<Link>`, but it automatically adds an `active` class to the element when its route is currently active. This is very useful for highlighting the current page in a navigation bar.

```jsx
import { NavLink } from "react-router";
import "./Navbar.css";

function Navbar() {
  return (
    <nav>
      <NavLink to="/" end>Home</NavLink>
      <NavLink to="/about">About</NavLink>
      <NavLink to="/products">Products</NavLink>
    </nav>
  );
}
```

```css
/* Navbar.css */
nav a {
  color: #555;
  text-decoration: none;
  padding: 0.5rem 1rem;
}

nav a.active {
  color: #007bff;
  font-weight: bold;
  border-bottom: 2px solid #007bff;
}
```

> The `end` prop on `<NavLink to="/">` prevents it from staying active on every page (since every route starts with `/`).

### `<Outlet>` - Rendering Child Routes

`<Outlet />` is a placeholder inside a **layout component** that tells React Router: _"render the matched child route here."_

Think of it like a window in a picture frame - the frame (layout) stays the same, but the view through the window changes.

```jsx
import { Outlet, Link } from "react-router";

function MainLayout() {
  return (
    <div>
      <header>
        <nav>
          <Link to="/">Home</Link>
          <Link to="/about">About</Link>
        </nav>
      </header>

      <main>
        {/* The matched child route renders here */}
        <Outlet />
      </main>

      <footer>
        <p>© 2025 My App</p>
      </footer>
    </div>
  );
}
```

---

## 4. Navigation Hooks

React Router provides several hooks to help you interact with routing programmatically - reading URL data or navigating without a `<Link>` click.

### `useParams` - Reading URL Parameters

When a route path contains a **dynamic segment** (like `:id` or `:slug`), `useParams` lets you read that value inside the component.

**Define a dynamic route:**

```jsx
<Route path="/products/:id" element={<ProductDetail />} />
```

**Read the param inside the component:**

```jsx
import { useParams } from "react-router";

function ProductDetail() {
  const { id } = useParams();

  // In a real app, you'd fetch from an API using this id
  return (
    <div>
      <h1>Product #{id}</h1>
      <p>Showing details for product with ID: {id}</p>
    </div>
  );
}
```

> `useParams` returns an object where each key matches a dynamic segment in the route path. If your path is `/posts/:category/:slug`, you'd destructure `const { category, slug } = useParams()`.

### `useNavigate` - Programmatic Navigation

`useNavigate` gives you a function you can call to navigate from within event handlers or effects - not just on `<Link>` clicks.

```jsx
import { useNavigate } from "react-router";

function LoginPage() {
  const navigate = useNavigate();

  function handleLogin() {
    // ...authenticate the user...
    navigate("/dashboard"); // redirect after login
  }

  function handleCancel() {
    navigate(-1); // go back one page in history (like clicking Back)
  }

  return (
    <div>
      <button onClick={handleLogin}>Log In</button>
      <button onClick={handleCancel}>Cancel</button>
    </div>
  );
}
```

**Common use cases for `useNavigate`:**

- Redirect after form submission.
- Redirect to login when a user isn't authenticated.
- Go back to the previous page.
- Skip straight to a success or error page after an async operation.

You can also pass a **state** object alongside the navigation, which the destination component can read:

```jsx
navigate("/checkout/success", { state: { orderId: "ORD-001" } });
```

### `useLocation` - Reading the Current URL

`useLocation` returns information about the current URL - the `pathname`, any `search` query parameters, and any `state` that was passed via `navigate()`.

```jsx
import { useLocation } from "react-router";

function SuccessPage() {
  const location = useLocation();

  // If navigated with: navigate("/success", { state: { orderId: "ORD-001" } })
  const { orderId } = location.state || {};

  // location.search would be "?session_id=abc123" for a URL like /success?session_id=abc123
  const queryParams = new URLSearchParams(location.search);
  const sessionId = queryParams.get("session_id");

  return (
    <div>
      <h1>Payment Successful!</h1>
      {orderId && <p>Order ID: {orderId}</p>}
      {sessionId && <p>Session: {sessionId}</p>}
    </div>
  );
}
```

**Summary of what `useLocation` returns:**

| Property   | Example Value          | What it is                                      |
| ---------- | ---------------------- | ----------------------------------------------- |
| `pathname` | `/checkout/success`    | The current URL path                            |
| `search`   | `?session_id=abc123`   | The query string (including `?`)                |
| `hash`     | `#section-2`           | The URL hash/anchor                             |
| `state`    | `{ orderId: "ORD-001" }` | Data passed via `navigate(path, { state: {} })` |

---

## 5. Nested Routes

### Why Nested Routes?

Most real apps share a common layout (navbar, sidebar, footer) across many pages. Without nested routes, you'd have to manually include `<Navbar />` and `<Footer />` inside every single page component, which leads to repetition and inconsistency.

**Nested routes** solve this by letting you define a **parent route** that renders a layout, and **child routes** that render inside that layout - via `<Outlet />`.

### Structure

```
/dashboard           → DashboardLayout + DashboardHome (index)
/dashboard/settings  → DashboardLayout + Settings
/dashboard/billing   → DashboardLayout + Billing
```

### Code Example

```jsx
// App.jsx
import { Routes, Route } from "react-router";
import MainLayout from "./layouts/MainLayout";
import Home from "./pages/Home";
import About from "./pages/About";
import DashboardLayout from "./layouts/DashboardLayout";
import DashboardHome from "./pages/DashboardHome";
import Settings from "./pages/Settings";
import Billing from "./pages/Billing";
import NotFound from "./pages/NotFound";

export default function App() {
  return (
    <Routes>
      {/* Public pages with shared layout */}
      <Route element={<MainLayout />}>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Route>

      {/* Dashboard nested routes */}
      <Route path="/dashboard" element={<DashboardLayout />}>
        <Route index element={<DashboardHome />} />
        <Route path="settings" element={<Settings />} />
        <Route path="billing" element={<Billing />} />
      </Route>

      <Route path="*" element={<NotFound />} />
    </Routes>
  );
}
```

```jsx
// layouts/DashboardLayout.jsx
import { Outlet, NavLink } from "react-router";

function DashboardLayout() {
  return (
    <div className="dashboard">
      <aside>
        <nav>
          <NavLink to="/dashboard" end>Overview</NavLink>
          <NavLink to="/dashboard/settings">Settings</NavLink>
          <NavLink to="/dashboard/billing">Billing</NavLink>
        </nav>
      </aside>

      <main>
        <Outlet /> {/* Child route renders here */}
      </main>
    </div>
  );
}

export default DashboardLayout;
```

> The `index` route matches the parent path exactly (`/dashboard`) and renders the default child - in this case `DashboardHome`. No `path` prop is needed on it.

---

## 6. Route Parameters (Dynamic Pages)

### The "Why"

Almost every real app has pages that depend on some ID or identifier:

- Product page: `/products/42`
- Blog post: `/posts/how-to-learn-react`
- Student profile: `/students/7`
- Invoice: `/invoices/INV-2025-001`

Instead of defining a route for every single product, you define **one parameterized route** and read the ID at render time.

### Defining and Using Dynamic Routes

```jsx
// In App.jsx
<Route path="/products/:id" element={<ProductDetail />} />
```

```jsx
// pages/ProductDetail.jsx
import { useParams, Link } from "react-router";
import { useEffect, useState } from "react";

function ProductDetail() {
  const { id } = useParams();
  const [product, setProduct] = useState(null);

  useEffect(() => {
    // Fetch the product from your API using the id
    fetch(`/api/products/${id}`)
      .then((res) => res.json())
      .then((data) => setProduct(data));
  }, [id]);

  if (!product) return <p>Loading...</p>;

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <p>Price: KSh {product.price}</p>
      <Link to="/products">← Back to Products</Link>
    </div>
  );
}

export default ProductDetail;
```

---

## 7. The 404 Not Found Page

### Why You Need It

Users mistype URLs, follow broken links, or land on routes that no longer exist. Without a catch-all route, they'll just see a blank screen - a terrible experience.

React Router uses `path="*"` as a wildcard that matches anything that wasn't matched by an earlier route. Always place it **last**.

```jsx
// pages/NotFound.jsx
import { Link } from "react-router";

function NotFound() {
  return (
    <div style={{ textAlign: "center", padding: "4rem" }}>
      <h1>404 - Page Not Found</h1>
      <p>The page you're looking for doesn't exist.</p>
      <Link to="/">Go back home</Link>
    </div>
  );
}

export default NotFound;
```

```jsx
// In App.jsx (always last)
<Route path="*" element={<NotFound />} />
```

---

## 8. The Biggest Real-World Gotcha: Routing + Deployment

### The Problem

Client-side routing works perfectly when you're navigating *inside* the app. But here's what happens when a user:

- Refreshes the browser on `/products/42`
- Or pastes a deep link like `/dashboard/settings` directly into the address bar

The browser sends that URL to the **server**. The server looks for a file at `/products/42` - which doesn't exist, because it's a client-side path. The server responds with a **404 error**, and the app never loads.

### The Fix: Server-Side URL Rewriting

You need to configure your web server to return `index.html` for **all** routes, and let React Router handle the routing client-side.

**Nginx config example:**

```nginx
server {
  listen 80;
  root /var/www/my-app;
  index index.html;

  location / {
    try_files $uri $uri/ /index.html;
  }
}
```

**Hosting platform equivalents:**

| Platform      | How to fix                                                          |
| ------------- | ------------------------------------------------------------------- |
| Vercel        | Automatic - no config needed for React apps                         |
| Netlify       | Add a `_redirects` file: `/* /index.html 200`                       |
| GitHub Pages  | Requires a workaround (custom 404.html redirecting to index)        |
| Nginx (VPS)   | Use `try_files $uri /index.html;` in server config                  |

> **This is one of the most common "works locally, breaks in production" issues for students.** Know why it happens and how to fix it.

---

## 9. Routing in Real-World Payment Flows

Routing is not just about navigation - it's a critical part of **multi-step user journeys**, especially payment flows.

### Typical E-Commerce Payment Flow

```
/products           → Browse products
/products/:id       → View product detail
/cart               → Review cart
/checkout           → Enter shipping + payment info
/checkout/processing → "Please wait..." loading screen
/checkout/success   → Confirmed! Show order summary
/checkout/cancel    → Payment was cancelled - try again
```

### Why Each Step Is Its Own Route

- Users should be able to **bookmark** the success page and return to it.
- The **Back button** should work sensibly.
- Each page has its own UI, logic, and data requirements.
- Payment providers (Stripe, Paystack, Flutterwave) often redirect back to specific URLs after payment - you need those routes to exist.

### Frontend vs. Backend Responsibilities

This is an important mindset to develop early:

| Responsibility              | Frontend (React)                              | Backend (Node/Express)                                     |
| --------------------------- | --------------------------------------------- | ---------------------------------------------------------- |
| Collect user info           | Form inputs, cart state                    |                                                          |
| Start a payment             | Call your backend API                      | Create payment session/intent with provider              |
| Redirect to provider        | `window.location.href = providerUrl`       |                                                          |
| Show processing/success UI  |                                             |                                                          |
| Receive & verify webhook    |                                             | Receives callback from Stripe/Paystack/Safaricom         |
| Mark order as paid          |                                             | Only after verifying the webhook                         |

> **The golden rule:** Never trust the frontend to confirm payment. The frontend can show a success page, but **backend verification via webhooks is the source of truth**.

### M-Pesa STK Push Flow (Conceptual)

```
1. User fills form on /checkout
2. Frontend calls: POST /api/payments/mpesa/stk-push  { phone, amount }
3. User receives push notification on their phone and confirms
4. Safaricom sends a callback to your backend
5. Backend verifies and marks order as paid
6. Frontend polls: GET /api/payments/status/:reference
7. On success → navigate("/checkout/success")
8. On failure → navigate("/checkout/failed")
```

**Reading a query param on the success page (Stripe-style):**

```jsx
// pages/CheckoutSuccess.jsx
import { useLocation } from "react-router";

function CheckoutSuccess() {
  const location = useLocation();
  const params = new URLSearchParams(location.search);
  const sessionId = params.get("session_id");

  return (
    <div>
      <h1>Payment Successful!</h1>
      <p>Your order has been confirmed.</p>
      {sessionId && <p>Reference: {sessionId}</p>}
    </div>
  );
}

export default CheckoutSuccess;
```

---

## 10. Recommended Project Structure

A clean folder structure makes routes easy to manage and scale:

```
src/
├── layouts/
│   ├── MainLayout.jsx      ← Shared navbar + footer
│   └── DashboardLayout.jsx ← Sidebar + outlet for dashboard pages
├── pages/
│   ├── Home.jsx
│   ├── About.jsx
│   ├── Products.jsx
│   ├── ProductDetail.jsx
│   ├── Cart.jsx
│   ├── Checkout.jsx
│   ├── CheckoutSuccess.jsx
│   ├── CheckoutCancel.jsx
│   ├── Dashboard.jsx
│   └── NotFound.jsx
├── components/
│   ├── Navbar.jsx
│   ├── Footer.jsx
│   └── ProductCard.jsx
└── App.jsx                 ← All routes defined here
```

**The key rule:** One file per page, pages live in `pages/`, reusable UI lives in `components/`. Routes are defined once in `App.jsx`.

---

## 11. Best Practices

- **Define all routes in one place** (`App.jsx`) - makes the route structure easy to see at a glance.
- **Use `<NavLink>` in navbars** instead of `<Link>` for automatic active-state styling.
- **Put the `path="*"` catch-all route last** - React Router matches from top to bottom and stops at the first match.
- **Never use `<a href>` for internal navigation** - always use `<Link>` or `<NavLink>`.
- **Use `useNavigate` for programmatic redirects** (after form submission, login, payment).
- **Use `useParams` when your route has dynamic segments** like `:id` or `:slug`.
- **Use `useLocation` when you need to read query params or state** passed alongside navigation.
- **Configure your deployment server** to return `index.html` for all routes - this is not optional for production.
- **Keep layouts in a `layouts/` folder** separate from `pages/` for clarity.

---

## 12. Practice Projects

### A) Portfolio Website

```
/           → Home (hero, intro)
/about      → Bio + skills
/projects   → Project list
/contact    → Contact form
*           → 404 Not Found
```

**What you practice:** Basic routing, shared layout, `<NavLink>` active states.

---

### B) Blog App

```
/posts          → List of all posts
/posts/:slug    → Individual post (read from params)
*               → 404 Not Found
```

**What you practice:** Dynamic params (`useParams`), fetching by ID/slug, content-first navigation.

---

### C) E-Commerce Frontend *(best for "real-world" readiness)*

```
/                   → Landing page
/products           → Product grid
/products/:id       → Product detail
/cart               → Cart review
/checkout           → Checkout form
/checkout/success   → Payment confirmed
/checkout/cancel    → Payment cancelled
*                   → 404 Not Found
```

**What you practice:** Everything above + payment UX flows, query params on success pages, programmatic navigation after async operations, and the frontend/backend responsibilities mindset.

---

## Quick Reference

| What you want to do                        | Tool to use                       |
| ------------------------------------------ | --------------------------------- |
| Define which component renders at a URL    | `<Route path="..." element={} />` |
| Navigate via a clickable link              | `<Link to="..." />`               |
| Navigate with active styling               | `<NavLink to="..." />`            |
| Render child routes inside a layout        | `<Outlet />`                      |
| Navigate programmatically (JS code)        | `useNavigate()`                   |
| Read a URL parameter (`:id`, `:slug`)      | `useParams()`                     |
| Read the current URL / query string        | `useLocation()`                   |

---

## Official Documentation

- [React Router - Installation](https://reactrouter.com/start/declarative/installation)
- [React Router - Routing & Nested Routes](https://reactrouter.com/start/declarative/routing)
- [react-router-dom on npm](https://www.npmjs.com/package/react-router-dom)
- [Fix React Router 404s on Nginx](https://oneuptime.com/blog/post/2025-12-16-react-router-404-nginx/view)
