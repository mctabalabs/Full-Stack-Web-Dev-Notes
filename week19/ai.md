# Week 19 - AI Boundaries

**Ratio this week: 35% Manual / 65% AI**
**Habit introduced: "You design the API, AI fills it in."**
**Shift from last week: AI room opens back up, but design authority remains yours. This is library authoring, and library interfaces are taste-defining.**

This week you extract the payments code into a standalone, testable, publishable library. Everything you wrote in Weeks 10, 16, 17, 18 becomes a package: one set of TypeScript interfaces (or JSDoc types), a documented API, a test suite, and a README. By Friday it is installable from an npm registry.

The habit is about interface ownership. AI can implement any function if you define the signature, parameters, and return shape. But the shape itself -- the *public API* of your library -- must come from you. Library design is a taste skill; you earn it by making choices and defending them.

---

## What You MUST Do Manually (35%)

### Day 1 -- Extracting the payments service
- Decide what is "inside" the library and what is "outside". Draw a boundary on paper. Write it down.
- Move each piece across the boundary deliberately. Not a rename; a real "does this make sense in a standalone package?" decision.
- The public API surface -- every function a consumer will call -- is manual. Every parameter name. Every return type.

### Day 2 -- Public API and README
- Write the README by hand. Start with one line that describes what the library is for. Then examples. Then installation. Then advanced usage. The README is the first thing any user will see; write it before the final code.
- For every public function, write one paragraph of docs: what it does, what it returns, what it throws, what side effects it has. No AI paraphrasing.

### Day 3 -- Testing the package
- Write at least one test per public function. Tests are the contract; they belong to you.
- Mock each provider (M-Pesa, Stripe, Airtel) at the HTTP layer. No real network calls in tests.
- Run the tests in CI mode: green on clean runs, red when you break them on purpose.

### Day 4 -- Publishing the package
- Create an npm (or internal registry) account and publish a pre-release version (0.0.1-alpha.0).
- Test installing your own package in a new project. Does the API feel right when you are the consumer?
- Fix the rough edges and publish 0.1.0.

---

## What You CAN Use AI For (65%)

- **Implementing functions** after you have written the signature and tests.
- **Mock implementations** for each provider.
- **README example code** -- but you write the prose around the examples.
- **TypeScript type inference** if you use TypeScript.

Forbidden:
- Designing the public API.
- Naming the exported functions.
- Deciding what to include and exclude.
- Writing the core README prose (examples can be AI-generated).

### Good vs bad prompts this week

**Bad:** "Write me a payments library."
**Good:** "Here is my public API surface [paste signatures]. `initiatePayment` returns `{ status, reference, providerData }`. Is that return shape flexible enough for the three providers I listed, or am I missing a case?"

**Bad:** "Generate tests for my library."
**Good:** "Here is my `initiatePayment` function [paste] and its signature. I want tests for: happy path Stripe, happy path M-Pesa, network failure, invalid amount, wrong provider name. Write one test per case; I will review each."

---

## Things AI Is Bad At This Week

- **API stability.** AI will happily suggest "improvements" that are breaking changes. Resist any suggestion that changes an existing signature.
- **Test isolation.** AI tests often depend on order or share state. Check every test is independent.
- **README tone.** AI README prose is flat and too long. Keep yours crisp.

---

## Core Mental Models For This Week

- **A library is a contract.** Once you publish, you own the contract forever. Breaking it hurts real users.
- **A good API is boring.** Few functions, clear names, stable shapes.
- **Tests are the contract enforced by machines.** Every new version must pass the existing tests unless you explicitly break them and bump a major version.

---

## This Week's AI Audit Focus

Include your final public API surface as a code block in the audit. Annotate each function with "designed by me" or "suggested by AI, accepted/rejected".

---

## Assessment

- Install your published library in a blank project. Live demo: one payment with each provider.
- Walk the facilitator through your README. Defend every example.
- Live task: add one new method to the public API without breaking existing consumers. You do this on screen.

---

## One Sentence To Remember

"You own the shape. AI fills the shape. Never the other way around."
