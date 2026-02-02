# 🔗 Integration Testing - Complete Guide

> A comprehensive guide to integration testing - database testing, API testing, testcontainers, and testing real component interactions.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Integration tests verify that multiple components work correctly together - testing real database operations, API endpoints, and service interactions without mocking, catching issues that unit tests miss at the seams between components."

### The 7 Key Concepts (Remember These!)
```
1. REAL DEPENDENCIES  → Test with actual database, cache, etc.
2. TESTCONTAINERS     → Spin up real services in Docker for tests
3. API TESTING        → Test HTTP endpoints end-to-end
4. DATABASE TESTING   → Test repositories with real database
5. TEST ISOLATION     → Each test gets clean state
6. TRANSACTION ROLLBACK → Reset database between tests
7. TEST ENVIRONMENT   → Dedicated config for testing
```

### Integration Test Scope
```
┌─────────────────────────────────────────────────────────────────┐
│                 INTEGRATION TEST SCOPE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Unit Tests (Mocked)          Integration Tests (Real)         │
│  ────────────────────         ─────────────────────────         │
│                                                                 │
│  ┌─────────────┐              ┌─────────────┐                  │
│  │   Service   │              │   Service   │                  │
│  └──────┬──────┘              └──────┬──────┘                  │
│         │                            │                          │
│         ▼                            ▼                          │
│  ┌─────────────┐              ┌─────────────┐                  │
│  │  Mock Repo  │              │  Real Repo  │                  │
│  └─────────────┘              └──────┬──────┘                  │
│                                      │                          │
│                                      ▼                          │
│                               ┌─────────────┐                  │
│                               │ Real DB     │                  │
│                               │(Testcontainer)│                │
│                               └─────────────┘                  │
│                                                                 │
│  Tests: Business logic        Tests: Data access works         │
│  Speed: Milliseconds          Speed: Seconds                   │
│  Isolation: Complete          Isolation: Per-test cleanup      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### What to Integration Test
```
┌─────────────────────────────────────────────────────────────────┐
│              WHAT TO INTEGRATION TEST                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ✅ DO INTEGRATION TEST:                                       │
│  • Database repositories (CRUD operations)                     │
│  • API endpoints (HTTP request/response)                       │
│  • Service layer with real dependencies                        │
│  • Cache integration (Redis operations)                        │
│  • Message queue producers/consumers                           │
│  • External API clients (with mocked external server)          │
│  • Authentication/authorization flows                          │
│                                                                 │
│  ❌ DON'T INTEGRATION TEST:                                    │
│  • Pure business logic (unit test it)                          │
│  • UI rendering (E2E test it)                                  │
│  • Third-party services (mock them)                            │
│  • Every permutation (test key paths)                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Testcontainers"** | "We use testcontainers for real PostgreSQL in tests" |
| **"Test isolation"** | "Transaction rollback ensures test isolation" |
| **"API integration"** | "API integration tests verify full request lifecycle" |
| **"Component testing"** | "Component tests verify service + repo together" |
| **"Test fixtures"** | "Fixtures seed consistent test data" |
| **"Database seeding"** | "We seed test data before each suite" |

### Key Numbers to Remember
| Metric | Target | Why |
|--------|--------|-----|
| Test duration | **< 5 seconds each** | Fast enough to run often |
| Total suite | **< 5 minutes** | Reasonable CI time |
| Coverage target | **Key paths** | Not 100%, critical flows |
| Container startup | **~5-10 seconds** | One-time per suite |
| % of test suite | **20-30%** | Balance with unit tests |

### The "Wow" Statement (Memorize This!)
> "We use testcontainers for integration tests - real PostgreSQL, Redis, and RabbitMQ in Docker. Each test suite starts containers once, runs migrations, then each test runs in a transaction that rolls back. Repository tests verify actual SQL operations work. API tests use supertest to hit real endpoints with real database. We test: CRUD operations, complex queries, transaction handling, error scenarios. Tests run in ~3 seconds each, total suite under 5 minutes. In CI, containers start fresh, ensuring reproducibility. This catches issues unit tests miss: bad SQL, missing indexes, transaction bugs, serialization errors."

---

## 📚 Table of Contents

1. [Testcontainers](#1-testcontainers)
2. [Database Testing](#2-database-testing)
3. [API Testing](#3-api-testing)
4. [Service Integration](#4-service-integration)
5. [Test Data Management](#5-test-data-management)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Testcontainers

```typescript
// ════════════════════════════════════════════════════════════════
// TESTCONTAINERS SETUP
// ════════════════════════════════════════════════════════════════

import { PostgreSqlContainer, StartedPostgreSqlContainer } from '@testcontainers/postgresql';
import { RedisContainer, StartedRedisContainer } from '@testcontainers/redis';
import { PrismaClient } from '@prisma/client';

// Global setup - runs once before all tests
let postgresContainer: StartedPostgreSqlContainer;
let redisContainer: StartedRedisContainer;
let prisma: PrismaClient;

beforeAll(async () => {
  // Start PostgreSQL container
  postgresContainer = await new PostgreSqlContainer('postgres:15')
    .withDatabase('testdb')
    .withUsername('test')
    .withPassword('test')
    .withExposedPorts(5432)
    .start();

  // Start Redis container
  redisContainer = await new RedisContainer('redis:7')
    .withExposedPorts(6379)
    .start();

  // Set environment variables for app
  process.env.DATABASE_URL = postgresContainer.getConnectionUri();
  process.env.REDIS_URL = `redis://${redisContainer.getHost()}:${redisContainer.getMappedPort(6379)}`;

  // Initialize Prisma
  prisma = new PrismaClient({
    datasources: { db: { url: process.env.DATABASE_URL } },
  });

  // Run migrations
  await prisma.$executeRaw`CREATE EXTENSION IF NOT EXISTS "uuid-ossp"`;
  // Or use prisma migrate deploy
}, 60000); // 60s timeout for container startup

afterAll(async () => {
  await prisma.$disconnect();
  await postgresContainer.stop();
  await redisContainer.stop();
});

// Export for tests
export { prisma, postgresContainer, redisContainer };

// ════════════════════════════════════════════════════════════════
// REUSABLE TEST SETUP
// tests/setup/integration.ts
// ════════════════════════════════════════════════════════════════

import { GenericContainer, StartedTestContainer, Wait } from 'testcontainers';

export class TestEnvironment {
  private containers: StartedTestContainer[] = [];
  public prisma!: PrismaClient;
  public redis!: Redis;

  async setup() {
    // PostgreSQL
    const postgres = await new PostgreSqlContainer()
      .withWaitStrategy(Wait.forHealthCheck())
      .start();
    this.containers.push(postgres);

    // Redis
    const redis = await new RedisContainer().start();
    this.containers.push(redis);

    // Initialize clients
    this.prisma = new PrismaClient({
      datasources: { db: { url: postgres.getConnectionUri() } },
    });

    this.redis = new Redis({
      host: redis.getHost(),
      port: redis.getMappedPort(6379),
    });

    // Run migrations
    await this.runMigrations();
  }

  async teardown() {
    await this.prisma.$disconnect();
    this.redis.disconnect();
    
    for (const container of this.containers) {
      await container.stop();
    }
  }

  async cleanDatabase() {
    // Truncate all tables
    const tables = await this.prisma.$queryRaw<{ tablename: string }[]>`
      SELECT tablename FROM pg_tables WHERE schemaname = 'public'
    `;
    
    for (const { tablename } of tables) {
      if (tablename !== '_prisma_migrations') {
        await this.prisma.$executeRawUnsafe(
          `TRUNCATE TABLE "${tablename}" CASCADE`
        );
      }
    }
  }

  private async runMigrations() {
    // Run Prisma migrations
    const { execSync } = require('child_process');
    execSync('npx prisma migrate deploy', {
      env: { ...process.env, DATABASE_URL: this.prisma.$connect.toString() },
    });
  }
}

// Usage
const testEnv = new TestEnvironment();
beforeAll(() => testEnv.setup(), 60000);
afterAll(() => testEnv.teardown());
beforeEach(() => testEnv.cleanDatabase());
```

---

## 2. Database Testing

```typescript
// ════════════════════════════════════════════════════════════════
// REPOSITORY INTEGRATION TESTS
// ════════════════════════════════════════════════════════════════

describe('UserRepository Integration', () => {
  let repository: UserRepository;
  let prisma: PrismaClient;

  beforeAll(async () => {
    // Use testcontainer setup from above
    prisma = testEnv.prisma;
    repository = new UserRepository(prisma);
  });

  beforeEach(async () => {
    // Clean slate for each test
    await prisma.user.deleteMany();
  });

  describe('create', () => {
    it('should create user in database', async () => {
      const userData = {
        email: 'test@example.com',
        name: 'Test User',
        password: 'hashed_password',
      };

      const created = await repository.create(userData);

      expect(created.id).toBeDefined();
      expect(created.email).toBe('test@example.com');
      expect(created.createdAt).toBeInstanceOf(Date);

      // Verify in database
      const fromDb = await prisma.user.findUnique({
        where: { id: created.id },
      });
      expect(fromDb).toMatchObject(userData);
    });

    it('should throw on duplicate email', async () => {
      await repository.create({ email: 'test@example.com', name: 'First' });

      await expect(
        repository.create({ email: 'test@example.com', name: 'Second' })
      ).rejects.toThrow(/unique constraint/i);
    });
  });

  describe('findByEmail', () => {
    it('should find existing user', async () => {
      await prisma.user.create({
        data: { email: 'find@example.com', name: 'Find Me' },
      });

      const found = await repository.findByEmail('find@example.com');

      expect(found).not.toBeNull();
      expect(found!.name).toBe('Find Me');
    });

    it('should return null for non-existent user', async () => {
      const found = await repository.findByEmail('nonexistent@example.com');
      expect(found).toBeNull();
    });
  });

  describe('complex queries', () => {
    beforeEach(async () => {
      // Seed test data
      await prisma.user.createMany({
        data: [
          { email: 'active1@test.com', name: 'Active 1', status: 'active' },
          { email: 'active2@test.com', name: 'Active 2', status: 'active' },
          { email: 'inactive@test.com', name: 'Inactive', status: 'inactive' },
        ],
      });
    });

    it('should find users by status with pagination', async () => {
      const result = await repository.findByStatus('active', {
        page: 1,
        limit: 10,
      });

      expect(result.users).toHaveLength(2);
      expect(result.total).toBe(2);
      expect(result.users.every(u => u.status === 'active')).toBe(true);
    });

    it('should search users by name', async () => {
      const result = await repository.search('Active');

      expect(result).toHaveLength(2);
    });
  });

  describe('transactions', () => {
    it('should rollback on failure', async () => {
      const createUserWithProfile = async () => {
        return prisma.$transaction(async (tx) => {
          const user = await tx.user.create({
            data: { email: 'tx@test.com', name: 'TX User' },
          });
          
          // This will fail (invalid data)
          await tx.profile.create({
            data: { userId: user.id, bio: null! }, // Required field
          });
          
          return user;
        });
      };

      await expect(createUserWithProfile()).rejects.toThrow();

      // User should not exist (rolled back)
      const user = await prisma.user.findUnique({
        where: { email: 'tx@test.com' },
      });
      expect(user).toBeNull();
    });
  });
});

// ════════════════════════════════════════════════════════════════
// TRANSACTION ROLLBACK FOR TEST ISOLATION
// ════════════════════════════════════════════════════════════════

describe('With Transaction Rollback', () => {
  let prisma: PrismaClient;
  
  beforeEach(async () => {
    // Start transaction
    await prisma.$executeRaw`BEGIN`;
  });

  afterEach(async () => {
    // Rollback - undo all changes
    await prisma.$executeRaw`ROLLBACK`;
  });

  it('test 1 - creates data', async () => {
    await prisma.user.create({ data: { email: 'test1@example.com' } });
    const count = await prisma.user.count();
    expect(count).toBe(1);
  });

  it('test 2 - has clean state', async () => {
    // Previous test's data was rolled back
    const count = await prisma.user.count();
    expect(count).toBe(0);
  });
});
```

---

## 3. API Testing

```typescript
// ════════════════════════════════════════════════════════════════
// API INTEGRATION TESTS WITH SUPERTEST
// ════════════════════════════════════════════════════════════════

import request from 'supertest';
import { app } from '../src/app';
import { createTestUser, generateAuthToken } from './helpers';

describe('User API Integration', () => {
  let authToken: string;

  beforeAll(async () => {
    // Create test user and get auth token
    const user = await createTestUser({ role: 'admin' });
    authToken = generateAuthToken(user);
  });

  beforeEach(async () => {
    await testEnv.cleanDatabase();
  });

  describe('POST /api/users', () => {
    it('should create user with valid data', async () => {
      const response = await request(app)
        .post('/api/users')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          email: 'new@example.com',
          name: 'New User',
          password: 'SecurePass123!',
        });

      expect(response.status).toBe(201);
      expect(response.body).toMatchObject({
        id: expect.any(String),
        email: 'new@example.com',
        name: 'New User',
      });
      expect(response.body.password).toBeUndefined(); // Should not expose
    });

    it('should return 400 for invalid email', async () => {
      const response = await request(app)
        .post('/api/users')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          email: 'invalid-email',
          name: 'Test',
          password: 'password',
        });

      expect(response.status).toBe(400);
      expect(response.body.error).toContain('email');
    });

    it('should return 401 without auth token', async () => {
      const response = await request(app)
        .post('/api/users')
        .send({ email: 'test@example.com', name: 'Test' });

      expect(response.status).toBe(401);
    });

    it('should return 409 for duplicate email', async () => {
      // Create user first
      await request(app)
        .post('/api/users')
        .set('Authorization', `Bearer ${authToken}`)
        .send({ email: 'duplicate@example.com', name: 'First' });

      // Try to create with same email
      const response = await request(app)
        .post('/api/users')
        .set('Authorization', `Bearer ${authToken}`)
        .send({ email: 'duplicate@example.com', name: 'Second' });

      expect(response.status).toBe(409);
    });
  });

  describe('GET /api/users/:id', () => {
    it('should return user by id', async () => {
      // Create user
      const createResponse = await request(app)
        .post('/api/users')
        .set('Authorization', `Bearer ${authToken}`)
        .send({ email: 'get@example.com', name: 'Get User' });

      const userId = createResponse.body.id;

      // Get user
      const response = await request(app)
        .get(`/api/users/${userId}`)
        .set('Authorization', `Bearer ${authToken}`);

      expect(response.status).toBe(200);
      expect(response.body.email).toBe('get@example.com');
    });

    it('should return 404 for non-existent user', async () => {
      const response = await request(app)
        .get('/api/users/non-existent-id')
        .set('Authorization', `Bearer ${authToken}`);

      expect(response.status).toBe(404);
    });
  });

  describe('GET /api/users (list)', () => {
    beforeEach(async () => {
      // Seed users
      for (let i = 1; i <= 25; i++) {
        await request(app)
          .post('/api/users')
          .set('Authorization', `Bearer ${authToken}`)
          .send({ email: `user${i}@example.com`, name: `User ${i}` });
      }
    });

    it('should paginate results', async () => {
      const response = await request(app)
        .get('/api/users?page=1&limit=10')
        .set('Authorization', `Bearer ${authToken}`);

      expect(response.status).toBe(200);
      expect(response.body.users).toHaveLength(10);
      expect(response.body.total).toBe(25);
      expect(response.body.page).toBe(1);
      expect(response.body.totalPages).toBe(3);
    });

    it('should filter by search query', async () => {
      const response = await request(app)
        .get('/api/users?search=User 1')
        .set('Authorization', `Bearer ${authToken}`);

      expect(response.status).toBe(200);
      // Matches User 1, User 10-19
      expect(response.body.users.length).toBeGreaterThan(0);
    });
  });
});
```

---

## 4. Service Integration

```typescript
// ════════════════════════════════════════════════════════════════
// SERVICE INTEGRATION TESTS
// ════════════════════════════════════════════════════════════════

describe('OrderService Integration', () => {
  let orderService: OrderService;
  let userRepository: UserRepository;
  let orderRepository: OrderRepository;
  let paymentService: PaymentService; // Mocked external service
  
  beforeAll(async () => {
    userRepository = new UserRepository(testEnv.prisma);
    orderRepository = new OrderRepository(testEnv.prisma);
    
    // Mock external payment service
    paymentService = {
      charge: jest.fn().mockResolvedValue({ 
        success: true, 
        transactionId: 'tx-123' 
      }),
    } as any;
    
    orderService = new OrderService(
      userRepository,
      orderRepository,
      paymentService
    );
  });

  beforeEach(async () => {
    await testEnv.cleanDatabase();
    jest.clearAllMocks();
  });

  describe('createOrder', () => {
    it('should create order for valid user', async () => {
      // Arrange - create user in real database
      const user = await testEnv.prisma.user.create({
        data: { email: 'buyer@example.com', name: 'Buyer' },
      });

      // Act
      const order = await orderService.createOrder({
        userId: user.id,
        items: [
          { productId: 'prod-1', quantity: 2, price: 25.00 },
          { productId: 'prod-2', quantity: 1, price: 50.00 },
        ],
      });

      // Assert
      expect(order.id).toBeDefined();
      expect(order.total).toBe(100.00);
      expect(order.status).toBe('pending');

      // Verify in database
      const fromDb = await testEnv.prisma.order.findUnique({
        where: { id: order.id },
        include: { items: true },
      });
      expect(fromDb).not.toBeNull();
      expect(fromDb!.items).toHaveLength(2);
    });

    it('should fail for non-existent user', async () => {
      await expect(
        orderService.createOrder({
          userId: 'non-existent-user',
          items: [{ productId: 'prod-1', quantity: 1, price: 10 }],
        })
      ).rejects.toThrow('User not found');

      // Verify no order was created
      const orders = await testEnv.prisma.order.findMany();
      expect(orders).toHaveLength(0);
    });

    it('should process payment and update status', async () => {
      const user = await testEnv.prisma.user.create({
        data: { email: 'payer@example.com', name: 'Payer' },
      });

      const order = await orderService.createOrder({
        userId: user.id,
        items: [{ productId: 'prod-1', quantity: 1, price: 100 }],
        processPayment: true,
      });

      expect(order.status).toBe('paid');
      expect(order.paymentTransactionId).toBe('tx-123');
      expect(paymentService.charge).toHaveBeenCalledWith(
        expect.objectContaining({ amount: 100 })
      );
    });

    it('should rollback order if payment fails', async () => {
      paymentService.charge = jest.fn().mockRejectedValue(
        new Error('Payment declined')
      );

      const user = await testEnv.prisma.user.create({
        data: { email: 'failed@example.com', name: 'Failed' },
      });

      await expect(
        orderService.createOrder({
          userId: user.id,
          items: [{ productId: 'prod-1', quantity: 1, price: 100 }],
          processPayment: true,
        })
      ).rejects.toThrow('Payment declined');

      // Order should not exist
      const orders = await testEnv.prisma.order.findMany();
      expect(orders).toHaveLength(0);
    });
  });
});

