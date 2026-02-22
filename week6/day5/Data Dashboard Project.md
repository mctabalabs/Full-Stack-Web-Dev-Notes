# Data Dashboard Project

## Learning Objectives
By the end of this project, you will be able to:
- Combine multiple asynchronous patterns (Promises and `async/await`) to build a cohesive UI.
- Fetch data concurrently from two independent APIs using `Promise.all`.
- Display and toggle loading states.
- Render successful results and display graceful error messages if APIs fail.
- Structure code into reusable helper functions for readability and maintainability.

---

## 1. Project Brief
We are going to build a small browser dashboard. When the page loads, it will concurrently fetch:
1. **Current weather** for a specific city.
2. **A random quote**.

Each piece of data will be displayed in its own "card" on the screen. The cards will show a spinner/loading text while the request is running, and will gracefully display distinct error messages if one (or both) of the requests fail.

---

## 2. HTML & CSS Skeleton
First, let's create our structural file: `dashboard.html`.

This file contains two side-by-side cards. Notice that inside each `.card`, there is a `.loading` div and a hidden `.data` div. We will toggle the visibility of these divs using JavaScript!

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Data Dashboard</title>
  
  <style>
    body { 
      font-family: sans-serif; 
      display: flex; 
      gap: 1rem; 
      padding: 2rem; 
      background-color: #f9f9f9;
    }
    .card { 
      flex: 1; 
      border: 1px solid #ccc; 
      border-radius: 8px; 
      padding: 1rem; 
      background-color: white;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1); 
    }
    .loading { 
      text-align: center; 
      font-style: italic; 
      color: #666;
    }
    .error { 
      color: #d9534f; /* A nice error red */
      font-weight: bold;
    }
    .data { 
      margin-top: 0.5rem; 
    }
  </style>
  
</head>
<body>

  <!-- Weather Card -->
  <div class="card" id="weather-card">
    <h2>Weather</h2>
    <div class="loading">Loading weather...</div>
    <div class="data" style="display:none;"></div>
  </div>

  <!-- Quote Card -->
  <div class="card" id="quote-card">
    <h2>Random Quote</h2>
    <div class="loading">Loading quote...</div>
    <div class="data" style="display:none;"></div>
  </div>

  <script src="dashboard.js"></script>
