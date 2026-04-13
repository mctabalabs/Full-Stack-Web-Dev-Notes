# Week 21 - Day 5 Assignment

## Title
Git Worktrees -- Two Branches Checked Out At Once

## Overview
Day 5 is a Git day. Today you learn `git worktree` so you can have two (or three) branches checked out simultaneously in different folders, without stashing or switching back and forth. Perfect for when you need to hotfix main while mid-feature.

## Learning Objectives Assessed
- Create a second worktree for another branch
- Understand where worktrees are stored
- Remove a worktree cleanly
- Know when to use worktrees vs stash vs clone

## Prerequisites
- Weeks 1-20 Day 5 completed

## AI Usage Rules

**Ratio this week:** 30% manual / 70% AI
**Habit:** AI writes schedules, you decide reliability. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Command explanations.
- **NOT ALLOWED FOR:** Recovery decisions if you break your worktree.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Create a feature worktree

**What to do:**
From your main repo:

```bash
git worktree add ../class-notes-hotfix main
cd ../class-notes-hotfix
```

You now have a second folder checked out to `main`. Your original folder keeps its current branch untouched.

**Expected output:**
`git worktree list` shows two entries.

### Task 2: Simulate a hotfix

**What to do:**
In the original repo, create and switch to `feat/experiment`. Make a change but do not commit.

Back in the hotfix worktree (`../class-notes-hotfix`), create a hotfix branch, make a one-line fix, commit, push.

Return to the feature worktree. Your uncommitted changes are still there.

**Expected output:**
Two branches progressed independently, no stash used.

### Task 3: Remove a worktree

**What to do:**
From the main folder:

```bash
git worktree remove ../class-notes-hotfix
```

Verify with `git worktree list`.

**Expected output:**
Worktree gone, branch still exists in the repo.

### Task 4: Worktrees vs alternatives

**What to do:**
In `worktree-notes.md`, write 5-7 sentences:
- When is `git worktree` better than `git stash`?
- When is it better than a second clone?
- What breaks if two worktrees try to check out the same branch? (Git refuses.)
- What happens to a worktree folder if you delete it with `rm -rf` without `git worktree remove`?

Your own words.

**Expected output:**
`worktree-notes.md` committed.

### Task 5: Week 21 wrap

**What to do:**
Update the repo `README.md` with a Week 21 section:

```markdown
## Week 21 -- Scheduled Work

- node-cron for in-process jobs
- pg_cron for SQL-only jobs
- Postgres advisory locks prevent overlap across instances
- ADR 0001 documents the choice
```

Push everything.

**Expected output:**
README updated, all Week 21 work pushed.

## Stretch Goals (Optional - Extra Credit)

- Use a worktree for reviewing a teammate's PR without polluting your current branch.
- Script `git worktree prune` into a local alias.
- Try `git worktree add --detach` to check out a commit instead of a branch.

## Submission Requirements

- **What to submit:** Repo with `worktree-notes.md`, README update, `AI_AUDIT.md`.
- **Deadline:** End of Day 5

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Second worktree created | 20 | `git worktree list` shows two. |
| Hotfix simulated | 25 | Two branches progressed independently. |
| Worktree removed cleanly | 15 | `git worktree list` shows one again. |
| Worktree notes | 25 | Four questions answered. |
| README updated | 10 | Week 21 section present. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Deleting a worktree folder with `rm -rf` first.** Git still thinks it exists. Run `git worktree prune` to clean up.
- **Trying to check out the same branch in two worktrees.** Git refuses. Use a new branch in the second worktree.
- **Forgetting which worktree you are in.** Always check `git branch` and `pwd` before committing.

## Resources

- Day 5 reading: [Week 21 Recap.md](./Week%2021%20Recap.md)
- Week 21 AI boundaries: [../ai.md](../ai.md)
- Git worktree docs: https://git-scm.com/docs/git-worktree
