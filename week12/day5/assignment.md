# Week 12 - Day 5 Assignment

## Title
Git Hooks With Husky And Conventional Commit Enforcement

## Overview
Week 12 Day 5 adds a guardrail to your Git workflow: automated checks that run before every commit. You install husky, add lint-staged to format and lint only changed files, and add commitlint to enforce conventional commit messages. This is how real teams keep a repository clean without relying on willpower.

## Learning Objectives Assessed
- Install and configure husky for Git hooks
- Configure lint-staged to run Prettier on staged files
- Enforce conventional commit messages with commitlint
- Understand when hooks are helpful and when they become friction

## Prerequisites
- Weeks 1-11 Day 5 completed
- Any repo with real commits this week

## AI Usage Rules

**Ratio:** 45/55. **Habit:** Never trust AI with auth. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Config YAML after you read the husky/commitlint docs.
- **NOT ALLOWED FOR:** Copy-pasting a config without understanding each field.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Install husky

**What to do:**
In your monorepo root:

```bash
npm install -D husky
npx husky init
```

This creates `.husky/pre-commit` with a sample hook. Make it executable.

**Expected output:**
`.husky/` folder exists with a pre-commit hook.

### Task 2: Run Prettier on staged files via lint-staged

**What to do:**
```bash
npm install -D lint-staged prettier
```

Add to root `package.json`:

```json
"lint-staged": {
  "*.{js,jsx,ts,tsx,md,json}": "prettier --write"
}
```

Replace `.husky/pre-commit` with:

```bash
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npx lint-staged
```

Make an edit, stage it, commit. Prettier should run automatically on the staged files only.

**Expected output:**
Screenshot `day5-husky-format.png` of a commit that triggered Prettier.

### Task 3: Commitlint for conventional commits

**What to do:**
```bash
npm install -D @commitlint/cli @commitlint/config-conventional
```

Create `commitlint.config.js` at the root:

```javascript
module.exports = { extends: ["@commitlint/config-conventional"] };
```

Add a commit-msg hook:

```bash
npx husky add .husky/commit-msg 'npx --no-install commitlint --edit "$1"'
```

Try committing with a bad message: `git commit -m "fixed stuff"`. The hook should reject.
Try: `git commit -m "fix: correct date formatter"`. The hook should accept.

**Expected output:**
Screenshot `day5-commitlint-reject.png` of a rejected bad message.

### Task 4: Short reflection

**What to do:**
In `hooks-notes.md`, write 5-7 sentences:
- What problems do pre-commit hooks solve?
- What is the cost of a slow pre-commit hook?
- When should you `git commit --no-verify` and when should you never?
- What would you add next? (lint? tests? security scan?)

Your own words.

**Expected output:**
`hooks-notes.md` committed.

### Task 5: Document the setup

**What to do:**
Add a section to your root README called "Developer setup" that explains:
- Run `npm install` at the root
- Husky hooks install automatically
- Commits are auto-formatted
- Commit messages must follow conventional commits

A new cohort member cloning the repo should be able to get the same hooks working.

**Expected output:**
README updated.

## Stretch Goals (Optional - Extra Credit)

- Add a pre-push hook that runs the full test suite.
- Add a commit-msg hook that enforces a maximum subject length.
- Configure commitlint with your own allowed types (e.g., add `chore:`, `docs:`).

## Submission Requirements

- **What to submit:** Repo with husky config, screenshots, `hooks-notes.md`, updated README, `AI_AUDIT.md`.
- **Deadline:** End of Day 5

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Husky installed and initialised | 15 | `.husky/` folder exists with working pre-commit. |
| lint-staged + Prettier on staged files | 25 | Prettier runs on commit. Screenshot confirms. |
| Commitlint enforces conventional commits | 25 | Bad message rejected. Good message accepted. Screenshot confirms. |
| Hooks reflection | 15 | Four questions answered in student's own words. |
| README developer setup section | 10 | Documented so new members can reproduce. |
| Clean commits | 10 | Conventional messages (which will pass your new hook). |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Making the hook so slow it encourages `--no-verify`.** Keep it fast. Run lint only on staged files.
- **Enforcing rules no one agreed on.** Pre-commit hooks that surprise your teammates cause friction. Document and align first.
- **Forgetting to commit the husky config files.** They must be in the repo or the hooks do not propagate.

## Resources

- Prior Day 5 assignments
- Week 12 AI boundaries: [../ai.md](../ai.md)
- husky docs: https://typicode.github.io/husky
- commitlint: https://commitlint.js.org/
