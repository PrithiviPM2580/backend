# HTTP (HyperText Transfer Protocol)

## What is HTTP?

**HTTP** = The language that browsers and servers use to talk to each other

Think of it like a conversation! 🗣️ When you visit a website, your browser asks the server "Can I have this page?" and the server responds with the page content.

---

## How HTTP Works

```
Browser (Client)  →  HTTP Request  →  Server
                  ←  HTTP Response ←
```

**Example:**

```
You type: www.example.com

Browser: "Hey server, give me the homepage" (REQUEST)
Server: "Here's the HTML, CSS, and images!" (RESPONSE)
```

---

## HTTP Request Headers

Headers are extra information sent with the request

### 1. **User-Agent**

Tells the server what browser/device you're using

**Example:**

```
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/120.0
```

**Why it matters:** Websites can show different content for mobile vs desktop

### 2. **Authorization**

Proves you're allowed to access the resource

**Example:**

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Common types:**

- `Bearer <token>` - JWT tokens
- `Basic <credentials>` - Username/password (base64)
- `API-Key <key>` - API keys

### 3. **Cookie**

Sends saved data (like login info) to the server

**Example:**

```
Cookie: sessionId=abc123; userId=42; theme=dark
```

**Why it matters:** This is how websites remember you're logged in!

### 4. **Accept**

Tells the server what type of data you want

**Example:**

```
Accept: application/json
Accept: text/html
Accept: image/png
```

**Use case:** API might return JSON, but browser might want HTML

---

## General Headers

Headers that apply to both request and response

### 1. **Date**

When the message was sent

**Example:**

```
Date: Mon, 14 Jan 2026 10:30:00 GMT
```

### 2. **Cache-Control**

How long to save the response

**Example:**

```
Cache-Control: max-age=3600          (cache for 1 hour)
Cache-Control: no-cache              (always check with server)
Cache-Control: no-store              (never cache)
```

### 3. **Connection**

How to handle the network connection

**Example:**

```
Connection: keep-alive    (keep connection open)
Connection: close         (close after response)
```

---

## Representation Headers

Describe the data being sent

### 1. **Content-Type**

What kind of data is in the body

**Example:**

```
Content-Type: application/json           (JSON data)
Content-Type: text/html; charset=utf-8   (HTML page)
Content-Type: image/png                  (PNG image)
Content-Type: multipart/form-data        (File upload)
```

### 2. **Content-Length**

Size of the data in bytes

**Example:**

```
Content-Length: 1234
```

### 3. **Content-Encoding**

How the data is compressed

**Example:**

```
Content-Encoding: gzip
Content-Encoding: br
Content-Encoding: deflate
```

### 4. **ETag**

Unique identifier for this version of the resource

**Example:**

```
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
```

**Why it matters:** Browser can check if page changed before re-downloading

---

## Security Headers

Headers that protect against attacks

### 1. **Strict-Transport-Security (HSTS)**

Forces browser to always use HTTPS

**Example:**

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

**Benefit:** Prevents man-in-the-middle attacks

### 2. **Content-Security-Policy (CSP)**

Controls what resources can be loaded

**Example:**

```
Content-Security-Policy: default-src 'self'; script-src 'self' cdn.example.com
```

**Benefit:** Prevents XSS (Cross-Site Scripting) attacks

### 3. **X-Frame-Options**

Prevents your site from being embedded in iframe

**Example:**

```
X-Frame-Options: DENY
X-Frame-Options: SAMEORIGIN
```

**Benefit:** Prevents clickjacking attacks

### 4. **X-Content-Type-Options**

Prevents browser from guessing file types

**Example:**

```
X-Content-Type-Options: nosniff
```

**Benefit:** Prevents MIME type attacks

### 5. **Set-Cookie**

Sends a cookie to be saved in browser

**Example:**

```
Set-Cookie: sessionId=abc123; HttpOnly; Secure; SameSite=Strict
```

**Options:**

- `HttpOnly` - JavaScript can't access it (security)
- `Secure` - Only sent over HTTPS
- `SameSite` - Prevents CSRF attacks

---

## HTTP Methods

Different types of requests for different actions

### 1. **GET** - Retrieve data

```javascript
// Get list of users
GET / api / users;

// Get specific user
GET / api / users / 42;
```

**Use:** Reading data, searching, viewing pages

### 2. **POST** - Create new data

```javascript
// Create a new user
POST /api/users
Body: { "name": "Alice", "age": 25 }
```

**Use:** Form submissions, creating records

### 3. **PUT** - Update entire resource

```javascript
// Replace entire user
PUT /api/users/42
Body: { "name": "Alice Updated", "age": 26, "email": "alice@example.com" }
```

**Use:** Full updates (must send all fields)

### 4. **PATCH** - Update part of resource

```javascript
// Update only the name
PATCH /api/users/42
Body: { "name": "Alice Updated" }
```

**Use:** Partial updates (send only changed fields)

### 5. **DELETE** - Remove data

```javascript
// Delete a user
DELETE / api / users / 42;
```

**Use:** Removing records

### 6. **OPTIONS** - Check what methods are allowed

```javascript
OPTIONS / api / users;
Response: Allow: GET, POST, DELETE;
```

**Use:** CORS preflight requests

---

## Idempotent vs Non-Idempotent

**Idempotent** = Doing the same action multiple times has the same result

| Method | Idempotent?   | Example                                       |
| ------ | ------------- | --------------------------------------------- |
| GET    | ✅ Yes        | Reading same data 10 times = same result      |
| PUT    | ✅ Yes        | Updating to same value 10 times = same result |
| DELETE | ✅ Yes        | Deleting same item 10 times = still deleted   |
| POST   | ❌ No         | Creating user 10 times = 10 users!            |
| PATCH  | ❌ Usually No | Depends on implementation                     |

**Real-world example:**

```javascript
// Idempotent (safe to retry)
PUT /api/users/42
Body: { "status": "active" }
// Run 5 times = user is still "active"

// Non-idempotent (NOT safe to retry)
POST /api/orders
Body: { "product": "Phone", "quantity": 1 }
// Run 5 times = 5 orders created! 💸
```

---

## Preflight Request

**What is it?** A security check that browsers do before certain requests

**When does it happen?**

- Cross-origin requests (different domain)
- Custom headers
- Methods other than GET/POST
- Content-Type other than standard types

**How it works:**

```
Step 1: Browser sends OPTIONS request (preflight)
OPTIONS /api/users
Origin: http://example.com
Access-Control-Request-Method: DELETE

Step 2: Server responds with permissions
Access-Control-Allow-Origin: http://example.com
Access-Control-Allow-Methods: GET, POST, DELETE

Step 3: If allowed, browser sends actual request
DELETE /api/users/42
```

**Example:**

```javascript
// Frontend (example.com) trying to call API (api.example.com)
fetch("https://api.example.com/users", {
  method: "DELETE",
  headers: { Authorization: "Bearer token123" },
});

// Browser automatically sends OPTIONS first
// Only if server allows it, DELETE request is sent
```

---

## HTTP Status Codes

Status codes tell you what happened with the request

### 1xx - Informational

Server received request and is processing

- **100 Continue** - Keep sending data
- **101 Switching Protocols** - Switching to WebSocket

### 2xx - Success

Request was successful!

- **200 OK** - Everything worked perfectly
- **201 Created** - New resource was created (POST)
- **204 No Content** - Success but no data to return (DELETE)

### 3xx - Redirection

Resource moved somewhere else

- **301 Moved Permanently** - Resource moved forever (update bookmarks!)
- **302 Found** - Resource moved temporarily
- **304 Not Modified** - Cached version is still good (use it!)

### 4xx - Client Error

You made a mistake in the request

- **400 Bad Request** - Invalid data sent
- **401 Unauthorized** - Need to log in
- **403 Forbidden** - Logged in but not allowed
- **404 Not Found** - Resource doesn't exist
- **429 Too Many Requests** - Slow down! Rate limit hit

### 5xx - Server Error

Server had a problem

- **500 Internal Server Error** - Something broke on server
- **502 Bad Gateway** - Server got invalid response from another server
- **503 Service Unavailable** - Server is down or overloaded

**Quick Guide:**

```
2xx = 😊 Success!
3xx = 👉 Go here instead
4xx = 🤦 You messed up
5xx = 💥 We messed up
```

---

## HTTP Caching

**Caching** = Saving responses to avoid downloading them again

**Benefits:**

- Faster load times
- Less bandwidth usage
- Reduced server load

**How it works:**

```
First Visit:
Browser → GET /style.css → Server
Browser ← Response + Cache-Control: max-age=3600

Later Visit (within 1 hour):
Browser uses cached file (no request to server!)

After 1 hour:
Browser → GET /style.css + If-None-Match: "etag123"
Server → 304 Not Modified (use cached version)
```

**Cache Headers:**

```
Cache-Control: max-age=3600              (cache for 1 hour)
Cache-Control: public                     (anyone can cache)
Cache-Control: private                    (only user's browser can cache)
Cache-Control: no-cache                   (check with server first)
Cache-Control: no-store                   (never cache)
```

**Example:**

```javascript
// Backend sets cache header
app.get('/api/products', (req, res) => {
  res.set('Cache-Control', 'public, max-age=300') // 5 minutes
  res.json({ products: [...] })
})
```

---

## Content Negotiation

**Content Negotiation** = Server sends different content based on what client wants

**How it works:**

```
Client: "I accept JSON"
Accept: application/json

Server: "Here's JSON"
Content-Type: application/json
{ "data": "..." }
```

**Example:**

```javascript
// Same endpoint, different formats
GET /api/users
Accept: application/json  → Returns JSON
Accept: application/xml   → Returns XML
Accept: text/csv          → Returns CSV

// Language negotiation
Accept-Language: en-US    → English content
Accept-Language: es       → Spanish content
```

**Backend example:**

```javascript
app.get("/api/data", (req, res) => {
  const acceptHeader = req.headers.accept;

  if (acceptHeader.includes("application/json")) {
    res.json({ data: "JSON format" });
  } else if (acceptHeader.includes("text/html")) {
    res.send("<html><body>HTML format</body></html>");
  }
});
```

---

## Compression

**Compression** = Making files smaller before sending them

**Common algorithms:**

1. **gzip** - Most common, good compression
2. **deflate** - Older algorithm
3. **br (Brotli)** - Better compression, modern browsers
4. **zstd** - Newest, very fast

**How it works:**

```
Client → Server: I accept compression
Accept-Encoding: gzip, deflate, br

Server → Client: Here's compressed data
Content-Encoding: gzip
[compressed data]

Browser automatically decompresses
```

**Benefits:**

```
Original file: 1000 KB
Compressed (gzip): 200 KB
Saved: 80% bandwidth! ⚡
```

**Example:**

```javascript
const compression = require("compression");
app.use(compression()); // Auto-compress all responses
```

---

## Persistent Connections and Keep-Alive

**Problem:** Opening/closing connection for each request is slow

**Solution:** Keep connection open for multiple requests!

**Without Keep-Alive:**

```
Request 1: Open connection → Request → Response → Close
Request 2: Open connection → Request → Response → Close
Request 3: Open connection → Request → Response → Close
```

**With Keep-Alive:**

```
Open connection once
  → Request 1 → Response
  → Request 2 → Response
  → Request 3 → Response
Close connection
```

**Benefits:**

- Faster (no connection overhead)
- Less CPU usage
- Better performance

**Headers:**

```
Connection: keep-alive
Keep-Alive: timeout=5, max=100
```

**Modern HTTP/2:**

- Keep-alive is default
- Multiple requests at same time!
- Even faster

---

## Handling Large Requests and Responses

### 1. **Multipart Requests (File Upload)**

**Used for:** Uploading files like images, videos, documents

**How it works:**

```javascript
// Frontend
const formData = new FormData();
formData.append("file", fileInput.files[0]);
formData.append("name", "My File");

fetch("/api/upload", {
  method: "POST",
  body: formData,
});

// Backend
app.post("/api/upload", upload.single("file"), (req, res) => {
  console.log(req.file); // File info
  console.log(req.body.name); // "My File"
  res.json({ success: true });
});
```

**Headers:**

```
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary
```

**Structure:**

```
------WebKitFormBoundary
Content-Disposition: form-data; name="file"; filename="photo.jpg"
Content-Type: image/jpeg

[binary file data]
------WebKitFormBoundary
Content-Disposition: form-data; name="name"

My File
------WebKitFormBoundary--
```

### 2. **Streaming Response (Real-Time Updates)**

**Used for:** Large files, live data, progress updates

**How it works:**

```javascript
// Backend - Stream large file
app.get("/api/video", (req, res) => {
  res.setHeader("Content-Type", "video/mp4");
  const stream = fs.createReadStream("movie.mp4");
  stream.pipe(res); // Send data in chunks
});

// Backend - Real-time updates
app.get("/api/progress", (req, res) => {
  res.setHeader("Content-Type", "text/event-stream");

  let progress = 0;
  const interval = setInterval(() => {
    progress += 10;
    res.write(`data: ${progress}%\n\n`);

    if (progress === 100) {
      clearInterval(interval);
      res.end();
    }
  }, 1000);
});
```

**Benefits:**

- Start displaying data before it's fully downloaded
- Lower memory usage
- Better user experience (see progress)

**Use cases:**

- Video streaming (Netflix, YouTube)
- File downloads with progress bar
- Live logs
- Server-Sent Events (SSE)

---

## Complete Example: API with All Concepts

```javascript
const express = require("express");
const compression = require("compression");
const app = express();

// Enable compression
app.use(compression());

// Enable JSON parsing
app.use(express.json());

// API endpoint with caching
app.get("/api/products", (req, res) => {
  // Set cache header
  res.set("Cache-Control", "public, max-age=300");

  // Set security headers
  res.set("X-Content-Type-Options", "nosniff");
  res.set("X-Frame-Options", "DENY");

  // Content negotiation
  if (req.accepts("json")) {
    res.json({ products: ["Product 1", "Product 2"] });
  } else {
    res.send("<html><body>Products...</body></html>");
  }
});

// POST with authorization
app.post("/api/products", (req, res) => {
  const token = req.headers.authorization;

  if (!token) {
    return res.status(401).json({ error: "Unauthorized" });
  }

  // Create product
  res.status(201).json({ message: "Product created", id: 123 });
});

// Streaming response
app.get("/api/download", (req, res) => {
  res.setHeader("Content-Type", "text/plain");
  res.setHeader("Content-Disposition", 'attachment; filename="data.txt"');

  const stream = fs.createReadStream("large-file.txt");
  stream.pipe(res);
});
```

---

## Key Points to Remember

1. **HTTP is request/response** → Client asks, server answers
2. **Headers carry metadata** → Info about the request/response
3. **Status codes tell what happened** → 2xx good, 4xx your fault, 5xx our fault
4. **Caching makes things faster** → Save responses to reuse
5. **Compression saves bandwidth** → Make files smaller
6. **Keep-alive is efficient** → Reuse connections
7. **Security headers protect users** → Prevent attacks

---

## Summary

Think of HTTP like mailing a package:

- **Request** = Order form you fill out
- **Headers** = Shipping labels with extra info
- **Body** = The actual package contents
- **Status Code** = Delivery confirmation
- **Caching** = Keeping copy at local post office
- **Compression** = Vacuum sealing to save space
- **Keep-Alive** = Keeping delivery truck ready for next package

HTTP is the foundation of the web - every time you visit a website, download an image, or submit a form, HTTP is working behind the scenes!
