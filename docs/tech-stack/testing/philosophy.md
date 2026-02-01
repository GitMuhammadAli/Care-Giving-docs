# Testing Philosophy

> How to think about testing in CareCircle.

---

## The Mental Model

Think of tests like **insurance policies**:

- **Unit Tests** = Home insurance (protects individual items)
- **Integration Tests** = Comprehensive coverage (how things work together)
- **E2E Tests** = Full inspection (the whole house actually functions)

You don't need every type of insurance for everything, but you need the right coverage for the right risks.

---

## The Testing Pyramid

```
                          ╱╲
                         ╱  ╲
                        ╱ E2E╲
                       ╱──────╲
                      ╱  Slow  ╲
                     ╱  Costly  ╲
                    ╱  Few tests ╲
                   ╱──────────────╲
                  ╱  Integration   ╲
                 ╱  Medium speed    ╲
                ╱  Some complexity   ╲
               ╱──────────────────────╲
              ╱         Unit           ╲
             ╱  Fast, cheap, many tests ╲
            ╱──────────────────────────────╲
            
INVEST MORE ────────────────────────► INVEST LESS
AT THE BASE                           AT THE TOP
```

### Why This Shape?

| Level | Cost | Speed | Flakiness | Confidence |
|-------|------|-------|-----------|------------|
| Unit | Low | Fast | Low | Specific behavior |
| Integration | Medium | Medium | Medium | Components work together |
| E2E | High | Slow | High | System works as user sees |

---

## What to Test (And What NOT to Test)

### The Decision Matrix

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           WHAT TO TEST                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ✅ DEFINITELY TEST                                                         │
│  ─────────────────                                                          │
│  • Business logic (calculations, validations, transformations)              │
│  • Edge cases (empty arrays, null values, boundary conditions)              │
│  • Error handling (what happens when things go wrong)                       │
│  • Security-critical code (auth, authorization, input validation)           │
│  • Complex algorithms (sorting, filtering, data processing)                 │
│                                                                              │
│  ⚠️  TEST SELECTIVELY                                                       │
│  ───────────────────                                                        │
│  • API endpoints (integration, not every permutation)                       │
│  • Database queries (test complex ones, skip simple CRUD)                   │
│  • UI components (test behavior, not every visual state)                    │
│                                                                              │
│  ❌ DON'T TEST                                                              │
│  ────────────                                                               │
│  • Framework code (React, NestJS, Prisma already tested)                   │
│  • Simple getters/setters                                                   │
│  • Configuration files                                                      │
│  • Third-party libraries                                                    │
│  • Implementation details (HOW, not WHAT)                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### The Golden Rule

```
TEST BEHAVIOR, NOT IMPLEMENTATION

❌ WRONG: Testing implementation details

test('should call setState with user', () => {
  const setStateSpy = jest.spyOn(component, 'setState');
  component.login(user);
  expect(setStateSpy).toHaveBeenCalledWith({ user });
});
// Test breaks if we refactor to use hooks


✅ RIGHT: Testing behavior

test('should display user name after login', () => {
  render(<App />);
  login(mockUser);
  expect(screen.getByText(mockUser.name)).toBeInTheDocument();
});
// Test passes regardless of how state is managed
```

---

## Unit Testing

### What Is a Unit?

```
A "unit" is the smallest piece that makes sense to test in isolation.

For CareCircle:
• A service method
• A utility function
• A React hook
• A validation rule

NOT a unit:
• A controller (depends on HTTP, guards, pipes)
• A full component tree (that's integration)
• An API endpoint (that's integration)
```

### Good Unit Test Characteristics

```
F - Fast (milliseconds, not seconds)
I - Independent (no test depends on another)
R - Repeatable (same result every time)
S - Self-validating (pass or fail, no manual check)
T - Timely (written with the code, not after)
```

### Example: Testing a Service Method

