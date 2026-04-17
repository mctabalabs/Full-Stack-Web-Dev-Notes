# Understand Common HTML Tags: Building Real Web Pages

## Learning Objectives
By the end of this chapter, you will be able to:
- Create internal and external hyperlinks using the `<a>` tag.
- Embed images correctly using paths and accessibility best practices.
- Organize data using Unordered, Ordered, and Nested Lists.
- Build structured data displays using HTML Tables.
- Design functional user input forms using a variety of input types.

---

## 1. Hyperlinks: The "Web" in World Wide Web
Hyperlinks are what connect the internet. Without the `<a>` (Anchor) tag, we’d just have isolated documents.

### Basic Syntax
```html
<a href="https://google.com">Search the Web</a>
```
-   **`href`**: Short for "Hypertext Reference." This is the destination URL.
-   **Anchor Text**: The clickable text between the tags.

### External vs. Internal Links
1.  **External Links**: Point to websites outside your own. 
    > [!TIP]
    > Use `target="_blank"` to open links in a new tab. For security, also add `rel="noopener"` when doing this.
    ```html
    <a href="https://github.com" target="_blank" rel="noopener">Visit GitHub</a>
    ```
2.  **Internal Links**: Point to pages within your own project.
    ```html
    <a href="about.html">Learn About Us</a>
    ```
3.  **Section Anchors**: Link to specific parts of the *same* page using an `id`.
    ```html
    <a href="#contact">Jump to Contact Section</a>
    ```

---

## 2. Images and Media
The `<img>` tag allows you to display visual content. It is a **void tag**, meaning it doesn't have a closing tag.

### The Anatomy of an Image Tag
```html
<img src="images/logo.png" alt="Company Logo" width="300" height="200">
```
-   **`src`**: The path to your image file.
-   **`alt`**: **The most important attribute.** It provides a description for screen readers and displays if the image fails to load.
    > [!IMPORTANT]
    > Never leave the `alt` attribute out! If the image is purely decorative, use `alt=""`.

### Paths: Absolute vs. Relative
-   **Absolute Path**: A full URL (e.g., `https://example.com/pic.jpg`). Use this for images hosted elsewhere.
-   **Relative Path**: A path relative to your current file (e.g., `images/me.jpg`). Always use this for your local project files.

---

## 3. Lists: Organizing Information
Lists are essential for navigation menus, feature lists, and step-by-step guides.

### Unordered Lists (`<ul>`) - Bulleted
Used when the order of items doesn't matter (e.g., a grocery list).
```html
<ul>
  <li>Eggs</li>
  <li>Milk</li>
  <li>Bread</li>
</ul>
```

### Ordered Lists (`<ol>`) - Numbered
Used when order is crucial (e.g., a recipe or tutorial).
```html
<ol>
  <li>Crack the eggs.</li>
  <li>Whisk until smooth.</li>
  <li>Pour into the pan.</li>
</ol>
```

### Nested Lists
You can put a list inside a list item (`<li>`) to create sub-menus.
```html
<ul>
  <li>Fruits
    <ul>
      <li>Apples</li>
      <li>Bananas</li>
    </ul>
  </li>
  <li>Vegetables</li>
</ul>
```

---

## 4. Tables: Handling Structured Data
Tables should be used for **tabulated data** (like a schedule or a leaderboard), not for layout.

### Modern Table Structure
```html
<table>
  <caption>Weekly Sales Report</caption>
  <thead>
    <tr>
      <th>Product</th>
      <th>Price</th>
      <th>Stock</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Laptop</td>
      <td>$999</td>
      <td>15</td>
    </tr>
    <tr>
      <td>Mouse</td>
      <td>$25</td>
      <td>120</td>
    </tr>
  </tbody>
  <tfoot>
    <tr>
      <td colspan="2">Total Stock</td>
      <td>135</td>
    </tr>
  </tfoot>
</table>
```
-   **`<thead>`, `<tbody>`, `<tfoot>`**: Help browsers and screen readers understand the structure.
-   **`<th>`**: Table Header (bold and centered by default).
-   **`<td>`**: Table Data (regular cell).
-   **`colspan`**: Allows a cell to stretch across multiple columns.

---

## 5. Forms: Engaging the User
Forms allow users to send data to a server. They are the backbone of search bars, logins, and checkouts.

### The `<form>` Wrapper
```html
<form action="/submit-form" method="POST">
  <!-- Inputs go here -->
</form>
```
-   **`action`**: Where the data goes (URL).
-   **`method`**: How it gets there (`GET` for searching, `POST` for sensitive data like passwords).

### Essential Input Types
-   **Text**: `<input type="text" placeholder="Full Name">`
-   **Email**: `<input type="email" required>` (Built-in validation!)
-   **Password**: `<input type="password">` (Hides characters)
-   **Radio Buttons**: Choose **one** from a group. (Must share the same `name`).
    ```html
    <input type="radio" name="plan" id="free"> <label for="free">Free</label>
    <input type="radio" name="plan" id="pro"> <label for="pro">Pro</label>
    ```
-   **Checkboxes**: Choose **multiple** options.
-   **Dropdowns**:
    ```html
    <select name="city">
      <option value="nairobi">Nairobi</option>
      <option value="lagos">Lagos</option>
    </select>
    ```

> [!IMPORTANT]
> Always use the `<label>` tag! Clicking a label focuses its associated input, which is great for mobile users and accessibility.

---

## Combined Mini Project: The "Project Submission" Page
**Objective**: Build a page that includes a header, an image, a list of requirements, and a submission form.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Project Submission</title>
</head>
<body>
    <h1>Phase 1 Project Submission</h1>
    <img src="https://via.placeholder.com/600x200" alt="Tech Banner">
    
    <h2>Before you submit:</h2>
    <ul>
        <li>Ensure your code is commented.</li>
        <li>Include a <code>README.md</code> file.</li>
        <li>Check that all images have <code>alt</code> text.</li>
    </ul>

    <hr>

    <form>
        <fieldset>
            <legend>Submission Details</legend>
            <label for="name">Name:</label>
            <input type="text" id="name" required><br><br>

            <label for="link">GitHub Repository Link:</label>
            <input type="url" id="link" placeholder="https://github.com/..." required><br><br>

            <label>Difficulty Rating:</label><br>
            <input type="radio" name="diff" id="easy"> <label for="easy">Easy</label>
            <input type="radio" name="diff" id="hard"> <label for="hard">Hard</label>
        </fieldset>
        
        <br>
        <button type="submit">Submit Project</button>
    </form>
</body>
</html>
```

---

## Master Quiz
1.  What is the difference between `target="_self"` and `target="_blank"`?
2.  Is `<img>` a container tag or a void tag?
3.  When should you use an `<ol>` instead of a `<ul>`?
4.  Why should you always use a `<label>` with an `<input>`?
5.  What attribute allows a table cell to span multiple columns?

---

## Deep Dive Resources
- [MDN: HTML Elements Reference](https://developer.mozilla.org/en-US/docs/Web/HTML/Element)
- [A11Y Project: Image Accessibility](https://www.a11yproject.com/posts/alt-text/)
- [CSS-Tricks: A Complete Guide to Tables](https://css-tricks.com/complete-guide-table-element/)