// ════════════════════════════════════════════════════════════════
// REDIS INTEGRATION TESTS
// ════════════════════════════════════════════════════════════════

describe('CacheService Integration', () => {
  let cacheService: CacheService;
  let redis: Redis;

  beforeAll(() => {
    redis = testEnv.redis;
    cacheService = new CacheService(redis);
  });

  beforeEach(async () => {
    await redis.flushall();
  });

  it('should cache and retrieve value', async () => {
    await cacheService.set('user:1', { name: 'Test' }, 3600);

    const cached = await cacheService.get('user:1');
    expect(cached).toEqual({ name: 'Test' });
  });

  it('should return null for missing key', async () => {
    const result = await cacheService.get('nonexistent');
    expect(result).toBeNull();
  });

  it('should expire after TTL', async () => {
    await cacheService.set('short-lived', 'value', 1); // 1 second

    // Immediately available
    expect(await cacheService.get('short-lived')).toBe('value');

    // Wait for expiry
    await new Promise(resolve => setTimeout(resolve, 1100));
    expect(await cacheService.get('short-lived')).toBeNull();
  });

  it('should delete cached value', async () => {
    await cacheService.set('to-delete', 'value');
    await cacheService.delete('to-delete');

    expect(await cacheService.get('to-delete')).toBeNull();
  });
});
```

---

## 5. Test Data Management

```typescript
// ════════════════════════════════════════════════════════════════
// TEST DATA SEEDING
// ════════════════════════════════════════════════════════════════

