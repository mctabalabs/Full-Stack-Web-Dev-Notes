# Week 5 - Day 1 Assignment

## Title
Object Literals, Property Access, and the Shape of Data

## Overview
Week 5 moves from primitives to objects -- the main shape your data will take for the rest of the programme. Today you practise creating, reading, updating, and deleting object properties using dot notation, bracket notation, destructuring, the spread operator, and short-hand property definitions. By the end of this assignment you will have a small "inventory" app that stores and manipulates product objects in memory.

## Learning Objectives Assessed
- Create object literals with shorthand properties and methods
- Access nested properties safely
- Use the spread operator to copy and merge objects
- Delete properties and iterate keys/values

## Prerequisites
- Week 4 completed
- Day 1 reading: [Object Literals & Property Access.md](./Object%20Literals%20&%20Property%20Access.md)

## AI Usage Rules

**Ratio this week:** 70% manual / 30% AI
**Habit:** AI as code reviewer -- paste your working code, ask for critique, decide what to accept. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Asking AI to review your object code and suggest improvements AFTER your version works.
- **NOT ALLOWED FOR:** Generating objects for you. Asking AI "what fields should I have?".
- **AUDIT REQUIRED:** Yes. Add a "Code review decisions" section to `AI_AUDIT.md`.

## Tasks

### Task 1: Create a product object

**What to do:**
In `js-lab/week5-day1/main.js`, create a product object with at least these fields:

```javascript
const product = {
  id: 1,
  name: "Notebook",
  priceCents: 35000,
  inStock: true,
  tags: ["stationery", "paper"],
  dimensions: { width: 14, height: 21, unit: "cm" },
  describe() {
    return `${this.name} -- KSh ${this.priceCents / 100}`;
  }
};
```

Then:
1. Access `product.name` with dot notation
2. Access `product["priceCents"]` with bracket notation
3. Access the nested `product.dimensions.width`
4. Call `product.describe()`

**Expected output:**
Four distinct `console.log` calls showing each access pattern works.

### Task 2: Spread and merge

**What to do:**
1. Create a second product object that is a copy of the first but with a different id and name. Use the spread operator:

```javascript
const product2 = { ...product, id: 2, name: "Pen" };
```

2. Verify that modifying `product2.tags.push("ink")` does NOT mutate `product.tags`. (Spoiler: it does, because spread is shallow. This is a learning moment.)
3. Fix the shallow-copy problem by deep-cloning: `const product3 = JSON.parse(JSON.stringify(product));`

**Expected output:**
A short comment in your code showing you noticed and fixed the shallow-copy bug.

### Task 3: Destructure with rename and default

**What to do:**
Write a function that takes a product object and destructures it in the signature with renamed variables and defaults:

```javascript
function summarise({ name: productName, priceCents, inStock = false }) {
  return `${productName}: ${inStock ? "available" : "out of stock"} at KSh ${priceCents / 100}`;
}

console.log(summarise(product));
console.log(summarise({ name: "Pen", priceCents: 5000 })); // uses default inStock
```

**Expected output:**
Function working for both calls.

### Task 4: Simple inventory

**What to do:**
Create an `inventory` object that maps product ids to product objects:

```javascript
const inventory = {};
function addProduct(p) { inventory[p.id] = p; }
function removeProduct(id) { delete inventory[id]; }
function listProducts() { return Object.values(inventory); }
```

Add at least three products. Remove one. List the remaining products. Use `Object.keys`, `Object.values`, and `Object.entries` at least once each.

**Expected output:**
Three added, one removed, two remaining, all visible in the console.

### Task 5: Code review cycle

**What to do:**
After your inventory works, paste it to an AI tool and ask: "Review this as a senior engineer would. What would you change and why? Do not rewrite -- just list issues."

Read the suggestions. In `AI_AUDIT.md`, for each suggestion:
- State whether you accept or reject it
- Explain why in one sentence

Minimum 3 accept/reject decisions.

**Expected output:**
Updated `AI_AUDIT.md` with a "Code review decisions" entry.

## Stretch Goals (Optional - Extra Credit)

- Add a `calculateTotal(inventory)` function that sums `priceCents * quantity` across items.
- Use `Object.freeze(product)` and try to mutate it. Note the error.
- Use computed property names: `const key = "name"; const obj = { [key]: "Notebook" };`

## Submission Requirements

- **What to submit:** Repo with `js-lab/week5-day1/`, updated `AI_AUDIT.md`.
- **Deadline:** End of Day 1

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Product object with all access patterns | 15 | Dot, bracket, nested, method call all working. |
| Spread + shallow-copy fix | 15 | Student noticed the shallow copy bug and documented/fixed it. |
| Destructuring with rename and default | 15 | Function works for both calls. |
| Inventory with add/remove/list | 20 | All three operations working. `Object.keys/values/entries` all used. |
| Code review decisions | 20 | At least 3 accept/reject decisions with reasoning. |
| Clean commits | 10 | Conventional messages. |
| Code quality | 5 | No `var`, descriptive names. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Thinking spread is a deep copy.** It is not. Nested objects and arrays are shared. Use `structuredClone()` or `JSON.parse(JSON.stringify())` for deep copies.
- **Accessing `obj.someMissing.field`.** If `obj.someMissing` is undefined, this crashes. Use optional chaining: `obj?.someMissing?.field`.
- **Accepting every AI review suggestion.** This week's habit is about judgement. Some suggestions are taste, some are correctness, some are wrong. Disagree when disagreement is warranted.

## Resources

- Day 1 reading: [Object Literals & Property Access.md](./Object%20Literals%20&%20Property%20Access.md)
- Week 5 AI boundaries: [../ai.md](../ai.md)
- MDN Working with objects: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_objects
