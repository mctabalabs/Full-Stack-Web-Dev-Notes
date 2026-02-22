# Promises API

## Learning Objectives
By the end of this lesson, you will be able to:
- Understand the Promise lifecycle its three states: `pending`, `fulfilled`, and `rejected`.
- Create new Promises using the `Promise` constructor.
- Chain asynchronous operations using `.then()`, handle errors centrally with `.catch()`, and perform cleanup tasks with `.finally()`.
- Leverage powerful Promise combinators (`Promise.all()`, `Promise.race()`, `Promise.allSettled()`) to coordinate multiple asynchronous tasks.

---

## 1. Why Promises?
Before Promises existed, JavaScript developers relied heavily on **callbacks** for asynchronous operations. Passing callbacks into other callbacks often led to deeply nested code (infamously known as "Callback Hell" or the "Pyramid of Doom") and made error handling incredibly tedious.

Promises were introduced to solve this. A Promise provides:
- A much cleaner, chainable syntax for writing sequential async steps.
- Centralized error handling via a single `.catch()` block.
- Composability through static methods that allow you to run multiple async tasks in parallel or sequence easily.

---

## 2. Promise States & Anatomy
A **Promise** is a JavaScript object that represents the eventual completion (or failure) of an asynchronous operation, and its resulting value.

At any given time, a Promise lives in exactly one of three states:
1. **Pending**: The initial state. The operation has not completed yet; it is neither fulfilled nor rejected.
2. **Fulfilled** *(Resolved)*: The operation completed successfully, and a value is now available.
3. **Rejected**: The operation failed, and a reason (an Error object) is available.

### Creating a Promise
You can build a Promise using the `new Promise()` constructor. It takes an executor function with two arguments: `resolve` and `reject`.

```javascript
const p = new Promise((resolve, reject) => {
  // This executor function runs immediately (synchronously)
  const isSuccessful = true;
  
  // We simulate an async task finishing:
  if (isSuccessful) {
    resolve('Data loaded successfully!'); // Triggers the 'Fulfilled' state
  } else {
    reject(new Error('Failed to load data')); // Triggers the 'Rejected' state
  }
});

console.log(p); // Output: Promise { <fulfilled>: 'Data loaded successfully!' }
```

> **Note:** The executor function itself runs *synchronously* the moment the Promise is created. However, the `resolve()` and `reject()` functions schedule the state changes and subsequent `.then()` blocks *asynchronously*.

---

## 3. Consuming a Promise
Once a promise settles (fulfills or rejects), we need to do something with the result. We "consume" promises using three main instance methods.

### 3.1 `.then(onFulfilled, onRejected)`
The `.then()` block executes when the Promise fulfills.
- It returns a brand *new* Promise, which is what enables chaining multiple `.then()` blocks together.
- If you don't provide an `onRejected` handler (the second argument), errors will automatically propagate down the chain until they find a `.catch()`.

```javascript
p
  .then((value) => {
    console.log('Success:', value);
    return value.length; // Returns passed to the next .then!
  })
  .then((length) => {
    console.log('Length of string:', length);
  });
```

### 3.2 `.catch(onRejected)`
The `.catch()` block is your safety net. It catches rejections (errors) anywhere in the `.then()` chain above it.
*(Under the hood, `.catch(fn)` is just syntactic sugar for `.then(undefined, fn)`).*

```javascript
p
  .then((value) => value.toUpperCase())
  .then((upper) => console.log(upper))
  .catch((err) => {
    // This will catch an error if the Promise rejected, 
    // OR if an error was thrown inside any of the .then() blocks!
    console.error('Error caught:', err.message);
  });
```

### 3.3 `.finally(onFinally)`
The `.finally()` block runs regardless of whether the Promise was fulfilled or rejected. It is the perfect place for cleanup code (like hiding a loading spinner or closing a database connection).

```javascript
p
  .then(() => console.log('All good!'))
  .catch(() => console.log('There was an error!'))
  .finally(() => console.log('Cleanup: Hiding loading spinner.'));
```

> **Pro Tip:** The `.finally()` callback takes NO arguments. It does not receive the resolved value or the rejection reason. It simply allows the original outcome to pass right through it.

---

## 4. Chaining & Returning in Promises
A defining feature of `.then()` is that it always returns a Promise. What occurs in the *next* block depends on what you return from the current block:

- **Returning a regular value:** Wraps the value in a resolved Promise and passes it to the next `.then()`.
- **Returning nothing:** Equivalent to returning `undefined`. The next `.then()` receives `undefined`.
- **Returning a Promise:** The next `.then()` will *wait* for this new Promise to settle before executing.

