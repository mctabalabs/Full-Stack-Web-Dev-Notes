# Lesson 1.4: Control Flow

## Learning Objectives
By the end of this lesson, you will be able to:
- Control the execution path of your code using `if/else`, `else if`, and `switch` statements.
- Repeat tasks efficiently using loops: `for`, `while`, `do...while`, and `for...of`.
- Manage and alter loop behavior with `break` and `continue`.
- Understand and avoid common infinite-loop pitfalls.
- Write nested loops and recognize when to use them.

---

## 1. Conditional Branching
Control flow is the order in which individual statements, instructions, or function calls are executed. Conditional branching allows your program to make decisions and execute different blocks of code based on certain conditions.

### 1.1 `if` / `else if` / `else`
This is the most fundamental way to choose between different code paths.

```javascript
const score = 75;

if (score >= 90) {
  console.log('Grade: A');
} else if (score >= 80) {
  console.log('Grade: B');
} else if (score >= 70) {
  console.log('Grade: C');
} else {
  console.log('Grade: D or below');
}
```

**How it works:**
- **`if`**: Evaluates its condition first. If it is truthy, its block of code runs, and the rest of the chain is skipped.
- **`else if`**: Checked *only* if all preceding `if` or `else if` conditions were falsy.
- **`else`**: The fallback block. It runs *only* when none of the above conditions matched.

> **Pro Tip:** Keep each condition simple. If you find yourself chaining 5 or 6 `else if` statements, it might be time to refactor your code into a lookup object or use a `switch` statement!

### 1.2 `switch` Statement
A cleaner alternative to many `else if` statements when you are comparing a single variable against many specific values (literals).

```javascript
const trafficLight = 'green';

switch (trafficLight) {
  case 'red':
    console.log('Stop');
    break;
  case 'yellow':
    console.log('Caution');
    break;
  case 'green':
    console.log('Go');
    break;
  default:
    console.log('Unknown signal. Treat as a 4-way stop.');
}
```

**Key Features of `switch`:**
- **Strict Equality (`===`):** The `case` labels are matched using strict equality.
- **The `break` Keyword:** Don't forget this! Without a `break`, execution "falls through" and automatically runs the code for the subsequent cases, even if they don't match.
- **`default`:** Acts exactly like the `else` in an `if/else` chain—it handles any unmatched values.

---

## 2. Loops
Loops let you run the exact same block of code repeatedly until a specific condition changes. They are essential for handling lists of data or repeating tasks.

### 2.1 The `for` Loop
The `for` loop is a general-purpose loop that uses an index counter. It is highly condensed, placing the setup, condition, and increment all on one line.

```javascript
for (let i = 0; i < 5; i++) {
  console.log(`Iteration number: ${i}`);
}
// Outputs: 0, 1, 2, 3, 4
```

**Anatomy of a `for` loop:**
1. **Initialization (`let i = 0`):** Runs exactly *once* before the loop starts.
2. **Condition (`i < 5`):** Checked *before* each iteration. If it's true, the loop body runs.
3. **Final-expression (`i++`):** Runs *after* each iteration finishes (usually increments or decrements the counter).

### 2.2 The `while` Loop
A `while` loop runs repeatedly as long as its condition remains truthy. It's best used when you *don't* know exactly how many times the loop needs to run in advance.

```javascript
let count = 3;

while (count > 0) {
  console.log(count);
  count--; // Decrement the count
}
// Outputs: 3, 2, 1
```

> **Common Pitfall: The Infinite Loop**  
> If you forget to update your counter (e.g., forgetting `count--`), the condition `count > 0` will *always* be true. Your code will run forever and crash the browser! Always ensure your loop variable moves toward the exit condition.

### 2.3 The `do...while` Loop
Similar to a `while` loop, but with one major difference: the condition is checked *after* the loop body runs. This guarantees the block executes **at least once**.

```javascript
let input;

do {
  // We want to ask the user at least once!
  input = prompt('Type "exit" to quit:');
} while (input !== 'exit');
```

### 2.4 The `for...of` Loop
Introduced in ES6, `for...of` is the cleanest way to iterate directly over iterable values like Arrays or Strings. You don't need to manually track an index.

