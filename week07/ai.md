# Week 7 - AI Boundaries

**Ratio this week: 60% Manual / 40% AI**
**Habit introduced: "New framework, manual first."**
**Shift from last week: React arrives. This is a new mental model, so the habit tightens for the first half of the week and loosens for the second half.**

React is the biggest conceptual jump in the programme so far. JSX, components, props, state, hooks, events, lists. Everything you learned about the DOM in Week 4 is still true, but now the DOM is managed for you by a virtual layer that redraws itself when state changes. That idea is simple to describe and strange to internalise.

The habit: the first few components of every new framework get written by hand, with the docs open. Then, once the patterns click, AI can scaffold similar components at speed. New concept = manual. Repeated pattern = assisted.

---

## Why The Ratio Moved

Three weeks ago you were writing raw `document.getElementById` calls. React is a different way of thinking about the same problem. If AI writes your first `useState` call, your brain records "useState is the thing that makes the counter go up". If you write it yourself, your brain records "useState gives me a value and a setter, and calling the setter triggers a re-render". The second memory is what you need for the next 24 weeks.

40% AI is available because by the end of the week you will build a Task Manager that needs CRUD operations, filtering, and styling. All of that is boilerplate after your third component. AI can absolutely handle the boilerplate. It cannot handle your understanding of the mental model.

---

## What You Will Feel This Week

- You will write a counter with `useState`, see it work, and think "that is it? That is the whole trick?" Yes. That is the whole trick.
- You will forget to call `setCount(count + 1)` with the setter and instead type `count = count + 1`. The UI will not update. You will stare at it.
- You will accidentally create an infinite render loop (updating state inside the render body without a condition) and your browser will freeze. Welcome to React.
- Props drilling will make sense immediately and annoy you by Friday.
- The Task Manager project will feel like a real product for the first time this year.

---

## What You MUST Do Manually (60%)

### Day 1 -- React foundations
- Set up a Vite React project from the command line. No AI help with commands.
- Delete the starter template. Write a "Hello, World" component by hand.
- Understand `App.jsx`: why does it return JSX? What is JSX compiling into? (Call `React.createElement` manually at least once to see.)
- Write three function components with no props. Import them into `App.jsx`. Render them in order.

### Day 2 -- Components deep dive
- Pass props down. Start simple: a `Greeting` component that takes a `name` prop and renders `Hello, {name}`.
- Destructure props in the function signature. `function Greeting({ name }) { ... }`.
- Pass multiple props, including objects and arrays. Render them.
- Children props: `<Card><p>Inner content</p></Card>`. Understand `{children}` as a special prop.

### Day 3 -- State and events
- `useState` from scratch. Build a counter by hand. Increment, decrement, reset. All manual.
- Handle events: `onClick`, `onChange`, `onSubmit`. Each one with a named handler function.
- Controlled inputs: `<input value={name} onChange={e => setName(e.target.value)} />`. Type this pattern at least 5 times so your fingers learn it.

### Day 4 -- Rendering lists and managing UI logic
- Render an array with `.map()` returning JSX elements. Always include a `key` prop.
- Conditional rendering with `&&`, ternary, and early return. Use all three in different places.
- Build a working Task Manager: add task, complete task, delete task, filter by status. All four operations, all manual.

### Weekend -- Extend the Task Manager
- Only after the manual version works, use AI to add: search, category tags, local storage persistence, animations, nicer styling.

---

## You Must Break Things On Purpose

- Forget to return JSX from a component function. See the error.
- Mutate state directly (`state.push(newItem)` instead of `setState([...state, newItem])`). Observe that the UI does not update.
- Forget to add a `key` prop in a list and see the warning in the console. Understand why React wants it.
- Update state inside the render body (without `useEffect`). Observe the infinite loop.

React errors are unusually readable. Use them.

---

## What You CAN Use AI For (40%)

Permitted uses this week (cumulative):

1. **Explanations** (ongoing).
2. **Comparisons** (ongoing).
3. **Extending working code** (ongoing).
4. **Code review** (ongoing).
5. **Architecture sketches** (ongoing).
6. **Pattern-based component scaffolding** (new this week, constrained).

### Pattern-based scaffolding (new permission, read carefully)

After you have written 3+ components manually and clearly understand the props/state loop, you may ask AI to scaffold a new component that follows the same pattern. Example:

> "I have three components that take a title and a list of items as props and render them as a card. Here is one of them [paste]. I want a new component called `TodoList` that follows the exact same shape but takes a `tasks` array. Can you write it?"

