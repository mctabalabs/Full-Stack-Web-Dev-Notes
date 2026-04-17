# Week 9 - Day 5 Assignment

## Title
GitHub Actions CI -- Automatic Tests On Every Push

## Overview
Week 9 Day 5 is your first taste of CI/CD. You will set up a GitHub Actions workflow that runs your tests on every push and every pull request. When a test fails, the PR shows a red X and cannot merge until you fix it. This is how real teams catch regressions.

## Learning Objectives Assessed
- Create a `.github/workflows/ci.yml` file
- Run `npm test` in CI on push and pull_request events
- Read CI logs when something fails
- Require CI to pass before merging a PR

## Prerequisites
- Weeks 1-8 Day 5 completed
- Week 9 Day 4: Vitest tests passing locally

## AI Usage Rules

**Ratio:** 50/50. **Habit:** AI reads docs, you make decisions. See [../ai.md](../ai.md).

- **ALLOWED FOR:** YAML help after you read the GitHub Actions quickstart.
- **NOT ALLOWED FOR:** Copying a workflow you do not understand.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Create the workflow file

**What to do:**
In your repo, create `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Use Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install
        run: npm ci

      - name: Test
        run: npm test -- --run
```

Note the `--run` flag. Vitest defaults to watch mode, which would hang CI. `--run` runs once and exits.

Commit and push this file to main.

**Expected output:**
A workflow run appears on the Actions tab after you push.

### Task 2: Watch it run

**What to do:**
Visit the Actions tab. Click the most recent run. Watch it execute:
1. Checkout
2. Setup Node
3. Install
4. Test

When it finishes, the commit on main should have a green check.

**Expected output:**
Green check on the commit. Screenshot `day5-ci-green.png`.

### Task 3: Break a test on purpose and push

**What to do:**
1. Change one of your tests to expect a wrong value.
2. Push to a feature branch and open a PR.
3. The CI runs and fails. The PR shows a red X.
4. Read the CI logs in the Actions tab. Find the line that says which test failed.

**Expected output:**
Screenshot `day5-ci-red.png` of the failed CI run with the failing test visible.

### Task 4: Fix the test and push again

**What to do:**
Fix the test. Push the fix to the same branch. CI re-runs. Green check. Merge the PR.

**Expected output:**
PR merged. CI green. Screenshot `day5-ci-fixed.png`.

### Task 5: Branch protection

**What to do:**
In repo Settings > Branches, add a protection rule for `main`:
- Require a pull request before merging
- Require status checks to pass before merging (pick the CI job)

Try to push directly to main from your terminal. GitHub should reject.

**Expected output:**
Screenshot `day5-protection.png` of the branch protection settings.

## Stretch Goals (Optional - Extra Credit)

- Add a build step: `- name: Build; run: npm run build`
- Add matrix builds across Node 18 and Node 20.
- Add a caching step for faster subsequent runs.

## Submission Requirements

- **What to submit:** Repo, workflow file, four screenshots, `AI_AUDIT.md`.
- **Deadline:** End of Day 5

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| ci.yml file committed | 15 | Workflow is in the right location and runs on push/PR. |
| CI passes on a clean commit | 20 | Green check visible on main. |
| CI fails on a deliberate break | 20 | Red X visible on a branch with a broken test. Logs show the failing test. |
| CI passes again after fix | 15 | Branch green and merged. |
| Branch protection enabled | 20 | Cannot push directly to main. PRs required. CI is a required check. |
| Clean commits | 10 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Forgetting `--run` on Vitest.** Without it, Vitest starts watch mode in CI and the job runs until it times out. Always `npm test -- --run` in CI.
- **Hardcoding secrets in YAML.** Never put API keys in workflow files. Use GitHub Secrets if you need them (not required for this assignment).
- **Making branch protection so strict no one can merge.** Require CI, but do not also require 5 approvals for a team of 1.

## Resources

- Prior Day 5 assignments (Weeks 1-8)
- Week 9 AI boundaries: [../ai.md](../ai.md)
- GitHub Actions quickstart: https://docs.github.com/en/actions/quickstart
- GitHub Actions for Node.js: https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-nodejs
