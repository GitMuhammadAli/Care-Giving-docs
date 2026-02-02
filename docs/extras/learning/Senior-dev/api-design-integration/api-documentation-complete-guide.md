# 📖 API Documentation - Complete Guide

> A comprehensive guide to API documentation - OpenAPI/Swagger, API-first design, documentation tools, and best practices for developer experience.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "API documentation is the technical content that describes how to use and integrate with an API, ideally generated from a machine-readable specification (OpenAPI) that serves as the single source of truth for the API contract."

### The 7 Key Concepts (Remember These!)
```
1. OPENAPI (SWAGGER)  → Standard API specification format
2. API-FIRST          → Design spec before implementation
3. CODE-FIRST         → Generate spec from implementation
4. INTERACTIVE DOCS   → Try endpoints in browser (Swagger UI)
5. SDK GENERATION     → Auto-generate client libraries
6. CONTRACT TESTING   → Validate implementation matches spec
7. DEVELOPER PORTAL   → Hub for all API documentation
```

### OpenAPI Specification Structure
```
┌─────────────────────────────────────────────────────────────────┐
│                  OPENAPI 3.0 STRUCTURE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  openapi: 3.0.3                    # OpenAPI version           │
│  info:                             # API metadata              │
│    title, version, description                                 │
│  servers:                          # API servers               │
│    - url: https://api.example.com                              │
│  paths:                            # Endpoints                 │
│    /users:                                                     │
│      get:                          # HTTP method               │
│        summary, description                                    │
│        parameters:                 # Query/path params         │
│        responses:                  # Response definitions      │
│          200:                                                  │
│            content:                                            │
│              application/json:                                 │
│                schema: $ref        # Reference to schema       │
│  components:                       # Reusable definitions      │
│    schemas:                        # Data models               │
│    parameters:                     # Shared parameters         │
│    responses:                      # Shared responses          │
│    securitySchemes:                # Auth definitions          │
│  security:                         # Global security           │
│  tags:                             # Grouping                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### API-First vs Code-First
```
┌─────────────────────────────────────────────────────────────────┐
│               API-FIRST vs CODE-FIRST                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  API-FIRST (Design-First)                                      │
│  ─────────────────────────                                      │
│  1. Write OpenAPI spec                                         │
│  2. Review with stakeholders                                   │
│  3. Generate server stubs                                      │
│  4. Implement business logic                                   │
│  5. Generate client SDKs                                       │
│                                                                 │
│  ✅ Better API design                                          │
│  ✅ Early feedback                                             │
│  ✅ Parallel frontend/backend work                             │
│  ❌ Spec can drift from implementation                         │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  CODE-FIRST                                                    │
│  ───────────                                                    │
│  1. Implement API                                              │
│  2. Add annotations/decorators                                 │
│  3. Generate OpenAPI spec                                      │
│  4. Generate documentation                                     │
│                                                                 │
│  ✅ Spec always matches code                                   │
│  ✅ Faster initial development                                 │
│  ❌ Design may be implementation-driven                        │
│  ❌ Less upfront planning                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"OpenAPI 3.0"** | "Our APIs are documented using OpenAPI 3.0 specification" |
| **"API-first"** | "We follow API-first design, spec before code" |
| **"Swagger UI"** | "Interactive docs served via Swagger UI" |
| **"Contract testing"** | "We contract test against the OpenAPI spec" |
| **"Developer portal"** | "Developers access docs through our portal" |
| **"SDK generation"** | "SDKs auto-generated from OpenAPI spec" |

### Key Numbers to Remember
| Metric | Target | Why |
|--------|-------|-----|
| Example coverage | **100%** | Every endpoint has examples |
| Schema coverage | **100%** | All request/response schemas |
| Authentication docs | **Clear** | Getting started guide |
| Time to first call | **< 5 min** | Developer experience |

### The "Wow" Statement (Memorize This!)
> "We use API-first design with OpenAPI 3.0 specification. The spec is the contract - reviewed before implementation, validated in CI, and used to generate docs, server stubs, and client SDKs. We serve interactive documentation via Swagger UI where developers can try endpoints with live requests. Every endpoint has examples, error scenarios documented, and authentication clearly explained. Our developer portal includes getting started guides, code samples in multiple languages, and a changelog. We use Spectral for linting the spec, ensuring consistency. Contract tests verify implementation matches spec - any drift fails CI."

---

## 📚 Table of Contents

1. [OpenAPI Specification](#1-openapi-specification)
2. [Documentation Tools](#2-documentation-tools)
3. [API-First Development](#3-api-first-development)
4. [Code-First Documentation](#4-code-first-documentation)
5. [Best Practices](#5-best-practices)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. OpenAPI Specification

```yaml
# ════════════════════════════════════════════════════════════════
# COMPLETE OPENAPI EXAMPLE
# ════════════════════════════════════════════════════════════════

openapi: 3.0.3

info:
  title: User Management API
  version: 1.0.0
  description: |
    API for managing users and their resources.
    
    ## Authentication
    All endpoints require Bearer token authentication.
    
    ## Rate Limiting
    - 100 requests per minute per API key
    - Rate limit headers included in responses
  contact:
    name: API Support
    email: api-support@example.com
    url: https://example.com/support
  license:
    name: MIT
    url: https://opensource.org/licenses/MIT

servers:
  - url: https://api.example.com/v1
    description: Production server
  - url: https://staging-api.example.com/v1
    description: Staging server
  - url: http://localhost:3000/v1
    description: Development server

tags:
  - name: Users
    description: User management operations
  - name: Orders
    description: Order management operations

# ════════════════════════════════════════════════════════════════
# PATHS (ENDPOINTS)
# ════════════════════════════════════════════════════════════════

paths:
  /users:
    get:
      tags:
        - Users
      summary: List all users
      description: |
        Retrieve a paginated list of users.
        Supports filtering and sorting.
      operationId: listUsers
      parameters:
        - $ref: '#/components/parameters/PageParam'
        - $ref: '#/components/parameters/LimitParam'
        - name: status
          in: query
          description: Filter by user status
          schema:
            type: string
            enum: [active, inactive, pending]
        - name: sort
          in: query
          description: Sort field and direction
          schema:
            type: string
            example: createdAt:desc
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserListResponse'
              example:
                data:
                  - id: "usr_123"
                    name: "John Doe"
                    email: "john@example.com"
                    status: active
                meta:
                  total: 100
                  page: 1
                  limit: 20
        '401':
          $ref: '#/components/responses/Unauthorized'
        '500':
          $ref: '#/components/responses/InternalError'
      security:
        - bearerAuth: []

    post:
      tags:
        - Users
      summary: Create a new user
      description: Create a new user account
      operationId: createUser
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserRequest'
            examples:
              basic:
                summary: Basic user creation
                value:
                  name: John Doe
                  email: john@example.com
                  password: securePassword123
              withRole:
                summary: User with admin role
                value:
                  name: Jane Admin
                  email: jane@example.com
                  password: securePassword123
                  role: admin
      responses:
        '201':
          description: User created successfully
          headers:
            Location:
              description: URL of created resource
              schema:
                type: string
                example: /users/usr_456
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '400':
          $ref: '#/components/responses/BadRequest'
        '409':
          description: Email already exists
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
              example:
                type: https://api.example.com/errors/conflict
                title: Conflict
                status: 409
                detail: User with this email already exists
      security:
        - bearerAuth: []

  /users/{id}:
    get:
      tags:
        - Users
      summary: Get user by ID
      operationId: getUser
      parameters:
        - name: id
          in: path
          required: true
          description: User ID
          schema:
            type: string
            pattern: '^usr_[a-zA-Z0-9]+$'
            example: usr_123
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          $ref: '#/components/responses/NotFound'
      security:
        - bearerAuth: []

    patch:
      tags:
        - Users
      summary: Update user
      operationId: updateUser
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UpdateUserRequest'
      responses:
        '200':
          description: User updated
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          $ref: '#/components/responses/NotFound'
      security:
        - bearerAuth: []

    delete:
      tags:
        - Users
      summary: Delete user
      operationId: deleteUser
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
      responses:
        '204':
          description: User deleted successfully
        '404':
          $ref: '#/components/responses/NotFound'
      security:
        - bearerAuth: []

# ════════════════════════════════════════════════════════════════
# COMPONENTS (Reusable Definitions)
# ════════════════════════════════════════════════════════════════

components:
  schemas:
    User:
      type: object
      required:
        - id
        - name
        - email
        - status
        - createdAt
      properties:
        id:
          type: string
          description: Unique user identifier
          example: usr_123
          readOnly: true
        name:
          type: string
          description: User's full name
          example: John Doe
          minLength: 1
          maxLength: 100
        email:
          type: string
          format: email
          description: User's email address
          example: john@example.com
        status:
          type: string
          enum: [active, inactive, pending]
          description: User account status
        role:
          type: string
          enum: [user, admin, moderator]
          default: user
        createdAt:
          type: string
          format: date-time
          readOnly: true
        updatedAt:
          type: string
          format: date-time
          readOnly: true

    CreateUserRequest:
      type: object
      required:
        - name
        - email
        - password
      properties:
        name:
          type: string
          minLength: 1
          maxLength: 100
        email:
          type: string
          format: email
        password:
          type: string
          format: password
          minLength: 8
          writeOnly: true
        role:
          type: string
          enum: [user, admin]
          default: user

    UpdateUserRequest:
      type: object
      properties:
        name:
          type: string
        email:
          type: string
          format: email
        status:
          type: string
          enum: [active, inactive]

    UserListResponse:
      type: object
      properties:
        data:
          type: array
          items:
            $ref: '#/components/schemas/User'
        meta:
          $ref: '#/components/schemas/PaginationMeta'

    PaginationMeta:
      type: object
      properties:
        total:
          type: integer
        page:
          type: integer
        limit:
          type: integer
        totalPages:
          type: integer

    Error:
      type: object
      required:
        - type
        - title
        - status
      properties:
        type:
          type: string
          format: uri
        title:
          type: string
        status:
          type: integer
        detail:
          type: string
        instance:
          type: string

  parameters:
    PageParam:
      name: page
      in: query
      description: Page number (1-indexed)
      schema:
        type: integer
        minimum: 1
        default: 1

    LimitParam:
      name: limit
      in: query
      description: Number of items per page
      schema:
        type: integer
        minimum: 1
        maximum: 100
        default: 20

  responses:
    BadRequest:
      description: Bad request
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
          example:
            type: https://api.example.com/errors/validation
            title: Validation Error
            status: 400
            detail: Request body contains invalid data

    Unauthorized:
      description: Authentication required
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
          example:
            type: https://api.example.com/errors/unauthorized
            title: Unauthorized
            status: 401
            detail: Valid authentication token required

    NotFound:
      description: Resource not found
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
          example:
            type: https://api.example.com/errors/not-found
            title: Not Found
            status: 404
            detail: The requested resource was not found

    InternalError:
      description: Internal server error
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'

  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: |
        Enter your JWT token in the format: `Bearer <token>`
        
        To obtain a token, use the `/auth/login` endpoint.

    apiKey:
      type: apiKey
      in: header
      name: X-API-Key
      description: API key for server-to-server communication

security:
  - bearerAuth: []
```

---

## 2. Documentation Tools

```typescript
// ════════════════════════════════════════════════════════════════
// SWAGGER UI SETUP (Express)
// ════════════════════════════════════════════════════════════════

import express from 'express';
import swaggerUi from 'swagger-ui-express';
import YAML from 'yamljs';

const app = express();

// Load OpenAPI spec
const swaggerDocument = YAML.load('./openapi.yaml');

// Serve Swagger UI
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument, {
  customCss: '.swagger-ui .topbar { display: none }',
  customSiteTitle: 'My API Documentation',
  swaggerOptions: {
    persistAuthorization: true,
    displayRequestDuration: true,
    filter: true,
  },
}));

// ════════════════════════════════════════════════════════════════
// REDOC SETUP (Alternative to Swagger UI)
// ════════════════════════════════════════════════════════════════

import redoc from 'redoc-express';

app.get('/docs', redoc({
  title: 'API Documentation',
  specUrl: '/openapi.yaml',
  redocOptions: {
    theme: {
      colors: {
        primary: {
          main: '#3f51b5',
        },
      },
    },
    hideDownloadButton: false,
    expandResponses: '200,201',
  },
}));

// ════════════════════════════════════════════════════════════════
// SPECTRAL (OpenAPI Linting)
// ════════════════════════════════════════════════════════════════

// .spectral.yaml - Linting rules
/*
extends: ["spectral:oas"]

rules:
  # Require descriptions
  operation-description: error
  operation-operationId: error
  
  # Require examples
  oas3-valid-schema-example: error
  
  # Naming conventions
  operation-operationId-valid-in-url: warn
  
  # Custom rules
  must-have-contact:
    given: "$.info"
    then:
      field: contact
      function: truthy
    message: "API must have contact information"
*/

// package.json
/*
{
  "scripts": {
    "lint:api": "spectral lint openapi.yaml",
    "lint:api:fix": "spectral lint openapi.yaml --format stylish"
  }
}
*/

// ════════════════════════════════════════════════════════════════
// SDK GENERATION (openapi-generator)
// ════════════════════════════════════════════════════════════════

// package.json scripts
/*
{
  "scripts": {
    "generate:typescript": "openapi-generator-cli generate -i openapi.yaml -g typescript-axios -o ./sdk/typescript",
    "generate:python": "openapi-generator-cli generate -i openapi.yaml -g python -o ./sdk/python",
    "generate:java": "openapi-generator-cli generate -i openapi.yaml -g java -o ./sdk/java"
  }
}
*/

// Generated SDK usage
import { UsersApi, Configuration } from './sdk/typescript';

const config = new Configuration({
  basePath: 'https://api.example.com/v1',
  accessToken: 'your-jwt-token',
});

const usersApi = new UsersApi(config);

// Fully typed API calls
const users = await usersApi.listUsers({ page: 1, limit: 20 });
const user = await usersApi.createUser({ name: 'John', email: 'john@example.com', password: 'secret' });
```

---

## 3. API-First Development

```typescript
// ════════════════════════════════════════════════════════════════
// API-FIRST WORKFLOW
// ════════════════════════════════════════════════════════════════

/*
 * API-FIRST DEVELOPMENT PROCESS
 * 
 * 1. DESIGN PHASE
 *    - Write OpenAPI specification
 *    - Review with stakeholders (frontend, mobile, partners)
 *    - Iterate on design
 * 
 * 2. MOCK PHASE
 *    - Generate mock server from spec
 *    - Frontend can start development
 * 
 * 3. IMPLEMENTATION PHASE
 *    - Generate server stubs
 *    - Implement business logic
 *    - Contract tests validate implementation
 * 
 * 4. DOCUMENTATION PHASE
 *    - Auto-generate documentation
 *    - Add examples and tutorials
 *    - Generate SDKs
 */

// ════════════════════════════════════════════════════════════════
// MOCK SERVER (Prism)
// ════════════════════════════════════════════════════════════════

// Start mock server
// npx prism mock openapi.yaml

// Mock server responds based on examples in spec
// GET /users/usr_123 returns example from spec

// ════════════════════════════════════════════════════════════════
// SERVER STUB GENERATION
// ════════════════════════════════════════════════════════════════

// Generate Express server stub
// npx openapi-generator-cli generate -i openapi.yaml -g nodejs-express-server -o ./server

// Generated code structure:
// server/
//   controllers/
//     UsersController.js    # Stub methods to implement
//   services/
//     UsersService.js       # Business logic here
//   api/
//     openapi.yaml          # Spec

// Implement the service
class UsersService {
  async listUsers({ page, limit, status }) {
    // Implement actual logic
    const users = await this.userRepository.findAll({ page, limit, status });
    return {
      data: users,
      meta: { total: users.length, page, limit },
    };
  }

  async createUser(body) {
    // Validation happens automatically based on spec
    return this.userRepository.create(body);
  }
}

// ════════════════════════════════════════════════════════════════
// CONTRACT TESTING (Dredd)
// ════════════════════════════════════════════════════════════════

// dredd.yml
/*
dry-run: false
hookfiles: ./test/hooks.js
language: nodejs
server: npm start
endpoint: 'http://localhost:3000'
path: ['./openapi.yaml']
*/

// test/hooks.js
const hooks = require('hooks');
const jwt = require('jsonwebtoken');

// Add auth token before each request
hooks.beforeEach((transaction, done) => {
  transaction.request.headers['Authorization'] = `Bearer ${generateTestToken()}`;
  done();
});

// Setup test data
hooks.before('Users > Create a new user', (transaction, done) => {
  // Clean up any existing test user
  done();
});

// Run: npx dredd
// Validates implementation matches OpenAPI spec

// ════════════════════════════════════════════════════════════════
// TYPESCRIPT TYPES FROM OPENAPI
// ════════════════════════════════════════════════════════════════

// Using openapi-typescript
// npx openapi-typescript openapi.yaml -o ./types/api.ts

// Generated types
export interface components {
  schemas: {
    User: {
      id: string;
      name: string;
      email: string;
      status: 'active' | 'inactive' | 'pending';
      role?: 'user' | 'admin' | 'moderator';
      createdAt: string;
      updatedAt?: string;
    };
    CreateUserRequest: {
      name: string;
      email: string;
      password: string;
      role?: 'user' | 'admin';
    };
  };
}

// Use in code
import type { components } from './types/api';

type User = components['schemas']['User'];
type CreateUserRequest = components['schemas']['CreateUserRequest'];

async function createUser(data: CreateUserRequest): Promise<User> {
  // TypeScript ensures type safety
  return api.post('/users', data);
}
```

---

## 4. Code-First Documentation

```typescript
// ════════════════════════════════════════════════════════════════
// CODE-FIRST WITH TSOA
// ════════════════════════════════════════════════════════════════

import { 
  Controller, 
  Get, 
  Post, 
  Put, 
  Delete, 
  Route, 
  Path, 
  Body, 
  Query, 
  Response, 
  Security, 
  Tags,
  Example,
} from 'tsoa';

interface User {
  id: string;
  name: string;
  email: string;
  status: 'active' | 'inactive';
}

interface CreateUserRequest {
  name: string;
  email: string;
  password: string;
}

@Route('users')
@Tags('Users')
@Security('bearerAuth')
export class UsersController extends Controller {
  /**
   * Retrieves a paginated list of users
   * @param page Page number (1-indexed)
   * @param limit Number of items per page
   */
  @Get()
  @Example<User[]>([
    { id: 'usr_123', name: 'John Doe', email: 'john@example.com', status: 'active' },
  ])
  public async listUsers(
    @Query() page: number = 1,
    @Query() limit: number = 20,
  ): Promise<{ data: User[]; total: number }> {
    // Implementation
  }

  /**
   * Get a user by ID
   * @param id User identifier
   */
  @Get('{id}')
  @Response<{ message: string }>(404, 'User not found')
  public async getUser(
    @Path() id: string,
  ): Promise<User> {
    // Implementation
  }

  /**
   * Create a new user
   * @param body User creation data
   */
  @Post()
  @Response<User>(201, 'User created')
  @Response<{ message: string }>(400, 'Validation error')
  @Response<{ message: string }>(409, 'Email already exists')
  public async createUser(
    @Body() body: CreateUserRequest,
  ): Promise<User> {
    this.setStatus(201);
    // Implementation
  }

  /**
   * Delete a user
   * @param id User identifier
   */
  @Delete('{id}')
  @Response(204, 'User deleted')
  @Response<{ message: string }>(404, 'User not found')
  public async deleteUser(
    @Path() id: string,
  ): Promise<void> {
    this.setStatus(204);
    // Implementation
  }
}

// tsoa.json configuration
/*
{
  "entryFile": "src/index.ts",
  "noImplicitAdditionalProperties": "throw-on-extras",
  "controllerPathGlobs": ["src/**/*Controller.ts"],
  "spec": {
    "outputDirectory": "public",
    "specVersion": 3,
    "securityDefinitions": {
      "bearerAuth": {
        "type": "http",
        "scheme": "bearer"
      }
    }
  },
  "routes": {
    "routesDir": "src/routes"
  }
}
*/

// Generate spec: npx tsoa spec
// Generate routes: npx tsoa routes

// ════════════════════════════════════════════════════════════════
// CODE-FIRST WITH NestJS + Swagger
// ════════════════════════════════════════════════════════════════

import { 
  Controller, 
  Get, 
  Post, 
  Body, 
  Param, 
  Query,
} from '@nestjs/common';
import { 
  ApiTags, 
  ApiOperation, 
  ApiResponse, 
  ApiQuery,
  ApiBearerAuth,
  ApiProperty,
} from '@nestjs/swagger';

class CreateUserDto {
  @ApiProperty({ example: 'John Doe', description: 'User full name' })
  name: string;

  @ApiProperty({ example: 'john@example.com', format: 'email' })
  email: string;

  @ApiProperty({ minLength: 8, description: 'User password' })
  password: string;
}

class UserDto {
  @ApiProperty({ example: 'usr_123' })
  id: string;

  @ApiProperty()
  name: string;

  @ApiProperty()
  email: string;

  @ApiProperty({ enum: ['active', 'inactive'] })
  status: string;
}

@ApiTags('Users')
@ApiBearerAuth()
@Controller('users')
export class UsersController {
  @Get()
  @ApiOperation({ summary: 'List all users' })
  @ApiQuery({ name: 'page', required: false, type: Number })
  @ApiQuery({ name: 'limit', required: false, type: Number })
  @ApiResponse({ status: 200, description: 'User list', type: [UserDto] })
  findAll(
    @Query('page') page = 1,
    @Query('limit') limit = 20,
  ): Promise<UserDto[]> {
    // Implementation
  }

  @Post()
  @ApiOperation({ summary: 'Create a user' })
  @ApiResponse({ status: 201, description: 'User created', type: UserDto })
  @ApiResponse({ status: 400, description: 'Validation error' })
  @ApiResponse({ status: 409, description: 'Email already exists' })
  create(@Body() createUserDto: CreateUserDto): Promise<UserDto> {
    // Implementation
  }

  @Get(':id')
  @ApiOperation({ summary: 'Get user by ID' })
  @ApiResponse({ status: 200, description: 'User found', type: UserDto })
  @ApiResponse({ status: 404, description: 'User not found' })
  findOne(@Param('id') id: string): Promise<UserDto> {
    // Implementation
  }
}

// main.ts - Setup Swagger
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const config = new DocumentBuilder()
    .setTitle('User API')
    .setDescription('User management API')
    .setVersion('1.0')
    .addBearerAuth()
    .build();

  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api-docs', app, document);

  await app.listen(3000);
}
```

---

## 5. Best Practices

```yaml
# ════════════════════════════════════════════════════════════════
# API DOCUMENTATION BEST PRACTICES
# ════════════════════════════════════════════════════════════════

content_best_practices:
  descriptions:
    - Every endpoint has clear summary and description
    - Parameters explain purpose, not just name
    - Response descriptions explain what's returned
    
  examples:
    - Every request body has realistic examples
    - Multiple examples for different scenarios
    - Response examples for success and error cases
    
  error_documentation:
    - Document all possible error responses
    - Include error codes and meanings
    - Show error response format

structure_best_practices:
  organization:
    - Group endpoints with tags
    - Consistent naming conventions
    - Logical endpoint ordering
    
  reusability:
    - Use $ref for shared schemas
    - Common parameters in components
    - Shared response definitions
    
  versioning:
    - Version in info section
    - Clear changelog
    - Deprecation notices

developer_experience:
  getting_started:
    - Quick start guide
    - Authentication walkthrough
    - First API call example
    
  code_samples:
    - Examples in multiple languages
    - cURL commands
    - SDK usage examples
    
  interactive:
    - Try-it-out functionality
    - Sandbox environment
    - Test credentials available

# ════════════════════════════════════════════════════════════════
# DOCUMENTATION CHECKLIST
# ════════════════════════════════════════════════════════════════

checklist:
  api_reference:
    - [ ] All endpoints documented
    - [ ] All parameters described
    - [ ] All schemas defined
    - [ ] Examples provided
    - [ ] Error responses documented
    - [ ] Authentication explained
    
  guides:
    - [ ] Getting started guide
    - [ ] Authentication guide
    - [ ] Common use cases
    - [ ] Error handling guide
    - [ ] Rate limiting explained
    
  tools:
    - [ ] Interactive documentation
    - [ ] API playground/sandbox
    - [ ] SDK downloads
    - [ ] Postman collection
    
  maintenance:
    - [ ] Changelog maintained
    - [ ] Deprecation notices
    - [ ] Version migration guides
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# API DOCUMENTATION PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: No examples
# Bad
parameters:
  - name: status
    in: query
    schema:
      type: string
# What values are valid?

# Good
parameters:
  - name: status
    in: query
    description: Filter users by account status
    schema:
      type: string
      enum: [active, inactive, pending]
    example: active

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Generic descriptions
# Bad
summary: Get user
description: Gets a user

# Good
summary: Get user by ID
description: |
  Retrieves a single user by their unique identifier.
  Returns 404 if user doesn't exist.
  Requires authentication.

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Missing error responses
# Bad
responses:
  '200':
    description: Success

# Good
responses:
  '200':
    description: User retrieved successfully
  '400':
    description: Invalid user ID format
  '401':
    description: Authentication required
  '404':
    description: User not found

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Outdated documentation
# Bad - Docs don't match implementation

# Good
# - Generate docs from code (code-first)
# - Or contract test docs against code (API-first)
# - CI validation

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: No authentication docs
# Bad
# Endpoints show lock icon but no explanation

# Good
securitySchemes:
  bearerAuth:
    type: http
    scheme: bearer
    description: |
      To authenticate:
      1. Call POST /auth/login with credentials
      2. Copy the token from response
      3. Click "Authorize" and enter: Bearer <token>
      
      Tokens expire after 24 hours.

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Copy-paste schemas instead of $ref
# Bad - Duplicated schema definition
paths:
  /users:
    get:
      responses:
        '200':
          content:
            application/json:
              schema:
                type: object
                properties:
                  id: { type: string }
                  name: { type: string }
  /users/{id}:
    get:
      responses:
        '200':
          content:
            application/json:
              schema:
                type: object
                properties:
                  id: { type: string }
                  name: { type: string }
                  # Same schema repeated!

# Good - Use $ref
components:
  schemas:
    User:
      type: object
      properties:
        id: { type: string }
        name: { type: string }

paths:
  /users:
    get:
      responses:
        '200':
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is OpenAPI/Swagger?"**
> "OpenAPI is a standard specification format for describing REST APIs. Originally called Swagger, it's now OpenAPI Specification (OAS). Defines endpoints, parameters, request/response schemas, authentication. Machine-readable (YAML/JSON), enabling tooling: documentation generation, SDK generation, validation, mocking."

**Q: "API-first vs code-first?"**
> "API-first: Design spec first, then implement. Better design, early stakeholder feedback, parallel development. Code-first: Implement first, generate spec. Ensures docs match code, faster initial development. Choose API-first for public APIs, code-first for rapid internal development."

**Q: "What makes good API documentation?"**
> "Good docs have: clear descriptions, realistic examples, all error responses documented, authentication explained, getting started guide, code samples in multiple languages, interactive try-it feature. Goal: developer can make first API call in under 5 minutes."

### Intermediate Questions

**Q: "How do you keep docs in sync with code?"**
> "Code-first: Generate spec from code using annotations (TSOA, NestJS Swagger). API-first: Contract testing - tools like Dredd validate implementation against spec. CI checks for drift. Either approach prevents outdated docs. Never manually maintain separate spec and code."

**Q: "How do you handle SDK generation?"**
> "Use openapi-generator to generate client SDKs from OpenAPI spec. Supports 40+ languages. Generated SDKs have full types, authentication handling, error types. Reduces integration effort. Publish to package registries (npm, PyPI). Version SDKs with API versions."

**Q: "What tools do you use for API documentation?"**
> "OpenAPI spec as source of truth. Swagger UI or Redoc for interactive docs. Spectral for spec linting. Prism for mock servers. openapi-generator for SDKs. Dredd or Prism for contract testing. Stoplight or ReadMe for developer portals. Pick tools based on team size and needs."

### Advanced Questions

**Q: "How do you ensure documentation quality?"**
> "1) Linting with Spectral (require descriptions, examples). 2) CI validation (spec valid, no breaking changes). 3) Contract testing (implementation matches spec). 4) Code review includes docs review. 5) Developer feedback channel. 6) Analytics on docs usage. Quality is ongoing process."

**Q: "How do you design a developer portal?"**
> "Portal includes: Landing page with value prop. Getting started (5-minute quickstart). Authentication guide. API reference (from OpenAPI). Guides for common use cases. Code samples. SDK downloads. Changelog. Support contact. Search. Maybe: sandbox, API explorer, status page."

**Q: "How would you document a complex workflow?"**
> "For multi-step workflows: 1) Overview explaining the complete flow. 2) Sequence diagram showing steps. 3) Each endpoint documented with its role in workflow. 4) Code sample showing complete implementation. 5) Common errors and how to handle. 6) Edge cases. Example: OAuth flow, checkout process."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│               API DOCUMENTATION CHECKLIST                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  OPENAPI SPEC:                                                  │
│  □ All endpoints documented                                    │
│  □ Descriptions are clear                                      │
│  □ Examples provided                                           │
│  □ Error responses documented                                  │
│  □ Authentication defined                                      │
│  □ Schemas use $ref                                            │
│                                                                 │
│  TOOLING:                                                       │
│  □ Interactive docs (Swagger UI/Redoc)                         │
│  □ Spec linting (Spectral)                                     │
│  □ Contract testing                                            │
│  □ SDK generation                                              │
│                                                                 │
│  DEVELOPER EXPERIENCE:                                          │
│  □ Getting started guide                                       │
│  □ Authentication walkthrough                                  │
│  □ Code samples                                                │
│  □ Changelog                                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

OPENAPI STRUCTURE:
┌────────────────────────────────────────────────────────────────┐
│ openapi: 3.0.3          # Version                              │
│ info: ...               # Metadata                             │
│ servers: ...            # API servers                          │
│ paths: ...              # Endpoints                            │
│ components:             # Reusable pieces                      │
│   schemas: ...          # Data models                          │
│   parameters: ...       # Shared params                        │
│   responses: ...        # Shared responses                     │
│   securitySchemes: ...  # Auth methods                         │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

