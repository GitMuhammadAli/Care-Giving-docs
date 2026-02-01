# Architecture Principles

> The foundational decisions that shape CareCircle's codebase.

---

## The Big Picture: Why These Principles?

CareCircle is a **healthcare-adjacent application**. This means:

- **Data accuracy matters** - Wrong medication time could harm someone
- **Reliability matters** - Missing an emergency alert is unacceptable
- **Security matters** - Health information is sensitive
- **Auditability matters** - Need to know who did what and when

These constraints drove our architectural decisions.

---

## Principle 1: Separation of Concerns

### The Mental Model

Think of a well-organized hospital:

```
Reception         →  Nurses' Station  →  Operating Room
(handles intake)     (coordinates)       (specialized work)

                  ≈

Controller        →  Service          →  Repository/Database
(handles request)    (coordinates)       (data operations)
```

Each layer has ONE job and does it well.

### Why It Matters

```
WITHOUT SEPARATION:
───────────────────

@Controller()
class MedicationsController {
  @Post()
  async create(@Body() dto) {
    // Validate input
    if (!dto.name) throw new Error('Name required');
    
    // Check authorization
    const user = await this.db.user.findUnique({...});
    if (!user.familyIds.includes(dto.familyId)) {
      throw new ForbiddenException();
    }
    
    // Check business rules
    const existing = await this.db.medication.count({...});
    if (existing > 100) throw new Error('Too many medications');
    
    // Create the medication
    const medication = await this.db.medication.create({...});
    
    // Send notification
    await this.pushService.send({...});
    
    // Log audit
    await this.db.auditLog.create({...});
    
    return medication;
  }
}

Problems:
• 50+ lines per endpoint
• Can't test business logic without HTTP
• Duplicate logic across endpoints
• Hard to find where things happen


WITH SEPARATION:
────────────────

@Controller()
class MedicationsController {
  @Post()
  @UseGuards(FamilyAccessGuard)  // Auth handled by guard
  async create(@Body() dto: CreateMedicationDto) {  // Validation by pipe
    return this.medicationsService.create(dto);  // Logic in service
  }
}

@Injectable()
class MedicationsService {
  async create(dto: CreateMedicationDto) {
    this.validateBusinessRules(dto);
    const medication = await this.repository.create(dto);
    this.eventEmitter.emit('medication.created', medication);
    return medication;
  }
}

Benefits:
• Controller: 5 lines, one responsibility
• Service: testable without HTTP
• Guards: reusable auth logic
• Events: decoupled side effects
```

### The Layer Rules

| Layer | Can Call | Cannot Call |
|-------|----------|-------------|
| Controller | Service | Repository, External APIs |
| Service | Repository, Other Services, Event Emitter | Controller |
| Repository | Database | Service, Controller |
| Guard | Service | Repository directly |

---

## Principle 2: Dependency Injection

### The Mental Model

Think of it like a **tool rental shop**:

```
WITHOUT DI (You buy your own tools):
────────────────────────────────────

class Carpenter {
  constructor() {
    this.hammer = new Hammer();        // Carpenter buys hammer
    this.saw = new ElectricSaw();      // Carpenter buys saw
    // What if you want a different saw? Buy a new Carpenter!
  }
}


WITH DI (Shop provides tools):
──────────────────────────────

class Carpenter {
  constructor(hammer: Hammer, saw: Saw) {  // Shop provides
    this.hammer = hammer;
    this.saw = saw;
  }
}

// Now you can provide any saw: ElectricSaw, HandSaw, TestSaw...
```

### Why It Matters for Testing

```typescript
// PRODUCTION: Real database
const medicationsService = new MedicationsService(
  new PrismaService(),  // Real database
  new NotificationService(),  // Real notifications
);

// TESTING: Mock everything
const medicationsService = new MedicationsService(
  mockPrismaService,      // In-memory fake
  mockNotificationService, // Records calls but doesn't send
);

// Now you can test business logic WITHOUT a database!
```

### How CareCircle Uses DI

```typescript
// 1. Services declare what they need
@Injectable()
export class MedicationsService {
  constructor(
    private prisma: PrismaService,      // "I need database access"
    private cache: CacheService,        // "I need cache access"
    private events: EventEmitter2,      // "I need to emit events"
  ) {}
}

// 2. Module declares what's available
@Module({
  imports: [
    PrismaModule,    // Provides PrismaService
    CacheModule,     // Provides CacheService
    EventModule,     // Provides EventEmitter2
  ],
  providers: [MedicationsService],  // Now NestJS can wire it up
})
export class MedicationsModule {}

// 3. NestJS handles the rest automatically
```

---

