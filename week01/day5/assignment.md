# Week 1 - Day 5 Assignment

## Title
The Git Workflow Fundamentals -- Branches, Merges, and a Clean History

## Overview
Week 1 Day 5 is the first of ten weekly Git and GitHub collaboration days. Up to now you have been running `git add`, `git commit`, and `git push` in a straight line on the `main` branch. That works for exactly one person. Today you learn the single most important Git workflow that every team uses: the branch-commit-merge cycle. You will create a feature branch, commit changes to it, merge it back into `main`, and practise the resolution workflow when Git has to decide between two versions of the same line.

## Learning Objectives Assessed
- Explain the difference between a commit, a branch, and a merge in your own words
- Create a branch, switch to it, make commits, and merge it back to main
- Write meaningful commit messages in the conventional commits style (`feat:`, `fix:`, `docs:`, `refactor:`, `chore:`)
- Resolve a basic merge conflict manually by choosing which line to keep
- Read `git log --oneline --graph` and interpret what it shows

## Prerequisites
- Completed Day 1 through Day 4 assignments
- Your `my-first-project` repository is cloned locally and pushed to GitHub
- At least three commits already on your main branch from the week

## AI Usage Rules

**Ratio this week:** 90% manual / 10% AI
**Habit:** Ask AI to explain, not to do.

- **ALLOWED FOR:** Asking AI to explain a Git error message you already read carefully. Asking AI what a specific flag (like `--no-ff`) does after you tried it.
- **NOT ALLOWED FOR:** Generating Git commands for you to copy and paste. Resolving a merge conflict on your behalf. Writing your commit messages.
- **AUDIT REQUIRED:** No. Week 1 still has no formal AI Audit.

## Tasks

### Task 1: Understand the current state of your repository

**What to do:**
From inside your `my-first-project` folder, run these commands one at a time, reading the output of each before moving on:

```bash
git status
git branch
git log --oneline
git log --oneline --graph --all
```

Take a screenshot of the output of `git log --oneline --graph --all` before you do anything else today. You will compare it to the state at the end of Task 4.

**Expected output:**
A screenshot named `day5-before.png` in `assets/` showing the current log graph.

### Task 2: Create a feature branch and commit to it

**What to do:**
1. Create a new branch called `feature/week-1-reflection` and switch to it in one step:

```bash
git checkout -b feature/week-1-reflection
```

2. Verify you are on the new branch:

```bash
git branch
```

You should see an asterisk next to `feature/week-1-reflection`.

3. Create a new file called `week-1-reflection.md` at the root of your repo. In 8-12 sentences, answer these three questions in your own words:
   - What is the most useful thing I learned this week?
   - What is the one thing I still do not understand and want to figure out next week?
   - What is one specific way I will use Git differently after today?

4. Stage and commit it using a conventional commit message:

```bash
git add week-1-reflection.md
git commit -m "docs: add week 1 reflection"
```

5. Make one more commit on the same branch: add a link to `week-1-reflection.md` from your `index.html` (a small change, one new `<a>` tag) and commit it:

```bash
git add index.html
git commit -m "feat: link to week 1 reflection from home page"
```

**Expected output:**
Two commits on `feature/week-1-reflection` that do not yet exist on `main`.

### Task 3: Merge the feature branch back into main

**What to do:**
1. Switch back to the main branch:

```bash
git checkout main
```

2. Merge the feature branch using the `--no-ff` flag so the merge is visible in the history:

```bash
git merge --no-ff feature/week-1-reflection -m "merge: week 1 reflection into main"
```

3. Run `git log --oneline --graph --all` again and take a second screenshot named `day5-after-merge.png`. You should see the branch and the merge commit in the graph.

4. Push main to GitHub:

```bash
git push
```

**Expected output:**
`main` now contains the two commits from the feature branch plus a merge commit. Screenshot `day5-after-merge.png` in `assets/`.

### Task 4: Delete the merged branch (local only)

**What to do:**
After a branch has been merged and you no longer need it, delete it locally:

```bash
git branch -d feature/week-1-reflection
```

Run `git branch` again to confirm the branch is gone locally. Do not delete anything on GitHub -- the merge commit on `main` already preserves the history.

**Expected output:**
`git branch` shows only `main`.

### Task 5: Practise a merge conflict (and resolve it manually)

**What to do:**
This task creates a conflict on purpose so you learn how to resolve one. Follow carefully:

1. Still on `main`, create a new branch `feature/about-update`:

```bash
git checkout -b feature/about-update
```

2. Open `bio.html` and change the `<h1>` text from your name to something slightly different (add a middle name, or add a short tagline). Commit:

