# Function Declarations & Expressions

## Learning Objectives
By the end of this lesson, you will be able to:
- Define functions using both declarations and function expressions.
- Explain what hoisting is and how it affects each form differently.
- Distinguish between named and anonymous functions, and understand when each is appropriate.
- Use the `arguments` object to handle a variable number of inputs.
- Choose the right function form for a given situation.

---

## 1. Why Functions Matter

Functions are the most fundamental building block in JavaScript. They let you:
- **Package logic once and reuse it** anywhere in your code.
- **Abstract complexity** behind a clear, readable name.
- **Break large programs** into small, focused units that are easier to read and test.

Without functions, every program would be a single block of instructions repeated wherever needed. Functions are what make code maintainable.

JavaScript has several ways to create a function. This lesson covers the two classic forms: **declarations** and **expressions**. Understanding both - and their differences - is essential before moving to arrow functions and higher-order patterns.

---

## 2. Function Declarations

A **function declaration** creates a named function using the `function` keyword as the first word of the statement.

### Syntax

```javascript
function functionName(parameter1, parameter2) {
  // body
  return result;
}
```

### Example

```javascript
function greet(name) {
  return `Hello, ${name}!`;
}

console.log(greet('Alice')); // "Hello, Alice!"
```

Breaking this down:
- `function` is the keyword that starts the declaration.
- `greet` is the function's name - a permanent, accessible identifier.
- `name` is a parameter - a placeholder for the value passed in when the function is called.
- `return` sends a value back to wherever the function was called.

---

## 3. Function Expressions

A **function expression** creates a function and assigns it to a variable. The function itself has no name attached to the statement - that name belongs to the variable.

### Syntax

```javascript
const functionName = function(parameter1, parameter2) {
  // body
  return result;
};
```

### Example

```javascript
const add = function(a, b) {
  return a + b;
};

console.log(add(2, 3)); // 5
```

Here, the function has no name on the right-hand side - it is an **anonymous** function. The label `add` belongs to the variable, not the function itself.

### Named Function Expressions

You can optionally give the function a name even within an expression. This name is only visible inside the function body:

```javascript
const multiply = function multiplyFn(x, y) {
  return x * y;
};

console.log(multiply(4, 5)); // 20
// multiplyFn is NOT accessible here - only inside the function body
```

Named function expressions are useful for:
- **Recursion**: The function can refer to itself by its internal name.
- **Debugging**: The name shows up in error stack traces instead of `anonymous`.

---

## 4. Hoisting - The Critical Difference

**Hoisting** is one of the most important concepts to understand when comparing declarations and expressions. It explains a behaviour that surprises almost every beginner.

### What Hoisting Means

When JavaScript reads your file before running it, it does a preparation pass called **compilation**. During this pass, function declarations are moved - conceptually - to the top of their scope. This means the function exists and can be called even before the line where you wrote it.

Function expressions behave differently. The variable is hoisted, but only as an empty placeholder. The function value is not assigned until that line of code actually runs.

### Side-by-Side Comparison

```javascript
// --- FUNCTION DECLARATION ---

// Call BEFORE definition: this works!
console.log(shout('hello')); // "HELLO"

function shout(str) {
  return str.toUpperCase();
}
```

```javascript
// --- FUNCTION EXPRESSION ---

// Call BEFORE definition: this throws an error!
console.log(double(5)); // TypeError: double is not a function

const double = function(n) {
  return n * 2;
};
```

### Why Does This Happen?

During the compilation pass:

| What JavaScript sees | Declaration | Expression |
|---|---|---|
| Is the name registered? | Yes | Yes (the variable name) |
| Is the function body available? | Yes, immediately | No - only after that line runs |
| Can you call it before the definition? | Yes | No |

For declarations, JavaScript reads the full function and makes it immediately available. For expressions, JavaScript creates the variable (`double`) but leaves it `undefined` until the assignment line is reached at runtime. Calling `undefined` as a function throws a `TypeError`.

### A Practical Analogy

Think of a function declaration like an announcement sent to the whole company before a meeting: "Sarah will handle this task." Everyone knows about Sarah before the meeting starts.

A function expression is like quietly hiring a contractor on the day you need them. You cannot refer to them for jobs that need to be done before they arrive.

### When Hoisting Matters in Practice

Most developers choose a consistent convention: either always declare functions before using them, or use declarations at the top of the file. But understanding hoisting matters because:
- You will encounter code written by others that calls a function "before" it appears.
- Bugs caused by accidentally calling an expression before assignment produce a confusing `TypeError` that is easier to understand if you know about hoisting.

---

## 5. Differences at a Glance

| Aspect | Function Declaration | Function Expression |
|---|---|---|
| Syntax position | Starts with `function` keyword | Assigned to a variable |
| Hoisted | Yes - callable before the line it is written | No - only usable after assignment |
| Named | Always named | Anonymous by default; can be optionally named |
| Recursion | Uses its own name | Requires a named function expression for self-reference |
| Common use | Core utility functions, top-level logic | Callbacks, passing functions to other functions, dynamic assignment |

---

## 6. Functions Are Values

This is one of the most important ideas in JavaScript: **functions are values**. A function expression makes this explicit - the function is just another value assigned to a variable, like a number or a string.