// tests/fixtures/seed.ts
import { PrismaClient } from '@prisma/client';
import { faker } from '@faker-js/faker';

export async function seedTestData(prisma: PrismaClient) {
  // Create users
  const users = await Promise.all(
    Array.from({ length: 10 }, (_, i) =>
      prisma.user.create({
        data: {
          email: `user${i}@test.com`,
          name: faker.person.fullName(),
          status: i < 8 ? 'active' : 'inactive',
        },
      })
    )
  );

  // Create products
  const products = await Promise.all(
    Array.from({ length: 20 }, () =>
      prisma.product.create({
        data: {
          name: faker.commerce.productName(),
          price: parseFloat(faker.commerce.price({ min: 10, max: 200 })),
          stock: faker.number.int({ min: 0, max: 100 }),
        },
      })
    )
  );

  // Create orders
  for (const user of users.slice(0, 5)) {
    await prisma.order.create({
      data: {
        userId: user.id,
        status: 'completed',
        items: {
          create: Array.from({ length: faker.number.int({ min: 1, max: 3 }) }, () => ({
            productId: faker.helpers.arrayElement(products).id,
            quantity: faker.number.int({ min: 1, max: 5 }),
            price: parseFloat(faker.commerce.price()),
          })),
        },
      },
    });
  }

  return { users, products };
}

