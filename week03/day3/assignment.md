# Week 3 - Day 3 Assignment

## Title
Operators, Expressions, and Truthy/Falsy Logic

## Overview
Today you met the operators JavaScript uses to compare, compute, and combine values. Arithmetic, comparison, logical, and assignment -- all tools you need for real programs. Your assignment is to run a battery of drills that train your eye for the subtle differences (`==` vs `===`, short-circuit evaluation, type coercion) that trip up every new JS developer.

## Learning Objectives Assessed
- Use arithmetic, comparison, and logical operators accurately
- Distinguish loose (`==`) from strict (`===`) equality and always default to strict
- Recognise truthy and falsy values in JavaScript
- Use short-circuit evaluation (`&&`, `||`) for default values
- Explain type coercion in two or three specific cases

## Prerequisites
- Days 1-2 completed

## AI Usage Rules

**Ratio:** 80/20. **Habit:** 15-minute rule. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Explaining a surprising coercion after you already wrote the code.
- **NOT ALLOWED FOR:** Generating the drills.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Arithmetic calculator

**What to do:**
In `js-lab/day3/main.js`, write a small "receipt calculator" using arithmetic operators:

```javascript
const price = 1500;        // cost per item in KSh
const quantity = 3;
const taxRate = 0.16;      // 16% VAT

const subtotal = price * quantity;
const tax = subtotal * taxRate;
const total = subtotal + tax;

console.log(`Subtotal: KSh ${subtotal}`);
console.log(`Tax: KSh ${tax}`);
console.log(`Total: KSh ${total.toFixed(2)}`);
```

Type this yourself. Then change the inputs three times and re-run to confirm the math works.

**Expected output:**
Three runs with different inputs, each producing correct totals.

### Task 2: Truthy/falsy table

**What to do:**
Create an array of at least 10 values that span truthy and falsy: `[0, 1, "", "hello", null, undefined, NaN, [], {}, false, true]`. Loop over them with `forEach` and log:

```javascript
const values = [0, 1, "", "hello", null, undefined, NaN, [], {}, false, true];
values.forEach((v) => {
  console.log(`${JSON.stringify(v)} is ${Boolean(v) ? "truthy" : "falsy"}`);
});
```

In `day3-notes.md`, list which values surprised you and why.

**Expected output:**
Console output with 10+ lines classifying each value. Reflection in `day3-notes.md`.

### Task 3: `==` vs `===` drills

**What to do:**
Write at least 5 comparisons that demonstrate the difference:

```javascript
console.log(0 == false);     // true (coerces)
console.log(0 === false);    // false (strict)
console.log("" == 0);        // true
console.log("" === 0);       // false
console.log(null == undefined);  // true
console.log(null === undefined); // false
```

For each pair, add a one-line comment explaining what JavaScript did. Commit to always using `===` going forward.

**Expected output:**
5+ comparison pairs with explanatory comments.

### Task 4: Short-circuit defaults

**What to do:**
Use `||` for default values and `&&` for conditional logging:

```javascript
const username = "";  // empty string
const displayName = username || "Anonymous";
console.log(`Hello, ${displayName}`);

const isLoggedIn = true;
isLoggedIn && console.log("Welcome back!");
```

Change the values and re-run. Understand that `||` uses the first truthy, `&&` returns the first falsy or the last truthy.

**Expected output:**
Two working examples, each demonstrating the pattern.

### Task 5: Fix three broken expressions

**What to do:**
Add these three broken lines to your file. Each one has a subtle bug. Find and fix each one. No AI for the first 15 minutes. Document what you found in `day3-notes.md`.

```javascript
// BROKEN 1: This should compute 25% of 200 but it does not
const discount = 200 * 25;

// BROKEN 2: This should print "You are an adult" for age 18
const age = 18;
if (age > 18) console.log("You are an adult");

// BROKEN 3: This should concatenate but it adds as numbers
const a = "5";
const b = 3;
console.log(a + b);
```

**Expected output:**
Three fixed lines with explanations in `day3-notes.md` of what was wrong and how you fixed each.

## Stretch Goals (Optional - Extra Credit)

- Use the nullish coalescing operator `??` instead of `||` and explain the difference using `0 ?? "default"` vs `0 || "default"`.
- Use the ternary operator to rewrite one of your `if` statements.
- Research what `parseInt("abc", 10)` returns and why.

## Submission Requirements

- **What to submit:** Repo with `js-lab/day3/` files, `day3-notes.md`, `AI_AUDIT.md` updated.
- **Where:** Week 3 submissions channel
- **Deadline:** End of Day 3

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Arithmetic calculator with three inputs | 15 | Correct math, three different inputs tested. |
| Truthy/falsy table | 15 | 10+ values classified, surprises reflected in notes. |
| `==` vs `===` drills with comments | 20 | 5+ pairs. Every comment accurate. |
| Short-circuit defaults | 15 | Two working examples. |
| Three broken expressions fixed | 15 | Each bug found in under 15 minutes of manual effort. No AI used before that. |
| AI Audit with 15-minute rule | 15 | "What I tried" populated for every prompt. |
| Clean commits and code quality | 5 | Conventional messages. No copy-paste from AI. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Using `==` to "be flexible".** The flexibility is coercion, and coercion is a bug farm. Use `===` always.
- **Forgetting operator precedence.** `2 + 3 * 4` is 14, not 20. When in doubt, use parentheses.
- **Not testing edge cases.** Age = 18 should be "adult" if your condition is `>=`, but `> 18` excludes 18. Read your conditions carefully.

## Resources

- Day 3 reading: [operators and expressions.md](./operators%20and%20expressions.md)
- Week 3 AI boundaries: [../ai.md](../ai.md)
- MDN equality comparisons: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness
