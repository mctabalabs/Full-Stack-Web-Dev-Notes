# Week 9 - Day 3 Assignment

## Title
Custom Hooks -- Extracting Reusable Logic

## Overview
Today you build your first custom hooks. A custom hook is just a function that starts with `use` and calls other hooks -- that is the entire definition. Custom hooks are how you share behaviour across components without duplicating code or inventing class hierarchies.

## Learning Objectives Assessed
- Write a custom hook that wraps `useState`
- Write a custom hook that wraps `useEffect`
- Replace duplicated component logic with a single hook call
- Name hooks clearly (always starting with `use`)

## Prerequisites
- Days 1-2 completed

## AI Usage Rules

**Ratio:** 50/50. **Habit:** AI reads docs, you make decisions. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Implementation help after you named and sketched the hook interface.
- **NOT ALLOWED FOR:** Naming or designing the hook interface for you.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: useToggle hook

**What to do:**
Create `src/hooks/useToggle.js`:

```jsx
import { useState } from "react";

export function useToggle(initial = false) {
  const [value, setValue] = useState(initial);
  const toggle = () => setValue((v) => !v);
  return [value, toggle];
}
```

Use it in two different components to prove reusability:

```jsx
import { useToggle } from "./hooks/useToggle";

function Modal() {
  const [isOpen, toggle] = useToggle(false);
  return (
    <>
      <button onClick={toggle}>{isOpen ? "Close" : "Open"}</button>
      {isOpen && <div>Modal content</div>}
    </>
  );
}

function DarkModeSwitch() {
  const [isDark, toggle] = useToggle(false);
  return <button onClick={toggle}>{isDark ? "Light" : "Dark"}</button>;
}
```

**Expected output:**
Both components use the same hook and work independently.

### Task 2: useLocalStorage hook

**What to do:**
Create `useLocalStorage(key, initialValue)`:

```jsx
import { useState, useEffect } from "react";

export function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initialValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue];
}
```

Use it to replace the manual localStorage code in your Week 7 Task Manager:

```jsx
const [tasks, setTasks] = useLocalStorage("tasks", []);
```

**Expected output:**
Task Manager tasks persist across reloads without any direct localStorage code in the component.

### Task 3: useFetch hook

**What to do:**
Create `useFetch(url)` that returns `{ data, loading, error, refetch }`:

```jsx
import { useState, useEffect, useCallback } from "react";

export function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  const fetchData = useCallback(async () => {
    setLoading(true);
    setError(null);
    try {
      const res = await fetch(url);
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      setData(await res.json());
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, [url]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return { data, loading, error, refetch: fetchData };
}
```

Replace the manual fetch in your Day 1 `UserList` with `useFetch`.

**Expected output:**
UserList still works but the fetching logic now lives in a reusable hook.

### Task 4: Rules of hooks

**What to do:**
In `hooks-notes.md`, write 4-6 sentences on the rules of hooks:
- Why must hooks be called at the top level (never inside conditions or loops)?
- Why must hooks start with `use`?
- What does the ESLint plugin enforce?

Read the react.dev rules page first. No AI paraphrasing.

**Expected output:**
Notes committed.

### Task 5: Reflection

**What to do:**
In `hooks-notes.md`, add one more section: what was the "aha" moment? When did the pattern click? Four sentences.

**Expected output:**
Notes updated.

## Stretch Goals (Optional - Extra Credit)

- Write `useDebounce(value, delay)` that returns a debounced version of a value. Use it in a search input.
- Write `useWindowSize()` that returns the current window width and height.
- Write `useInterval(callback, delay)` that runs a function on an interval with cleanup.

## Submission Requirements

- **What to submit:** Repo with three hooks, components using them, `hooks-notes.md`, `AI_AUDIT.md`.
- **Deadline:** End of Day 3

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| useToggle hook | 20 | Working, used in two components. |
| useLocalStorage hook | 25 | Working, Task Manager uses it. |
| useFetch hook | 25 | Returns data/loading/error/refetch. UserList refactored to use it. |
| Rules of hooks notes | 15 | Three questions answered in student's own words. |
| Clean commits | 10 | Conventional messages. |
| Code quality | 5 | Hooks named with `use` prefix, placed in /hooks folder. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Calling a hook conditionally.** `if (condition) useSomething()` breaks the rules of hooks. React relies on call order. Always call at the top level.
- **Naming a hook without `use`.** The ESLint plugin will not lint it. Hook naming is a contract: all hooks start with `use`.
- **Storing stale closures in custom hooks.** If `useFetch` captures an old `url`, refactor to pass it as a parameter or use a ref.

## Resources

- Day 3 reading: [Custom Hooks and Reusability.md](./Custom%20Hooks%20and%20Reusability.md)
- Week 9 AI boundaries: [../ai.md](../ai.md)
- react.dev Rules of Hooks: https://react.dev/warnings/invalid-hook-call-warning
- react.dev Reusing Logic with Custom Hooks: https://react.dev/learn/reusing-logic-with-custom-hooks
