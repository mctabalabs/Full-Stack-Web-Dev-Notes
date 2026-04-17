# Working with APIs Properly

> **AI boundaries this week:** 50% manual / 50% AI. Habit: *AI reads docs, you make decisions -- the 50/50 line holds only if every architectural choice stays yours.* See [ai.md](../ai.md).

## Learning Goals

By the end of this topic, learners should be able to:

- Understand the **RESTful lifecycle** of a request and response
- Build robust, production-ready data fetching patterns using `useEffect` and `fetch`/`Axios`
- Organize API logic into a **Service Layer** for maintainability and reuse
- Prevent common pitfalls like **race conditions** and memory leaks
- Master **React Query** (TanStack Query) for professional-grade server state management
- Handle security, environment variables, and CORS errors intentionally
- Implement high-quality UX patterns like **Loading Skeletons**, **Optimistic UI**, and **Retry Logic**

---

## 1) REST APIs: The Communication Language

A REST (Representational State Transfer) API is a set of rules that allows your React frontend to talk to a backend. It uses **Resources** (like `users`, `posts`, `transactions`) and standard **HTTP Methods**.

### The HTTP Toolkit

| Method | Use Case | Analogy |
|---|---|---|
| **GET** | Retrieve data | Reading a book |
| **POST** | Create new data | Writing a new book |
| **PUT** | Replace existing data | Replacing a whole book with a new one |
| **PATCH** | Update part of data | Editing a specific page in a book |
| **DELETE** | Remove data | Tearing a book up |

### Understanding Status Codes (The "Why")

When you make a request, the server responds with a 3-digit number. You must handle these to build a "resilient" UI.

