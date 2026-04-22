# Understanding Data Encoding: Base64 and Beyond

## Learning Goals

By the end of this lesson, you should be able to:

- Explain what encoding is and how it differs from encryption
- Describe why Base64 exists and where you will encounter it
- Encode and decode Base64 strings in JavaScript (both browser and Node.js)
- Recognize Base64 in the wild: Basic Auth headers, JWTs, data URIs, and M-Pesa passwords
- Understand URL encoding and when to apply it

**Prior-week concepts you will use today:**
- String manipulation, template literals (Week 3)
- Buffer in Node.js (Week 10, Day 2 -- M-Pesa auth)
- HTTP headers and Authorization (Week 9, Week 10)

**Estimated time:** 1 hour

---

## 1. Encoding vs Encryption

These two words sound similar but mean very different things:

| | Encoding | Encryption |
|---|---|---|
| **Purpose** | Transform data into a format that a system can handle | Protect data so only authorized parties can read it |
| **Reversible?** | Yes, by anyone -- no key needed | Yes, but only with the correct key |
| **Secret?** | No. The algorithm is public. Anyone can decode. | Yes. Without the key, the data is unreadable. |
| **Example** | Base64, URL encoding, UTF-8 | AES, RSA, HTTPS/TLS |

> **Critical:** Encoding is NOT security. Base64-encoding a password does not protect it. Anyone can decode it instantly. If you need to protect data, you need encryption or hashing -- not encoding.

So why does encoding exist? Because different systems have different rules about what characters they accept. Email systems only handle ASCII text. URLs cannot contain spaces. JSON cannot contain raw binary data. Encoding translates data into formats these systems can handle, and the receiver decodes it back to the original.

---

## 2. Base64 Encoding

Base64 is the encoding scheme you will encounter most often as a web developer. It converts any data (including binary data like images) into a string that uses only 64 "safe" characters:

```
A-Z  (26 characters)
a-z  (26 characters)
0-9  (10 characters)
+    (1 character)
/    (1 character)
=    (padding)
```

These 64 characters are safe in virtually every system: HTTP headers, JSON strings, XML, email, URLs (with a slight variation). That is why Base64 is everywhere.

### How It Works (Simplified)

Base64 takes every 3 bytes of input and converts them into 4 characters of output. This means Base64-encoded data is always about 33% larger than the original.

```
Original:  "Hi"  ->  72 105  (two bytes in ASCII)
Base64:    "SGk="              (four characters)
```

The `=` at the end is padding. Base64 works in groups of 3 bytes. If your input is not a multiple of 3, it adds `=` signs to fill the gap. One `=` means 2 bytes were encoded in the last group. Two `==` means 1 byte was encoded.

You do not need to memorize the algorithm. You just need to know: Base64 turns any data into a long string of letters, numbers, `+`, `/`, and `=`.

---

## 3. Base64 in JavaScript

### In Node.js (Server-Side)

Node.js uses the `Buffer` class for encoding. You saw this on Day 2 when building the M-Pesa service:

```javascript
// Encoding: string -> Base64
const encoded = Buffer.from("Hello, World!").toString("base64");
console.log(encoded); // "SGVsbG8sIFdvcmxkIQ=="

// Decoding: Base64 -> string
const decoded = Buffer.from("SGVsbG8sIFdvcmxkIQ==", "base64").toString("utf8");
console.log(decoded); // "Hello, World!"
```

`Buffer.from(string)` creates a buffer from the string. `.toString("base64")` converts that buffer to a Base64 string. To decode, you create a buffer from the Base64 string (telling it the input is `"base64"`), then convert to UTF-8 text.

### In the Browser (Client-Side)

Browsers provide two built-in functions:

```javascript
// Encoding
const encoded = btoa("Hello, World!");
console.log(encoded); // "SGVsbG8sIFdvcmxkIQ=="

// Decoding
const decoded = atob("SGVsbG8sIFdvcmxkIQ==");
console.log(decoded); // "Hello, World!"
```

`btoa` stands for "binary to ASCII" (encode). `atob` stands for "ASCII to binary" (decode).

> **Gotcha:** `btoa` only works with ASCII strings. If your string contains characters outside ASCII (like emoji or Swahili with diacritics), use this pattern instead:
> ```javascript
> const encoded = btoa(unescape(encodeURIComponent("Habari! 🇰🇪")));
> ```

---

## 4. Where You Encounter Base64

### 4.1 HTTP Basic Authentication

On Day 2, you built the M-Pesa OAuth token request. The Daraja API uses HTTP Basic Authentication:

```javascript
// From your mpesa.js service
const auth = Buffer.from(
  `${process.env.MPESA_CONSUMER_KEY}:${process.env.MPESA_CONSUMER_SECRET}`
).toString("base64");

// The header looks like:
// Authorization: Basic Y29uc3VtZXJfa2V5OmNvbnN1bWVyX3NlY3JldA==
```

