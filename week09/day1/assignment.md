# Week 9 - Day 1 Assignment

## Title
Working With APIs Properly -- Loading, Error, Empty, and Refetch

## Overview
Week 9 makes React a real front-end for real data. Today you build a component that fetches from a public API, handles all four UI states (loading, error, empty, success), and supports a refetch button. This is the pattern you will reuse for every data-bound component from now on.

## Learning Objectives Assessed
- Fetch data in `useEffect` on mount
- Track loading, error, empty, and success states explicitly
- Implement a refetch function without duplication
- Avoid stale state on rapid refetches
- Use AbortController for cleanup

## Prerequisites
- Week 8 completed

## AI Usage Rules

**Ratio this week:** 50% manual / 50% AI
**Habit:** AI reads docs, you make decisions. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Implementation help after you decided the state shape.
- **NOT ALLOWED FOR:** Deciding the state shape or naming the hook. Those are your decisions.
- **AUDIT REQUIRED:** Yes. Include a "Decisions I owned" section this week.

## Tasks

### Task 1: Decide the state shape first

**What to do:**
Before writing any code, in `state-plan.md`, write down:

- What state do you need? (Examples: `data`, `loading`, `error`.)
- Is it one object or multiple useState calls? Why?
- How will you represent "empty" vs "not loaded yet"?
- What triggers a refetch?

No AI for this planning step. Your own decisions.

**Expected output:**
`state-plan.md` committed.

### Task 2: Build the component

**What to do:**
Create `src/components/UserList.jsx` that fetches users from `https://jsonplaceholder.typicode.com/users`:

```jsx
import { useState, useEffect } from "react";

function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  async function loadUsers() {
    setLoading(true);
    setError(null);
    try {
      const res = await fetch("https://jsonplaceholder.typicode.com/users");
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      const data = await res.json();
      setUsers(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }

  useEffect(() => {
    loadUsers();
  }, []);

  if (loading) return <p>Loading users...</p>;
  if (error) return <p>Error: {error}</p>;
  if (users.length === 0) return <p>No users.</p>;

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
      <button onClick={loadUsers}>Refresh</button>
    </ul>
  );
}

export default UserList;
```

Type it yourself.

**Expected output:**
Loading state visible briefly, then the list of users, with a working refresh button.

### Task 3: AbortController for cleanup

**What to do:**
Add an AbortController so the fetch is cancelled if the component unmounts before the response arrives:

```jsx
useEffect(() => {
  const controller = new AbortController();

  async function load() {
    try {
      const res = await fetch("https://jsonplaceholder.typicode.com/users", {
        signal: controller.signal,
      });
      // ... rest same
    } catch (err) {
      if (err.name === "AbortError") return; // expected
      setError(err.message);
    }
  }

  load();

  return () => controller.abort();
}, []);
```

**Expected output:**
Unmounting the component during load no longer fires the "can't update state on unmounted" warning.

### Task 4: Test all four states

**What to do:**
Deliberately trigger each UI state and take a screenshot of each:
1. **Loading:** Simulated by throttling the network in DevTools. Screenshot `day1-loading.png`.
2. **Error:** Break the URL temporarily. Screenshot `day1-error.png`.
3. **Empty:** Point fetch at an endpoint that returns `[]` (or simulate with `setUsers([])`). Screenshot `day1-empty.png`.
4. **Success:** Screenshot `day1-success.png` of the user list.

Restore the working code after.

**Expected output:**
Four screenshots in `assets/`.

### Task 5: Decisions I owned

**What to do:**
In `AI_AUDIT.md`, start a new section called "Decisions I owned". List every architectural decision you made today:
- State shape (one object or multiple useState?)
- Loading strategy (initial loading = true or false?)
- Error handling (throw on non-OK? catch-all or specific?)
- Cleanup strategy (AbortController or ignore stale?)

For each, record the decision, the alternative, and why you picked yours.

**Expected output:**
Audit entry with 3-5 owned decisions.

## Stretch Goals (Optional - Extra Credit)

- Add a search input that filters the user list by name (client-side).
- Cache the data so a refetch is only needed after X minutes.
- Display an API error message from the response body when the request fails with a body.

## Submission Requirements

- **What to submit:** Repo with UserList, `state-plan.md`, 4 screenshots, `AI_AUDIT.md`.
- **Deadline:** End of Day 1

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| State plan written before coding | 15 | state-plan.md real and thoughtful. |
| UserList with all 4 states | 25 | Loading, error, empty, success all rendering correctly. |
| Refresh button | 10 | Click refetches without page reload. |
| AbortController cleanup | 15 | No unmounted-component warnings. |
| Four state screenshots | 10 | All four in assets/. |
| Decisions I owned audit section | 20 | 3-5 decisions with alternatives and reasoning. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Starting `loading` at false.** Then the user sees "empty" before the fetch even starts. Always start loading at true or use a "not started" state.
- **Not handling `response.ok`.** `fetch` only rejects on network failure, not on HTTP errors. You must check `res.ok` yourself.
- **Missing AbortController cleanup.** Leads to the classic warning. Always cancel in the cleanup function.

## Resources

- Day 1 reading: [Working with APIs properly.md](./Working%20with%20APIs%20properly.md)
- Week 9 AI boundaries: [../ai.md](../ai.md)
- MDN AbortController: https://developer.mozilla.org/en-US/docs/Web/API/AbortController