```javascript
// A helper function that returns a Promise which resolves after a delay
function delay(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

delay(500)
  .then(() => {
    console.log('500ms passed');
    return delay(300); // We return a Promise, so the chain pauses here!
  })
  .then(() => {
    console.log('Another 300ms passed');
  });
```

---

## 5. Promise Combinators
Sometimes you need to coordinate multiple Promises at once. JavaScript provides static methods on the `Promise` class to handle arrays of Promises.

### 5.1 `Promise.all(iterable)`
Waits for **all** Promises in the array to fulfill. If even **one** Promise rejects, the entire `Promise.all` immediately rejects.

```javascript
Promise.all([
  fetch('/api/users').then(r => r.json()),
  fetch('/api/posts').then(r => r.json())
])
  // Destructuring the array of results
  .then(([users, posts]) => console.log('Loaded both:', users, posts))
  .catch(err => console.error('One or more requests failed:', err));
```

### 5.2 `Promise.race(iterable)`
Settles as soon as **any** Promise in the array settles (whether it fulfills OR rejects). It's a literal race.

```javascript
Promise.race([
  delay(1000).then(() => 'Turtle'),
  delay(200).then(() => 'Rabbit')
])
  .then(winner => console.log('Winner:', winner)); // Outputs: "Rabbit"
```

### 5.3 `Promise.allSettled(iterable)`
Waits for **all** Promises to settle, regardless of success or failure. It *never* rejects based on an inner Promise failing. Instead, it fulfills with an array of outcome objects.

```javascript
Promise.allSettled([
  delay(100).then(() => 'A (Success)'),
  Promise.reject(new Error('B failed'))
])
  .then(results => {
    console.log(results);
    /* Output:
      [
        { status: 'fulfilled', value: 'A (Success)' },
        { status: 'rejected',  reason: Error: 'B failed' }
      ]
    */
  });
```

---

## 6. In-Class Activity: Converting Callbacks to Promises

Let's convert a messy, nested callback structure into a clean Promise chain.

**The Legacy Code (Callbacks):**
Imagine we have three legacy functions that use callbacks:
```javascript
function step1(cb) { /* calls cb(null, 'one') after a delay */ }
function step2(cb) { /* calls cb(null, 'two') after a delay */ }
function step3(cb) { /* calls cb(null, 'three') after a delay */ }
```

**Task 1: The Wrapper**
Wrap each function in a new function that returns a Promise.

```javascript
function step1Promise() {
  return new Promise((resolve, reject) => {
    step1((err, res) => {
      // If error exists, reject. Otherwise, resolve with result.
      if (err) return reject(err);
      resolve(res);
    });
  });
}
// Assume we do the same for step2Promise() and step3Promise()
```

**Task 2: The Chain**
Now, chain them sequentially. Compare this to deeply nested callback hell!

```javascript
step1Promise()
  .then((res1) => {
    console.log(res1);
    return step2Promise();
  })
  .then((res2) => {
    console.log(res2);
    return step3Promise();
  })
  .then((res3) => console.log(res3))
  .catch((err) => console.error('Error in chain:', err));
```

> **Check Point:** If the order in which these steps run didn't matter, how could we refactor this to run them all at the same time? *(Refactor using `Promise.all()`!)*

---

## 7. Assignment: Polyfill `Promise.allSettled`

**Goal:** Write a function `allSettledPoly(promises)` that recreates the behavior of `Promise.allSettled`.

**Requirements:**
1. Do NOT use the built-in `Promise.allSettled`.
2. Accept an array of Promises (or regular values).
3. Return a new Promise that fulfills with an array of outcome objects:
   - Success format: `{ status: 'fulfilled', value: ... }`
   - Failure format: `{ status: 'rejected', reason: ... }`
4. **Hint:** You can use `Promise.all` combined with `.then` and `.catch` inside a `.map()` to achieve this! By catching errors on individual promises, you transform rejections into fulfilled values representing the error state.

**Test Case:**
```javascript
const p1 = Promise.resolve(1);
const p2 = Promise.reject(new Error('fail'));
const p3 = Promise.resolve(3);

allSettledPoly([p1, p2, p3]).then(results => console.log(results));

/* Expected Output:
[
  { status: 'fulfilled', value: 1 },
  { status: 'rejected',  reason: Error('fail') },
  { status: 'fulfilled', value: 3 }
]
*/
```

### Stretch Goal: Polyfill `Promise.any`
Implement a `Promise.any` polyfill. 
*(Hint: `Promise.any` resolves with the value of the FIRST fulfilled promise. It only rejects if ALL promises reject, in which case it rejects with an `AggregateError`.)*

---
*By mastering Promises, you’ll write significantly more maintainable asynchronous code, and be perfectly prepared for the next lesson: `async/await` & Error Handling!*