```typescript
// The service we're testing
class MedicationService {
  calculateNextDose(lastDose: Date, frequencyHours: number): Date {
    return addHours(lastDose, frequencyHours);
  }
  
  isDoseOverdue(scheduledTime: Date): boolean {
    return scheduledTime < new Date();
  }
}

// The tests
describe('MedicationService', () => {
  describe('calculateNextDose', () => {
    it('should add frequency hours to last dose', () => {
      const service = new MedicationService();
      const lastDose = new Date('2026-01-30T08:00:00Z');
      
      const nextDose = service.calculateNextDose(lastDose, 8);
      
      expect(nextDose).toEqual(new Date('2026-01-30T16:00:00Z'));
    });
    
    it('should handle crossing midnight', () => {
      const service = new MedicationService();
      const lastDose = new Date('2026-01-30T22:00:00Z');
      
      const nextDose = service.calculateNextDose(lastDose, 4);
      
      expect(nextDose).toEqual(new Date('2026-01-31T02:00:00Z'));
    });
  });
  
  describe('isDoseOverdue', () => {
    it('should return true if scheduled time is in the past', () => {
      const service = new MedicationService();
      const pastTime = new Date(Date.now() - 3600000); // 1 hour ago
      
      expect(service.isDoseOverdue(pastTime)).toBe(true);
    });
    
    it('should return false if scheduled time is in the future', () => {
      const service = new MedicationService();
      const futureTime = new Date(Date.now() + 3600000); // 1 hour from now
      
      expect(service.isDoseOverdue(futureTime)).toBe(false);
    });
  });
});
```

---

## Integration Testing

### When to Write Integration Tests

```
INTEGRATION TESTS answer:
"Do these components work together correctly?"

Write integration tests for:
• API endpoints (controller → service → database)
• Database queries (Prisma → PostgreSQL)
• External service calls (our code → third-party API)
• Module interactions (auth module → family module)

Don't write integration tests for:
• Pure functions (unit test instead)
• Single components in isolation (unit test instead)
• Every possible input combination (too expensive)
```

### Example: API Integration Test

```typescript
describe('POST /medications', () => {
  let app: INestApplication;
  let prisma: PrismaService;
  let authToken: string;
  let testFamily: Family;

  beforeAll(async () => {
    // Setup test application
    const module = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();
    
    app = module.createNestApplication();
    await app.init();
    
    prisma = module.get<PrismaService>(PrismaService);
    
    // Create test data
    testFamily = await prisma.family.create({ ... });
    authToken = await getTestAuthToken(testFamily.adminId);
  });

  afterAll(async () => {
    // Cleanup
    await prisma.family.delete({ where: { id: testFamily.id } });
    await app.close();
  });

  it('should create medication for family member', async () => {
    const response = await request(app.getHttpServer())
      .post('/medications')
      .set('Authorization', `Bearer ${authToken}`)
      .send({
        name: 'Aspirin',
        dosage: '100mg',
        frequency: 'DAILY',
        careRecipientId: testFamily.careRecipientId,
      });
    
    expect(response.status).toBe(201);
    expect(response.body.name).toBe('Aspirin');
    
    // Verify in database
    const created = await prisma.medication.findUnique({
      where: { id: response.body.id }
    });
    expect(created).not.toBeNull();
  });

  it('should reject request from non-family member', async () => {
    const outsiderToken = await getTestAuthToken(nonMemberUserId);
    
    const response = await request(app.getHttpServer())
      .post('/medications')
      .set('Authorization', `Bearer ${outsiderToken}`)
      .send({ ... });
    
    expect(response.status).toBe(403);
  });
});
```

---

## E2E Testing

### The Philosophy

```
E2E tests verify the COMPLETE user journey:
  Browser → Frontend → API → Database → and back

E2E tests are:
  ✅ The closest to real user behavior
  ✅ Catch integration issues unit tests miss
  
  ❌ Slow (minutes, not seconds)
  ❌ Flaky (network, timing, browser quirks)
  ❌ Expensive to maintain
  
RULE: Write FEW, CRITICAL E2E tests for happy paths
      Don't try to cover every edge case with E2E
```

### What to Test E2E

