# Week 1 - Day 2 Assignment

## Title
Build Your First Web Page and Explain How It Reached the Browser

## Overview
Today you learned how the internet moves bytes between computers and how HTML structures the content a browser paints on screen. Your assignment is to build your very first real web page -- by hand, no AI -- and to write a short explanation of the journey every character on that page took from your laptop to your browser window. This ties the theory of HTTP and DNS to something you can see working on your own screen.

## Learning Objectives Assessed
- Create a valid HTML5 document from scratch, typing every tag
- Differentiate between the Internet, the World Wide Web, and HTML in your own words
- Trace the request/response cycle when a browser loads a local HTML file
- Use VS Code and Live Server to preview a page while editing

## Prerequisites
- Completed Day 1 assignment (dev environment and first commit)
- Completed Day 2 reading: [what is the internet.md](./what%20is%20the%20internet.md)
- A working clone of your `my-first-project` repo from Day 1

## AI Usage Rules

**Ratio this week:** 90% manual / 10% AI
**Habit:** Ask AI to explain, not to do.

- **ALLOWED FOR:** Asking AI to explain any term you already met in the Day 2 reading (for example, "what does TCP actually do?"). The question must name what you already read and what you did not understand.
- **NOT ALLOWED FOR:** Generating the HTML for your page. Generating the written explanation. Asking AI "what should I put on my first web page?".
- **AUDIT REQUIRED:** No. Week 1 still has no formal AI Audit.

## Tasks

### Task 1: Build index.html from scratch

**What to do:**
Inside your `my-first-project` repository (created on Day 1), overwrite `index.html` with a hand-typed page that includes at minimum:

1. A full HTML5 boilerplate: `<!DOCTYPE html>`, `<html lang="en">`, `<head>`, `<body>`.
2. A `<meta charset="UTF-8">` and `<meta name="viewport" ...>` inside the head.
3. A `<title>` that reads "My First Web Page".
4. Inside the body: one `<h1>` with your full name, one `<p>` introducing yourself in 2-3 sentences, and one `<p>` describing one thing you learned from today's Day 2 reading.

You must type every character yourself. No copy-paste from AI. Copy-paste from the Day 2 reading is allowed, but you should retype at least the boilerplate once so your fingers learn it.

**Expected output:**
A working `index.html` file that opens in the browser via Live Server and shows your name, a short intro, and one reflection.

### Task 2: Preview the page with Live Server

**What to do:**
1. In VS Code, right-click inside `index.html` and pick "Open with Live Server".
2. Your default browser opens to an address like `http://127.0.0.1:5500/index.html`.
3. Edit one word inside the file in VS Code and save it. The browser should refresh automatically.
4. Take a screenshot of the browser showing your page and the URL bar.

**Expected output:**
A screenshot named `day2-livepreview.png` committed to your repo under an `assets/` folder.

### Task 3: Write a request-journey explanation in your own words

**What to do:**
Create a new file called `day2-journey.md` inside your repository. In 6-10 sentences, describe what happens step-by-step when someone types a URL into a browser and presses Enter. Your explanation must use these terms in your own words: **URL, DNS, IP address, HTTP, HTML, browser, server**. Do not paste the Day 2 reading. Say it in your own words, even if the wording is simpler.

At the bottom of the file, add a single sentence that answers: "How is the Internet different from the World Wide Web?"

**Expected output:**
`day2-journey.md` committed to the repo.

### Task 4: Commit your work in at least two steps

**What to do:**
Use two separate commits for this assignment so your history is readable:

1. First commit message: `feat: hand-built index.html for day 2`
2. Second commit message: `docs: add day 2 journey explanation and screenshot`

Run `git log --oneline` after both commits and confirm you see both messages.

**Expected output:**
Screenshot of `git log --oneline` showing the two commits. Push both commits to GitHub.

## Stretch Goals (Optional - Extra Credit)

- Add a third `<p>` that contains a clickable `<a>` link to a resource you found useful while studying HTML. The link must open in a new tab.
- Create a second page called `about.html` in the same folder and link to it from `index.html`. Both pages must have valid HTML5 boilerplate.
- Draw a small diagram on paper of the request/response flow from browser to server and back. Photograph it and commit the image to `assets/`.

## Submission Requirements

- **What to submit:**
  - Link to your updated `my-first-project` GitHub repository.
  - The screenshot from Task 2 visible in the repo under `assets/`.
  - The `day2-journey.md` file visible in the repo.
- **Where to submit:** Post the repo link in the Week 1 submissions channel.
- **Deadline:** End of Day 2, before your Day 3 session begins.
- **Format:** All files inside your existing `my-first-project` repository. Do not create a new repo.

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Valid HTML5 document | 20 | Doctype present, `<html lang="en">`, `<head>` with meta and title, `<body>` with required elements. Validates via the HTML validator at validator.w3.org with no errors. |
| Hand-typed (not copy-pasted) | 15 | Student must walk a facilitator through any line of their index.html on request. If a student cannot explain a line, it costs points here. |
| Live Server preview screenshot | 10 | Clear screenshot showing the page, the URL in the bar, and the expected text visible. |
| Journey explanation clarity | 20 | All required terms used in context. Explanation reads as the student's own words. No copy-paste. Final one-sentence Internet vs Web answer is correct and concise. |
| Clean commit history | 15 | Two separate commits with the required messages. Screenshot of `git log --oneline` matches. Both pushed to GitHub. |
| File organisation | 10 | Files in the correct locations (`index.html` at root, screenshot in `assets/`, journey file at root). No random junk committed. |
| Code quality | 10 | Proper indentation in the HTML. No trailing whitespace. No accidental `console.log` or stray text. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Forgetting the `<!DOCTYPE html>` line.** Without it, browsers fall back to "quirks mode" and behave inconsistently. It must be the very first line of the file.
- **Using the wrong file extension.** A file named `index.txt` or `index` will not render in the browser. It must be `index.html`.
- **Committing only once at the end.** Your Git history teaches you more than your final code does. Commit in small, meaningful steps -- that is why Task 4 requires two commits.

## Resources

- Day 2 reading: [what is the internet.md](./what%20is%20the%20internet.md)
- Week 1 AI boundaries: [../ai.md](../ai.md)
- HTML validator: https://validator.w3.org/
- MDN Web Docs HTML basics: https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/HTML_basics
