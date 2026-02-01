# NestJS Concepts

> Understanding the architecture and patterns of NestJS.

---

## 1. What Is NestJS?

### Plain English Explanation

NestJS is a **framework for building server-side applications** with Node.js. It provides structure and patterns inspired by Angular.

Think of it like a **well-organized office building**:
- **Modules** = Departments (Finance, HR, Engineering)
- **Controllers** = Reception desks (handle incoming requests)
- **Services** = Workers (do the actual work)
- **Guards** = Security checkpoints (verify access)

### The Core Problem NestJS Solves

Express.js gives you freedom, but freedom leads to chaos at scale. NestJS provides:
- Enforced structure
- Dependency injection
- Decorator-based configuration
- TypeScript-first development

---

## 2. Core Concepts & Terminology

### The Module System

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         MODULE ANATOMY                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  @Module({                                                                   │
│    imports: [OtherModule],     // Modules this module needs                 │
│    controllers: [MyController], // Request handlers                          │
│    providers: [MyService],      // Injectable services                       │
│    exports: [MyService],        // What other modules can use               │
│  })                                                                          │
│                                                                              │
│  Each module is a SELF-CONTAINED feature package                            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Key Terminology

| Term | Definition | Purpose |
|------|------------|---------|
| **Module** | Feature container | Organize related code |
| **Controller** | Request handler | Define routes, call services |
| **Service** | Business logic | Reusable operations |
| **Provider** | Injectable class | Anything that can be injected |
| **Guard** | Access control | Verify permissions |
| **Pipe** | Data transformation | Validate/transform input |
| **Interceptor** | Request/response wrapper | Logging, caching, transformation |
| **Middleware** | Request preprocessing | CORS, logging, parsing |

---

## 3. How Dependency Injection Works

### The Mental Model

**Dependency Injection** = Don't create your own tools, ask for them.

```
WITHOUT DI:
class ChefService {
  constructor() {
    this.oven = new Oven();  // Chef creates own oven
    // What if we want a different oven for testing?
  }
}

WITH DI:
class ChefService {
  constructor(private oven: Oven) {}  // Oven provided from outside
  // Now we can inject any oven: RealOven, TestOven, MockOven
}
```

### Why It Matters

| Benefit | Explanation |
|---------|-------------|
| **Testability** | Replace real DB with mock for tests |
| **Flexibility** | Swap implementations without changing code |
| **Singleton management** | One database connection shared by all |
| **Clear dependencies** | Constructor shows what's needed |

---

## 4. Why NestJS for CareCircle

| Requirement | Why NestJS Fits |
|-------------|-----------------|
| TypeScript-first | Full type safety, better DX |
| Scalable structure | Modules keep code organized |
| Enterprise patterns | DI, decorators, guards |
| Rich ecosystem | Swagger, WebSocket, GraphQL support |
| Testing built-in | Jest integration, testing utilities |

---

## 5. When to Use NestJS Patterns ✅

### Use Controllers When:
- Defining HTTP routes
- Handling request/response
- Coordinating service calls

### Use Services When:
- Implementing business logic
- Accessing databases
- Calling external APIs
- Code needs to be reusable

### Use Guards When:
- Checking authentication
- Checking authorization
- Route-level access control

### Use Pipes When:
- Validating input
- Transforming data types
- Parsing request bodies

---

## 6. When to AVOID Patterns ❌

### DON'T Put Business Logic in Controllers

```
❌ BAD: Controller does everything
@Post()
create(@Body() dto) {
  if (!dto.name) throw new Error();
  const user = await this.db.user.findOne(...);
  if (!user.canCreate) throw new ForbiddenException();
  return this.db.medication.create({ data: dto });
}

✅ GOOD: Controller delegates to service
@Post()
create(@Body() dto: CreateMedicationDto) {
  return this.medicationsService.create(dto);
}
```

### DON'T Create "God Modules"

Split large modules into focused, single-responsibility modules.

### DON'T Skip Validation

Always use DTOs with class-validator decorators for input validation.

---

## 7. Best Practices & Recommendations

### Layer Responsibilities

| Layer | Does | Doesn't |
|-------|------|---------|
| **Controller** | Route handling, call services | Business logic, DB access |
| **Service** | Business logic, orchestration | HTTP concerns, responses |
| **Repository** | Data access | Business rules |

### File Organization

```
src/
├── medications/
│   ├── medications.module.ts
│   ├── medications.controller.ts
│   ├── medications.service.ts
│   ├── dto/
│   │   ├── create-medication.dto.ts
│   │   └── update-medication.dto.ts
│   └── entities/
│       └── medication.entity.ts
```

### Naming Conventions

```
Modules: PascalCase + Module (MedicationsModule)
Controllers: PascalCase + Controller (MedicationsController)
Services: PascalCase + Service (MedicationsService)
DTOs: Action + Resource + Dto (CreateMedicationDto)
```

---

## 8. Common Mistakes & How to Avoid Them

### Mistake 1: Circular Dependencies

Module A imports Module B which imports Module A. Use `forwardRef()` or restructure.

### Mistake 2: Missing @Injectable()

Services must have `@Injectable()` decorator to be injected.

### Mistake 3: Forgetting to Register Providers

If you create a service, add it to the module's `providers` array.

---

## 9. Request Lifecycle

```
REQUEST
   │
   ▼
Middleware (global, then module-specific)
   │
   ▼
Guards (global → controller → route)
   │
   ▼
Interceptors (pre-controller)
   │
   ▼
Pipes (transform/validate)
   │
   ▼
Controller method
   │
   ▼
Interceptors (post-controller)
   │
   ▼
Exception filters (if error)
   │
   ▼
RESPONSE
```

---

## 10. Quick Reference

### Common Decorators

| Decorator | Purpose |
|-----------|---------|
| `@Controller('path')` | Define controller with route prefix |
| `@Get()`, `@Post()`, etc. | HTTP method handlers |
| `@Body()` | Extract request body |
| `@Param('id')` | Extract URL parameter |
| `@Query('name')` | Extract query parameter |
| `@UseGuards(Guard)` | Apply guard |
| `@Injectable()` | Mark as injectable service |

### CLI Commands

```bash
# Generate module
nest g module medications

# Generate controller
nest g controller medications

# Generate service
nest g service medications

# Generate resource (module + controller + service + DTOs)
nest g resource medications
```

---

## 11. Learning Resources

- [NestJS Documentation](https://docs.nestjs.com)
- [NestJS Fundamentals Course](https://courses.nestjs.com)
- Fireship's "NestJS in 100 Seconds"

---

*Next: [API Design](api-design.md) | [Authentication](authentication.md)*
