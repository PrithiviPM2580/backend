# Routing in Backend

## What is Routing?

**Routing** = Deciding what code to run based on the URL that someone visits

Think of it like a post office! 📬 When a letter arrives with an address, the post office knows exactly where to deliver it. Similarly, when a request comes to your server with a URL, routing decides which function should handle it.

---

## Why Do We Need Routing?

Imagine you have a website:

- `/home` → Show home page
- `/about` → Show about page
- `/contact` → Show contact form

Without routing, your server wouldn't know what to show for each URL!

---

## Types of Routes

### 1. **Static Routes**

Routes with fixed, unchanging paths

**Format:** `GET /api/books`

**Example:**

```javascript
// Get all books
app.get("/api/books", (req, res) => {
  res.json({ books: ["Book 1", "Book 2", "Book 3"] });
});

// Get user profile
app.get("/api/profile", (req, res) => {
  res.json({ name: "Alice", age: 25 });
});
```

**When to use:** When the endpoint always does the same thing

- `/api/users` - Get all users
- `/api/products` - Get all products
- `/api/health` - Check server health

---

### 2. **Dynamic Routes (URL Parameters)**

Routes with variable parts that change

**Format:** `GET /api/books/:id`

The `:id` part is a placeholder that can be any value!

**Example:**

```javascript
// Get a specific book by ID
app.get("/api/books/:id", (req, res) => {
  const bookId = req.params.id;
  res.json({ id: bookId, title: "Book Title" });
});

// URLs that match:
// /api/books/1      → id = "1"
// /api/books/42     → id = "42"
// /api/books/abc    → id = "abc"
```

**Real-world examples:**

- `/api/users/:userId` → Get specific user
- `/api/products/:productId` → Get specific product
- `/api/posts/:postId` → Get specific post

**Multiple parameters:**

```javascript
app.get("/api/categories/:category/items/:itemId", (req, res) => {
  const { category, itemId } = req.params;
  // /api/categories/electronics/items/123
  // category = "electronics", itemId = "123"
});
```

---

### 3. **Query Parameters**

Additional information added to the URL with `?`

**Format:** `GET /api/books/search?query=some-value`

**Example:**

```javascript
// Search for books
app.get("/api/books/search", (req, res) => {
  const searchQuery = req.query.query;
  const sortBy = req.query.sort;

  res.json({
    search: searchQuery,
    sort: sortBy,
    results: ["Result 1", "Result 2"],
  });
});

// URLs that work:
// /api/books/search?query=harry
// /api/books/search?query=harry&sort=price
// /api/books/search?query=harry&sort=price&limit=10
```

**Common uses:**

- Filtering: `/api/products?category=electronics`
- Sorting: `/api/products?sort=price&order=asc`
- Pagination: `/api/products?page=2&limit=20`
- Search: `/api/search?q=laptop`

**Difference from dynamic routes:**
| Dynamic Route | Query Parameter |
|--------------|-----------------|
| `/api/books/123` | `/api/books?id=123` |
| Part of the path | Added with `?` |
| Required | Optional |
| `req.params.id` | `req.query.id` |

---

### 4. **Nested Routes**

Routes with multiple levels of resources

**Format:** `GET /api/users/:id/posts/:postId`

**Example:**

```javascript
// Get a specific post from a specific user
app.get("/api/users/:userId/posts/:postId", (req, res) => {
  const { userId, postId } = req.params;
  res.json({
    userId: userId,
    postId: postId,
    message: `Post ${postId} from User ${userId}`,
  });
});

// Example URL:
// /api/users/42/posts/100
// userId = "42", postId = "100"
```

**Real-world examples:**

- `/api/users/:userId/comments/:commentId` → User's specific comment
- `/api/companies/:companyId/employees/:employeeId` → Company's specific employee
- `/api/courses/:courseId/lessons/:lessonId` → Course's specific lesson

---

### 5. **Route Versioning**

Different versions of the same API

**Format:** `GET /api/v1/products`

**Why is this important?**
When you update your API, old apps might break. Versioning lets both old and new versions work!

**Example:**

```javascript
// Version 1 - Old format
app.get("/api/v1/products", (req, res) => {
  res.json({
    products: [{ id: 1, name: "Phone" }],
  });
});

// Version 2 - New format with more info
app.get("/api/v2/products", (req, res) => {
  res.json({
    products: [{ id: 1, name: "Phone", price: 599, inStock: true }],
    timestamp: new Date(),
  });
});
```

