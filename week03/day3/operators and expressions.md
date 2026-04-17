# Lesson 1.3: Operators & Expressions

## Learning Objectives
By the end of this lesson, you will be able to:
- Use arithmetic, comparison, logical, and ternary operators to manipulate data.
- Understand operator precedence (order of operations) and associativity.
- Distinguish between strict (`===`) and loose (`==`) equality.
- Identify truthy and falsy values and how they impact conditional logic.
- Build mathematical and decision-making expressions.

---

## 1. Arithmetic Operators
Arithmetic operators perform mathematical operations on numeric values (and sometimes strings).

| Operator | Meaning | Example | Result |
| :---: | --- | --- | --- |
| `+` | Addition | `2 + 3` | `5` |
| `-` | Subtraction | `5 - 2` | `3` |
| `*` | Multiplication | `4 * 3` | `12` |
| `/` | Division | `10 / 2` | `5` |
| `%` | Remainder (Modulo) | `7 % 3` | `1` |
| `**`| Exponentiation | `2 ** 3` | `8` |

### Code Example
```javascript
let a = 10;
let b = 3;

console.log(a + b);  // 13
console.log(a - b);  // 7
console.log(a * b);  // 30
console.log(a / b);  // 3.3333333333333335
console.log(a % b);  // 1 (Because 10 divided by 3 is 9, with a remainder of 1)
console.log(a ** b); // 1000 (10 to the power of 3)
```

> **Note on the `+` Operator:** 
> When used with strings, the `+` operator **concatenates** (joins) them together instead of performing math.
> ```javascript
> console.log('Hello, ' + 'World!'); // Output: "Hello, World!"
> console.log('Number: ' + 5);       // Output: "Number: 5" (The number is coerced into a string)
> ```

---

## 2. Comparison Operators
Comparison operators compare two values and always produce a **boolean** result (`true` or `false`).

| Operator | Meaning | Example | Result |
| :---: | --- | --- | --- |
| `==` | Loose equality | `5 == '5'` | `true` |
| `===`| Strict equality | `5 === '5'`| `false`|
| `!=` | Loose inequality | `5 != '5'` | `false`|
| `!==`| Strict inequality| `5 !== '5'`| `true` |
| `<` | Less than | `3 < 5` | `true` |
| `<=` | Less than or equal| `5 <= 5` | `true` |
| `>` | Greater than | `7 > 2` | `true` |
| `>=` | Greater than or equal| `7 >= 7` | `true` |

### Strict vs. Loose Equality (Crucial Concept!)
- **`===` (Strict Equality):** Checks both the **type** and the **value**. `5` (Number) and `'5'` (String) are different types, so they are strictly not equal.
- **`==` (Loose Equality):** Coerces (converts) the types to match before comparing. This can lead to very surprising and bug-prone behavior!

```javascript
console.log(0 == false);         // true (0 is coerced to a boolean)
console.log(0 === false);        // false (Number vs Boolean)

console.log(null == undefined);  // true
console.log(null === undefined); // false
```
> **Best Practice:** ALWAYS use `===` and `!==`. Pretend `==` and `!=` don't exist. This will save you from countless painful bugs!

---

## 3. Logical Operators
Logical operators are used to combine multiple conditions or invert boolean values.

| Operator | Name | Example | Result |
| :---: | --- | --- | --- |
| `&&` | Logical AND | `true && false` | `false` (Needs BOTH to be true) |
| `\|\|` | Logical OR | `true \|\| false` | `true` (Needs AT LEAST ONE to be true) |
| `!` | Logical NOT | `!true` | `false` (Flips the boolean) |

### Code Example
```javascript
let isLoggedIn = true;
let hasPaid = false;

console.log(isLoggedIn && hasPaid); // false (Both conditions aren't met)
console.log(isLoggedIn || hasPaid); // true (At least one condition is met)
console.log(!hasPaid);              // true (Flipped false to true)
```

### Short-Circuiting
JavaScript is lazy (in a good way) when evaluating logical operators:
- For `&&`, if the left side evaluates to `false`, the entire expression *must* be false. JS completely ignores the right side.
- For `||`, if the left side evaluates to `true`, the entire expression *must* be true. JS completely ignores the right side.

```javascript
function expensiveOperation() {
  console.log('Running expensive calculation...');
  return true;
}

false && expensiveOperation(); // This instantly resolves to false. expensiveOperation() NEVER runs!
true || expensiveOperation();  // This instantly resolves to true. expensiveOperation() NEVER runs!
```

---

## 4. Ternary (Conditional) Operator
The ternary operator is a compact way to write simple `if/else` statements directly inside an expression.

**Syntax:** `condition ? expressionIfTrue : expressionIfFalse`

```javascript
let age = 18;
let status = age >= 18 ? 'Adult' : 'Minor';

console.log(status); // Output: "Adult"
```

> **Pro Tip:** Ternaries are fantastic for simple variable assignments. However, avoid chaining multiple ternaries together, as it makes your code notoriously difficult to read.

---

## 5. Operator Precedence & Associativity
What happens when you mix operators? Just like in math (PEMDAS/BODMAS), JavaScript has a defined order of operations.

- **Precedence:** Which operator goes first. 
  Order: `**` > `*`, `/`, `%` > `+`, `-` > Comparison (`<`, `>`, `===`) > Logical (`&&`) > Logical (`||`)
- **Associativity:** The direction expressions are evaluated when operators have the same precedence. Most are evaluated left-to-right, but exponentiation (`**`) evaluates right-to-left.

```javascript
console.log(2 + 3 * 4);   // 14 (Multiplication happens before addition)

console.log((2 + 3) * 4); // 20 (Parentheses override precedence!)

console.log(2 ** 3 ** 2); // 512 (Evaluated right-to-left: 3**2=9, then 2**9=512)
```
> **Rule of Thumb:** When in doubt about precedence, just use `()` parentheses to explicitly define your intended order. It makes your code easier to read for humans, too!

---

## 6. Truthy & Falsy Values
In JavaScript, a value doesn’t have to be strictly `true` or `false` to be evaluated in a condition (like an `if` statement or `&&`). When evaluated in a boolean context, JavaScript coerces values into "truthy" or "falsy".

### The 6 Falsy Values
If you memorize these six values, you know all of them. These evaluate to `false`:
1. `false`
2. `0`
3. `''`, `""`, ``` `` ``` (Empty strings)
4. `null`
5. `undefined`
6. `NaN` (Not a Number)

### Truthy Values
**Everything else** evaluates to `true`! This includes:
- Non-empty strings (`'hello'`, `'false'`, `' '`)
- Non-zero numbers (`42`, `-1`, `3.14`)
- Objects and Arrays (even empty ones: `{}` and `[]` are truthy!)

```javascript
if ('') console.log('This will NOT run because empty strings are falsy');
if (0)  console.log('This will NOT run because 0 is falsy');

if (42) console.log('This WILL run because 42 is truthy');
if ([]) console.log('This WILL run because empty arrays are truthy');
```

> **Common Pitfall:**
> Be very careful with `0`. Since `0` is falsy, code like this can accidentally fail:
> ```javascript
> const itemCount = 0;
> if (!itemCount) {
>   console.log('You have no items in your cart!'); 
>   // This runs, because 0 coerces to false, and the ! flips it to true.
> }
> ```

---

## 7. In-Class Activity: FizzBuzz Variations

**Task:** Iterate through numbers 1 to 30.
- If the number is divisible by 3 → print "Fizz"
- If the number is divisible by 5 → print "Buzz"
- If the number is divisible by both 3 and 5 (i.e., 15) → print "FizzBuzz"
- Otherwise → print the number itself.

**The Twist:** Write this using ternary operators inside a `console.log`! Let's see how compact we can make our logic.

```javascript
for (let i = 1; i <= 30; i++) {
  // Using chained ternaries!
  const output = 
    i % 15 === 0 ? 'FizzBuzz' : 
    i % 3 === 0  ? 'Fizz' : 
    i % 5 === 0  ? 'Buzz' : 
    i;
    
  console.log(output);
}
```

---

## 8. Assignment: Build a Calculator Function

**Goal:** Build a single function that handles multiple math operations and edge cases.

1. Open your `main.js` and structure a `calculate` function based on this boilerplate:

```javascript
/**
 * Performs basic arithmetic.
 * @param {number} a - The first number
 * @param {string} operator - One of "+", "-", "*", "/", "%"
 * @param {number} b - The second number
 * @returns {number | string} The math result or an error message
 */
function calculate(a, operator, b) {
  // Your code goes here!
}
```

### Requirements:
1. **Core Logic:** Support the 5 primary operators: `+`, `-`, `*`, `/`, `%`. If `operator` matches one of these, perform the operation and return the result.
2. **Invalid Operators:** If an unsupported operator (like `'^'`) is passed, return the string `"Invalid operator"`.
3. **Edge Case 1 - Divide by Zero:** If the user tries to divide by `0` (using `/` or `%`), immediately return the string `"Cannot divide by zero"`.
4. **Edge Case 2 - Type Coercion:** If the user passes a string instead of a number for `a` or `b` (e.g., `'8'`), use `Number()` to coerce it before doing the math.

### Test Cases
Once you build the function, paste these console logs at the bottom of your file to verify it works perfectly:

```javascript
console.log(calculate(10, '+', 5));    // Expected: 15
console.log(calculate(10, '/', 0));    // Expected: "Cannot divide by zero"
console.log(calculate(10, '%', 0));    // Expected: "Cannot divide by zero"
console.log(calculate(10, '^', 3));    // Expected: "Invalid operator"
console.log(calculate('8', '*', '7')); // Expected: 56
```
