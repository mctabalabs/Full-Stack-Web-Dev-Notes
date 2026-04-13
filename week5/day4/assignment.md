# Week 5 - Day 4 Assignment

## Title
ES6 Modules and the Cohort Project Pre-Weekend Checklist

## Overview
Day 4 is pre-weekend polish day. Today you split your Day 1-3 code into ES6 modules -- each class in its own file, with clean `export`/`import` statements. Then you integrate your modularised code with the cohort project shell and run through a checklist that proves you are ready to ship this weekend.

## Learning Objectives Assessed
- Use named and default exports
- Import from relative paths with `.js` extensions
- Split a small app into logical module files
- Understand that imports are at the top of the file and are hoisted

## Prerequisites
- Days 1-3 completed

## AI Usage Rules

**Ratio:** 70/30. **Habit:** AI as code reviewer. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Reviewing your module structure after you split your files.
- **NOT ALLOWED FOR:** Deciding the module boundaries for you.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Split BankAccount into modules

**What to do:**
Reorganise your Day 2-3 code into this structure:

```
week5-day4/
  index.html
  main.js
  accounts/
    BankAccount.js
    SavingsAccount.js
    CurrentAccount.js
```

Each account file should export its class:

```javascript
// BankAccount.js
export class BankAccount {
  // ...
}
```

```javascript
// SavingsAccount.js
import { BankAccount } from "./BankAccount.js";
export class SavingsAccount extends BankAccount {
  // ...
}
```

In `main.js`, import all three classes and instantiate one of each:

```javascript
import { BankAccount } from "./accounts/BankAccount.js";
import { SavingsAccount } from "./accounts/SavingsAccount.js";
import { CurrentAccount } from "./accounts/CurrentAccount.js";
```

In `index.html`, load `main.js` as a module:

```html
<script type="module" src="main.js"></script>
```

**Expected output:**
Opening the page in Live Server runs all three classes without errors. The network tab shows the module files loading.

### Task 2: Named vs default export

**What to do:**
Add one helper function `formatCurrency(cents)` in `accounts/format.js`. Export it as a named export. Also create a `defaults.js` with a default-exported object of default settings:

```javascript
// defaults.js
export default {
  minBalance: 1000,
  interestRate: 0.05,
};
```

Import both correctly in main.js:

```javascript
import { formatCurrency } from "./accounts/format.js";
import settings from "./accounts/defaults.js";
```

**Expected output:**
Both imports working. At least one call to `formatCurrency` visible in the console.

### Task 3: Integrate with the cohort project

**What to do:**
The cohort project (see [../weekend/cohort1-project.md](../weekend/cohort1-project.md)) will be team work this weekend. Today you prepare. Create a `cohort-plan.md` describing:
1. Which class in your modular code will map to what in the cohort project
2. Three features you are responsible for
3. Three features you depend on another teammate for
4. One design decision you want to raise with your team

**Expected output:**
`cohort-plan.md` committed.

### Task 4: Pre-weekend checklist

**What to do:**
Create `CHECKLIST.md`:

```markdown
## Week 5 Day 4 Pre-Weekend Checklist

- [ ] Day 1 inventory objects working
- [ ] Day 2 BankAccount constructor and class both working
- [ ] Day 3 SavingsAccount and CurrentAccount working
- [ ] Day 4 code is split into modules and loads via script type="module"
- [ ] All classes have at least one test call in main.js
- [ ] cohort-plan.md exists and is realistic
- [ ] AI_AUDIT.md has code review entries for each day
- [ ] Repo pushed and clean
```

Tick every box honestly.

**Expected output:**
`CHECKLIST.md` committed.

### Task 5: Module structure AI review

**What to do:**
Share your folder structure (just the file names and one sentence about each file) with AI and ask: "Is this a reasonable split for a small OOP project? What would you change?"

Evaluate the suggestions. Record accept/reject in `AI_AUDIT.md`.

**Expected output:**
Audit entry.

## Stretch Goals (Optional - Extra Credit)

- Add an `index.js` barrel file in `accounts/` that re-exports all account classes. Import from `./accounts` instead of individual files.
- Use dynamic imports: `import("./accounts/BankAccount.js").then(mod => ...)` for lazy loading.
- Configure a local `package.json` with `"type": "module"` and run one of your modules through Node.

## Submission Requirements

- **What to submit:** Repo with Day 4 files, `cohort-plan.md`, `CHECKLIST.md`, `AI_AUDIT.md`.
- **Deadline:** End of Day 4

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Classes split into module files | 25 | Each class in its own file with proper export. Main.js imports all three and instantiates. Page loads without errors. |
| Named and default exports both demonstrated | 15 | format.js (named) and defaults.js (default) both imported correctly. |
| cohort-plan.md realistic | 15 | Four questions answered concretely. |
| CHECKLIST.md complete | 10 | All boxes honest. |
| Module structure AI review | 20 | At least two suggestions with accept/reject. |
| Clean commits | 10 | Conventional messages. |
| Code quality | 5 | Consistent import style (relative paths with .js extensions). |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Omitting `.js` from import paths.** Browsers require the full path including the extension: `import ... from "./file.js"`. Node has similar rules depending on config.
- **Forgetting `type="module"` on the script tag.** Without it, the browser treats the file as a classic script and imports will throw.
- **Circular imports.** Two files importing each other can create initialisation ordering bugs. Refactor to a shared third file if you hit this.

## Resources

- Day 4 reading: [ES6 modules.md](./ES6%20modules.md)
- Week 5 AI boundaries: [../ai.md](../ai.md)
- MDN JavaScript modules: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules
