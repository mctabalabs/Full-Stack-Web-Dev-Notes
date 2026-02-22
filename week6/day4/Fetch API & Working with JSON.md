# Fetch API & Working with JSON

## Learning Objectives
By the end of this lesson, you will be able to:
- Use the Fetch API to perform HTTP requests in the browser.
- Parse JSON responses and handle common response body formats.
- Inspect the properties of the Response object (status, headers, body).
- Handle HTTP domain errors gracefully in addition to network failures.
- Send data securely via `POST`, `PUT`, and `DELETE` requests.
- Integrate fetch logic into a simple UI to render dynamic data.

---

## 1. Introduction to the Fetch API
The `fetch()` API provides a modern, Promise-based way to make network requests (like getting data from an external server). It completely replaces the older, clunkier `XMLHttpRequest` approach.

**Basic Syntax:**
```javascript
fetch(url, [options])
  .then(response => { /* handle the HTTP response */ })
  .catch(error => { /* handle a total network failure */ });
```
- **`url`**: The endpoint you are requesting data from (e.g., `'https://api.example.com/data'`).
- **`options`** *(optional)*: An object where you define the `method` (GET, POST, etc.), `headers`, `body`, and more. If omitted, `fetch` defaults to a `GET` request.

---

## 2. The Response Object
When `fetch()` successfully reaches a server, it fulfills its Promise and returns a **Response object**. This object contains information about the server's reply.

```javascript
fetch('https://jsonplaceholder.typicode.com/users/1')
  .then((response) => {
    console.log('Status Code:', response.status);        // e.g., 200, 404, 500
    console.log('Is it OK?:', response.ok);              // true if status is 200-299
    console.log('Content Type:', response.headers.get('Content-Type'));
    
    // We must parse the body! This returns ANOTHER promise.
    return response.json(); 
  })
  .then((data) => {
    // Now we have the actual JavaScript object!
    console.log('User Data:', data);
  })
  .catch((err) => console.error('Network error:', err));
```

### Body Parsing Methods
Data transferred over the internet isn't sent as JavaScript objects; it's sent as text, streams, or buffers. You must call a method to read it.
- **`.json()`**: Parses a JSON string and returns a Promise that resolves to a JavaScript object/array. (This is the most common!)
- **`.text()`**: Returns a Promise that resolves to raw text.
- **`.blob()`**: Used for downloading files (like images or PDFs).

> **Common Pitfall: You can only read the body ONCE.**
> Calling `.json()` or `.text()` consumes the data stream. If you try to do `response.json()` and then immediately do `response.text()` on the same response object, JavaScript will throw an error!

---

## 3. Handling HTTP Errors
This is the most common mistake beginners make with `fetch()`!
**`fetch()` ONLY rejects its Promise on a total network error** (like the user losing internet connection or DNS failing). 

It **DOES NOT reject** if the server returns a 404 (Not Found) or 500 (Internal Server Error)! To handle HTTP errors correctly, you MUST manually check `response.ok`.

```javascript
fetch('https://jsonplaceholder.typicode.com/users/999') // This user doesn't exist
  .then((response) => {
    // Manually intercept HTTP errors
    if (!response.ok) {
      throw new Error(`HTTP Error! Status: ${response.status} - ${response.statusText}`);
    }
    return response.json();
  })
  .then((user) => console.log(user))
  .catch((err) => {
    // This now catches BOTH network failures AND our manually thrown HTTP errors!
    console.error('Request failed:', err.message);
  });
```

---

## 4. Sending Data: POST, PUT, DELETE
To send data *to* a server (e.g., creating a new user or submitting a form), you must configure the `options` object.

