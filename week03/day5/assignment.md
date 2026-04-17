# Week 3 - Day 5 Assignment

## Title
Forks, Upstream Remotes, and Contributing To Someone Else's Repo

## Overview
Week 2 Day 5 you opened a pull request on your own repo. This week you open one on someone else's. Forking is how open source works: you make a personal copy of a repo, change something, and propose the change back upstream. Today you fork the cohort's shared sandbox repo (or a teammate's), add a small contribution, and open a PR from your fork.

## Learning Objectives Assessed
- Fork a repository on GitHub
- Clone your fork, add an upstream remote, and keep it synced
- Commit to a feature branch in your fork
- Open a pull request from your fork to the upstream repo
- Respond to review comments from a maintainer you do not control

## Prerequisites
- Weeks 1-2 completed
- Week 3 Days 1-4 assignments submitted
- A cohort partner or a public sandbox repo provided by the facilitator

## AI Usage Rules

**Ratio:** 80/20. **Habit:** 15-minute rule. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Explaining a Git error you already read. Explaining what `upstream` means after you searched.
- **NOT ALLOWED FOR:** Generating Git commands. Writing your PR description.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Fork a shared sandbox repo

**What to do:**
1. Navigate to the cohort sandbox repo (the facilitator will share a link) or a classmate's repo with their permission.
2. Click "Fork" at the top right of GitHub. Pick your own account as the destination.
3. Wait for GitHub to create the fork. You now have a copy under `github.com/YOUR_USERNAME/sandbox`.

**Expected output:**
A visible fork in your GitHub profile.

### Task 2: Clone your fork and add upstream remote

**What to do:**
```bash
git clone https://github.com/YOUR_USERNAME/sandbox.git
cd sandbox
git remote -v     # should show origin = your fork
git remote add upstream https://github.com/ORIGINAL_OWNER/sandbox.git
git remote -v     # should now show both origin and upstream
git fetch upstream
```

**Expected output:**
`git remote -v` shows two remotes: `origin` (your fork) and `upstream` (the original).

### Task 3: Create a feature branch and contribute

**What to do:**
Sync your main branch with upstream, then branch off:

```bash
git checkout main
git fetch upstream
git merge upstream/main
git checkout -b feature/add-YOUR_NAME
```

Add a small contribution. The cohort sandbox will have a `contributors/` folder with a template. Copy `template.md` to `contributors/YOUR_NAME.md` and fill it in with:
- Your full name
- Your GitHub username
- One sentence about why you joined the Marathon
- One learning from Week 3

Commit:

```bash
git add contributors/YOUR_NAME.md
git commit -m "docs: add YOUR_NAME to contributors"
git push -u origin feature/add-YOUR_NAME
```

**Expected output:**
New file in your fork, pushed to GitHub, visible on the feature branch.

### Task 4: Open a pull request to upstream

**What to do:**
1. On GitHub, visit your fork. GitHub detects the new branch and offers "Compare & pull request" -- click it.
2. Verify the PR is going FROM `YOUR_USERNAME/sandbox:feature/add-YOUR_NAME` TO `ORIGINAL_OWNER/sandbox:main`. This is the key step. If the target is your own main, start over.
3. Fill in the PR title: `docs: add YOUR_NAME to contributors`
4. Fill in the PR description using the What / Why / How to test template from Week 2.
5. Click "Create pull request".

**Expected output:**
A PR open on the upstream repo, not your fork. Screenshot `day5-pr-upstream.png`.

### Task 5: Respond to review comments

**What to do:**
Ask a cohort mate or facilitator to leave at least two review comments on your PR. Respond to each:
- If the comment is a requested change, make it, commit, push. The PR updates automatically.
- If the comment is a question, reply in the thread on GitHub.
- Mark threads as Resolved when done.

**Expected output:**
PR with at least two review threads resolved.

### Task 6: Sync with upstream again (the essential habit)

**What to do:**
After your PR is merged (or even before), pretend there are new changes on upstream/main. Simulate by syncing:

```bash
git checkout main
git fetch upstream
git merge upstream/main
git push origin main
```

This keeps your fork's main in sync with the upstream. Running this before starting any new feature branch is the healthy fork habit.

**Expected output:**
Your fork's main is now up to date with upstream's main.

## Stretch Goals (Optional - Extra Credit)

- Use `git rebase upstream/main` instead of `merge` for a linear history. Explain the difference in `notes.md`.
- Set up a GitHub SSH key if you have not yet, and switch your `origin` and `upstream` remotes to use SSH URLs.
- Configure a git alias `sync-fork` that runs the three sync commands as one.

## Submission Requirements

- **What to submit:** Link to the PR on the upstream repo, screenshot `day5-pr-upstream.png`, updated `AI_AUDIT.md`.
- **Where:** Week 3 submissions channel
- **Deadline:** End of Day 5

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Fork created | 10 | Visible fork in your GitHub account. |
| Both remotes configured | 15 | `origin` and `upstream` both set. Verified with `git remote -v`. |
| Feature branch with contribution | 15 | Branch created, file added, commit with conventional message, pushed to fork. |
| PR opened to upstream (not to own fork) | 25 | PR visible on the upstream repo's pulls tab. PR target is upstream/main, not fork/main. This is the single biggest mistake -- doing it wrong costs the full 25 points. |
| Review comments addressed | 15 | Two threads resolved. Replies visible or commits added. |
| Sync with upstream demonstrated | 10 | After-the-fact sync commands run successfully. |
| AI Audit | 10 | 15-minute rule entries included. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Opening the PR to your own fork.** GitHub defaults to the upstream repo as the target when you click "Compare & pull request", but if you start the PR from "New pull request" manually you can accidentally target your fork. Always check the "base repository" dropdown.
- **Forgetting `git fetch upstream` before branching.** If you branch from a stale main, your PR will contain commits that already exist on upstream, which confuses reviewers.
- **Pushing to upstream main.** You cannot (you do not have permissions), but students sometimes try. The whole workflow works because pull requests are the only way changes reach upstream.

## Resources

- Week 2 Day 5 assignment (pull request basics)
- GitHub Forks documentation: https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks
- GitHub Flow guide: https://docs.github.com/en/get-started/quickstart/github-flow
