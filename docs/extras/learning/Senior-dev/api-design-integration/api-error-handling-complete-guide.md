# ⚠️ API Error Handling - Complete Guide

> A comprehensive guide to API error handling - error formats, Problem Details RFC, error codes, and building consistent, developer-friendly error responses.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "API error handling is the practice of returning consistent, informative error responses that help clients understand what went wrong, why it happened, and how to fix it - using appropriate HTTP status codes and structured error formats."

### The 7 Key Concepts (Remember These!)
```
1. HTTP STATUS CODE  → 4xx client errors, 5xx server errors
2. ERROR CODE        → Machine-readable identifier (VALIDATION_ERROR)
3. ERROR MESSAGE     → Human-readable description
4. ERROR DETAILS     → Additional context (field errors)
5. RFC 7807          → Problem Details standard format
6. CORRELATION ID    → Trace ID for debugging
7. RETRY-AFTER       → When to retry (rate limits, maintenance)
```

### HTTP Status Code Categories
```
┌─────────────────────────────────────────────────────────────────┐
│                   HTTP STATUS CODES                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  4xx CLIENT ERRORS (Client did something wrong)                │
│  ─────────────────────────────────────────────                  │
│  400 Bad Request      │ Malformed syntax, invalid JSON         │
│  401 Unauthorized     │ Not authenticated                      │
│  403 Forbidden        │ Authenticated but not authorized       │
│  404 Not Found        │ Resource doesn't exist                 │
│  405 Method Not Allowed│ Wrong HTTP method                     │
│  409 Conflict         │ State conflict (duplicate, locked)     │
│  410 Gone             │ Resource permanently deleted           │
│  422 Unprocessable    │ Validation errors                      │
│  429 Too Many Requests│ Rate limited                           │
│                                                                 │
│  5xx SERVER ERRORS (Server did something wrong)                │
│  ─────────────────────────────────────────────                  │
│  500 Internal Server  │ Unexpected server error                │
│  502 Bad Gateway      │ Upstream service error                 │
│  503 Service Unavail. │ Temporarily unavailable                │
│  504 Gateway Timeout  │ Upstream timeout                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### RFC 7807 Problem Details
```
┌─────────────────────────────────────────────────────────────────┐
│              RFC 7807 PROBLEM DETAILS                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  HTTP/1.1 422 Unprocessable Entity                             │
│  Content-Type: application/problem+json                        │
│                                                                 │
│  {                                                             │
│    "type": "https://api.example.com/errors/validation",        │
│    "title": "Validation Failed",                               │
│    "status": 422,                                              │
│    "detail": "The request contains invalid data",              │
│    "instance": "/users/123",                                   │
│    "errors": [                                                 │
│      {                                                         │
│        "field": "email",                                       │
│        "code": "invalid_format",                               │
│        "message": "Invalid email format"                       │
│      },                                                        │
│      {                                                         │
│        "field": "age",                                         │
│        "code": "out_of_range",                                 │
│        "message": "Must be between 18 and 120"                 │
│      }                                                         │
│    ],                                                          │
│    "traceId": "abc123"                                         │
│  }                                                             │
│                                                                 │
│  STANDARD FIELDS:                                              │
│  • type: URI identifying error type (documentation link)       │
│  • title: Short summary (same for all instances of type)       │
│  • status: HTTP status code                                    │
│  • detail: Human-readable explanation                          │
│  • instance: URI of specific occurrence                        │
│                                                                 │
│  EXTENSION FIELDS:                                             │
│  • errors: Array of field-level errors                         │
│  • traceId: Correlation ID for debugging                       │
│  • retryAfter: Seconds to wait before retry                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Problem Details"** | "We follow RFC 7807 Problem Details for errors" |
| **"Correlation ID"** | "Every error includes a correlation ID for tracing" |
| **"Semantic codes"** | "We use semantic error codes, not just status" |
| **"Graceful degradation"** | "503 with Retry-After for graceful degradation" |
| **"Client vs server error"** | "4xx is client's fault, 5xx is server's fault" |
| **"Error envelope"** | "Consistent error envelope across all endpoints" |

### Key Numbers to Remember
| Code | Meaning | Use When |
|------|---------|----------|
| 400 | Bad Request | Syntax error, malformed JSON |
| 401 | Unauthorized | Missing or invalid auth |
| 403 | Forbidden | Valid auth, no permission |
| 404 | Not Found | Resource doesn't exist |
| 422 | Unprocessable | Validation failed |
| 429 | Too Many | Rate limited |
| 500 | Internal Error | Unexpected server error |

