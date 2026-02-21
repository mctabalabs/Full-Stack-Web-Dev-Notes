# Lesson 1.2: Variables & Data Types

## Learning Objectives
By the end of this lesson, you will be able to:
- Declare and use variables safely with `let` and `const`.
- Understand the differences (and pitfalls) of `var`.
- Identify and work with JavaScript’s primitive types: string, number, boolean, null, undefined, (and briefly Symbol and BigInt).
- Leverage dynamic typing and the `typeof` operator.
- Format string output cleanly using template literals.

---

## 1. Variables: Containers for Values
JavaScript variables are named "boxes" that store data (values). As your program runs, you can read from or change what's inside a box. How you create (declare) that box changes its behavior, specifically regarding **scoping**, **mutability** (can it change?), and **safety**.

### 1.1 `var` vs. `let` vs. `const`

| Keyword | Scope | Reassignable? | Hoisting Behavior | Notes |
|---------|-------|---------------|-------------------|-------|
| `var` | Function or global | Yes | Hoisted to top as `undefined`; no Temporal Dead Zone (TDZ) | **Legacy.** Avoid using this in modern JavaScript. |
| `let` | Block (`{ ... }`) | Yes | Hoisted but in TDZ | Good for variables that will change (e.g., loop counters). |
| `const` | Block (`{ ... }`) | No | Hoisted but in TDZ; must be initialized right away | **Best Practice.** Default choice for most variables. |

> **Key Takeaway & Best Practice:**
> - Default to `const` for values you don’t intend to reassign. This prevents accidental changes and bugs!
> - Use `let` only when you know the value will change (e.g., a score counter in a game, or inside a loop).
> - **Avoid `var`**. Its function scope and hoisting behavior often lead to confusing, subtle bugs.

#### Example: The confusion of `var` hoisting
```javascript
// var example: Unintuitive hoisting
console.log(x); // Outputs: undefined (Not an error!)
var x = 5;

// let/const example: Temporal Dead Zone (TDZ)
console.log(y); // ReferenceError: Cannot access 'y' before initialization
let y = 10;
```
*Why does this happen?* With `let` and `const`, JavaScript knows the variable exists but refuses to let you use it before the line where it is defined. This period is called the **Temporal Dead Zone (TDZ)** and helps you catch errors early.

---

## 2. Primitive Data Types
JavaScript has seven primitive (basic) data types. The first five are the most common in everyday coding:

1. **String**: Text data.
2. **Number**: All numeric values (integers, decimals).
3. **Boolean**: True or false.
4. **null**: Intentional absence of any value.
5. **undefined**: A variable declared but not yet assigned a value.
6. **Symbol**: Unique identifiers (introduced in ES6).
7. **BigInt**: For arbitrarily large integers.

### 2.1 Strings
A string is a sequence of characters used to represent text. In JavaScript, you can wrap strings in single quotes (`'...'`), double quotes (`"..."`), or backticks (`` `...` ``).

```javascript
let single = 'Hello';
let double = "World";
let backtick = `Hello, ${single}!`; // This is a template literal (more on this later!)
```
> **Common Pitfall:** Don't mix quotes. If you start a string with a single quote, you must end it with a single quote. If you need a single quote *inside* the string, use double quotes outside: `"It's a beautiful day"`.

### 2.2 Numbers
Unlike some languages that have different types for integers and decimals, JavaScript uses a single `Number` type for all numbers (they are IEEE-754 double-precision floats under the hood). This includes special numeric values like `NaN` (Not a Number) and `Infinity`.

```javascript
let age = 25;
let temp = 98.6;
```
> **Floating-Point Quirk:** Because of how decimals are handled in memory, `0.1 + 0.2` does not exactly equal `0.3`.
```javascript
const precisionIssue = 0.1 + 0.2; 
console.log(precisionIssue); // Output: 0.30000000000000004
```

### 2.3 Booleans
Booleans have exactly two possible values: `true` or `false`. They are essential for decision-making in code (e.g., `if` statements).

```javascript
let isRaining = true;
let isAdult = age >= 18; // Resolves to true or false
```

### 2.4 `null` vs. `undefined`
Both represent "nothing", but they serve different purposes.
- **`undefined`**: The default state for a variable that has been declared but not assigned a value. It means "value not yet known."
- **`null`**: An intentional, explicit empty value assigned by the developer. It means "I meant for this to be empty."

