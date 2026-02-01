# Backend Architecture Overview

> Understanding how the server-side pieces fit together.

---

## The Mental Model

Think of the backend like a **restaurant kitchen**:

- **Controllers** = The order window (receives requests, sends responses)
- **Services** = The chefs (actual cooking/business logic)
- **Repositories/Prisma** = The pantry (where ingredients/data are stored)
- **Guards** = The bouncer (checks if you're allowed in)
- **Pipes** = The food inspector (validates what comes in)
- **Interceptors** = The food stylist (transforms what goes out)

---

## Request Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            HTTP REQUEST                                      │
│                                  │                                           │
│                    ┌─────────────▼─────────────┐                            │
│                    │       MIDDLEWARE          │                            │
│                    │   • Helmet (security)     │                            │
│                    │   • CORS                  │                            │
│                    │   • Rate limiting         │                            │
│                    │   • Body parsing          │                            │
│                    └─────────────┬─────────────┘                            │
│                                  │                                           │
│                    ┌─────────────▼─────────────┐                            │
│                    │         GUARDS            │                            │
│                    │   • JwtAuthGuard          │                            │
│                    │   • FamilyAccessGuard     │                            │
│                    │   • RolesGuard            │                            │
│                    │                           │                            │
│                    │   Can this user access    │                            │
│                    │   this resource?          │                            │
│                    └─────────────┬─────────────┘                            │
│                              YES │                                           │
│                    ┌─────────────▼─────────────┐                            │
│                    │         PIPES             │                            │
│                    │   • ValidationPipe        │                            │
│                    │   • ParseUUIDPipe         │                            │
│                    │                           │                            │
│                    │   Is the input valid?     │                            │
│                    └─────────────┬─────────────┘                            │
│                              YES │                                           │
│                    ┌─────────────▼─────────────┐                            │
│                    │       CONTROLLER          │                            │
│                    │                           │                            │
│                    │   Route handler           │                            │
│                    │   Receives request        │                            │
│                    │   Calls service           │                            │
│                    └─────────────┬─────────────┘                            │
│                                  │                                           │
│                    ┌─────────────▼─────────────┐                            │
│                    │         SERVICE           │                            │
│                    │                           │                            │
│                    │   Business logic          │                            │
│                    │   Data transformation     │                            │
│                    │   Coordination            │                            │
│                    └─────────────┬─────────────┘                            │
│                                  │                                           │
│         ┌────────────────────────┼────────────────────────┐                 │
│         │                        │                        │                 │
│         ▼                        ▼                        ▼                 │
│  ┌─────────────┐         ┌─────────────┐         ┌─────────────┐           │
│  │   Prisma    │         │   Redis     │         │  RabbitMQ   │           │
│  │  Database   │         │   Cache     │         │   Events    │           │
│  └─────────────┘         └─────────────┘         └─────────────┘           │
│                                  │                                           │
│                    ┌─────────────▼─────────────┐                            │
│                    │      INTERCEPTORS         │                            │
│                    │   • TransformInterceptor  │                            │
│                    │   • LoggingInterceptor    │                            │
│                    │                           │                            │
│                    │   Format the response     │                            │
│                    └─────────────┬─────────────┘                            │
│                                  │                                           │
│                    ┌─────────────▼─────────────┐                            │
│                    │      HTTP RESPONSE        │                            │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## The Module System (Why NestJS?)

### The Problem with Express

Express gives you freedom, but freedom can lead to chaos:

```
Express Project at Scale:
  src/
    routes/
      users.js        # Some business logic here
      medications.js  # Some here too
    controllers/      # Wait, also logic here?
    services/         # More logic...
    utils/            # Random shared stuff
    helpers/          # What's the difference from utils?
    
Where do I put new code? 🤷
```

### NestJS's Solution: Enforced Structure

```
NestJS Module = A self-contained feature package

┌─────────────────────────────────────────────────────────────────┐
│                     MEDICATIONS MODULE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Controller          Service              DTOs                  │
│   ──────────          ───────              ────                  │
│   Route handlers      Business logic       Request shapes        │
│   Input/output        Orchestration        Validation rules      │
│                       Data access                                │
│                                                                  │
│   MedicationsController → MedicationsService → Prisma            │
│                                                                  │
│   Exports: MedicationsService (for other modules to use)        │
│   Imports: PrismaModule, NotificationsModule                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Mental Model: Modules as Lego Blocks

```
                    AppModule (Root)
                         │
         ┌───────────────┼───────────────┐
         │               │               │
    AuthModule     FamilyModule    MedicationsModule
         │               │               │
    ┌────┴────┐     ┌────┴────┐     ┌────┴────┐
    Controller   Controller     Controller
    Service      Service        Service
    Guards       Guards         DTOs
```

Each module:
- Has clear boundaries
- Declares its dependencies
- Exports what others can use
- Can be tested in isolation

---

## Dependency Injection (The Most Important Concept)

### What Is It?

**Dependency Injection** = Don't create your own tools, ask for them.

```
WITHOUT Dependency Injection:
─────────────────────────────

class ChefService {
  cook() {
    const pantry = new PantryService();  // Chef creates own pantry
    const oven = new OvenService();      // Chef creates own oven
    // What if we want a different oven for testing?
    // What if two chefs need to share the same pantry?
  }
}


WITH Dependency Injection:
──────────────────────────

class ChefService {
  constructor(
    private pantry: PantryService,  // Pantry provided from outside
    private oven: OvenService       // Oven provided from outside
  ) {}
  
  cook() {
    // Just use what you're given
  }
}

// NestJS creates and provides the dependencies automatically
```

### Why Does This Matter?

| Benefit | Explanation |
|---------|-------------|
| **Testability** | Replace real DB with mock DB for tests |
| **Flexibility** | Swap implementations without changing code |
| **Singleton management** | One database connection shared by all |
| **Clear dependencies** | Constructor tells you what's needed |

### How NestJS Does It

```typescript
// 1. Mark class as injectable
@Injectable()
class MedicationsService {
  constructor(private prisma: PrismaService) {}  // Asks for Prisma
}

// 2. Register in module
@Module({
  imports: [PrismaModule],  // Makes PrismaService available
  providers: [MedicationsService],  // NestJS will inject Prisma
})
class MedicationsModule {}

// 3. NestJS handles the rest
// When MedicationsService is needed, NestJS:
// - Finds PrismaService instance
// - Creates MedicationsService with Prisma injected
// - Caches and reuses (singleton by default)
```

---

## Layers & Responsibilities

### The Layer Cake

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         CONTROLLER LAYER                                     │
│                                                                              │
│  Responsibilities:                  │  Should NOT do:                        │
│  • Define routes                    │  • Business logic                      │
│  • Handle HTTP concerns             │  • Direct database access              │
│  • Call service methods             │  • Complex validation                  │
│  • Return responses                 │  • Data transformation                 │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           SERVICE LAYER                                      │
│                                                                              │
│  Responsibilities:                  │  Should NOT do:                        │
│  • Business logic                   │  • HTTP-specific code                  │
│  • Orchestrating operations         │  • Return HTTP responses               │
│  • Authorization checks             │  • Deal with request/response          │
│  • Calling other services           │                                        │
│  • Event emission                   │                                        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          DATA ACCESS LAYER                                   │
│                                                                              │
│  (Prisma in our case)                                                       │
│                                                                              │
│  Responsibilities:                  │  Should NOT do:                        │
│  • Database queries                 │  • Business logic                      │
│  • Data persistence                 │  • Authorization checks                │
│  • Transactions                     │  • HTTP concerns                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Example: Creating a Medication

```
Controller:
  1. Receives POST /medications with body
  2. NestJS validates body against DTO (via ValidationPipe)
  3. Calls medicationsService.create(userId, dto)
  4. Returns result to client

Service:
  1. Checks user has permission for this care recipient
  2. Validates business rules (max medications, interactions, etc.)
  3. Creates medication via Prisma
  4. Emits 'medication.created' event
  5. Returns created medication

Prisma:
  1. Generates SQL INSERT
  2. Executes against PostgreSQL
  3. Returns new record
```

---

## Guards: Authorization Made Clear

### The Concept

Guards answer: **"Can this request proceed?"**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           GUARD DECISION TREE                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Request arrives                                                             │
│       │                                                                      │
│       ▼                                                                      │
│  ┌─────────────────┐                                                        │
│  │ JwtAuthGuard    │  Is there a valid JWT?                                 │
│  └────────┬────────┘                                                        │
│           │                                                                  │
│       NO  │  YES                                                            │
│       ↓   ↓                                                                  │
│    401    ┌─────────────────┐                                               │
│           │FamilyAccessGuard│  Is user a member of this family?             │
│           └────────┬────────┘                                               │
│                    │                                                         │
│                NO  │  YES                                                   │
│                ↓   ↓                                                         │
│             403    ┌─────────────────┐                                      │
│                    │   RolesGuard    │  Does user have required role?       │
│                    └────────┬────────┘                                      │
│                             │                                                │
│                         NO  │  YES                                          │
│                         ↓   ↓                                                │
│                      403    Continue to controller ✓                        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### CareCircle's Guard Strategy

```typescript
// Global guard (applied to all routes by default)
@UseGuards(JwtAuthGuard)  // From AppModule

// Route-level guards
@UseGuards(FamilyAccessGuard)  // Check family membership
@FamilyAccess({ param: 'familyId', roles: [FamilyRole.ADMIN] })
createMedication() { ... }

// Public routes (bypass auth)
@Public()  // Decorator that JwtAuthGuard checks for
register() { ... }
```

---

## Error Handling Philosophy

### The Error Hierarchy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         ERROR CATEGORIES                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  CLIENT ERRORS (4xx) - "You did something wrong"                            │
│  ──────────────────────────────────────────────                             │
│  400 Bad Request     = Invalid input data                                   │
│  401 Unauthorized    = No valid credentials                                 │
│  403 Forbidden       = Valid credentials, but not allowed                   │
│  404 Not Found       = Resource doesn't exist                               │
│  409 Conflict        = Resource already exists (email taken)                │
│  422 Unprocessable   = Valid format, but business rule violation           │
│  429 Too Many Req    = Rate limited                                         │
│                                                                              │
│  SERVER ERRORS (5xx) - "We did something wrong"                             │
│  ──────────────────────────────────────────────                             │
│  500 Internal Error  = Unexpected server problem                            │
│  502 Bad Gateway     = Upstream service failed                              │
│  503 Unavailable     = Server overloaded/maintenance                        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### How We Handle Errors

```typescript
// 1. EXPECTED ERRORS - Throw specific exceptions
throw new NotFoundException('Medication not found');
throw new ForbiddenException('Not a family member');
throw new ConflictException('Email already registered');

// 2. VALIDATION ERRORS - Handled by ValidationPipe
// Automatically returns 400 with field-level errors

// 3. UNEXPECTED ERRORS - Caught by global filter
// Logs error, returns generic 500 to client
// Never exposes stack traces in production
```

### Error Response Format

```json
{
  "success": false,
  "statusCode": 400,
  "message": "Validation failed",
  "errors": [
    { "field": "email", "message": "must be a valid email" },
    { "field": "password", "message": "must be at least 8 characters" }
  ],
  "timestamp": "2026-01-30T10:00:00.000Z"
}
```

---

## The Service Pattern

### What Services Should Look Like

```typescript
@Injectable()
export class MedicationsService {
  // Dependencies injected via constructor
  constructor(
    private prisma: PrismaService,
    private notifications: NotificationsService,
    private eventEmitter: EventEmitter2,
  ) {}

  // Public methods = the "API" of this service
  async create(userId: string, dto: CreateMedicationDto) {
    // 1. Authorization check
    await this.verifyAccess(dto.careRecipientId, userId);
    
    // 2. Business validation
    this.validateMedicationRules(dto);
    
    // 3. Data operation
    const medication = await this.prisma.medication.create({
      data: { ...dto, createdById: userId },
    });
    
    // 4. Side effects
    this.eventEmitter.emit('medication.created', medication);
    
    // 5. Return result
    return medication;
  }

  // Private methods = internal helpers
  private async verifyAccess(careRecipientId: string, userId: string) {
    // Check user can access this care recipient
  }

  private validateMedicationRules(dto: CreateMedicationDto) {
    // Business rule validation
  }
}
```

### Service Anti-patterns

```typescript
❌ WRONG: HTTP concerns in service

async create(req: Request, res: Response) {
  // Services shouldn't know about HTTP
}

❌ WRONG: Direct database queries in controller

@Post()
create(@Body() dto) {
  return this.prisma.medication.create({ data: dto });
  // Business logic belongs in service
}

❌ WRONG: God service that does everything

class AppService {
  createUser() { }
  createMedication() { }
  sendEmail() { }
  // Break into domain-specific services
}
```

---

## Caching Strategy

### When to Cache

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         CACHING DECISION MATRIX                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  CACHE WHEN:                          │  DON'T CACHE WHEN:                  │
│  ───────────                          │  ─────────────────                  │
│  ✅ Data rarely changes               │  ❌ Data changes frequently         │
│  ✅ Read >> Write ratio               │  ❌ Data is user-specific & fresh   │
│  ✅ Query is expensive                │  ❌ Small, fast queries             │
│  ✅ Data is shared across users       │  ❌ Real-time accuracy required     │
│                                       │                                     │
│  Examples:                            │  Examples:                          │
│  • Family member list                 │  • Notification unread count        │
│  • Care recipient details             │  • Real-time emergency status       │
│  • Medication list                    │  • Live shift check-in              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Cache Pattern: Cache-Aside

```typescript
async getMedications(careRecipientId: string) {
  const cacheKey = `medications:${careRecipientId}`;
  
  // 1. Check cache
  const cached = await this.cache.get(cacheKey);
  if (cached) return cached;
  
  // 2. Query database
  const medications = await this.prisma.medication.findMany({
    where: { careRecipientId },
  });
  
  // 3. Store in cache
  await this.cache.set(cacheKey, medications, 300); // 5 min TTL
  
  return medications;
}

// Don't forget to invalidate on write!
async createMedication(careRecipientId: string, dto: CreateMedicationDto) {
  const medication = await this.prisma.medication.create({ ... });
  
  // Invalidate cache
  await this.cache.del(`medications:${careRecipientId}`);
  
  return medication;
}
```

---

## Event-Driven Communication

### Why Events?

Without events:
```typescript
// Service becomes tightly coupled
async createEmergencyAlert(data) {
  const alert = await this.createAlert(data);
  await this.notificationService.sendPush(...);  // Tight coupling
  await this.emailService.sendEmail(...);        // More coupling
  await this.auditService.log(...);              // Even more
  await this.analyticsService.track(...);        // It never ends
}
```

With events:
```typescript
// Service is focused and decoupled
async createEmergencyAlert(data) {
  const alert = await this.createAlert(data);
  this.eventEmitter.emit('emergency.alert.created', alert);  // Fire and forget
  return alert;
}

// Listeners handle side effects independently
@OnEvent('emergency.alert.created')
handleEmergencyForNotifications(alert) { /* send push */ }

@OnEvent('emergency.alert.created')
handleEmergencyForEmail(alert) { /* send email */ }

@OnEvent('emergency.alert.created')
handleEmergencyForAudit(alert) { /* log audit */ }
```

### Event Types in CareCircle

```
Domain Events (RabbitMQ)
├── medication.logged.*
├── emergency.alert.*
├── appointment.*
└── shift.*

Internal Events (EventEmitter)
├── user.created
├── family.member.added
└── medication.refill.needed
```

---

## Quick Reference

### NestJS Decorators Cheatsheet

| Decorator | Purpose | Example |
|-----------|---------|---------|
| `@Controller()` | Define route prefix | `@Controller('medications')` |
| `@Get()`, `@Post()`, etc. | HTTP method | `@Get(':id')` |
| `@Body()` | Request body | `@Body() dto: CreateDto` |
| `@Param()` | URL parameters | `@Param('id') id: string` |
| `@Query()` | Query string | `@Query('limit') limit: number` |
| `@UseGuards()` | Apply guards | `@UseGuards(JwtAuthGuard)` |
| `@Injectable()` | Mark as service | Class decorator |
| `@Module()` | Define module | Module configuration |

### Response Status Convention

| Operation | Success Status | Common Errors |
|-----------|----------------|---------------|
| GET single | 200 | 404 |
| GET list | 200 | - |
| POST create | 201 | 400, 409 |
| PUT/PATCH update | 200 | 400, 404 |
| DELETE | 200 or 204 | 404 |

---

*Next: [NestJS Deep Dive](nestjs.md) | [API Design Principles](api-design.md)*


