# Parameters, Rest & Default Values

## 🎯 Objectives
- Understand how to define **default parameter values** to guard against missing arguments.
- Use **rest parameters** (`...args`) to capture an arbitrary number of arguments in a real Array.
- Apply the **spread operator** (`...`) to expand arrays or objects into function calls or literals.
- Distinguish between **rest** (in definitions) and **spread** (in calls/literals).

---

## 1. Default Parameter Values

When a function is called with fewer arguments than parameters, missing parameters become `undefined`. Default values let you supply a fallback.

### Without Defaults
```javascript
function greet(name, greeting) {
  // If greeting is undefined, message will be "undefined, Alice!"
  console.log(`${greeting}, ${name}!`);
}

greet('Alice'); // "undefined, Alice!"
```

### With Defaults
```javascript
function greet(name = 'Guest', greeting = 'Hello') {
  console.log(`${greeting}, ${name}!`);
}

greet('Alice');         // "Hello, Alice!"
greet();                // "Hello, Guest!"
greet(undefined, 'Hi'); // "Hi, Guest!"
```

> [!NOTE]
> Defaults are used **only** when the argument is `undefined`. Passing `null` or other falsy values (e.g., `0` or `''`) will not trigger the default.

### Dynamic Defaults
You can use expressions or function calls as defaults:
```javascript
function rollDice(sides = 6) {
  return Math.floor(Math.random() * sides) + 1;
}
```

---

## 2. Rest Parameters: Capturing "The Rest"

Rest parameters gather all remaining arguments into a true `Array`. This replaces the old `arguments` object and lets you work with methods like `.map()`, `.filter()`, etc.

```javascript
// Sum any number of numeric arguments
function sumAll(...nums) {
  console.log(nums); // an actual Array
  return nums.reduce((total, n) => total + n, 0);
}

console.log(sumAll(1, 2, 3));        // 6
console.log(sumAll(10, 20, 30, 40)); // 100
console.log(sumAll());               // 0 (empty array → reduce with initial 0)
```

> [!IMPORTANT]
> - Rest must be the **last** parameter in the function signature.
> - You can have other named parameters before it.

```javascript
function joinWith(separator, ...items) {
  return items.join(separator);
}

console.log(joinWith('-', 'a', 'b', 'c')); // "a-b-c"
```

---

## 3. Spread Operator: Expanding Iterables

The spread operator looks like rest (`...`) but is used in **calls** or **literals** to expand arrays (or any iterable) into individual elements.

```javascript
const nums = [1, 2, 3];

// Expanding in function calls:
console.log(Math.max(...nums));      // 3

// Copying arrays:
const copy = [...nums];             
console.log(copy);                   // [1, 2, 3]

// Concatenating:
const more = [...nums, 4, 5];
console.log(more);                   // [1, 2, 3, 4, 5]
```

> [!TIP]
> - Spread also works on strings: `[...'hello']` → `['h', 'e', 'l', 'l', 'o']`.
> - You can spread objects (ES2018+):
>   ```javascript
>   const obj = { a: 1, b: 2 };
>   const clone = { ...obj, c: 3 }; // { a: 1, b: 2, c: 3 }
>   ```
> - Unlike `Object.assign`, object spread creates a **shallow clone** without mutating the original.

---

## 4. Putting It All Together

A common pattern is merging default options with overrides using destructuring and rest/spread.

```javascript
// Merge default options with overrides
function createUser(username, {
  role = 'guest',
  active = true,
  theme = 'light',
  ...extras        // collect any extra settings
} = {}) {          // default the entire config object
  return {
    username,
    role,
    active,
    theme,
    ...extras      // spread extras into the new object
  };
}

// Usage examples:
console.log(createUser('alice')); 
/* Result:
{
  username: 'alice',
  role: 'guest',
  active: true,
  theme: 'light'
}
*/

console.log(createUser('bob', { role: 'admin', theme: 'dark', age: 30 }));
/* Result:
{
  username: 'bob',
  role: 'admin',
  active: true,
  theme: 'dark',
  age: 30
}
*/
```

### Key Breakdown:
1. **Default Empty Object**: `{} = {}` prevents errors if no second argument is passed.
2. **Destructuring with Defaults**: Handles `role`, `active`, and `theme`.
3. **Rest Capturing**: `...extras` grabs any additional properties.
4. **Spread Merging**: Re-combines everything into the final object.

---

## 5. In-Class Activity: Parameter Utilities

### `mergeOptions(defaults, overrides)`
- Takes two objects.
- Returns a new object where properties from `overrides` replace those in `defaults`.
- **Hint**: Use object spread.

```javascript
const defaults = { a: 1, b: 2 };
const overrides = { b: 42, c: 3 };
// result: { a: 1, b: 42, c: 3 }
```

### `logAll(prefix, ...items)`
- Logs each item to the console, prefixed by the given string.
- **Example**: `logAll('ITEM:', 'apple', 'banana')` logs `ITEM: apple` and `ITEM: banana`.

---

## 6. Assignment: Flexible Calculator

Create a `flexCalc` function that:
1. Accepts an **operator string** (`'+'`, `'-'`, `'*'`, `'/'`) as its first parameter.
2. Uses **rest** to accept any number of numeric arguments.
3. Uses **defaults** to assume `0` for missing numbers (if fewer than two are provided).
4. Returns the cumulative result.

### Example Usage:
```javascript
flexCalc('+', 1, 2, 3, 4) // 10
flexCalc('*', 2, 3, 4)    // 24
flexCalc('-', 10)         // 10 - 0 = 10
flexCalc('/', 20)         // Handle division by zero
```

### Edge Cases to Handle:
- **No numbers passed**: Return `0` for `+`, `1` for `*`, or `null` for others.
- **Division by zero**: Return `'Error: division by zero'`.

> [!TIP]
> Implement this using a single function with rest parameters and a `reduce` or a loop.