# REST API Design Concepts

> Understanding API design principles for CareCircle.

---

## 1. What Is REST?

### Plain English Explanation

REST (Representational State Transfer) is an **architectural style for designing APIs** that uses HTTP methods and URLs to represent resources.

Think of it like a **library catalog system**:
- Resources = Books (users, medications, appointments)
- URLs = Shelf locations (/users/123, /medications/456)
- HTTP methods = Actions (GET = read, POST = add, DELETE = remove)

### The Core Problem REST Solves

```
WITHOUT REST:
─────────────
/getUserById?id=123
/createNewUser
/deleteUserFromSystem?userId=123
/updateUserInfo
→ Inconsistent, hard to remember, no standards

WITH REST:
──────────
GET    /users/123      → Read user
POST   /users          → Create user
DELETE /users/123      → Delete user
PUT    /users/123      → Update user
→ Predictable, consistent, self-documenting
```

---

## 2. Core Concepts & Terminology

### HTTP Methods

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         HTTP METHODS                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  GET     Read data (safe, idempotent)                                       │
│          GET /medications → List all                                        │
│          GET /medications/123 → Get one                                     │
│                                                                              │
│  POST    Create new resource (not idempotent)                               │
│          POST /medications → Create medication                              │
│                                                                              │
│  PUT     Replace entire resource (idempotent)                               │
│          PUT /medications/123 → Replace medication                          │
│                                                                              │
│  PATCH   Partial update (idempotent)                                        │
│          PATCH /medications/123 → Update some fields                        │
│                                                                              │
│  DELETE  Remove resource (idempotent)                                       │
│          DELETE /medications/123 → Delete medication                        │
│                                                                              │
│  Idempotent = Same request multiple times = Same result                     │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Status Codes

| Code | Meaning | When to Use |
|------|---------|-------------|
| **200** | OK | Successful GET, PUT, PATCH |
| **201** | Created | Successful POST |
| **204** | No Content | Successful DELETE |
| **400** | Bad Request | Invalid input |
| **401** | Unauthorized | Not authenticated |
| **403** | Forbidden | Not authorized |
| **404** | Not Found | Resource doesn't exist |
| **409** | Conflict | Duplicate, constraint violation |
| **422** | Unprocessable | Validation failed |
| **500** | Server Error | Something broke |

---

## 3. CareCircle API Conventions

### URL Structure

```
/api/v1/families/{familyId}/care-recipients/{careRecipientId}/medications

  │   │     │          │              │                │
  │   │     │          │              │                └── Resource (plural)
  │   │     │          │              └── Nested resource
  │   │     │          └── Resource ID (UUID)
  │   │     └── Resource (plural)
  │   └── Version
  └── API prefix
```

### Resource Naming

```
✅ GOOD: Plural nouns, lowercase, hyphens
/medications
/care-recipients
/caregiver-shifts

❌ BAD: Verbs, singular, underscores, camelCase
/getMedication
/medication
/care_recipients
/caregiverShifts
```

---

## 4. When to Use Each Method ✅

### GET - Safe & Idempotent
```
Use for: Reading data, searching, filtering
Never: Modify state

GET /medications                    # List all
GET /medications?status=active      # Filter
GET /medications/123                # Get one
GET /medications/123/logs           # Nested resource
```

### POST - Create
```
Use for: Creating new resources
Returns: Created resource with ID

POST /medications
Body: { name: "Aspirin", dosage: "100mg" }
Response: 201 Created + { id: "abc-123", name: "Aspirin", ... }
```

### PUT - Full Replace
```
Use for: Replacing entire resource
Requires: All fields (replaces everything)

PUT /medications/123
Body: { name: "Aspirin", dosage: "200mg", frequency: "daily" }
```

### PATCH - Partial Update
```
Use for: Updating specific fields
Requires: Only changed fields

PATCH /medications/123
Body: { dosage: "200mg" }  # Only update dosage
```

### DELETE - Remove
```
Use for: Deleting resources
Returns: 204 No Content (or 200 with confirmation)

DELETE /medications/123
```

---

## 5. When to AVOID Patterns ❌

### DON'T Use Verbs in URLs

```
❌ BAD: Action in URL
POST /medications/123/delete
GET /medications/createNew

✅ GOOD: Method indicates action
DELETE /medications/123
POST /medications
```

### DON'T Return Wrong Status Codes

```
❌ BAD: Always 200
{ "status": 200, "error": "User not found" }

✅ GOOD: Proper status code
404 Not Found
{ "message": "User not found" }
```

### DON'T Ignore Pagination

```
❌ BAD: Return all records
GET /medications → Returns 10,000 items

✅ GOOD: Paginate
GET /medications?page=1&limit=20
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "totalPages": 8
  }
}
```

---

## 6. Best Practices

### Request/Response Format

```json
// Request
POST /medications
Content-Type: application/json
{
  "name": "Aspirin",
  "dosage": "100mg",
  "careRecipientId": "uuid-123"
}

// Success Response
201 Created
{
  "id": "uuid-456",
  "name": "Aspirin",
  "dosage": "100mg",
  "careRecipientId": "uuid-123",
  "createdAt": "2026-01-30T10:00:00Z"
}

// Error Response
400 Bad Request
{
  "statusCode": 400,
  "message": ["dosage must not be empty"],
  "error": "Bad Request"
}
```

### Filtering, Sorting, Pagination

```
# Filtering
GET /medications?status=active&careRecipientId=123

# Sorting
GET /medications?sort=name:asc,createdAt:desc

# Pagination
GET /medications?page=2&limit=20

# Combined
GET /medications?status=active&sort=name:asc&page=1&limit=20
```

### Nested Resources

```
# Shallow nesting (preferred)
GET /families/123/medications

# Deep nesting (use sparingly)
GET /families/123/care-recipients/456/medications/789/logs
→ Consider: GET /medication-logs?medicationId=789
```

---

## 7. Common Mistakes & How to Avoid Them

### Mistake 1: Inconsistent Naming

```
❌ Mixed styles:
/medications
/CareRecipients
/caregiver_shifts

✅ Consistent:
/medications
/care-recipients
/caregiver-shifts
```

### Mistake 2: Missing Versioning

```
❌ No version:
/api/medications

✅ Versioned:
/api/v1/medications
```

### Mistake 3: Leaking Internal Details

```
❌ Exposing database IDs:
{ "id": 12345 }  # Sequential, predictable

✅ Using UUIDs:
{ "id": "550e8400-e29b-41d4-a716-446655440000" }
```

---

## 8. Error Handling

### Standard Error Format

```json
{
  "statusCode": 400,
  "message": "Validation failed",
  "errors": [
    {
      "field": "email",
      "message": "must be a valid email"
    },
    {
      "field": "password",
      "message": "must be at least 8 characters"
    }
  ],
  "timestamp": "2026-01-30T10:00:00Z",
  "path": "/api/v1/auth/register"
}
```

### Error Categories

| Code Range | Meaning | Example |
|------------|---------|---------|
| 4xx | Client error | Bad input, unauthorized |
| 5xx | Server error | Database down, bug |

---

## 9. Quick Reference

### CareCircle Endpoints Pattern

```
# Authentication
POST   /auth/register
POST   /auth/login
POST   /auth/refresh
POST   /auth/logout

# Family Resources
GET    /families
POST   /families
GET    /families/:id
PUT    /families/:id
DELETE /families/:id

# Nested Resources
GET    /families/:familyId/care-recipients
POST   /families/:familyId/care-recipients
GET    /families/:familyId/care-recipients/:id

# Actions (use sparingly)
POST   /medications/:id/log
POST   /shifts/:id/check-in
POST   /shifts/:id/check-out
```

### HTTP Headers

| Header | Purpose |
|--------|---------|
| `Content-Type: application/json` | Request body format |
| `Accept: application/json` | Response format |
| `Authorization: Bearer <token>` | Authentication |
| `X-Request-ID` | Request tracing |

---

*Next: [NestJS](nestjs.md) | [Authentication](authentication.md)*

