# Closures & Scope Chains

## Learning Objectives
By the end of this lesson, you will be able to:
- Explain what scope is and identify the three types: global, function, and block.
- Describe how JavaScript searches for variables using the scope chain.
- Define a closure and explain why it forms.
- Use closures for data privacy, encapsulation, and factory functions.
- Avoid the classic closure-in-a-loop pitfall.

---

## 1. What Is Scope?

**Scope** is the set of variables a piece of code can access at any given moment. Not every variable is visible everywhere in a program. Scope is the set of rules that determines which variables are accessible from which locations.

There are three kinds of scope in JavaScript.

---

## 2. The Three Types of Scope

### Global Scope

A variable declared outside all functions and blocks belongs to the **global scope**. It is accessible from anywhere in the program.

```javascript
const appName = 'MyApp'; // global scope

function showName() {
  console.log(appName); // accessible - it is in the global scope
}

showName(); // "MyApp"
```

### Function Scope

Variables declared inside a function using `let`, `const`, or `var` belong to that function's scope. They are not accessible from outside it.

```javascript
function calculate() {
  const result = 42; // function scope - private to calculate()
  console.log(result); // 42
}

calculate();
console.log(result); // ReferenceError: result is not defined
```

The variable `result` lives inside `calculate`. Once the function finishes, nothing outside can reach it.

### Block Scope

Variables declared with `let` or `const` inside a block - any code inside `{}` curly braces (such as `if`, `for`, `while`) - are scoped to that block only. They disappear when the block ends.

```javascript
if (true) {
  const message = 'I am inside a block';
  console.log(message); // "I am inside a block"
}

console.log(message); // ReferenceError: message is not defined
```

Note: Variables declared with `var` do not follow block scope. `var` is function-scoped and leaks out of blocks, which is one of the reasons modern JavaScript strongly prefers `let` and `const`.

```javascript
if (true) {
  var leaked = 'I escape the block'; // var ignores block scope!
}

console.log(leaked); // "I escape the block" - no error, but this is usually unintended
```

---

## 3. The Scope Chain: How JavaScript Finds Variables

### The Nested Rooms Analogy

Imagine a building with nested rooms. Each function is a room inside another.

