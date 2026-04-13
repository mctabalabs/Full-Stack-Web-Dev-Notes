# Week 3 - Day 2 Assignment

## Title
Variables, Data Types, and Template Literals in Practice

## Overview
Today you met the core building blocks of JavaScript: variables (`let`, `const`, `var`), primitive types (string, number, boolean, null, undefined), and template literals. Your assignment is a set of small coding drills that prove you can declare the right variable for the right value, check a type at runtime, and format strings cleanly.

## Learning Objectives Assessed
- Choose between `const`, `let`, and (never) `var`
- Identify each primitive type and use `typeof` to verify
- Use template literals for string composition
- Understand the difference between `null` and `undefined`
- Recognise dynamic typing and its surprises

## Prerequisites
- Completed Day 1 assignment
- Day 2 reading: [variables and data types.md](./variables%20and%20data%20types.md)

## AI Usage Rules

**Ratio this week:** 80% manual / 20% AI
**Habit:** 15-minute rule. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Explaining one surprising outcome (e.g., `typeof null`) after you tried to explain it yourself first.
- **NOT ALLOWED FOR:** Solving drill exercises for you.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Type ten declarations yourself

**What to do:**
In `js-lab/day2/main.js`, write and run the following declarations exactly as described. No copy-paste.

1. A `const` holding your full name as a string
2. A `const` holding your age as a number
3. A `const` holding your current level of JavaScript comfort as a boolean
4. A `let` holding a placeholder that you will later reassign (start with `undefined` or `null` -- your choice, but justify it in a comment)
5. A `const` holding an array of at least three strings (your favourite languages so far)
6. A `const` holding an object with two properties: `city` and `country`
7. A template literal that composes a sentence using the name and city variables
8. A `typeof` check on at least three of your variables with `console.log`
9. A reassignment of the `let` from step 4 to a string value, logged before and after
10. A line that *intentionally* tries to reassign one of your `const` variables -- observe the error in the console, then comment the line out

**Expected output:**
`main.js` runs without errors (after commenting out the intentional const-reassignment). Console output visible and explained.

### Task 2: typeof null investigation

**What to do:**
Add this line to your file:

```javascript
console.log(typeof null);
```

The output will surprise you. In `day2-notes.md`, write 3-5 sentences explaining:
- What the output was
- Why it is considered a famous JavaScript quirk
- What the "correct" type of `null` should logically be

Try to explain this in your own words first. If stuck after 15 minutes, ask AI and record it in `AI_AUDIT.md`.

**Expected output:**
`day2-notes.md` with your analysis.

### Task 3: Build a personal profile generator

**What to do:**
Write a function called `buildProfile` (the function syntax was covered briefly; if not, you will master it next week -- the simple form `function buildProfile(args) { ... }` is enough).

```javascript
function buildProfile(name, age, city, languages) {
  return `Name: ${name}
Age: ${age}
City: ${city}
Languages: ${languages.join(", ")}`;
}

console.log(buildProfile("Wanjiru", 22, "Nairobi", ["English", "Kiswahili", "JavaScript"]));
```

**Expected output:**
The console prints a multi-line profile string.

### Task 4: Drills

**What to do:**
In the same file, complete these drills. Write, test, and commit:

1. Declare a `let` variable. Assign it three different types in sequence (number, string, boolean). Log `typeof` after each assignment. Write a comment observing that JavaScript is dynamically typed.
2. Create two strings and concatenate them using `+`. Then do the same join using a template literal. Compare the readability in a comment.
3. Create an object with a property whose value is a function. Call that function from the object using dot notation.

**Expected output:**
Three separate sections in `main.js`, each with its drill and a one-line comment observation.

## Stretch Goals (Optional - Extra Credit)

- Use `Number.isInteger`, `Number.isFinite`, `Number.isNaN` on at least three different values and explain the results.
- Compare `==` and `===` using a surprising case like `0 == false` and `"1" == 1`. Explain in a comment why `===` is always preferred.
- Use `Object.freeze` on an object and try to mutate a property. Observe.

## Submission Requirements

- **What to submit:** Repo with `js-lab/day2/` files, `day2-notes.md`, updated `AI_AUDIT.md`.
- **Where to submit:** Week 3 submissions channel
- **Deadline:** End of Day 2
- **Format:** Standard

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Ten declarations correct | 20 | All ten tasks completed with correct variable choices (`const` default, `let` only when reassigned). |
| `typeof null` analysis | 15 | day2-notes.md accurately describes the quirk in student's own words. |
| buildProfile function works | 15 | Correct multi-line output using template literals. |
| Three drills completed | 20 | Dynamic typing observed, string concat vs template literal compared, object method called. |
| AI Audit with 15-minute rule | 15 | Every AI interaction has "What I tried" before the prompt. |
| Clean commit history | 10 | At least 2 commits with conventional messages. |
| Code quality | 5 | `const` used where possible, no `var`, comments explain non-obvious choices. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Using `var`.** There is no reason to use `var` in modern JavaScript. Use `const` by default, `let` when you need reassignment.
- **Hardcoding instead of using template literals.** String concatenation with `+` is painful to read once you have more than two pieces. Template literals are always the better choice for multi-part strings.
- **Confusing `null` with `undefined`.** `undefined` = "I have not been assigned a value yet". `null` = "I have been assigned, and the value is intentionally nothing". Pick the right one deliberately.

## Resources

- Day 2 reading: [variables and data types.md](./variables%20and%20data%20types.md)
- Week 3 AI boundaries: [../ai.md](../ai.md)
- MDN typeof operator: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof
