# 🌐 RESTful Best Practices - Complete Guide

> A comprehensive guide to RESTful API design - resource naming, HTTP methods, status codes, HATEOAS, and Richardson Maturity Model.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "REST (Representational State Transfer) is an architectural style for APIs using HTTP semantics - resources identified by URIs, manipulated through standard HTTP methods, with stateless communication and uniform interface constraints."

### The 7 Key Concepts (Remember These!)
```
1. RESOURCES        → Nouns, not verbs (users, not getUsers)
2. HTTP METHODS     → GET, POST, PUT, PATCH, DELETE
3. STATUS CODES     → Meaningful HTTP response codes
4. STATELESS        → No server-side session state
5. UNIFORM INTERFACE → Consistent, predictable API structure
6. HATEOAS          → Hypermedia links in responses
7. IDEMPOTENCY      → Same request = same result
```

### Richardson Maturity Model
```
┌─────────────────────────────────────────────────────────────────┐
│               RICHARDSON MATURITY MODEL                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  LEVEL 3: Hypermedia Controls (HATEOAS)                        │
│  ────────────────────────────────────────                       │
│  • Links tell client what actions are possible                 │
│  • Self-documenting API                                        │
│  • True REST                                                   │
│  Example: { "user": {...}, "_links": { "orders": "/users/1/orders" }}│
│                                                                 │
│  LEVEL 2: HTTP Verbs                                           │
│  ────────────────────                                           │
│  • Proper use of GET, POST, PUT, DELETE                        │
│  • Meaningful status codes                                     │
│  • Most "RESTful" APIs stop here                               │
│  Example: GET /users, POST /users, DELETE /users/1             │
│                                                                 │
│  LEVEL 1: Resources                                            │
│  ──────────────────                                             │
│  • Individual URIs for resources                               │
│  • But may use POST for everything                             │
│  Example: POST /getUser, POST /createUser                      │
│                                                                 │
│  LEVEL 0: The Swamp of POX                                     │
│  ─────────────────────────                                      │
│  • Single endpoint                                             │
│  • POST everything                                             │
│  • RPC-style                                                   │
│  Example: POST /api with action in body                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### HTTP Methods Reference
```
┌─────────────────────────────────────────────────────────────────┐
│                    HTTP METHODS                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  METHOD  │ CRUD    │ IDEMPOTENT │ SAFE │ BODY │ USE CASE       │
│  ────────┼─────────┼────────────┼──────┼──────┼────────────────│
│  GET     │ Read    │ Yes        │ Yes  │ No   │ Retrieve data  │
│  POST    │ Create  │ No         │ No   │ Yes  │ Create new     │
│  PUT     │ Replace │ Yes        │ No   │ Yes  │ Full update    │
│  PATCH   │ Update  │ No*        │ No   │ Yes  │ Partial update │
│  DELETE  │ Delete  │ Yes        │ No   │ No** │ Remove data    │
│                                                                 │
│  * PATCH can be idempotent depending on implementation         │
│  ** DELETE can have body but typically doesn't                 │
│                                                                 │
│  SAFE: Doesn't modify resources                                │
│  IDEMPOTENT: Multiple identical requests = same result         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Resource-oriented"** | "We design resource-oriented APIs with noun-based URLs" |
| **"Richardson Level 2"** | "Our APIs are Richardson Level 2 with proper HTTP verbs" |
| **"Idempotent"** | "PUT and DELETE are idempotent by design" |
| **"Content negotiation"** | "We support content negotiation via Accept headers" |
| **"HATEOAS"** | "HATEOAS links enable discoverability" |
| **"Stateless"** | "Our API is stateless - each request is self-contained" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| 2xx codes | **Success** | 200, 201, 204 |
| 4xx codes | **Client error** | 400, 401, 403, 404, 409 |
| 5xx codes | **Server error** | 500, 502, 503 |
| URL depth | **< 3 levels** | /users/123/orders not deeper |

