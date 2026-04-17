# HTML Semantics and Page Structure

Mastering the structure of an HTML document is the first step toward building professional, accessible, and SEO-friendly websites. This guide covers how to visualize document trees, use semantic elements correctly, and understand why structure matters.

---

## Learning Objectives

By the end of this chapter, you will be able to:
- **Visualize** a web page as a nested tree structure (the DOM).
- **Identify** and use core semantic HTML elements (`<header>`, `<main>`, `<section>`, `<article>`, `<aside>`, `<footer>`).
- **Distinguish** between semantic elements and generic containers (`<div>` and `<span>`).
- **Explain** how semantic structure improves accessibility (A11y) and search engine optimization (SEO).
- **Construct** a well-structured, production-ready HTML layout.

---

## 1. The HTML Document Tree (The DOM)

Every HTML page is represented by the browser as a **nested tree structure**. This is known as the **Document Object Model (DOM)**.

The `<html>` element is the **root** of the tree. Everything else branches out from there.

### The Two Main Branches:
1.  **`<head>`**: The "brain" of the document. Contains metadata, character sets, page title, and links to external styles/scripts. Users don't see this content directly.
2.  **`<body>`**: The "body" of the document. Contains everything that is rendered on the screen (text, images, videos, etc.).

### Visualizing Relationships:
Elements in the tree have specific relationships:
- **Parent**: An element that contains other elements (e.g., `<body>` is the parent of `<header>`).
- **Child**: An element contained within another (e.g., `<h1>` is a child of `<header>`).
- **Sibling**: Elements that share the same parent (e.g., `<header>` and `<main>` are siblings).

**Example Tree Representation:**
```text
html
 └── body
      ├── header
      │     └── h1 (Welcome)
      ├── main
      │     ├── section
      │     │    └── p (Introduction text)
      │     └── section
      │          └── p (Content details)
      └── footer
            └── p (Copyright)
```

> [!TIP]
> **Pro Tip:** Right-click any web page, select **Inspect**, and look at the **Elements** tab in Chrome/Firefox DevTools to explore the live document tree!

---

## 2. Non-Semantic Containers: `<div>` and `<span>`

Sometimes you need to group elements for styling or layout purposes without adding specific meaning. For this, we use **generic containers**.

### `<div>` (Block-level)
- Used for grouping large sections of content.
- Creates a new line before and after the element.
- **Common use:** Creating layout wrappers or "boxes" for CSS grid/flexbox.

### `<span>` (Inline-level)
- Used for grouping small chunks of content *inside* a line (like a few words in a paragraph).
- Does not create a new line.
- **Common use:** Styling a specific word or phrase differently.

> [!WARNING]
> **Overuse Warning:** Avoid "Div-itis"—using `<div>` for everything. If a tag like `<header>` or `<nav>` fits, use it!

---

## 3. Semantic HTML: Adding Meaning

Semantic elements describe the **purpose** of the content they contain. This helps browsers, developers, and assistive technologies understand your page.

| Tag | Name | Purpose |
| :--- | :--- | :--- |
| `<header>` | Header | Introductory content or navigation links for a page or section. |
| `<nav>` | Navigation | A section containing major navigation links. |
| `<main>` | Main | The unique, central content of the document (only one per page). |
| `<section>` | Section | A thematic grouping of content, typically with a heading. |
| `<article>` | Article | Self-contained content that could be distributed independently (e.g., a blog post). |
| `<aside>` | Aside | Content tangentially related to the main content (e.g., sidebars, callouts). |
| `<footer>` | Footer | Closing information, contact info, or copyright at the bottom of a page/section. |

### Section vs. Article: Which one?
- **`<article>`**: If the content makes sense if you "ripped it out" and put it on another site (like a news story or blog post).
- **`<section>`**: If the content is just a logical chapter or part of a larger whole.

---

## Why Semantics Matter

1.  **SEO (Search Engine Optimization):** Search engine bots use semantic tags to identify what is important on your page, helping you rank higher.
2.  **Accessibility (A11y):** Screen readers use these tags to allow vision-impaired users to skip directly to the `<main>` content or navigate via `<nav>`.
3.  **Maintainability:** Clean, semantic code is significantly easier for other developers (and your future self) to read and debug.
4.  **Readability Mode:** Many browsers use semantic tags to generate "Reader View" versions of your articles.

---

## Mini Project: Standard Page Layout

Below is a template for a professional semantic layout. Notice how each piece of content has a logical "home."

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Modern Portfolio Layout</title>
</head>
<body>

  <!-- Site branding and navigation -->
  <header>
    <h1>Jane Developer</h1>
    <nav>
      <ul>
        <li><a href="#about">About</a></li>
        <li><a href="#projects">Projects</a></li>
        <li><a href="#contact">Contact</a></li>
      </ul>
    </nav>
  </header>

  <!-- The heart of the page -->
  <main>
    <section id="about">
      <h2>About Me</h2>
      <p>Full-stack developer passionate about semantic HTML and clean code.</p>
    </section>

    <section id="projects">
      <h2>Featured Projects</h2>
      
      <article>
        <h3>Weather App</h3>
        <p>A real-time weather tracker built with JavaScript.</p>
      </article>

      <article>
        <h3>Task Master</h3>
        <p>A productivity tool focused on focus and simplicity.</p>
      </article>
    </section>
  </main>

  <!-- Extra info or sidebar -->
  <aside>
    <h3>Latest Tweets</h3>
    <p>"Just learned about the DOM tree! #WebDev #CoreHTML"</p>
  </aside>

  <!-- Closing info -->
  <footer>
    <p>&copy; 2026 Jane Developer. Built with Semantic HTML.</p>
  </footer>

</body>
</html>
```

---

## Recap Quiz

1.  What is the difference between `<main>` and `<div>`?
2.  Which tag would you use for a sidebar containing "Related Posts"?
3.  Why should you avoid using `<div>` for everything?
4.  What is the "root" element of an HTML document?

<details>
<summary>Click to see Answers</summary>

1.  `<main>` provides semantic meaning (tells the browser it's the core content), while `<div>` is a generic container with no meaning.
2.  `<aside>`.
3.  It makes the page harder to navigate for screen readers and less optimized for search engines.
4.  The `<html>` element.
</details>

---

## Extra Resources

- [MDN Web Docs: HTML Elements Reference](https://developer.mozilla.org/en-US/docs/Web/HTML/Element)
- [A11y Project: Accessibility Basics](https://www.a11yproject.com/)