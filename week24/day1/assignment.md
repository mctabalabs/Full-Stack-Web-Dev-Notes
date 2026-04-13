# Week 24 - Day 1 Assignment

## Title
Why Microservices -- And Why Usually Not Yet

## Overview
Week 24 is an honest look at microservices. Today is theory: when a split is worth it, when it is premature, and how to recognise a "distributed monolith". You write a decision document for your own project.

## Learning Objectives Assessed
- Describe the tradeoffs of microservices vs modular monolith
- Identify when a split is justified
- Spot a "distributed monolith" anti-pattern
- Write a defensible architecture decision

## Prerequisites
- Weeks 1-23 completed

## AI Usage Rules

**Ratio this week:** 45% manual / 55% AI
**Habit:** Architecture dip -- you drive the thinking. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Summarising external articles you cite.
- **NOT ALLOWED FOR:** Your opinion. The ADR is yours.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Read two opposing views

**What to do:**
Read:
- Martin Fowler: "Monolith First"
- Any pro-microservices piece (Sam Newman, or a real company postmortem)

Summarise each in three sentences inside `reading-notes.md`.

**Expected output:**
`reading-notes.md` committed.

### Task 2: When to split

**What to do:**
In `when-to-split.md`, write the conditions under which you would split a service. Your own list. At least six items, with a one-line justification each. Examples:
- Different scaling profile (CPU vs IO)
- Different data ownership
- Different release cadence
- Different team ownership
- Different compliance boundary
- Different language/runtime required

**Expected output:**
Committed.

### Task 3: Distributed monolith anti-pattern

**What to do:**
In `distributed-monolith.md`, explain what it is in 5-7 sentences:
- Services that must all deploy together
- Services that share a database
- Services that call each other synchronously in long chains
- Why it is worse than a monolith (all the costs of both, benefits of neither)

**Expected output:**
Committed.

### Task 4: Your ADR

**What to do:**
Write `docs/adr/0002-split-or-not.md`. For your current project (Mctaba shop), decide:
- Do you split anything now? (Probably not.)
- If yes, what service and why?
- If no, what would you monitor that would later justify a split?

Be specific. Cite your own metrics/load profile (even if made up, at least consistent).

**Expected output:**
ADR committed.

### Task 5: Module boundaries exercise

**What to do:**
In `module-boundaries.md`, list the modules inside your backend that COULD become services:
- notifications
- payments
- leads
- auth

For each, answer: what does it own, what does it depend on, what would break if it deployed separately?

**Expected output:**
Committed.

## Stretch Goals (Optional - Extra Credit)

- Read "Building Microservices" chapter 1-2 and write a one-page summary.
- Find a Kenyan tech company blog post about their architecture and write a reaction.
- Sketch a "strangler fig" migration path for your project.

## Submission Requirements

- **What to submit:** Five notes files, `AI_AUDIT.md`.
- **Deadline:** End of Day 1

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Reading notes | 15 | Two sources summarised. |
| When to split | 20 | Six items with reasoning. |
| Distributed monolith | 15 | Explained clearly. |
| ADR 0002 | 30 | Specific, defensible. |
| Module boundaries | 15 | All four modules analysed. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Splitting because "microservices are modern".** Never a reason.
- **Treating module boundaries as service boundaries.** They are not the same.
- **Copying an ADR template without filling in specifics.** It is your project, not a sample.

## Resources

- Day 1 reading: [Why Microservices.md](./Why%20Microservices.md)
- Week 24 AI boundaries: [../ai.md](../ai.md)
- Martin Fowler Monolith First: https://martinfowler.com/bliki/MonolithFirst.html
