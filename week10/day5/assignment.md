# Week 10 - Day 5 Assignment

## Title
Secrets, Environment Variables, and Never Leaking a Consumer Secret

## Overview
Week 10 Day 5 is the final Day 5 of the first ten weeks. Appropriately, it is about the single most common real-world Git mistake: committing a secret. You will audit your Week 10 repo for leaked secrets, practise rotating a compromised key, and set up git-secret or git-leaks to catch future mistakes before they hit GitHub.

## Learning Objectives Assessed
- Audit a repo for leaked secrets using `git log` and a scanner
- Rotate a compromised API key safely
- Rewrite Git history to purge a secret (with warnings)
- Set up pre-commit hooks to prevent future leaks

## Prerequisites
- Weeks 1-9 Day 5 completed
- Week 10 Days 1-4 completed (backend with Daraja keys in .env)

## AI Usage Rules

**Ratio:** 50/50. **Habit:** NEVER trust AI with money (and its secrets). See [../ai.md](../ai.md).

- **ALLOWED FOR:** Explaining the output of a secret scanner after you ran it.
- **NOT ALLOWED FOR:** Generating `git filter-branch` commands that might corrupt history.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Audit your repo for leaked secrets

**What to do:**
Run these commands in your Week 10 repo and read the output carefully:

```bash
git log --all --full-history --oneline -S "CONSUMER_KEY" -- .
git log --all --full-history --oneline -S "MPESA_CONSUMER_SECRET" -- .
git grep "CONSUMER_KEY"
git grep "MPESA"
```

These search the entire history for any mention of sensitive keys. Write a short report in `day5-audit.md` with what you found:
- Were any secrets committed?
- If yes, to which commits?
- Are they still in the current main?

**Expected output:**
`day5-audit.md` committed with honest findings.

### Task 2: Install a secret scanner

**What to do:**
Install gitleaks:

macOS: `brew install gitleaks`
Linux: download from https://github.com/gitleaks/gitleaks/releases
Windows: download and add to PATH

Run in your repo:

```bash
gitleaks detect --source . --verbose
```

If it finds anything, note it. If it does not, great -- move on.

**Expected output:**
Screenshot `day5-gitleaks.png` of the scan output.

### Task 3: Simulate a leak and rotation

**What to do:**
1. Deliberately commit a fake secret to a throwaway branch:

```bash
git checkout -b feature/fake-leak
echo "FAKE_SECRET_KEY=supersecret123" > leak.txt
git add leak.txt
git commit -m "oops: do not do this"
```

2. Run gitleaks again. It should find the leak.

3. Pretend the key was real. The correct response is:
   - Immediately rotate the secret in the service dashboard (for real Daraja, you would generate a new Consumer Secret and invalidate the old one)
   - Remove the secret from the latest commit with `git rm leak.txt && git commit -m "remove leaked secret"`
   - Know that the secret is STILL in history and still compromised -- rotation is the only real fix

Document this process in `day5-rotation.md`. 5-7 sentences explaining why you cannot just "delete" a secret from Git history and expect it to be safe.

**Expected output:**
`day5-rotation.md` committed. Branch with the fake leak kept for proof.

### Task 4: Pre-commit hook to block future leaks

**What to do:**
Install husky (the standard pre-commit tool) or use a simple shell hook. Here is the simpler shell version:

Create `.git/hooks/pre-commit`:

```bash
#!/bin/sh
if git diff --cached | grep -E "CONSUMER_SECRET|API_KEY|PASSKEY" > /dev/null; then
  echo "ERROR: Potential secret detected in staged changes. Aborting commit."
  exit 1
fi
exit 0
```

Make it executable:

```bash
chmod +x .git/hooks/pre-commit
```

Try to commit a file containing the string `CONSUMER_SECRET=abc`. The hook should block it.

**Expected output:**
Screenshot `day5-hook-blocked.png` showing the hook blocking a bad commit.

### Task 5: Write a secrets policy

**What to do:**
In `SECURITY.md` at the repo root, write a short secrets policy in your own words:

```markdown
# Secrets Policy

1. Never commit `.env` files. They are in .gitignore.
2. Never paste a Consumer Secret in AI prompts, Discord, or screenshots.
3. If a secret leaks, rotate it immediately.
4. Pre-commit hooks scan for known patterns.
5. Every environment variable has a documented counterpart in `.env.example`.
```

Expand with your own rules if you want.

**Expected output:**
`SECURITY.md` committed.

## Stretch Goals (Optional - Extra Credit)

- Set up GitHub secret scanning alerts in the repo settings.
- Add a CI step that runs gitleaks on every push.
- Research `BFG Repo-Cleaner` as an alternative to `git filter-branch` for removing secrets from history.

## Submission Requirements

- **What to submit:** Repo, `day5-audit.md`, `day5-rotation.md`, `SECURITY.md`, screenshots.
- **Deadline:** End of Day 5

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Repo audited for secrets | 15 | day5-audit.md documents findings honestly. |
| Gitleaks installed and run | 20 | Screenshot proves the scanner ran. |
| Rotation scenario documented | 25 | day5-rotation.md explains why "rotate first, clean later" is the right order. |
| Pre-commit hook blocking fake secrets | 20 | Screenshot shows the hook blocking a bad commit. |
| SECURITY.md policy | 15 | At least 5 rules, in student's own words. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Thinking `git rm` deletes a secret.** It removes the file from the working tree, but the secret lives in the history forever. Rotation is the only real fix.
- **Force-pushing to rewrite history without coordinating with your team.** You will nuke their work. Only do it if you are the sole owner of the repo.
- **Running `git filter-branch` from AI output.** This command can corrupt your repo if misused. Understand every flag before running it.

## Resources

- Prior Day 5 assignments (Weeks 1-9)
- Week 10 AI boundaries: [../ai.md](../ai.md)
- gitleaks: https://github.com/gitleaks/gitleaks
- GitHub secret scanning: https://docs.github.com/en/code-security/secret-scanning
- OWASP cheat sheet on secrets: https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html
