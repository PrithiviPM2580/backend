# Validation and Transformation

## What is Validation?

**Validation** = Checking if incoming data meets your rules before using it

Think of it like a security checkpoint at an airport! 🛂 Before data enters your system, you verify it's safe, correct, and in the right format.

---

## Why Validate?

**Security:**

- Prevent SQL injection, XSS attacks
- Stop malicious payloads
- Protect against buffer overflows

**Data Integrity:**

- Ensure database constraints are met
- Maintain consistent data format
- Prevent corrupted records

**User Experience:**

- Give clear error messages
- Catch mistakes early
- Guide users to correct input

**System Stability:**

- Prevent crashes from unexpected data
- Avoid type errors
- Reduce debugging time

---

## Types of Validation

### 1. Syntactic Validation

**What is it?** Checking if the format/structure is correct

**Examples:**

```javascript
// Email format
"user@example.com"     ✅ Valid
"userexample.com"      ❌ Missing @
"user@"                ❌ Incomplete

// Phone number format
"+1-555-123-4567"      ✅ Valid
"555-123-4567"         ✅ Valid
"12345"                ❌ Too short
"abcd-efgh"            ❌ Contains letters

// Date format
"2026-01-20"           ✅ Valid (ISO format)
"01/20/2026"           ✅ Valid (US format)
"20/13/2026"           ❌ Invalid month
"abc"                  ❌ Not a date
```

**How to implement:**

```javascript
// Using regex
const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
if (!emailRegex.test(email)) {
  throw new Error("Invalid email format");
}

// Using built-in validation
const url = new URL(inputUrl); // Throws if invalid URL format

// Using validation libraries
const schema = z.object({
  email: z.string().email(),
  phone: z.string().regex(/^\+?[\d\s-()]+$/),
  url: z.string().url(),
});
```

**Common syntactic validations:**

- Email addresses
- URLs
- Phone numbers
- Credit card numbers
- ZIP/postal codes
- IP addresses
- Date/time formats

---

### 2. Semantic Validation

**What is it?** Checking if the data makes sense in context

**Examples:**

```javascript
// Date ranges
startDate: "2026-01-20"
endDate: "2026-01-15"
❌ End date is before start date!

// Age validation
birthDate: "2025-01-01"
currentDate: "2026-01-20"
age: 1
❌ Too young to register (must be 18+)

// Quantity validation
cartItem: { product: "Phone", quantity: -5 }
❌ Negative quantity doesn't make sense!

// Price validation
product: { name: "Laptop", price: -500 }
❌ Negative price is invalid!

// Business logic
appointment: { doctorId: 123, date: "2026-01-20", time: "14:00" }
❌ Doctor already has appointment at 14:00!
```

**How to implement:**

```javascript
// Date range validation
if (new Date(endDate) < new Date(startDate)) {
  throw new Error("End date must be after start date");
}

// Age calculation and validation
const age = calculateAge(birthDate);
if (age < 18) {
  throw new Error("Must be 18 or older to register");
}

// Business rule validation
const existingAppointment = await Appointment.findOne({
  doctorId,
  date,
  time,
});
if (existingAppointment) {
  throw new Error("Time slot already booked");
}

// Quantity validation
if (quantity <= 0) {
  throw new Error("Quantity must be positive");
}
if (quantity > inventory.available) {
  throw new Error(`Only ${inventory.available} items in stock`);
}
```

**Common semantic validations:**

- Date ranges (start before end)
- Numeric ranges (min/max values)
- Business rules (appointments, inventory)
- Relationships (foreign keys exist)
- Context-dependent rules

---

### 3. Type Validation

**What is it?** Ensuring data is the correct type and structure

**Why it matters in JavaScript/TypeScript:**

JavaScript is loosely typed:

```javascript
"5" + 5 = "55"    // String concatenation (not 10!)
"5" - 5 = 0       // Number subtraction (coercion)
```

**Examples:**