### The "Wow" Statement (Memorize This!)
> "We design RESTful APIs at Richardson Level 2+, using proper HTTP semantics. Resources are nouns (users, orders, products) with consistent URL patterns. GET is safe and cacheable, PUT/DELETE are idempotent. We use meaningful status codes: 201 for creation with Location header, 204 for successful deletion, 409 for conflicts, 422 for validation errors. Partial updates use PATCH with JSON Patch or merge patch. We include pagination with Link headers, filtering via query params, and rate limit headers. For complex workflows, we use sub-resources (users/123/orders) or action endpoints sparingly (orders/123/cancel). Our error responses follow RFC 7807 Problem Details format for consistency."

---

## 📚 Table of Contents

1. [Resource Design](#1-resource-design)
2. [URL Structure](#2-url-structure)
3. [HTTP Methods](#3-http-methods)
4. [Status Codes](#4-status-codes)
5. [Request/Response Design](#5-requestresponse-design)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Resource Design

```typescript
// ════════════════════════════════════════════════════════════════
// RESOURCE NAMING CONVENTIONS
// ════════════════════════════════════════════════════════════════

// ✅ GOOD: Nouns, plural, lowercase, hyphenated
GET /users
GET /users/123
GET /users/123/orders
GET /order-items
GET /product-categories

// ❌ BAD: Verbs, singular, camelCase
GET /getUser
GET /user/123
POST /createOrder
GET /orderItems
GET /productCategories

// ════════════════════════════════════════════════════════════════
// RESOURCE RELATIONSHIPS
// ════════════════════════════════════════════════════════════════

// Sub-resources (nested)
GET /users/123/orders           // Orders belonging to user 123
GET /users/123/orders/456       // Order 456 of user 123
POST /users/123/orders          // Create order for user 123

// When to use sub-resources:
// - Strong parent-child relationship
// - Child doesn't exist without parent
// - Usually limited depth (2-3 levels max)

// Alternative: query parameters for filtering
GET /orders?userId=123          // All orders, filtered by user
GET /orders?status=pending      // All orders with status

// ════════════════════════════════════════════════════════════════
// RESOURCE DESIGN PATTERNS
// ════════════════════════════════════════════════════════════════

// Collection resource
interface UsersCollection {
  users: User[];
  total: number;
  page: number;
  limit: number;
  _links: {
    self: string;
    next?: string;
    prev?: string;
  };
}

// Single resource
interface UserResource {
  id: string;
  name: string;
  email: string;
  createdAt: string;
  _links: {
    self: string;
    orders: string;
    profile: string;
  };
}

// Action as sub-resource (for non-CRUD operations)
POST /orders/123/cancel         // Cancel order
POST /users/123/activate        // Activate user
POST /payments/123/refund       // Refund payment
```

---

## 2. URL Structure

```typescript
// ════════════════════════════════════════════════════════════════
// URL BEST PRACTICES
// ════════════════════════════════════════════════════════════════

// Base URL patterns
https://api.example.com/v1/users
https://example.com/api/v1/users

// ✅ GOOD URL patterns
GET  /users                     // List users
GET  /users/123                 // Get user 123
POST /users                     // Create user
PUT  /users/123                 // Replace user 123
PATCH /users/123                // Update user 123
DELETE /users/123               // Delete user 123

// Filtering, sorting, pagination via query params
GET /users?status=active&role=admin
GET /users?sort=createdAt&order=desc
GET /users?page=2&limit=20
GET /users?fields=id,name,email  // Sparse fieldsets

// Search
GET /users?q=john               // Simple search
GET /users/search?name=john     // Search endpoint

// ════════════════════════════════════════════════════════════════
// URL ANTI-PATTERNS
// ════════════════════════════════════════════════════════════════

// ❌ BAD: Verbs in URL
GET  /getUsers
POST /createUser
GET  /deleteUser/123

// ❌ BAD: Too deep nesting
GET /companies/1/departments/2/teams/3/employees/4/tasks/5

// Better: Flatten with query params
GET /tasks/5
GET /tasks?employeeId=4

// ❌ BAD: Inconsistent pluralization
GET /user/123
GET /orders

// ❌ BAD: File extensions
GET /users.json
GET /users.xml

// ✅ Use Accept header for content negotiation
GET /users
Accept: application/json

// ════════════════════════════════════════════════════════════════
// EXPRESS ROUTER EXAMPLE
// ════════════════════════════════════════════════════════════════

import { Router } from 'express';

const router = Router();

// Users resource
router.get('/users', listUsers);
router.post('/users', createUser);
router.get('/users/:id', getUser);
router.put('/users/:id', replaceUser);
router.patch('/users/:id', updateUser);
router.delete('/users/:id', deleteUser);

// Nested resources
router.get('/users/:userId/orders', listUserOrders);
router.post('/users/:userId/orders', createUserOrder);

// Actions (when CRUD doesn't fit)
router.post('/users/:id/activate', activateUser);
router.post('/users/:id/deactivate', deactivateUser);

export default router;
```

---

## 3. HTTP Methods

```typescript
// ════════════════════════════════════════════════════════════════
// HTTP METHODS IN PRACTICE
// ════════════════════════════════════════════════════════════════

// GET - Retrieve resource(s)
// - Safe (no side effects)
// - Idempotent
// - Cacheable
// - No request body

app.get('/users/:id', async (req, res) => {
  const user = await userRepository.findById(req.params.id);
  
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  res.json(user);
});

// POST - Create new resource
// - Not idempotent (multiple calls = multiple resources)
// - Return 201 with Location header
// - Return created resource

app.post('/users', async (req, res) => {
  const user = await userRepository.create(req.body);
  
  res
    .status(201)
    .location(`/users/${user.id}`)
    .json(user);
});

// PUT - Replace entire resource
// - Idempotent
// - Client sends complete resource
// - Create if doesn't exist (optional)

app.put('/users/:id', async (req, res) => {
  const { id } = req.params;
  
  // PUT replaces entire resource
  const user = await userRepository.replace(id, {
    id,
    name: req.body.name,
    email: req.body.email,
    role: req.body.role,
    // All fields required for PUT
  });
  
  res.json(user);
});

// PATCH - Partial update
// - May or may not be idempotent
// - Client sends only changed fields
// - Two formats: JSON Merge Patch or JSON Patch

// JSON Merge Patch (simpler)
app.patch('/users/:id', async (req, res) => {
  const user = await userRepository.update(req.params.id, req.body);
  res.json(user);
});

// Request: PATCH /users/123
// Content-Type: application/merge-patch+json
// { "name": "New Name" }

// JSON Patch (more powerful)
// Content-Type: application/json-patch+json
// [
//   { "op": "replace", "path": "/name", "value": "New Name" },
//   { "op": "add", "path": "/tags/-", "value": "premium" }
// ]

// DELETE - Remove resource
// - Idempotent
// - Return 204 (no content) or 200 with deleted resource

app.delete('/users/:id', async (req, res) => {
  const deleted = await userRepository.delete(req.params.id);
  
  if (!deleted) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  res.status(204).send();
});

// ════════════════════════════════════════════════════════════════
// IDEMPOTENCY EXAMPLE
// ════════════════════════════════════════════════════════════════

// PUT is idempotent - same request = same result
// Request 1: PUT /users/123 { name: "John" }  -> User updated
// Request 2: PUT /users/123 { name: "John" }  -> Same result

// POST is NOT idempotent - same request = different results
// Request 1: POST /users { name: "John" }  -> User 1 created
// Request 2: POST /users { name: "John" }  -> User 2 created (duplicate!)

// Making POST idempotent with Idempotency-Key header
app.post('/payments', async (req, res) => {
  const idempotencyKey = req.headers['idempotency-key'];
  
  // Check if we've seen this key before
  const existing = await idempotencyStore.get(idempotencyKey);
  if (existing) {
    return res.status(200).json(existing);
  }
  
  // Process payment
  const payment = await paymentService.create(req.body);
  
  // Store result for idempotency
  await idempotencyStore.set(idempotencyKey, payment, '24h');
  
  res.status(201).json(payment);
});
```

---

## 4. Status Codes

```typescript
// ════════════════════════════════════════════════════════════════
// STATUS CODES REFERENCE
// ════════════════════════════════════════════════════════════════

const StatusCodes = {
  // 2XX Success
  OK: 200,                    // Successful GET, PUT, PATCH
  CREATED: 201,               // Successful POST, resource created
  ACCEPTED: 202,              // Request accepted, processing async
  NO_CONTENT: 204,            // Successful DELETE, no body

  // 3XX Redirection
  MOVED_PERMANENTLY: 301,     // Resource moved permanently
  FOUND: 302,                 // Temporary redirect
  NOT_MODIFIED: 304,          // Cached response still valid

  // 4XX Client Errors
  BAD_REQUEST: 400,           // Invalid syntax, malformed request
  UNAUTHORIZED: 401,          // Not authenticated
  FORBIDDEN: 403,             // Authenticated but not authorized
  NOT_FOUND: 404,             // Resource doesn't exist
  METHOD_NOT_ALLOWED: 405,    // HTTP method not supported
  CONFLICT: 409,              // Conflict (duplicate, state conflict)
  GONE: 410,                  // Resource permanently deleted
  UNPROCESSABLE_ENTITY: 422,  // Validation errors
  TOO_MANY_REQUESTS: 429,     // Rate limited

  // 5XX Server Errors
  INTERNAL_SERVER_ERROR: 500, // Unexpected server error
  NOT_IMPLEMENTED: 501,       // Feature not implemented
  BAD_GATEWAY: 502,           // Upstream server error
  SERVICE_UNAVAILABLE: 503,   // Server overloaded or maintenance
  GATEWAY_TIMEOUT: 504,       // Upstream timeout
};

// ════════════════════════════════════════════════════════════════
// STATUS CODE USAGE EXAMPLES
// ════════════════════════════════════════════════════════════════

class UserController {
  // GET /users/:id
  async getUser(req: Request, res: Response) {
    const user = await this.userService.findById(req.params.id);
    
    if (!user) {
      return res.status(404).json({
        error: 'Not Found',
        message: 'User not found',
      });
    }
    
    res.status(200).json(user);  // 200 OK
  }

  // POST /users
  async createUser(req: Request, res: Response) {
    // Validation errors
    const errors = await validate(req.body);
    if (errors.length > 0) {
      return res.status(422).json({  // 422 Unprocessable Entity
        error: 'Validation Failed',
        details: errors,
      });
    }

    // Check for duplicate
    const existing = await this.userService.findByEmail(req.body.email);
    if (existing) {
      return res.status(409).json({  // 409 Conflict
        error: 'Conflict',
        message: 'Email already exists',
      });
    }

    const user = await this.userService.create(req.body);
    
    res
      .status(201)  // 201 Created
      .location(`/users/${user.id}`)
      .json(user);
  }

  // DELETE /users/:id
  async deleteUser(req: Request, res: Response) {
    const deleted = await this.userService.delete(req.params.id);
    
    if (!deleted) {
      return res.status(404).json({
        error: 'Not Found',
        message: 'User not found',
      });
    }
    
    res.status(204).send();  // 204 No Content
  }

  // POST /users/:id/verify-email (async operation)
  async verifyEmail(req: Request, res: Response) {
    await this.emailQueue.add('verify', { userId: req.params.id });
    
    res.status(202).json({  // 202 Accepted
      message: 'Verification email will be sent',
      statusUrl: `/users/${req.params.id}/verification-status`,
    });
  }
}
```

---

## 5. Request/Response Design

```typescript
// ════════════════════════════════════════════════════════════════
// REQUEST DESIGN
// ════════════════════════════════════════════════════════════════

// Query parameters for filtering/pagination
// GET /users?status=active&role=admin&page=1&limit=20&sort=name&order=asc

interface ListUsersQuery {
  status?: 'active' | 'inactive';
  role?: string;
  page?: number;
  limit?: number;
  sort?: string;
  order?: 'asc' | 'desc';
  fields?: string;  // Sparse fieldsets: fields=id,name,email
}

// Request body for creation
interface CreateUserRequest {
  name: string;
  email: string;
  password: string;
  role?: string;
}

// Request body for partial update
interface UpdateUserRequest {
  name?: string;
  email?: string;
  role?: string;
}

// ════════════════════════════════════════════════════════════════
// RESPONSE DESIGN
// ════════════════════════════════════════════════════════════════

// Single resource response
interface UserResponse {
  id: string;
  name: string;
  email: string;
  role: string;
  createdAt: string;
  updatedAt: string;
  _links?: {
    self: string;
    orders: string;
  };
}

// Collection response with pagination
interface UsersListResponse {
  data: UserResponse[];
  meta: {
    total: number;
    page: number;
    limit: number;
    totalPages: number;
  };
  links: {
    self: string;
    first: string;
    last: string;
    prev?: string;
    next?: string;
  };
}

// ════════════════════════════════════════════════════════════════
// HATEOAS (Hypermedia)
// ════════════════════════════════════════════════════════════════

// Response with hypermedia links
const userResponse = {
  id: '123',
  name: 'John Doe',
  email: 'john@example.com',
  status: 'active',
  _links: {
    self: { href: '/users/123' },
    orders: { href: '/users/123/orders' },
    deactivate: { href: '/users/123/deactivate', method: 'POST' },
  },
  _embedded: {
    orders: [
      { id: '456', total: 99.99, _links: { self: { href: '/orders/456' } } }
    ],
  },
};

// HAL (Hypertext Application Language) format
const halResponse = {
  _links: {
    self: { href: '/orders' },
    next: { href: '/orders?page=2' },
    find: { href: '/orders{?id}', templated: true },
  },
  _embedded: {
    orders: [
      {
        _links: { self: { href: '/orders/123' } },
        id: '123',
        total: 99.99,
      },
    ],
  },
  total: 100,
  page: 1,
};

// ════════════════════════════════════════════════════════════════
// ERROR RESPONSE (RFC 7807 Problem Details)
// ════════════════════════════════════════════════════════════════

interface ProblemDetails {
  type: string;        // URI identifying error type
  title: string;       // Short summary
  status: number;      // HTTP status code
  detail: string;      // Human-readable explanation
  instance?: string;   // URI of specific occurrence
  errors?: Array<{     // Validation errors
    field: string;
    message: string;
  }>;
}

// Example error response
const errorResponse: ProblemDetails = {
  type: 'https://api.example.com/errors/validation',
  title: 'Validation Failed',
  status: 422,
  detail: 'The request body contains invalid data',
  instance: '/users',
  errors: [
    { field: 'email', message: 'Invalid email format' },
    { field: 'password', message: 'Password must be at least 8 characters' },
  ],
};
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# REST API PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Verbs in URLs
# Bad
GET /getUser/123
POST /createOrder
DELETE /deleteUser/123

# Good
GET /users/123
POST /orders
DELETE /users/123

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Wrong HTTP methods
# Bad
POST /users/123          # Updating with POST
GET /users/delete/123    # Deleting with GET

# Good
PUT /users/123           # Full update
PATCH /users/123         # Partial update
DELETE /users/123        # Delete

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Wrong status codes
# Bad
# Everything returns 200, error in body
{ "status": "error", "code": 404 }

# Good
# Use actual HTTP status codes
HTTP/1.1 404 Not Found
{ "error": "User not found" }

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Inconsistent response format
# Bad
GET /users     -> { users: [...] }
GET /orders    -> { data: [...] }
GET /products  -> [...]

# Good - Consistent envelope
GET /users     -> { data: [...], meta: {...} }
GET /orders    -> { data: [...], meta: {...} }
GET /products  -> { data: [...], meta: {...} }

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Exposing internal IDs
# Bad
GET /users/1  # Auto-increment ID - security/enumeration risk

# Good
GET /users/550e8400-e29b-41d4-a716-446655440000  # UUID
GET /users/user_abc123  # Prefixed ID

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Not using query params for filtering
# Bad
GET /users/active
GET /users/admin
GET /users/active/admin

# Good
GET /users?status=active
GET /users?role=admin
GET /users?status=active&role=admin
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is REST?"**
> "REST is an architectural style for APIs using HTTP semantics. Key principles: Resources identified by URIs, standard HTTP methods (GET, POST, PUT, DELETE), stateless communication, uniform interface. It's not a protocol but a set of constraints that make APIs scalable and maintainable."

**Q: "What's the difference between PUT and PATCH?"**
> "PUT replaces the entire resource - client sends complete object. Idempotent. PATCH updates partially - client sends only changed fields. May not be idempotent depending on implementation. Use PUT for full updates, PATCH for partial."

**Q: "What is idempotency?"**
> "An operation is idempotent if calling it multiple times has the same effect as calling it once. GET, PUT, DELETE are idempotent. POST is not - each call may create a new resource. For POST, use Idempotency-Key header to enable safe retries."

### Intermediate Questions

**Q: "What status codes would you use?"**
> "200 OK for successful GET/PUT/PATCH. 201 Created for POST with Location header. 204 No Content for DELETE. 400 Bad Request for syntax errors. 401 Unauthorized for auth issues. 403 Forbidden for authorization. 404 Not Found. 409 Conflict for duplicates. 422 Unprocessable for validation."

**Q: "How do you design nested resources?"**
> "Use sub-resources for parent-child relationships: /users/123/orders. Keep nesting shallow (2-3 levels). For deep hierarchies, flatten with query params: /orders?userId=123. Consider if child can exist without parent - if yes, maybe don't nest."

**Q: "What is HATEOAS?"**
> "Hypermedia As The Engine Of Application State. Responses include links to related resources and available actions. Client doesn't hardcode URLs but follows links. Example: user response includes link to their orders, profile edit action. Enables discoverable, self-documenting APIs."

### Advanced Questions

**Q: "What is Richardson Maturity Model?"**
> "4 levels of REST maturity. Level 0: Single endpoint, RPC-style. Level 1: Resources with individual URIs. Level 2: Proper HTTP verbs and status codes. Level 3: HATEOAS with hypermedia links. Most APIs are Level 2. Level 3 is true REST but often overkill."

**Q: "REST vs GraphQL - when to use each?"**
> "REST: Simple CRUD, caching important, public APIs. GraphQL: Complex data requirements, multiple clients with different needs, avoiding over-fetching. REST is simpler, better caching. GraphQL is flexible, single endpoint. Choose based on use case, not hype."

**Q: "How do you handle actions that don't fit CRUD?"**
> "Options: 1) Sub-resource verbs: POST /orders/123/cancel. 2) State changes via PATCH: PATCH /orders/123 {status: 'cancelled'}. 3) Treat action as resource: POST /cancellations {orderId: 123}. Prefer state changes when possible, use sub-resource verbs for complex workflows."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                   REST API CHECKLIST                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  URL DESIGN:                                                    │
│  □ Nouns, not verbs                                            │
│  □ Plural resource names                                       │
│  □ Lowercase, hyphenated                                       │
│  □ Max 2-3 nesting levels                                      │
│                                                                 │
│  HTTP METHODS:                                                  │
│  □ GET for reading (safe, idempotent)                          │
│  □ POST for creating (not idempotent)                          │
│  □ PUT for full update (idempotent)                            │
│  □ PATCH for partial update                                    │
│  □ DELETE for removing (idempotent)                            │
│                                                                 │
│  STATUS CODES:                                                  │
│  □ 200 OK, 201 Created, 204 No Content                         │
│  □ 400, 401, 403, 404, 409, 422                                │
│  □ 500, 502, 503                                               │
│                                                                 │
│  RESPONSES:                                                     │
│  □ Consistent format                                           │
│  □ Pagination metadata                                         │
│  □ Error details (RFC 7807)                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

HTTP METHODS QUICK REFERENCE:
┌────────────────────────────────────────────────────────────────┐
│ GET    /resources       - List all                             │
│ GET    /resources/:id   - Get one                              │
│ POST   /resources       - Create new → 201 + Location          │
│ PUT    /resources/:id   - Replace → 200                        │
│ PATCH  /resources/:id   - Update → 200                         │
│ DELETE /resources/:id   - Remove → 204                         │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

