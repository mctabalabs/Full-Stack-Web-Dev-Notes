# Week 9 - AI Boundaries

**Ratio this week: 50% Manual / 50% AI**
**Habit introduced: "AI reads docs, you make decisions."**
**Shift from last week: The 50/50 line. This week is the first week where AI can write roughly as much code as you do -- but you own every architectural choice.**

You are halfway through the programme. Halfway in skill. Halfway in ratio. This week you reach the midpoint: APIs (fetching real data from external services), state management (one source of truth), custom hooks (your own reusable React primitives), and testing basics.

The habit is the most important one so far: **AI is your reader, summariser, and researcher. You are still the decider.** Every architectural choice -- "where does this state live", "what is the shape of this hook", "should this be split into two components" -- is yours. AI helps you implement what you decide; it does not decide for you.

---

## Why The Ratio Reached The Middle

By now you have a real React mental model. You can write a useState hook without thinking. You understand `useEffect` dependencies well enough to debug them. You can build a form. You know how routing works.

That foundation is the thing that lets you lean on AI without losing the thread. At 30% AI you would drown in new concepts; at 60% AI you would lose understanding. 50% is the sweet spot this week -- enough help to move faster, enough manual work to keep your brain sharp.

The catch: 50% is only sustainable if the decisions remain yours. If AI starts deciding where state should live and what your hooks should be called, you are no longer a 50/50 engineer -- you are a dictation-taker. The habit is the guardrail.

---

## What You Will Feel This Week

- You will write a custom hook and feel like you unlocked a secret room of React. You did.
- State management across multiple components will feel messy before it feels clean. Lift state up, pass it down, remember what it was for.
- Writing tests will feel slow and boring. It is slow and boring. It is also how real engineers sleep at night.
- You will be tempted to skip tests. Do not. Week 10 will punish you if you do.

---

## What You MUST Do Manually (50%)

### Day 1 -- Working with APIs properly
- Pick a public API (a different one from Week 6). Read its documentation end to end. This is non-negotiable -- no summarising by AI.
- Fetch from it in a React component using `useEffect`. Handle loading, error, and success states. All three must visibly render different UI.
- Build the AbortController pattern: cancel the fetch if the component unmounts before the response arrives. MDN has the exact snippet.

### Day 2 -- State management
- Read the React docs section on "Sharing State Between Components" (react.dev). No AI summary.
- Lift state up: take a state that lives in a child and move it to a parent so two siblings can share it. Do it manually first.
- Introduce React Context: create one context, provide it at the top of the app, consume it in a deeply nested child. Understand when context is overkill (most of the time) and when it is right (auth, theme, a few others).

### Day 3 -- Custom hooks and reusability
- Write your first custom hook: `useToggle(initial = false)` that returns `[value, toggle]`. Use it in two different components to prove it reuses.
- Write a `useLocalStorage(key, initial)` hook that wraps `useState` and syncs to localStorage. Use it to replace the localStorage effect in your Task Manager.
- Write a `useFetch(url)` hook that returns `{ data, loading, error }`. Use it to replace a manual fetch inside a component.

### Day 4 -- Testing basics
- Install Vitest (or Jest) manually. Read the getting-started page, not an AI summary.
- Write your first test: `expect(2 + 2).toBe(4)`. Run it. Watch it pass.
- Write a test for one of your pure utility functions. Run it. Make it fail by breaking the function. Fix it. Watch it pass again.
- Write one test for a React component using React Testing Library: render the component, find a button, simulate a click, assert the new state.

### Weekend -- Extend the multi-page Task Manager
- Add a real API: move your tasks from localStorage to a public backend (jsonplaceholder, supabase, or similar).
- Add one custom hook for data fetching.
- Add at least three tests: one for a utility, one for a component, one for a hook.

---

## You Must Break Things On Purpose

- Unmount a component mid-fetch. Observe the "cannot update state on unmounted component" warning and fix it with AbortController.
- Consume a context outside its provider. See the error.
- Call your custom hook inside an `if` block (breaking hook rules). See the lint error.
- Write a test that asserts the wrong thing. Watch it fail, then realise your test is wrong rather than your code.

Tests that never fail teach you nothing.

---

## What You CAN Use AI For (50%)

Cumulative permitted uses plus this week's new ones:

1. All previous permissions.
2. **Doc summarisation for your research** (new, constrained).
3. **Test generation from working code** (new, constrained).

### Doc summarisation (with the brake)

You may ask AI to summarise a section of docs after you have read them yourself. Example:

