# Week 14, Day 4: Layouts, Tailwind, and SEO

By the end of today, your shop will have proper nested layouts, a branded design system built on Tailwind, a dynamic `sitemap.xml` that Google can crawl, a `robots.txt`, and auto-generated Open Graph images per product. The shop will look and feel like something you would put your name next to.

**Prior-week concepts you will use today:**
- Tailwind basics (Week 8, Day 4)
- The App Router file conventions (Week 14, Days 1-3)
- Server Components fetching from Postgres

**Estimated time:** 3-4 hours

---

## Nested Layouts

Yesterday you used `app/layout.js` as the site-wide wrapper. Any folder can also have its own `layout.js`, and that layout wraps every page inside the folder. Create `app/products/layout.js`:

```jsx
// app/products/layout.js
import Link from "next/link";
import { query } from "@/lib/db";

export default async function ProductsLayout({ children }) {
  const { rows: categories } = await query(
    "SELECT DISTINCT category FROM products WHERE category IS NOT NULL ORDER BY category"
  );

  return (
    <div className="max-w-6xl mx-auto p-6 grid grid-cols-1 md:grid-cols-[200px_1fr] gap-8">
      <aside className="border-r pr-6">
        <h3 className="font-semibold mb-3">Categories</h3>
        <ul className="space-y-2 text-sm">
          <li><Link href="/products" className="hover:underline">All</Link></li>
          {categories.map((c) => (
            <li key={c.category}>
              <Link href={`/products/category/${c.category}`} className="hover:underline capitalize">
                {c.category}
              </Link>
            </li>
          ))}
        </ul>
      </aside>
      <div>{children}</div>
    </div>
  );
}
```

Now `/products`, `/products/nokia-105`, and `/products/category/phones` all get the category sidebar. The root layout (header, footer) still wraps everything, and the products layout is inserted between. Layouts nest from the outside in:

```
RootLayout (header + footer)
  ProductsLayout (category sidebar)
    ProductsPage / ProductDetailPage / CategoryPage
```

This is why Next.js's layout system is better than React Router's: layouts are co-located with the pages they wrap, data fetches inside them stream in parallel with page data, and navigation between pages inside the same layout does not re-render the layout. Clicking from one product to another does not re-fetch the category list.

---

## Design System Basics

Tailwind is a utility-first CSS framework. Out of the box, every class is a low-level style. Good teams extend the defaults in `tailwind.config.js` with their own palette, spacing scale, and components.

Edit `tailwind.config.js`:

```javascript
module.exports = {
  content: ["./app/**/*.{js,jsx}"],
  theme: {
    extend: {
      colors: {
        brand: {
          DEFAULT: "#1a1a1a",
          accent: "#16a34a",
          bg: "#faf9f6",
        },
      },
      fontFamily: {
        sans: ["var(--font-inter)", "system-ui", "sans-serif"],
      },
      borderRadius: {
        DEFAULT: "0.375rem",
      },
    },
  },
  plugins: [],
};
```

Now `bg-brand` gives you the brand colour, `bg-brand-accent` the accent, `bg-brand-bg` the soft cream background. Instead of hardcoding `#16a34a` in six places, you reference `bg-brand-accent` and change it once if the brand evolves.

### Load a web font

Next.js has `next/font/google` that self-hosts Google Fonts automatically -- no FOUT, no external requests. In `app/layout.js`:

```jsx
import { Inter } from "next/font/google";
import "./globals.css";

const inter = Inter({
  subsets: ["latin"],
  variable: "--font-inter",
});

export default function RootLayout({ children }) {
  return (
    <html lang="en" className={inter.variable}>
      <body className="font-sans bg-brand-bg text-brand">
        {/* ... */}
      </body>
    </html>
  );
}
```

Two things worth noting.

**`Inter({ variable: "--font-inter" })`** makes the font available as a CSS variable. The `tailwind.config.js` above picks it up via `fontFamily.sans`. Now every `font-sans` class (which is the default) renders in Inter.

**Self-hosted, not external.** `next/font/google` downloads the font files at build time and serves them from your own origin. Zero external fetches, zero tracking, zero "flash of unstyled text". This is a one-liner that takes most projects an hour to get right in CRA+Vite.

### Small reusable components

Create `app/components/Button.js`:

```jsx
export default function Button({ variant = "primary", className = "", children, ...props }) {
  const base = "inline-flex items-center justify-center px-6 py-3 rounded font-medium transition";
  const variants = {
    primary: "bg-brand text-white hover:opacity-90 disabled:opacity-50",
    secondary: "bg-white text-brand border border-brand hover:bg-gray-50",
    ghost: "text-brand hover:bg-gray-100",
  };
  return (
    <button className={`${base} ${variants[variant]} ${className}`} {...props}>
      {children}
    </button>
  );
}
```

One of the traps in Tailwind-heavy codebases is repeating long class strings. A handful of well-named components (`Button`, `Card`, `Input`, `Badge`) cut most of the repetition. Do not go overboard -- do not build a full component library on day one -- but a `<Button>` and a `<Card>` pay for themselves fast.

Use it in `AddToCartButton.js`:

```jsx
"use client";
import Button from "./Button";

export default function AddToCartButton({ product }) {
  // ... state code ...
  return (
    <Button disabled={product.stock === 0} onClick={handleClick}>
      {product.stock === 0 ? "Out of stock" : "Add to cart"}
    </Button>
  );
}
```

The Client Component imports the (Server-safe) `Button` just fine -- `Button` is a pure component with no server-only imports, so it can run on either side.

---

## A `robots.txt` and a Sitemap

Search engines find your shop through two files: `robots.txt` (which tells them what to crawl) and `sitemap.xml` (which lists all your pages).

Next.js App Router has conventions for both. Create `app/robots.js`:

```javascript
// app/robots.js
export default function robots() {
  return {
    rules: [
      { userAgent: "*", allow: "/", disallow: ["/api/", "/admin"] },
    ],
    sitemap: "https://yourdomain.com/sitemap.xml",
  };
}
```

Visiting `/robots.txt` now serves that file generated from the function. Create `app/sitemap.js`:

```javascript
// app/sitemap.js
import { query } from "@/lib/db";

export default async function sitemap() {
  const { rows: products } = await query("SELECT slug, created_at FROM products");

  const staticRoutes = [
    { url: "https://yourdomain.com/", lastModified: new Date() },
    { url: "https://yourdomain.com/products", lastModified: new Date() },
    { url: "https://yourdomain.com/about", lastModified: new Date() },
    { url: "https://yourdomain.com/contact", lastModified: new Date() },
  ];

  const productRoutes = products.map((p) => ({
    url: `https://yourdomain.com/products/${p.slug}`,
    lastModified: new Date(p.created_at),
  }));

  return [...staticRoutes, ...productRoutes];
}
```

Visit `/sitemap.xml`. You should see a proper XML sitemap with every product listed. Submit this to Google Search Console once you deploy and your products start showing up in search within a few days.

This is a big deal for small Kenyan e-commerce shops -- most never submit a sitemap, which is why they are invisible on Google. Ten lines of code put you ahead.

---

## Open Graph Images

When you paste a product URL into WhatsApp, Instagram, or Twitter, you want a preview card with a real image, not a generic fallback. Yesterday we set `openGraph.images` to the product's stored `image_url`. That works, but if the product does not have an image, you get nothing.

Next.js can generate Open Graph images on the fly. Create `app/products/[slug]/opengraph-image.js`:

```jsx
// app/products/[slug]/opengraph-image.js
import { ImageResponse } from "next/og";
import { query } from "@/lib/db";

export const size = { width: 1200, height: 630 };
export const contentType = "image/png";