- You are standing inside the innermost room.
- If you need something, you first look around your own room.
- If it is not there, you open the door and look in the hallway (the outer function's scope).
- If it is not in the hallway, you go further out to the next corridor, and eventually to the entrance lobby (global scope).

**Crucially: the hallway cannot look into your room.** Outer scopes have no access to inner scopes. Visibility only flows outward, never inward.

### How the Search Works

When JavaScript encounters a variable name, it searches for it in the following order:

1. **The current scope** - is the variable declared here?
2. **The enclosing outer scope** - if not found, look one level up.
3. **Continue upward** - keep looking at each outer scope in turn.
4. **Global scope** - the final place to look.
5. **ReferenceError** - if the variable is not found anywhere in the chain.

```javascript
const level = 'global';

function outer() {
  const level = 'outer function';

  function inner() {
    const level = 'inner function';
    console.log(level); // "inner function" - found immediately in current scope
  }

  inner();
  console.log(level); // "outer function" - inner's scope is invisible from here
}

outer();
console.log(level); // "global" - outer's and inner's scopes are invisible from here
```

### Variable Shadowing

When a variable in an inner scope shares the same name as one in an outer scope, the inner one **shadows** the outer one within that scope. JavaScript finds the inner one first and stops searching.

```javascript
const colour = 'blue';

function paint() {
  const colour = 'red'; // shadows the global 'colour' inside this function
  console.log(colour);  // "red"
}

paint();
console.log(colour); // "blue" - the global is unchanged
```

---

## 4. What Is a Closure?

A **closure** is what happens when a function remembers the variables from the scope where it was created, even after that outer scope has finished executing.

This sounds abstract, so here is the key insight to carry:

> A function does not just remember its own code. It also carries a reference to all the variables that existed in the scope where it was **born**.

This "birth environment" stays attached to the function for as long as the function itself exists.

### A Step-by-Step Example

```javascript
function makeGreeter(name) {
  const greeting = 'Hello'; // 'greeting' belongs to makeGreeter's scope

  return function() {       // this inner function is returned to the caller
    console.log(`${greeting}, ${name}!`);
    // 'greeting' and 'name' are from the outer scope - closure keeps them alive
  };
}

const greetAlice = makeGreeter('Alice');
// makeGreeter has now finished executing
// Normally, its local variables would be gone

greetAlice(); // "Hello, Alice!"
// They are NOT gone - the returned function still holds a reference to them
```

What happened:
1. `makeGreeter('Alice')` ran and created two local variables: `greeting` and `name`.
2. `makeGreeter` returned the inner function.
3. `makeGreeter` finished executing. Normally this would discard its variables.
4. But the returned function still refers to `greeting` and `name`.
5. JavaScript keeps those variables alive as long as the returned function exists.
6. When `greetAlice()` is called later, it still has access to them.

That retained access is the closure. The function **remembers its birthplace**.

---

## 5. Data Privacy and Encapsulation

The most practical use of closures is **data privacy**: hiding a variable so that only specific functions can interact with it.

### Why This Matters

In plain JavaScript, variables in the global scope are accessible and modifiable by anything. This is dangerous in larger programs - code in one place can accidentally overwrite a value used elsewhere.

Closures let you create **truly private** variables. The variables live inside a function's scope and are completely unreachable from outside, except through the specific functions you intentionally expose.

### Counter Example: The Classic Demonstration

```javascript
function createCounter() {
  let count = 0; // private - nothing outside can touch this directly

  return {
    increment() {
      count++;
      return count;
    },
    decrement() {
      count--;
      return count;
    },
    getValue() {
      return count;
    }
  };
}

const counter = createCounter();

console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.decrement()); // 1
console.log(counter.getValue());  // 1

console.log(counter.count); // undefined - 'count' is private and hidden
```

Notice that `counter.count` is `undefined`. There is no way to read or change `count` directly from outside. The only way to interact with it is through `increment`, `decrement`, and `getValue` - the three methods you chose to expose.

This is **encapsulation**: bundling data and the functions that act on that data together, while hiding the internal details.

---

## 6. Factory Functions

A **factory function** is a function that creates and returns another function (or object), where each call produces a new, independent closure.

This pattern is powerful because each created function has its own private copy of the outer variables.

```javascript
function multiplyBy(factor) {
  // 'factor' is captured in a fresh closure each time
  return function(number) {
    return number * factor;
  };
}

const double = multiplyBy(2);
const triple = multiplyBy(3);
const tenTimes = multiplyBy(10);

console.log(double(5));   // 10
console.log(triple(5));   // 15
console.log(tenTimes(5)); // 50
```

`double`, `triple`, and `tenTimes` are completely independent. Each has its own enclosed `factor` value. Changing the behaviour of one has no effect on the others.

With arrow functions, the same pattern is written more concisely:

```javascript
const multiplyBy = factor => number => number * factor;

const double = multiplyBy(2);
console.log(double(7)); // 14
```

---

## 7. The `once` Utility: Closures for State Tracking

Closures can track internal state across calls. The `once` pattern is a good example: it creates a function that runs only on the first call and ignores all subsequent calls.

```javascript
function once(fn) {
  let hasRun = false;  // private state - tracks whether fn has been called
  let result;          // stores the return value from the first call

  return function(...args) {
    if (!hasRun) {
      hasRun = true;
      result = fn(...args);
    }
    return result;
  };
}

function sendWelcomeEmail() {
  console.log('Welcome email sent!');
  return 'done';
}

const sendOnce = once(sendWelcomeEmail);

sendOnce(); // logs "Welcome email sent!", returns 'done'
sendOnce(); // logs nothing, returns 'done'
sendOnce(); // logs nothing, returns 'done'
```

The `hasRun` and `result` variables are private to the closure. Nothing outside `sendOnce` can read or reset them. This guarantees the function runs exactly once.

---

## 8. Common Beginner Mistakes

### Mistake 1: Closures Inside Loops

This is the most famous closure bug. It trips up almost every beginner who encounters it.

**The broken version:**

```javascript
const functions = [];

for (var i = 0; i < 3; i++) {
  functions.push(function() {
    console.log(i);
  });
}

functions[0](); // 3 - unexpected!
functions[1](); // 3 - unexpected!
functions[2](); // 3 - unexpected!
```

You might expect this to log `0`, `1`, `2`. It logs `3` three times. Why?

Because `var i` is **function-scoped**, not block-scoped. All three functions close over the **same single `i` variable**. By the time the loop finishes, `i` is `3`. All three functions read that same final value when they are eventually called.

**Why it happens:** The functions do not capture the value of `i` at the moment they are created. They capture a **reference to the variable `i`**. When they are called later, they look up `i` and find its current value - which is `3` after the loop ends.

**Fix 1: Use `let` instead of `var`**

```javascript
const functions = [];

for (let i = 0; i < 3; i++) {
  // 'let' creates a new 'i' for each iteration - each function gets its own
  functions.push(function() {
    console.log(i);
  });
}

functions[0](); // 0
functions[1](); // 1
functions[2](); // 2
```

`let` creates a new binding of `i` for each loop iteration. Each function now closes over its own independent copy of `i`.

**Fix 2: Use an IIFE (older approach, seen in legacy code)**

```javascript
const functions = [];

for (var i = 0; i < 3; i++) {
  (function(capturedI) {
    functions.push(function() {
      console.log(capturedI);
    });
  })(i); // immediately call with the current value of i
}

functions[0](); // 0
functions[1](); // 1
functions[2](); // 2
```

The IIFE creates a new scope per iteration, freezing the current value of `i` into `capturedI`. The modern fix with `let` is simpler and preferred.

---

### Mistake 2: Expecting Closures to Capture Values, Not References

When a closure captures a primitive value like a number or string, that value is copied at the time of creation and cannot be changed externally. But when a closure captures an object or array, it captures a **reference** to that object - not a copy. Mutating the object after the closure is created affects what the closure sees.

```javascript
const user = { name: 'Alice' };

function greetUser() {
  console.log(`Hello, ${user.name}!`);
}

greetUser(); // "Hello, Alice!"
user.name = 'Bob'; // mutate the object
greetUser(); // "Hello, Bob!" - the closure sees the mutation
```

This is not always wrong, but it is important to be aware of. If you need a snapshot of an object at a specific moment, make a copy of it before capturing it.

---

### Mistake 3: Creating Unnecessary Closures in Performance-Sensitive Code

Every closure holds a reference to its outer scope's variables, keeping them in memory. If you create thousands of functions inside a loop that each close over large data, that data cannot be garbage collected even if you no longer need it yourself.

For the Bank Account assignment and similar real-world patterns, this is not a concern. But in situations involving very large datasets or many generated functions, prefer placing shared functions outside the closure when they do not need access to the private state.

---

## 9. Memory Considerations

Closures keep the variables they reference alive in memory for as long as the closure itself exists. This is by design - it is what makes closures work.

However, if a closure holds onto a large object that you no longer need, that object will not be garbage collected because the closure still has a reference to it.

```javascript
function processData(largeDataset) {
  const processed = doExpensiveWork(largeDataset);

  return function report() {
    console.log(processed.summary);
  };
}

let report = processData(largeDataset);
// 'processed' stays in memory as long as 'report' exists

// When done, release the reference so memory can be collected
report = null;
```

The practical rule: if you create a closure that holds onto resources and you know you are done with it, set the variable holding the returned function to `null`. This removes the last reference and allows the garbage collector to free the memory.

---

## 10. In-Class Activity

### Part A: `makeCounter`

Write a `makeCounter` function that:
- Returns a function.
- Each call to the returned function increments and returns an internal count.
- The count is private - not accessible from outside.

```javascript
const counter = makeCounter();
console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3
```

Create two separate counters and verify they are independent of each other:

```javascript
const counterA = makeCounter();
const counterB = makeCounter();

console.log(counterA()); // 1
console.log(counterA()); // 2
console.log(counterB()); // 1 - completely independent
```

### Part B: `once`

Re-implement the `once` function from Section 7 without looking at the notes.

The function should:
- Accept any function `fn` as its argument.
- Return a new function that calls `fn` only on the first invocation.
- Return the same result on every subsequent call, without running `fn` again.

---

## 11. Assignment: Bank Account Module

Build a `createBankAccount(initialBalance)` function that uses closures to maintain a **private** balance.

The function should return an object with three methods:
- `deposit(amount)` - adds to the balance and returns the new balance.
- `withdraw(amount)` - deducts from the balance and returns the new balance.
- `getBalance()` - returns the current balance without changing it.

### Requirements

- The `balance` variable must be completely private - accessing `account.balance` from outside should return `undefined`.
- `initialBalance` should default to `0` if not provided.
- `withdraw` must refuse transactions that would make the balance negative. Return an informative message, or throw an error.

### Example Usage

```javascript
const account = createBankAccount(100);

console.log(account.deposit(50));       // 150
console.log(account.withdraw(30));      // 120
console.log(account.getBalance());      // 120
console.log(account.balance);           // undefined - private!
console.log(account.withdraw(200));     // "Insufficient funds"
```

### Stretch Goal

Add a `transactionHistory()` method that returns a copy of an array of strings describing each transaction in order:

```javascript
account.transactionHistory();
// ["Deposited 50", "Withdrew 30", "Withdraw of 200 failed: insufficient funds"]
```

The history array must be private - it should not be directly accessible from outside.

---

## Summary

| Concept | Key Point |
|---|---|
| Global scope | Accessible everywhere in the program |
| Function scope | Accessible only inside the function where it is declared |
| Block scope | Accessible only inside the `{}` block where it is declared; only with `let`/`const` |
| Scope chain | JavaScript searches outward from inner to outer scope to find a variable |
| Visibility direction | Inner scopes can see outer scopes; outer scopes cannot see inner scopes |
| Closure | A function that retains access to variables from the scope where it was created |
| Data privacy | Using closures to hide variables so only chosen functions can access them |
| Factory function | A function that returns new functions with their own independent closures |
| Closure in a loop | Using `var` causes all closures to share one variable; use `let` to fix this |