# Unit Testing Concepts

> Understanding how to test CareCircle effectively.

---

## 1. What Is Unit Testing?

### Plain English Explanation

Unit testing is **testing small pieces of code in isolation**.

Think of it like **quality control in a factory**:
- Test each component separately
- If a wheel is defective, you know before assembling the car
- Find problems early, fix them cheaply

### The Testing Pyramid

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         TESTING PYRAMID                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                          /\                                                  │
│                         /  \     E2E Tests (few)                            │
│                        /    \    Slow, expensive, test whole system         │
│                       /──────\                                               │
│                      /        \  Integration Tests (some)                   │
│                     /          \ Test components together                   │
│                    /────────────\                                            │
│                   /              \ Unit Tests (many)                        │
│                  /                \ Fast, cheap, test in isolation          │
│                 /──────────────────\                                         │
│                                                                              │
│  MORE TESTS ◄──────────────────────────────────────────────► FEWER TESTS   │
│  FAST, CHEAP ◄────────────────────────────────────────────► SLOW, EXPENSIVE│
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Core Concepts & Terminology

### Test Anatomy

```typescript
describe('MedicationsService', () => {          // Test suite
  let service: MedicationsService;              // System under test

  beforeEach(() => {                            // Setup (runs before each test)
    service = new MedicationsService(mockRepo);
  });

  it('should create a medication', async () => { // Test case
    // Arrange: Set up test data
    const dto = { name: 'Aspirin', dosage: '100mg' };
    
    // Act: Execute the code
    const result = await service.create(dto);
    
    // Assert: Verify the outcome
    expect(result.name).toBe('Aspirin');
  });
});
```

### Key Terminology

| Term | Definition |
|------|------------|
| **Test Suite** | Group of related tests |
| **Test Case** | Single test scenario |
| **Assertion** | Verification statement |
| **Mock** | Fake dependency |
| **Stub** | Predetermined response |
| **Fixture** | Test data setup |
| **Coverage** | % of code executed by tests |

---

## 3. What to Test

### Unit Test Targets

```
✅ TEST THESE:
─────────────
• Business logic (services)
• Data transformations
• Validation rules
• Edge cases
• Error handling

❌ DON'T TEST:
──────────────
• Framework code (NestJS internals)
• Simple getters/setters
• External libraries
• Database queries (integration test)
• Third-party APIs (integration test)
```

### The Right Level of Testing

```typescript
// ❌ TOO LOW: Testing language features
it('should add numbers', () => {
  expect(1 + 1).toBe(2);  // JavaScript already works
});

// ❌ TOO HIGH: Testing whole flow
it('should register, login, create family, add medication', () => {
  // This is an E2E test, not a unit test
});

// ✅ JUST RIGHT: Testing one unit
it('should throw if medication name is empty', () => {
  expect(() => service.create({ name: '' }))
    .toThrow('Name is required');
});
```

---

## 4. Mocking Dependencies

### Why Mock?

```
Real Service:
MedicationsService → Repository → Database → Network

Unit Test:
MedicationsService → Mock Repository
                     └── Returns fake data instantly
                     └── No database needed
                     └── Fast, isolated, predictable
```

### Mocking in Jest

```typescript
// Create mock
const mockRepository = {
  findOne: jest.fn(),
  save: jest.fn(),
  delete: jest.fn(),
};

// Set up return value
mockRepository.findOne.mockResolvedValue({
  id: '123',
  name: 'Aspirin',
});

// Verify calls
expect(mockRepository.findOne).toHaveBeenCalledWith({
  where: { id: '123' }
});
```

### Mocking in NestJS

```typescript
const module = await Test.createTestingModule({
  providers: [
    MedicationsService,
    {
      provide: MedicationsRepository,
      useValue: mockRepository,  // Inject mock instead of real
    },
  ],
}).compile();

service = module.get<MedicationsService>(MedicationsService);
```

---

## 5. When to Use Unit Tests ✅

### Test Business Logic

```typescript
it('should calculate next dose time correctly', () => {
  const medication = {
    frequency: 'TWICE_DAILY',
    scheduledTimes: ['09:00', '21:00'],
  };
  
  const nextDose = service.calculateNextDose(medication);
  
  expect(nextDose).toBe('21:00');
});
```

### Test Edge Cases

```typescript
it('should handle empty medication list', () => {
  const result = service.getActiveMedications([]);
  expect(result).toEqual([]);
});

it('should handle null care recipient', () => {
  expect(() => service.create(null))
    .toThrow('Care recipient is required');
});
```

### Test Error Handling

```typescript
it('should throw NotFoundException when medication not found', async () => {
  mockRepository.findOne.mockResolvedValue(null);
  
  await expect(service.findById('invalid-id'))
    .rejects
    .toThrow(NotFoundException);
});
```

---

## 6. When to AVOID Unit Tests ❌

### DON'T Test Implementation Details

```typescript
// ❌ BAD: Testing private methods
it('should call _validateInput internally', () => {
  const spy = jest.spyOn(service, '_validateInput');
  service.create(dto);
  expect(spy).toHaveBeenCalled();
});

// ✅ GOOD: Test behavior, not implementation
it('should throw on invalid input', () => {
  expect(() => service.create(invalidDto)).toThrow();
});
```

### DON'T Test Trivial Code

```typescript
// ❌ BAD: Testing simple getter
it('should return name', () => {
  user.name = 'John';
  expect(user.name).toBe('John');  // Useless test
});
```

### DON'T Test External Dependencies

```typescript
// ❌ BAD: Testing that bcrypt works
it('should hash password', async () => {
  const hash = await bcrypt.hash('password', 10);
  expect(hash).not.toBe('password');
  // We're testing bcrypt, not our code
});

// ✅ GOOD: Test that WE call bcrypt correctly
it('should hash password before saving', async () => {
  await service.register({ password: 'test123' });
  expect(mockUser.passwordHash).not.toBe('test123');
});
```

---

## 7. Best Practices

### Arrange-Act-Assert (AAA)

```typescript
it('should deactivate expired medications', async () => {
  // ARRANGE: Set up test data
  const expiredMed = { id: '1', endDate: pastDate, isActive: true };
  mockRepository.findExpired.mockResolvedValue([expiredMed]);
  
  // ACT: Execute
  await service.deactivateExpired();
  
  // ASSERT: Verify
  expect(mockRepository.save).toHaveBeenCalledWith(
    expect.objectContaining({ id: '1', isActive: false })
  );
});
```

### One Assertion Per Test (Guideline)

```typescript
// ❌ BAD: Multiple unrelated assertions
it('should create medication', async () => {
  const result = await service.create(dto);
  expect(result.name).toBe('Aspirin');
  expect(result.id).toBeDefined();
  expect(mockRepository.save).toHaveBeenCalled();
  expect(mockEventEmitter.emit).toHaveBeenCalled();
});

// ✅ GOOD: Focused tests
it('should save medication to repository', async () => {
  await service.create(dto);
  expect(mockRepository.save).toHaveBeenCalled();
});

it('should emit medication.created event', async () => {
  await service.create(dto);
  expect(mockEventEmitter.emit).toHaveBeenCalledWith('medication.created', expect.any(Object));
});
```

### Descriptive Test Names

```typescript
// ❌ BAD: Vague names
it('should work');
it('test create');

// ✅ GOOD: Describe behavior
it('should throw BadRequestException when dosage is negative');
it('should return empty array when no active medications exist');
it('should send notification to all family members on emergency alert');
```

---

## 8. Common Mistakes & How to Avoid Them

### Mistake 1: Testing Too Much at Once

Break large tests into smaller, focused ones.

### Mistake 2: Shared Mutable State

```typescript
// ❌ BAD: Shared state between tests
let medications = [];
beforeAll(() => { medications.push(testMed); });

// ✅ GOOD: Fresh state each test
beforeEach(() => { medications = []; });
```

### Mistake 3: Ignoring Async

```typescript
// ❌ BAD: Not awaiting
it('should create', () => {
  service.create(dto);  // Promise not awaited!
  expect(mockRepo.save).toHaveBeenCalled();  // Might fail randomly
});

// ✅ GOOD: Await async operations
it('should create', async () => {
  await service.create(dto);
  expect(mockRepo.save).toHaveBeenCalled();
});
```

---

## 9. Quick Reference

### Jest Matchers

```typescript
// Equality
expect(value).toBe(exact);           // Strict equality
expect(value).toEqual(similar);      // Deep equality
expect(value).toBeNull();
expect(value).toBeDefined();
expect(value).toBeTruthy();

// Numbers
expect(value).toBeGreaterThan(3);
expect(value).toBeLessThanOrEqual(5);

// Strings
expect(value).toMatch(/regex/);
expect(value).toContain('substring');

// Arrays
expect(array).toContain(item);
expect(array).toHaveLength(3);

// Objects
expect(obj).toHaveProperty('name');
expect(obj).toMatchObject({ name: 'test' });

// Exceptions
expect(() => fn()).toThrow();
expect(() => fn()).toThrow('specific error');

// Async
await expect(promise).resolves.toBe(value);
await expect(promise).rejects.toThrow();

// Mocks
expect(mock).toHaveBeenCalled();
expect(mock).toHaveBeenCalledWith(arg1, arg2);
expect(mock).toHaveBeenCalledTimes(3);
```

### Running Tests

```bash
# Run all tests
pnpm test

# Run specific file
pnpm test medications.service.spec.ts

# Run with coverage
pnpm test --coverage

# Run in watch mode
pnpm test --watch

# Run matching pattern
pnpm test -t "should create"
```

---

## 10. Test Coverage

### What Coverage Means

```
Line Coverage:    % of lines executed
Branch Coverage:  % of if/else paths taken
Function Coverage: % of functions called
Statement Coverage: % of statements executed
```

### Coverage Targets

```
Recommended minimums:
• Statements: 80%
• Branches: 70%
• Functions: 80%
• Lines: 80%

CareCircle targets:
• Services: 90%+
• Controllers: 80%+
• Utils: 90%+
```

---

*Next: [Testing Philosophy](philosophy.md) | [Integration Testing](integration-testing.md)*