```javascript
// The data we want to send
const newPost = { 
  title: 'Hello World', 
  body: 'My first post!', 
  userId: 1 
};

fetch('https://jsonplaceholder.typicode.com/posts', {
  method: 'POST', // Tell the server this is a creation request
  headers: {
    // Critical: Tell the server to expect JSON data
    'Content-Type': 'application/json' 
  },
  // We must serialize the JS Object into a JSON String before sending
  body: JSON.stringify(newPost) 
})
  .then(res => {
    if (!res.ok) throw new Error(res.statusText);
    return res.json();
  })
  .then(data => console.log('Successfully created:', data))
  .catch(err => console.error('Creation failed:', err));
```

> **Pro Tip:** APIs often require authentication to accept POST requests. You would include that in the headers, like so: 
> `'Authorization': 'Bearer YOUR_SECRET_TOKEN'`.

---

## 5. CORS & Credentials
For security, browsers restrict requests made to *different* domains (e.g., your site `mysite.com` requesting data from `api.anothersite.com`). This is called the **Same-Origin Policy**.

**CORS (Cross-Origin Resource Sharing)** is how servers bypass this restriction. By default, `fetch` uses `mode: 'cors'`.
- If the external API's server doesn't respond with a specific header (`Access-Control-Allow-Origin`), your browser will block the response and throw a CORS error.
- **Cookies & Logins:** By default, `fetch` does not send cookies on cross-origin requests. If your API relies on cookies for logging in, you must add `credentials: 'include'` to your `fetch` options!

```javascript
fetch('https://api.example.com/secure-data', {
  credentials: 'include' // Tells the browser to send cookies along with the request
});
```

---

## 6. In-Class Activity: Build a Post Viewer

Let's combine `fetch`, `async/await`, and DOM manipulation!

**1. The HTML (`index.html`)**
```html
<button id="load">Load Posts</button>
<ul id="posts-list"></ul>
```

**2. The JavaScript (`app.js`)**
```javascript
const loadBtn = document.getElementById('load');
const list = document.getElementById('posts-list');

loadBtn.addEventListener('click', async () => {
  // Show a loading state!
  list.innerHTML = '<li>Loading posts...</li>';
  
  try {
    const res = await fetch('https://jsonplaceholder.typicode.com/posts');
    
    if (!res.ok) throw new Error(`Server error: ${res.status}`);
    
    const posts = await res.json();
    
    // Clear the loading message
    list.innerHTML = '';
    
    // Slice grabbing just the first 10 posts
    posts.slice(0, 10).forEach(post => {
      const li = document.createElement('li');
      li.innerHTML = `<strong>${post.id}</strong>: ${post.title}`;
      list.appendChild(li);
    });
    
  } catch (err) {
    list.innerHTML = `<li style="color: red;">Error: ${err.message}</li>`;
  }
});
```

---

## 7. Assignment: Enhance the Post Viewer

**Goal:** Allow users to create their own posts and have them appear in the list.

### Requirements:
1. **Add a Form:** Update your HTML to include a form for creating a new post.
```html
<form id="post-form">
  <input id="title" placeholder="Post Title" required />
  <textarea id="body" placeholder="Post Body" required></textarea>
  <button type="submit">Create Post</button>
</form>
```

2. **Handle the Submission:** In your JavaScript, add an `submit` event listener to the form.
3. **Prevent Reload:** Call `e.preventDefault()` so the page doesn't refresh.
4. **Send the Request:** Grab the values from the inputs and make a `POST` request to `https://jsonplaceholder.typicode.com/posts`.
5. **Update the UI:** If the request succeeds, manually create a new `<li>`, fill it with the successful response data, and `prepend()` it to your `<ul>` so it shows up at the top! Make sure to handle errors gracefully.

### Stretch Goal: Deleting a Post
Add a "Delete" `<button>` inside every single `<li>` that gets generated. When clicked:
1. Fire a `DELETE` request to `https://jsonplaceholder.typicode.com/posts/POST_ID`.
2. If the request is successful (the API will return an empty object `{}` with a 200 OK status), use `.remove()` to delete that specific `<li>` from the DOM entirely!
