# Weekend Challenge: The "Student Developer Hub"

**Goal:** Level up your initial HTML project into a professional-grade, multi-page portfolio. This isn't just a website; it's your digital identity as a developer.

---

> [!IMPORTANT]
> **Primary Rule:** This is an **HTML-only** challenge (for now!). No CSS or JS allowed. We are focusing on building a rock-solid structural foundation using **Semantic HTML**.

---

## Roadmap for the Weekend

Follow these phases to ensure a high-quality submission.

### Phase 1: The Setup (Architecture)
Create your folder structure first. Organization is a superpower.

```text
portfolio-v2/
  ├── index.html        (Home)
  ├── about.html        (Bio & Interests)
  ├── projects.html     (The Showcase)
  ├── project-1.html     (Deep Dive 1)
  ├── project-2.html     (Deep Dive 2)
  ├── project-3.html     (Deep Dive 3)
  ├── blog.html         (Devlog / Insights)
  ├── contact.html      (The Link-up)
  ├── assets/
  │   ├── images/
  │   └── media/
  └── README.md
```

### Phase 2: The Core Experience
Build the pages that tell your story.

*   **`index.html` (Home):** Your elevator pitch.
    *   `header` with your name and a catchy tagline.
    *   `nav` with links to **all** 7+ pages.
    *   `article` cards summarizing your key sections.
*   **`about.html` (The Person):** High-quality profile image and a timeline of your journey so far.
*   **`projects.html` (The Work):** A list of at least 6 projects. Show off what you’ve built!

### Phase 3: The Blog (Your Developer Voice)
This is where you share your learning journey. This is a crucial requirement for this weekend.

**Requirements for `blog.html`:**
*   **Table of Contents:** A list of blog posts at the top using anchor links (`#id`) to jump to the full content below.
*   **Minimum 3 Posts:** Each post should be wrapped in an `<article>` tag.
*   **Post Content Ideas:**
    1.  *“My First Week at Mctaba Labs”* - Reflections on where you started.
    2.  *“Why Semantic HTML Matters”* - Explaining what you've learned about clean code.
    3.  *“The Project I'm Most Proud Of”* - A deep dive into one of your creations.

### Phase 4: Project Deep Dives
Don't just list projects, explain them. Create **3 detail pages** (`project-1.html`, etc.) that cover:
*   What problem does it solve?
*   What was your approach?
*   What was the hardest thing you learned while building it?

### Phase 5: Interaction & Forms
Build a functional `contact.html`.
*   A full form with: `name`, `email`, a `dropdown` for subject, `radio buttons` for contact preference, and a `textarea`.
*   An FAQ section using the `<details>` and `<summary>` tags.

---

## Required Components

| Component | Requirement |
| :--- | :--- |
| **Semantic HTML** | Use `header`, `nav`, `main`, `section`, `article`, `aside`, `footer`. |
| **Navigation** | Consistent `nav` on every single page. |
| **Media** | At least 1 video, audio, or iframe (e.g., Google Map/YouTube). |
| **Tables** | At least one table (e.g., a pricing table or a skills matrix). |
| **Accessibility** | All images MUST have `alt` tags. All inputs MUST have `<label>`. |

---

## Git & Submission

You are expected to have **at least 8 clean commits**. Don't just commit everything at the end!

**Example Commit Messages:**
*   `feat: initial folder structure and boilerplate`
*   `feat: implement shared navigation and footer`
*   `feat: add home page content and structure`
*   `feat: build out internal blog with anchor links`