```javascript
// Expected vs Received
Expected: number
Received: "123"        ❌ String, not number

Expected: boolean
Received: "true"       ❌ String, not boolean

Expected: array
Received: { 0: 'a' }   ❌ Object, not array

Expected: Date
Received: "2026-01-20" ❌ String, not Date object
```

**How to implement:**

```javascript
// Manual type checking
if (typeof age !== "number") {
  throw new Error("Age must be a number");
}
if (!Array.isArray(items)) {
  throw new Error("Items must be an array");
}
if (!(date instanceof Date)) {
  throw new Error("Date must be a Date object");
}

// Using Zod
const UserSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
  age: z.number().int().positive(),
  isActive: z.boolean(),
  tags: z.array(z.string()),
  createdAt: z.date(),
  metadata: z.record(z.unknown()), // Object with any keys
});

// TypeScript (compile-time only!)
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}
// Note: TypeScript types disappear at runtime!
// Still need runtime validation!
```

**Nested object validation:**

```javascript
const OrderSchema = z.object({
  orderId: z.string().uuid(),
  customer: z.object({
    id: z.number(),
    name: z.string(),
    email: z.string().email(),
  }),
  items: z.array(
    z.object({
      productId: z.number(),
      quantity: z.number().int().positive(),
      price: z.number().positive(),
    }),
  ),
  total: z.number().positive(),
  status: z.enum(["pending", "paid", "shipped", "delivered"]),
});
```

**Common type validations:**

- Primitive types (string, number, boolean)
- Arrays
- Objects with specific shape
- Enums/literal types
- Nullable/optional fields
- Union types

---

### 4. Data Transformation

**What is it?** Converting data from one format to another

**Why transform?**

- Normalize data for consistency
- Convert types
- Sanitize input
- Format for display
- Adapt to different systems

**Examples:**

```javascript
// Input transformation
Input: "  JOHN DOE  "
Transformed: "john doe"          // Trim and lowercase

Input: "555-123-4567"
Transformed: "5551234567"        // Remove formatting

Input: "$1,234.56"
Transformed: 1234.56             // Parse to number

Input: "2026-01-20T10:30:00Z"
Transformed: Date object          // Parse to Date

// Output transformation
Database: "john_doe"
API Response: "johnDoe"          // snake_case to camelCase

Database: 1234567890000 (timestamp)
API Response: "2026-01-20"       // Format date

Database: 0.15
API Response: "15%"              // Format percentage
```

**How to implement:**

```javascript
// String transformations
const normalize = (str) => str.trim().toLowerCase();
const sanitize = (str) => str.replace(/[<>]/g, ""); // Remove HTML chars
const slugify = (str) => str.toLowerCase().replace(/\s+/g, "-");

// Using Zod transforms
const UserInputSchema = z.object({
  name: z.string().trim().toLowerCase(),
  email: z.string().email().toLowerCase(),
  age: z.string().transform((val) => parseInt(val, 10)), // String to number
  tags: z.string().transform((val) => val.split(",")), // CSV to array
});

// Custom transformers
const phoneTransformer = (phone) => phone.replace(/[\s\-()]/g, "");
const priceTransformer = (price) => Math.round(price * 100) / 100; // 2 decimals
const dateTransformer = (dateStr) => new Date(dateStr);

// Example: Input transformation pipeline
function processUserInput(raw) {
  return {
    name: normalize(raw.name),
    email: raw.email.toLowerCase().trim(),
    phone: phoneTransformer(raw.phone),
    age: parseInt(raw.age, 10),
    interests: raw.interests.split(",").map((s) => s.trim()),
  };
}
```

**Common transformations:**

**String transformations:**

```javascript
// Case conversion
"Hello World" -> "hello world"    (lowercase)
"Hello World" -> "HELLO WORLD"    (uppercase)
"hello_world" -> "helloWorld"     (camelCase)
"helloWorld" -> "hello_world"     (snake_case)

// Trimming/padding
"  hello  " -> "hello"            (trim)
"5" -> "005"                      (padStart)

// Sanitization
"<script>alert('xss')</script>" -> "alert('xss')" (strip HTML)
"user@email.com " -> "user@email.com" (trim)
```

