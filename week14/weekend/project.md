# Week 14 Weekend Project: Catalogue MVP

Build a real browsable catalogue for a real product niche you care about. By Monday morning the site must let a visitor browse all products, filter by category, search, view a detail page, and see a WhatsApp preview card when the URL is shared. No cart, no checkout -- those are next week.

**Estimated time:** 6-8 hours.

**Deadline:** Monday morning before Week 15 Day 1.

---

## Requirements

### Content (this is the hard part)

- **10-15 real products**, not placeholders. Pick a niche: kitenge fabric, kids' sneakers, secondhand phones, power tools, baby formula, second-hand textbooks, anything.
- Each product has: name, slug, price (in cents), description (2-3 sentences), image, stock count, category.
- At least **3 categories**.
- Photos are your own or properly credited -- no copyright violations.

### Pages

- `/` -- home page with brand tagline, featured products, category tiles.
- `/products` -- all products, grid layout, sidebar with categories.
- `/products/[slug]` -- detail page with image, name, description, price, stock, "Contact to buy on WhatsApp" button (placeholder href, opens wa.me link).
- `/products/category/[cat]` -- filtered listing by category.
- `/search?q=...` -- working search by name and description.
- `/about`, `/contact` -- static pages with your story.

### Technical

- Uses Next.js 14 App Router.
- All data comes from Postgres via Server Components. No API routes.
- `sitemap.xml` and `robots.txt` work.
- Dynamic Open Graph images per product (name, price, category baked into the image).
- Dynamic metadata (title, description, og:image) per product page.
- `loading.js` for the products list.
- `not-found.js` for bad slugs.
- `error.js` for route failures.
- Tailwind config has a custom brand colour palette that matches your niche (boda boda project should not use pastel pink).
- Lighthouse score >= 90 on Performance, Accessibility, SEO, Best Practices for the home page.

### Shareable

- Pasting a product URL in WhatsApp shows a nice preview card.
- The site works on a 360px-wide mobile screen.
- The site works with JavaScript disabled (progressive enhancement).
- No horizontal scroll at any breakpoint.

---

## Grading Rubric (100 pts)

| Area | Points |
|---|---|
| Real product content | 15 |
| Catalogue browsing + filtering + search | 20 |
| Product detail pages | 15 |
| Dynamic metadata + OG images | 15 |
| Design quality and mobile responsiveness | 15 |
| Performance (Lighthouse >= 90) | 10 |
| SEO basics (sitemap, robots, metadata) | 5 |
| Accessibility (no axe critical violations) | 5 |

---

## Submission Checklist

- [ ] `npm run dev` starts cleanly.
- [ ] `npm run build` completes with no errors.
- [ ] `/products` shows your grid.
- [ ] `/products/[slug]` shows every product correctly.
- [ ] Search finds products by name and by description word.
- [ ] Clicking "Contact on WhatsApp" opens wa.me with a pre-filled message.
- [ ] Pasting a product URL in the WhatsApp web preview shows a preview card.
- [ ] Running Lighthouse on the home page: scores >= 90 in all four categories.
- [ ] The site renders correctly at 360px width.
- [ ] `README.md` describes your niche, what is live, and what you would add next.

---

## Hints

**Photos matter more than code.** A mediocre site with great photos out-demos a great site with bad photos. Spend Saturday morning taking real photos or downloading CC-licensed ones. Crop them square, compress them under 200KB each, and name them to match your slugs.

**Start from data, not design.** Write the SQL `INSERT` statements for your 10 products first, then build the pages. The alternative -- designing UI without real data -- always leads to rework.

**Do not build a cart.** It is seductive. Resist. A half-built cart is worse than no cart. Stick to browsing.

**Use a single font and two colours.** Resist the urge to "make it pop". Restraint reads as quality. Look at your favourite niche shop websites -- they use one font, two colours, and lots of whitespace. Copy that.

**WhatsApp contact button.** `wa.me/254712000000?text=Hi, I'm interested in ${product.name}` is a one-line feature that delivers real business value. Every product page must have it.

**Deploy to Vercel Sunday evening.** Even if you do not hand in the URL, the deploy pressure forces you to fix the things you have been skipping (env vars, `next.config.js`, image hosts). Sign up for a free Vercel account and push your repo; it deploys in under five minutes.

---

## If You Finish Early

- **Add a "New arrivals" section** on the home page sorted by `created_at DESC`, limit 4.
- **Per-product schema.org JSON-LD** -- Google loves this for product rich results. Add `<script type="application/ld+json">` inside the product detail page with `Product` schema. Ten lines of code, bumps your SEO.
- **A small admin-only form** to add a product. No styling needed. Behind a hardcoded password for now; real auth comes in Week 15.
- **Internationalisation** -- Swahili labels for all UI strings. Same `t()` pattern as Week 13's USSD app, applied to JSX.

Good luck. Bring the live URL, screenshots, and a two-sentence story about your niche to Monday's session. Week 15 turns this catalogue into an actual shop.
