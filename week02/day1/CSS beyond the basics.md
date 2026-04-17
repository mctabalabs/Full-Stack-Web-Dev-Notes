# CSS Foundations: Beyond the Basics

## 1. The CSS Cascade, Specificity, and Inheritance

### 1.1 The Cascade
CSS stands for **Cascading Style Sheets**. The "Cascade" is the algorithm that decides which style to apply when multiple rules target the same element. It processes rules in this order of priority:
1.  **Origin & Importance**: `!important` declarations > Author styles (your CSS) > User styles > Browser defaults.
2.  **Specificity**: More targeted rules beat generic ones.
3.  **Order of Appearance**: If everything else is equal, the last rule in the code wins.

### 1.2 Specificity Calculation
Specificity is calculated as three numbers: `(ID, Class, Type)`.

| Selector | Examples | Specificity |
| :--- | :--- | :--- |
| **ID** | `#header`, `#nav` | `1, 0, 0` |
| **Class / Attribute / Pseudo-class** | `.btn`, `[type="text"]`, `:hover` | `0, 1, 0` |
| **Element / Pseudo-element** | `p`, `div`, `::before` | `0, 0, 1` |
| **Universal / :where()** | `*`, `:where(.foo)` | `0, 0, 0` |

**Example**:
- `div.main-content` -> Specificity: `0, 1, 1`
- `#main-header` -> Specificity: `1, 0, 0` (Wins over the above)

### 1.3 Inheritance
Some properties (like `color` and `font-family`) are passed down from parents to children. Others (like `border` and `margin`) are not.
- Use `inherit` to force a child to take a parent's value.
- Use `initial` to reset to the browser default.

---

## 2. CSS Values and Units

### 2.1 Absolute Units
- **px (Pixels)**: Fixed size. Good for borders and tiny details, but not recommended for layout or typography on modern web pages because it doesn't scale with user settings.

### 2.2 Relative Units (Recommended)
- **rem**: Relative to the root (`<html>`) font-size. **Best for typography.** (If 1rem = 16px, then 2rem = 32px).
- **em**: Relative to the parent's font-size. Good for padding/margins that should scale with the text in that specific element.
- **%**: Relative to the parent element's size.
- **vw / vh**: Viewport Width / Viewport Height (1vw = 1% of the screen width). Great for full-screen sections.

---

## 3. Layout Modes & Normal Flow

### 3.1 Normal Flow
By default, elements are either **Block** (stack vertically, take 100% width) or **Inline** (sit side-by-side, width is based on content).

### 3.2 Display Types
- `display: block`: Starts on a new line.
- `display: inline`: Stays in the same line; width/height are ignored.
- `display: inline-block`: Sits inline but obeys width/height.
- `display: flex`: One-dimensional layout (see below).
- `display: grid`: Two-dimensional layout (see below).

---

## 4. Positioning
The `position` property determines where an element sits relative to the document or its ancestors.

- **static**: Default flow.
- **relative**: Offset relative to itself. It stays in the "flow," meaning its original space is preserved.
- **absolute**: Removed from flow. Positioned relative to its nearest **positioned** (non-static) ancestor.
- **fixed**: Relative to the browser window. Stays there when you scroll.
- **sticky**: Static until a scroll point, then acts like fixed. Use `top: 0` to trigger.

**Pro Tip: `z-index`**
`z-index` only works on non-static elements. It controls the stacking order (which element covers which).

---

## 5. Flexbox: One-Dimensional Layout
Flexbox is designed for laying out items in a single row or column.

### Core Properties
- `flex-direction`: `row` (default) or `column`.
- `justify-content`: Aligns items on the **Main Axis** (`center`, `space-between`, `space-around`).
- `align-items`: Aligns items on the **Cross Axis** (`center`, `stretch`, `flex-start`).
- `flex-grow`: Allows an item to fill the remaining space.

**Common Pattern: Centering a Div**
```css
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}
```

---

## 6. CSS Grid: Two-Dimensional Layout
Grid layout allows you to handle both columns and rows simultaneously.

### The Power of `fr` Units
The `fr` (fractional) unit represents a fraction of the available space in the grid container.
```css
.grid {
  display: grid;
  grid-template-columns: 2fr 1fr; /* First column is twice as wide */
  gap: 20px;
}
```

### The "Auto-Magic" Responsive Grid
No media queries needed for this one:
```css
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 1rem;
}
```

---

## 7. Responsive Design & Media Queries
Responsive design ensures your site looks good on phones, tablets, and desktops.

### Mobile-First Approach
Start with styles for small screens, then expand using media queries.
```css
/* Default Mobile Styles */
body { font-size: 14px; }

/* Desktop Adjustment */
@media screen and (min-width: 768px) {
  body { font-size: 18px; }
}
```

---

## 8. CSS Variables (Custom Properties)
Variables allow you to store values and reuse them. They are reactive—change the variable once, and every element using it updates instantly.

```css
:root {
  --primary-color: #6c5ce7;
  --spacing: 20px;
}

.button {
  background-color: var(--primary-color);
  padding: var(--spacing);
}
```

---

## 9. Animations & Transitions

### 9.1 Transitions
Smoothly change a property over time (usually on hover).
```css
.btn {
  background: blue;
  transition: background 0.3s ease-in-out;
}
.btn:hover {
  background: purple;
}
```

### 9.2 Animations
Define complex motions using `@keyframes`.
```css
@keyframes slide-in {
  from { transform: translateX(-100%); opacity: 0; }
  to { transform: translateX(0); opacity: 1; }
}

.box {
  animation: slide-in 0.5s forwards;
}
```

---

## Further Reading
1. [MDN: The CSS Cascade](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Cascade/Introduction)
2. [A Complete Guide to Flexbox (CSS-Tricks)](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
3. [A Complete Guide to Grid (CSS-Tricks)](https://css-tricks.com/snippets/css/complete-guide-grid/)
4. [Modern CSS Units (MDN)](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Styling_basics/Values_and_units)