**Number transformations:**

```javascript
"123" -> 123                      (string to number)
123.456 -> 123.46                 (round to 2 decimals)
0.15 -> "15%"                     (format percentage)
1234567 -> "1,234,567"            (add thousands separator)
```

**Date transformations:**

```javascript
"2026-01-20" -> Date object       (parse date)
Date object -> "Jan 20, 2026"     (format date)
Date object -> 1737331200000      (to timestamp)
"America/New_York" -> UTC         (timezone conversion)
```

**Array/Object transformations:**

```javascript
"a,b,c" -> ["a", "b", "c"]        (CSV to array)
["a", "b"] -> "a, b"              (array to string)
{user_name: "john"} -> {userName: "john"} (snake to camel)
```

---

### 5. Complex Validation

**What is it?** Validations that involve multiple fields, external data, or async operations

**Examples:**

#### Cross-field validation

```javascript
// Password confirmation
{
  password: "secret123",
  confirmPassword: "secret456"
}
❌ Passwords don't match!

// Date range consistency
{
  startDate: "2026-01-20",
  endDate: "2026-01-15"
}
❌ End date before start date!

// Conditional requirements
{
  shippingMethod: "express",
  shippingAddress: null
}
❌ Express shipping requires address!
```

**Implementation:**

```javascript
// Zod cross-field validation
const RegistrationSchema = z
  .object({
    password: z.string().min(8),
    confirmPassword: z.string(),
    agreeToTerms: z.boolean(),
  })
  .refine((data) => data.password === data.confirmPassword, {
    message: "Passwords don't match",
    path: ["confirmPassword"],
  })
  .refine((data) => data.agreeToTerms === true, {
    message: "Must agree to terms",
    path: ["agreeToTerms"],
  });

// Custom validator
function validateDateRange(data) {
  if (new Date(data.endDate) < new Date(data.startDate)) {
    throw new Error("End date must be after start date");
  }
}
```

#### Database/external validation

```javascript
// Check if email already exists
const existingUser = await User.findOne({ email });
if (existingUser) {
  throw new Error("Email already registered");
}

// Check if referenced entity exists
const product = await Product.findById(productId);
if (!product) {
  throw new Error("Product not found");
}

// Check if user has permission
const hasPermission = await checkPermission(userId, "delete:posts");
if (!hasPermission) {
  throw new Error("Insufficient permissions");
}
```

#### Async validation with Zod

```javascript
const UserSchema = z.object({
  username: z.string().refine(
    async (username) => {
      const existing = await User.findOne({ username });
      return !existing;
    },
    { message: "Username already taken" },
  ),
  email: z
    .string()
    .email()
    .refine(
      async (email) => {
        const existing = await User.findOne({ email });
        return !existing;
      },
      { message: "Email already registered" },
    ),
});

// Use with parseAsync
const validated = await UserSchema.parseAsync(input);
```

#### Business logic validation

```javascript
// Validate order total matches items
function validateOrderTotal(order) {
  const calculatedTotal = order.items.reduce((sum, item) => {
    return sum + item.price * item.quantity;
  }, 0);

  if (Math.abs(calculatedTotal - order.total) > 0.01) {
    throw new Error("Order total doesn't match items");
  }
}

// Validate inventory availability
async function validateInventory(items) {
  for (const item of items) {
    const product = await Product.findById(item.productId);
    if (product.stock < item.quantity) {
      throw new Error(`Insufficient stock for ${product.name}`);
    }
  }
}

// Validate time slot availability
async function validateAppointment(appointment) {
  const conflicts = await Appointment.find({
    doctorId: appointment.doctorId,
    date: appointment.date,
    time: appointment.time,
  });

  if (conflicts.length > 0) {
    throw new Error("Time slot not available");
  }
}
```

---

## Validation Libraries

### 1. Zod (Recommended)

**Why Zod?**

- TypeScript-first
- Runtime validation
- Type inference
- Great error messages
- Transformations built-in

**Example:**

```javascript
import { z } from "zod";

// Define schema
const UserSchema = z.object({
  username: z.string().min(3).max(20),
  email: z.string().email(),
  age: z.number().int().positive().max(120),
  role: z.enum(["user", "admin", "moderator"]),
  tags: z.array(z.string()).optional(),
  metadata: z.record(z.unknown()).optional(),
});

// Validate
try {
  const user = UserSchema.parse(input);
  // user is typed and validated!
} catch (error) {
  console.log(error.errors);
  // [{ path: ['email'], message: 'Invalid email' }]
}

// Type inference
type User = z.infer<typeof UserSchema>;
// User type is automatically generated!
```

### 2. Joi

**Features:**

- Mature and stable
- Rich validation rules
- Good for Node.js

**Example:**

```javascript
const Joi = require("joi");

const schema = Joi.object({
  username: Joi.string().min(3).max(20).required(),
  email: Joi.string().email().required(),
  age: Joi.number().integer().min(18).max(120),
  role: Joi.string().valid("user", "admin"),
});

const { error, value } = schema.validate(input);
if (error) {
  console.log(error.details);
}
```

### 3. Yup

**Features:**

- Similar to Joi
- Good React integration
- Schema builder API

**Example:**

```javascript
import * as yup from "yup";

const schema = yup.object({
  name: yup.string().required(),
  email: yup.string().email().required(),
  age: yup.number().positive().integer(),
});

await schema.validate(input);
```

### 4. Validator.js

**Features:**

- Lightweight
- String validators only
- No schema building

**Example:**

```javascript
import validator from "validator";

validator.isEmail("user@example.com"); // true
validator.isURL("https://example.com"); // true
validator.isUUID("abc"); // false
```

---

## Best Practices

### 1. Validate Early

Validate at the entry points (controllers, API routes):

```javascript
// ✅ Good: Validate at entry point
app.post("/api/users", async (req, res) => {
  try {
    const validatedData = UserSchema.parse(req.body);
    const user = await createUser(validatedData);
    res.json(user);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// ❌ Bad: No validation
app.post("/api/users", async (req, res) => {
  const user = await createUser(req.body); // Unsafe!
  res.json(user);
});
```

### 2. Separate Validation from Business Logic

```javascript
// ✅ Good: Validation layer separate
const validateUser = (data) => UserSchema.parse(data);
const createUser = (data) => User.create(data);

// In controller
const validatedData = validateUser(req.body);
const user = await createUser(validatedData);

// ❌ Bad: Mixed concerns
const createUser = (data) => {
  if (!data.email) throw new Error("Email required");
  if (data.age < 18) throw new Error("Too young");
  return User.create(data);
};
```

### 3. Use Schemas for Documentation

```javascript
// Schema serves as documentation
const CreatePostSchema = z.object({
  title: z.string().min(1).max(200),
  content: z.string().min(10),
  tags: z.array(z.string()).max(5),
  status: z.enum(["draft", "published"]).default("draft"),
  publishAt: z.date().optional(),
});

// Anyone reading this knows exactly what's required!
```

### 4. Provide Clear Error Messages

```javascript
// ✅ Good: Clear, actionable errors
z.string().min(8, "Password must be at least 8 characters");
z.number().positive("Price must be greater than 0");
z.string().email("Please enter a valid email address");

// ❌ Bad: Generic errors
z.string().min(8); // "String must contain at least 8 character(s)"
```

### 5. Don't Trust Client-Side Validation

```javascript
// Client-side validation = UX improvement
// Server-side validation = Security requirement

// Always validate on the server!
app.post("/api/orders", (req, res) => {
  // Even if frontend validated, validate again here!
  const validated = OrderSchema.parse(req.body);
  // ...
});
```

### 6. Sanitize User Input

```javascript
// Remove dangerous characters
const sanitize = (str) =>
  str
    .trim()
    .replace(/[<>]/g, "") // Remove HTML brackets
    .replace(/javascript:/gi, "") // Remove javascript: protocol
    .slice(0, 1000); // Limit length

// Use libraries
import DOMPurify from "isomorphic-dompurify";
const clean = DOMPurify.sanitize(dirtyHTML);
```

