# Learning Objectives

By the end of this chapter, you will be able to:

- Differentiate between the Internet and the World Wide Web.
- Explain how web browsers and servers communicate.
- Understand the role of HTML in web development.
- Create and view a basic HTML page in your browser.

## 1. What is the Internet?

The Internet is a vast network of interconnected computers and devices that communicate with each other globally. It is the infrastructure that allows data to travel from one point to another.

**Analogy:** Think of the Internet as a system of highways connecting cities (computers) across the world.

## 2. What is the World Wide Web?

The World Wide Web (WWW) is a collection of information, such as web pages and multimedia content, accessed via the Internet using web browsers. It is a service that operates over the Internet, utilizing protocols like HTTP to transmit data.

**Analogy:** If the Internet is the highway system, the Web is the collection of restaurants, shops, and attractions you can visit along the routes.

## 3. Key Components of the Web

- **Web Browser:** A software application (e.g., Chrome, Firefox, Safari) that retrieves and displays web content.
- **Web Server:** A computer system that hosts websites and serves web pages to users upon request.
- **URL (Uniform Resource Locator):** The unique address used to access a specific resource on the Web.
- **HTTP/HTTPS (HyperText Transfer Protocol/Secure):** Protocols used for transmitting web pages over the Internet.

## 4. How Does a Website Work?

1. You enter a URL into your web browser.
2. The browser sends a request to a DNS (Domain Name System) server to find the IP address associated with the URL.
3. The browser uses the IP address to send a request to the appropriate web server.
4. The web server processes the request and sends back the requested HTML page.
5. The browser receives the HTML content and renders the web page for you to view.

## 5. What is HTML?

HTML (HyperText Markup Language) is the standard language used to create and structure content on the Web. It uses a system of tags to define elements like headings, paragraphs, links, and images.

### Example

```html
<!DOCTYPE html>
<html>
<head>
  <title>My First Web Page</title>
</head>
<body>
  <h1>Hello, World!</h1>
  <p>Welcome to my website.</p>
</body>
</html>
```

## Mini Project: Create Your First Web Page

**Objective:** Build a simple HTML page and view it in your browser.

### Steps

1. Open a text editor (e.g., Notepad, VS Code).
2. Type the following HTML code:

   ```html
   <!DOCTYPE html>
   <html>
   <head>
     <title>My First Web Page</title>
   </head>
   <body>
     <h1>Welcome to My Website</h1>
     <p>This is my first web page using HTML.</p>
   </body>
   </html>
   ```

3. Save the file with a `.html` extension (e.g., `index.html`).
4. Open the file in your web browser by double-clicking it.

**Expected Result:**
A web page displaying a heading "Welcome to My Website" and a paragraph "This is my first web page using HTML."

## Recap Quiz

1. **What is the main difference between the Internet and the World Wide Web?**
   - **Answer:** The Internet is the global network of interconnected computers (the infrastructure), while the World Wide Web is a collection of information (a service) accessed via the Internet.

2. **What does a web browser do?**
   - **Answer:** It retrieves and displays web content from web servers.

3. **What is the purpose of HTML?**
   - **Answer:** HTML structures and defines the content of web pages.

## Additional Resource

For a visual explanation and demonstration, check out this tutorial:
[Introduction to the Internet, the Web and HTML for Absolute Beginners](https://youtu.be/wTfCQSGxTjQ?si=AWzRkygGhyGhqMIm)