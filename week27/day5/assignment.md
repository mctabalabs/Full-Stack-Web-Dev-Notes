# Week 27 - Day 5 Assignment

## Title
Repo Scaffolding, Milestones, And Git Submodules

## Overview
Day 5 scaffolds the capstone repo, sets up milestones and issues for Weeks 28-30, and teaches `git submodule` for sharing a common package across repos.

## Learning Objectives Assessed
- Scaffold a monorepo capstone layout
- Create GitHub milestones and issues
- Use a git submodule for shared code
- Understand when submodules are a bad idea

## Prerequisites
- Days 1-4 completed

## AI Usage Rules

**Ratio this week:** 55% manual / 45% AI
**Habit:** Architecture dip. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Folder scaffolds.
- **NOT ALLOWED FOR:** Milestone/issue prioritisation.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Monorepo layout

**What to do:**
```
capstone/
  apps/
    web/         (Next.js)
    api/         (Express)
    workers/     (BullMQ)
  packages/
    db/          (migrations + pool)
    types/       (shared TS types)
  infra/
    docker-compose.yml
    nginx/
```

Create the folder tree. Drop a `package.json` in each. Turbo/nx optional but welcome.

**Expected output:**
Tree committed.

### Task 2: Milestones

**What to do:**
Create three milestones on GitHub:
- Capstone Week 28 -- Multi-tenant core
- Capstone Week 29 -- Integrations
- Capstone Week 30 -- Launch

Attach at least five issues per milestone based on your user stories from Day 1.

**Expected output:**
Screenshot `day5-milestones.png`.

### Task 3: Submodule demo

**What to do:**
Create a second tiny repo `mctaba-shared-types` with some TS types. Add it as a submodule:

```bash
git submodule add https://github.com/you/mctaba-shared-types packages/shared-types
git commit -m "chore: add shared-types submodule"
```

Verify `.gitmodules` looks sane.

**Expected output:**
Submodule committed.

### Task 4: When NOT to use submodules

**What to do:**
In `submodule-notes.md`, write 5-7 sentences:
- What does a submodule pin? (A specific commit.)
- Why is onboarding harder with submodules? (`git clone --recurse-submodules`.)
- When is an npm workspace or monorepo package a better choice?
- What is the main risk of a submodule at a moving branch?

Your own words.

**Expected output:**
`submodule-notes.md` committed.

### Task 5: Week 27 wrap

**What to do:**
Update the capstone `README.md` with:
- Vertical
- Tech stack
- How to run locally
- Link to SPEC.md, ROADMAP.md, api.md

Push everything. Tag `v0.27.0`.

**Expected output:**
Pushed.

## Stretch Goals (Optional - Extra Credit)

- Use Turborepo for the monorepo build.
- Try `npm workspaces` as the alternative to submodules.
- Create a CODEOWNERS file.

## Submission Requirements

- **What to submit:** Repo, milestones screenshot, submodule notes, `AI_AUDIT.md`.
- **Deadline:** End of Day 5

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Monorepo tree | 20 | Folders + package.json. |
| Milestones + issues | 25 | 3 x 5+. |
| Submodule demo | 20 | .gitmodules visible. |
| Submodule notes | 25 | Four questions answered. |
| README + tag | 5 | Pushed. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Reaching for submodules when npm workspaces would do.** Usually the wrong call.
- **Forgetting `--recurse-submodules` on clone.** Contributors see an empty folder.
- **Milestones with no issues.** Unhelpful.

## Resources

- Day 5 reading: [Repo Scaffolding and Milestones.md](./Repo%20Scaffolding%20and%20Milestones.md)
- Week 27 AI boundaries: [../ai.md](../ai.md)
- git submodules: https://git-scm.com/book/en/v2/Git-Tools-Submodules
