# Getting Started with HTML: The Foundation of the Web

## Learning Objectives
By the end of this chapter, you will be able to:
- Define HTML and explain its role in web development.
- Create and save a valid HTML5 document.
- Understand the purpose of the `<head>` and `<body>` sections.
- Master key tags for text, lists, and media.
- Use attributes to provide extra information to tags.
- Apply best practices for code readability (comments, indentation).

---

## 1. What is HTML?
**HTML** stands for **HyperText Markup Language**. 
- **HyperText**: Text that contains links to other text (the "web").
- **Markup Language**: A way to annotate text to tell the browser how to display it.

> [!IMPORTANT]
> HTML is **NOT** a programming language. It is a markup language used to define the **structure** and **content** of a web page. CSS handles the styling, and JavaScript handles the logic.

---

## 2. Anatomy of an HTML Document
Every HTML document follows a strict skeleton. The browser uses these tags to interpret your file correctly.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My First Web Page</title>
</head>
<body>
    <h1>Hello World!</h1>
    <!-- Page content goes here -->
</body>
</html>
```

### The Breakdown:
1.  **`<!DOCTYPE html>`**: Tells the browser "Hey, this is a modern HTML5 document!"
2.  **`<html lang="en">`**: The root element. The `lang` attribute helps search engines and screen readers know the language.
3.  **`<head>`**: The "brain" of the page. Contains **metadata** (data about the data) that isn't visible to users but is crucial for the browser and search engines.
    -   `<meta charset="UTF-8">`: Ensures special characters (like emojis or non-Latin letters) display correctly.
    -   `<meta name="viewport" ...>`: Ensures the page looks good on mobile devices (Responsive Design).
    -   `<title>`: The name that appears on the browser tab.
4.  **`<body>`**: The "body" of the page. Everything you see—text, images, buttons—goes here.

---

## 3. Essential Content Tags

### Headings
HTML provides six levels of headings, from `<h1>` (most important) to `<h6>` (least important).

```html
<h1>Main Title</h1>
<h2>Section Title</h2>
<h3>Subsection</h3>
```

> [!TIP]
> Use only **one** `<h1>` per page. It represents the main topic of the page and is critical for SEO (Search Engine Optimization).

### Paragraphs and Text Flow
- **`<p>`**: Defines a paragraph. It automatically adds some space (margin) above and below.
- **`<br>`**: Inserts a line break without starting a new paragraph.
- **`<hr>`**: Creates a horizontal line (thematic break) to separate sections.

```html
<p>This is a paragraph.<br>This same paragraph continues on a new line.</p>
<hr>
<p>This is a new section entirely.</p>
```

---

## 4. Attributes: Adding Extra Data
Attributes provide additional information about an element. They always go inside the **opening tag** and usually look like `name="value"`.

### Links (`<a>`)
The anchor tag needs an `href` (Hypertext Reference) attribute to know where to go.
```html
<a href="https://www.google.com" target="_blank">Visit Google</a>
```
- `target="_blank"`: Opens the link in a new tab.

### Images (`<img>`)
Images are "void tags" (they don't have a closing tag).
```html
<img src="logo.png" alt="Company Logo" width="200">
```
- `src`: The path to the image file.
- `alt`: Alternate text. **CRUCIAL** for accessibility (read by screen readers) and if the image fails to load.

---

## 5. Lists: Organizing Information
HTML has two main types of lists:

### Unordered Lists (Bulleted)
```html
<ul>
  <li>Html</li>
  <li>CSS</li>
  <li>JavaScript</li>
</ul>
```

### Ordered Lists (Numbered)
```html
<ol>
  <li>Open Browser</li>
  <li>Search for Course</li>
  <li>Start Learning</li>
</ol>
```

---

## 6. Formatting & Comments
- **`<strong>`**: Makes text bold (and indicates it's important).
- **`<em>`**: Makes text italic (adds emphasis).
- **Comments**: `<!-- This is hidden -->`. Use them to explain your code to yourself or other developers.

---

## Mini Project: Build a Simple Student Bio
**Objective**: Create a personal bio page using everything we've learned.

1.  Create a file named `bio.html`.
2.  Add the standard HTML5 boilerplate (`<!DOCTYPE html>`, `<html>`, etc.).
3.  Add an `<h1>` with your name.
4.  Add an `<img>` (use a placeholder if you don't have one: `https://via.placeholder.com/150`).
5.  Add a `<h2>` titled "About Me" followed by a paragraph.
6.  Add an `<h3>` titled "Skills" followed by an unordered list of 3 skills.
7.  Add a link to your favorite website.

---

## Recap Quiz
1.  Which tag is used for the largest heading?
2.  What does the `alt` attribute in an `<img>` tag do?
3.  True or False: Metadata in the `<head>` is visible to the user on the page.
4.  Which tag creates a numbered list?
5.  Why is indentation important in HTML?

---

## 🔗 Deep Dive Resources
- [MDN Web Docs: HTML Basics](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/HTML_basics)
- [W3Schools HTML Tutorial](https://www.w3schools.com/html/)
- [Companion Video: HTML Structure Explained](https://www.youtube.com/results?search_query=html+structure+for+beginners)