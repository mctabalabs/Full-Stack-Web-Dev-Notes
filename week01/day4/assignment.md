# Week 1 - Day 4 Assignment

## Title
Ship a Day 4 Pre-Weekend Checkpoint: A Semantic Blog Post Page With Real Media

## Overview
Today you met semantic HTML (`<header>`, `<main>`, `<section>`, `<article>`, `<aside>`, `<footer>`) and HTML5 media (`<audio>`, `<video>`, `<iframe>`). Your assignment today is the week's pre-weekend polish day: build a complete semantic blog post page with at least one piece of real embedded media, and run through a checklist that proves your Week 1 portfolio is ready to expand in the weekend project. Day 4 is "ship it" day -- everything must work and be committed before you close your laptop tonight.

## Learning Objectives Assessed
- Structure a page using semantic landmarks correctly (`<header>`, `<main>`, `<article>`, `<footer>`)
- Distinguish between `<section>` and `<article>` based on content meaning
- Embed at least one media type (audio, video, or iframe) with appropriate attributes
- Avoid "div-itis" by replacing generic containers with semantic alternatives where they exist
- Complete a pre-submission checklist that proves the page is production-ready for the weekend project

## Prerequisites
- Completed Day 1, Day 2, and Day 3 assignments
- Completed Day 4 readings: [Semantics and Page Structure.md](./Semantics%20and%20Page%20Structure.md) and [HTML5 Media.md](./HTML5%20Media.md)
- Your `my-first-project` repo is still clean and pushed to GitHub

## AI Usage Rules

**Ratio this week:** 90% manual / 10% AI
**Habit:** Ask AI to explain, not to do.

- **ALLOWED FOR:** Asking AI to explain the difference between `<section>` and `<article>` after you have already tried to articulate it yourself. Asking AI to explain what the `preload` attribute does after you read the Day 4 notes.
- **NOT ALLOWED FOR:** Generating the semantic structure of your blog post. Generating the blog post content (the writing is yours). Choosing which media type to embed.
- **AUDIT REQUIRED:** No. Week 1 still has no formal AI Audit.

## Tasks

### Task 1: Create post.html with a real semantic layout

**What to do:**
Inside your `my-first-project` repo, create a new file called `post.html`. Build it with the following structure, typing every tag:

```
html
 └── body
      ├── header (contains site title and nav linking to index, bio, post)
      ├── main
      │    └── article
      │         ├── header (contains h1 title and published date)
      │         ├── section (the main body of the post, multiple paragraphs)
      │         └── footer (contains author name and a list of tags)
      ├── aside (related posts or a short about-the-author blurb)
      └── footer (site footer with copyright)
```

Write a real blog post about something you learned in Week 1. Minimum 3 paragraphs of actual prose (not lorem ipsum, not AI-generated). Give it a clear title in the `<h1>`.

**Expected output:**
`post.html` rendered via Live Server, passing the W3C validator with zero errors.

### Task 2: Embed at least one piece of media

**What to do:**
Pick ONE of the following three options and add it inside the `<section>` body of your article:

1. **Audio:** An `<audio controls preload="metadata">` element with at least one `<source>`. If you do not have audio of your own, use a free CC-licensed file. The `controls` attribute is mandatory.
2. **Video:** A `<video controls poster="...">` element with at least one `<source>` and a poster image. Use any free short clip.
3. **iframe:** An `<iframe>` embedding a YouTube video related to your post. Include the `title` attribute for accessibility. Do not embed Google Maps or anything that requires an API key.

Regardless of choice, the media must render in the browser and play or display correctly when the page loads.

**Expected output:**
The media element visible on the page, and a screenshot named `day4-media.png` in `assets/` showing it rendered.

### Task 3: Replace div-itis in your earlier pages

**What to do:**
Open your Day 2 `index.html` and your Day 3 `bio.html`. Find any `<div>` elements (if any). For each `<div>`, decide whether a semantic element would be more appropriate (`<header>`, `<main>`, `<section>`, `<article>`, `<aside>`, `<footer>`, `<nav>`). Replace the `<div>` with the right semantic tag -- or, if the `<div>` is genuinely just a styling wrapper with no meaning, leave it and write a one-line HTML comment above it explaining why you kept it.

**Expected output:**
Updated `index.html` and `bio.html` with at least one semantic replacement (or a comment explaining why you kept a `<div>`). Re-run the validator on both files.

### Task 4: Pre-weekend checklist

**What to do:**
Create a file called `CHECKLIST.md` at the root of your repo. Copy the following checklist into it and tick each box as you verify each item. Every unticked box costs points.

