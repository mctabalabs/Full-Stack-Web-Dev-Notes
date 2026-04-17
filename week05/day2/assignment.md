# Week 5 - Day 2 Assignment

## Title
Constructor Functions, ES6 Classes, and Building a BankAccount

## Overview
Today you learned how to define "classes" in JavaScript using both the old-school constructor function approach and the ES6 `class` syntax. Your assignment builds the same concept -- a `BankAccount` -- two ways and compares them. This is the closest JavaScript gets to traditional OOP.

## Learning Objectives Assessed
- Write a constructor function with methods on its prototype
- Write the same thing as an ES6 class
- Use `new` to instantiate objects correctly
- Define instance methods, static methods, getters, setters
- Understand how `this` binds inside class methods

## Prerequisites
- Day 1 completed
- Day 2 reading: [Constructor Functions & ES6 Classes.md](./Constructor%20Functions%20&%20ES6%20Classes.md)

## AI Usage Rules

**Ratio:** 70/30. **Habit:** AI as code reviewer. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Reviewing your class code for OOP anti-patterns after you finished the class.
- **NOT ALLOWED FOR:** Generating the class structure for you.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: BankAccount as a constructor function

**What to do:**
In `js-lab/week5-day2/constructor.js`, write:

```javascript
function BankAccount(owner, openingBalance = 0) {
  this.owner = owner;
  this.balanceCents = openingBalance;
  this.transactions = [];
}

BankAccount.prototype.deposit = function(amountCents) {
  this.balanceCents += amountCents;
  this.transactions.push({ type: "deposit", amountCents });
};

BankAccount.prototype.withdraw = function(amountCents) {
  if (amountCents > this.balanceCents) {
    throw new Error("Insufficient funds");
  }
  this.balanceCents -= amountCents;
  this.transactions.push({ type: "withdraw", amountCents });
};

BankAccount.prototype.getBalance = function() {
  return this.balanceCents / 100;
};

const acc = new BankAccount("Wanjiru", 100000);
acc.deposit(50000);
acc.withdraw(30000);
console.log(acc.getBalance()); // 1200
console.log(acc.transactions);
```

Type it yourself.

**Expected output:**
Working constructor function with `deposit`, `withdraw`, `getBalance`, and a transactions log.

### Task 2: Same thing as an ES6 class

**What to do:**
In `js-lab/week5-day2/classic.js`, rewrite the BankAccount as an ES6 class with identical behaviour:

```javascript
class BankAccount {
  constructor(owner, openingBalance = 0) {
    this.owner = owner;
    this.balanceCents = openingBalance;
    this.transactions = [];
  }

  deposit(amountCents) {
    // ...
  }

  withdraw(amountCents) {
    // ...
  }

  get balance() {
    return this.balanceCents / 100;
  }
}
```

Include one **getter** (`get balance()`), one **static method** (`BankAccount.minimumOpeningBalance()` returning 1000 cents), and one **setter** (`set owner(newName)` that uppercases the name before storing).

**Expected output:**
Class file with all four (constructor, method, getter, setter, static).

### Task 3: Compare the two files

**What to do:**
Create `comparison-notes.md`. In 5-8 sentences, compare the constructor function version to the class version. Specifically answer:
1. Which is shorter?
2. Which feels easier to read?
3. What does the class syntax give you that the constructor function does not (getters, setters, static methods, etc.)?
4. Is the class version "really" different under the hood?

Your own words. No AI prose.

**Expected output:**
`comparison-notes.md` committed.

### Task 4: Forgetting `new`

**What to do:**
Try calling your constructor without `new`:

```javascript
const acc = BankAccount("Wanjiru", 100000); // no new
console.log(acc); // undefined
```

In strict mode (which is the default in ES6 modules and classes), this throws. Try it both with the constructor function and with the class. Document the difference in behaviour in `comparison-notes.md`.

**Expected output:**
Notes include the "forgetting `new`" finding.

### Task 5: AI code review

**What to do:**
Paste your class version to AI. Ask: "Review this BankAccount class as a senior engineer would. What are three improvements I could make? Do not rewrite -- just list them."

Evaluate each suggestion. Accept, reject, or "need more context". Record in `AI_AUDIT.md`.

**Expected output:**
New entry in `AI_AUDIT.md` with accept/reject decisions.

## Stretch Goals (Optional - Extra Credit)

- Use private fields (`#balance`) to prevent direct mutation of the balance from outside the class.
- Add a `history` method that returns a formatted string of all transactions.
- Write a simple test suite that makes 10 deposits and withdrawals and asserts the final balance is correct.

## Submission Requirements

- **What to submit:** Repo with both implementations, `comparison-notes.md`, updated `AI_AUDIT.md`.
- **Deadline:** End of Day 2

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Constructor function BankAccount | 20 | deposit, withdraw, getBalance, transactions array -- all working. Prototype methods used. |
| ES6 class BankAccount | 25 | constructor, method, getter, setter, static -- all present. Behaviour matches constructor version. |
| Comparison notes | 15 | Four questions answered. Student's own words. |
| "Forgetting new" finding | 10 | Difference noted in writing. |
| AI code review decisions | 20 | At least three suggestions evaluated with reasoning. |
| Clean commits | 5 | Conventional messages. |
| Code quality | 5 | Descriptive names, consistent style. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Putting methods inside the constructor.** `this.deposit = function() {...}` inside the constructor works but creates a new copy of the method for every instance. Use the prototype (old school) or class methods (new school) so there is one shared copy.
- **Forgetting `new`.** Classes throw on this; constructor functions silently set properties on the wrong `this` (either global or undefined in strict mode).
- **Using classes where a plain object would do.** If you have a one-off data bag with no behaviour, a plain object literal is simpler than a class.

## Resources

- Day 2 reading: [Constructor Functions & ES6 Classes.md](./Constructor%20Functions%20&%20ES6%20Classes.md)
- Week 5 AI boundaries: [../ai.md](../ai.md)
- MDN Classes: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes
