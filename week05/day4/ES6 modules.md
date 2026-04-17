# ES6 Modules (Import/Export)

## Objectives
- Understand the motivation for modules: encapsulation, reuse, and dependency management.
- Learn named vs. default exports.
- Use `import` statements (static and dynamic) with correct paths.
- Configure the environment for modules (browser & Node).
- Handle common pitfalls: circular dependencies, file extensions, and bundler basics.

---

## 1. Why Modules?
Before ES6, all script files shared a single global scope, leading to naming collisions and maintenance headaches. Modules introduce:
- **Encapsulation**: Each file has its own scope.
- **Explicit Dependencies**: You declare exactly what you use.
- **Tree Shaking**: Bundlers can eliminate unused code.
- **Maintainability**: Clear boundaries and single-responsibility files.

---

## 2. Enabling Modules in the Browser

### 2.1 `<script type="module">`
```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>ES6 Modules Demo</title>
</head>
<body>
  <script type="module" src="main.js"></script>
</body>
</html>
```

**Key Behaviors:**
- **Strict Mode**: Automatic in modules.
- **Deferred Loading**: Module scripts are deferred by default (they wait for HTML parsing).
- **CORS**: Modules follow Cross-Origin Resource Sharing (CORS) rules.

---

## 3. Named Exports
Export multiple values by name from a module.

```javascript
// mathUtils.js
export function add(a, b) {
  return a + b;
}

export const PI = 3.14159;

function subtract(a, b) {
  return a - b;
}
export { subtract }; // alternative syntax
```

### 3.1 Importing Named Exports
```javascript
// main.js
import { add, PI, subtract } from './mathUtils.js';

console.log(add(2, 3));       // 5
console.log(subtract(5, 2));  // 3
```

### 3.2 Aliasing
```javascript
import { add as sum, subtract as diff } from './mathUtils.js';
console.log(sum(4,1), diff(4,1)); // 5, 3
```

---

## 4. Default Exports
A module can specify one default export—often the primary value or class.

```javascript
// logger.js
export default function log(message) {
  console.log(`[LOG]: ${message}`);
}
```

### 4.1 Importing Defaults
```javascript
// main.js
import log from './logger.js';
log('Application started');
```

### 4.2 Mixing Named & Default
```javascript
// config.js
export const ENV = 'development';
export default { debug: true };

// consumer.js
import config, { ENV } from './config.js';
console.log(ENV, config.debug);
```

---

## 5. Aggregating & Re-exporting
Create a **barrel file** to simplify imports.

```javascript
// services/index.js
export { default as authService } from './auth.js';
export { default as userService } from './users.js';
```

Then elsewhere:
```javascript
import { authService, userService } from './services/index.js';
```

---

## 6. Dynamic `import()`
Load modules on demand—returns a Promise.

```javascript
// main.js
document.getElementById('loadBtn').addEventListener('click', async () => {
  const { add } = await import('./mathUtils.js');
  console.log(add(10, 20));
});
```
**Use Cases**: Code-splitting, feature flags, lazy-loading large libraries.

---

## 7. Modules in Node.js
By default, Node treats `.js` files as CommonJS. To use ES modules:
1. Rename files to `.mjs`.
2. **OR** Add `"type": "module"` in `package.json`.

```json
// package.json
{
  "name": "esm-demo",
  "version": "1.0.0",
  "type": "module"
}
```

---

## Deep Dive: Advanced Concepts

### Live Bindings
Unlike CommonJS (which copies values), ES Modules use **Live Bindings**. This means the imported value is a reference to the original variable in the exporting module.
- If the exporting module updates the value, the importing module sees the change instantly.
- Imported values are **read-only**; you cannot reassign them in the importing module.

### Top-Level `await`
Inside a module, you can use the `await` keyword at the top level without wrapping it in an `async` function.
```javascript
// db.js
const connection = await db.connect();
export default connection;
```
Modules using top-level `await` will block the execution of modules that import them until the promise is resolved.

### Module Scoping
Each module has its own top-level scope. Variables defined at the top level of a module are **not** global and cannot be accessed from other scripts unless explicitly exported.

### Performance & Static Analysis
ESModules are **statically analyzable**. Bundlers like Vite or Webpack can determine exactly which functions are used before the code even runs. This enables **Tree Shaking** (removing unused code), resulting in smaller bundle sizes.

---

## 8. Common Pitfalls
- **Missing Extensions**: Browsers require full paths: `import { x } from './mod.js'` (not `./mod`).
- **Circular Dependencies**: If Module A imports B and B imports A, you might get `undefined` values. Refactor to a shared dependency.
- **CORS Issues**: Loading local module files via `file://` protocol fails; use a local server.

---

## 9. In-Class Activity

### Refactor Calculator UI into Modules
1. Split your logic into `calculate.js`, `display.js`, and `ui.js`.
2. Use `import`/`export` syntax to connect them.
3. Update `index.html` to load only `ui.js` via `<script type="module">`.

---

## 10. Assignment

### NPM-Style Utility Library
1. Create a folder `utils-lib/`.
2. Implement `sumAll.js` (named export) and `average.js` (default export).
3. Create an `index.js` as a barrel file.
4. Initialize with `npm init -y` and `"type": "module"`.
5. Import into a separate project and test.

---

*With ES6 modules in your toolkit, you’ll structure larger applications with clear, maintainable, and performant code.*