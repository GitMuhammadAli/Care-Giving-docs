# 🔴🟢🔵 Test-Driven Development (TDD) - Complete Guide

> A comprehensive guide to TDD - red-green-refactor cycle, when to use TDD, benefits, and practical implementation patterns.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "TDD is a development practice where you write a failing test first (Red), write minimal code to make it pass (Green), then improve the code while keeping tests passing (Refactor) - driving design through tests and ensuring every line of code has a reason to exist."

### The 7 Key Concepts (Remember These!)
```
1. RED           → Write a failing test first
2. GREEN         → Write minimal code to pass the test
3. REFACTOR      → Improve code while tests stay green
4. BABY STEPS    → Small incremental changes
5. YAGNI         → You Aren't Gonna Need It - only write what's tested
6. EMERGENT DESIGN → Design emerges from tests
7. FAST FEEDBACK → Tests run in milliseconds
```

### The Red-Green-Refactor Cycle
```
┌─────────────────────────────────────────────────────────────────┐
│                  RED-GREEN-REFACTOR CYCLE                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                         ┌─────────┐                            │
│                         │   RED   │                            │
│                         │  Write  │                            │
│                         │ failing │                            │
│                         │  test   │                            │
│                         └────┬────┘                            │
│                              │                                  │
│                              ▼                                  │
│              ┌───────────────────────────────┐                 │
│              │                               │                 │
│              │    Test FAILS (Red bar)       │                 │
│              │    This is GOOD!              │                 │
│              │                               │                 │
│              └───────────────┬───────────────┘                 │
│                              │                                  │
│                              ▼                                  │
│                         ┌─────────┐                            │
│                         │  GREEN  │                            │
│                         │  Write  │                            │
│                         │ minimal │                            │
│                         │  code   │                            │
│                         └────┬────┘                            │
│                              │                                  │
│                              ▼                                  │
│              ┌───────────────────────────────┐                 │
│              │                               │                 │
│              │    Test PASSES (Green bar)    │                 │
│              │    Code works!                │                 │
│              │                               │                 │
│              └───────────────┬───────────────┘                 │
│                              │                                  │
│                              ▼                                  │
│                         ┌─────────┐                            │
│                         │REFACTOR │                            │
│                         │ Improve │                            │
│                         │  code   │                            │
│                         └────┬────┘                            │
│                              │                                  │
│                              ▼                                  │
│              ┌───────────────────────────────┐                 │
│              │                               │                 │
│              │    Tests still PASS           │                 │
│              │    Code is clean!             │                 │
│              │                               │                 │
│              └───────────────┬───────────────┘                 │
│                              │                                  │
│                              └───────► Repeat                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### TDD Laws (Uncle Bob's Three Laws)
```
┌─────────────────────────────────────────────────────────────────┐
│                     THREE LAWS OF TDD                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  LAW 1:                                                        │
│  ───────                                                        │
│  You may not write production code until you have written      │
│  a failing unit test.                                          │
│                                                                 │
│  LAW 2:                                                        │
│  ───────                                                        │
│  You may not write more of a unit test than is sufficient      │
│  to fail, and not compiling is failing.                        │
│                                                                 │
│  LAW 3:                                                        │
│  ───────                                                        │
│  You may not write more production code than is sufficient     │
│  to pass the currently failing test.                           │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  In practice: Write ONE failing assertion, make it pass,       │
│  then refactor. Repeat in tiny cycles (seconds to minutes).    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Red-green-refactor"** | "I follow strict red-green-refactor cycles" |
| **"Test-first"** | "Test-first development ensures testable design" |
| **"Emergent design"** | "The design emerges naturally from TDD cycles" |
| **"Baby steps"** | "TDD works best with baby steps - tiny increments" |
| **"Triangulation"** | "I use triangulation to generalize solutions" |
| **"Fake it till you make it"** | "Start by faking, then make it real" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Cycle time | **1-5 minutes** | Fast feedback |
| Test run time | **< 10 seconds** | Instant feedback |
| Lines per cycle | **1-10** | Small increments |
| Refactor frequency | **Every green** | Keep code clean |
| Coverage from TDD | **~100%** | By definition |

