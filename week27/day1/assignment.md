# Week 27 - Day 1 Assignment

## Title
Capstone Kickoff -- Scope The Multi-Tenant SaaS

## Overview
Week 27 begins the capstone: a multi-tenant SaaS built on everything from Weeks 1-26. Today you write the one-page product spec, pick your vertical (stylists, gyms, clinics, chamas, shops -- one), and define the MVP.

## Learning Objectives Assessed
- Pick a defensible capstone vertical
- Scope an MVP that fits the remaining four weeks
- Write clear user stories
- Avoid feature creep

## Prerequisites
- Weeks 1-26 completed

## AI Usage Rules

**Ratio this week:** 55% manual / 45% AI
**Habit:** Architecture dip -- you drive the thinking. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Summarising existing SaaS examples for inspiration.
- **NOT ALLOWED FOR:** The scope itself.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Pick the vertical

**What to do:**
Choose ONE from your previous projects and turn it multi-tenant. Write `capstone/PICK.md` with three sentences on why.

**Expected output:**
Picked and justified.

### Task 2: One-page spec

**What to do:**
`capstone/SPEC.md` answering:
- Who is a "tenant"? (A business or owner account.)
- Who uses the app inside a tenant? (Staff, customers.)
- What are the three core actions? (Example: create resource, book it, pay for it.)
- What is the monetisation model? (Per-tenant monthly, or per-transaction.)
- What is NOT in the MVP? (List three things.)

Keep it one page.

**Expected output:**
SPEC committed.

### Task 3: User stories

**What to do:**
`capstone/stories.md`, at least 10 stories in "As a <role>, I want <action>, so that <outcome>" format. Group by role (owner, staff, customer).

**Expected output:**
Committed.

### Task 4: Four-week roadmap

**What to do:**
`capstone/ROADMAP.md`:
- Week 27 (this week): architecture, schema, API design
- Week 28: multi-tenant core + auth
- Week 29: external integrations (M-Pesa, WhatsApp, email)
- Week 30: launch, capstone demo day

Under each week, list concrete deliverables.

**Expected output:**
Committed.

### Task 5: Anti-scope

**What to do:**
`capstone/WILL_NOT_BUILD.md`, at least eight things you will explicitly NOT build for the MVP. This is the most important file of Day 1.

Examples:
- Native mobile apps
- Multi-language UI
- White-labelling
- Per-user permission matrices
- Advanced analytics

**Expected output:**
Committed.

## Stretch Goals (Optional - Extra Credit)

- Sketch the landing page hero and tagline.
- List three real target users you could interview.
- Draft the pricing page.

## Submission Requirements

- **What to submit:** Five markdown files under `capstone/`, `AI_AUDIT.md`.
- **Deadline:** End of Day 1

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Vertical picked | 10 | Defensible. |
| One-page spec | 25 | Tight, all questions answered. |
| User stories | 20 | 10+ grouped. |
| Roadmap | 20 | Weekly deliverables. |
| Anti-scope | 20 | 8+ items. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Scope the MVP by what is cool.** Scope by what the user pays for.
- **Multi-language + multi-currency + multi-everything.** Focus.
- **No anti-scope.** Without it, feature creep is certain.

## Resources

- Day 1 reading: [Capstone Kickoff.md](./Capstone%20Kickoff.md)
- Week 27 AI boundaries: [../ai.md](../ai.md)