### 7. Handle Validation Errors Consistently

```javascript
// Centralized error handler
app.use((error, req, res, next) => {
  if (error instanceof z.ZodError) {
    return res.status(400).json({
      error: "Validation failed",
      details: error.errors.map((e) => ({
        field: e.path.join("."),
        message: e.message,
      })),
    });
  }
  // Handle other errors
  next(error);
});
```

---

## Complete Example: User Registration

```javascript
import { z } from "zod";
import bcrypt from "bcrypt";

// 1. Define validation schema
const RegisterSchema = z
  .object({
    username: z
      .string()
      .min(3, "Username must be at least 3 characters")
      .max(20, "Username must be at most 20 characters")
      .regex(
        /^[a-zA-Z0-9_]+$/,
        "Username can only contain letters, numbers, and underscores",
      )
      .transform((val) => val.toLowerCase()),

    email: z
      .string()
      .email("Invalid email address")
      .transform((val) => val.toLowerCase()),

    password: z
      .string()
      .min(8, "Password must be at least 8 characters")
      .regex(/[A-Z]/, "Password must contain at least one uppercase letter")
      .regex(/[a-z]/, "Password must contain at least one lowercase letter")
      .regex(/[0-9]/, "Password must contain at least one number"),

    confirmPassword: z.string(),

    age: z
      .number()
      .int("Age must be a whole number")
      .min(18, "Must be at least 18 years old")
      .max(120, "Please enter a valid age"),

    agreeToTerms: z.boolean().refine((val) => val === true, {
      message: "You must agree to the terms and conditions",
    }),
  })
  .refine((data) => data.password === data.confirmPassword, {
    message: "Passwords don't match",
    path: ["confirmPassword"],
  });

// 2. Add async database validations
async function validateUniqueFields(data) {
  const existingUsername = await User.findOne({
    username: data.username,
  });
  if (existingUsername) {
    throw new Error("Username already taken");
  }

  const existingEmail = await User.findOne({ email: data.email });
  if (existingEmail) {
    throw new Error("Email already registered");
  }
}

// 3. Registration endpoint
app.post("/api/register", async (req, res) => {
  try {
    // Syntactic, semantic, type validation + transformation
    const validatedData = RegisterSchema.parse(req.body);

    // Complex validation (database uniqueness)
    await validateUniqueFields(validatedData);

    // Data transformation (hash password)
    const hashedPassword = await bcrypt.hash(validatedData.password, 10);

    // Create user
    const user = await User.create({
      username: validatedData.username,
      email: validatedData.email,
      password: hashedPassword,
      age: validatedData.age,
    });

    // Don't send password back!
    const { password, ...userWithoutPassword } = user.toObject();

    res.status(201).json({
      message: "Registration successful",
      user: userWithoutPassword,
    });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return res.status(400).json({
        error: "Validation failed",
        details: error.errors.map((e) => ({
          field: e.path.join("."),
          message: e.message,
        })),
      });
    }

    res.status(400).json({ error: error.message });
  }
});
```

---

## Summary

**Validation is your first line of defense!**

1. **Syntactic** - Is the format correct? (regex, patterns)
2. **Semantic** - Does it make sense? (ranges, logic)
3. **Type** - Is it the right type? (string, number, array)
4. **Transformation** - Convert to the right format (normalize, parse)
5. **Complex** - Check database, cross-fields, async operations

**Remember:**

- Validate early (at entry points)
- Use schemas (Zod, Joi, Yup)
- Provide clear error messages
- Sanitize user input
- Never trust client-side validation alone
- Transform data consistently
- Handle errors gracefully

**Think of validation like airport security** ✈️

- Check format (passport format) = Syntactic
- Check expiration (valid dates) = Semantic
- Check type (right document) = Type validation
- Convert timezone/currency = Transformation
- Check against database (no-fly list) = Complex validation

Every piece of data coming into your system should pass through validation!
