# Week 5 - Day 3 Assignment

## Title
Prototype Chain and Inheritance With `extends`

## Overview
Yesterday you built a `BankAccount`. Today you learn how JavaScript shares methods across instances via the prototype chain, and how to extend a base class into a specialised one using `extends` and `super`. Your assignment is to add a `SavingsAccount` and a `CurrentAccount` that share the base BankAccount behaviour but each add their own twist.

## Learning Objectives Assessed
- Explain the prototype chain in your own words
- Use `extends` to create a subclass
- Call `super()` in a subclass constructor
- Override a method in a subclass
- Understand when inheritance is the right tool and when composition is better

## Prerequisites
- Day 1 and Day 2 completed

## AI Usage Rules

**Ratio:** 70/30. **Habit:** AI as code reviewer. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Reviewing your inheritance chain for design flaws.
- **NOT ALLOWED FOR:** Generating the subclasses.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: SavingsAccount extends BankAccount

**What to do:**
In `js-lab/week5-day3/accounts.js`, import or recreate your BankAccount class from Day 2. Then create:

```javascript
class SavingsAccount extends BankAccount {
  constructor(owner, openingBalance, interestRate = 0.05) {
    super(owner, openingBalance);
    this.interestRate = interestRate;
  }

  addInterest() {
    const interest = Math.floor(this.balanceCents * this.interestRate);
    this.balanceCents += interest;
    this.transactions.push({ type: "interest", amountCents: interest });
  }
}

const s = new SavingsAccount("Wanjiru", 500000, 0.08);
s.deposit(100000);
s.addInterest();
console.log(s.getBalance());
console.log(s.transactions);
```

**Expected output:**
SavingsAccount instance can deposit, withdraw, addInterest, and show balance.

### Task 2: CurrentAccount with an override

**What to do:**
Create a `CurrentAccount` class that extends `BankAccount` and overrides `withdraw` to allow going into overdraft up to KSh 5,000:

```javascript
class CurrentAccount extends BankAccount {
  constructor(owner, openingBalance) {
    super(owner, openingBalance);
    this.overdraftLimit = 500000; // KSh 5000 in cents
  }

  withdraw(amountCents) {
    if (amountCents > this.balanceCents + this.overdraftLimit) {
      throw new Error("Overdraft limit exceeded");
    }
    this.balanceCents -= amountCents;
    this.transactions.push({ type: "withdraw", amountCents });
  }
}
```

Test by withdrawing more than the balance but within the overdraft limit.

**Expected output:**
CurrentAccount allowing controlled negative balances.

### Task 3: Explore the prototype chain

**What to do:**
Add these lines and observe what they print:

```javascript
const s = new SavingsAccount("Amina", 100000);
console.log(s instanceof SavingsAccount); // true
console.log(s instanceof BankAccount);    // true
console.log(Object.getPrototypeOf(s).constructor.name); // SavingsAccount
console.log(Object.getPrototypeOf(Object.getPrototypeOf(s)).constructor.name); // BankAccount
```

Draw (in text or photograph) the prototype chain: `s -> SavingsAccount.prototype -> BankAccount.prototype -> Object.prototype -> null`. Save in `prototype-chain.md`.

**Expected output:**
`prototype-chain.md` with the sketch and a short explanation.

### Task 4: Inheritance vs composition trade-off

**What to do:**
In `design-notes.md`, answer this question in 4-6 sentences in your own words: "When should I use inheritance (extends) vs composition (having an object own another object as a property)?" Use the BankAccount example. Would you model a `LoanAccount` as a subclass of `BankAccount`, or as a separate class that `has-a` BankAccount? Defend your choice.

**Expected output:**
`design-notes.md` committed.

### Task 5: AI review of your class hierarchy

**What to do:**
Paste your `SavingsAccount` and `CurrentAccount` to AI and ask: "Is this inheritance hierarchy a good idea? Are there any anti-patterns? What would you suggest differently?"

Record the discussion in `AI_AUDIT.md` with accept/reject decisions.

**Expected output:**
Audit entry.

## Stretch Goals (Optional - Extra Credit)

- Add a `FixedDepositAccount` that forbids `withdraw` until a maturity date. Override `withdraw` to throw before maturity.
- Add a static factory: `BankAccount.open("savings", owner, balance)` that returns the right subclass.
- Use `Object.getOwnPropertyNames` to list just the methods defined on a subclass (not inherited).

## Submission Requirements

- **What to submit:** Repo with Day 3 files, `prototype-chain.md`, `design-notes.md`, updated `AI_AUDIT.md`.
- **Deadline:** End of Day 3

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| SavingsAccount extends BankAccount | 20 | super called correctly, addInterest works. |
| CurrentAccount overrides withdraw | 20 | Overdraft logic correct. Throws when limit exceeded. |
| Prototype chain sketch | 15 | Chain drawn and explained. `instanceof` verified. |
| Inheritance vs composition notes | 15 | Real answer with reasoning, not "it depends". |
| AI code review | 20 | At least two suggestions evaluated with accept/reject. |
| Clean commits | 5 | Conventional messages. |
| Code quality | 5 | `super` used correctly. No code duplication. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Forgetting `super()` in the subclass constructor.** Subclass constructors MUST call `super()` before using `this`. Otherwise you get a ReferenceError.
- **Deep inheritance hierarchies.** More than 2 levels is usually a design smell. Prefer composition.
- **Overriding without calling `super.method()`.** If you override `withdraw`, you might still want to call the parent's logic for the common parts. Use `super.withdraw(amountCents)` to do that.

## Resources

- Day 3 reading: [prototype chain and inheritance.md](./prototype%20chain%20and%20inheritance.md)
- Week 5 AI boundaries: [../ai.md](../ai.md)
- MDN Inheritance: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain
