# Week 1 - Day 3 Assignment

## Title
Ship a Hand-Coded HTML Student Bio Page With Lists, Links, and Images

## Overview
Today you met the core content tags of HTML: headings, paragraphs, lists, links, images, and text formatting. Your assignment is to extract the Day 3 mini project ("Simple Student Bio") and expand it into a polished, accessible, properly-structured bio page that lives in your existing GitHub repository. This is the first assignment where your page must pass basic accessibility rules and a real HTML validator -- no shortcuts.

## Learning Objectives Assessed
- Use headings (`<h1>` through `<h3>`) correctly and only once per heading level where appropriate
- Build ordered and unordered lists with proper `<li>` nesting
- Embed images with meaningful `alt` text for accessibility
- Add external links that open in a new tab using the `target="_blank"` attribute safely (with `rel="noopener"`)
- Use `<strong>` and `<em>` semantically, not decoratively

## Prerequisites
- Completed Day 1 and Day 2 assignments
- Completed Day 3 reading: [getting started with HTML.md](./getting%20started%20with%20HTML.md) and [common HTML tags.md](./common%20HTML%20tags.md)
- Your `my-first-project` repository is still cloned and working

## AI Usage Rules

**Ratio this week:** 90% manual / 10% AI
**Habit:** Ask AI to explain, not to do.

- **ALLOWED FOR:** Asking AI to explain a specific tag or attribute after you already read about it in the Day 3 notes. Example: "The notes said `target='_blank'` should use `rel='noopener'`. What does noopener protect me from?"
- **NOT ALLOWED FOR:** Generating any HTML for your page. Generating your bio prose. Asking AI which skills to list.
- **AUDIT REQUIRED:** No. Week 1 still has no formal AI Audit.

## Tasks

### Task 1: Create bio.html from the Day 3 mini project

**What to do:**
Inside your `my-first-project` repo, create a new file called `bio.html`. Starting from nothing (no copy-paste from AI, no copy-paste from previous files), write:

1. The full HTML5 boilerplate (doctype, `<html lang="en">`, `<head>`, `<body>`).
2. A `<title>` that reads "Bio - Your Name".
3. An `<h1>` with your full name.
4. An `<img>` of yourself (or a placeholder from `https://placehold.co/300x300`) with a meaningful `alt` attribute describing the image.
5. An `<h2>` titled "About Me" followed by a paragraph of 3-4 sentences.
6. An `<h2>` titled "Skills" followed by a `<ul>` with at least 5 `<li>` items.
7. An `<h2>` titled "My Learning Journey" followed by an ordered list (`<ol>`) of at least 3 steps showing what you have learned this week in order.
8. An `<h2>` titled "Favourite Resources" with at least 3 external links, each opening in a new tab. Each `<a>` must have `target="_blank"` and `rel="noopener"`.

**Expected output:**
A working `bio.html` file that renders in the browser via Live Server. The validator at validator.w3.org must return zero errors.

### Task 2: Link your bio to your index.html

**What to do:**
Open the `index.html` from Day 2. Add a new paragraph near the top with a link: `<a href="bio.html">Read my bio</a>`. Make sure the link is plain (no target attribute) because you want users to stay inside your site for this one.

**Expected output:**
Clicking the link on `index.html` opens `bio.html`. Clicking the browser back button returns to `index.html`.

### Task 3: Use strong and em correctly

**What to do:**
Inside your "About Me" paragraph, highlight one phrase with `<strong>` (a phrase of real importance -- for example, the country you are from or the field you are studying) and one phrase with `<em>` (a phrase with emphasis rather than importance -- for example, a word you want the reader to notice). In a comment `<!-- -->` next to each one, explain in one short sentence why you chose that tag over the other.

**Expected output:**
The rendered page shows one word bolded (via `<strong>`) and one italicised (via `<em>`). The HTML source has a comment next to each explaining the reasoning.

### Task 4: Validate your HTML

**What to do:**
1. Open https://validator.w3.org/#validate_by_input.
2. Paste the full source of `bio.html` into the text box and click "Check".
3. If you get errors, read each one, fix the HTML, and re-validate until you have zero errors and zero warnings.
4. Take a screenshot of the "Document checking completed. No errors or warnings to show." message.

**Expected output:**
A screenshot named `day3-validation.png` committed under `assets/`.

### Task 5: Explain one tag choice to your rubber duck

**What to do:**
At the bottom of `bio.html`, inside an HTML comment, write 2 sentences explaining why you used `<ul>` for Skills and `<ol>` for Learning Journey. The answer is about semantic meaning -- one is ordered, one is not -- but it has to be in your own words.

**Expected output:**
A clearly visible HTML comment at the bottom of the file containing your reasoning.

## Stretch Goals (Optional - Extra Credit)

- Add a `<blockquote>` tag with a quote that inspires you. Cite the author after the quote using `<cite>`.
- Add a `<details>` / `<summary>` collapsible section titled "Goals for the Marathon" with three bullet goals inside.
- Add a second image with a different `alt` and wrap it in an `<a>` so clicking the image takes you to a related external page.

## Submission Requirements

- **What to submit:**
  - Updated `my-first-project` repository link showing `bio.html` and the updated `index.html` link.
  - `assets/day3-validation.png` showing a clean W3C validator pass.
- **Where to submit:** Post the repo link in the Week 1 submissions channel.
- **Deadline:** End of Day 3, before your Day 4 session begins.
- **Format:** The new files must be inside your existing repo, not a new one.

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| bio.html structure | 20 | All eight required elements present (title, h1, img with alt, h2+p, h2+ul, h2+ol, h2+links, all links with target and rel). Valid HTML5 boilerplate. |
| Accessible images and links | 15 | Every `<img>` has meaningful alt text (not "image" or empty). Every external `<a>` uses `rel="noopener"` with `target="_blank"`. |
| Correct use of `<strong>` and `<em>` | 10 | Student uses each tag semantically and can explain the choice via the required comments. |
| HTML validator pass | 15 | Screenshot shows zero errors and zero warnings from validator.w3.org. |
| Internal link between index and bio | 10 | Clicking "Read my bio" navigates to `bio.html` without opening a new tab. |
| Bio content quality | 10 | About Me is a real paragraph, Skills list has at least 5 items, Learning Journey list has at least 3. No lorem ipsum. No AI-sounding bio prose. |
| Clean commit history | 10 | At least two commits for this day. Messages are meaningful. Repo pushed. |
| Code quality | 10 | Proper indentation. Comments used where required. No random junk files committed. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Skipping `alt` text or writing `alt="image"`.** Alt text is read by screen readers and shown if the image fails to load. An empty or meaningless alt is worse than no image. Describe what the image shows.
- **Nesting lists incorrectly.** `<li>` must be a direct child of `<ul>` or `<ol>`. Never put raw text between `<ul>` and `<li>`. If the validator complains about "element not allowed as child", this is usually the cause.
- **Opening too many tabs with `target="_blank"`.** Only add it to links that leave your site. Internal navigation (like `index.html` to `bio.html`) should stay in the same tab.

## Resources

- Day 3 reading: [getting started with HTML.md](./getting%20started%20with%20HTML.md)
- Day 3 reading: [common HTML tags.md](./common%20HTML%20tags.md)
- Week 1 AI boundaries: [../ai.md](../ai.md)
- W3C HTML validator: https://validator.w3.org/
- MDN alt attribute guide: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img#alt
- MDN rel="noopener": https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/rel/noopener
