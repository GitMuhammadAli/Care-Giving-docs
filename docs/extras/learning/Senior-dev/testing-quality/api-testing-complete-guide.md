# 🔌 API Testing - Complete Guide

> A comprehensive guide to API testing - Postman, REST Client, automated API tests, assertions, and building reliable API test suites.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "API testing validates that your APIs function correctly, return proper responses, handle errors gracefully, and meet performance requirements - testing the contract between services without involving the UI."

### The 7 Key Concepts (Remember These!)
```
1. REQUEST/RESPONSE → Test correct HTTP semantics
2. STATUS CODES     → Verify appropriate status codes
3. PAYLOAD          → Validate request/response bodies
4. HEADERS          → Test required headers, auth
5. ERROR HANDLING   → Test error responses, edge cases
6. CHAINED TESTS    → Multi-step test workflows
7. ENVIRONMENT      → Variables for different environments
```

### API Testing Scope
```
┌─────────────────────────────────────────────────────────────────┐
│                    API TESTING SCOPE                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  FUNCTIONAL TESTING                                            │
│  • Correct status codes (200, 201, 400, 404, 500)              │
│  • Response body structure and values                          │
│  • Required fields present                                     │
│  • Data types correct                                          │
│  • Business logic validated                                    │
│                                                                 │
│  ERROR HANDLING                                                │
│  • Invalid input (400 Bad Request)                             │
│  • Not found (404)                                             │
│  • Unauthorized (401) / Forbidden (403)                        │
│  • Server errors (500)                                         │
│  • Error message format                                        │
│                                                                 │
│  SECURITY TESTING                                              │
│  • Authentication required                                     │
│  • Authorization enforced                                      │
│  • Input validation (injection prevention)                     │
│  • Rate limiting                                               │
│                                                                 │
│  PERFORMANCE TESTING                                           │
│  • Response time                                               │
│  • Throughput                                                  │
│  • Concurrent requests                                         │
│                                                                 │
│  CONTRACT TESTING                                              │
│  • Schema validation                                           │
│  • Backward compatibility                                      │
│  • Versioning                                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### HTTP Status Codes to Test
```
┌─────────────────────────────────────────────────────────────────┐
│                  STATUS CODES TO TEST                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  2XX SUCCESS                                                   │
│  • 200 OK - Successful GET, PUT                                │
│  • 201 Created - Successful POST (resource created)            │
│  • 204 No Content - Successful DELETE                          │
│                                                                 │
│  4XX CLIENT ERRORS                                             │
│  • 400 Bad Request - Invalid input, validation error           │
│  • 401 Unauthorized - Missing/invalid authentication           │
│  • 403 Forbidden - Authenticated but not authorized            │
│  • 404 Not Found - Resource doesn't exist                      │
│  • 409 Conflict - Duplicate resource                           │
│  • 422 Unprocessable - Semantic errors                         │
│  • 429 Too Many Requests - Rate limited                        │
│                                                                 │
│  5XX SERVER ERRORS                                             │
│  • 500 Internal Server Error - Unexpected error                │
│  • 502 Bad Gateway - Upstream failure                          │
│  • 503 Service Unavailable - Overloaded/maintenance            │
│  • 504 Gateway Timeout - Upstream timeout                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Contract testing"** | "We validate API contracts in our test suite" |
| **"Schema validation"** | "JSON Schema validation catches response changes" |
| **"Chained requests"** | "Tests use chained requests for workflows" |
| **"Environment variables"** | "Environment variables allow testing across environments" |
| **"Collection runner"** | "Postman collection runner executes full test suites" |
| **"API mocking"** | "We mock third-party APIs in our test environment" |

### Key Numbers to Remember
| Metric | Target | Notes |
|--------|--------|-------|
| Response time | **< 200ms** | P95 for most APIs |
| Test coverage | **All endpoints** | Including errors |
| CI runtime | **< 5 minutes** | Full API suite |
| Error scenarios | **5+ per endpoint** | Validation, auth, not found |

### The "Wow" Statement (Memorize This!)
> "We have comprehensive API tests running in CI using Jest with Supertest. Every endpoint has tests for: happy path, validation errors (400), not found (404), authentication (401), authorization (403), and business rule violations. We use JSON Schema validation to catch contract changes. Tests use factories for consistent test data and transaction rollback for isolation. Environment variables allow running against local, staging, and production. We also run Postman collections for smoke tests and have Newman in CI for regression. Response time assertions ensure performance doesn't degrade. The suite catches 95% of API bugs before they reach production."

---

## 📚 Table of Contents

1. [Postman Testing](#1-postman-testing)
2. [Automated API Tests](#2-automated-api-tests)
3. [Schema Validation](#3-schema-validation)
4. [Test Patterns](#4-test-patterns)
5. [CI Integration](#5-ci-integration)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Postman Testing

```javascript
// ════════════════════════════════════════════════════════════════
// POSTMAN TEST SCRIPTS
// ════════════════════════════════════════════════════════════════

// === Pre-request Script (runs before request) ===
// Set timestamp
pm.environment.set("timestamp", new Date().toISOString());

// Generate random data
pm.environment.set("randomEmail", `test-${Date.now()}@example.com`);

// Get auth token
if (!pm.environment.get("authToken")) {
    pm.sendRequest({
        url: pm.environment.get("baseUrl") + "/auth/login",
        method: "POST",
        header: { "Content-Type": "application/json" },
        body: {
            mode: "raw",
            raw: JSON.stringify({
                email: pm.environment.get("testEmail"),
                password: pm.environment.get("testPassword")
            })
        }
    }, function (err, res) {
        pm.environment.set("authToken", res.json().token);
    });
}

// === Test Scripts (runs after response) ===

// Basic status code test
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Response time test
pm.test("Response time is acceptable", function () {
    pm.expect(pm.response.responseTime).to.be.below(500);
});

// Response body tests
pm.test("Response has correct structure", function () {
    const response = pm.response.json();
    
    pm.expect(response).to.have.property('id');
    pm.expect(response).to.have.property('name');
    pm.expect(response).to.have.property('email');
    pm.expect(response.email).to.be.a('string');
    pm.expect(response.id).to.match(/^[0-9a-f-]{36}$/); // UUID format
});

// Array response tests
pm.test("Returns list of users", function () {
    const response = pm.response.json();
    
    pm.expect(response.users).to.be.an('array');
    pm.expect(response.users.length).to.be.greaterThan(0);
    pm.expect(response.total).to.be.a('number');
    
    // Check first item structure
    const firstUser = response.users[0];
    pm.expect(firstUser).to.have.all.keys('id', 'name', 'email', 'createdAt');
});

// JSON Schema validation
pm.test("Response matches schema", function () {
    const schema = {
        type: "object",
        required: ["id", "name", "email"],
        properties: {
            id: { type: "string", format: "uuid" },
            name: { type: "string", minLength: 1 },
            email: { type: "string", format: "email" },
            createdAt: { type: "string", format: "date-time" }
        }
    };
    
    pm.response.to.have.jsonSchema(schema);
});

// Save values for chained requests
pm.test("Save user ID for next request", function () {
    const response = pm.response.json();
    pm.environment.set("createdUserId", response.id);
});

// Error response tests
pm.test("Returns validation error for invalid input", function () {
    pm.response.to.have.status(400);
    
    const response = pm.response.json();
    pm.expect(response).to.have.property('error');
    pm.expect(response.error).to.include('validation');
});

// Headers tests
pm.test("Response has correct headers", function () {
    pm.response.to.have.header('Content-Type', 'application/json; charset=utf-8');
    pm.response.to.have.header('X-Request-Id');
});
```

```json
// ════════════════════════════════════════════════════════════════
// POSTMAN COLLECTION STRUCTURE
// collection.json
// ════════════════════════════════════════════════════════════════

{
  "info": {
    "name": "User API Tests",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "variable": [
    { "key": "baseUrl", "value": "http://localhost:3000/api" }
  ],
  "item": [
    {
      "name": "Authentication",
      "item": [
        {
          "name": "Login - Success",
          "request": {
            "method": "POST",
            "url": "{{baseUrl}}/auth/login",
            "body": {
              "mode": "raw",
              "raw": "{\"email\":\"{{testEmail}}\",\"password\":\"{{testPassword}}\"}",
              "options": { "raw": { "language": "json" } }
            }
          },
          "event": [
            {
              "listen": "test",
              "script": {
                "exec": [
                  "pm.test('Login successful', () => {",
                  "    pm.response.to.have.status(200);",
                  "    pm.environment.set('authToken', pm.response.json().token);",
                  "});"
                ]
              }
            }
          ]
        },
        {
          "name": "Login - Invalid credentials",
          "request": {
            "method": "POST",
            "url": "{{baseUrl}}/auth/login",
            "body": {
              "mode": "raw",
              "raw": "{\"email\":\"wrong@example.com\",\"password\":\"wrong\"}"
            }
          },
          "event": [
            {
              "listen": "test",
              "script": {
                "exec": [
                  "pm.test('Returns 401', () => pm.response.to.have.status(401));"
                ]
              }
            }
          ]
        }
      ]
    },
    {
      "name": "Users",
      "item": [
        {
          "name": "Create User",
          "request": {
            "method": "POST",
            "url": "{{baseUrl}}/users",
            "header": [
              { "key": "Authorization", "value": "Bearer {{authToken}}" }
            ],
            "body": {
              "mode": "raw",
              "raw": "{\"name\":\"Test User\",\"email\":\"{{$randomEmail}}\"}"
            }
          }
        },
        {
          "name": "Get User",
          "request": {
            "method": "GET",
            "url": "{{baseUrl}}/users/{{createdUserId}}",
            "header": [
              { "key": "Authorization", "value": "Bearer {{authToken}}" }
            ]
          }
        }
      ]
    }
  ]
}
```

---

## 2. Automated API Tests

```typescript
// ════════════════════════════════════════════════════════════════
// SUPERTEST WITH JEST
// ════════════════════════════════════════════════════════════════

import request from 'supertest';
import { app } from '../src/app';
import { prisma } from '../src/db';
import { createUser, generateToken } from './helpers';

describe('User API', () => {
  let authToken: string;

  beforeAll(async () => {
    // Setup: Create test user and get token
    const user = await createUser({ role: 'admin' });
    authToken = generateToken(user);
  });

  beforeEach(async () => {
    // Clean database before each test
    await prisma.user.deleteMany({ where: { email: { contains: 'test-' }}});
  });

  afterAll(async () => {
    await prisma.$disconnect();
  });

  // ═══════════════════════════════════════════════════════════════
  // GET /api/users
  // ═══════════════════════════════════════════════════════════════

  describe('GET /api/users', () => {
    it('returns list of users', async () => {
      // Create test users
      await createUser({ email: 'test-1@example.com' });
      await createUser({ email: 'test-2@example.com' });

      const response = await request(app)
        .get('/api/users')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200)
        .expect('Content-Type', /json/);

      expect(response.body.users).toBeInstanceOf(Array);
      expect(response.body.users.length).toBeGreaterThanOrEqual(2);
      expect(response.body).toHaveProperty('total');
      expect(response.body).toHaveProperty('page');
    });

    it('supports pagination', async () => {
      // Create 25 users
      for (let i = 0; i < 25; i++) {
        await createUser({ email: `test-${i}@example.com` });
      }

      const response = await request(app)
        .get('/api/users?page=2&limit=10')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);

      expect(response.body.users).toHaveLength(10);
      expect(response.body.page).toBe(2);
      expect(response.body.totalPages).toBe(3);
    });

    it('returns 401 without auth token', async () => {
      const response = await request(app)
        .get('/api/users')
        .expect(401);

      expect(response.body.error).toBe('Unauthorized');
    });
  });

  // ═══════════════════════════════════════════════════════════════
  // POST /api/users
  // ═══════════════════════════════════════════════════════════════

  describe('POST /api/users', () => {
    const validUser = {
      name: 'Test User',
      email: 'test-new@example.com',
      password: 'SecurePass123!',
    };

    it('creates user with valid data', async () => {
      const response = await request(app)
        .post('/api/users')
        .set('Authorization', `Bearer ${authToken}`)
        .send(validUser)
        .expect(201);

      expect(response.body).toMatchObject({
        id: expect.any(String),
        name: validUser.name,
        email: validUser.email,
      });
      expect(response.body).not.toHaveProperty('password'); // Not exposed

      // Verify in database
      const dbUser = await prisma.user.findUnique({
        where: { id: response.body.id },
      });
      expect(dbUser).not.toBeNull();
    });

    it('returns 400 for missing required fields', async () => {
      const response = await request(app)
        .post('/api/users')
        .set('Authorization', `Bearer ${authToken}`)
        .send({ name: 'Test' }) // Missing email and password
        .expect(400);

      expect(response.body.error).toContain('validation');
      expect(response.body.details).toContainEqual(
        expect.objectContaining({ field: 'email' })
      );
    });

    it('returns 400 for invalid email format', async () => {
      const response = await request(app)
        .post('/api/users')
        .set('Authorization', `Bearer ${authToken}`)
        .send({ ...validUser, email: 'invalid-email' })
        .expect(400);

      expect(response.body.details).toContainEqual(
        expect.objectContaining({ 
          field: 'email',
          message: expect.stringContaining('valid email')
        })
      );
    });

    it('returns 409 for duplicate email', async () => {
      await createUser({ email: 'duplicate@example.com' });

      const response = await request(app)
        .post('/api/users')
        .set('Authorization', `Bearer ${authToken}`)
        .send({ ...validUser, email: 'duplicate@example.com' })
        .expect(409);

      expect(response.body.error).toContain('already exists');
    });

    it('returns 403 for non-admin users', async () => {
      const regularUser = await createUser({ role: 'user' });
      const regularToken = generateToken(regularUser);

      await request(app)
        .post('/api/users')
        .set('Authorization', `Bearer ${regularToken}`)
        .send(validUser)
        .expect(403);
    });
  });

  // ═══════════════════════════════════════════════════════════════
  // GET /api/users/:id
  // ═══════════════════════════════════════════════════════════════

  describe('GET /api/users/:id', () => {
    it('returns user by id', async () => {
      const user = await createUser({ email: 'get-test@example.com' });

      const response = await request(app)
        .get(`/api/users/${user.id}`)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);

      expect(response.body).toMatchObject({
        id: user.id,
        email: user.email,
      });
    });

    it('returns 404 for non-existent user', async () => {
      const response = await request(app)
        .get('/api/users/non-existent-id')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(404);

      expect(response.body.error).toBe('User not found');
    });
  });

  // ═══════════════════════════════════════════════════════════════
  // PUT /api/users/:id
  // ═══════════════════════════════════════════════════════════════

  describe('PUT /api/users/:id', () => {
    it('updates user', async () => {
      const user = await createUser({ name: 'Original Name' });

      const response = await request(app)
        .put(`/api/users/${user.id}`)
        .set('Authorization', `Bearer ${authToken}`)
        .send({ name: 'Updated Name' })
        .expect(200);

      expect(response.body.name).toBe('Updated Name');
    });

    it('returns 400 for invalid update data', async () => {
      const user = await createUser();

      await request(app)
        .put(`/api/users/${user.id}`)
        .set('Authorization', `Bearer ${authToken}`)
        .send({ email: 'invalid-email' })
        .expect(400);
    });
  });

  // ═══════════════════════════════════════════════════════════════
  // DELETE /api/users/:id
  // ═══════════════════════════════════════════════════════════════

  describe('DELETE /api/users/:id', () => {
    it('deletes user and returns 204', async () => {
      const user = await createUser();

      await request(app)
        .delete(`/api/users/${user.id}`)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(204);

      // Verify deleted
      const dbUser = await prisma.user.findUnique({ where: { id: user.id }});
      expect(dbUser).toBeNull();
    });

    it('returns 404 when deleting non-existent user', async () => {
      await request(app)
        .delete('/api/users/non-existent-id')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(404);
    });
  });
});
```

---

## 3. Schema Validation

```typescript
// ════════════════════════════════════════════════════════════════
// JSON SCHEMA VALIDATION
// ════════════════════════════════════════════════════════════════

import Ajv from 'ajv';
import addFormats from 'ajv-formats';

const ajv = new Ajv({ allErrors: true });
addFormats(ajv);

// Define schemas
const userSchema = {
  type: 'object',
  required: ['id', 'name', 'email', 'createdAt'],
  properties: {
    id: { type: 'string', format: 'uuid' },
    name: { type: 'string', minLength: 1, maxLength: 100 },
    email: { type: 'string', format: 'email' },
    role: { type: 'string', enum: ['user', 'admin', 'moderator'] },
    createdAt: { type: 'string', format: 'date-time' },
    updatedAt: { type: 'string', format: 'date-time' },
  },
  additionalProperties: false,
};

const userListSchema = {
  type: 'object',
  required: ['users', 'total', 'page', 'limit'],
  properties: {
    users: {
      type: 'array',
      items: userSchema,
    },
    total: { type: 'integer', minimum: 0 },
    page: { type: 'integer', minimum: 1 },
    limit: { type: 'integer', minimum: 1, maximum: 100 },
    totalPages: { type: 'integer', minimum: 0 },
  },
};

const errorSchema = {
  type: 'object',
  required: ['error'],
  properties: {
    error: { type: 'string' },
    code: { type: 'string' },
    details: {
      type: 'array',
      items: {
        type: 'object',
        properties: {
          field: { type: 'string' },
          message: { type: 'string' },
        },
      },
    },
  },
};

// Compile validators
const validateUser = ajv.compile(userSchema);
const validateUserList = ajv.compile(userListSchema);
const validateError = ajv.compile(errorSchema);

// Use in tests
describe('API Schema Validation', () => {
  it('GET /users response matches schema', async () => {
    const response = await request(app)
      .get('/api/users')
      .set('Authorization', `Bearer ${authToken}`);

    const isValid = validateUserList(response.body);
    
    if (!isValid) {
      console.error('Schema validation errors:', validateUserList.errors);
    }
    
    expect(isValid).toBe(true);
  });

  it('POST /users response matches user schema', async () => {
    const response = await request(app)
      .post('/api/users')
      .set('Authorization', `Bearer ${authToken}`)
      .send(validUserData);

    expect(validateUser(response.body)).toBe(true);
  });

  it('Error response matches error schema', async () => {
    const response = await request(app)
      .post('/api/users')
      .set('Authorization', `Bearer ${authToken}`)
      .send({ invalid: 'data' });

    expect(validateError(response.body)).toBe(true);
  });
});

// ════════════════════════════════════════════════════════════════
// OPENAPI SCHEMA VALIDATION
// ════════════════════════════════════════════════════════════════

import SwaggerParser from '@apidevtools/swagger-parser';
import { OpenAPIV3 } from 'openapi-types';

describe('OpenAPI Compliance', () => {
  let apiSpec: OpenAPIV3.Document;

  beforeAll(async () => {
    apiSpec = await SwaggerParser.validate('./openapi.yaml') as OpenAPIV3.Document;
  });

  it('response matches OpenAPI spec', async () => {
    const response = await request(app)
      .get('/api/users')
      .set('Authorization', `Bearer ${authToken}`);

    const schema = apiSpec.paths['/api/users']?.get?.responses?.['200']?.content?.['application/json']?.schema;
    
    expect(ajv.validate(schema, response.body)).toBe(true);
  });
});
```

---

## 4. Test Patterns

```typescript
// ════════════════════════════════════════════════════════════════
// API TEST PATTERNS
// ════════════════════════════════════════════════════════════════

// PATTERN 1: Test Helper Functions
// tests/helpers/api.ts

export async function authenticatedRequest(app: Express) {
  const agent = request.agent(app);
  
  // Login and maintain session
  await agent
    .post('/api/auth/login')
    .send({ email: 'test@example.com', password: 'password' });
  
  return agent;
}

export function expectPaginatedResponse(response: Response) {
  expect(response.body).toHaveProperty('data');
  expect(response.body).toHaveProperty('total');
  expect(response.body).toHaveProperty('page');
  expect(response.body).toHaveProperty('limit');
  expect(Array.isArray(response.body.data)).toBe(true);
}

export function expectErrorResponse(response: Response, statusCode: number) {
  expect(response.status).toBe(statusCode);
  expect(response.body).toHaveProperty('error');
}

// PATTERN 2: Data Factories
// tests/factories/user.factory.ts

import { faker } from '@faker-js/faker';

export function buildUser(overrides = {}) {
  return {
    name: faker.person.fullName(),
    email: faker.internet.email(),
    password: 'SecurePass123!',
    ...overrides,
  };
}

export async function createUser(prisma: PrismaClient, overrides = {}) {
  const data = buildUser(overrides);
  return prisma.user.create({ data });
}

// PATTERN 3: Chained Request Tests

describe('Order Workflow', () => {
  let userId: string;
  let productId: string;
  let orderId: string;

  it('1. creates a user', async () => {
    const response = await request(app)
      .post('/api/users')
      .send(buildUser());
    
    userId = response.body.id;
    expect(userId).toBeDefined();
  });

  it('2. creates a product', async () => {
    const response = await request(app)
      .post('/api/products')
      .set('Authorization', `Bearer ${adminToken}`)
      .send({ name: 'Test Product', price: 99.99 });
    
    productId = response.body.id;
    expect(productId).toBeDefined();
  });

  it('3. adds product to cart', async () => {
    await request(app)
      .post(`/api/users/${userId}/cart`)
      .set('Authorization', `Bearer ${userToken}`)
      .send({ productId, quantity: 2 })
      .expect(200);
  });

  it('4. creates order from cart', async () => {
    const response = await request(app)
      .post('/api/orders')
      .set('Authorization', `Bearer ${userToken}`)
      .send({ userId })
      .expect(201);
    
    orderId = response.body.id;
    expect(response.body.total).toBe(199.98);
  });

  it('5. order appears in user orders', async () => {
    const response = await request(app)
      .get(`/api/users/${userId}/orders`)
      .set('Authorization', `Bearer ${userToken}`)
      .expect(200);
    
    expect(response.body.orders).toContainEqual(
      expect.objectContaining({ id: orderId })
    );
  });
});

// PATTERN 4: Test Matrix for Validation

describe('POST /api/users validation', () => {
  const validUser = buildUser();
  
  const validationCases = [
    { field: 'name', value: '', error: 'Name is required' },
    { field: 'name', value: 'a'.repeat(101), error: 'Name too long' },
    { field: 'email', value: '', error: 'Email is required' },
    { field: 'email', value: 'invalid', error: 'Invalid email' },
    { field: 'password', value: '123', error: 'Password too short' },
    { field: 'password', value: 'nodigits', error: 'Password must contain number' },
  ];

  test.each(validationCases)(
    'returns error for invalid $field: $value',
    async ({ field, value, error }) => {
      const response = await request(app)
        .post('/api/users')
        .set('Authorization', `Bearer ${authToken}`)
        .send({ ...validUser, [field]: value })
        .expect(400);

      expect(response.body.details).toContainEqual(
        expect.objectContaining({ field, message: expect.stringContaining(error) })
      );
    }
  );
});

// PATTERN 5: Response Time Assertions

it('responds within acceptable time', async () => {
  const start = Date.now();
  
  await request(app)
    .get('/api/users')
    .set('Authorization', `Bearer ${authToken}`)
    .expect(200);
  
  const duration = Date.now() - start;
  expect(duration).toBeLessThan(500); // 500ms threshold
});
```

---

## 5. CI Integration

```yaml
# ════════════════════════════════════════════════════════════════
# GITHUB ACTIONS - API TESTS
# ════════════════════════════════════════════════════════════════

name: API Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  api-tests:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_DB: test
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run database migrations
        run: npx prisma migrate deploy
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test

      - name: Run API tests
        run: npm run test:api -- --coverage
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test
          JWT_SECRET: test-secret

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info

  postman-tests:
    runs-on: ubuntu-latest
    needs: api-tests
    
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3

      - name: Install Newman
        run: npm install -g newman newman-reporter-htmlextra

      - name: Start server
        run: |
          npm ci
          npm run start:test &
          sleep 10
        env:
          DATABASE_URL: ${{ secrets.TEST_DATABASE_URL }}

      - name: Run Postman collection
        run: |
          newman run ./postman/api-tests.json \
            --environment ./postman/test-env.json \
            --reporters cli,htmlextra \
            --reporter-htmlextra-export ./newman-report.html

      - name: Upload report
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: newman-report
          path: newman-report.html
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# API TESTING PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Only testing happy paths
# Bad
it('creates user', async () => {
  await request(app).post('/users').send(validData).expect(201);
});
# No error case tests!

# Good
# Test: success, validation errors, duplicates, auth, permissions

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Hardcoded test data
# Bad
.send({ email: 'test@example.com' })  # Same email every test
# Tests fail when run in parallel or without cleanup

# Good
.send({ email: `test-${Date.now()}@example.com` })
# Or use factories with unique data

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Tests depend on order
# Bad
# Test 1 creates user
# Test 2 expects user from test 1 to exist
# Running test 2 alone fails

# Good
# Each test creates its own data
# Or use beforeEach to set up required data

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Not testing response structure
# Bad
expect(response.status).toBe(200);
# Doesn't verify response body

# Good
expect(response.body).toMatchObject({
  id: expect.any(String),
  name: 'Expected Name',
});
# Verifies structure and values

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Testing against production
# Bad
const baseUrl = 'https://api.production.com';
# Risk of modifying real data!

# Good
# Always test against local or test environment
# Use environment variables
# Seed and clean test data

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Ignoring response time
# Bad
# Tests pass but API takes 10 seconds
# No performance validation

# Good
expect(response.headers['x-response-time']).toBeLessThan('500ms');
# Or measure and assert duration
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is API testing?"**
> "Testing APIs directly without UI - validating requests/responses, status codes, error handling, authentication, and performance. Faster than E2E, more thorough than unit tests for API behavior. Tests the contract between services."

**Q: "What status codes should you test?"**
> "Success: 200 (OK), 201 (Created), 204 (No Content). Client errors: 400 (Bad Request), 401 (Unauthorized), 403 (Forbidden), 404 (Not Found), 409 (Conflict). Server errors: 500 (Internal Error). Test both happy paths and error scenarios."

**Q: "What tools do you use for API testing?"**
> "Manual/exploratory: Postman, Insomnia, REST Client (VS Code). Automated: Supertest with Jest, axios for Node.js, requests for Python. CI: Newman (Postman CLI). Contract: Pact for consumer-driven contracts."

### Intermediate Questions

**Q: "How do you handle authentication in API tests?"**
> "Get auth token in beforeAll/beforeEach, store in variable. Use test accounts with known credentials. For session-based: use request agent to maintain cookies. For OAuth: mock the OAuth provider or use test tokens. Never test against production credentials."

**Q: "How do you test API validation?"**
> "Test each validation rule separately. Use test matrix for all invalid inputs: missing fields, invalid formats, too long/short, invalid types. Verify 400 status and error message includes field name. Test boundary values (exactly at limit)."

**Q: "How do you handle test data?"**
> "Create data per test using factories. Clean up in afterEach or use transaction rollback. Generate unique values (timestamps, UUIDs) to avoid conflicts. Seed reference data in beforeAll. Never depend on data from previous tests."

### Advanced Questions

**Q: "How do you test API backwards compatibility?"**
> "Contract tests ensure changes don't break consumers. Version APIs (/v1/, /v2/). Test that old clients still work with new API. Schema validation catches unintended changes. Consumer-driven contracts (Pact) verify compatibility across services."

**Q: "How do you test webhooks?"**
> "Create mock endpoint to receive webhooks. Trigger event that causes webhook. Verify: correct endpoint called, payload matches expected, retry logic works, signature validation. Use tools like ngrok for local testing or mock servers."

**Q: "How do you load test APIs?"**
> "Use k6, JMeter, or Artillery for load testing. Test: expected load, stress (find breaking point), spike (sudden traffic). Measure: response time (p50, p95, p99), throughput (RPS), error rate. Run against staging with production-like data."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                  API TESTING CHECKLIST                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  PER ENDPOINT:                                                  │
│  □ Success response (200/201)                                  │
│  □ Validation errors (400)                                     │
│  □ Not found (404)                                             │
│  □ Unauthorized (401)                                          │
│  □ Forbidden (403)                                             │
│  □ Conflict (409) if applicable                                │
│  □ Response schema validation                                  │
│                                                                 │
│  TEST DATA:                                                     │
│  □ Use factories for test data                                 │
│  □ Unique values (no hardcoding)                               │
│  □ Clean up after tests                                        │
│  □ Independent tests (no order dependency)                     │
│                                                                 │
│  ASSERTIONS:                                                    │
│  □ Status code                                                 │
│  □ Response body structure                                     │
│  □ Response body values                                        │
│  □ Headers (Content-Type, etc.)                                │
│  □ Response time                                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

SUPERTEST QUICK REFERENCE:
┌────────────────────────────────────────────────────────────────┐
│ request(app).get('/path')           # GET request              │
│ .set('Authorization', 'Bearer ...')  # Set header              │
│ .query({ page: 1 })                  # Query params            │
│ .send({ data })                      # Request body            │
│ .expect(200)                         # Assert status           │
│ .expect('Content-Type', /json/)      # Assert header           │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

