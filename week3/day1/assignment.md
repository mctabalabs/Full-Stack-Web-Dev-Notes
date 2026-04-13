# Week 3 - Day 1 Assignment

## Title
Set Up JavaScript, Write Your First Script, and Debug In The Browser

## Overview
Week 3 opens the programming week. Up to now your pages have been static -- HTML for structure, CSS for presentation. Today JavaScript joins the party. Your assignment is to set up a proper JavaScript project, write your first handful of statements, run them both in the browser and in Node, and practise reading errors in the console.

## Learning Objectives Assessed
- Link an external JavaScript file to an HTML page correctly
- Run JavaScript in the browser console and in Node
- Use `console.log` to inspect values
- Set a breakpoint in Chrome DevTools and inspect paused state
- Distinguish between syntax errors and runtime errors

## Prerequisites
- Completed Weeks 1-2
- Node.js installed (verified by `node --version`)
- Your portfolio repo still exists

## AI Usage Rules

**Ratio this week:** 80% manual / 20% AI
**Habit:** The 15-minute rule -- try for 15 minutes before you prompt. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Explaining an error message after you already read it twice and searched the text in a search engine. Explaining why `var` and `let` behave differently.
- **NOT ALLOWED FOR:** Generating JavaScript for you to copy. Writing your first script. Debugging your code for you within the first 15 minutes.
- **AUDIT REQUIRED:** Yes. From Week 3 onwards, `AI_AUDIT.md` must include a "What I tried in the 15 minutes" entry for every AI interaction.

## Tasks

### Task 1: Create a JavaScript project folder

**What to do:**
1. In your portfolio repo (or a new repo `js-lab`), create a folder `js-lab/day1/`.
2. Inside, create two files: `index.html` and `main.js`.
3. Link `main.js` from `index.html` using a `<script>` tag just before the closing `</body>` tag:

```html
<script src="main.js"></script>
```

Write the HTML boilerplate by hand.

**Expected output:**
Opening `index.html` in Live Server and opening DevTools shows the page loading `main.js` in the Network tab.

### Task 2: Write your first five JavaScript statements

**What to do:**
Inside `main.js`, type each of the following lines yourself (no copy-paste). After each line, open the browser console and verify the output is what you expect:

```javascript
console.log("Hello from JavaScript");
const name = "Your Name";
console.log(`Hi, my name is ${name}`);
let year = 2026;
year = year + 1;
console.log(`Next year will be ${year}`);
const marathon = { weeks: 30, phase: "foundations" };
console.log(marathon);
console.log(typeof marathon);
```

**Expected output:**
Five distinct console outputs visible in the DevTools Console tab.

### Task 3: Run the same script in Node

**What to do:**
Copy the same code (not the HTML, just the JavaScript) into a new file `main.node.js`. In the terminal, run it:

```bash
node js-lab/day1/main.node.js
```

Observe the output in the terminal. It should be the same as in the browser console for these simple statements.

**Expected output:**
Terminal output matching the browser console output. A screenshot of the terminal saved as `day1-node.png`.

### Task 4: Set a breakpoint in DevTools

**What to do:**
1. In the browser, open DevTools and click the "Sources" tab.
2. Find `main.js` in the file tree on the left.
3. Click the line number next to your `year = year + 1;` line. A blue arrow appears (that is a breakpoint).
4. Reload the page. Execution pauses at the breakpoint.
5. Hover over `year` in the code view -- you see its current value.
6. Click the "Resume" button (or press F8) to continue.
7. Take a screenshot showing the paused state and save it as `day1-breakpoint.png` in `assets/`.

**Expected output:**
Screenshot showing the breakpoint, the pause indicator, and the value of `year` at that moment.

### Task 5: Break something on purpose, then read the error

**What to do:**
Temporarily change `const name = "Your Name";` to `const name = "Your Name"` (remove the semicolon) AND change `let year = 2026;` to `let year = ;` (remove the value).

Reload the page. The Console should show an error. Read it carefully. In a new file `day1-notes.md` write:

1. The exact error message
2. Which line number it pointed to
3. Whether you think it is a syntax error or a runtime error and why
4. How you fixed it

Restore the code to working state after.

**Expected output:**
`day1-notes.md` with the error text and your analysis. No AI-generated explanation -- your own words.

## Stretch Goals (Optional - Extra Credit)

- Use `console.table(marathon)` for the object and observe the nicer output format.
- Open a second `main.js` in a different folder and discover how browsers cache JS files. Use hard-reload (`Cmd/Ctrl + Shift + R`) to bust the cache.
- Investigate the `debugger` statement: add `debugger;` to your code and see how DevTools reacts.

## Submission Requirements

- **What to submit:** Repo link, `js-lab/day1/` files, screenshots `day1-node.png` and `day1-breakpoint.png`, `day1-notes.md`, `AI_AUDIT.md`.
- **Where to submit:** Week 3 submissions channel
- **Deadline:** End of Day 1
- **Format:** All inside your chosen repo

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Project set up and JS linked to HTML | 15 | Correct folder structure, `<script>` tag in HTML, file loads without 404. |
| Five statements working in browser | 15 | All five console outputs visible in DevTools. Correct syntax. |
| Same code running in Node | 10 | Terminal screenshot confirms Node execution. |
| Breakpoint set and inspected | 15 | Screenshot shows paused execution. Student can explain what they saw. |
| Error analysis in day1-notes.md | 15 | Error text quoted exactly, line number correct, syntax-vs-runtime classification correct, fix described. |
| AI Audit with 15-minute rule entries | 15 | Every AI interaction includes "What I tried" before the prompt. No empty "What I tried" fields. |
| Clean commit history | 10 | Conventional prefixes. Three-plus commits for Day 1. |
| Code quality | 5 | No lingering broken code. `const` used where possible. `var` not used. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Forgetting the `<script>` tag.** If your JS does not seem to run, check DevTools' Network tab -- the file should load. If it is not in the tab, your HTML is not referencing it.
- **Using `var` instead of `let`/`const`.** `var` has historical scoping quirks you do not need. Always prefer `const`, fall back to `let`, avoid `var`.
- **Asking AI before 15 minutes of trying.** This week the rule is firm. Your audit must prove you tried first.

## Resources

- Day 1 reading: [introduction and dev environment.md](./introduction%20and%20dev%20environment.md)
- Week 3 AI boundaries: [../ai.md](../ai.md)
- MDN JavaScript Guide: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide
- Chrome DevTools Sources tutorial: https://developer.chrome.com/docs/devtools/javascript/