// tests/fixtures/factories.ts
export const userFactory = {
  build: (overrides = {}) => ({
    email: faker.internet.email(),
    name: faker.person.fullName(),
    password: faker.internet.password(),
    ...overrides,
  }),

  create: async (prisma: PrismaClient, overrides = {}) => {
    return prisma.user.create({
      data: userFactory.build(overrides),
    });
  },

  createMany: async (prisma: PrismaClient, count: number, overrides = {}) => {
    return Promise.all(
      Array.from({ length: count }, () => userFactory.create(prisma, overrides))
    );
  },
};

export const orderFactory = {
  build: (userId: string, overrides = {}) => ({
    userId,
    status: 'pending',
    items: [
      { productId: faker.string.uuid(), quantity: 1, price: 10 },
    ],
    ...overrides,
  }),

  create: async (prisma: PrismaClient, userId: string, overrides = {}) => {
    const data = orderFactory.build(userId, overrides);
    return prisma.order.create({
      data: {
        ...data,
        items: {
          create: data.items,
        },
      },
      include: { items: true },
    });
  },
};

// Usage in tests
describe('Order queries', () => {
  beforeEach(async () => {
    const users = await userFactory.createMany(testEnv.prisma, 3);
    for (const user of users) {
      await orderFactory.create(testEnv.prisma, user.id);
    }
  });

  it('should find all orders', async () => {
    const orders = await orderRepository.findAll();
    expect(orders).toHaveLength(3);
  });
});
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# INTEGRATION TESTING PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Not isolating tests
# Bad
it('test 1', async () => {
  await prisma.user.create({ data: { email: 'test@example.com' } });
});
it('test 2', async () => {
  # Fails because user already exists from test 1!
});

# Good
beforeEach(async () => {
  await prisma.user.deleteMany(); # Clean slate
});

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Slow tests from container restart
# Bad
beforeEach(async () => {
  await container.stop();
  container = await new PostgreSqlContainer().start();
});
# Each test takes 10+ seconds!

# Good
beforeAll(() => startContainer()); # Once
beforeEach(() => cleanDatabase()); # Fast cleanup

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Testing everything as integration
# Bad
# Every test hits real database, even for pure logic
# Suite takes 30+ minutes

# Good
# Unit test business logic (fast)
# Integration test data access (moderate)
# E2E test critical paths (few)

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Hardcoded test data
# Bad
it('should find user', async () => {
  const user = await repo.findById('user-123'); # Assumes exists
});