```markdown
## Week 1 Day 4 Pre-Weekend Checklist

- [ ] index.html has a valid HTML5 boilerplate and renders in Live Server
- [ ] bio.html has a valid HTML5 boilerplate and renders in Live Server
- [ ] post.html has a valid HTML5 boilerplate and renders in Live Server
- [ ] All three pages pass the W3C validator with zero errors
- [ ] Every image in the repo has a meaningful alt attribute
- [ ] Every external link has target="_blank" and rel="noopener"
- [ ] post.html uses at least five distinct semantic elements
- [ ] post.html embeds at least one working media element
- [ ] index.html has a nav linking to bio.html and post.html
- [ ] bio.html has a nav linking back to index.html
- [ ] post.html has a nav linking to index.html and bio.html
- [ ] All pages committed and pushed to GitHub
- [ ] Repo README has been updated with a short description of the project
```

**Expected output:**
`CHECKLIST.md` committed with all boxes honestly checked. If any box is unchecked, you write one line underneath explaining why and what you will do in the weekend to fix it.

### Task 5: Commit with meaningful messages

**What to do:**
Your Day 4 work should be split into at least three commits:

1. `feat: add post.html with semantic layout and embedded media`
2. `refactor: replace divs with semantic elements on index and bio`
3. `docs: add day 4 pre-weekend checklist`

Push all three commits to GitHub.

**Expected output:**
`git log --oneline` shows all three commits. GitHub shows them in your repo.

## Stretch Goals (Optional - Extra Credit)

- Add a second `<article>` to your post as a "related post" teaser in the `<aside>`, fully structured.
- Add an HTML `<table>` to your bio with a skills matrix (skill, level, years of experience). Every column must have a `<th>`.
- Add a short `<audio>` clip of yourself introducing your portfolio in addition to the main media on post.html.

## Submission Requirements

- **What to submit:**
  - Updated `my-first-project` repository link showing `post.html`, the updated `index.html` and `bio.html`, and the `CHECKLIST.md` file.
  - Screenshot `assets/day4-media.png` showing your embedded media rendered in the browser.
- **Where to submit:** Post the repo link in the Week 1 submissions channel.
- **Deadline:** End of Day 4. You must complete this assignment before starting the weekend project.
- **Format:** All files committed into your existing repo. Do not create new repositories.

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Semantic structure of post.html | 25 | All seven landmark elements used correctly (header, nav, main, article, section, aside, footer). `<article>` contains an `<h1>`. The distinction between `<section>` and `<article>` is respected. |
| Working embedded media | 15 | One media element correctly configured with required attributes (controls, source, poster or title). Media actually plays or renders. |
| Div-itis refactor | 10 | At least one `<div>` replaced with a semantic alternative on prior pages. If none existed, a one-line comment confirms that. |
| Pre-weekend checklist honest and complete | 15 | All boxes honestly ticked. Unticked boxes come with a written explanation. |
| Validator pass on all three pages | 10 | All of index.html, bio.html, and post.html pass validator.w3.org with zero errors. |
| Content quality | 10 | Blog post content is 3+ real paragraphs on a topic you learned. No AI-generated prose. |
| Clean commit history | 10 | Three meaningful commits with the required message shapes. All pushed. |
| Code quality | 5 | Proper indentation. No accidental junk files. No secrets. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Using `<section>` where `<article>` is correct.** If the content makes sense on its own if "ripped out" (a blog post, a news story, a comment), it is an `<article>`. If it is a logical part of a larger whole, it is a `<section>`. When in doubt, ask: "Would this make sense on its own?" Yes -> article. No -> section.
- **Using `<main>` more than once per page.** There must be exactly one `<main>` per page. The validator will flag more than one.
- **Forgetting the `controls` attribute on `<audio>` or `<video>`.** Without `controls`, the user has no way to play or pause. Always include it unless you are writing JavaScript to control it (which you are not this week).

## Resources

- Day 4 reading: [Semantics and Page Structure.md](./Semantics%20and%20Page%20Structure.md)
- Day 4 reading: [HTML5 Media.md](./HTML5%20Media.md)
- Week 1 AI boundaries: [../ai.md](../ai.md)
- Week 1 weekend project brief: [../weekend/project.md](../weekend/project.md)
- MDN semantic elements: https://developer.mozilla.org/en-US/docs/Glossary/Semantics#semantics_in_html
- W3C validator: https://validator.w3.org/
- Free media for testing: https://www.pexels.com/videos/ and https://freesound.org/