```
CRITICAL USER JOURNEYS:
───────────────────────

1. Authentication Flow
   Register → Verify Email → Login → Access Protected Route

2. Core Feature Happy Path
   Login → Add Medication → See it in List → Log a Dose

3. Error Recovery
   Network failure → Retry → Success

DO NOT E2E TEST:
────────────────

• Every form validation (unit test the validation logic)
• Every UI state (Storybook for visual testing)
• Every API error code (integration tests)
```

---

## Testing Patterns for CareCircle

### Pattern 1: Arrange-Act-Assert (AAA)

```typescript
test('should send reminder for overdue medication', async () => {
  // ARRANGE - Set up preconditions
  const medication = await createTestMedication({
    scheduledTime: hourAgo(),
  });
  const notificationService = mockNotificationService();

  // ACT - Perform the action
  await reminderService.checkOverdueMedications();

  // ASSERT - Verify the outcome
  expect(notificationService.send).toHaveBeenCalledWith({
    userId: medication.userId,
    message: expect.stringContaining(medication.name),
  });
});
```

### Pattern 2: Test Data Builders

```typescript
// Instead of verbose object creation:
❌ const medication = {
  id: 'test-id',
  name: 'Aspirin',
  dosage: '100mg',
  frequency: 'DAILY',
  careRecipientId: 'test-recipient',
  createdAt: new Date(),
  updatedAt: new Date(),
  // ... 20 more fields
};

// Use a builder:
✅ const medication = medicationBuilder()
  .withName('Aspirin')
  .overdue()
  .build();

// The builder handles all defaults
```

### Pattern 3: Mock at the Boundary

```typescript
// Mock external services, not internal implementation

❌ WRONG: Mock internal method
jest.spyOn(service, 'privateMethod');

✅ RIGHT: Mock external boundary
const mockEmailClient = {
  send: jest.fn().mockResolvedValue({ messageId: '123' }),
};

// Inject mock via DI
const service = new NotificationService(mockEmailClient);
```

---

## Testing Anti-patterns

### Anti-pattern 1: Testing Implementation Details

```typescript
❌ BAD: Breaks when implementation changes

test('should call repository.findOne', () => {
  service.getMedication('123');
  expect(repository.findOne).toHaveBeenCalled();
});
// What if we switch to findUnique? Test breaks.


✅ GOOD: Tests behavior/outcome

test('should return medication by ID', async () => {
  const medication = await service.getMedication('123');
  expect(medication.id).toBe('123');
});
// Implementation can change, test still passes
```

### Anti-pattern 2: Over-mocking

```typescript
❌ BAD: Everything is mocked

test('should create medication', () => {
  mockPrisma.medication.create.mockResolvedValue(mockMedication);
  mockValidator.validate.mockReturnValue(true);
  mockEventEmitter.emit.mockReturnValue(undefined);
  // ... 10 more mocks
  
  // What are we even testing at this point?
});


✅ GOOD: Mock only external boundaries

test('should create medication', async () => {
  // Real service, real validation
  // Only mock database and external services
  const medication = await service.create(validDto);
  expect(mockPrisma.medication.create).toHaveBeenCalledWith({
    data: expect.objectContaining({ name: validDto.name }),
  });
});
```

### Anti-pattern 3: Non-deterministic Tests

```typescript
❌ BAD: Depends on current time

test('should be overdue', () => {
  const scheduledTime = new Date('2026-01-30T08:00:00Z');
  expect(service.isOverdue(scheduledTime)).toBe(true);
  // Fails when run on different days!
});


✅ GOOD: Control time in test

test('should be overdue when past scheduled time', () => {
  jest.useFakeTimers();
  jest.setSystemTime(new Date('2026-01-30T10:00:00Z'));
  
  const scheduledTime = new Date('2026-01-30T08:00:00Z');
  expect(service.isOverdue(scheduledTime)).toBe(true);
  
  jest.useRealTimers();
});
```

---

## Code Coverage: A Balanced View

### The Misconception

