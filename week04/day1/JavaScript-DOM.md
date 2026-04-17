# JavaScript DOM and DOM Manipulation

## Introduction: What is the DOM?
The Document Object Model (DOM) is a programming interface that browsers use to represent web documents in memory. When a browser loads an HTML page, it builds a logical tree in which each branch ends in a node and each node contains objects. The DOM connects web pages to scripts (usually JavaScript) by allowing programs to access and change the document’s structure, style and content.

Although the DOM is often accessed via JavaScript, it is not part of the JavaScript language itself; it is a **Web API** implemented by browsers.

A web page can be displayed as HTML source or in the browser window, but both views refer to the same document. The DOM provides an object-oriented representation of this document so it can be modified with a scripting language.

The DOM is language-independent: although JavaScript is the common language used on the web, any programming language could theoretically use the DOM API.

The root of the DOM tree for an HTML document is the `<html>` node. Child nodes represent elements like `<head>`, `<body>`, `<p>`, text nodes, comments and even whitespace. Understanding this tree structure is critical for DOM manipulation.

---

## Browser Environment: Important Objects
Before manipulating the DOM, it helps to understand key browser objects:

| Object | Description |
| :--- | :--- |
| `window` | Represents the browser window. It provides global scope for JavaScript variables and functions and acts as the default context for browser APIs (e.g., `alert()`, `setTimeout()`). |
| `document` | Represents the DOM of the current web page. You use the `document` object to access and manipulate nodes in the DOM tree. |
| `navigator` | Represents information about the browser (name, version, user-agent string). Useful for feature detection. |

---

## Node Relationships and Terminology
The DOM tree uses standard tree-based terminology:

*   **Root node:** The top of the tree (`<html>` in an HTML document).
*   **Parent node:** A node that has other nodes inside it. For example, `<body>` is the parent of `<section>`.
*   **Child node:** A node directly inside another node. For example, `<img>` is a child of `<section>`.
*   **Sibling nodes:** Nodes on the same level under the same parent.
*   **Descendant node:** Any node inside another node, regardless of depth (e.g., `<img>` is a descendant of `<body>`).

---

## Selecting Elements

### Modern Selection Methods
The recommended way to obtain references to elements is using CSS-selector–based methods:

*   `document.querySelector(selector)`: Returns the **first** element in the document matching the CSS selector.
    ```javascript
    const link = document.querySelector('a');
    ```
*   `document.querySelectorAll(selector)`: Returns a **NodeList** of all elements matching the selector. You can iterate through it using `forEach`.
    ```javascript
    const paragraphs = document.querySelectorAll('p');
    paragraphs.forEach(p => console.log(p.textContent));
    ```

### Older Selection Methods
For compatibility with older browsers, you may encounter these methods:

*   `document.getElementById(id)`: Returns the element with the specified `id`.
*   `document.getElementsByTagName(tag)`: Returns an array-like object (HTMLCollection) of elements with the given tag name.
*   `document.getElementsByClassName(className)`: Returns elements matching the class name.

---

## Reading and Modifying Content
Once you have a reference to an element, you can read or change its content:

*   `textContent`: Gets/sets the text content of a node. It replaces all child nodes with a single text node.
    ```javascript
    const link = document.querySelector('a');
    link.textContent = 'New link text';
    ```
*   `innerHTML`: Gets/sets the HTML inside an element. 
    > [!WARNING]
    > Be careful—setting `innerHTML` parses and replaces all children, which may have security implications (XSS). Use `textContent` for plain text.

*   **Attributes:** Use `getAttribute(name)` and `setAttribute(name, value)`.
    ```javascript
    const img = document.querySelector('img');
    const alt = img.getAttribute('alt');
    img.setAttribute('src', 'new-image.jpg');
    ```

*   **Properties:** Many attributes have property equivalents (e.g., `element.href`).

---

## Working with Classes and Styles

### The classList API
The `classList` property provides convenient methods to manipulate CSS classes:

| Method | Description |
| :--- | :--- |
| `element.classList.add('class')` | Adds a class to the element. |
| `element.classList.remove('class')` | Removes a class. |
| `element.classList.toggle('class')` | Adds the class if not present; removes it if present. |
| `element.classList.contains('class')` | Checks whether a class is present. |

**Example: Click Toggle**
```javascript
const button = document.querySelector('button');
button.addEventListener('click', () => {
    button.classList.toggle('active');
});
```

### Manipulating Styles
You can directly change inline styles via the `style` property.

```javascript
const paragraphs = document.getElementsByTagName('p');
const secondParagraph = paragraphs[1];
secondParagraph.style.backgroundColor = 'red';
```
> [!TIP]
> Use CSS classes whenever possible; direct style manipulation should be reserved for dynamic effects.

---

## Creating, Inserting, and Removing Nodes

### Creating Elements
Use `document.createElement(tagName)` to create a new element node.

```javascript
const newPara = document.createElement('p');
newPara.textContent = 'This paragraph was created dynamically!';
document.body.appendChild(newPara);
```

### Creating Text Nodes
```javascript
const textNode = document.createTextNode('Hello world');
newPara.appendChild(textNode);
```
*Note: Setting `textContent` is usually preferred as it implicitly creates text nodes.*

### Inserting and Appending
*   `appendChild(node)`: Appends the node as the last child.
*   `insertBefore(newNode, referenceNode)`: Inserts `newNode` before `referenceNode`.
*   `prepend(node)`: Inserts as the first child.

### Removing Elements
*   `removeChild(node)`: Removes node from its parent (still exists in memory).
*   `remove()`: Modern method that removes the element directly (`element.remove()`).

---

## Events and Event Handling

### What Is an Event?
An event is a signal fired by the browser when something significant happens (clicks, key presses, page load).

### Adding Event Listeners
Use `addEventListener(eventType, handler)` to register event handlers.

**Example: Random Color Button**

#### HTML
```html
<button id="colorBtn">Random Color</button>
```

#### JavaScript
```javascript
const button = document.getElementById('colorBtn');

function randomColor() {
    const r = Math.floor(Math.random() * 256);
    const g = Math.floor(Math.random() * 256);
    const b = Math.floor(Math.random() * 256);
    document.body.style.backgroundColor = `rgb(${r}, ${g}, ${b})`;
}

button.addEventListener('click', randomColor);
```

### Event Object
When an event fires, the browser passes an **event object** to the handler containing information like the target element (`e.target`) or mouse coordinates.
Use `e.preventDefault()` to stop default behaviors (like form submission).

---

## Putting It All Together: Shopping List Example

This example demonstrates selecting, creating, styling, and handling events to build a dynamic list.

### HTML
```html
<input id="itemInput" placeholder="Enter item" />
<button id="addBtn">Add</button>
<ul id="list"></ul>
```

### CSS
```css
ul { list-style-type: none; padding: 0; }
li { margin: 0.5em 0; }
.remove { margin-left: 0.5em; color: red; cursor: pointer; }
```

### JavaScript
```javascript
const input = document.getElementById('itemInput');
const addBtn = document.getElementById('addBtn');
const list = document.getElementById('list');

function addItem() {
    const value = input.value.trim();
    if (!value) return; 

    // Create list item and remove button
    const li = document.createElement('li');
    li.textContent = value;

    const removeBtn = document.createElement('span');
    removeBtn.textContent = '×';
    removeBtn.classList.add('remove');

    // Attach click handler to remove button
    removeBtn.addEventListener('click', () => {
        list.removeChild(li);
    });

    li.appendChild(removeBtn);
    list.appendChild(li);
    input.value = '';
}

addBtn.addEventListener('click', addItem);

// Also add item when pressing Enter
input.addEventListener('keypress', (e) => {
    if (e.key === 'Enter') addItem();
});
```

---

## Best Practices
1.  **Cache element references:** Store references in variables to avoid repeated DOM queries.
2.  **Minimize reflows:** Use `DocumentFragment` when building large subtrees.
3.  **Separate concerns:** Keep structure (HTML), style (CSS), and behavior (JS) distinct. Avoid inline handlers like `onclick=""`.

---

## References
*   [MDN: Document Object Model (DOM)](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)
*   [Learn Web Development: DOM scripting](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Scripting/DOM_scripting)
*   [MDN: Introduction to events](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Scripting/Events)
