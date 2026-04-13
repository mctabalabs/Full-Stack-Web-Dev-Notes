# Week 4 - AI Boundaries

**Ratio this week: 75% Manual / 25% AI**
**Habit introduced: "Build first, enhance with AI."**
**Shift from last week: Another 5% AI room, and the 15-minute rule stays. But now you are allowed to ask AI to *extend* code you already wrote, not just explain it.**

This is the first week where you will hear yourself say "this is a skill I will actually use at work". Functions are the unit of professional code. DOM manipulation is where JavaScript leaves the terminal and starts moving things on a real page. Closures are the weirdest idea you will meet this week and the most important one.

The habit is subtle but big: **the manual version must exist and work before AI touches it.** AI extends. It never originates.

---

## Why The Ratio Moved

Functions, arrow functions, rest/default parameters, closures, scope chains, and DOM manipulation -- that is a lot of new concepts in one week. You will need AI help before Friday. Accept that.

But every piece of this week's syllabus has the same shape: a mental model that is surprising the first time you meet it. AI cannot grow that model for you. If you let AI write your first function, your brain records "functions are mysterious things AI makes". If you write your own first function, your brain records "a function is a recipe I can call repeatedly, and the thing between the parentheses is a handoff".

The first recording stays with you for three decades. The second one evaporates in a day.

---

## What You Will Feel This Week

- You will write your first function and feel like you are cheating the computer. "I can... name this and use it later? That is it?"
- You will write an arrow function and not understand why anyone would use it over a regular function.
- You will meet closures and feel genuinely confused for ten minutes, then feel like a magician for the next ten.
- The calculator UI lab will work, then break when you change one small thing, then work again for reasons you do not fully understand.
- By Sunday night you will forget what life was like before functions.

---

## What You MUST Do Manually (75%)

### Day 1 -- Function declarations and expressions
- Write 10 functions. Each one takes parameters, does something, returns a value. Each one runs successfully at least twice with different arguments.
- Named functions, anonymous functions, IIFEs -- type at least one of each.
- Call a function before it is declared (hoisting). Try the same with a function expression. Observe the difference.

### Day 2 -- Arrow functions and implicit returns
- Convert every function from Day 1 into an arrow function. Keep both versions side by side.
- Implicit vs explicit return: write the same function both ways. Which feels clearer for your specific example?
- When does `this` behave differently in an arrow function vs a regular function? Write one example you can remember.

### Day 3 -- Parameters: rest and default values
- Write a function with 3 parameters, each with a default value. Call it with 0, 1, 2, and 3 arguments. Observe.
- Write a function that takes `...args` and sums them. Call it with `sum(1)`, `sum(1,2,3)`, `sum(1,2,3,4,5)`.
- Destructure an object parameter: `function greet({ name, age }) { ... }`. Call it with an object.

### Day 4 -- Closures and scope chains
- Write a counter function that uses a closure. It returns a function that increments and returns a private count. Run it three times and observe that the count is remembered.
- Draw the scope chain on paper for one of your functions. Global -> outer -> inner. Label each variable with the scope it lives in.
- Read your own code out loud and say where each variable is visible. "This `x` is only visible inside this function. This `y` is visible everywhere."

### Day 5 -- DOM manipulation
- Select elements with `document.getElementById`, `querySelector`, `querySelectorAll`. Minimum 10 selections on a test HTML page.
- Change text content, change styles, add/remove classes. Minimum 10 manipulations.
- Attach event listeners: `click`, `input`, `submit`. Each one with an inline function first, then refactored to a named function.

### Lab -- Simple Calculator UI
- Build the calculator from the curriculum brief manually. Every button, every handler, every function.
- No AI until the calculator can add, subtract, multiply, and divide two numbers with displayed output.
- Only after the base calculator works are you allowed to use AI to extend it (percent button, memory, history panel, theme toggle).

---

## You Must Break Things On Purpose

- Forget to `return` from a function and observe what the caller receives (`undefined`).
- Call `document.getElementById("thing")` on an element that does not exist. What is the return value?
- Inside a closure, change the inner variable name to collide with an outer one. Observe the shadowing.
- Attach two click listeners to the same button and click once. Which runs first? Do they both run?

Every surprise is a lesson. Write down what you expected and what happened.

---

## What You CAN Use AI For (25%)

Three permitted uses this week:

1. **Explaining things you already tried** (ongoing from Weeks 1-3).
2. **Comparison** (ongoing from Week 2).
3. **Extending working code with features you did not originate** (new this week).

### Extending working code (the new permission)

