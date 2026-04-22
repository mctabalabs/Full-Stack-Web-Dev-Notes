# useRef: Mutable References That Don't Trigger Re-Renders

## Learning Goals

By the end of this lesson, you should be able to:

- Explain what `useRef` does and how it differs from `useState`
- Use `useRef` to access DOM elements directly
- Use `useRef` to store mutable values that persist across renders (like interval IDs)
- Understand why `useRef` does not cause re-renders and when that matters
- Recognize the common patterns where `useRef` is the right tool

**Prior-week concepts you will use today:**
- useState and re-rendering (Week 7)
- useEffect and cleanup functions (Week 8, Day 2)
- setInterval and setTimeout (Week 3)

**Estimated time:** 1 hour

---

## 1. The Problem useRef Solves

You already know `useState`. When you call `setState`, React re-renders the component. That is exactly what you want for data that affects the UI -- when the count changes, the screen should update.

But sometimes you need to remember a value between renders **without** triggering a re-render. For example:

- A reference to a DOM element (to focus an input or measure its size)
- An interval or timeout ID (to clear it later in a cleanup function)
- A previous value of a prop or state (to compare "before" and "after")
- A flag that tracks whether the component is still mounted

`useRef` gives you a container that persists across renders but does not trigger a re-render when you change it.

---

## 2. How useRef Works

```javascript
import { useRef } from "react";

function MyComponent() {
  const myRef = useRef(initialValue);

  // Read the value
  console.log(myRef.current); // initialValue

  // Change the value (does NOT trigger a re-render)
  myRef.current = "something else";
}
```

`useRef` returns an object with a single property: `.current`. You read and write to `.current`. React guarantees that the same object is returned on every render -- it does not get recreated.

### useRef vs useState

| | `useState` | `useRef` |
|---|---|---|
| **Triggers re-render on change?** | Yes | No |
| **Persists between renders?** | Yes | Yes |
| **Used for UI-affecting data?** | Yes | No |
| **How to update** | `setState(newValue)` | `ref.current = newValue` |
| **Returns** | `[value, setter]` | `{ current: value }` |

> **Rule of thumb:** If changing a value should update what the user sees on screen, use `useState`. If the value is "behind the scenes" -- something your code needs but the user never sees -- use `useRef`.

---

## 3. Pattern 1: Accessing DOM Elements

The most common use of `useRef` is to get a direct reference to a DOM element. You attach the ref to a JSX element, and React fills in `ref.current` with the actual DOM node after rendering.

### Example: Auto-Focus an Input

```jsx
import { useRef, useEffect } from "react";

function SearchBar() {
  const inputRef = useRef(null);

  // Focus the input when the component first mounts
  useEffect(() => {
    inputRef.current.focus();
  }, []);

  return <input ref={inputRef} placeholder="Search..." />;
}
```

When React renders the `<input>`, it sets `inputRef.current` to the actual `<input>` DOM element. Then the `useEffect` runs and calls `.focus()` on it. The user sees the cursor appear in the input automatically.

### Example: Scrolling to an Element

```jsx
function ChatMessages({ messages }) {
  const bottomRef = useRef(null);

  // Scroll to the bottom whenever a new message arrives
  useEffect(() => {
    bottomRef.current.scrollIntoView({ behavior: "smooth" });
  }, [messages]);

  return (
    <div className="chat">
      {messages.map((msg) => (
        <p key={msg.id}>{msg.text}</p>
      ))}
      <div ref={bottomRef} />
    </div>
  );
}
```

The empty `<div>` at the bottom acts as a scroll target. When `messages` changes, the effect scrolls it into view.

---

## 4. Pattern 2: Storing Interval and Timeout IDs

This is the pattern you will use in the capstone project. When you start a `setInterval`, you get back an ID number. You need to store that ID so you can call `clearInterval(id)` later -- either when the component unmounts or when a condition is met.

Why not use `useState` for this? Because storing the interval ID does not affect what the user sees. You do not render the interval ID on screen. Calling `setState` would trigger an unnecessary re-render every time you start polling.

### Example: Polling for Payment Status

```jsx
import { useState, useEffect, useRef } from "react";

function PaymentStatus({ linkId }) {
  const [status, setStatus] = useState("pending");
  const pollingRef = useRef(null);

  useEffect(() => {
    // Start polling every 3 seconds
    pollingRef.current = setInterval(async () => {
      const result = await fetch(`/api/payment-status/${linkId}`);
      const data = await result.json();

      if (data.status === "paid") {
        clearInterval(pollingRef.current); // Stop polling
        setStatus("paid");                 // Update the UI
      }
    }, 3000);

    // Cleanup: if the component unmounts, stop polling
    return () => {
      if (pollingRef.current) {
        clearInterval(pollingRef.current);
      }
    };
  }, [linkId]);

  return <p>Payment status: {status}</p>;
}
```