**Benefits:**

- Old apps keep working with `/api/v1/products`
- New apps use `/api/v2/products`
- You can phase out old versions gradually

---

### 6. **Catch-All Routes (Wildcard)**

Routes that match everything under a path

**Format:** `GET /api/v3/products/*`

**Example:**

```javascript
// Matches ANY path starting with /api/v3/products/
app.get("/api/v3/products/*", (req, res) => {
  res.json({ message: "Catch-all route matched!" });
});

// These all match:
// /api/v3/products/anything
// /api/v3/products/anything/here
// /api/v3/products/a/b/c/d
```

**Common uses:**

- Handling 404 errors
- Serving static files
- Fallback routes

**404 Example:**

```javascript
// All specific routes first
app.get("/api/users", (req, res) => {
  /* ... */
});
app.get("/api/products", (req, res) => {
  /* ... */
});

// Catch-all at the end for 404
app.get("*", (req, res) => {
  res.status(404).json({ error: "Route not found" });
});
```

---

## HTTP Methods with Routes

Routes can use different HTTP methods:

```javascript
// GET - Retrieve data
app.get("/api/users", (req, res) => {
  /* Get all users */
});

// POST - Create new data
app.post("/api/users", (req, res) => {
  /* Create new user */
});

// PUT - Update existing data (full update)
app.put("/api/users/:id", (req, res) => {
  /* Update user */
});

// PATCH - Update existing data (partial update)
app.patch("/api/users/:id", (req, res) => {
  /* Update user fields */
});

// DELETE - Delete data
app.delete("/api/users/:id", (req, res) => {
  /* Delete user */
});
```

---

## Complete Example: Book Store API

```javascript
const express = require("express");
const app = express();

// 1. Static - Get all books
app.get("/api/books", (req, res) => {
  res.json({ books: ["Book 1", "Book 2"] });
});

// 2. Dynamic - Get specific book
app.get("/api/books/:id", (req, res) => {
  const bookId = req.params.id;
  res.json({ id: bookId, title: "Book Title", price: 20 });
});

// 3. Query Params - Search books
app.get("/api/books/search", (req, res) => {
  const { query, sort, limit } = req.query;
  res.json({
    search: query,
    results: ["Result 1", "Result 2"],
  });
});

// 4. Nested - Get reviews for a book
app.get("/api/books/:bookId/reviews/:reviewId", (req, res) => {
  const { bookId, reviewId } = req.params;
  res.json({
    book: bookId,
    review: reviewId,
    text: "Great book!",
  });
});

// 5. Versioned - Different API versions
app.get("/api/v1/books", (req, res) => {
  res.json({ version: 1, books: [] });
});

app.get("/api/v2/books", (req, res) => {
  res.json({ version: 2, books: [], metadata: {} });
});

// 6. Catch-all - 404 handler
app.get("*", (req, res) => {
  res.status(404).json({ error: "Route not found" });
});
```

---

## Common Mistakes

❌ **Wrong order of routes**

```javascript
// This catches everything!
app.get("/api/*", (req, res) => {
  /* ... */
});

// This will NEVER run
app.get("/api/books", (req, res) => {
  /* ... */
});
```

✅ **Correct order - specific first, general last**

```javascript
app.get("/api/books", (req, res) => {
  /* ... */
});
app.get("/api/*", (req, res) => {
  /* ... */
});
```

---

## Key Points to Remember

1. **Routing maps URLs to code** → Each URL runs specific code
2. **Order matters** → More specific routes should come before general ones
3. **Dynamic routes use `:parameter`** → `/users/:id`
4. **Query params use `?`** → `/search?query=value`
5. **Different HTTP methods = different actions** → GET, POST, PUT, DELETE
6. **Version your API** → Prevents breaking changes

---

## Summary

Think of routing like a restaurant menu:

- **Static routes** → Fixed items: "Burger", "Pizza"
- **Dynamic routes** → "Burger with :topping" (any topping)
- **Query params** → "Burger?size=large&cheese=extra"
- **Nested routes** → "Combo 1 / Item 2" (combo within combo)
- **Versioning** → Menu v1 vs Menu v2
- **Catch-all** → "Everything else goes to chef's special"

The server is the kitchen, and routing is the system that makes sure each order goes to the right chef!