HTTP Basic Auth is a standard: combine the username and password with a colon, Base64-encode the result, and send it in the `Authorization` header with the prefix `Basic `.

Every API that uses Basic Auth works this way. You will see it in database connections, third-party APIs, and CI/CD systems.

### 4.2 The M-Pesa STK Push Password

The STK push password is another Base64-encoded value:

```javascript
// Shortcode + Passkey + Timestamp, Base64-encoded
const password = Buffer.from(shortcode + passkey + timestamp).toString("base64");
```

Safaricom uses this to verify that your server is authorized to trigger payments for this shortcode at this time. The timestamp prevents replay attacks -- someone cannot reuse an old password because the timestamp will not match.

### 4.3 JSON Web Tokens (JWTs)

If you work with authentication systems (Week 18+), you will encounter JWTs. A JWT looks like this:

```
eyJhbGciOiJIUzI1NiJ9.eyJ1c2VySWQiOjEyM30.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

Those three parts separated by dots are all Base64-encoded JSON objects:

```javascript
// Header (first part)
atob("eyJhbGciOiJIUzI1NiJ9");
// '{"alg":"HS256"}'

// Payload (second part)
atob("eyJ1c2VySWQiOjEyM30");
// '{"userId":123}'
```

The third part is a cryptographic signature (not just encoding -- this part provides security).

> **Important:** The payload of a JWT is only Base64-encoded, not encrypted. Anyone can decode it and read the contents. Never put secrets (passwords, credit card numbers) in a JWT payload.

### 4.4 Data URIs

You can embed small files directly in HTML or CSS using Base64 data URIs:

```html
<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUg..." />
```

Instead of loading an image from a server, the image data is embedded directly in the HTML as a Base64 string. This is useful for small icons or logos where saving an extra HTTP request matters more than the 33% size increase.

---

## 5. URL Encoding

URL encoding (also called percent-encoding) is a different encoding scheme used specifically for URLs. URLs can only contain certain characters. Spaces, special characters, and non-ASCII characters must be encoded.

```javascript
// Encoding
encodeURIComponent("hello world & goodbye");
// "hello%20world%20%26%20goodbye"

// Decoding
decodeURIComponent("hello%20world%20%26%20goodbye");
// "hello world & goodbye"
```

Each unsafe character is replaced with `%` followed by its two-digit hex code. A space becomes `%20`. An ampersand becomes `%26`.

### When to use URL encoding

- Query parameters: `?name=John%20Kamau&amount=5000`
- Path segments with special characters
- Any user input that goes into a URL

JavaScript's `encodeURIComponent()` handles this automatically. Use it whenever you build URLs from dynamic data:

```javascript
const name = "John & Jane Kamau";
const url = `/api/search?name=${encodeURIComponent(name)}`;
// "/api/search?name=John%20%26%20Jane%20Kamau"
```

---

## 6. Quick Reference

| Encoding | Purpose | Characters Used | Size Change |
|---|---|---|---|
| **Base64** | Binary data in text-only contexts | A-Z, a-z, 0-9, +, /, = | +33% |
| **URL encoding** | Special characters in URLs | %XX hex codes | Varies |
| **UTF-8** | Universal text representation | Variable-length bytes | Depends on characters |
| **Hex** | Binary data as readable hex | 0-9, A-F | +100% |

---

## Activity

1. Open a Node.js REPL (`node` in your terminal) and try these:
   - Base64-encode your name: `Buffer.from("Your Name").toString("base64")`
   - Decode it back: `Buffer.from("result", "base64").toString("utf8")`
   - Encode a key:secret pair like the M-Pesa auth: `Buffer.from("mykey:mysecret").toString("base64")`

2. Open your browser console and try the browser equivalents:
   - `btoa("Your Name")`
   - `atob("result")`

3. Find the Base64-encoded values in your capstone project's M-Pesa service (`server/services/mpesa.js`). Identify which values are being encoded and why.

### Stretch Goals:
- Take a small image file (under 1KB), read it with `fs.readFileSync`, and Base64-encode it. Create a data URI string. Paste it into an `<img>` tag in an HTML file and verify the image displays.
- Decode the header and payload of a JWT from jwt.io. What information is in each part?

---

## Key References

- [MDN: Base64 encoding and decoding](https://developer.mozilla.org/en-US/docs/Glossary/Base64)
- [MDN: btoa() and atob()](https://developer.mozilla.org/en-US/docs/Web/API/btoa)
- [MDN: encodeURIComponent()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/encodeURIComponent)
- [Node.js Buffer documentation](https://nodejs.org/api/buffer.html)