> "I just read the React docs page on 'Sharing State Between Components'. I understood the main idea (lift state to the common parent) but I am unsure when to use Context instead of lifting. In two bullet points, summarise the trade-off."

The rule: you must read first, then ask for a summary, then write your own one-sentence version in your notes. The AI summary is a check, not a substitute.

### Test generation (with the brake)

After you have written a component manually, you may ask AI to generate test cases for it. The rule: you must be able to run every test it generates, and you must reject tests that do not actually verify anything meaningful.

AI's generated tests often include useless ones like:

```js
test("renders without crashing", () => {
  render(<MyComponent />);
});
```

That test will never catch a real bug. Reject it. A real test looks like:

```js
test("clicking the button increments the counter", () => {
  render(<Counter />);
  const button = screen.getByRole("button", { name: /increment/i });
  fireEvent.click(button);
  expect(screen.getByText("Count: 1")).toBeInTheDocument();
});
```

Keep the good. Reject the theatre.

### Good vs bad prompts this week

**Bad:** "Manage my app state."
**Good:** "I have three components that all need the current user. Right now I pass `user` as a prop from App.jsx down through each one. Is this prop drilling painful enough to justify Context, or should I keep lifting?"

**Bad:** "Write me a useFetch hook."
**Good:** "I am designing a useFetch hook. My shape is `useFetch(url) -> { data, loading, error }`. Is there a standard argument I am missing (like refetch, options, or cleanup)?"

**Bad:** "Test my component."
**Good:** "Here is my TaskList component [paste]. It renders tasks, handles delete, and shows a confirmation. Write one test for the delete flow. Focus on what the user sees, not implementation details."

---

## The Decide-Yourself Habit

Three rules for the 50/50 week:

1. **AI never names your files.** When AI suggests `UserProfile.jsx`, you can accept the name. But when it invents `useUserProfileDataFetcher.jsx`, you say no.
2. **AI never picks where state lives.** If AI says "put this in a global store", you answer with "why?". If the answer is not "because three unrelated components need it", reject.
3. **AI never decides what to test.** "What should I test?" is an AI question. "Write a test for this scenario I picked" is a good one.

If you catch AI deciding, you are violating the habit. Your audit will show it.

---

## The 25-Minute Rule (Still)

Same as Week 8. By Week 10 the rule becomes softer -- "try until genuinely stuck" -- as you become fluent enough to know when more trying will not help. Use this week to build that self-awareness.

---

## Things AI Is Bad At This Week

- **Your app's state shape.** AI defaults to whatever pattern was most common in its training data. Your app is unique; its shape should reflect your data.
- **Context performance.** Context re-renders every consumer when the provider's value changes. AI rarely warns about this. If your context value is an object created fresh on every render, you are re-rendering everyone.
- **Hook dependencies.** Same warning as Week 8. The ESLint plugin is your friend; AI is a suspect.
- **Test assertions.** AI will generate tests that technically run but assert nothing meaningful. Reject theatre.

---

## Core Mental Models For This Week

- **State lives at the lowest common ancestor of every component that needs it.** That is the "lift state up" rule.
- **Context is a slower state mechanism for truly global values.** Not a replacement for props.
- **A custom hook is a function that starts with `use` and calls other hooks.** That is the entire definition.
- **A test is an assertion about behaviour, not implementation.** "When the user clicks X, they see Y" is a good test. "The state variable foo equals 42" is usually noise.

---

## This Week's AI Audit Focus

Add a section: **Decisions I owned.** List every architectural decision you made this week and where AI fit in (if at all):

```markdown
### Decision: Where does the current user state live?
- **My choice:** Context at App level
- **Alternative I considered:** Prop drilling from App.jsx
- **Why my choice:** Five deep components need it
- **Did AI decide for me?** No. I asked AI to compare the two options and picked based on depth count.
```

Three to five such decisions should appear in the audit. This is how facilitators verify you are still the decider.

---

## Assessment

Week 9 assessment is an architecture walkthrough:

- Walk through your Task Manager's state. Where does each piece live, and why?
- Show your `useFetch` (or equivalent) custom hook. Explain its interface choices.
- Show three tests. Explain what each one actually verifies.
- Live refactor: facilitator asks you to move one piece of state from a child to a parent. You do it live.
- Explain why context is not always the answer.

### Live Rebuild Check

Facilitator deletes your custom hook and asks you to rewrite it from memory. If you cannot, note it and come back with proof.

---

## One Sentence To Remember

"AI reads, researches, writes. You decide." The 50/50 line is fair only if decision-making stays on your side of it.