```bash
git add bio.html
git commit -m "feat: add tagline to bio h1"
```

3. Switch back to main:

```bash
git checkout main
```

4. On `main`, open the same `bio.html` and change the `<h1>` text in a different way (for example, add emoji text like "(developer)"). Commit:

```bash
git add bio.html
git commit -m "feat: tweak bio h1 on main"
```

5. Now attempt to merge:

```bash
git merge feature/about-update
```

Git will tell you there is a conflict in `bio.html`. Open the file in VS Code and look for the conflict markers:

```
<<<<<<< HEAD
... your version on main ...
=======
... version from feature/about-update ...
>>>>>>> feature/about-update
```

Pick the final version you want (you can keep one side, combine both, or write a new one entirely), then delete the conflict markers. Save the file.

6. Run:

```bash
git add bio.html
git commit -m "merge: resolve bio h1 conflict"
```

7. Take a screenshot named `day5-conflict-resolved.png` of the file after resolution and of `git log --oneline --graph --all`.

**Expected output:**
`bio.html` with one final agreed-upon `<h1>`, a commit that resolves the merge, and the screenshot showing the resolved history.

### Task 6: Push everything and write a short reflection

**What to do:**
1. Push everything to GitHub:

```bash
git push
```

2. At the top of `week-1-reflection.md`, add a new section called "Merge Conflict Lessons" and write 3-5 sentences describing:
   - What the conflict markers looked like
   - How you decided which side to keep
   - One thing you would do differently next time to avoid the conflict in the first place

3. Commit the updated reflection:

```bash
git add week-1-reflection.md
git commit -m "docs: add merge conflict lessons"
git push
```

**Expected output:**
`week-1-reflection.md` on GitHub includes the new section.

## Stretch Goals (Optional - Extra Credit)

- Use `git log --graph --pretty=format:"%h %s"` to produce a cleaner graph view and include a screenshot in `assets/`.
- Revert one of your commits using `git revert <commit-hash>` and explain in `week-1-reflection.md` what `revert` does differently from `reset`.
- Configure `git aliases` so you can type `git lg` instead of `git log --oneline --graph --all`. Add the config command to `week-1-reflection.md`.

## Submission Requirements

- **What to submit:**
  - Updated `my-first-project` repository link.
  - Three screenshots in `assets/`: `day5-before.png`, `day5-after-merge.png`, `day5-conflict-resolved.png`.
  - `week-1-reflection.md` on the repo with both the reflection and the merge conflict lessons section.
- **Where to submit:** Post the repo link in the Week 1 submissions channel.
- **Deadline:** End of Day 5, before the weekend project begins.
- **Format:** Everything inside the existing repo. Conventional commit messages required.

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Feature branch created, committed to, merged | 20 | Branch exists in history, contains at least two commits, merged back via `--no-ff`. Merge commit visible in graph. |
| Merge conflict reproduced and resolved manually | 20 | Student created the conflict on purpose, resolved the conflict markers by hand, and committed the resolution. No AI-generated resolution. |
| Conventional commit messages | 15 | Every commit on Day 5 uses a prefix (`feat:`, `fix:`, `docs:`, `refactor:`, `merge:`, `chore:`) and is written as an imperative ("add X", not "added X"). |
| week-1-reflection.md quality | 15 | Reflection is 8+ sentences in student's own words, covers the three prompts, and includes the merge conflict lessons section. |
| Screenshots demonstrating the workflow | 10 | Three required screenshots saved in `assets/` with the correct filenames. |
| Branch cleanup | 5 | Merged branch deleted locally (not on GitHub -- GitHub keeps the history). |
| Final state of repo | 10 | Repo pushed, graph is clean, no stray uncommitted changes. `git status` shows a clean working tree. |
| Code and content quality | 5 | No junk commits. Files named correctly. No secrets. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Forgetting to switch branches before committing.** If you commit to `main` when you meant to commit to `feature/...`, the history becomes confusing fast. Always run `git branch` or `git status` before committing to double-check which branch you are on.
- **Deleting conflict markers without reading them.** The markers tell you exactly which version came from which side. Read both, pick, then delete. A merge resolved by deleting both sides without reading them is a silent bug.
- **Pushing directly to main without branching.** For the rest of this programme, any meaningful change goes through a feature branch. Start the habit this week.

## Resources

- Day 1 through Day 4 readings for context on the files you are merging
- Week 1 AI boundaries: [../ai.md](../ai.md)
- Official Git book (free): https://git-scm.com/book/en/v2
- GitHub documentation on branches: https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-branches
- Conventional commits spec: https://www.conventionalcommits.org/