Here is what happens step by step:

1. Component mounts. `useEffect` runs. `setInterval` starts and returns an ID (e.g., `42`).
2. We store `42` in `pollingRef.current`. No re-render happens.
3. Every 3 seconds, the interval callback fires and checks the payment status.
4. When the payment is confirmed, we call `clearInterval(pollingRef.current)` -- which is `clearInterval(42)` -- and the polling stops.
5. If the user navigates away before the payment completes, the cleanup function runs and clears the interval. No memory leak, no errors from updating unmounted state.

> **Why not just use a regular variable?** A regular `let intervalId` inside the component function would be reset to `undefined` on every render. The ref persists.

---

## 5. Pattern 3: Tracking Previous Values

Sometimes you need to compare the current value of a prop or state with its value from the previous render. `useRef` is perfect for this because you can update it without causing an extra re-render.

```jsx
import { useState, useEffect, useRef } from "react";

function PriceDisplay({ price }) {
  const prevPriceRef = useRef(price);

  useEffect(() => {
    prevPriceRef.current = price; // Update after each render
  });

  const priceWentUp = price > prevPriceRef.current;

  return (
    <p style={{ color: priceWentUp ? "green" : "red" }}>
      KES {price.toLocaleString()}
      {priceWentUp ? " (up)" : " (down)"}
    </p>
  );
}
```

The ref stores the "old" price. After each render, the effect updates it to the current price. On the next render, `prevPriceRef.current` holds the previous value while `price` holds the new one.

---

## 6. Pattern 4: Mounted Flag

When you have an async operation (like a fetch) that might complete after the component unmounts, updating state on an unmounted component causes a React warning. A ref can track whether the component is still around:

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const isMounted = useRef(true);

  useEffect(() => {
    async function loadUser() {
      const response = await fetch(`/api/users/${userId}`);
      const data = await response.json();

      // Only update state if the component is still mounted
      if (isMounted.current) {
        setUser(data);
      }
    }

    loadUser();

    return () => {
      isMounted.current = false;
    };
  }, [userId]);

  if (!user) return <p>Loading...</p>;
  return <p>{user.name}</p>;
}
```

> **Note:** This pattern is less necessary in React 18+ with StrictMode, but you will see it in production codebases and it is good to understand why it works.

---

## 7. Common Mistakes

### Mistake 1: Using useRef when you need useState

```jsx
// Wrong: Changing the ref does not update the screen
function Counter() {
  const countRef = useRef(0);

  function increment() {
    countRef.current += 1; // Value changes, but the UI does NOT update
  }

  return <p>Count: {countRef.current}</p>; // Always shows 0
}
```

If the value should be visible to the user, use `useState`.

### Mistake 2: Reading ref.current during render to make decisions

Refs are for side effects and event handlers, not for rendering logic. During render, use state and props. Refs should be read in `useEffect`, event handlers, or callbacks.

The `prevPriceRef` example above is a rare exception where reading a ref during render makes sense -- and even then, a library like React Query would handle this pattern for you in production.

---

## Quick Summary

| Use Case | Tool |
|---|---|
| Data that appears on screen | `useState` |
| Focus an input or scroll to an element | `useRef` with `ref={myRef}` |
| Store an interval/timeout ID | `useRef` |
| Track previous value of state or props | `useRef` |
| Track whether component is mounted | `useRef` |
| Store any mutable value that should not trigger re-renders | `useRef` |

---

## Activity

Build a simple stopwatch component:

1. A "Start" button that starts a `setInterval` incrementing a counter every second
2. A "Stop" button that clears the interval
3. A "Reset" button that clears the interval and sets the counter to 0

Use `useState` for the displayed counter (it appears on screen). Use `useRef` to store the interval ID (it does not appear on screen).

### Requirements:
1. The timer should display seconds elapsed
2. Clicking "Start" multiple times should not create multiple intervals (check if one is already running)
3. Navigating away from the component should clean up the interval (use `useEffect` cleanup)

### Stretch Goals:
- Add lap tracking: a "Lap" button that records the current time into an array
- Display laps as a list below the stopwatch
- Format the time as `MM:SS` instead of just seconds

---

## Key References

- [React Docs: Referencing Values with Refs](https://react.dev/learn/referencing-values-with-refs)
- [React Docs: Manipulating the DOM with Refs](https://react.dev/learn/manipulating-the-dom-with-refs)