The critical words: "exact same shape". You are asking AI to repeat a pattern you have already established. You are not asking it to invent new patterns.

If AI returns a component using hooks you have not learned yet (`useEffect`, `useContext`, `useReducer`), reject it. Explicitly say: "I only know `useState` -- rewrite using only that."

### Good vs bad prompts this week

**Bad:** "Build me a Task Manager in React."
**Good:** "Here is my working Task Manager [paste]. It has add, complete, delete, filter. I want to add a search input. What is the smallest change to `App.jsx` to support filtering by search text?"

**Bad:** "Fix my React state."
**Good:** "When I click my increment button, the count goes up by 2 instead of 1. I suspect a double render. Here is my component [paste]. Is StrictMode double-invoking something?"

**Bad:** "Make my UI look good."
**Good:** "Here is my TaskCard component [paste]. I want to add a subtle hover effect and a small badge showing the task category. What CSS or inline styles would you suggest?"

---

## The Manual-First Habit For New Frameworks

Every week from now on that introduces a new framework or library repeats this week's habit: the first 3-5 components, routes, files, or primitives you work with must be written by hand. No exceptions. Only after that 3-5 count does AI scaffolding turn on.

Why: frameworks are opinions in code. React has strong opinions about state, rendering, and composition. You cannot inherit those opinions through AI output. You have to type them yourself long enough for your brain to notice the shape.

Future weeks where this rule repeats: Week 8 (React routing), Week 10 (Express + PDF), Week 11 (WhatsApp API), Week 13 (USSD), Week 14 (Next.js), Week 20 (Telegram), Week 23 (BullMQ). Get used to it now.

---

## The 20-Minute Rule (Still)

Twenty minutes of real effort before asking. Same as Week 6. Keep MDN open -- and add the official React docs (react.dev) to your bookmarks. React's docs are also excellent, especially the "Learn" section. Open them before prompting.

---

## Things AI Is Bad At This Week

- **Stale closure bugs.** A classic: a `useEffect` captures an old state value because its dependency array is empty. AI will sometimes miss this unless you specifically ask about effect dependencies. Week 8 will deepen this -- for now, prefer writing effects without AI help.
- **Key prop mistakes.** AI will happily use array indexes as keys, which is a subtle anti-pattern. Use real ids when possible and audit generated lists.
- **Your file structure.** "Where should this component live?" is a taste question. AI will default to whatever its training data shows. Ignore and organise the way your project wants.
- **Hook order.** Hooks must be called in the same order every render. AI sometimes suggests conditional hooks (inside `if` blocks). This is always wrong. If AI ever does this, reject the suggestion and note it in the audit.

---

## Core Mental Models For This Week

- **A component is a function that returns JSX.** That is it. Everything else is variations.
- **Props are inputs to a component.** Immutable from the child's view; passed down from the parent.
- **State is a value that, when changed, triggers a re-render.** That is the whole magic.
- **React re-renders on state change.** Your job is to make state changes meaningful and rendering cheap.

The loop is: state -> render -> user acts -> state changes -> re-render. Internalise the loop before you internalise anything else.

---

## This Week's AI Audit Focus

Add a required section: **Pattern scaffolding log.** For every time you asked AI to scaffold a component by pattern, record:

```markdown
### [Component name]
- **Pattern I had already established:** [paste of the reference component]
- **Prompt used:** [...]
- **What AI returned:** [brief summary]
- **Did it use hooks I have not learned yet?** [yes/no -- if yes, how I corrected it]
- **Changes I made before pasting it in:** [bulleted list]
```

The "hooks I have not learned yet" line is specifically designed to catch AI reaching ahead. Facilitators will read it.

---

## Assessment

Week 7 assessment is a live component rebuild:

- Facilitator gives you a specific task: "Build a component called `Badge` that takes a `label` and a `colour` prop, renders it with a coloured background, and handles click events by calling an `onClick` prop." You have 10 minutes. No AI.
- Walk through your Task Manager. Explain why state lives where it lives. What would change if you moved it up or down?
- Live change: "make the filter also search task descriptions, not just titles". You do it on screen.
- Explain props vs state in your own words, 30 seconds.
- Show one AI pattern-scaffold example and defend your corrections.

### Live Rebuild Check

Facilitator deletes your `useState` hook from the counter and asks you to rewrite it. If you cannot, note it and come back next week with proof.

---

## One Sentence To Remember

"Type the framework into your fingers before you let AI type it for you." Three components by hand is a small tax for twenty-four weeks of smooth sailing.