### The "Wow" Statement (Memorize This!)
> "We follow RFC 7807 Problem Details for all error responses. Every error has: type (documentation URI), title, status, detail, and instance. Validation errors return 422 with field-level errors array. All errors include traceId for debugging - clients can report it, we can find the exact request in logs. We distinguish 401 (not authenticated) from 403 (authenticated but forbidden). Rate limits return 429 with Retry-After header. We never expose stack traces in production - generic message plus traceId. Error types are documented with examples and solutions. This consistency makes integration easier and debugging faster."

---

## 📚 Table of Contents

1. [Error Response Format](#1-error-response-format)
2. [Status Code Usage](#2-status-code-usage)
3. [Validation Errors](#3-validation-errors)
4. [Error Handling Implementation](#4-error-handling-implementation)
5. [Best Practices](#5-best-practices)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Error Response Format

```typescript
// ════════════════════════════════════════════════════════════════
// RFC 7807 PROBLEM DETAILS INTERFACE
// ════════════════════════════════════════════════════════════════

interface ProblemDetails {
  // Required fields
  type: string;        // URI identifying error type
  title: string;       // Short summary
  status: number;      // HTTP status code
  
  // Optional fields
  detail?: string;     // Human-readable explanation
  instance?: string;   // URI of specific occurrence
  
  // Extension fields (custom)
  code?: string;       // Machine-readable error code
  traceId?: string;    // Correlation ID for debugging
  timestamp?: string;  // When error occurred
  errors?: FieldError[]; // Validation errors
  retryAfter?: number; // Seconds to wait
}

interface FieldError {
  field: string;       // Field path (e.g., "user.email")
  code: string;        // Error code (e.g., "invalid_format")
  message: string;     // Human-readable message
  value?: any;         // Rejected value (careful with sensitive data)
}

// ════════════════════════════════════════════════════════════════
// ERROR TYPE REGISTRY
// ════════════════════════════════════════════════════════════════

const ErrorTypes = {
  VALIDATION: 'https://api.example.com/errors/validation',
  NOT_FOUND: 'https://api.example.com/errors/not-found',
  UNAUTHORIZED: 'https://api.example.com/errors/unauthorized',
  FORBIDDEN: 'https://api.example.com/errors/forbidden',
  CONFLICT: 'https://api.example.com/errors/conflict',
  RATE_LIMIT: 'https://api.example.com/errors/rate-limit',
  INTERNAL: 'https://api.example.com/errors/internal',
  SERVICE_UNAVAILABLE: 'https://api.example.com/errors/service-unavailable',
} as const;

// ════════════════════════════════════════════════════════════════
// ERROR RESPONSE EXAMPLES
// ════════════════════════════════════════════════════════════════

// Validation Error (422)
const validationError: ProblemDetails = {
  type: ErrorTypes.VALIDATION,
  title: 'Validation Failed',
  status: 422,
  detail: 'The request body contains invalid data',
  instance: '/api/users',
  code: 'VALIDATION_ERROR',
  traceId: 'req-abc123',
  errors: [
    {
      field: 'email',
      code: 'invalid_format',
      message: 'Must be a valid email address',
    },
    {
      field: 'password',
      code: 'too_short',
      message: 'Must be at least 8 characters',
    },
  ],
};

// Not Found Error (404)
const notFoundError: ProblemDetails = {
  type: ErrorTypes.NOT_FOUND,
  title: 'Resource Not Found',
  status: 404,
  detail: 'User with ID usr_123 was not found',
  instance: '/api/users/usr_123',
  code: 'USER_NOT_FOUND',
  traceId: 'req-def456',
};

// Conflict Error (409)
const conflictError: ProblemDetails = {
  type: ErrorTypes.CONFLICT,
  title: 'Conflict',
  status: 409,
  detail: 'A user with this email already exists',
  instance: '/api/users',
  code: 'DUPLICATE_EMAIL',
  traceId: 'req-ghi789',
};

// Rate Limit Error (429)
const rateLimitError: ProblemDetails = {
  type: ErrorTypes.RATE_LIMIT,
  title: 'Too Many Requests',
  status: 429,
  detail: 'Rate limit exceeded. Please wait before retrying.',
  instance: '/api/users',
  code: 'RATE_LIMIT_EXCEEDED',
  traceId: 'req-jkl012',
  retryAfter: 60,
};

// Internal Error (500)
const internalError: ProblemDetails = {
  type: ErrorTypes.INTERNAL,
  title: 'Internal Server Error',
  status: 500,
  detail: 'An unexpected error occurred. Please try again later.',
  instance: '/api/users',
  code: 'INTERNAL_ERROR',
  traceId: 'req-mno345',  // Client can report this
};
```

---

## 2. Status Code Usage

```typescript
// ════════════════════════════════════════════════════════════════
// STATUS CODE DECISION TREE
// ════════════════════════════════════════════════════════════════

/*
 * Is the request malformed? (bad JSON, wrong content-type)
 * └── Yes → 400 Bad Request
 * 
 * Is authentication missing or invalid?
 * └── Yes → 401 Unauthorized
 * 
 * Is the user authenticated but not allowed?
 * └── Yes → 403 Forbidden
 * 
 * Does the resource exist?
 * └── No → 404 Not Found
 * 
 * Is the HTTP method allowed?
 * └── No → 405 Method Not Allowed
 * 
 * Is there a state conflict? (duplicate, locked, etc.)
 * └── Yes → 409 Conflict
 * 
 * Is the request body valid but logically wrong?
 * └── Yes → 422 Unprocessable Entity
 * 
 * Is the client rate limited?
 * └── Yes → 429 Too Many Requests
 * 
 * Did something unexpected happen on the server?
 * └── Yes → 500 Internal Server Error
 * 
 * Is an upstream service down?
 * └── Yes → 502 Bad Gateway or 503 Service Unavailable
 */

// ════════════════════════════════════════════════════════════════
// STATUS CODE USAGE EXAMPLES
// ════════════════════════════════════════════════════════════════

class UserController {
  // 200 OK - Successful GET
  async getUser(req: Request, res: Response) {
    const user = await userService.findById(req.params.id);
    if (!user) {
      throw new NotFoundError('User', req.params.id);
    }
    res.json(user);
  }

  // 201 Created - Successful POST
  async createUser(req: Request, res: Response) {
    const user = await userService.create(req.body);
    res.status(201)
       .location(`/users/${user.id}`)
       .json(user);
  }

  // 204 No Content - Successful DELETE
  async deleteUser(req: Request, res: Response) {
    const deleted = await userService.delete(req.params.id);
    if (!deleted) {
      throw new NotFoundError('User', req.params.id);
    }
    res.status(204).send();
  }

  // 400 Bad Request - Malformed request
  async badRequestExample(req: Request, res: Response) {
    // Middleware handles JSON parse errors with 400
    // Manual example:
    if (!req.is('application/json')) {
      throw new BadRequestError('Content-Type must be application/json');
    }
  }

  // 401 Unauthorized - Not authenticated
  async authRequired(req: Request, res: Response) {
    if (!req.headers.authorization) {
      throw new UnauthorizedError('Authentication required');
    }
  }

  // 403 Forbidden - Not authorized
  async adminOnly(req: Request, res: Response) {
    if (req.user.role !== 'admin') {
      throw new ForbiddenError('Admin access required');
    }
  }

  // 409 Conflict - Duplicate
  async createWithDuplicateCheck(req: Request, res: Response) {
    const existing = await userService.findByEmail(req.body.email);
    if (existing) {
      throw new ConflictError('A user with this email already exists');
    }
    // ... create user
  }

  // 422 Unprocessable Entity - Validation error
  async validateInput(req: Request, res: Response) {
    const errors = validate(req.body);
    if (errors.length > 0) {
      throw new ValidationError(errors);
    }
  }
}
```

---

## 3. Validation Errors

```typescript
// ════════════════════════════════════════════════════════════════
// VALIDATION ERROR HANDLING
// ════════════════════════════════════════════════════════════════

import Joi from 'joi';
import { z } from 'zod';

// ════════════════════════════════════════════════════════════════
// JOI VALIDATION
// ════════════════════════════════════════════════════════════════

const userSchema = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string().min(8).required(),
  name: Joi.string().min(1).max(100).required(),
  age: Joi.number().integer().min(18).max(120).optional(),
});

function validateWithJoi(schema: Joi.Schema, data: any): FieldError[] {
  const { error } = schema.validate(data, { abortEarly: false });
  
  if (!error) return [];

  return error.details.map(detail => ({
    field: detail.path.join('.'),
    code: detail.type,           // e.g., "string.email"
    message: detail.message,
    value: detail.context?.value,
  }));
}

// ════════════════════════════════════════════════════════════════
// ZOD VALIDATION
// ════════════════════════════════════════════════════════════════

const UserSchema = z.object({
  email: z.string().email('Invalid email format'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
  name: z.string().min(1).max(100),
  age: z.number().int().min(18).max(120).optional(),
});

function validateWithZod<T>(schema: z.Schema<T>, data: unknown): FieldError[] {
  const result = schema.safeParse(data);
  
  if (result.success) return [];

  return result.error.errors.map(err => ({
    field: err.path.join('.'),
    code: err.code,              // e.g., "invalid_string"
    message: err.message,
  }));
}

// ════════════════════════════════════════════════════════════════
// VALIDATION MIDDLEWARE
// ════════════════════════════════════════════════════════════════

function validate(schema: z.Schema) {
  return async (req: Request, res: Response, next: NextFunction) => {
    const errors = validateWithZod(schema, req.body);
    
    if (errors.length > 0) {
      return res.status(422).json({
        type: 'https://api.example.com/errors/validation',
        title: 'Validation Failed',
        status: 422,
        detail: 'The request body contains invalid data',
        instance: req.path,
        traceId: req.id,
        errors,
      });
    }
    
    next();
  };
}

// Usage
app.post('/users', validate(UserSchema), userController.create);

// ════════════════════════════════════════════════════════════════
// NESTED VALIDATION ERRORS
// ════════════════════════════════════════════════════════════════

// For nested objects:
const OrderSchema = z.object({
  customer: z.object({
    email: z.string().email(),
    address: z.object({
      street: z.string().min(1),
      city: z.string().min(1),
      zip: z.string().regex(/^\d{5}$/),
    }),
  }),
  items: z.array(z.object({
    productId: z.string(),
    quantity: z.number().int().positive(),
  })).min(1),
});

// Error response would have field paths like:
// "customer.address.zip", "items.0.quantity"
{
  "errors": [
    {
      "field": "customer.address.zip",
      "code": "invalid_string",
      "message": "Invalid zip code format"
    },
    {
      "field": "items.0.quantity",
      "code": "too_small",
      "message": "Must be positive"
    }
  ]
}
```

---

## 4. Error Handling Implementation

```typescript
// ════════════════════════════════════════════════════════════════
// CUSTOM ERROR CLASSES
// ════════════════════════════════════════════════════════════════

abstract class AppError extends Error {
  abstract readonly status: number;
  abstract readonly type: string;
  abstract readonly title: string;
  readonly code?: string;
  readonly details?: any;

  constructor(message: string, code?: string, details?: any) {
    super(message);
    this.code = code;
    this.details = details;
    Error.captureStackTrace(this, this.constructor);
  }

  toProblemDetails(instance: string, traceId: string): ProblemDetails {
    return {
      type: this.type,
      title: this.title,
      status: this.status,
      detail: this.message,
      instance,
      code: this.code,
      traceId,
      ...this.details,
    };
  }
}

class BadRequestError extends AppError {
  readonly status = 400;
  readonly type = 'https://api.example.com/errors/bad-request';
  readonly title = 'Bad Request';
}

class UnauthorizedError extends AppError {
  readonly status = 401;
  readonly type = 'https://api.example.com/errors/unauthorized';
  readonly title = 'Unauthorized';
}

class ForbiddenError extends AppError {
  readonly status = 403;
  readonly type = 'https://api.example.com/errors/forbidden';
  readonly title = 'Forbidden';
}

class NotFoundError extends AppError {
  readonly status = 404;
  readonly type = 'https://api.example.com/errors/not-found';
  readonly title = 'Not Found';

  constructor(resource: string, id: string) {
    super(`${resource} with ID ${id} was not found`, `${resource.toUpperCase()}_NOT_FOUND`);
  }
}

class ConflictError extends AppError {
  readonly status = 409;
  readonly type = 'https://api.example.com/errors/conflict';
  readonly title = 'Conflict';
}

class ValidationError extends AppError {
  readonly status = 422;
  readonly type = 'https://api.example.com/errors/validation';
  readonly title = 'Validation Failed';

  constructor(errors: FieldError[]) {
    super('The request body contains invalid data', 'VALIDATION_ERROR', { errors });
  }
}

class RateLimitError extends AppError {
  readonly status = 429;
  readonly type = 'https://api.example.com/errors/rate-limit';
  readonly title = 'Too Many Requests';

  constructor(retryAfter: number) {
    super('Rate limit exceeded', 'RATE_LIMIT_EXCEEDED', { retryAfter });
  }
}

// ════════════════════════════════════════════════════════════════
// GLOBAL ERROR HANDLER
// ════════════════════════════════════════════════════════════════

function errorHandler(
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction
) {
  // Generate trace ID if not present
  const traceId = req.id || crypto.randomUUID();

  // Log error
  console.error({
    traceId,
    error: err.message,
    stack: err.stack,
    path: req.path,
    method: req.method,
  });

  // Handle known errors
  if (err instanceof AppError) {
    const problem = err.toProblemDetails(req.path, traceId);
    
    // Set Retry-After header for rate limits
    if (err instanceof RateLimitError) {
      res.setHeader('Retry-After', err.details.retryAfter);
    }
    
    return res
      .status(err.status)
      .contentType('application/problem+json')
      .json(problem);
  }

  // Handle Joi validation errors
  if (err.name === 'ValidationError' && err.isJoi) {
    const errors = err.details.map(d => ({
      field: d.path.join('.'),
      code: d.type,
      message: d.message,
    }));
    
    return res
      .status(422)
      .contentType('application/problem+json')
      .json({
        type: 'https://api.example.com/errors/validation',
        title: 'Validation Failed',
        status: 422,
        detail: 'The request body contains invalid data',
        instance: req.path,
        traceId,
        errors,
      });
  }

  // Handle JWT errors
  if (err.name === 'JsonWebTokenError' || err.name === 'TokenExpiredError') {
    return res
      .status(401)
      .contentType('application/problem+json')
      .json({
        type: 'https://api.example.com/errors/unauthorized',
        title: 'Unauthorized',
        status: 401,
        detail: err.name === 'TokenExpiredError' ? 'Token has expired' : 'Invalid token',
        instance: req.path,
        traceId,
      });
  }

  // Handle unknown errors (500)
  // Don't expose internal details in production
  const isProduction = process.env.NODE_ENV === 'production';
  
  return res
    .status(500)
    .contentType('application/problem+json')
    .json({
      type: 'https://api.example.com/errors/internal',
      title: 'Internal Server Error',
      status: 500,
      detail: isProduction 
        ? 'An unexpected error occurred. Please try again later.'
        : err.message,
      instance: req.path,
      traceId,
      // Stack trace only in development
      ...(isProduction ? {} : { stack: err.stack }),
    });
}

// Apply to Express app
app.use(errorHandler);
```

---

## 5. Best Practices

```yaml
# ════════════════════════════════════════════════════════════════
# ERROR HANDLING BEST PRACTICES
# ════════════════════════════════════════════════════════════════

response_format:
  standard: RFC 7807 Problem Details
  content_type: application/problem+json
  
  always_include:
    - type: URI for error documentation
    - title: Short, consistent summary
    - status: HTTP status code
    - detail: Specific explanation
    - traceId: Correlation ID
    
  include_when_applicable:
    - instance: Request path
    - errors: Field-level validation errors
    - retryAfter: Rate limit recovery time

status_codes:
  client_errors:
    400: Malformed request (bad JSON, wrong content-type)
    401: Missing or invalid authentication
    403: Authenticated but not authorized
    404: Resource not found
    405: HTTP method not allowed
    409: State conflict (duplicate, locked)
    422: Validation errors
    429: Rate limited
    
  server_errors:
    500: Unexpected error (catch-all)
    502: Upstream service returned error
    503: Service temporarily unavailable
    504: Upstream service timeout

security:
  never_expose:
    - Stack traces in production
    - Internal implementation details
    - Database query errors
    - File paths
    
  always_include:
    - Trace ID (for support)
    - Generic message (user-friendly)

documentation:
  for_each_error_type:
    - Description
    - When it occurs
    - Example response
    - How to fix

logging:
  log_all_errors:
    - Trace ID
    - HTTP status
    - Error message
    - Stack trace
    - Request context
    
  alerting:
    - High 5xx rate
    - Unusual 4xx patterns
    - Specific error codes
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# ERROR HANDLING PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Exposing stack traces
# Bad
{
  "error": "Cannot read property 'id' of undefined",
  "stack": "TypeError: Cannot read property 'id'...\n    at UserService.find (/app/src/...)"
}

# Good
{
  "type": "https://api.example.com/errors/internal",
  "title": "Internal Server Error",
  "detail": "An unexpected error occurred",
  "traceId": "abc123"
}

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Generic 500 for everything
# Bad
try {
  // code
} catch (error) {
  res.status(500).json({ error: "Something went wrong" });
}

# Good
# Distinguish between different error types
# Return appropriate status codes

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Inconsistent error format
# Bad
GET /users/1   -> { "error": "Not found" }
POST /users    -> { "message": "Validation failed", "errors": [...] }
DELETE /users/1 -> { "success": false, "reason": "forbidden" }

# Good
# Consistent RFC 7807 format for all errors

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: 200 with error in body
# Bad
HTTP 200 OK
{ "success": false, "error": "User not found" }

# Good
HTTP 404 Not Found
{ "type": "...", "title": "Not Found", "status": 404, ... }

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Wrong status code
# Bad
401 for "You don't have permission" (should be 403)
400 for validation errors (should be 422)
200 for "resource created" (should be 201)

# Good
401: Not authenticated
403: Not authorized
422: Validation failed

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: No trace ID
# Bad
# User reports error, support can't find it in logs

# Good
# Every response includes traceId
# Same ID in logs for correlation
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What's the difference between 400, 401, 403, and 404?"**
> "400: Malformed request (bad JSON syntax). 401: Not authenticated (no token or invalid token). 403: Authenticated but not authorized (valid token, no permission). 404: Resource doesn't exist. Each tells client specifically what's wrong."

**Q: "What is RFC 7807?"**
> "RFC 7807 'Problem Details for HTTP APIs' defines a standard error format. Fields: type (URI identifying error), title (short summary), status (HTTP code), detail (specific explanation), instance (request URI). Enables consistent, machine-readable errors."

**Q: "Why use 422 instead of 400 for validation errors?"**
> "400 means malformed request (can't be parsed). 422 means request is well-formed but semantically invalid (parsed but validation failed). 422 is more specific - client knows syntax is fine, data is wrong. Both are valid choices, consistency matters."

### Intermediate Questions

**Q: "How do you handle errors in production vs development?"**
> "Production: Generic messages, no stack traces, no internal details. Always include traceId for debugging. Development: Include stack trace, detailed messages, maybe query that failed. Log everything in both, but expose less in production."

**Q: "How do you document API errors?"**
> "Error catalog with: type URI, description, when it occurs, example response, how to fix. OpenAPI spec includes error schemas. Each error type is a documentable URI that clients can look up. Makes integration easier."

**Q: "How do you handle validation errors from nested objects?"**
> "Use dot notation for field paths: 'user.address.zip', 'items.0.quantity'. Return array of field errors. Each has: field (path), code (machine-readable), message (human-readable). Clients can map errors to form fields."

### Advanced Questions

**Q: "How do you correlate errors across microservices?"**
> "Pass traceId in headers (e.g., X-Request-ID) across services. Each service logs with same traceId. Error response includes traceId. User reports ID, support can trace entire request flow. Use distributed tracing (OpenTelemetry) for visualization."

**Q: "How do you handle partial failures in batch operations?"**
> "Return 207 Multi-Status. Response body contains per-item results with individual status codes. Summary shows total/succeeded/failed. Each failed item has its own error details. Client can identify exactly what failed and why."

**Q: "How do you prevent error messages from leaking sensitive info?"**
> "Never include: passwords, tokens, PII, SQL queries, stack traces, file paths. Use error codes instead of dynamic messages where possible. Review error messages in code review. Test error responses don't leak info. Log full details internally."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                ERROR HANDLING CHECKLIST                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  FORMAT (RFC 7807):                                             │
│  □ type: URI for documentation                                 │
│  □ title: Short summary                                        │
│  □ status: HTTP status code                                    │
│  □ detail: Specific explanation                                │
│  □ traceId: For debugging                                      │
│                                                                 │
│  STATUS CODES:                                                  │
│  □ 400: Malformed request                                      │
│  □ 401: Not authenticated                                      │
│  □ 403: Not authorized                                         │
│  □ 404: Not found                                              │
│  □ 422: Validation failed                                      │
│  □ 429: Rate limited                                           │
│  □ 500: Server error                                           │
│                                                                 │
│  SECURITY:                                                      │
│  □ No stack traces in production                               │
│  □ No internal details                                         │
│  □ Generic messages + traceId                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

RFC 7807 EXAMPLE:
┌────────────────────────────────────────────────────────────────┐
│ {                                                              │
│   "type": "https://api.example.com/errors/validation",        │
│   "title": "Validation Failed",                                │
│   "status": 422,                                               │
│   "detail": "The request contains invalid data",              │
│   "instance": "/api/users",                                    │
│   "traceId": "abc123",                                         │
│   "errors": [{ "field": "email", "message": "Invalid" }]      │
│ }                                                              │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