Because functions are values:
- You can store them in a variable: `const fn = function() {};`
- You can pass them as arguments: `arr.map(fn)`
- You can return them from other functions: `return function() {};`
- You can store them in arrays or objects.

This property - that functions are first-class values - is the foundation for callbacks, closures, and higher-order functions, which you will encounter in upcoming lessons.

---

## 7. The `arguments` Object

Inside any non-arrow function, JavaScript provides a special object called `arguments`. It is an array-like collection of every value passed to the function when it was called - even if you did not define any parameters.

```javascript
function inspect() {
  console.log(arguments);
  console.log('First argument:', arguments[0]);
  console.log('Number of arguments:', arguments.length);
}

inspect('hello', 42, true);
// Output:
// { '0': 'hello', '1': 42, '2': true }
// First argument: hello
// Number of arguments: 3
```

### When Is This Useful?

The `arguments` object allows a function to handle a variable number of inputs without declaring every parameter upfront. This was the standard approach before ES6.

### The Modern Alternative: Rest Parameters

In modern JavaScript (ES6 and later), the **rest parameter syntax** (`...args`) is preferred. It gives you a real `Array` instead of an array-like object, which means you get access to standard array methods like `.map()`, `.filter()`, and `.reduce()`:

```javascript
function sum(...numbers) {
  return numbers.reduce((total, n) => total + n, 0);
}

console.log(sum(1, 2, 3, 4)); // 10
```

Use `arguments` if you need to understand older code. Use rest parameters for new code.

---

## 8. In-Class Activity: Calculator, Two Ways

Implement a simple calculator function in both styles and test the hoisting behaviour of each.

### Declaration Version

```javascript
function calculate(a, operator, b) {
  switch (operator) {
    case '+': return a + b;
    case '-': return a - b;
    case '*': return a * b;
    case '/':
      if (b === 0) return 'Cannot divide by zero';
      return a / b;
    default:
      return 'Unknown operator';
  }
}
```

### Expression Version

```javascript
const calculate = function(a, operator, b) {
  switch (operator) {
    case '+': return a + b;
    case '-': return a - b;
    case '*': return a * b;
    case '/':
      if (b === 0) return 'Cannot divide by zero';
      return a / b;
    default:
      return 'Unknown operator';
  }
};
```

### Test Both

```javascript
console.log(calculate(10, '+', 5));    // 15
console.log(calculate(8, '*', 7));     // 56
console.log(calculate(10, '/', 0));    // Cannot divide by zero
```

### Demonstrate Hoisting

Try calling the **declaration version** above the line where you wrote the function. It should work. Then try calling the **expression version** before its assignment line. Observe the `TypeError` it throws. Compare the two and note the difference in the error message.

---

## 9. Common Beginner Mistakes

| Mistake | Explanation |
|---|---|
| Calling a function expression before its line | The variable exists but holds `undefined` until the assignment runs. This causes a `TypeError`. |
| Expecting `arguments` to work inside an arrow function | Arrow functions do not have their own `arguments` object. Use rest parameters instead. |
| Forgetting that the named expression's name is internal only | `const multiply = function multiplyFn() {}` - `multiplyFn` is not accessible outside the function. |
| Confusing the variable name with the function name | In `const add = function sum() {}`, the external name is `add`, not `sum`. |

---

## 10. Assignment: Factorial Function

Build a factorial calculator in two different styles.

**Factorial** is defined as the product of all positive integers up to `n`. For example:
- `5! = 5 * 4 * 3 * 2 * 1 = 120`
- `0! = 1` (by mathematical convention)
- `1! = 1`

### Part 1: Iterative Version (using a loop)

```javascript
function factorialIter(n) {
  // Use a for or while loop to multiply down from n to 1
}
```

### Part 2: Recursive Version (using a named function expression)

```javascript
const factorialRec = function rec(n) {
  // Base case: if n is 0 or 1, return 1
  // Recursive case: return n * rec(n - 1)
};
```

### Requirements

- Both functions must handle `n = 0` and `n = 1` by returning `1`.
- Both functions must handle negative inputs by returning `null` or throwing an error with a clear message.
- Both functions must produce the same results for valid inputs.

### Tests to Run

```javascript
console.log(factorialIter(5));  // 120
console.log(factorialRec(5));   // 120
console.log(factorialIter(0));  // 1
console.log(factorialRec(0));   // 1
console.log(factorialIter(-1)); // null or error
console.log(factorialRec(-1));  // null or error
```

### Stretch Goals

1. **Stack overflow test**: Try running `factorialRec(10000)`. What happens? Why does the recursive version break at large inputs but the iterative version does not?
2. **IIFE**: Wrap one of your functions in an Immediately Invoked Function Expression so it runs once without creating a global variable:
   ```javascript
   (function() {
     console.log(factorialIter(6)); // 720
   })();
   ```

---

## Summary

| Concept | Key Takeaway |
|---|---|
| Function declaration | Named, hoisted, callable before its line |
| Function expression | Assigned to a variable, not hoisted, only callable after assignment |
| Named function expression | Useful for recursion and readable stack traces |
| Hoisting | Declarations are fully available from the start of their scope; expressions are not |
| `arguments` object | Available in non-arrow functions; prefer rest parameters in new code |
| Functions as values | Functions can be passed, returned, and stored like any other value |

With function declarations and expressions solid, you are ready for arrow functions, default and rest parameters, and scope in the next lesson.