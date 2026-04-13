# Week 3 - Day 4 Assignment

## Title
Control Flow -- Conditionals, Loops, and the Number Guessing Game Setup

## Overview
Day 4 is pre-weekend polish day. Today you learn the control-flow primitives that make programs do different things based on inputs: `if`/`else`/`else if`, `switch`, `for`, `while`, `do...while`, and `break`/`continue`. You will complete a battery of small drills, then begin the weekend Number Guessing Game by writing its core conditional and loop structure.

## Learning Objectives Assessed
- Use `if`/`else if`/`else` chains to express branching logic
- Use `switch` appropriately (and know when `if` is better)
- Choose the right loop for the job (`for`, `while`, `do...while`)
- Use `break` and `continue` correctly
- Begin the weekend project with a working skeleton

## Prerequisites
- Days 1-3 completed

## AI Usage Rules

**Ratio:** 80/20. **Habit:** 15-minute rule. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Explaining a loop you already wrote. Explaining why a condition did not fire.
- **NOT ALLOWED FOR:** Generating the Number Guessing Game core logic. Writing your control-flow drills.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: if/else chain drills

**What to do:**
Create `js-lab/day4/main.js`. Write the following conditionals by hand:

1. A function `gradeFor(score)` that takes a score 0-100 and returns a letter grade (A >= 90, B >= 80, C >= 70, D >= 60, F < 60).
2. A function `seasonFor(month)` (1-12) that returns "Long rains", "Cool dry", "Short rains", or "Hot dry" based on Kenyan seasons.
3. A function `canVote(age, citizenship)` that returns true only if age >= 18 AND citizenship is "Kenyan".

Test each function with at least three different inputs.

**Expected output:**
All three functions working with sample calls and console outputs visible.

### Task 2: switch vs if

**What to do:**
Write a function `dayOfWeek(num)` that returns the day name for numbers 1-7 (1=Monday). Write it once with `switch` and once with an `if`/`else if` chain. In a comment, say which you prefer for this case and why.

**Expected output:**
Two working versions of the same function.

### Task 3: Loop drills

**What to do:**
Complete each of these as separate exercises in the same file:

1. Use a `for` loop to print numbers 1 to 20.
2. Use a `while` loop to count down from 10 to 1.
3. Use a `do...while` loop that runs at least once even when its condition is false from the start.
4. Use a `for` loop with `break` that stops at the first multiple of 7 between 1 and 100.
5. Use a `for` loop with `continue` that prints only odd numbers from 1 to 20.
6. FizzBuzz for 1 to 30: print "Fizz" for multiples of 3, "Buzz" for multiples of 5, "FizzBuzz" for multiples of both, and the number otherwise.

**Expected output:**
All six drills working, outputs visible in the console.

### Task 4: Start the Number Guessing Game

**What to do:**
In a new file `game.js`, begin the core of the weekend project. Today you write only the skeleton -- the full game ships this weekend. At minimum:

1. Generate a random number between 1 and 100:

```javascript
const secret = Math.floor(Math.random() * 100) + 1;
```

2. Use `prompt` (in the browser via a small HTML wrapper, or `readline` in Node -- your choice) to ask for a guess.
3. Use an `if`/`else if`/`else` to compare the guess to the secret and print "Too high", "Too low", or "Correct!".
4. Wrap the comparison in a loop so the user can guess until correct.
5. Count the number of guesses.

You do NOT need to ship the full game today. A working core loop is enough.

**Expected output:**
Running `game.js` asks for a guess, compares to secret, loops until correct, and shows the guess count.

### Task 5: Pre-weekend checklist

**What to do:**
Create `CHECKLIST.md` at the repo root with:

```markdown
## Week 3 Day 4 Pre-Weekend Checklist

- [ ] All Day 1-3 drills saved and committed
- [ ] Day 4 control flow drills all passing
- [ ] game.js has a working guess loop
- [ ] game.js uses if/else if/else for comparison
- [ ] game.js counts guesses
- [ ] AI_AUDIT.md includes every 15-minute-rule entry for Week 3
- [ ] Repo is pushed and clean
```

Tick each box honestly.

**Expected output:**
`CHECKLIST.md` committed with honest ticks.

## Stretch Goals (Optional - Extra Credit)

- Add a "give up" option that reveals the secret if the user types "quit".
- Add a maximum-guesses limit and print "Game over" when reached.
- Add input validation -- handle the case where the user types something that is not a number.

## Submission Requirements

- **What to submit:** Repo link, all Day 4 files, `game.js`, `CHECKLIST.md`, updated `AI_AUDIT.md`.
- **Where:** Week 3 submissions channel
- **Deadline:** End of Day 4

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Three conditional functions | 20 | gradeFor, seasonFor, canVote all working with three test cases each. |
| switch vs if comparison | 10 | Two versions of dayOfWeek. Preference stated with reasoning. |
| Six loop drills | 20 | All six drills producing correct output. |
| Number Guessing Game skeleton | 20 | game.js runs, loops until correct guess, counts attempts. |
| Pre-weekend checklist | 10 | CHECKLIST.md honest and complete. |
| AI Audit with 15-minute entries | 15 | All AI interactions documented with "What I tried". |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Infinite loops.** A `while` loop that never updates its condition runs forever. Always double-check your condition updates inside the loop. If your browser tab freezes, you have an infinite loop. Use Ctrl+C in the terminal to kill Node.
- **Off-by-one errors.** `for (let i = 1; i < 20; i++)` stops at 19, not 20. Decide carefully: `<` or `<=`?
- **`switch` without `break`.** Without `break`, execution falls through to the next case. Always include `break` unless fall-through is intentional.

## Resources

- Day 4 reading: [control flow.md](./control%20flow.md)
- Weekend project: [../weekend/project-number-guessing-game.md](../weekend/project-number-guessing-game.md)
- Week 3 AI boundaries: [../ai.md](../ai.md)
- MDN Loops and iteration: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Loops_and_iteration
