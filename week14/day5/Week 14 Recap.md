# Week 14, Day 5: Week Recap

This week was the jump from React+Vite to Next.js. You built a catalogue frontend that reads from Postgres directly, has dynamic routes, generates sitemaps and Open Graph images, and ships as a single deployable unit. Next week adds the cart and checkout state machine; Week 16 wires up payments and ships Project 3.

---

## The Four Shifts This Week

### 1. From client-heavy SPA to Server Components first

The default in the App Router is the server. You only drop into the client when you need interactivity, and you do so in small islands. This is a reversal of the React+Vite mental model where everything was a client component.

The test for "server or client?": if the component handles a click, uses state, or reads `window` -- client. Otherwise -- server. 80% of your components will be server.

### 2. From API routes to direct database access

Server Components can call `query("SELECT ...")` directly. No `/api/products` endpoint. No `useEffect`. No loading guard. Data flows straight from Postgres into JSX in one function.

This is not "skipping best practices". It is a different shape. There is still a server (Node), there is still a database (Postgres), there is still a frontend (React). What is missing is the HTTP layer between server-side data and server-side rendering -- because they are both on the server.

### 3. From React Router to file-based routing with layouts

`app/products/[slug]/page.js` is the whole route definition. No router config file. Layouts nest automatically. Loading and error states are file conventions. Metadata is a function export. Everything you previously had to wire up manually is now a file name.

### 4. From "ship it" to "ship it well"

Sitemap, robots.txt, Open Graph images, dynamic metadata, error boundaries, accessibility -- all things you could do in any framework but had to remember. In Next.js they are all file conventions, so you cannot forget them. The bar for polish is higher because the cost of polish is lower.

---

## Self-Review Questions

**Mental model**
1. What is a Server Component? What is a Client Component? When do you pick each?
2. Can a Client Component import a Server Component? What about the reverse?
3. How do you pass data from a Server Component to a Client Component?
4. What does `"use client"` do at the top of a file, and what does it affect?

**Routing**
5. How do you add a page at `/about/team`?
6. How do you add a dynamic route like `/products/[slug]`?
7. What does `loading.js` do, and when does Next.js show it?
8. What does `notFound()` do?

**Data fetching**
9. How do you query Postgres from a Server Component without an API route?
10. What does `unstable_cache` do, and when would you use it?
11. What does `revalidatePath` do?
12. What is Incremental Static Regeneration (ISR) and when is it the right choice?

**Production polish**
13. What goes in `app/robots.js` and `app/sitemap.js`?
14. How does Next.js generate Open Graph images, and what are the CSS limitations?
15. What is the `template` field in a `metadata` object, and what problem does it solve?

Target: 12/15 correct without looking.

---

## Peer Coding Session

Pair up and pick a track.

### Track A: Migrate one page of the Week 12 CRM to Next.js

The Week 12 CRM dashboard runs on React + Vite. Pick one page (e.g., the leads list) and port it to a new Next.js route backed by the same Postgres. Report back on:
- Line count before vs after.
- Which boilerplate disappeared.
- What was harder in Next.js than in Vite.

This is the "try it on a real codebase" reality check.

### Track B: Add a second admin-only layout

Create `app/admin/layout.js` with a protected sidebar (products, orders, customers). Hook up auth by checking a cookie and redirecting to login if missing. The products page inside it lists all products from the database with an "Edit" button (not functional yet -- just the link).

This previews next week's territory.

### Track C: A real sitemap

Pick your actual domain (a free Vercel subdomain works). Deploy the shop. Submit the sitemap to Google Search Console. Report on what Search Console says about your pages 24 hours later. This is homework -- you may not finish in the session.

### Track D: Lighthouse audit

Run Chrome DevTools -> Lighthouse on your shop. Aim for 90+ on Performance, Accessibility, Best Practices, and SEO. Note which suggestions are cheap (alt tags, robots.txt) and which are expensive (image optimisation, bundle splitting). Fix the cheap ones.

---

## Weekend Prep

This weekend is a **catalogue MVP**, not an e-commerce store. No cart, no checkout. Just the shoppable front half: browse, search, filter, detail pages, nice design. The brief is in `week14/weekend/project.md`.

Before you log off:

1. Pick the product niche you will build for. It does not have to be tech -- kitchenware, kitenge fabric, kids' shoes, secondhand phones, whatever.
2. Have 10-15 real products in mind, with names, prices, and short descriptions. Real is better than placeholder.
3. Find or take 10-15 photos. You can use your phone for this. The weekend you spend on product photos pays off at demo time more than any amount of polish on code.

Week 15 we add the cart. Week 16 we ship Project 3. Next.js is the vehicle; commerce is the destination.