- **2xx (Success)**: `200 OK` (Standard success), `201 Created` (New item made).
- **4xx (Client Error)**: 
    - `400 Bad Request` (You sent garbage data)
    - `401 Unauthorized` (You aren't logged in)
    - `403 Forbidden` (You are logged in, but not allowed to see this)
    - `404 Not Found` (This resource doesn't exist)
    - `429 Too Many Requests` (Rate limiting - you're attacking the server!)
- **5xx (Server Error)**: `500 Internal Server Error` (The backend crashed - not your fault, but you must handle the UI crash).

---

## 2) The "React-Native" Way: Fetch & useEffect

Before reaching for libraries, you must understand how React handles the network lifecycle. Data fetching is considered a **Side Effect** because it's an "escape hatch" from the rendering logic into the outside world (the internet).

### The Essential 3-State Model

Every network call in React has three distinct UI states: **Idle/Loading**, **Success**, and **Error**.

```jsx
import { useEffect, useState } from "react";

export default function PostsList() {
  const [posts, setPosts] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    // 1. The Clean-up Flag (Prevents "Race Conditions")
    let isMounted = true;

    async function fetchData() {
      setIsLoading(true);
      setError(null);

      try {
        const response = await fetch("https://jsonplaceholder.typicode.com/posts");
        
        // Critical: fetch only throws on network failure, not on 404/500!
        if (!response.ok) {
          throw new Error(`Error: ${response.status} - ${response.statusText}`);
        }

        const data = await response.json();
        
        if (isMounted) {
          setPosts(data);
          setIsLoading(false);
        }
      } catch (err) {
        if (isMounted) {
          setError(err.message);
          setIsLoading(false);
        }
      }
    }

    fetchData();

    // 2. The Cleanup Function
    return () => {
      isMounted = false;
    };
  }, []);

  if (isLoading) return <div className="skeleton">Loading posts...</div>;
  if (error) return <div className="error-card">Failed to load: {error}</div>;
  if (posts.length === 0) return <div>No posts found.</div>;

  return (
    <ul className="post-grid">
      {posts.slice(0, 10).map((post) => (
        <li key={post.id} className="post-card">{post.title}</li>
      ))}
    </ul>
  );
}
```

### Why the `isMounted` / `ignore` flag?
Imagine a user clicks "Next Page" quickly 5 times. 
Five requests go out. They may finish in a different order than they were sent.
The `ignore` flag ensures that only the **last** request updates the state, preventing "flickering" or data from Page 1 appearing on Page 5.

---

## 3) Axios: Professional HTTP Client

While `fetch` is built-in, **Axios** is favored in production because it's more expressive and handles errors better.

### Why Axios?
- **Throws Errors Automatically**: Raw `fetch` considers a `500 Server Error` as a "success" (it reached the server!). Axios automatically triggers the `catch` block for any non-2xx code.
- **JSON by Default**: No more `response.json()`.
- **Interceptors**: Global "checkpoints" for your requests (perfect for adding Auth headers).
- **Timeouts**: Cancel a request if the server is taking too long.

### Recommended Pattern: The API Client

Don't use `axios.get()` everywhere. Create a centralized client.

```js
// src/api/client.js
import axios from "axios";

const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL || "https://api.example.com",
  timeout: 8000,
  headers: {
    "Content-Type": "application/json",
  },
});

// Example Interceptor: Attaching a Bearer Token
apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem("auth_token");
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

export default apiClient;
```

---

## 4) Organizing Logic: The Service Layer

As apps grow, your components shouldn't know the exact URL strings. Components should say: *"Hey, get me the list of jobs."* They shouldn't care if it's `/v1/jobs` or `/api/listing`.

### Folder Structure
```
src/
  api/
    client.js      // Axios instance
    posts.service.js
    auth.service.js
```

### Example Service
```js
// src/api/posts.service.js
import apiClient from "./client";

export const PostsService = {
  getAll: async (params = {}) => {
    const response = await apiClient.get("/posts", { params });
    return response.data;
  },

  getById: async (id) => {
    const response = await apiClient.get(`/posts/${id}`);
    return response.data;
  },

  create: async (postData) => {
    const response = await apiClient.post("/posts", postData);
    return response.data;
  },
};
```

**Why do this?**
- **Single Source of Truth**: If the URL changes, you update one line, not 50 components.
- **Type Safety**: Easier to add TypeScript types to one central service.
- **Clean Components**: `const posts = await PostsService.getAll()` is much more readable.

---

## 5) Managing Cache: React Query (TanStack)

The biggest problem with `useEffect` for fetching is that it **doesn't cache**. 
If you visit "Page A", go to "Page B", and come back to "Page A", `useEffect` will fetch again. This feels slow.

**React Query** manages "Server State" (data from the backend) separately from "UI State" (modals, forms).

### The Big 4 Concepts

1. **QueryClient**: The brain that holds the cache.
2. **queryKey**: The unique ID for a piece of data (e.g., `['posts', userId]`).
3. **queryFn**: The function that gets the data (your Service Layer!).
4. **staleTime**: How long the data stays "fresh" before React Query tries to refetch it.

### Basic Fetch with `useQuery`

```jsx
import { useQuery } from "@tanstack/react-query";
import { PostsService } from "../api/posts.service";

function RecentPosts() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['posts', 'recent'],
    queryFn: () => PostsService.getAll({ limit: 5 }),
    staleTime: 1000 * 60 * 5, // Keep fresh for 5 minutes
  });

  if (isLoading) return <p>Loading...</p>;
  if (error) return <p>Oops: {error.message}</p>;

  return data.map(post => <div key={post.id}>{post.title}</div>);
}
```

### Mutating Data with `useMutation`

Creating, updating, or deleting data requires `useMutation`.

```jsx
import { useMutation, useQueryClient } from "@tanstack/react-query";
import { PostsService } from "../api/posts.service";

function CreatePost() {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: (newPost) => PostsService.create(newPost),
    onSuccess: () => {
      // "Invalidate" tells React Query: "The posts list is now old, please refetch it!"
      queryClient.invalidateQueries({ queryKey: ['posts'] });
      alert("Post created!");
    },
  });

  return (
    <button onClick={() => mutation.mutate({ title: "Hello World" })}>
      {mutation.isPending ? 'Saving...' : 'Create Post'}
    </button>
  );
}
```

---

## 6) Security & Environment Variables

### Frontend Environment Variables
In Vite, you use `.env` files. Variables must start with `VITE_`.

```
# .env.local
VITE_API_BASE_URL=https://api.mctaba.labs/v1
```

### CRITICAL: Secrets Lesson
**Nothing in a React `.env` file is a secret.**
When you run `npm run build`, these values are hardcoded into the Javascript. Any user can view them by looking at the network tab or the source code.
- **DO NOT** put database passwords or private API keys (like AWS_SECRET_KEY) in your React project.
- **DO** use them for backend URLs, public keys, or feature flags.

### Dealing with CORS
If you see "CORS Error" in the console, it means the server doesn't trust your domain. This is a **server-side fix**. The server must set the `Access-Control-Allow-Origin` header to allow your React app's URL.

---

## 7) Advanced UX: Optimistic UI

Optimistic UI is the secret to apps like Instagram and Twitter feeling "instant."
**The Pattern**: When you click "Like", the UI updates **before** the server responds. If the server failure happens later, the UI "rolls back."

React Query makes this relatively simple via the `onMutate` callback.

---

## 8) Outcome Checklist (Learner Competencies)

- [ ] Can explain the difference between GET and POST in plain English
- [ ] can handle `fetch` errors properly using `response.ok`
- [ ] can build a centralized `apiClient` using Axios
- [ ] understands why `useEffect` cleanup is needed for fetching
- [ ] can switch a project from `useEffect` to `useQuery`
- [ ] can differentiate between development and production API URLs using `.env`

---

## 9) Practice Projects

### Project 1: The Transaction Ledger (Fintech Style)
**Goal**: Build a mini M-Pesa style transaction dashboard.
- Fetch transaction history from a mock API (JSON Server).
- Implement filters (by Date, by Status).
- Use **Loading Skeletons** instead of simple "Loading..." text.
- Use a **Service Layer** to keep logic clean.

### Project 2: Github Profile Explorer
**Goal**: Use the real public Github API (`api.github.com`).
- Input a username and fetch profile + repositories.
- Handle `404 Not Found` (User doesn't exist) and `403` (Rate limit reached) gracefully.
- Practice `useQuery` for instant data when clicking "Back".

### Project 3: Admin Dashboard (Mutations)
**Goal**: A full CRUD app for managing a product inventory.
- List products (GET).
- Add new product with a form (POST).
- Delete product with a confirmation dialog (DELETE).
- Use `queryClient.invalidateQueries` to refresh the list automatically after changes.

---

## Recommended Reading

| Link | Importance |
|---|---|
| [React Docs: Fetching Data](https://react.dev/learn/synchronizing-with-effects#fetching-data) | Core React concept |
| [Axios Official Docs](https://axios-http.com/docs/intro) | Essential tool |
| [TanStack Query Quickstart](https://tanstack.com/query/latest/docs/framework/react/quickstart) | Next-level data management |
| [MDN: HTTP Response Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) | Universal standard |
| [Vite: Env Variables Guide](https://vite.dev/guide/env-and-mode) | Configuration standard |

---
*McTaba Labs - Week 9, Day 1 - Working with APIs properly*