```javascript
const fruits = ['🍎', '🍌', '🍇'];

for (const fruit of fruits) {
  console.log(`I love ${fruit}`);
}
// Outputs: I love 🍎, I love 🍌, I love 🍇
```

---

## 3. Controlling Flow: `break` & `continue`
Sometimes you need to interrupt a loop mid-execution.
- **`break`**: Immediately destroys and exits the innermost loop entirely.
- **`continue`**: Immediately skips the remainder of the *current* iteration, and jumps to the next iteration.

```javascript
for (let i = 1; i <= 5; i++) {
  if (i === 3) continue; // Skips the number 3
  if (i === 5) break;    // Stops the loop entirely before logging 5
  
  console.log(i);
}
// Outputs: 1, 2, 4
```

---

## 4. Nested Loops
Loops can live inside other loops. This is particularly useful for working with multi-dimensional data, like grids, matrices, or tables.

```javascript
for (let row = 1; row <= 3; row++) {
  for (let col = 1; col <= 3; col++) {
    console.log(`Coordinate: (${row}, ${col})`);
  }
}
/* Outputs:
Coordinate: (1, 1) Coordinate: (1, 2) Coordinate: (1, 3)
Coordinate: (2, 1) Coordinate: (2, 2) Coordinate: (2, 3) ... and so on
*/
```

> **Caution:** Deeply nested loops (a loop inside a loop inside a loop...) can become exponentially slow and extremely difficult to read. Try to keep nesting shallow, or refactor inner loops into separate functions.

---

## 5. In-Class Activity: Prime Number Checker

**Task:** Write a function `isPrime(n)` that returns `true` if a number is a prime number, and `false` otherwise. (A prime number is only divisible by 1 and itself).

**Logic:**
1. Numbers less than 2 are not prime.
2. Use a `for` loop to check numbers from `2` up to `n - 1`.
3. If `n` is cleanly divisible (`% === 0`) by *any* of those numbers, it is not prime. Return `false` immediately (which acts like a break).
4. If the loop finishes without finding any divisors, the number is prime. Return `true`.

```javascript
function isPrime(n) {
  if (n < 2) return false;
  
  for (let i = 2; i < n; i++) {
    if (n % i === 0) {
      return false; // Found a divisor, so it's not prime!
    }
  }
  
  return true; // No divisors found, it must be prime.
}

// Test cases
console.log(isPrime(2));  // true
console.log(isPrime(4));  // false
console.log(isPrime(17)); // true
```

---

## 6. Assignment: Loop Utilities

**Goal:** Practice loops and nested loops by building two utility functions. Add these to your `main.js` file and test them using `console.log`.

### Part 1: `flattenArray(arr)`
**Input:** An array that contains other arrays (e.g., `[[1, 2], [3, 4], [5]]`).
**Output:** A single, flat array (e.g., `[1, 2, 3, 4, 5]`).

**Hint:** Create an empty result array. Use a `for...of` loop to iterate through the outer array, and a nested `for...of` loop inside it to extract the inner numbers and `push()` them to your result array.

```javascript
// Test Case:
console.log(flattenArray([[1, 2], [3, 4], [5]])); // Expected: [1, 2, 3, 4, 5]
```

### Part 2: `sumRange(start, end)`
**Input:** Two integer parameters, `start` and `end` (inclusive).
**Output:** The sum of all integers between those two numbers.

**Example:** `sumRange(5, 10)` should calculate `5 + 6 + 7 + 8 + 9 + 10 = 45`.

```javascript
// Test Cases:
console.log(sumRange(5, 10)); // Expected: 45

// Stretch Goal Test Case:
console.log(sumRange(10, 5)); // What happens if start is greater than end? Fix your function to handle this gracefully!
```

### Stretch Goals:
- **`flattenArray`**: Can you handle edge cases, like empty internal arrays (`[[1], [], [2, 3]]`)?
- **`sumRange`**: Rather than breaking if `start` is greater than `end`, use an `if` statement at the top of your function to politely swap the values and continue calculating the sum! (Hint: you might need a temporary variable).
