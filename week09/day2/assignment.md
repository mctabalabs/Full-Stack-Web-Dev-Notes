# Week 9 - Day 2 Assignment

## Title
State Management -- Lifting State, Context, and the "Is Context Overkill?" Question

## Overview
Today you practise the decision every React developer makes hundreds of times: where should this state live? You will lift state from a child to a shared parent, and then -- only when it is truly needed -- introduce React Context for values that many components share.

## Learning Objectives Assessed
- Lift state up to the lowest common ancestor of the components that need it
- Pass state down as props
- Create a Context and a Provider
- Consume a Context with `useContext`
- Decide when Context is overkill vs when it is justified

## Prerequisites
- Day 1 completed

## AI Usage Rules

**Ratio:** 50/50. **Habit:** AI reads docs, you make decisions. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Code help after you decided where state should live.
- **NOT ALLOWED FOR:** Deciding where state should live.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Lifting state up

**What to do:**
Build three components:

1. `TemperatureInput` -- a numeric input for entering temperature
2. `TemperatureDisplay` -- shows the temperature in both Celsius and Fahrenheit
3. `TemperaturePage` -- the parent holding both

Put the temperature state in `TemperaturePage` (the common parent). Pass it down as a prop to both children. When the input changes, the display updates.

```jsx
function TemperaturePage() {
  const [celsius, setCelsius] = useState(20);

  return (
    <div>
      <TemperatureInput celsius={celsius} onChange={setCelsius} />
      <TemperatureDisplay celsius={celsius} />
    </div>
  );
}
```

**Expected output:**
Typing in the input updates the display in both units.

### Task 2: Understand the decision

**What to do:**
In `lifting-notes.md`, answer:
- Why could you not just hold the state in `TemperatureInput`?
- What is the "lowest common ancestor" rule?
- When does lifting state up become painful?

Your own words.

**Expected output:**
Notes committed.

### Task 3: Introduce Context for a theme

**What to do:**
Create a Context for a light/dark theme. This is a case where Context is justified because many components far apart might need to know the current theme.

```jsx
// ThemeContext.jsx
import { createContext, useContext, useState } from "react";

const ThemeContext = createContext(null);

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState("light");
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error("useTheme must be inside ThemeProvider");
  return ctx;
}
```

Wrap your app in `<ThemeProvider>` in `main.jsx`. Use `useTheme()` in at least two components to show the theme.

**Expected output:**
Theme visible in two components. A button toggles between light and dark.

### Task 4: When NOT to use Context

**What to do:**
In `lifting-notes.md`, add a second section: "When Context is overkill". Write 4-5 sentences answering:
- What problem does Context solve that prop drilling does not?
- When is prop drilling fine and Context unnecessary?
- Why is "global state for everything" an anti-pattern?

**Expected output:**
Notes updated.

### Task 5: Decisions I owned

**What to do:**
Add another entry to the "Decisions I owned" section in `AI_AUDIT.md`. At least two decisions:
- Where did you place the temperature state and why?
- When did you reach for Context and when did you reject it?

**Expected output:**
Audit updated.

## Stretch Goals (Optional - Extra Credit)

- Persist the theme in localStorage so it survives reloads.
- Add a second Context for the current user. Compose providers.
- Benchmark the re-render cost of a Context value that changes every second.

## Submission Requirements

- **What to submit:** Repo with components, `lifting-notes.md`, `AI_AUDIT.md`.
- **Deadline:** End of Day 2

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Temperature lifting-up pattern | 20 | State in parent, passed down, both children reactive. |
| Lifting notes | 15 | Three questions answered in student's own words. |
| Theme Context with provider and hook | 25 | Context created, Provider wraps app, useTheme used in 2+ places. |
| Context-overkill notes | 15 | Three questions answered. |
| Decisions audit | 20 | At least two decisions owned and defended. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Lifting state too high.** If only one component needs the state, put it there. Do not promote it to the top just because you are nervous.
- **Using Context for everything.** Context re-renders every consumer when its value changes. For fast-changing state, prefer lifting up or a dedicated state library.
- **Forgetting the Provider.** `useContext(Ctx)` with no Provider gives you the default value, not an error. Guard with a custom hook that throws if not wrapped.

## Resources

- Day 2 reading: [State management.md](./State%20management.md)
- Week 9 AI boundaries: [../ai.md](../ai.md)
- react.dev: Sharing State Between Components: https://react.dev/learn/sharing-state-between-components
- react.dev: Passing Data Deeply with Context: https://react.dev/learn/passing-data-deeply-with-context