# Good
it('should find user', async () => {
  const user = await userFactory.create(prisma); # Create test data
  const found = await repo.findById(user.id);
});

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Not testing error paths
# Bad
# Only test happy paths
# Errors in production surprise you

# Good
it('should handle duplicate email', async () => { ... });
it('should handle network timeout', async () => { ... });
it('should handle invalid data', async () => { ... });

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Flaky tests from timing
# Bad
await service.processAsync(data);
expect(result).toBe('processed'); # Might not be done yet

# Good
await service.processAsync(data);
await waitFor(() => expect(result).toBe('processed'));
# Or use proper async/await patterns
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is integration testing?"**
> "Testing that multiple components work correctly together. Unlike unit tests (isolated with mocks), integration tests use real dependencies - real database, real cache. They catch issues at seams between components: bad SQL, serialization errors, transaction bugs."

**Q: "What are testcontainers?"**
> "Library that spins up real services (PostgreSQL, Redis, Kafka) in Docker for tests. Containers start before tests, run in isolation, stop after. Tests use real database, not mocks. Reproducible across machines and CI. Available for Node, Java, Python, etc."

**Q: "How do you isolate integration tests?"**
> "Clean database between tests. Options: truncate tables, transaction rollback, or delete specific data. Start containers once per suite, clean data per test. Each test should be independent - can run in any order."

### Intermediate Questions

**Q: "Integration vs E2E tests?"**
> "Integration: Test component interactions (service + database, API + service). Faster, more focused. E2E: Test complete user flows through UI. Slower, broader. Integration tests verify internals work; E2E tests verify user experience works. Both needed, different purposes."

**Q: "How do you handle test data?"**
> "Use factories to generate realistic test data (faker). Create data in beforeEach for each test. Seed common data in beforeAll for suite. Clean up in afterEach. Don't rely on shared state. Use transactions with rollback for speed."

**Q: "How do you test external API integrations?"**
> "Don't call real external APIs (slow, flaky, costs). Options: 1) Mock at HTTP level (nock, msw). 2) Use sandbox/test environment if available. 3) Create fake server. Test your client handles responses correctly. Integration test your code, not their API."

### Advanced Questions

**Q: "How do you speed up integration tests?"**
> "Start containers once, clean data per test (faster than restart). Run tests in parallel with separate databases. Use transaction rollback instead of delete. Index test database appropriately. Run integration tests separately from unit tests. Consider in-memory alternatives for CI."

**Q: "How do you test database migrations?"**
> "Test migration: apply, verify schema is correct. Test rollback: apply, rollback, verify previous state. Test data migration: create data, migrate, verify transformed correctly. Use separate test database. Run in CI before production migration."

**Q: "How do you handle integration test failures in CI?"**
> "Investigate immediately - integration failures often indicate real issues. Check: container startup (logs), database state, test isolation. Add retries for flaky external connections (carefully). Quarantine persistently flaky tests. Don't skip - fix root cause."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│               INTEGRATION TESTING CHECKLIST                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SETUP:                                                         │
│  □ Testcontainers configured                                   │
│  □ Database migrations run                                     │
│  □ Clean database between tests                                │
│  □ Factories for test data                                     │
│                                                                 │
│  TEST SCOPE:                                                    │
│  □ Repository operations                                       │
│  □ API endpoints                                               │
│  □ Service integration                                         │
│  □ Error handling paths                                        │
│                                                                 │
│  ISOLATION:                                                     │
│  □ No shared mutable state                                     │
│  □ Tests can run in any order                                  │
│  □ Cleanup in afterEach                                        │
│                                                                 │
│  PERFORMANCE:                                                   │
│  □ Container starts once per suite                             │
│  □ < 5 seconds per test                                        │
│  □ Suite completes in reasonable time                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

TESTCONTAINERS QUICK START:
┌────────────────────────────────────────────────────────────────┐
│ npm install @testcontainers/postgresql                         │
│                                                                │
│ const container = await new PostgreSqlContainer().start();     │
│ const url = container.getConnectionUri();                      │
│ // Use url for your database client                            │
│ await container.stop();                                        │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