</body>
</html>
```

---

## 3. The APIs We'll Use
We will be requesting data from two free external endpoints:

- **Weather API:**  
  `https://api.openweathermap.org/data/2.5/weather?q=Nairobi&units=metric&appid=YOUR_API_KEY`  
  *(Note: You can get a free API key by signing up at [OpenWeatherMap](https://openweathermap.org/)).*
- **Quote API:**  
  `https://zenquotes.io/api/random`

---

## 4. JavaScript: `dashboard.js`
Create a `dashboard.js` file. We are going to build this out in 5 logical steps. 

```javascript
// dashboard.js

// ==========================================
// 1. Cache DOM Elements
// ==========================================
// It is best practice to grab all the elements you need at the top of your file.
const weatherCard = document.getElementById('weather-card');
const quoteCard   = document.getElementById('quote-card');

const weatherLoading = weatherCard.querySelector('.loading');
const weatherData    = weatherCard.querySelector('.data');

const quoteLoading   = quoteCard.querySelector('.loading');
const quoteData      = quoteCard.querySelector('.data');

// ==========================================
// 2. Helper Functions
// ==========================================
// A reusable function to show an error on a specific card
function showError(loadingEl, dataEl, message) {
  loadingEl.style.display = 'none'; // Hide the loading text
  dataEl.style.display = 'block';   // Show the data div
  dataEl.innerHTML = `<p class="error">❌ ${message}</p>`; // Insert the error
}

// ==========================================
// 3. Data Fetching Functions
// ==========================================
async function fetchWeather() {
  const API_KEY = 'YOUR_API_KEY'; // Replace with your actual key!
  const url = `https://api.openweathermap.org/data/2.5/weather?q=Nairobi&units=metric&appid=${API_KEY}`;
  
  const res = await fetch(url);
  if (!res.ok) throw new Error(`Weather API Error: ${res.status}`);
  
  const json = await res.json();
  
  // Return only the data we actually care about styling!
  return {
    temp: json.main.temp,
    desc: json.weather[0].description,
    city: json.name
  };
}

async function fetchQuote() {
  const url = 'https://zenquotes.io/api/random';
  const res = await fetch(url);
  
  if (!res.ok) throw new Error(`Quote API Error: ${res.status}`);
  
  const [quoteObj] = await res.json(); // zenquotes API returns an array, so we destructure it!
  
  return {
    text: quoteObj.q,
    author: quoteObj.a
  };
}

// ==========================================
// 4. Initialization & Concurrency
// ==========================================
async function initDashboard() {
  try {
    /* 
     PRO TIP: Promise.all with individual .catch()
     By attaching a .catch() directly to the individual fetches inside the array,
     we prevent Promise.all from failing entirely if only ONE of them fails. 
     Instead, it resolves the failed one as an object: { error: "message" }.
    */
    const [weather, quote] = await Promise.all([
      fetchWeather().catch(err => ({ error: err.message })),
      fetchQuote().catch(err => ({ error: err.message }))
    ]);

    // Handle Weather Result
    if (weather.error) {
      showError(weatherLoading, weatherData, weather.error);
    } else {
      weatherLoading.style.display = 'none';
      weatherData.style.display = 'block';
      weatherData.innerHTML = `
        <p><strong>${weather.city}</strong></p>
        <p>${weather.temp}°C</p>
        <p style="text-transform: capitalize;">${weather.desc}</p>
      `;
    }

    // Handle Quote Result
    if (quote.error) {
      showError(quoteLoading, quoteData, quote.error);
    } else {
      quoteLoading.style.display = 'none';
      quoteData.style.display = 'block';
      quoteData.innerHTML = `
        <blockquote style="font-size: 1.1rem; font-style: italic;">"${quote.text}"</blockquote>
        <p style="text-align: right;">— <strong>${quote.author}</strong></p>
      `;
    }
  } catch (err) {
    // This catch is a fallback for catastrophic failures in Promise.all itself
    console.error('Dashboard init fatally crashed:', err);
  }
}

// ==========================================
// 5. Kick off!
// ==========================================
// Wait for the HTML to be fully loaded before running our script
document.addEventListener('DOMContentLoaded', initDashboard);
```

---

## 5. Why Did We Write It This Way?

### Concurrent Fetching
We used `Promise.all([...])` to fire both requests in parallel. If `fetchWeather` takes 2 seconds and `fetchQuote` takes 2 seconds, the whole app takes 2 seconds to load (not 4!). 

The trick here is that we wrapped each individual fetch in a `.catch()`. If one fails, it doesn't crash the other card!

### Error Handling & DOM Manipulation
We cleanly separated our concerns. The `fetchX()` functions only care about getting data and throwing HTTP errors. The `initDashboard()` function only cares about painting the screen based on whether an error was returned. We also built a reusable `showError` helper to avoid writing the same DOM logic twice.

> **Security Note:** We used `innerHTML` for brevity in this lesson. In a real-world production app, it is safer to use `document.createElement()` and `.textContent` to prevent Cross-Site Scripting (XSS) attacks.

---

## 6. Testing & Validation
Because we built error handling per-card, we can test it thoroughly:
1. **Happy Path:** Put in a valid API key. Both cards should load and show data.
2. **Weather Failure:** Put in a fake API key (e.g., `'12345'`). The first card should turn red with "Weather API Error: 401", but the quote should still load perfectly!
3. **Total Network Down:** Turn off your WiFi and refresh. Both cards should show red `Failed to fetch` errors.

---

## 7. Assignment: Extend the Dashboard

Take your dashboard to the next level by implementing the following features:

### 1. Add a Third API!
Add a third card to the dashboard that fetches a random image of a dog using the [Dog CEO API](https://dog.ceo/dog-api/).
- Endpoint: `https://dog.ceo/api/breeds/image/random`
- Show a spinner, then render an `<img src="...">` tag inside the `.data` div.

### 2. A "Refresh All" Button
Place a button at the top of the HTML page. Ensure that when clicked, it:
1. Resets all cards to show "Loading...".
2. Hides the `.data` divs.
3. Re-runs `initDashboard()`.

### 3. Pagination for Quotes
Add a "Next Quote" button *specifically inside* the Quote card. Clicking this button should *only* trigger `fetchQuote()` again without reloading the Weather or the Dog image. 

*(Hint: You may need to extract the quote DOM manipulation logic into its own reusable function!)*

### Stretch Goal: Theming
Add a toggle switch to flip the entire dashboard between "Light" and "Dark" mode by toggling a CSS class on the `<body>`. To make it professional, save the user's preference in `localStorage` so it remembers their choice when they refresh!