export default async function Image({ params }) {
  const { rows } = await query(
    "SELECT name, price_cents FROM products WHERE slug = $1",
    [params.slug]
  );
  const product = rows[0];

  return new ImageResponse(
    (
      <div
        style={{
          width: "100%",
          height: "100%",
          background: "#faf9f6",
          display: "flex",
          flexDirection: "column",
          justifyContent: "center",
          alignItems: "center",
          fontFamily: "sans-serif",
        }}
      >
        <div style={{ fontSize: 64, fontWeight: 700, color: "#1a1a1a" }}>
          {product?.name || "Mctaba"}
        </div>
        <div style={{ fontSize: 48, color: "#16a34a", marginTop: 24 }}>
          KSh {product ? (product.price_cents / 100).toLocaleString() : ""}
        </div>
        <div style={{ fontSize: 24, color: "#888", marginTop: 40 }}>mctaba.co.ke</div>
      </div>
    ),
    { ...size }
  );
}
```

Restart the dev server. Visit `/products/nokia-105/opengraph-image.png` directly in the browser -- a real PNG appears, 1200x630, with the product name and price baked in.

Next.js picks this up automatically and uses it as the `og:image` for the product page. Share the URL on WhatsApp -- the preview card is the generated PNG, not your fallback. Every product now has a custom social preview without you shipping a single image file.

The one caveat: `ImageResponse` only supports a subset of CSS (flexbox layout, web-safe fonts, basic colors). No grids, no gradients beyond linear, no animations. Keep the design simple.

---

## Metadata Inheritance

You have `metadata` in `app/layout.js` (site-wide), `generateMetadata` in `app/products/[slug]/page.js` (per product), and you should also set a default for the shop section in `app/products/layout.js`:

```jsx
// app/products/layout.js (add to the top)
export const metadata = {
  title: {
    template: "%s | Mctaba Shop",
    default: "Shop | Mctaba",
  },
};
```

The `template` field is the trick: every page inside `/products/*` that sets its own `title: "Nokia 105"` will render as "Nokia 105 | Mctaba Shop" in the browser tab. No more manually appending " | Mctaba Shop" to every title. And if a page forgets to set a title at all, the `default` kicks in.

Put this on every route section and you end up with a consistent title structure across the whole site.

---

## Error Boundaries

Similar to `loading.js` and `not-found.js`, Next.js has `error.js` for when something blows up in a route. Create `app/products/error.js`:

```jsx
// app/products/error.js
"use client";

export default function Error({ error, reset }) {
  return (
    <div className="p-16 text-center">
      <h1 className="text-2xl font-bold">Something went wrong</h1>
      <p className="mt-4 text-gray-600">
        We could not load this page. Please try again in a moment.
      </p>
      <button
        onClick={() => reset()}
        className="mt-6 bg-brand text-white px-6 py-3 rounded"
      >
        Try again
      </button>
    </div>
  );
}
```

This has to be a Client Component (it uses `onClick` and receives a `reset` function). When an unhandled error happens inside `/products/*`, the error boundary renders this instead of the broken page. Clicking "Try again" calls `reset()` which re-renders the route. If the error was transient (a Postgres blip), it will succeed. If it was persistent, the user gets the error again.

Every route section should have an `error.js`. Missing one means users see a generic Next.js error page with a stack trace in dev, and a blank page in production.

---

## Accessibility Quick Pass

You have been writing JSX without thinking much about accessibility. Fix the obvious things now because they are cheap and the muscle memory matters.

- Every `<img>` or `<Image>` needs an `alt` attribute. Empty string `alt=""` is correct for decorative images; a description is correct for content images.
- Every `<button>` and `<a>` needs readable text content, not just an icon.
- Form inputs need `<label>` associated by `htmlFor` + `id`.
- Headings should go `h1 -> h2 -> h3` in order on each page. Do not skip levels.
- Colour contrast: `text-gray-600 on bg-brand-bg` -- check it in the browser devtools. If the ratio is under 4.5, darken the text.

Run `npm install --save-dev @axe-core/react` and inside a dev-only file initialise it -- it prints accessibility issues to the console in dev mode. Fix what it flags.

Accessibility is not a Kenya-specific concern, but low-vision and blind users exist in every market, and a catalogue that does not work for a screen reader is a catalogue that excludes customers. Screen readers are free and growing in Swahili language support. Do not ignore them.

---

## Checkpoint

1. `/products` has the sidebar layout with categories; it is present on every `/products/*` route.
2. The browser tab shows "Nokia 105 | Mctaba Shop" without the code duplicating the suffix on every page.
3. Inter font loads. No flash of unstyled text.
4. `/robots.txt` and `/sitemap.xml` return valid content. The sitemap lists every product.
5. `/products/nokia-105/opengraph-image.png` returns a real PNG with the product name baked in.
6. Pasting the product URL into https://www.opengraph.xyz/ shows the generated OG image in the preview.
7. Forcing an error inside a product page (throw inside the component) shows the `error.js` boundary with a retry button.
8. `@axe-core/react` shows no critical accessibility violations on any page.

Commit:

```bash
git add .
git commit -m "feat: layouts design system og images sitemap"
```

---

## What You Learned

- Nested layouts share data fetching and do not re-render on sub-navigation.
- Tailwind's config is where the brand lives; components are where consistency comes from.
- `next/font/google` self-hosts Google fonts in one line.
- Next.js generates `robots.txt`, `sitemap.xml`, and Open Graph images from code.
- Error boundaries prevent route failures from nuking the whole site.
- Accessibility is cheap if you do it while you build, expensive if you retrofit later.

Tomorrow is the week recap, and the weekend project is a catalogue MVP with products, categories, search, and polished design -- no checkout yet, that is Week 15.