## Principle 3: Event-Driven Side Effects

### The Problem with Direct Calls

```typescript
// The "Telephone Game" anti-pattern
async createEmergencyAlert(data) {
  const alert = await this.createAlert(data);
  
  // Service knows about ALL side effects
  await this.notificationService.sendPush(alert);
  await this.emailService.sendUrgentEmail(alert);
  await this.smsService.sendText(alert);
  await this.auditService.logEmergency(alert);
  await this.analyticsService.trackAlert(alert);
  await this.slackService.notifyOnCall(alert);
  
  // Adding new side effect = modifying this service
  // Testing = mocking 6 services
  // Failure in one = failure in all
}
```

### The Event-Driven Solution

```typescript
// Emergency service only knows about emergencies
async createEmergencyAlert(data) {
  const alert = await this.createAlert(data);
  this.eventEmitter.emit('emergency.created', alert);  // That's it!
  return alert;
}

// Each side effect is independent
@OnEvent('emergency.created')
class NotificationHandler {
  handle(alert) { this.sendPush(alert); }
}

@OnEvent('emergency.created')
class EmailHandler {
  handle(alert) { this.sendEmail(alert); }
}

@OnEvent('emergency.created')
class AuditHandler {
  handle(alert) { this.logAudit(alert); }
}
```

### Benefits of Event-Driven

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      EVENT-DRIVEN BENEFITS                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  LOOSE COUPLING                      │  EXTENSIBILITY                       │
│  ──────────────                      │  ─────────────                       │
│  Alert service doesn't know          │  Add new listener without            │
│  about notifications                 │  changing existing code              │
│                                                                              │
│  FAILURE ISOLATION                   │  TESTABILITY                         │
│  ─────────────────                   │  ───────────                         │
│  Email failure doesn't               │  Test alert creation without         │
│  break alert creation                │  testing notifications               │
│                                                                              │
│  ASYNC PROCESSING                    │  AUDITABILITY                        │
│  ────────────────                    │  ────────────                        │
│  Push to queue, respond fast         │  All events can be logged            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Principle 4: Explicit Over Implicit

### The Anti-Pattern: Magic

```typescript
// Implicit/Magic behavior
class MedicationService {
  @AutoCache()  // What does this do? When does cache invalidate?
  @LogActivity()  // What's logged? Where?
  @CheckPermission()  // What permission? How?
  async getMedication(id: string) {
    return this.db.medication.findUnique({ where: { id } });
  }
}

// Developer questions:
// - Is this cached? For how long?
// - What gets logged?
// - What happens if permission check fails?
// - In what order do decorators run?
```

### The Principle: Be Explicit

```typescript
// Explicit behavior
class MedicationService {
  async getMedication(id: string, userId: string) {
    // Explicit permission check
    const hasAccess = await this.checkFamilyAccess(userId, id);
    if (!hasAccess) {
      throw new ForbiddenException('Not a family member');
    }
    
    // Explicit caching with clear TTL
    const cacheKey = `medication:${id}`;
    const cached = await this.cache.get(cacheKey);
    if (cached) return cached;
    
    // Explicit data fetch
    const medication = await this.db.medication.findUnique({ 
      where: { id } 
    });
    
    // Explicit cache write
    await this.cache.set(cacheKey, medication, 300);  // 5 min TTL
    
    return medication;
  }
}

// Now every developer can:
// - See exactly what happens
// - Debug step by step
// - Modify behavior confidently
```

### Where We Allow "Magic" (Intentionally)

| Pattern | Where Used | Why It's OK |
|---------|------------|-------------|
| Validation decorators | DTOs | Standard NestJS pattern, well-documented |
| Auth guards | Controllers | Single responsibility, explicit application |
| Prisma relations | Queries | Type-safe, explicit in schema |

---

## Principle 5: Fail Fast, Fail Loud

### The Anti-Pattern: Silent Failures

```typescript
// Silent failure - DANGEROUS
async sendReminder(medicationId: string) {
  try {
    await this.pushService.send(notification);
  } catch (error) {
    // Silently ignore! User never gets reminder!
  }
}
```

### The Principle: Errors Should Be Visible

```typescript
// Fail loud - SAFE
async sendReminder(medicationId: string) {
  try {
    await this.pushService.send(notification);
  } catch (error) {
    // Log with full context
    this.logger.error('Failed to send medication reminder', {
      medicationId,
      error: error.message,
      stack: error.stack,
    });
    
    // Alert on-call if critical
    if (isCritical(medicationId)) {
      await this.alerting.notifyOnCall('Reminder delivery failed');
    }
    
    // Re-throw so caller knows
    throw new ReminderDeliveryException(medicationId, error);
  }
}
```

### Error Handling Hierarchy

```
EXPECTED ERRORS (Handle gracefully):
────────────────────────────────────
• User not found → 404, helpful message
• Validation failed → 400, field errors
• Permission denied → 403, explain why
• Conflict → 409, suggest resolution

UNEXPECTED ERRORS (Fail loud):
──────────────────────────────
• Database connection lost → 500, log, alert
• External API down → 500, log, alert, queue for retry
• Null pointer → 500, log with stack trace

NEVER DO:
─────────
• Empty catch blocks
• Generic error messages hiding real cause
• Swallowing errors without logging
```

---

## Principle 6: Security by Default

### The Mindset

```
WRONG: "This route doesn't need auth, I'll add @Public()"
RIGHT: "This route doesn't have auth, is that intentional?"

Implementation:
• Global JwtAuthGuard (everything requires auth by default)
• @Public() decorator explicitly opts out
• Review all @Public() routes regularly
```

### Defense in Depth

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        SECURITY LAYERS                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Layer 1: Network                                                           │
│  • HTTPS everywhere                                                         │
│  • CORS configured properly                                                 │
│  • Rate limiting                                                            │
│                                                                              │
│  Layer 2: Authentication                                                    │
│  • JWT validation                                                           │
│  • Token expiration                                                         │
│  • Refresh token rotation                                                   │
│                                                                              │
│  Layer 3: Authorization                                                     │
│  • Family membership check                                                  │
│  • Role-based access (ADMIN, CAREGIVER, VIEWER)                            │
│  • Resource ownership verification                                          │
│                                                                              │
│  Layer 4: Input Validation                                                  │
│  • DTO validation (class-validator)                                         │
│  • SQL injection prevention (Prisma)                                        │
│  • XSS prevention (sanitization)                                            │
│                                                                              │
│  Layer 5: Data                                                              │
│  • Encrypted at rest                                                        │
│  • Encrypted in transit                                                     │
│  • PII handling policies                                                    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Principle 7: Monorepo with Clear Boundaries

### The Structure

```
Care-Giving/
├── apps/
│   ├── api/           # NestJS backend
│   ├── web/           # Next.js frontend
│   └── workers/       # Background job processors
│
├── packages/
│   ├── database/      # Prisma schema & client
│   ├── config/        # Shared configuration
│   └── logger/        # Shared logging
│
└── env/               # Environment configs
```

### The Rules

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      MONOREPO DEPENDENCY RULES                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  APPS can import from:                                                       │
│  • packages/*                                                                │
│  • node_modules                                                              │
│                                                                              │
│  APPS cannot import from:                                                    │
│  • Other apps (❌ api cannot import from web)                               │
│                                                                              │
│  PACKAGES can import from:                                                   │
│  • Other packages                                                            │
│  • node_modules                                                              │
│                                                                              │
│  PACKAGES cannot import from:                                                │
│  • apps/* (❌ database cannot import from api)                              │
│                                                                              │
│                                                                              │
│  Visual:                                                                     │
│                                                                              │
│      api  ←──────────────────┐                                              │
│       │                      │                                              │
│       ▼                      │                                              │
│    packages/database ◄───────┤                                              │
│       ▲                      │                                              │
│       │                      │                                              │
│     web  ←───────────────────┤                                              │
│       │                      │                                              │
│       ▼                      │                                              │
│    packages/config ◄─────────┘                                              │
│                                                                              │
│  (Arrows show allowed import direction)                                     │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Why This Structure?

| Benefit | Explanation |
|---------|-------------|
| **Shared types** | Prisma types used by both API and workers |
| **Consistent config** | Same env validation everywhere |
| **Independent deploys** | Can deploy API without touching workers |
| **Clear ownership** | Each app has clear responsibility |

---

## Quick Reference: Decision Checklist

### Before Writing Code, Ask:

```
□ Which layer does this belong in? (Controller/Service/Repository)
□ What are the dependencies? Can they be injected?
□ What side effects does this have? Should they be events?
□ What can go wrong? How will I handle each case?
□ Who can access this? What permissions are needed?
□ Is there existing code I should reuse?
□ How will I test this?
```

### Code Review Checklist:

```
□ Is business logic in services, not controllers?
□ Are dependencies injected, not constructed?
□ Are side effects event-driven where appropriate?
□ Are errors handled explicitly with good messages?
□ Is auth applied? If not, is @Public() intentional?
□ Does it follow existing patterns in the codebase?
```

---

*Next: [Separation of Concerns](separation-of-concerns.md) | [Event-Driven Design](event-driven.md)*