```javascript
let a;
console.log(a); // Output: undefined

let b = null;
console.log(b); // Output: null
```

### 2.5 Dynamic Typing
JavaScript is a **dynamically typed** language. This means a single variable can hold a string, and later be reassigned to hold a number or a boolean without causing an error.

```javascript
let foo = 'bar'; // currently a string
foo = 42;        // now a number
foo = true;      // now a boolean
```
> **Pro Tip:** While dynamic typing is powerful, it can lead to confusing code. As a best practice, try to keep a variable's purpose consistent. If `score` starts as a number, keep it a number!

---

## 3. The `typeof` Operator
You can use the `typeof` operator to inspect a value's data type while the program is running.

```javascript
console.log(typeof 'hello');      // "string"
console.log(typeof 123);          // "number"
console.log(typeof false);        // "boolean"
console.log(typeof undefined);    // "undefined"

// Special cases:
console.log(typeof null);         // "object"  <-- This is a historical bug in JavaScript!
console.log(typeof Symbol());     // "symbol"
console.log(typeof BigInt(10));   // "bigint"
```
> **Watch out:** Because `typeof null === 'object'`, always treat `null` as its own special case if you are checking types.

---

## 4. Template Literals vs. Concatenation

When you want to combine variables and words into a single sentence, you have two options.

### 4.1 String Concatenation (The Old Way)
You can use the `+` operator to join strings. Note how you have to manually manage spaces.

```javascript
let firstName = 'Jane';
let lastName = 'Doe';
console.log('Hello, ' + firstName + ' ' + lastName + '!');
// Output: "Hello, Jane Doe!"
```

### 4.2 Template Literals (Recommended)
Template literals use backticks (`` ` ``) instead of quotes. They let you embed variables directly inside the string using `${...}` syntax, resolving the spacing headaches. They also support multi-line strings effortlessly.

```javascript
let firstName = 'Jane';
let lastName = 'Doe';
console.log(`Hello, ${firstName} ${lastName}! 
Welcome to JS Foundations.`);

/* Output:
Hello, Jane Doe!
Welcome to JS Foundations.
*/
```

---

## 5. In-Class Activity: Greeting with Prompts

**Goal:** Prompt for user input and print a formatted greeting to the console.

1. Open your `main.js` file and add the following code:

```javascript
// prompt() asks the user for input in a browser popup
const userName = prompt('What is your name?');
const ageInput = prompt('How old are you?');

// Coerce (convert) the string input into a Number
const ageNum = Number(ageInput);

// Use a template literal to greet them
console.log(`Hi, ${userName}! You are ${ageNum} years old.`);
```

2. **Run in the browser:**
   - Open your `index.html` file in the browser.
   - Answer the prompts that pop up.
   - Open Developer Tools (`F12` or `Cmd+Option+I`) and look at the Console tab to see your greeting.

> **Experiment:**
> - What happens if you leave the age blank? (`Number('')` converts empty strings to `0`).
> - What happens if you type "twenty" instead of a number? (`Number('twenty')` results in `NaN`, printing "You are NaN years old.")
> - How do we fix this? We'll learn how to validate user input in future lessons!

---

## 6. Assignment: Profile Card

**Goal:** Build a "Profile Card" entirely in the developer console.

1. Create (or extend) your `main.js` with three variables, representing yourself or a fictional character:

```javascript
const profileName = 'Your Name';
const profileAge = 30; // Use a number
const isStudent = true; // Use a boolean
```

2. Log a neatly formatted profile summary using template literals:

```javascript
console.log('Profile Card');
console.log(`Name      : ${profileName}`);
console.log(`Age       : ${profileAge}`);

// This uses a ternary operator (condition ? trueResult : falseResult)
// to print "Yes" or "No" instead of "true" or "false"
console.log(`Student?  : ${isStudent ? 'Yes' : 'No'}`);
```

### Stretch Goals:
- Add a `favoriteColors` array and list them in your output (e.g., `['Red', 'Blue', 'Green'].join(', ')`).
- Change your `profileAge` variable to use `let` instead of `const`. Simulate a birthday by adding 1 to the age (`profileAge = profileAge + 1`) and re-run your `console.log` statements to see the updated age. 

*Happy coding!*
