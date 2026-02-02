# 🔄 API Versioning - Complete Guide

> A comprehensive guide to API versioning - URL vs header strategies, deprecation, breaking changes, and maintaining backward compatibility.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "API versioning is the practice of managing changes to your API over time, allowing you to evolve the API while maintaining backward compatibility for existing clients and providing clear migration paths for breaking changes."

### The 7 Key Concepts (Remember These!)
```
1. URL VERSIONING      → /api/v1/users, /api/v2/users
2. HEADER VERSIONING   → Accept: application/vnd.api+json;version=1
3. QUERY PARAM         → /api/users?version=1
4. BREAKING CHANGE     → Change that breaks existing clients
5. DEPRECATION         → Announcing end-of-life for old version
6. SUNSET              → Final removal date for deprecated API
7. ADDITIVE CHANGE     → Non-breaking addition (new field)
```

### Versioning Strategies Comparison
```
┌─────────────────────────────────────────────────────────────────┐
│                  VERSIONING STRATEGIES                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  URL PATH VERSIONING (/api/v1/...)                             │
│  ─────────────────────────────────                              │
│  ✅ Explicit and visible                                       │
│  ✅ Easy to understand and implement                           │
│  ✅ Works with any HTTP client                                 │
│  ✅ Easy to cache (different URLs)                             │
│  ❌ Not RESTful (resource URL shouldn't change)                │
│  ❌ Large version jumps feel heavy                             │
│                                                                 │
│  HEADER VERSIONING (Accept header)                             │
│  ─────────────────────────────────                              │
│  ✅ Clean URLs                                                 │
│  ✅ More RESTful                                               │
│  ✅ Content negotiation pattern                                │
│  ❌ Less visible/discoverable                                  │
│  ❌ Harder to test (need to set headers)                       │
│  ❌ Some clients can't set custom headers                      │
│                                                                 │
│  QUERY PARAMETER (?version=1)                                  │
│  ───────────────────────────────                                │
│  ✅ Easy to implement                                          │
│  ✅ Visible in URL                                             │
│  ❌ Considered bad practice                                    │
│  ❌ Pollutes query string                                      │
│  ❌ Caching complications                                      │
│                                                                 │
│  RECOMMENDATION: URL path for public APIs                      │
│                  Header for internal APIs                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### What is a Breaking Change?
```
┌─────────────────────────────────────────────────────────────────┐
│                    BREAKING CHANGES                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ⚠️ BREAKING (Requires new version):                           │
│  • Removing an endpoint                                        │
│  • Removing a field from response                              │
│  • Renaming a field                                            │
│  • Changing field type (string → number)                       │
│  • Adding required request field                               │
│  • Changing URL structure                                      │
│  • Changing authentication method                              │
│  • Changing error response format                              │
│                                                                 │
│  ✅ NON-BREAKING (Safe to add):                                │
│  • Adding new endpoint                                         │
│  • Adding optional request field                               │
│  • Adding new field to response                                │
│  • Adding new enum value                                       │
│  • Adding new HTTP method to endpoint                          │
│  • Making required field optional                              │
│  • Deprecating (not removing) field                            │
│                                                                 │
│  RULE: Be additive, not subtractive                            │
│        Changes should be backward compatible                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Semantic versioning"** | "We follow semantic versioning for our API" |
| **"Backward compatible"** | "We ensure all changes are backward compatible" |
| **"Sunset header"** | "Deprecated endpoints return Sunset header" |
| **"Deprecation policy"** | "Our deprecation policy gives 6 months notice" |
| **"API lifecycle"** | "We manage the full API lifecycle" |
| **"Contract"** | "The API version is the contract with consumers" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Deprecation notice | **6-12 months** | Time for migration |
| Supported versions | **2-3 max** | Maintenance burden |
| Major version life | **2-3 years** | Long-term support |
| Migration period | **3-6 months** | After deprecation |

### The "Wow" Statement (Memorize This!)
> "We use URL-based versioning (/api/v1/, /api/v2/) for clear visibility and easy caching. Our policy: only breaking changes warrant a new major version. We follow additive-only changes when possible - new fields, new endpoints. Before any breaking change, we add the new functionality to the existing version with deprecation warnings. Deprecated endpoints return Sunset and Deprecation headers with the removal date. We maintain N-1 versions (current and previous), with 6-month migration windows. API changes go through review - is it breaking? Can we make it additive? We use OpenAPI specs to diff versions and detect breaking changes automatically in CI."

---

## 📚 Table of Contents

1. [URL Versioning](#1-url-versioning)
2. [Header Versioning](#2-header-versioning)
3. [Deprecation Strategy](#3-deprecation-strategy)
4. [Version Migration](#4-version-migration)
5. [Best Practices](#5-best-practices)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. URL Versioning

```typescript
// ════════════════════════════════════════════════════════════════
// URL PATH VERSIONING
// ════════════════════════════════════════════════════════════════

// URL structure
// https://api.example.com/v1/users
// https://api.example.com/v2/users

// Express implementation
import express from 'express';

const app = express();

// Version 1 routes
import v1Router from './routes/v1';
app.use('/api/v1', v1Router);

// Version 2 routes  
import v2Router from './routes/v2';
app.use('/api/v2', v2Router);

// Default to latest (optional)
app.use('/api', v2Router);

// ════════════════════════════════════════════════════════════════
// VERSIONED ROUTERS
// ════════════════════════════════════════════════════════════════

// routes/v1/users.ts
const router = Router();

// V1: Original user format
router.get('/users/:id', async (req, res) => {
  const user = await userService.findById(req.params.id);
  res.json({
    id: user.id,
    name: user.name,        // V1 has combined name
    email: user.email,
  });
});

export default router;

// routes/v2/users.ts
const router = Router();

// V2: Split name into firstName/lastName
router.get('/users/:id', async (req, res) => {
  const user = await userService.findById(req.params.id);
  res.json({
    id: user.id,
    firstName: user.firstName,  // V2 has split name
    lastName: user.lastName,
    email: user.email,
    createdAt: user.createdAt,  // V2 adds new field
  });
});

export default router;

// ════════════════════════════════════════════════════════════════
// SHARED SERVICES, DIFFERENT TRANSFORMERS
// ════════════════════════════════════════════════════════════════

// Common service (version-agnostic)
class UserService {
  async findById(id: string): Promise<UserEntity> {
    return this.repository.findById(id);
  }
}

// V1 transformer
class UserTransformerV1 {
  transform(user: UserEntity): UserResponseV1 {
    return {
      id: user.id,
      name: `${user.firstName} ${user.lastName}`,
      email: user.email,
    };
  }
}

// V2 transformer
class UserTransformerV2 {
  transform(user: UserEntity): UserResponseV2 {
    return {
      id: user.id,
      firstName: user.firstName,
      lastName: user.lastName,
      email: user.email,
      createdAt: user.createdAt.toISOString(),
    };
  }
}

// Controller using version-specific transformers
class UserController {
  constructor(
    private userService: UserService,
    private transformer: UserTransformer,
  ) {}

  async getUser(req: Request, res: Response) {
    const user = await this.userService.findById(req.params.id);
    res.json(this.transformer.transform(user));
  }
}
```

---

## 2. Header Versioning

```typescript
// ════════════════════════════════════════════════════════════════
// HEADER-BASED VERSIONING
// ════════════════════════════════════════════════════════════════

// Accept header approach
// Accept: application/vnd.example.v1+json
// Accept: application/vnd.example.v2+json

// Custom header approach
// X-API-Version: 1
// X-API-Version: 2

// Middleware to extract version
function versionMiddleware(req: Request, res: Response, next: NextFunction) {
  // Try Accept header first
  const accept = req.headers.accept || '';
  const versionMatch = accept.match(/application\/vnd\.example\.v(\d+)\+json/);
  
  if (versionMatch) {
    req.apiVersion = parseInt(versionMatch[1], 10);
  } else {
    // Fall back to custom header
    const customVersion = req.headers['x-api-version'];
    req.apiVersion = customVersion ? parseInt(customVersion as string, 10) : 2; // Default to latest
  }
  
  // Set response content type
  res.setHeader('Content-Type', `application/vnd.example.v${req.apiVersion}+json`);
  
  next();
}

app.use(versionMiddleware);

// ════════════════════════════════════════════════════════════════
// VERSION-AWARE CONTROLLER
// ════════════════════════════════════════════════════════════════

class UserController {
  async getUser(req: Request, res: Response) {
    const user = await this.userService.findById(req.params.id);
    
    // Transform based on version
    switch (req.apiVersion) {
      case 1:
        return res.json(this.transformV1(user));
      case 2:
        return res.json(this.transformV2(user));
      default:
        return res.json(this.transformV2(user)); // Default to latest
    }
  }

  private transformV1(user: User) {
    return {
      id: user.id,
      name: `${user.firstName} ${user.lastName}`,
      email: user.email,
    };
  }

  private transformV2(user: User) {
    return {
      id: user.id,
      firstName: user.firstName,
      lastName: user.lastName,
      email: user.email,
      metadata: user.metadata,
    };
  }
}

// ════════════════════════════════════════════════════════════════
// CONTENT NEGOTIATION
// ════════════════════════════════════════════════════════════════

// Request
// GET /users/123
// Accept: application/vnd.example.v2+json

// Response
// HTTP/1.1 200 OK
// Content-Type: application/vnd.example.v2+json
// {
//   "firstName": "John",
//   "lastName": "Doe",
//   ...
// }

// Using content-type library
import contentType from 'content-type';

function parseAcceptVersion(acceptHeader: string): number | null {
  try {
    const parsed = contentType.parse(acceptHeader);
    const versionParam = parsed.parameters.version;
    return versionParam ? parseInt(versionParam, 10) : null;
  } catch {
    return null;
  }
}
```

---

## 3. Deprecation Strategy

```typescript
// ════════════════════════════════════════════════════════════════
// DEPRECATION HEADERS
// ════════════════════════════════════════════════════════════════

// RFC 8594 - Sunset Header
// RFC 8288 - Deprecation Header

function deprecationMiddleware(deprecatedEndpoints: DeprecatedEndpoint[]) {
  return (req: Request, res: Response, next: NextFunction) => {
    const endpoint = deprecatedEndpoints.find(
      e => e.path === req.path && e.method === req.method
    );

    if (endpoint) {
      // Deprecation header (RFC draft)
      res.setHeader('Deprecation', endpoint.deprecatedAt);
      
      // Sunset header (RFC 8594)
      res.setHeader('Sunset', endpoint.sunsetAt);
      
      // Link to documentation
      res.setHeader('Link', `<${endpoint.docUrl}>; rel="deprecation"`);
      
      // Custom warning header
      res.setHeader(
        'X-API-Warn',
        `This endpoint is deprecated and will be removed on ${endpoint.sunsetAt}. ` +
        `Please migrate to ${endpoint.replacement}`
      );
    }

    next();
  };
}

// Configuration
const deprecatedEndpoints = [
  {
    path: '/api/v1/users',
    method: 'GET',
    deprecatedAt: '2024-01-01',
    sunsetAt: 'Sat, 01 Jul 2024 00:00:00 GMT',
    replacement: '/api/v2/users',
    docUrl: 'https://docs.example.com/migration/v1-to-v2',
  },
];

app.use(deprecationMiddleware(deprecatedEndpoints));

// ════════════════════════════════════════════════════════════════
// DEPRECATION RESPONSE WRAPPER
// ════════════════════════════════════════════════════════════════

interface DeprecatedResponse<T> {
  data: T;
  _warnings: {
    code: string;
    message: string;
    link: string;
  }[];
}

function wrapDeprecatedResponse<T>(
  data: T,
  warnings: Array<{ code: string; message: string; link: string }>
): DeprecatedResponse<T> {
  return {
    data,
    _warnings: warnings,
  };
}

// Usage
app.get('/api/v1/users/:id', async (req, res) => {
  const user = await userService.findById(req.params.id);
  
  res.json(wrapDeprecatedResponse(transformV1(user), [
    {
      code: 'DEPRECATED_ENDPOINT',
      message: 'This endpoint will be removed on 2024-07-01',
      link: 'https://docs.example.com/migration/users-v1-to-v2',
    },
    {
      code: 'DEPRECATED_FIELD',
      message: 'The "name" field is deprecated. Use firstName/lastName instead.',
      link: 'https://docs.example.com/api/users#name-field',
    },
  ]));
});

// ════════════════════════════════════════════════════════════════
// DEPRECATION MONITORING
// ════════════════════════════════════════════════════════════════

// Track deprecated endpoint usage
function trackDeprecatedUsage(req: Request, res: Response, next: NextFunction) {
  res.on('finish', () => {
    if (res.getHeader('Deprecation')) {
      metrics.increment('api.deprecated_endpoint.called', {
        path: req.path,
        method: req.method,
        clientId: req.headers['x-client-id'] || 'unknown',
      });
    }
  });
  next();
}

// Alert on deprecated endpoint usage
// Set up alerts when deprecated endpoints still have significant traffic
// close to sunset date
```

---

## 4. Version Migration

```typescript
// ════════════════════════════════════════════════════════════════
// MIGRATION STRATEGY
// ════════════════════════════════════════════════════════════════

/*
 * MIGRATION TIMELINE EXAMPLE
 * 
 * Month 1: Announce V2, deprecate V1
 *   - V2 released with new features
 *   - V1 marked deprecated
 *   - Documentation updated
 *   - Migration guide published
 * 
 * Month 2-4: Migration Period
 *   - Support both V1 and V2
 *   - Monitor V1 usage
 *   - Reach out to high-volume V1 users
 * 
 * Month 5: Warnings Intensify
 *   - V1 returns 299 Warning header
 *   - Email notifications to V1 users
 * 
 * Month 6: Sunset
 *   - V1 returns 410 Gone
 *   - Or redirect to V2 (optional)
 */

// ════════════════════════════════════════════════════════════════
// GRADUAL MIGRATION WITH FEATURE FLAGS
// ════════════════════════════════════════════════════════════════

// Allow clients to opt-in to new version features
class UserController {
  async getUsers(req: Request, res: Response) {
    const users = await this.userService.findAll();
    
    // Check if client opts into V2 format
    const useV2Format = req.headers['x-use-v2-format'] === 'true' ||
                        req.query.format === 'v2';
    
    if (useV2Format) {
      return res.json(users.map(u => this.transformV2(u)));
    }
    
    // Default to V1 format with deprecation notice
    res.setHeader('X-API-Warn', 'V1 format deprecated. Set X-Use-V2-Format: true');
    return res.json(users.map(u => this.transformV1(u)));
  }
}

// ════════════════════════════════════════════════════════════════
// OPENAPI SPEC DIFF FOR BREAKING CHANGES
// ════════════════════════════════════════════════════════════════

// package.json script
{
  "scripts": {
    "api:diff": "openapi-diff specs/v1.yaml specs/v2.yaml",
    "api:breaking": "oasdiff breaking specs/v1.yaml specs/v2.yaml"
  }
}

// CI check for breaking changes
// .github/workflows/api-check.yml
name: API Breaking Change Check

on:
  pull_request:
    paths:
      - 'specs/**'

jobs:
  check-breaking:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Check for breaking changes
        run: |
          npx oasdiff breaking \
            specs/openapi.yaml \
            specs/openapi-pr.yaml \
            --fail-on ERR

// ════════════════════════════════════════════════════════════════
// VERSION COEXISTENCE PATTERN
// ════════════════════════════════════════════════════════════════

// Both versions call same service, different transformers
class OrderService {
  async createOrder(input: CreateOrderInput): Promise<OrderEntity> {
    // Shared business logic
    return this.repository.create(input);
  }
}

// V1 endpoint
app.post('/api/v1/orders', async (req, res) => {
  // V1 input transformation
  const input = transformV1Input(req.body);
  const order = await orderService.createOrder(input);
  res.status(201).json(transformV1Output(order));
});

// V2 endpoint
app.post('/api/v2/orders', async (req, res) => {
  // V2 input transformation (may have different structure)
  const input = transformV2Input(req.body);
  const order = await orderService.createOrder(input);
  res.status(201).json(transformV2Output(order));
});
```

---

## 5. Best Practices

```yaml
# ════════════════════════════════════════════════════════════════
# API VERSIONING BEST PRACTICES
# ════════════════════════════════════════════════════════════════

versioning_strategy:
  # Prefer additive changes
  prefer:
    - Adding new optional fields
    - Adding new endpoints
    - Adding new query parameters
    - Adding new enum values
    
  avoid:
    - Removing fields
    - Renaming fields
    - Changing field types
    - Adding required fields to requests

major_version_policy:
  # When to create new major version
  new_major_when:
    - Fundamental architecture change
    - Authentication method change
    - Response format overhaul
    - Multiple breaking changes grouped
    
  avoid_new_major_when:
    - Single field change (find additive solution)
    - Can be handled with optional fields
    - Change affects small subset of users

deprecation_policy:
  announcement: "6 months before sunset"
  minimum_support: "2 major versions"
  communication:
    - Deprecation headers on responses
    - Email to registered developers
    - Documentation updates
    - Changelog entries
    - API portal notices

documentation:
  required:
    - Version changelog
    - Migration guides
    - Breaking change list
    - Deprecation timeline
    - Code examples for each version

# ════════════════════════════════════════════════════════════════
# VERSIONING DECISION MATRIX
# ════════════════════════════════════════════════════════════════

decision_matrix:
  # Is it breaking?
  breaking_change:
    yes:
      # Can we make it additive?
      can_make_additive:
        yes: "Make additive change, deprecate old"
        no: "New major version required"
    no: "Add to current version"

  # Examples
  examples:
    - change: "Add optional field to response"
      breaking: false
      action: "Add to current version"
      
    - change: "Rename field"
      breaking: true
      action: "Add new field, deprecate old, keep both"
      
    - change: "Change field type"
      breaking: true
      action: "Add new field with new type, deprecate old"
      
    - change: "Remove field"
      breaking: true
      action: "Deprecate with sunset, then remove in next major"
      
    - change: "New required request field"
      breaking: true
      action: "Make optional with default, or new version"
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# API VERSIONING PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Version per endpoint
# Bad
/api/users/v1/123
/api/orders/v2/456
# Different versions per resource - confusing!

# Good
/api/v1/users/123
/api/v1/orders/456
# Consistent version across API

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Too many versions
# Bad
/api/v1/... (2019)
/api/v2/... (2020)
/api/v3/... (2021)
/api/v4/... (2022)
/api/v5/... (2023)
# 5 versions to maintain!

# Good
# Support N and N-1 only
# Deprecate and sunset aggressively

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Breaking changes without version bump
# Bad
# V1 Monday: { "name": "John" }
# V1 Tuesday: { "firstName": "John" }  # Same version, different response!

# Good
# V1: { "name": "John" }
# V2: { "firstName": "John", "lastName": "Doe" }

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: No deprecation period
# Bad
# Monday: V1 available
# Tuesday: V1 returns 404

# Good
# Month 1: Announce deprecation
# Month 3: Intensify warnings
# Month 6: Sunset

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Different versioning schemes
# Bad
/api/v1/users       # Numbered
/api/2024/orders    # Date-based
/api/stable/products # Named

# Good
# Pick one scheme and stick with it
/api/v1/users
/api/v1/orders
/api/v1/products

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Forgetting internal consumers
# Bad
# External API versioned
# Internal services break on changes

# Good
# Treat internal APIs with same versioning discipline
# Or use contract testing between services
```

---

## 7. Interview Questions

### Basic Questions

**Q: "Why do we version APIs?"**
> "To manage change over time while maintaining backward compatibility. Clients depend on specific behavior - changing it breaks them. Versioning lets us evolve the API, introduce improvements, and fix issues without breaking existing integrations."

**Q: "What's the difference between URL and header versioning?"**
> "URL versioning (/api/v1/) is explicit, easy to test, and works everywhere. Header versioning (Accept header) is cleaner URLs, more RESTful. URL is better for public APIs (visibility), header for internal (cleaner). Most choose URL for simplicity."

**Q: "What is a breaking change?"**
> "A change that causes existing clients to fail. Examples: removing field, renaming field, changing type, adding required parameter, changing URL. Non-breaking: adding optional field, new endpoint, making required optional. Rule: be additive, not subtractive."

### Intermediate Questions

**Q: "How do you handle deprecation?"**
> "1) Announce early (6+ months). 2) Add Deprecation and Sunset headers. 3) Document migration path. 4) Monitor usage. 5) Reach out to heavy users. 6) Intensify warnings near sunset. 7) Return 410 Gone after sunset. Never surprise users with removal."

**Q: "When should you create a new major version?"**
> "When breaking changes are unavoidable: fundamental architecture change, auth overhaul, response format change. Try additive solutions first. Group multiple breaking changes into single version bump. Avoid frequent major versions - maintenance burden."

**Q: "How many versions should you support?"**
> "Typically N and N-1 (current and previous). More versions = more maintenance, testing, documentation. Set clear lifecycle: 2-3 years per major version. Have deprecation policy. Some critical APIs (banking) may need longer support."

### Advanced Questions

**Q: "How do you detect breaking changes in CI?"**
> "Use OpenAPI spec diff tools (oasdiff, openapi-diff). Compare PR spec against main branch. Flag breaking changes: removed endpoints, removed fields, type changes. Can block merge or require approval for breaking changes. Part of API governance."

**Q: "How do you handle versioning in microservices?"**
> "Each service versions independently. Use contract testing between services. Internal APIs need versioning too. Consider: API gateway can transform versions, GraphQL federation for unified version. Challenge: coordinating deprecation across services."

**Q: "REST vs GraphQL for versioning?"**
> "REST: Explicit versions, clear boundaries. GraphQL: Version-less evolution through schema additions. GraphQL can add fields without breaking, deprecate via @deprecated directive. Both need governance. GraphQL shifts complexity but doesn't eliminate versioning need entirely."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                 API VERSIONING CHECKLIST                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  VERSIONING STRATEGY:                                           │
│  □ Choose URL or header versioning                             │
│  □ Be consistent across API                                    │
│  □ Document versioning scheme                                  │
│                                                                 │
│  CHANGE MANAGEMENT:                                             │
│  □ Prefer additive changes                                     │
│  □ Avoid breaking changes                                      │
│  □ Use OpenAPI diff in CI                                      │
│                                                                 │
│  DEPRECATION:                                                   │
│  □ 6+ months notice                                            │
│  □ Deprecation/Sunset headers                                  │
│  □ Migration documentation                                     │
│  □ Monitor usage                                               │
│                                                                 │
│  LIFECYCLE:                                                     │
│  □ Support N and N-1 versions                                  │
│  □ Clear sunset dates                                          │
│  □ Changelog maintained                                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

BREAKING VS NON-BREAKING:
┌────────────────────────────────────────────────────────────────┐
│ BREAKING:        Remove field, rename, type change, required  │
│ NON-BREAKING:    Add optional field, new endpoint, deprecate  │
└────────────────────────────────────────────────────────────────┘

DEPRECATION HEADERS:
┌────────────────────────────────────────────────────────────────┐
│ Deprecation: Mon, 01 Jan 2024 00:00:00 GMT                    │
│ Sunset: Sat, 01 Jul 2024 00:00:00 GMT                         │
│ Link: <https://docs/migration>; rel="deprecation"             │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

