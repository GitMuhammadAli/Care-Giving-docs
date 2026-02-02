# 🔺 Testing Pyramid - Complete Guide

> A comprehensive guide to the testing pyramid - unit, integration, E2E testing balance, testing strategy, and building a robust test suite.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "The testing pyramid is a strategy that balances test types by quantity and scope - many fast unit tests at the base, fewer integration tests in the middle, and minimal E2E tests at the top - optimizing for fast feedback, reliability, and maintainability."

### The 7 Key Concepts (Remember These!)
```
1. UNIT TESTS        → Test individual functions/components in isolation
2. INTEGRATION TESTS → Test how components work together
3. E2E TESTS         → Test complete user flows through entire system
4. TEST ISOLATION    → Tests shouldn't depend on each other
5. FAST FEEDBACK     → Quick tests catch issues early
6. CONFIDENCE        → Tests should give confidence to deploy
7. MAINTENANCE       → Tests should be easy to maintain
```

### The Testing Pyramid
```
┌─────────────────────────────────────────────────────────────────┐
│                    THE TESTING PYRAMID                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                          /\                                     │
│                         /  \        E2E / UI Tests              │
│                        /    \       • 5-10% of tests            │
│                       /  E2E \      • Slowest (minutes)         │
│                      /________\     • Most brittle              │
│                     /          \    • Highest confidence        │
│                    /            \                               │
│                   / Integration  \  Integration Tests           │
│                  /                \ • 20-30% of tests           │
│                 /                  \• Medium speed (seconds)    │
│                /____________________\• Test component interaction│
│               /                      \                          │
│              /                        \                         │
│             /         Unit             \  Unit Tests            │
│            /                            \ • 60-70% of tests     │
│           /                              \• Fastest (ms)        │
│          /________________________________\• Isolated           │
│                                            • Most numerous      │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  KEY PRINCIPLE:                                                │
│  More tests at bottom (fast, isolated)                         │
│  Fewer tests at top (slow, integrated)                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Test Type Comparison
```
┌─────────────────────────────────────────────────────────────────┐
│                   TEST TYPE COMPARISON                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  UNIT TESTS                                                    │
│  ──────────                                                     │
│  Scope:      Single function/class/component                   │
│  Speed:      Very fast (milliseconds)                          │
│  Isolation:  Complete (mocked dependencies)                    │
│  Confidence: Low (doesn't test integration)                    │
│  Quantity:   Many (60-70%)                                     │
│  Maintenance: Easy                                             │
│  Example:    Test calculateTotal() returns correct sum         │
│                                                                 │
│  INTEGRATION TESTS                                             │
│  ─────────────────                                              │
│  Scope:      Multiple components together                      │
│  Speed:      Medium (seconds)                                  │
│  Isolation:  Partial (some real dependencies)                  │
│  Confidence: Medium (tests real interactions)                  │
│  Quantity:   Moderate (20-30%)                                 │
│  Maintenance: Medium                                           │
│  Example:    Test API endpoint writes to database correctly    │
│                                                                 │
│  E2E TESTS                                                      │
│  ─────────                                                      │
│  Scope:      Entire application flow                           │
│  Speed:      Slow (minutes)                                    │
│  Isolation:  None (full system)                                │
│  Confidence: High (tests real user experience)                 │
│  Quantity:   Few (5-10%)                                       │
│  Maintenance: Difficult (brittle)                              │
│  Example:    Test user can register, login, purchase item      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Testing pyramid"** | "We follow the testing pyramid with 70% unit, 20% integration, 10% E2E" |
| **"Test isolation"** | "Test isolation ensures failures pinpoint exact issues" |
| **"Shift left"** | "Shift left testing catches bugs earlier and cheaper" |
| **"Test flakiness"** | "We minimize E2E tests to reduce test flakiness" |
| **"Feedback loop"** | "Unit tests provide the fastest feedback loop" |
| **"Test confidence"** | "Integration tests give us confidence in real interactions" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Unit tests | **60-70%** | Fast, isolated, many |
| Integration tests | **20-30%** | Component interaction |
| E2E tests | **5-10%** | Critical paths only |
| Unit test time | **< 10ms** each | Instant feedback |
| Integration time | **< 5 seconds** each | Reasonable feedback |
| E2E time | **< 2 minutes** each | Full flow validation |

### The "Wow" Statement (Memorize This!)
> "We follow the testing pyramid with ~70% unit tests, ~25% integration tests, and ~5% E2E tests. Unit tests run in under 10ms each, giving instant feedback on every save. Integration tests use testcontainers for real database and Redis instances, ensuring our repository and service layers work correctly. E2E tests cover only critical user journeys - registration, checkout, core workflows - running in CI before deployment. This balance gives us a test suite that runs in under 5 minutes locally, catches 95% of bugs before they reach production, and rarely has flaky tests. We measure test quality with mutation testing, maintaining 80%+ mutation score on critical business logic."

---

## 📚 Table of Contents

1. [Unit Testing](#1-unit-testing)
2. [Integration Testing](#2-integration-testing)
3. [E2E Testing](#3-e2e-testing)
4. [Testing Strategy](#4-testing-strategy)
5. [Test Organization](#5-test-organization)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Unit Testing

```typescript
// ════════════════════════════════════════════════════════════════
// UNIT TESTING EXAMPLES
// ════════════════════════════════════════════════════════════════

// Function to test
function calculateOrderTotal(items: OrderItem[], discount: number = 0): number {
  const subtotal = items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  const discountAmount = subtotal * (discount / 100);
  return Math.round((subtotal - discountAmount) * 100) / 100;
}

// Unit tests - testing in isolation
describe('calculateOrderTotal', () => {
  // Happy path
  it('should calculate total for single item', () => {
    const items = [{ price: 10, quantity: 2 }];
    expect(calculateOrderTotal(items)).toBe(20);
  });

  it('should calculate total for multiple items', () => {
    const items = [
      { price: 10, quantity: 2 },
      { price: 5, quantity: 3 },
    ];
    expect(calculateOrderTotal(items)).toBe(35);
  });

  // Edge cases
  it('should return 0 for empty items array', () => {
    expect(calculateOrderTotal([])).toBe(0);
  });

  it('should apply discount correctly', () => {
    const items = [{ price: 100, quantity: 1 }];
    expect(calculateOrderTotal(items, 10)).toBe(90);
  });

  it('should handle decimal prices', () => {
    const items = [{ price: 10.99, quantity: 3 }];
    expect(calculateOrderTotal(items)).toBe(32.97);
  });

  // Boundary conditions
  it('should handle 100% discount', () => {
    const items = [{ price: 100, quantity: 1 }];
    expect(calculateOrderTotal(items, 100)).toBe(0);
  });

  it('should handle zero quantity', () => {
    const items = [{ price: 100, quantity: 0 }];
    expect(calculateOrderTotal(items)).toBe(0);
  });
});

// ════════════════════════════════════════════════════════════════
// TESTING WITH MOCKS
// ════════════════════════════════════════════════════════════════

class OrderService {
  constructor(
    private orderRepository: OrderRepository,
    private paymentService: PaymentService,
    private emailService: EmailService
  ) {}

  async createOrder(orderData: CreateOrderInput): Promise<Order> {
    // Validate
    if (orderData.items.length === 0) {
      throw new Error('Order must have at least one item');
    }

    // Calculate total
    const total = calculateOrderTotal(orderData.items);

    // Process payment
    const payment = await this.paymentService.charge(
      orderData.userId,
      total,
      orderData.paymentMethod
    );

    // Save order
    const order = await this.orderRepository.create({
      ...orderData,
      total,
      paymentId: payment.id,
      status: 'confirmed',
    });

    // Send confirmation
    await this.emailService.sendOrderConfirmation(order);

    return order;
  }
}

// Unit tests with mocks
describe('OrderService', () => {
  let orderService: OrderService;
  let mockOrderRepository: jest.Mocked<OrderRepository>;
  let mockPaymentService: jest.Mocked<PaymentService>;
  let mockEmailService: jest.Mocked<EmailService>;

  beforeEach(() => {
    // Create mocks
    mockOrderRepository = {
      create: jest.fn(),
      findById: jest.fn(),
    } as any;

    mockPaymentService = {
      charge: jest.fn(),
    } as any;

    mockEmailService = {
      sendOrderConfirmation: jest.fn(),
    } as any;

    orderService = new OrderService(
      mockOrderRepository,
      mockPaymentService,
      mockEmailService
    );
  });

  describe('createOrder', () => {
    const validOrderData = {
      userId: 'user-123',
      items: [{ price: 100, quantity: 2 }],
      paymentMethod: 'card',
    };

    it('should create order successfully', async () => {
      // Arrange
      mockPaymentService.charge.mockResolvedValue({ id: 'payment-123' });
      mockOrderRepository.create.mockResolvedValue({
        id: 'order-123',
        ...validOrderData,
        total: 200,
        status: 'confirmed',
      });
      mockEmailService.sendOrderConfirmation.mockResolvedValue(undefined);

      // Act
      const result = await orderService.createOrder(validOrderData);

      // Assert
      expect(result.id).toBe('order-123');
      expect(result.total).toBe(200);
      expect(mockPaymentService.charge).toHaveBeenCalledWith('user-123', 200, 'card');
      expect(mockEmailService.sendOrderConfirmation).toHaveBeenCalled();
    });

    it('should throw error for empty items', async () => {
      const invalidData = { ...validOrderData, items: [] };

      await expect(orderService.createOrder(invalidData))
        .rejects.toThrow('Order must have at least one item');

      // Verify no side effects
      expect(mockPaymentService.charge).not.toHaveBeenCalled();
      expect(mockOrderRepository.create).not.toHaveBeenCalled();
    });

    it('should handle payment failure', async () => {
      mockPaymentService.charge.mockRejectedValue(new Error('Card declined'));

      await expect(orderService.createOrder(validOrderData))
        .rejects.toThrow('Card declined');

      expect(mockOrderRepository.create).not.toHaveBeenCalled();
    });
  });
});
```

---

## 2. Integration Testing

```typescript
// ════════════════════════════════════════════════════════════════
// INTEGRATION TESTING WITH REAL DATABASE
// ════════════════════════════════════════════════════════════════

import { PostgreSqlContainer, StartedPostgreSqlContainer } from '@testcontainers/postgresql';
import { PrismaClient } from '@prisma/client';

describe('OrderRepository Integration', () => {
  let container: StartedPostgreSqlContainer;
  let prisma: PrismaClient;
  let orderRepository: OrderRepository;

  beforeAll(async () => {
    // Start real PostgreSQL container
    container = await new PostgreSqlContainer()
      .withDatabase('test')
      .withUsername('test')
      .withPassword('test')
      .start();

    // Connect Prisma
    prisma = new PrismaClient({
      datasources: {
        db: { url: container.getConnectionUri() },
      },
    });

    // Run migrations
    await prisma.$executeRaw`CREATE TABLE orders ...`;
    
    orderRepository = new OrderRepository(prisma);
  }, 60000); // 60s timeout for container startup

  afterAll(async () => {
    await prisma.$disconnect();
    await container.stop();
  });

  beforeEach(async () => {
    // Clean database between tests
    await prisma.order.deleteMany();
  });

  it('should create and retrieve order', async () => {
    // Create
    const created = await orderRepository.create({
      userId: 'user-123',
      items: [{ productId: 'prod-1', quantity: 2, price: 50 }],
      total: 100,
    });

    expect(created.id).toBeDefined();

    // Retrieve
    const found = await orderRepository.findById(created.id);
    expect(found).toMatchObject({
      userId: 'user-123',
      total: 100,
    });
  });

  it('should find orders by user', async () => {
    // Create multiple orders
    await orderRepository.create({ userId: 'user-1', total: 100, items: [] });
    await orderRepository.create({ userId: 'user-1', total: 200, items: [] });
    await orderRepository.create({ userId: 'user-2', total: 300, items: [] });

    // Find by user
    const user1Orders = await orderRepository.findByUserId('user-1');
    expect(user1Orders).toHaveLength(2);
    expect(user1Orders.map(o => o.total)).toEqual([100, 200]);
  });
});

// ════════════════════════════════════════════════════════════════
// API INTEGRATION TESTING
// ════════════════════════════════════════════════════════════════

import request from 'supertest';
import { app } from '../src/app';

describe('Order API Integration', () => {
  let authToken: string;

  beforeAll(async () => {
    // Get auth token
    const loginResponse = await request(app)
      .post('/api/auth/login')
      .send({ email: 'test@example.com', password: 'password' });
    authToken = loginResponse.body.token;
  });

  describe('POST /api/orders', () => {
    it('should create order and return 201', async () => {
      const response = await request(app)
        .post('/api/orders')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          items: [{ productId: 'prod-1', quantity: 2 }],
          shippingAddress: {
            street: '123 Main St',
            city: 'NYC',
            zip: '10001',
          },
        });

      expect(response.status).toBe(201);
      expect(response.body).toMatchObject({
        id: expect.any(String),
        status: 'pending',
        items: expect.arrayContaining([
          expect.objectContaining({ productId: 'prod-1', quantity: 2 }),
        ]),
      });
    });

    it('should return 401 without auth token', async () => {
      const response = await request(app)
        .post('/api/orders')
        .send({ items: [] });

      expect(response.status).toBe(401);
    });

    it('should return 400 for invalid input', async () => {
      const response = await request(app)
        .post('/api/orders')
        .set('Authorization', `Bearer ${authToken}`)
        .send({ items: [] }); // Empty items

      expect(response.status).toBe(400);
      expect(response.body.error).toContain('at least one item');
    });
  });

  describe('GET /api/orders/:id', () => {
    it('should return order by id', async () => {
      // Create order first
      const createResponse = await request(app)
        .post('/api/orders')
        .set('Authorization', `Bearer ${authToken}`)
        .send({ items: [{ productId: 'prod-1', quantity: 1 }] });

      const orderId = createResponse.body.id;

      // Get order
      const response = await request(app)
        .get(`/api/orders/${orderId}`)
        .set('Authorization', `Bearer ${authToken}`);

      expect(response.status).toBe(200);
      expect(response.body.id).toBe(orderId);
    });

    it('should return 404 for non-existent order', async () => {
      const response = await request(app)
        .get('/api/orders/non-existent-id')
        .set('Authorization', `Bearer ${authToken}`);

      expect(response.status).toBe(404);
    });
  });
});
```

---

## 3. E2E Testing

```typescript
// ════════════════════════════════════════════════════════════════
// E2E TESTING WITH PLAYWRIGHT
// ════════════════════════════════════════════════════════════════

import { test, expect, Page } from '@playwright/test';

// Page Object Pattern
class CheckoutPage {
  constructor(private page: Page) {}

  async goto() {
    await this.page.goto('/checkout');
  }

  async fillShippingAddress(address: ShippingAddress) {
    await this.page.fill('[data-testid="street"]', address.street);
    await this.page.fill('[data-testid="city"]', address.city);
    await this.page.fill('[data-testid="zip"]', address.zip);
  }

  async selectPaymentMethod(method: string) {
    await this.page.click(`[data-testid="payment-${method}"]`);
  }

  async placeOrder() {
    await this.page.click('[data-testid="place-order-btn"]');
  }

  async getConfirmationNumber(): Promise<string> {
    const element = await this.page.waitForSelector('[data-testid="order-confirmation"]');
    return element.textContent() || '';
  }
}

test.describe('Checkout Flow', () => {
  test('complete checkout as authenticated user', async ({ page }) => {
    // Login
    await page.goto('/login');
    await page.fill('[data-testid="email"]', 'test@example.com');
    await page.fill('[data-testid="password"]', 'password123');
    await page.click('[data-testid="login-btn"]');
    await expect(page).toHaveURL('/dashboard');

    // Add item to cart
    await page.goto('/products');
    await page.click('[data-testid="product-1"] [data-testid="add-to-cart"]');
    await expect(page.locator('[data-testid="cart-count"]')).toHaveText('1');

    // Checkout
    const checkout = new CheckoutPage(page);
    await checkout.goto();
    await checkout.fillShippingAddress({
      street: '123 Main St',
      city: 'New York',
      zip: '10001',
    });
    await checkout.selectPaymentMethod('card');
    await checkout.placeOrder();

    // Verify confirmation
    const confirmationNumber = await checkout.getConfirmationNumber();
    expect(confirmationNumber).toMatch(/^ORD-\d+$/);
  });

  test('show error for invalid payment', async ({ page }) => {
    // ... setup ...
    
    await page.fill('[data-testid="card-number"]', '4000000000000002'); // Decline card
    await page.click('[data-testid="place-order-btn"]');

    await expect(page.locator('[data-testid="error-message"]'))
      .toHaveText('Your card was declined');
  });
});

// ════════════════════════════════════════════════════════════════
// E2E WITH CYPRESS
// ════════════════════════════════════════════════════════════════

// cypress/e2e/checkout.cy.ts
describe('Checkout Flow', () => {
  beforeEach(() => {
    // Reset database state
    cy.task('db:seed');
    
    // Login
    cy.login('test@example.com', 'password123');
  });

  it('completes purchase successfully', () => {
    // Add to cart
    cy.visit('/products');
    cy.get('[data-testid="product-1"]').within(() => {
      cy.get('[data-testid="add-to-cart"]').click();
    });

    // Go to checkout
    cy.get('[data-testid="cart-icon"]').click();
    cy.get('[data-testid="checkout-btn"]').click();

    // Fill shipping
    cy.get('[data-testid="street"]').type('123 Main St');
    cy.get('[data-testid="city"]').type('New York');
    cy.get('[data-testid="zip"]').type('10001');

    // Select payment
    cy.get('[data-testid="payment-card"]').click();
    cy.get('[data-testid="card-number"]').type('4242424242424242');
    cy.get('[data-testid="card-expiry"]').type('12/25');
    cy.get('[data-testid="card-cvc"]').type('123');

    // Place order
    cy.get('[data-testid="place-order-btn"]').click();

    // Verify
    cy.url().should('include', '/order-confirmation');
    cy.get('[data-testid="order-number"]').should('exist');
    
    // Verify email sent (check test mailbox)
    cy.task('mail:getLatest', 'test@example.com').then((email) => {
      expect(email.subject).to.include('Order Confirmation');
    });
  });
});

// Custom commands
Cypress.Commands.add('login', (email: string, password: string) => {
  cy.session([email, password], () => {
    cy.visit('/login');
    cy.get('[data-testid="email"]').type(email);
    cy.get('[data-testid="password"]').type(password);
    cy.get('[data-testid="login-btn"]').click();
    cy.url().should('include', '/dashboard');
  });
});
```

---

## 4. Testing Strategy

```yaml
# ════════════════════════════════════════════════════════════════
# TESTING STRATEGY FRAMEWORK
# ════════════════════════════════════════════════════════════════

testing_strategy:
  principles:
    - Test behavior, not implementation
    - Tests should be deterministic (no flakiness)
    - Tests should be independent
    - Tests should be fast
    - Tests should be maintainable

  pyramid_distribution:
    unit_tests:
      percentage: 70%
      focus:
        - Pure functions (calculators, validators)
        - Business logic
        - Data transformations
        - Edge cases and error handling
      characteristics:
        - No I/O (database, network, file system)
        - Mocked dependencies
        - Run in milliseconds

    integration_tests:
      percentage: 25%
      focus:
        - Repository/database interactions
        - API endpoints
        - Service layer with real dependencies
        - External service integration
      characteristics:
        - Real database (testcontainers)
        - Real Redis/message queues
        - Seconds to run

    e2e_tests:
      percentage: 5%
      focus:
        - Critical user journeys only
        - Happy path scenarios
        - Core business flows
      examples:
        - User registration → login → purchase
        - Admin creates product → user buys
        - Password reset flow
      characteristics:
        - Full browser/app
        - Minutes to run
        - Only critical paths

  what_to_test_where:
    unit_tests:
      - "calculateTotal returns correct sum"
      - "validateEmail rejects invalid format"
      - "formatDate handles timezones"
      - "OrderService.create validates input"

    integration_tests:
      - "POST /orders creates order in database"
      - "UserRepository.findByEmail returns user"
      - "PaymentService charges Stripe correctly"
      - "EmailService sends via SendGrid"

    e2e_tests:
      - "User completes checkout flow"
      - "Admin can manage products"
      - "User resets password"

  test_data_strategy:
    unit_tests: "Inline test data, factories"
    integration_tests: "Database seeders, testcontainers"
    e2e_tests: "Seeded test environment"

  ci_cd_integration:
    pre_commit:
      - Lint
      - Unit tests (affected)
    
    pull_request:
      - All unit tests
      - All integration tests
      - E2E smoke tests
    
    main_branch:
      - All tests
      - Full E2E suite
      - Performance tests (nightly)
```

---

## 5. Test Organization

```
# ════════════════════════════════════════════════════════════════
# TEST FILE ORGANIZATION
# ════════════════════════════════════════════════════════════════

src/
├── orders/
│   ├── order.service.ts
│   ├── order.service.spec.ts        # Unit tests (co-located)
│   ├── order.repository.ts
│   └── order.repository.spec.ts     # Unit tests
│
├── users/
│   ├── user.service.ts
│   └── user.service.spec.ts
│
tests/
├── unit/                             # Or co-located with source
│   └── ...
│
├── integration/
│   ├── setup.ts                      # Shared setup (testcontainers)
│   ├── orders/
│   │   ├── order.api.test.ts        # API integration tests
│   │   └── order.repository.test.ts # DB integration tests
│   └── users/
│       └── user.api.test.ts
│
├── e2e/
│   ├── setup/
│   │   ├── global-setup.ts          # Start services
│   │   └── global-teardown.ts       # Cleanup
│   ├── fixtures/
│   │   └── test-data.ts
│   ├── pages/                        # Page objects
│   │   ├── login.page.ts
│   │   └── checkout.page.ts
│   └── specs/
│       ├── auth.spec.ts
│       └── checkout.spec.ts
│
└── factories/                        # Test data factories
    ├── user.factory.ts
    └── order.factory.ts
```

```typescript
// ════════════════════════════════════════════════════════════════
// TEST DATA FACTORIES
// ════════════════════════════════════════════════════════════════

import { faker } from '@faker-js/faker';

// Factory pattern for test data
export const userFactory = {
  build(overrides: Partial<User> = {}): User {
    return {
      id: faker.string.uuid(),
      email: faker.internet.email(),
      name: faker.person.fullName(),
      createdAt: new Date(),
      ...overrides,
    };
  },

  buildList(count: number, overrides: Partial<User> = {}): User[] {
    return Array.from({ length: count }, () => this.build(overrides));
  },
};

export const orderFactory = {
  build(overrides: Partial<Order> = {}): Order {
    return {
      id: faker.string.uuid(),
      userId: faker.string.uuid(),
      items: [
        {
          productId: faker.string.uuid(),
          quantity: faker.number.int({ min: 1, max: 5 }),
          price: parseFloat(faker.commerce.price()),
        },
      ],
      total: parseFloat(faker.commerce.price({ min: 10, max: 500 })),
      status: 'pending',
      createdAt: new Date(),
      ...overrides,
    };
  },
};

// Usage in tests
describe('OrderService', () => {
  it('should process order', async () => {
    const user = userFactory.build();
    const order = orderFactory.build({ userId: user.id, status: 'pending' });

    // ... test logic
  });
});
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# TESTING PYRAMID PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Ice cream cone (inverted pyramid)
# Bad
tests:
  e2e: 70%      # Too many slow, brittle tests
  integration: 20%
  unit: 10%     # Not enough fast, isolated tests

# Good
tests:
  unit: 70%
  integration: 25%
  e2e: 5%

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Testing implementation, not behavior
# Bad
it('should call repository.save', () => {
  await service.createOrder(data);
  expect(mockRepo.save).toHaveBeenCalledTimes(1);
});
# Brittle - breaks if implementation changes

# Good
it('should create order with correct total', () => {
  const result = await service.createOrder(data);
  expect(result.total).toBe(expectedTotal);
});
# Tests behavior - survives refactoring

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Not testing edge cases in unit tests
# Bad
it('should calculate total', () => {
  expect(calculateTotal([{ price: 10, qty: 2 }])).toBe(20);
});
# Only happy path

# Good
describe('calculateTotal', () => {
  it('should calculate for multiple items', () => { ... });
  it('should return 0 for empty array', () => { ... });
  it('should handle decimal prices', () => { ... });
  it('should throw for negative quantities', () => { ... });
});

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Flaky E2E tests
# Bad
await page.click('#submit');
await page.waitForTimeout(2000);  # Arbitrary wait
expect(await page.textContent('.result')).toBe('Success');

# Good
await page.click('#submit');
await expect(page.locator('.result')).toHaveText('Success', { 
  timeout: 5000 
});
# Wait for specific condition

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Tests depend on each other
# Bad
it('should create user', () => {
  createdUser = await createUser(data);  # Shared state!
});
it('should update user', () => {
  await updateUser(createdUser.id, ...);  # Depends on previous test
});

# Good
beforeEach(() => {
  testUser = await createUser(data);  # Fresh setup each test
});
it('should update user', () => {
  await updateUser(testUser.id, ...);
});

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Too much mocking in integration tests
# Bad - Integration test
it('should create order', async () => {
  mockDb.query.mockResolvedValue(...);
  mockRedis.get.mockResolvedValue(...);
  # This is a unit test with extra steps!
});

# Good - Integration test
it('should create order', async () => {
  # Use real database (testcontainers)
  const result = await orderRepository.create(data);
  const fromDb = await prisma.order.findUnique({ where: { id: result.id }});
  expect(fromDb).toMatchObject(data);
});
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is the testing pyramid?"**
> "A testing strategy that recommends having many fast unit tests at the base, fewer integration tests in the middle, and minimal E2E tests at the top. Typical distribution: 70% unit, 25% integration, 5% E2E. This balances fast feedback (unit), realistic testing (integration), and user confidence (E2E)."

**Q: "What's the difference between unit and integration tests?"**
> "Unit tests: Test single function/component in complete isolation with mocked dependencies. Fast (milliseconds). Integration tests: Test how components work together with real dependencies (database, APIs). Slower (seconds). Unit tests find logic bugs, integration tests find interaction bugs."

**Q: "Why have fewer E2E tests?"**
> "E2E tests are slow (minutes), brittle (break easily), expensive to maintain, and flaky. They test through UI which changes often. Keep them for critical user journeys only - registration, checkout, core workflows. For most testing, unit and integration tests are more efficient."

### Intermediate Questions

**Q: "How do you decide what to test at each level?"**
> "Unit: Pure logic, calculations, validations, transformations. Integration: Database operations, API endpoints, service interactions. E2E: Critical user journeys only. Ask: 'What's the cheapest test that gives confidence?' Start with unit, move up only when necessary."

**Q: "How do you handle test data?"**
> "Unit: Inline data or factories. Integration: Database seeders, testcontainers with migrations. E2E: Seeded test environment. Use factories (faker) for realistic random data. Reset state between tests for isolation. Never share mutable state between tests."

**Q: "What makes tests flaky and how do you fix it?"**
> "Flakiness causes: timing issues (arbitrary waits), shared state, external dependencies, race conditions. Fixes: Wait for specific conditions not timeouts, isolate tests, use deterministic test data, retry flaky tests (temporary), fix root cause. E2E most prone - minimize them."

### Advanced Questions

**Q: "How do you test microservices?"**
> "Unit tests per service (isolated). Integration tests: test each service with its database. Contract tests: verify API contracts between services (Pact). E2E: test critical flows across services. Use testcontainers for dependencies. Service virtualization for external services."

**Q: "How do you measure test quality?"**
> "Mutation testing: deliberately introduce bugs, measure if tests catch them. 80%+ mutation score for critical code. Code coverage: measure lines covered, but coverage ≠ quality (can have 100% coverage with bad tests). Test failure rate: should fail when code breaks, not randomly (flakiness)."

**Q: "How do you handle testing legacy code?"**
> "Start with characterization tests: tests that document current behavior. Add integration tests around boundaries. Refactor for testability (dependency injection). Add unit tests as you refactor. Don't aim for 100% coverage initially - focus on changed/critical code."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                   TESTING PYRAMID CHECKLIST                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  UNIT TESTS (70%):                                              │
│  □ Pure functions and business logic                           │
│  □ Edge cases and error handling                               │
│  □ Mocked dependencies                                         │
│  □ Run in milliseconds                                         │
│                                                                 │
│  INTEGRATION TESTS (25%):                                       │
│  □ Database operations                                         │
│  □ API endpoints                                               │
│  □ Service interactions                                        │
│  □ Real dependencies (testcontainers)                          │
│                                                                 │
│  E2E TESTS (5%):                                                │
│  □ Critical user journeys only                                 │
│  □ Happy path scenarios                                        │
│  □ No arbitrary waits                                          │
│  □ Page object pattern                                         │
│                                                                 │
│  BEST PRACTICES:                                                │
│  □ Test behavior, not implementation                           │
│  □ Tests are independent                                       │
│  □ Deterministic (no flakiness)                                │
│  □ Fast feedback loop                                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

TEST DISTRIBUTION:
┌────────────────────────────────────────────────────────────────┐
│ Unit Tests       │ 70%  │ Fast, isolated, many                 │
│ Integration Tests│ 25%  │ Real dependencies, moderate          │
│ E2E Tests        │ 5%   │ Critical paths only, few             │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

