# Week 5 - Day 5 Assignment

## Title
Tags, Releases, and Rebasing Instead of Merging

## Overview
Week 5 Day 5 advances your Git literacy with two more workflow tools: `git tag` for marking specific commits as releases, and `git rebase` for keeping a linear history instead of merge commits. Every serious codebase uses one or both of these. You will tag a release of your portfolio and rebase a feature branch onto the latest main.

## Learning Objectives Assessed
- Create annotated tags on commits
- Push tags to GitHub and see them appear under "Releases"
- Rebase a feature branch onto main
- Understand the trade-off between merge and rebase
- Recover from a failed rebase using `git rebase --abort`

## Prerequisites
- Weeks 1-4 Day 5 assignments completed
- A working repo with at least 5 commits on main

## AI Usage Rules

**Ratio:** 70/30. **Habit:** AI as code reviewer. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Explaining a rebase conflict after you read it.
- **NOT ALLOWED FOR:** Generating rebase commands or skipping the manual resolution.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Create an annotated tag

**What to do:**
Pick a meaningful commit on your portfolio or `js-lab` repo (the "end of Week 5" state is a good choice). Tag it:

```bash
git tag -a v0.1.0 -m "End of Week 5: OOP foundations"
git show v0.1.0           # shows the tag and the commit it points to
```

Push the tag:

```bash
git push origin v0.1.0
```

Go to github.com and click "Releases" (usually in the right sidebar or under "Tags"). Your tag is visible.

**Expected output:**
Screenshot `day5-tag.png` showing the tag in GitHub UI.

### Task 2: Create a GitHub release

**What to do:**
On GitHub, click "Draft a new release". Pick your `v0.1.0` tag. Fill in:
- Release title: "Week 5: OOP Foundations"
- Description: 3-5 bullets summarising what you learned (in your own words)
- Click "Publish release"

**Expected output:**
Screenshot `day5-release.png` of the published release.

### Task 3: Rebase a feature branch

**What to do:**
1. Create a new feature branch:

```bash
git checkout -b feature/rebase-practice
# make at least 2 commits on this branch
```

2. Switch to main and make at least one commit there to create divergence:

```bash
git checkout main
# edit something, commit
git commit -am "docs: update readme"
```

3. Now rebase your feature branch onto the updated main:

```bash
git checkout feature/rebase-practice
git rebase main
```

4. If a conflict arises, resolve it manually (no AI). Use `git rebase --continue` after fixing. If you get stuck, `git rebase --abort` resets.

5. Verify the result with `git log --oneline --graph --all`. The feature branch should now start from the latest main, with a linear history.

**Expected output:**
Screenshot `day5-rebase.png` showing the linear history.

### Task 4: Merge vs rebase notes

**What to do:**
In `rebase-notes.md`, write 6-8 sentences answering:
1. What is the visual difference between a merge commit and a rebase?
2. When would you use merge vs rebase?
3. What is the "golden rule" of rebasing public branches? (Hint: it rhymes with "never".)
4. What happened when you rebased: did anything unexpected occur?

Your own words.

**Expected output:**
`rebase-notes.md` committed.

### Task 5: Rebase recovery drill

**What to do:**
Intentionally cause a conflicting rebase, then abort:

1. Create another feature branch and make a conflicting change with main.
2. Start a rebase: `git rebase main`.
3. When the conflict appears, do NOT resolve it. Instead:

```bash
git rebase --abort
```

4. Verify your feature branch is back to its pre-rebase state with `git status` and `git log`.

**Expected output:**
Screenshot `day5-abort.png` of the status after the abort.

## Stretch Goals (Optional - Extra Credit)

- Do an interactive rebase: `git rebase -i HEAD~5` to squash, edit, or reorder recent commits.
- Create a pre-release tag (like `v0.1.0-alpha`) and explain SemVer in `rebase-notes.md`.
- Use `git log --all --decorate --oneline --graph` alias (call it `git lg`) to visualise history.

## Submission Requirements

- **What to submit:** Repo link, all screenshots, `rebase-notes.md`, `AI_AUDIT.md`.
- **Deadline:** End of Day 5

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Annotated tag created and pushed | 15 | Tag visible on GitHub. |
| GitHub release published | 10 | Release with title, description, and tag. |
| Rebase completed on a feature branch | 25 | Feature branch successfully rebased onto latest main. Linear history visible. |
| Rebase conflict handled (or abort done) | 15 | Either resolved manually or aborted cleanly. Screenshot provided. |
| Merge vs rebase notes | 20 | All four questions answered in student's own words. |
| Clean commits | 10 | Conventional messages. |
| AI Audit | 5 | Any AI interactions documented. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Rebasing a shared branch.** The golden rule: never rebase a branch that other people have pulled from. Rebase your own feature branches before merging, not after.
- **Using `git push --force` without understanding it.** After a rebase, you need to force-push your own branch because the history changed. Use `--force-with-lease` instead of `--force` to avoid overwriting someone else's work.
- **Aborting halfway through a rebase manually.** Use `git rebase --abort`, not `git reset`. The rebase leaves state that only `--abort` cleans up correctly.

## Resources

- Weeks 1-4 Day 5 assignments (prior Git content)
- Week 5 AI boundaries: [../ai.md](../ai.md)
- Pro Git book, Ch. 3.6 Rebasing: https://git-scm.com/book/en/v2/Git-Branching-Rebasing
- GitHub Releases documentation: https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases
