# Async/Await & Error Handling

## Learning Objectives
By the end of this lesson, you will be able to:
- Convert `.then()` Promise chains into `async/await` syntax for cleaner, top-to-bottom code flow.
- Handle asynchronous errors effectively using `try/catch` blocks.
- Use `Promise.all` inside `async` functions to run tasks in parallel.
- Identify and avoid common pitfalls like forgetting `await`, using `forEach` incorrectly, and unhandled rejections.

---

## 1. `async` & `await` Basics
The `async` and `await` keywords are syntactic sugar on top of Promises. They allow you to write asynchronous, Promise-based code that *looks* and *behaves* like synchronous code.

### The Rules
1. **`async`**: Prepend this keyword to a function definition. It forces the function to *always* return a Promise.
2. **`await`**: Inside an `async` function, use `await` before an expression that returns a Promise. This pauses the execution of that specific function until the Promise settles (resolves or rejects).

### Example
```javascript
// 1. Define the async function
async function fetchJson(url) {
  // Execution pauses here until the fetch request completes
  const res = await fetch(url);
  
  // Execution pauses here until the data is parsed as JSON
  const data = await res.json();
  
  // This return value resolves the Promise returned by fetchJson()
  return data;
}

// 2. Consume it
fetchJson('/api/data')
  .then(data => console.log('Data received:', data))
  .catch(err => console.error('Fetch failed:', err));
```

> **Note:** `await` can **only** be used directly inside `async` functions. (The exception is modern ES modules, which support "top-level await").

---

## 2. Error Handling with `try/catch`
With `.then()` chains, you catch errors with `.catch()`. With `async/await`, you use standard JavaScript `try/catch` blocks.

If an awaited Promise rejects, it "throws" an error just like a synchronous runtime exception.

```javascript
async function getUserProfile(userId) {
  try {
    const res = await fetch(`/api/users/${userId}`);
    
    // Manually throw an error if the HTTP response isn't OK (e.g., 404, 500)
    if (!res.ok) {
      throw new Error(`Server returned status: ${res.status}`);
    }
    
    const profile = await res.json();
    return profile;
    
  } catch (err) {
    // This block catches network errors OR the manual error we threw above!
    console.error('Error fetching profile:', err.message);
    
    // Pro Tip: Rethrow the error if the function calling this needs to know it failed!
    throw err;  
  }
}

// Usage
getUserProfile(42)
  .then(profile => console.log('Profile loaded:', profile))
  .catch(err => console.log('UI Update: Could not load profile.')); // Catches the rethrown error
```

---

## 3. Sequential vs. Parallel Execution
One of the most important concepts to master with `async/await` is knowing when to pause your code.

### 3.1 Sequential `await` (Slower)
If you `await` requests one after the other, they run in sequence. Request 2 won't start until Request 1 finishes.
*Use this when operations depend on each other.*

```javascript
async function loadSequential() {
  const user = await fetchJson('/api/user'); // Wait 1 sec...
  
  // We NEED the user ID from the first request to fire the second request
  const orders = await fetchJson(`/api/orders?user=${user.id}`); // Wait 1 sec...
  
  return orders;
}
```

### 3.2 Parallel `await` with `Promise.all` (Faster)
If tasks are independent, start them at the same time and `await` `Promise.all()`.
*Use this when operations DO NOT depend on each other.*

```javascript
async function loadParallel() {
  // Both requests fire instantly at the exact same time!
  const [user, posts] = await Promise.all([
    fetchJson('/api/user'),
    fetchJson('/api/posts')
  ]);
  
  return { user, posts };
}
```

---

## 4. Common Pitfalls & Gotchas

### Pitfall 1: Forgetting `await`
If you forget `await`, the code doesn't wait for the Promise to resolve. It just assigns the pending Promise object directly to your variable.

```javascript
async function foo() {
  const data = fetchJson('/api/data'); // Forgot await!
  console.log(data); // Outputs: Promise { <pending> }
}
```

### Pitfall 2: `forEach` with `await`
`Array.prototype.forEach` does not wait for Promises. It will fire off all your async calls immediately without waiting for them to finish.

```javascript
const items = [1, 2, 3];

// BAD: The loop won't wait!
items.forEach(async (item) => {
  await doAsyncWork(item);
});

// GOOD: Use a for...of loop to await them sequentially
for (const item of items) {
  await doAsyncWork(item);
}
```

### Pitfall 3: Unhandled Rejections
If an `async` function throws an error and you didn't wrap it in a `try/catch` (or append a `.catch()` when calling it), you'll get an "Unhandled Promise Rejection." In Node.js, this can crash your entire application! Always handle your errors.

---

## 5. In-Class Activity: Refactoring to `async/await`

**Given the following messy Promise chain:**
```javascript
function step1() { return Promise.resolve('one'); }
function step2() { return Promise.resolve('two'); }
function step3() { return Promise.resolve('three'); }

step1()
  .then(res1 => {
    console.log(res1);
    return step2();
  })
  .then(res2 => {
    console.log(res2);
    return step3();
  })
  .then(res3 => console.log(res3))
  .catch(err => console.error('Error:', err));
```

**Task 1: Rewrite as a single `async` function using `try/catch`.**
```javascript
async function runSteps() {
  try {
    const res1 = await step1();
    console.log(res1);
    
    const res2 = await step2();
    console.log(res2);
    
    const res3 = await step3();
    console.log(res3);
  } catch (err) {
    console.error('Error caught in try/catch:', err);
  }
}
runSteps();
```

**Task 2: Enhance it to run `step2()` and `step3()` in parallel AFTER `step1()` finishes.**
```javascript
async function runStepsOptimized() {
  try {
    const res1 = await step1();
    console.log(res1);
    
    // Fire step 2 and 3 sequentially!
    const [r2, r3] = await Promise.all([step2(), step3()]);
    console.log(r2, r3);
  } catch (err) {
    console.error('Error:', err);
  }
}
runStepsOptimized();
```

---

## 6. Assignment: `withRetry` Helper

**Goal:** Implement an async utility function that attempts to run a flaky function. If it fails, it tries again.

**Requirements:**
1. Implement `async function withRetry(fn, attempts = 3, delayMs = 0)`.
2. It should `await fn()`.
3. On success, return the resolved value.
4. If it fails, wait `delayMs`, then try again (up to the max `attempts`).
5. On the final failure, `throw` the last error.

**Helper Code & Test Case:**
```javascript
// Provides a pause mechanism
function delay(ms) {
  return new Promise(res => setTimeout(res, ms));
}

// A mock function that fails twice, then succeeds on the 3rd try
let count = 0;
async function flakyTask() {
  count++;
  if (count < 3) throw new Error('Network timeout - Try again');
  return `Success on attempt ${count}!`;
}

// Your utility
async function withRetry(fn, attempts = 3, delayMs = 0) {
  // TODO: Implement the loop, try/catch, and delay logic here
}

// Testing the utility
withRetry(flakyTask, 5, 200)
  .then(res => console.log(res)) // Should print: "Success on attempt 3!"
  .catch(err => console.error('All attempts failed:', err.message));
```

---
*By mastering `async/await` and robust error handling, your asynchronous JavaScript becomes both readable and resilient—paving the way for our final foundation lesson: The Fetch API & JSON!*