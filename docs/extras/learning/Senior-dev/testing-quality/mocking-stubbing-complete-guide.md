# 🎭 Mocking & Stubbing - Complete Guide

> A comprehensive guide to mocking and stubbing - test doubles, dependency injection, Jest mocks, and isolation patterns for effective unit testing.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Mocking and stubbing are techniques for replacing real dependencies with controlled test doubles, allowing you to isolate the unit under test, control inputs/outputs, and verify interactions without relying on external systems like databases, APIs, or third-party services."

### The 7 Key Concepts (Remember These!)
```
1. TEST DOUBLE    → Generic term for any fake dependency
2. STUB           → Returns predetermined responses
3. MOCK           → Records interactions for verification
4. SPY            → Wraps real implementation, tracks calls
5. FAKE           → Simplified working implementation
6. DUMMY          → Placeholder that's never actually used
7. DEPENDENCY INJECTION → Passing dependencies to enable testing
```

### Test Double Types
```
┌─────────────────────────────────────────────────────────────────┐
│                     TEST DOUBLE TYPES                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  DUMMY                                                         │
│  ─────                                                          │
│  • Passed but never used                                       │
│  • Fills parameter requirements                                │
│  • Example: new Logger() passed but method doesn't log         │
│                                                                 │
│  STUB                                                           │
│  ────                                                           │
│  • Returns canned/predetermined responses                      │
│  • No verification of calls                                    │
│  • Example: userRepo.findById() → always returns testUser      │
│                                                                 │
│  SPY                                                            │
│  ───                                                            │
│  • Wraps real implementation                                   │
│  • Records calls for later verification                        │
│  • Real behavior still executes                                │
│  • Example: Track if emailService.send() was called            │
│                                                                 │
│  MOCK                                                           │
│  ────                                                           │
│  • Pre-programmed with expectations                            │
│  • Verifies correct calls were made                            │
│  • Fails if expectations not met                               │
│  • Example: Expect paymentService.charge(100) called once      │
│                                                                 │
│  FAKE                                                           │
│  ────                                                           │
│  • Working implementation (simplified)                         │
│  • Not suitable for production                                 │
│  • Example: In-memory database instead of PostgreSQL           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### When to Use Each
```
┌─────────────────────────────────────────────────────────────────┐
│                   WHEN TO USE WHAT                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  USE STUBS WHEN:                                               │
│  • You need to control what a dependency returns               │
│  • Testing different response scenarios                        │
│  • Simulating error conditions                                 │
│  • You don't care HOW many times it was called                 │
│                                                                 │
│  USE MOCKS WHEN:                                               │
│  • Verifying a method WAS called                               │
│  • Verifying call arguments                                    │
│  • Verifying call count                                        │
│  • Testing side effects (email sent, event published)          │
│                                                                 │
│  USE SPIES WHEN:                                               │
│  • Want real behavior but need to track calls                  │
│  • Partial mocking (some methods real, some stubbed)           │
│  • Verifying callbacks were invoked                            │
│                                                                 │
│  USE FAKES WHEN:                                               │
│  • Need working implementation without external system         │
│  • In-memory database for integration tests                    │
│  • Simplified file system                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Test double"** | "I use test doubles to isolate the unit under test" |
| **"Dependency injection"** | "DI makes mocking trivial - just pass the mock" |
| **"Interaction testing"** | "Mocks enable interaction testing with dependencies" |
| **"State testing"** | "Stubs support state testing - verify output" |
| **"Seam"** | "DI creates seams where we can inject test doubles" |
| **"Mock granularity"** | "Mock at the right granularity - not too shallow or deep" |

### Key Numbers to Remember
| Metric | Guideline | Why |
|--------|-----------|-----|
| Mocks per test | **1-3** | More suggests bad design |
| Mock depth | **Direct dependencies only** | Don't mock transitive |
| Stub returns | **Specific to test case** | Clear intent |
| Mock verifications | **Essential interactions only** | Avoid over-specification |