You built the calculator. It works. Adding a new feature -- say, a history of previous calculations -- is a legitimate "extend with AI" moment. The rule: the base must work first, and you must read every line of AI-generated code and understand it before pasting it in.

A good extension prompt looks like:

> "Here is my working calculator [paste]. It handles four operations. I want to add a history panel that shows the last 5 calculations. What is the smallest change I can make to my code to support this?"

The words "smallest change I can make" are important. You are asking AI to respect your architecture, not rewrite it.

### Good vs bad prompts this week

**Bad:** "Write a calculator in JavaScript."
**Good:** "I have a working calculator [paste]. When I click `=` a second time after getting a result, nothing happens. What am I missing in my state handling?"

**Bad:** "Explain closures."
**Good:** "I expected this counter [paste] to print 1, 2, 3 when called three times. It prints 1, 1, 1. I think the `count` variable is being re-initialised each call. What should I do instead?"

**Bad:** "Make my DOM code better."
**Good:** "I have three separate `addEventListener` calls for three similar buttons [paste]. Is there a way to attach one handler and detect which button was clicked? What is the standard JavaScript pattern for this?"

---

## The Build-First Habit

The shape every time you add a new feature this week:

1. **Write the feature description.** One sentence. Before code.
2. **Pseudocode it.** Three to five lines in a comment. Not real code; just the shape.
3. **Write the manual version.** Real code. Run it. Debug it until it works.
4. **Only now ask AI.** Either for comparison ("is there a simpler pattern for this?") or for extension ("how would I add X on top of this?").

If you skip step 3 and go straight from pseudocode to AI, you have failed the habit. The facilitator will spot it in your AI Audit because your "what I already tried" answer will be empty.

---

## The 15-Minute Rule (Still)

Same rule as Week 3. Fifteen minutes of trying before you ask. Next week it becomes twenty.

One new trick for this week: before you type a prompt, **explain your bug to a rubber duck** (or a cup, or a friend who does not code). Saying the problem out loud solves half of the bugs this week on its own. Prompt AI only if the duck cannot help.

---

## Things AI Is Bad At This Week

- **Your exact HTML.** AI does not know your element IDs, class names, or structure unless you paste them. It will suggest `getElementById("calculator-display")` when your element is actually `calc-screen`.
- **Event handler context.** `this` inside an event handler is not what you think. AI will often give you confidently wrong answers about `this` if you do not specify whether you are using an arrow function or a regular function.
- **Closure subtleties.** A classic bug: a loop that creates functions that all capture the same loop variable. AI will sometimes miss this unless the question is very specific. This is a Week 4 classic -- expect to hit it.
- **Whether your code is "good".** Code can work and still be bad. AI will usually say code is fine if it runs. Only you can ask "is this the shape I want my code to take?"

---

## Core Mental Models For This Week

- **A function is a recipe** with ingredients (parameters) and a result (return value). Everything else is variations.
- **Scope is visibility.** A variable lives in the innermost function that declared it. Inner functions can see outer ones, but not the other way around.
- **A closure is a function that remembers.** It holds onto variables from the scope where it was created, even after that scope has returned.
- **The DOM is a tree.** Every element is a node. Your JavaScript asks the browser for a node, changes its properties, or attaches a listener. That is all DOM work.

If you get lost, come back to these four. Four mental models are your entire toolkit this week.

---

## This Week's AI Audit Focus

Add one required section to your `AI_AUDIT.md`: **The build-first log.**

For every feature you added with AI help, prove you built a manual version first. Format:

```markdown
### [Feature name]
- **My pseudocode:** [the 3-5 line comment version]
- **My first working version:** [link to the commit or paste the code]
- **What I asked AI to add:** [one sentence]
- **What I kept from AI's version:** [1-3 bullets]
- **What I rejected:** [if anything]
```

An AI Audit that jumps straight from "prompt" to "final code" will cost you marks. The build-first log is your evidence.

---

## Assessment

Week 4 assessment is a live coding session:

- Walk through your calculator on screen. Explain one function of your choice.
- Facilitator asks: "rename this variable from `x` to `result` everywhere it is used." Do it live. No find-and-replace; you must understand where it is used.
- Facilitator asks: "add a clear button that resets the display." You build it live, with AI allowed only if you get stuck for 15 minutes. AI usage is noted.
- Explain closures in thirty seconds using your counter example.
- Explain why arrow functions behave differently with `this`.

### Live Rebuild Check

Facilitator may delete your event listener code and ask you to re-attach it from memory. If you cannot, that means you copied listeners from AI without internalising. Come back next week with the rewrite.

---

## One Sentence To Remember

"AI extends. You originate." The manual version must exist and work before AI touches it -- for every feature, every week, from now until Week 30.