```
"100% code coverage means the code is well-tested"

WRONG!

Coverage tells you: "This line was executed during tests"
Coverage does NOT tell you:
  • If you tested the right things
  • If your assertions are meaningful
  • If edge cases are covered
  • If the behavior is correct

Example of 100% coverage, 0% value:

test('medications service', () => {
  const service = new MedicationsService(mockPrisma);
  service.create({ name: 'Test' });
  // No assertion! But line was "covered"
});
```

### Useful Coverage Guidelines

```
AIM FOR:
────────
• 80% coverage as a baseline (diminishing returns after)
• 100% coverage on critical paths (auth, payments, health logic)
• Focus on branch coverage (all if/else paths)

IGNORE COVERAGE FOR:
───────────────────
• Configuration files
• Simple DTOs
• Generated code
• Framework boilerplate
```

---

## Decision Flowchart: What Test to Write?

```
                    ┌─────────────────────────────────────┐
                    │   I need to test this feature...    │
                    └─────────────────┬───────────────────┘
                                      │
                    ┌─────────────────▼───────────────────┐
                    │  Is it a pure function/calculation? │
                    └─────────────────┬───────────────────┘
                                      │
              ┌───────────YES─────────┴─────────NO────────────┐
              │                                                │
              ▼                                                ▼
    ┌─────────────────┐              ┌─────────────────────────────────┐
    │   UNIT TEST     │              │  Does it involve multiple       │
    │                 │              │  components working together?   │
    │  Fast, focused  │              └──────────────┬──────────────────┘
    └─────────────────┘                             │
                                    ┌───────YES─────┴─────NO──────────┐
                                    │                                  │
                                    ▼                                  ▼
                       ┌─────────────────────┐         ┌─────────────────────┐
                       │ INTEGRATION TEST    │         │  Is it a critical   │
                       │                     │         │  user journey?      │
                       │ Tests API endpoint, │         └──────────┬──────────┘
                       │ module interaction  │                    │
                       └─────────────────────┘       ┌────YES─────┴────NO────┐
                                                     │                       │
                                                     ▼                       ▼
                                          ┌─────────────────┐    ┌────────────────┐
                                          │    E2E TEST     │    │  Maybe don't   │
                                          │                 │    │  need a test   │
                                          │ Full user flow  │    │                │
                                          └─────────────────┘    │  Or unit test  │
                                                                 │  the pieces    │
                                                                 └────────────────┘
```

---

## Quick Reference

### Test File Naming

```
component.ts        → component.test.ts       (unit)
component.ts        → component.spec.ts       (alternative)
                   → component.integration.ts (integration)
                   → component.e2e.ts         (e2e)
```

### Common Jest Commands

```bash
# Run all tests
npm test

# Run tests in watch mode
npm test -- --watch

# Run specific test file
npm test -- medications.service.test.ts

# Run with coverage
npm test -- --coverage

# Run only tests matching pattern
npm test -- -t "should create medication"
```

### Assertion Cheatsheet

```typescript
// Equality
expect(value).toBe(expected);           // Strict equality
expect(value).toEqual(expected);        // Deep equality

// Truthiness
expect(value).toBeTruthy();
expect(value).toBeFalsy();
expect(value).toBeNull();
expect(value).toBeDefined();

// Numbers
expect(value).toBeGreaterThan(3);
expect(value).toBeLessThanOrEqual(10);

// Strings
expect(string).toMatch(/pattern/);
expect(string).toContain('substring');

// Arrays/Objects
expect(array).toContain(item);
expect(array).toHaveLength(3);
expect(object).toHaveProperty('key');

// Exceptions
expect(() => fn()).toThrow();
expect(() => fn()).toThrow('message');

// Async
await expect(promise).resolves.toBe(value);
await expect(promise).rejects.toThrow();

// Mocks
expect(mockFn).toHaveBeenCalled();
expect(mockFn).toHaveBeenCalledWith(arg);
expect(mockFn).toHaveBeenCalledTimes(2);
```

---

*Next: [Unit Testing Patterns](unit-testing.md) | [Integration Testing Guide](integration-testing.md)*