### The "Wow" Statement (Memorize This!)
> "I practice TDD for all business logic and complex algorithms. The red-green-refactor cycle keeps me focused - I write one failing test, implement just enough to pass, then refactor while tests stay green. This gives me near-100% coverage by definition, because every line of code exists to pass a test. TDD has changed how I think about design - I write code that's testable by default, with clear interfaces and minimal dependencies. For legacy code, I add characterization tests first, then TDD new features. The initial slowdown pays off with fewer bugs, easier refactoring, and documentation through tests."

---

## 📚 Table of Contents

1. [TDD in Practice](#1-tdd-in-practice)
2. [TDD Patterns](#2-tdd-patterns)
3. [When to Use TDD](#3-when-to-use-tdd)
4. [TDD with Different Architectures](#4-tdd-with-different-architectures)
5. [Common Challenges](#5-common-challenges)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. TDD in Practice

```typescript
// ════════════════════════════════════════════════════════════════
// TDD EXAMPLE: Building a Password Validator
// ════════════════════════════════════════════════════════════════

// CYCLE 1: RED - Write failing test
// ────────────────────────────────
describe('PasswordValidator', () => {
  it('should reject empty password', () => {
    const validator = new PasswordValidator();
    const result = validator.validate('');
    expect(result.isValid).toBe(false);
  });
});

// CYCLE 1: GREEN - Minimal code to pass
// ────────────────────────────────────
class PasswordValidator {
  validate(password: string): ValidationResult {
    return { isValid: false, errors: [] };
  }
}
// Test passes! (But code is obviously incomplete)

// CYCLE 2: RED - Add next requirement
// ───────────────────────────────────
it('should accept valid password', () => {
  const validator = new PasswordValidator();
  const result = validator.validate('ValidPass123!');
  expect(result.isValid).toBe(true);
});

// CYCLE 2: GREEN - Make it pass
// ─────────────────────────────
class PasswordValidator {
  validate(password: string): ValidationResult {
    if (password === '') {
      return { isValid: false, errors: ['Password is required'] };
    }
    return { isValid: true, errors: [] };
  }
}

// CYCLE 3: RED - Add minimum length
// ─────────────────────────────────
it('should reject password shorter than 8 characters', () => {
  const validator = new PasswordValidator();
  const result = validator.validate('Short1!');
  expect(result.isValid).toBe(false);
  expect(result.errors).toContain('Password must be at least 8 characters');
});

// CYCLE 3: GREEN
// ───────────────
class PasswordValidator {
  validate(password: string): ValidationResult {
    const errors: string[] = [];
    
    if (password === '') {
      errors.push('Password is required');
    } else if (password.length < 8) {
      errors.push('Password must be at least 8 characters');
    }
    
    return {
      isValid: errors.length === 0,
      errors,
    };
  }
}

// CYCLE 4: RED - Require uppercase
// ────────────────────────────────
it('should reject password without uppercase letter', () => {
  const validator = new PasswordValidator();
  const result = validator.validate('lowercase123!');
  expect(result.isValid).toBe(false);
  expect(result.errors).toContain('Password must contain uppercase letter');
});

// CYCLE 4: GREEN
// ───────────────
class PasswordValidator {
  validate(password: string): ValidationResult {
    const errors: string[] = [];
    
    if (password === '') {
      errors.push('Password is required');
    }
    if (password.length < 8) {
      errors.push('Password must be at least 8 characters');
    }
    if (!/[A-Z]/.test(password)) {
      errors.push('Password must contain uppercase letter');
    }
    
    return {
      isValid: errors.length === 0,
      errors,
    };
  }
}

// REFACTOR: Extract validation rules
// ──────────────────────────────────
interface ValidationRule {
  test: (password: string) => boolean;
  message: string;
}

class PasswordValidator {
  private rules: ValidationRule[] = [
    { test: (p) => p.length > 0, message: 'Password is required' },
    { test: (p) => p.length >= 8, message: 'Password must be at least 8 characters' },
    { test: (p) => /[A-Z]/.test(p), message: 'Password must contain uppercase letter' },
    { test: (p) => /[a-z]/.test(p), message: 'Password must contain lowercase letter' },
    { test: (p) => /[0-9]/.test(p), message: 'Password must contain number' },
    { test: (p) => /[!@#$%^&*]/.test(p), message: 'Password must contain special character' },
  ];

  validate(password: string): ValidationResult {
    const errors = this.rules
      .filter(rule => !rule.test(password))
      .map(rule => rule.message);
    
    return {
      isValid: errors.length === 0,
      errors,
    };
  }
}
// All tests still pass! Design emerged from tests.
```

---

## 2. TDD Patterns

```typescript
// ════════════════════════════════════════════════════════════════
// TDD PATTERN: Fake It Till You Make It
// ════════════════════════════════════════════════════════════════

// Start with hardcoded value, generalize later
describe('Calculator', () => {
  it('should add two numbers', () => {
    expect(add(2, 3)).toBe(5);
  });
});

// GREEN - Fake it
function add(a: number, b: number): number {
  return 5; // Hardcoded!
}

// Add another test to force generalization
it('should add different numbers', () => {
  expect(add(1, 1)).toBe(2);
});

// GREEN - Make it real
function add(a: number, b: number): number {
  return a + b;
}

// ════════════════════════════════════════════════════════════════
// TDD PATTERN: Triangulation
// ════════════════════════════════════════════════════════════════

// Add multiple test cases to triangulate correct implementation
describe('FizzBuzz', () => {
  it('should return number as string for regular numbers', () => {
    expect(fizzBuzz(1)).toBe('1');
    expect(fizzBuzz(2)).toBe('2');
    expect(fizzBuzz(4)).toBe('4');
  });

  it('should return Fizz for multiples of 3', () => {
    expect(fizzBuzz(3)).toBe('Fizz');
    expect(fizzBuzz(6)).toBe('Fizz');
    expect(fizzBuzz(9)).toBe('Fizz');
  });

  it('should return Buzz for multiples of 5', () => {
    expect(fizzBuzz(5)).toBe('Buzz');
    expect(fizzBuzz(10)).toBe('Buzz');
    expect(fizzBuzz(20)).toBe('Buzz');
  });

  it('should return FizzBuzz for multiples of both', () => {
    expect(fizzBuzz(15)).toBe('FizzBuzz');
    expect(fizzBuzz(30)).toBe('FizzBuzz');
  });
});

// ════════════════════════════════════════════════════════════════
// TDD PATTERN: Obvious Implementation
// ════════════════════════════════════════════════════════════════

// When solution is obvious, implement directly
it('should format currency', () => {
  expect(formatCurrency(1234.56)).toBe('$1,234.56');
});

// Obvious - implement directly
function formatCurrency(amount: number): string {
  return '$' + amount.toLocaleString('en-US', {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2,
  });
}

// ════════════════════════════════════════════════════════════════
// TDD PATTERN: Test List
// ════════════════════════════════════════════════════════════════

// Before coding, write a list of tests to implement
/*
 Shopping Cart Tests:
 - [ ] Empty cart has zero total
 - [ ] Can add item to cart
 - [ ] Adding same item increases quantity
 - [ ] Can remove item from cart
 - [ ] Can update item quantity
 - [ ] Total reflects all items
 - [ ] Discount applied to total
 - [ ] Cannot add negative quantity
 - [ ] Cart limit enforced
*/

// Then implement one test at a time, red-green-refactor
```

---

## 3. When to Use TDD

```yaml
# ════════════════════════════════════════════════════════════════
# WHEN TO USE TDD
# ════════════════════════════════════════════════════════════════

tdd_recommended:
  - Business logic and algorithms
  - Complex calculations
  - State machines
  - Data transformations
  - API contracts
  - Bug fixes (write failing test first)
  - When requirements are clear

tdd_optional:
  - Simple CRUD operations
  - UI layout (visual testing better)
  - Prototyping / exploration
  - Third-party integrations
  - Performance-critical code (benchmark first)

tdd_not_ideal:
  - Exploratory coding / spikes
  - UI design iteration
  - When requirements are unclear
  - Legacy code without tests (add tests first)
  - Generated code

# ════════════════════════════════════════════════════════════════
# TDD BENEFITS
# ════════════════════════════════════════════════════════════════

benefits:
  design:
    - Forces testable design
    - Loose coupling emerges
    - Clear interfaces
    - Single responsibility
    
  quality:
    - High test coverage by definition
    - Catches bugs early
    - Regression protection
    - Living documentation
    
  productivity:
    - Fast feedback
    - Confidence to refactor
    - Less debugging time
    - Predictable velocity

  psychology:
    - Small wins (green tests)
    - Clear progress
    - Reduced anxiety about changes
```

---

## 4. TDD with Different Architectures

```typescript
// ════════════════════════════════════════════════════════════════
// TDD WITH CLEAN ARCHITECTURE
// ════════════════════════════════════════════════════════════════

// Start from use case, work outward

// 1. First test: Define what the use case does
describe('CreateOrderUseCase', () => {
  it('should create order with calculated total', async () => {
    // Arrange
    const orderRepository = mockOrderRepository();
    const useCase = new CreateOrderUseCase(orderRepository);
    
    // Act
    const result = await useCase.execute({
      userId: 'user-1',
      items: [{ productId: 'prod-1', quantity: 2, price: 10 }],
    });
    
    // Assert
    expect(result.total).toBe(20);
    expect(orderRepository.save).toHaveBeenCalled();
  });
});

// 2. Implement use case
class CreateOrderUseCase {
  constructor(private orderRepository: OrderRepository) {}

  async execute(input: CreateOrderInput): Promise<Order> {
    const total = input.items.reduce(
      (sum, item) => sum + item.price * item.quantity,
      0
    );
    
    const order = new Order({
      userId: input.userId,
      items: input.items,
      total,
      status: 'pending',
    });
    
    await this.orderRepository.save(order);
    return order;
  }
}

// 3. TDD the repository implementation separately
describe('PrismaOrderRepository', () => {
  it('should save order to database', async () => {
    const repository = new PrismaOrderRepository(prisma);
    const order = new Order({ ... });
    
    await repository.save(order);
    
    const saved = await prisma.order.findUnique({ where: { id: order.id }});
    expect(saved).toMatchObject({ ... });
  });
});

// ════════════════════════════════════════════════════════════════
// TDD WITH REACT COMPONENTS
// ════════════════════════════════════════════════════════════════

// 1. Test behavior, not implementation
describe('Counter', () => {
  it('should display initial count', () => {
    render(<Counter initialCount={5} />);
    expect(screen.getByText('Count: 5')).toBeInTheDocument();
  });

  it('should increment count when button clicked', () => {
    render(<Counter initialCount={0} />);
    fireEvent.click(screen.getByRole('button', { name: /increment/i }));
    expect(screen.getByText('Count: 1')).toBeInTheDocument();
  });

  it('should call onChange with new count', () => {
    const onChange = jest.fn();
    render(<Counter initialCount={0} onChange={onChange} />);
    fireEvent.click(screen.getByRole('button', { name: /increment/i }));
    expect(onChange).toHaveBeenCalledWith(1);
  });
});

// 2. Implement component
function Counter({ initialCount, onChange }: CounterProps) {
  const [count, setCount] = useState(initialCount);

  const handleIncrement = () => {
    const newCount = count + 1;
    setCount(newCount);
    onChange?.(newCount);
  };

  return (
    <div>
      <span>Count: {count}</span>
      <button onClick={handleIncrement}>Increment</button>
    </div>
  );
}
```

---

## 5. Common Challenges

```typescript
// ════════════════════════════════════════════════════════════════
// CHALLENGE: Testing Async Code
// ════════════════════════════════════════════════════════════════

// Use async/await in tests
describe('AsyncService', () => {
  it('should fetch data', async () => {
    const service = new DataService(mockApi);
    mockApi.get.mockResolvedValue({ data: 'test' });

    const result = await service.fetchData();

    expect(result).toBe('test');
  });

  it('should handle errors', async () => {
    mockApi.get.mockRejectedValue(new Error('Network error'));

    await expect(service.fetchData()).rejects.toThrow('Network error');
  });
});

// ════════════════════════════════════════════════════════════════
// CHALLENGE: Testing Time-Dependent Code
// ════════════════════════════════════════════════════════════════

// Inject time dependency
describe('TokenService', () => {
  it('should check token expiry', () => {
    const now = new Date('2024-01-15T10:00:00Z');
    const clock = { now: () => now };
    const service = new TokenService(clock);

    const token = { expiresAt: new Date('2024-01-15T09:00:00Z') };
    
    expect(service.isExpired(token)).toBe(true);
  });
});

// Or use fake timers
beforeEach(() => {
  jest.useFakeTimers();
  jest.setSystemTime(new Date('2024-01-15T10:00:00Z'));
});

afterEach(() => {
  jest.useRealTimers();
});

// ════════════════════════════════════════════════════════════════
// CHALLENGE: Testing External Dependencies
// ════════════════════════════════════════════════════════════════

// Wrap external dependencies behind interfaces
interface PaymentGateway {
  charge(amount: number, card: Card): Promise<PaymentResult>;
}

class StripePaymentGateway implements PaymentGateway {
  async charge(amount: number, card: Card): Promise<PaymentResult> {
    // Real Stripe API call
  }
}

// In tests, use mock implementation
class MockPaymentGateway implements PaymentGateway {
  async charge(amount: number, card: Card): Promise<PaymentResult> {
    return { success: true, transactionId: 'mock-tx-123' };
  }
}

describe('PaymentService', () => {
  it('should process payment', async () => {
    const gateway = new MockPaymentGateway();
    const service = new PaymentService(gateway);

    const result = await service.processPayment(100, mockCard);

    expect(result.success).toBe(true);
  });
});
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# TDD PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Skipping the Red step
# Bad
# Write code first, then tests
# Not TDD - you lose design benefits

# Good
# Always write failing test first
# Verify test actually fails for right reason

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Writing too many tests at once
# Bad
describe('Calculator', () => {
  it('adds', () => { ... });
  it('subtracts', () => { ... });
  it('multiplies', () => { ... });
  it('divides', () => { ... });
  // All written before any implementation!
});

# Good
# Write ONE test, make it pass, refactor
# Then write next test

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Writing more code than needed
# Bad
# Test: "should add two numbers"
# Implementation: Full calculator with history, memory, scientific functions

# Good
# Write ONLY what's needed to pass current test
# Add features only when tests require them

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Skipping refactor step
# Bad
# Test passes → immediately write next test
# Code becomes messy

# Good
# Test passes → refactor while green → then next test
# Refactor: remove duplication, improve names, extract methods

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Testing implementation details
# Bad
it('should call repository.save once', () => {
  await service.createUser(data);
  expect(repo.save).toHaveBeenCalledTimes(1);
});
# Brittle - breaks if implementation changes

# Good
it('should create user with correct data', () => {
  const user = await service.createUser(data);
  expect(user.email).toBe(data.email);
});
# Tests behavior, survives refactoring

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Slow tests
# Bad
# Tests take minutes to run
# Developers stop running them frequently

# Good
# Unit tests: milliseconds each
# Total suite: < 10 seconds
# Run on every save
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is TDD?"**
> "Test-Driven Development: write a failing test first (Red), write minimal code to pass (Green), then refactor while keeping tests green (Refactor). Repeat in small cycles. Every line of code exists to pass a test, resulting in high coverage and testable design."

**Q: "What are the three laws of TDD?"**
> "1) Don't write production code until you have a failing test. 2) Don't write more of a test than is sufficient to fail. 3) Don't write more production code than is sufficient to pass the test. Result: tiny cycles of seconds to minutes."

**Q: "What are the benefits of TDD?"**
> "Design: Forces testable, loosely coupled design. Quality: Near-100% coverage, catches bugs early. Confidence: Safe to refactor. Documentation: Tests document behavior. Productivity: Less debugging, faster feedback."

### Intermediate Questions

**Q: "What is red-green-refactor?"**
> "The TDD cycle. Red: Write a test that fails (proves test works). Green: Write minimal code to pass (just enough, even hardcoded). Refactor: Improve code while tests stay green (remove duplication, improve names). Repeat with next test."

**Q: "When would you NOT use TDD?"**
> "Exploratory coding/spikes (don't know what to build). UI design iteration (visual feedback more important). Unclear requirements (need to discover first). Simple boilerplate/CRUD. Legacy code (add characterization tests first, then TDD new code)."

**Q: "What is triangulation in TDD?"**
> "Adding multiple test cases to force generalization of the solution. If one test case passes with hardcoded value, add another that requires the real algorithm. Example: add(2,3)→5 passes with 'return 5', add(1,1)→2 forces 'return a+b'."

### Advanced Questions

**Q: "How do you TDD code with external dependencies?"**
> "Wrap dependencies behind interfaces. TDD against the interface using mocks/fakes. Implement the real adapter separately. Example: PaymentGateway interface, mock for unit tests, StripePaymentGateway for production. Integration tests verify real implementation."

**Q: "How does TDD affect design?"**
> "TDD naturally leads to: Dependency injection (testability requires it). Single responsibility (easier to test small units). Interface segregation (mock only what you need). Low coupling (isolated tests). Clear contracts (tests define behavior). Design emerges from tests."

**Q: "How do you introduce TDD to a team?"**
> "Start with pair programming, demonstrate cycles. Begin with simple functions, build confidence. Practice on new code first, not legacy. Do coding katas together. Share wins (bugs caught, easy refactoring). Don't mandate - show value. Accept learning curve."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                      TDD CHECKLIST                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  RED PHASE:                                                     │
│  □ Write ONE failing test                                      │
│  □ Test fails for the RIGHT reason                             │
│  □ Test is small and focused                                   │
│                                                                 │
│  GREEN PHASE:                                                   │
│  □ Write MINIMAL code to pass                                  │
│  □ It's okay to hardcode initially                             │
│  □ Don't write more than needed                                │
│                                                                 │
│  REFACTOR PHASE:                                                │
│  □ Remove duplication                                          │
│  □ Improve names                                               │
│  □ Extract methods/classes                                     │
│  □ Tests still pass                                            │
│                                                                 │
│  PRINCIPLES:                                                    │
│  □ Baby steps (small cycles)                                   │
│  □ YAGNI (only what's tested)                                  │
│  □ Test behavior, not implementation                           │
│  □ Fast tests (< 10ms each)                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

TDD CYCLE:
┌────────────────────────────────────────────────────────────────┐
│  🔴 RED    → Write failing test                                │
│  🟢 GREEN  → Make it pass (minimal code)                       │
│  🔵 REFACTOR → Clean up (tests stay green)                     │
│  ↻ REPEAT                                                      │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