### The "Wow" Statement (Memorize This!)
> "I follow a clear strategy for test doubles: stubs for controlling inputs, mocks for verifying outputs and side effects. Dependencies are injected via constructor, making mocking trivial. I mock at the architectural boundary - repository interfaces, external API clients, not internal classes. I avoid over-mocking; if I need more than 3 mocks, it's a sign of poor design or testing at wrong level. For complex scenarios, I create reusable mock factories. I distinguish between testing state (assert on return values with stubs) and testing interactions (verify calls with mocks). The goal is isolated, fast, deterministic unit tests."

---

## 📚 Table of Contents

1. [Jest Mocking](#1-jest-mocking)
2. [Dependency Injection](#2-dependency-injection)
3. [Mock Patterns](#3-mock-patterns)
4. [Advanced Mocking](#4-advanced-mocking)
5. [Mock Factories](#5-mock-factories)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Jest Mocking

```typescript
// ════════════════════════════════════════════════════════════════
// JEST MOCK FUNCTIONS
// ════════════════════════════════════════════════════════════════

describe('Jest Mock Functions', () => {
  // Creating mock functions
  it('creates basic mock', () => {
    const mockFn = jest.fn();
    
    mockFn('arg1', 'arg2');
    mockFn('arg3');
    
    // Verification
    expect(mockFn).toHaveBeenCalled();
    expect(mockFn).toHaveBeenCalledTimes(2);
    expect(mockFn).toHaveBeenCalledWith('arg1', 'arg2');
    expect(mockFn).toHaveBeenLastCalledWith('arg3');
    expect(mockFn.mock.calls).toEqual([['arg1', 'arg2'], ['arg3']]);
  });

  // Stubbing return values
  it('stubs return values', () => {
    const mockFn = jest.fn();
    
    // Single return value
    mockFn.mockReturnValue(42);
    expect(mockFn()).toBe(42);
    
    // Return once (then default)
    mockFn.mockReturnValueOnce(1).mockReturnValueOnce(2);
    expect(mockFn()).toBe(1);
    expect(mockFn()).toBe(2);
    expect(mockFn()).toBe(42); // Falls back to mockReturnValue
  });

  // Async mocks
  it('mocks async functions', async () => {
    const mockFn = jest.fn();
    
    mockFn.mockResolvedValue({ data: 'success' });
    const result = await mockFn();
    expect(result.data).toBe('success');
    
    // Rejection
    mockFn.mockRejectedValueOnce(new Error('Failed'));
    await expect(mockFn()).rejects.toThrow('Failed');
  });

  // Mock implementation
  it('mocks implementation', () => {
    const mockFn = jest.fn((a: number, b: number) => a + b);
    
    expect(mockFn(2, 3)).toBe(5);
    expect(mockFn).toHaveBeenCalledWith(2, 3);
    
    // Change implementation
    mockFn.mockImplementation((a, b) => a * b);
    expect(mockFn(2, 3)).toBe(6);
  });
});

// ════════════════════════════════════════════════════════════════
// MOCKING MODULES
// ════════════════════════════════════════════════════════════════

// Mock entire module
jest.mock('./userRepository');
import { UserRepository } from './userRepository';

// Now UserRepository is auto-mocked
const mockRepo = UserRepository as jest.Mocked<typeof UserRepository>;

describe('UserService', () => {
  it('uses mocked repository', async () => {
    mockRepo.prototype.findById.mockResolvedValue({ id: '1', name: 'Test' });
    
    const service = new UserService(new UserRepository());
    const user = await service.getUser('1');
    
    expect(user.name).toBe('Test');
  });
});

// Mock with factory
jest.mock('./emailService', () => ({
  EmailService: jest.fn().mockImplementation(() => ({
    send: jest.fn().mockResolvedValue({ success: true }),
  })),
}));

// Mock specific methods
jest.mock('./utils', () => ({
  ...jest.requireActual('./utils'), // Keep other exports
  generateId: jest.fn(() => 'mock-id-123'),
}));

// ════════════════════════════════════════════════════════════════
// SPYING ON METHODS
// ════════════════════════════════════════════════════════════════

describe('Spies', () => {
  it('spies on object methods', () => {
    const calculator = {
      add: (a: number, b: number) => a + b,
    };
    
    const spy = jest.spyOn(calculator, 'add');
    
    const result = calculator.add(2, 3);
    
    expect(result).toBe(5); // Real implementation runs
    expect(spy).toHaveBeenCalledWith(2, 3);
    
    spy.mockRestore(); // Restore original
  });

  it('spies and overrides implementation', () => {
    const spy = jest.spyOn(Math, 'random').mockReturnValue(0.5);
    
    expect(Math.random()).toBe(0.5);
    
    spy.mockRestore();
  });

  it('spies on console', () => {
    const consoleSpy = jest.spyOn(console, 'log').mockImplementation();
    
    console.log('Test message');
    
    expect(consoleSpy).toHaveBeenCalledWith('Test message');
    
    consoleSpy.mockRestore();
  });
});
```

---

## 2. Dependency Injection

```typescript
// ════════════════════════════════════════════════════════════════
// DEPENDENCY INJECTION FOR TESTABILITY
// ════════════════════════════════════════════════════════════════

// BAD: Hard dependencies - difficult to test
class UserService {
  private repository = new UserRepository(); // Hardcoded!
  private emailService = new EmailService(); // Can't mock!
  
  async createUser(data: CreateUserInput) {
    const user = await this.repository.create(data);
    await this.emailService.send(user.email, 'Welcome!');
    return user;
  }
}

// GOOD: Dependency injection - easy to test
class UserService {
  constructor(
    private repository: IUserRepository,
    private emailService: IEmailService
  ) {}
  
  async createUser(data: CreateUserInput) {
    const user = await this.repository.create(data);
    await this.emailService.send(user.email, 'Welcome!');
    return user;
  }
}

// Now easy to test with mocks
describe('UserService', () => {
  let userService: UserService;
  let mockRepository: jest.Mocked<IUserRepository>;
  let mockEmailService: jest.Mocked<IEmailService>;

  beforeEach(() => {
    // Create mocks
    mockRepository = {
      create: jest.fn(),
      findById: jest.fn(),
      update: jest.fn(),
    };
    
    mockEmailService = {
      send: jest.fn(),
    };
    
    // Inject mocks
    userService = new UserService(mockRepository, mockEmailService);
  });

  it('should create user and send welcome email', async () => {
    // Arrange
    const userData = { email: 'test@example.com', name: 'Test' };
    const createdUser = { id: '1', ...userData };
    mockRepository.create.mockResolvedValue(createdUser);
    mockEmailService.send.mockResolvedValue(undefined);

    // Act
    const result = await userService.createUser(userData);

    // Assert
    expect(result).toEqual(createdUser);
    expect(mockRepository.create).toHaveBeenCalledWith(userData);
    expect(mockEmailService.send).toHaveBeenCalledWith(
      'test@example.com',
      'Welcome!'
    );
  });

  it('should not send email if user creation fails', async () => {
    mockRepository.create.mockRejectedValue(new Error('DB Error'));

    await expect(userService.createUser({ email: 'test@example.com', name: 'Test' }))
      .rejects.toThrow('DB Error');
    
    expect(mockEmailService.send).not.toHaveBeenCalled();
  });
});

// ════════════════════════════════════════════════════════════════
// INTERFACE-BASED MOCKING
// ════════════════════════════════════════════════════════════════

// Define interfaces for dependencies
interface IUserRepository {
  create(data: CreateUserInput): Promise<User>;
  findById(id: string): Promise<User | null>;
  update(id: string, data: UpdateUserInput): Promise<User>;
}

interface IEmailService {
  send(to: string, message: string): Promise<void>;
}

// Create typed mock helper
function createMockRepository(): jest.Mocked<IUserRepository> {
  return {
    create: jest.fn(),
    findById: jest.fn(),
    update: jest.fn(),
  };
}

function createMockEmailService(): jest.Mocked<IEmailService> {
  return {
    send: jest.fn().mockResolvedValue(undefined),
  };
}
```

---

## 3. Mock Patterns

```typescript
// ════════════════════════════════════════════════════════════════
// COMMON MOCK PATTERNS
// ════════════════════════════════════════════════════════════════

// PATTERN 1: Stub for State Testing
// Test WHAT is returned
describe('PricingService - State Testing', () => {
  it('should calculate discounted price', () => {
    // Stub: Control what discount service returns
    const discountService = {
      getDiscount: jest.fn().mockReturnValue(0.2), // 20% discount
    };
    
    const pricingService = new PricingService(discountService);
    const price = pricingService.calculatePrice(100, 'user-1');
    
    // Verify STATE (return value)
    expect(price).toBe(80);
  });
});

// PATTERN 2: Mock for Interaction Testing
// Test HOW dependencies are used
describe('OrderService - Interaction Testing', () => {
  it('should publish event when order created', async () => {
    const eventPublisher = {
      publish: jest.fn().mockResolvedValue(undefined),
    };
    
    const orderService = new OrderService(orderRepo, eventPublisher);
    await orderService.createOrder(orderData);
    
    // Verify INTERACTION
    expect(eventPublisher.publish).toHaveBeenCalledWith(
      'OrderCreated',
      expect.objectContaining({ orderId: expect.any(String) })
    );
  });
});

// PATTERN 3: Stub Different Scenarios
describe('UserService - Scenario Testing', () => {
  it('should return user when found', async () => {
    mockRepo.findById.mockResolvedValue(existingUser);
    
    const result = await userService.getUser('1');
    expect(result).toEqual(existingUser);
  });

  it('should throw when user not found', async () => {
    mockRepo.findById.mockResolvedValue(null);
    
    await expect(userService.getUser('1'))
      .rejects.toThrow('User not found');
  });

  it('should handle repository errors', async () => {
    mockRepo.findById.mockRejectedValue(new Error('DB Connection Lost'));
    
    await expect(userService.getUser('1'))
      .rejects.toThrow('DB Connection Lost');
  });
});

// PATTERN 4: Verify Call Arguments
describe('NotificationService', () => {
  it('should send notification with correct payload', async () => {
    await notificationService.notify(userId, message);
    
    expect(mockPushService.send).toHaveBeenCalledWith({
      userId,
      title: expect.any(String),
      body: message,
      timestamp: expect.any(Date),
    });
  });
});

// PATTERN 5: Verify Call Order
describe('TransactionService', () => {
  it('should validate before processing', async () => {
    await transactionService.process(txData);
    
    // Verify order of operations
    const validateCall = mockValidator.validate.mock.invocationCallOrder[0];
    const processCall = mockProcessor.process.mock.invocationCallOrder[0];
    
    expect(validateCall).toBeLessThan(processCall);
  });
});

// PATTERN 6: Partial Mock (Spy)
describe('CacheService', () => {
  it('should call real method but track calls', () => {
    const cache = new CacheService();
    const spy = jest.spyOn(cache, 'get');
    
    cache.set('key', 'value');
    const result = cache.get('key');
    
    expect(result).toBe('value'); // Real behavior
    expect(spy).toHaveBeenCalledWith('key'); // Tracked
  });
});
```

---

## 4. Advanced Mocking

```typescript
// ════════════════════════════════════════════════════════════════
// MOCKING TIME
// ════════════════════════════════════════════════════════════════

describe('Time-dependent tests', () => {
  beforeEach(() => {
    jest.useFakeTimers();
    jest.setSystemTime(new Date('2024-01-15T10:00:00Z'));
  });

  afterEach(() => {
    jest.useRealTimers();
  });

  it('should check token expiry correctly', () => {
    const token = { expiresAt: new Date('2024-01-15T09:00:00Z') };
    expect(tokenService.isExpired(token)).toBe(true);
  });

  it('should schedule task correctly', () => {
    const callback = jest.fn();
    scheduler.scheduleIn(5000, callback);
    
    expect(callback).not.toHaveBeenCalled();
    
    jest.advanceTimersByTime(5000);
    
    expect(callback).toHaveBeenCalled();
  });
});

// ════════════════════════════════════════════════════════════════
// MOCKING FETCH / AXIOS
// ════════════════════════════════════════════════════════════════

// Global fetch mock
global.fetch = jest.fn();

describe('API Client', () => {
  beforeEach(() => {
    (fetch as jest.Mock).mockClear();
  });

  it('should handle successful response', async () => {
    (fetch as jest.Mock).mockResolvedValue({
      ok: true,
      json: () => Promise.resolve({ data: 'test' }),
    });
    
    const result = await apiClient.get('/users');
    
    expect(result.data).toBe('test');
    expect(fetch).toHaveBeenCalledWith(
      expect.stringContaining('/users'),
      expect.any(Object)
    );
  });

  it('should handle errors', async () => {
    (fetch as jest.Mock).mockResolvedValue({
      ok: false,
      status: 404,
    });
    
    await expect(apiClient.get('/unknown'))
      .rejects.toThrow('Not Found');
  });
});

// Axios mock
jest.mock('axios');
import axios from 'axios';
const mockedAxios = axios as jest.Mocked<typeof axios>;

describe('Service with Axios', () => {
  it('fetches data', async () => {
    mockedAxios.get.mockResolvedValue({ data: { users: [] }});
    
    const users = await userService.fetchUsers();
    
    expect(mockedAxios.get).toHaveBeenCalledWith('/api/users');
  });
});

// ════════════════════════════════════════════════════════════════
// MOCKING CLASSES
// ════════════════════════════════════════════════════════════════

// Auto-mock class
jest.mock('./PaymentGateway');
import { PaymentGateway } from './PaymentGateway';

const MockPaymentGateway = PaymentGateway as jest.MockedClass<typeof PaymentGateway>;

describe('Checkout', () => {
  beforeEach(() => {
    MockPaymentGateway.mockClear();
    MockPaymentGateway.prototype.charge.mockResolvedValue({ success: true });
  });

  it('should process payment', async () => {
    const checkout = new Checkout(new PaymentGateway());
    await checkout.complete(100);
    
    expect(MockPaymentGateway.prototype.charge).toHaveBeenCalledWith(100);
  });
});

// ════════════════════════════════════════════════════════════════
// MOCKING PRIVATE METHODS (Use Sparingly!)
// ════════════════════════════════════════════════════════════════

describe('Testing private method behavior', () => {
  it('should test through public interface', () => {
    // GOOD: Test through public interface
    const result = service.publicMethod();
    expect(result).toBe(expected);
  });

  it('should spy on private method if absolutely needed', () => {
    // NOT RECOMMENDED but sometimes necessary
    const spy = jest.spyOn(service as any, 'privateMethod');
    
    service.publicMethod();
    
    expect(spy).toHaveBeenCalled();
  });
});
```

---

## 5. Mock Factories

```typescript
// ════════════════════════════════════════════════════════════════
// REUSABLE MOCK FACTORIES
// ════════════════════════════════════════════════════════════════

// tests/mocks/repositories.ts
export function createMockUserRepository(): jest.Mocked<IUserRepository> {
  return {
    create: jest.fn(),
    findById: jest.fn(),
    findByEmail: jest.fn(),
    update: jest.fn(),
    delete: jest.fn(),
  };
}

export function createMockOrderRepository(): jest.Mocked<IOrderRepository> {
  return {
    create: jest.fn(),
    findById: jest.fn(),
    findByUserId: jest.fn(),
    updateStatus: jest.fn(),
  };
}

// tests/mocks/services.ts
export function createMockEmailService(): jest.Mocked<IEmailService> {
  return {
    send: jest.fn().mockResolvedValue({ messageId: 'mock-id' }),
    sendTemplate: jest.fn().mockResolvedValue({ messageId: 'mock-id' }),
  };
}

export function createMockPaymentService(): jest.Mocked<IPaymentService> {
  return {
    charge: jest.fn().mockResolvedValue({ 
      success: true, 
      transactionId: 'tx-123' 
    }),
    refund: jest.fn().mockResolvedValue({ success: true }),
  };
}

// tests/mocks/fixtures.ts
export const mockUsers = {
  regular: {
    id: 'user-1',
    email: 'user@example.com',
    name: 'Regular User',
    role: 'user',
  },
  admin: {
    id: 'admin-1',
    email: 'admin@example.com',
    name: 'Admin User',
    role: 'admin',
  },
};

export const mockOrders = {
  pending: {
    id: 'order-1',
    userId: 'user-1',
    status: 'pending',
    total: 100,
  },
  completed: {
    id: 'order-2',
    userId: 'user-1',
    status: 'completed',
    total: 200,
  },
};

// Usage in tests
describe('OrderService', () => {
  let orderService: OrderService;
  let mockUserRepo: jest.Mocked<IUserRepository>;
  let mockOrderRepo: jest.Mocked<IOrderRepository>;
  let mockPaymentService: jest.Mocked<IPaymentService>;

  beforeEach(() => {
    mockUserRepo = createMockUserRepository();
    mockOrderRepo = createMockOrderRepository();
    mockPaymentService = createMockPaymentService();
    
    orderService = new OrderService(
      mockUserRepo,
      mockOrderRepo,
      mockPaymentService
    );
  });

  it('should create order for valid user', async () => {
    // Use fixtures
    mockUserRepo.findById.mockResolvedValue(mockUsers.regular);
    mockOrderRepo.create.mockResolvedValue(mockOrders.pending);
    
    const result = await orderService.createOrder('user-1', []);
    
    expect(result.status).toBe('pending');
  });
});
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# MOCKING PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Over-mocking
# Bad - Mock everything, test nothing real
mockService.calculateTotal.mockReturnValue(100);
expect(result.total).toBe(100);
# You're testing the mock, not the code!

# Good - Mock only external dependencies
# Let real business logic run

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Mocking implementation details
# Bad
expect(mockRepo.save).toHaveBeenCalledTimes(1);
expect(mockRepo.save).toHaveBeenCalledBefore(mockEmail.send);
# Brittle - breaks if implementation changes

# Good
expect(result.id).toBeDefined();
expect(mockEmail.send).toHaveBeenCalled();
# Tests behavior, not implementation

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Forgetting to reset mocks
# Bad
it('test 1', () => { mockFn.mockReturnValue(1); });
it('test 2', () => { expect(mockFn()).toBe(undefined); }); # Fails!

# Good
beforeEach(() => {
  jest.clearAllMocks();
  # Or jest.resetAllMocks() to also reset implementations
});

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Mocking what you don't own
# Bad - Mock third-party library internals
jest.mock('lodash', () => ({
  debounce: jest.fn(fn => fn),
}));
# Fragile if lodash changes

# Good - Wrap third-party in your own interface
class MyDebouncer implements IDebouncer {
  debounce(fn, wait) { return _.debounce(fn, wait); }
}
# Mock your wrapper instead

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Mock doesn't match interface
# Bad
const mockRepo = { find: jest.fn() };
# Missing other methods, may cause runtime errors

# Good - Type-safe mock
const mockRepo: jest.Mocked<IUserRepository> = {
  create: jest.fn(),
  findById: jest.fn(),
  update: jest.fn(),
  delete: jest.fn(),
};

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Too many mocks = design smell
# Bad
# Need 10 mocks to test one class
# Sign of too many dependencies

# Good
# 1-3 mocks per test
# If more needed, refactor the class
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What's the difference between a mock and a stub?"**
> "Stub: Returns predetermined values, used for controlling test inputs. You don't verify stub calls. Mock: Records interactions for verification - did this method get called? With what arguments? How many times? Stubs are for state testing, mocks for interaction testing."

**Q: "What is a test double?"**
> "Generic term for any object that replaces a real dependency in tests. Types: Dummy (placeholder, never used), Stub (canned responses), Spy (wraps real, tracks calls), Mock (verifies interactions), Fake (simplified working implementation like in-memory DB)."

**Q: "Why use dependency injection?"**
> "Makes classes testable by allowing mock injection. Instead of `this.db = new Database()` (hardcoded), use `constructor(db: IDatabase)`. In production, inject real implementation. In tests, inject mock. Creates seams for test doubles."

### Intermediate Questions

**Q: "When should you use mocks vs stubs?"**
> "Stubs: When you need to control what a dependency returns (database returns specific user). Mocks: When you need to verify interactions happened (email was sent, event was published). Use stubs for inputs, mocks for outputs/side effects."

**Q: "How do you avoid over-mocking?"**
> "Only mock external dependencies (database, APIs, filesystem). Let business logic run for real. If you need many mocks, it's a design smell - too many dependencies. Test behavior, not implementation. Ask: am I testing my code or testing the mocks?"

**Q: "How do you handle async mocks?"**
> "Jest: `mockResolvedValue()` for successful promises, `mockRejectedValue()` for errors. Use async/await in tests. Test both success and failure paths. Example: `mockRepo.findById.mockResolvedValue(user)` then `await service.getUser()`."

### Advanced Questions

**Q: "How do you mock at the right level?"**
> "Mock at architectural boundaries: repositories, API clients, external services. Don't mock internal classes or utilities. Don't mock transitive dependencies. Example: Mock UserRepository, not the database driver it uses internally. Keeps tests stable and meaningful."

**Q: "How do you test code with many dependencies?"**
> "First, question the design - too many dependencies is a code smell. Consider: extracting classes, using facade pattern. For testing: create mock factories for reusability, use builder pattern for complex mocks. Keep test setup readable."

**Q: "Mock vs Fake - when to use each?"**
> "Mocks: For verifying interactions and controlling responses in unit tests. Fast, specific. Fakes: For integration tests needing working implementation without external system. Example: SQLite fake for PostgreSQL, in-memory event bus. Fakes have behavior, mocks just record."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                    MOCKING CHECKLIST                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SETUP:                                                         │
│  □ Use dependency injection                                    │
│  □ Define interfaces for dependencies                          │
│  □ Create typed mock factories                                 │
│  □ Reset mocks in beforeEach                                   │
│                                                                 │
│  WHEN TO USE:                                                   │
│  □ STUB: Control what dependency returns                       │
│  □ MOCK: Verify interaction happened                           │
│  □ SPY: Track calls but keep real behavior                     │
│  □ FAKE: Need working simplified implementation                │
│                                                                 │
│  BEST PRACTICES:                                                │
│  □ Mock at architectural boundaries                            │
│  □ 1-3 mocks per test                                          │
│  □ Test behavior, not implementation                           │
│  □ Type-safe mocks                                             │
│                                                                 │
│  AVOID:                                                         │
│  □ Over-mocking (testing mocks, not code)                      │
│  □ Mocking implementation details                              │
│  □ Mocking what you don't own                                  │
│  □ Forgetting to reset between tests                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

JEST MOCK CHEAT SHEET:
┌────────────────────────────────────────────────────────────────┐
│ jest.fn()                    - Create mock function            │
│ .mockReturnValue(val)        - Return value                    │
│ .mockResolvedValue(val)      - Return resolved promise         │
│ .mockRejectedValue(err)      - Return rejected promise         │
│ .mockImplementation(fn)      - Custom implementation           │
│ jest.spyOn(obj, 'method')    - Spy on existing method          │
│ jest.mock('./module')        - Mock entire module              │
│ jest.clearAllMocks()         - Clear call history              │
│ jest.resetAllMocks()         - Clear + reset implementations   │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

