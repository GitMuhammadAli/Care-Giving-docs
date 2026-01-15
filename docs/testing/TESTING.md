# CareCircle Testing Guide

Comprehensive testing documentation for the CareCircle application.

## Table of Contents

- [Overview](#overview)
- [Test Types](#test-types)
- [Running Tests](#running-tests)
- [Test Coverage](#test-coverage)
- [Writing Tests](#writing-tests)
- [CI/CD Integration](#cicd-integration)
- [Best Practices](#best-practices)

## Overview

The CareCircle project includes comprehensive testing across multiple layers:

1. **Unit Tests**: Test individual functions and services in isolation
2. **Integration Tests**: Test how modules work together
3. **E2E Tests**: Test complete user flows from API endpoints
4. **Load Tests**: Test system performance under load
5. **Security Tests**: Test security vulnerabilities and protections

## Test Types

### 1. Unit Tests

**Location**: `apps/api/src/**/*.spec.ts`

**Purpose**: Test individual services, controllers, and utilities in isolation

**Example Test Files**:
- [auth.service.spec.ts](apps/api/src/auth/service/auth.service.spec.ts) - Authentication service tests
- [notifications.service.spec.ts](apps/api/src/notifications/notifications.service.spec.ts) - Notification service tests
- [medications.service.spec.ts](apps/api/src/medications/service/medications.service.spec.ts) - Medication service tests

**Running Unit Tests**:
```bash
cd apps/api
npm test                    # Run all unit tests
npm test -- --watch         # Run in watch mode
npm test -- --coverage      # Run with coverage report
npm test -- auth.service    # Run specific test file
```

**Coverage Goals**:
- Services: 80%+ coverage
- Controllers: 70%+ coverage
- Utilities: 90%+ coverage

### 2. E2E (End-to-End) Tests

**Location**: `apps/api/test/**/*.e2e-spec.ts`

**Purpose**: Test complete API flows from HTTP request to database

**Example Test Files**:
- [auth.e2e-spec.ts](apps/api/test/auth.e2e-spec.ts) - Authentication flow tests
- [medications.e2e-spec.ts](apps/api/test/functional/medications.e2e-spec.ts) - Medication API tests

**Running E2E Tests**:
```bash
cd apps/api
npm run test:e2e            # Run all E2E tests
npm run test:e2e -- --watch # Watch mode
```

**Prerequisites**:
- PostgreSQL test database running
- Redis test instance running
- Environment variables set in `.env.test`

### 3. Load Tests

**Location**: `apps/api/test/load-testing/`

**Purpose**: Test system performance under various load conditions

**Tools**:
- **K6**: Modern load testing tool
- **Artillery**: Node.js load testing toolkit

**Running Load Tests**:

```bash
# K6 Load Tests
k6 run apps/api/test/load-testing/k6-load-test.js

# Artillery Load Tests
artillery run apps/api/test/load-testing/artillery-load-test.yml

# Quick Artillery test
artillery quick --count 10 --num 100 http://localhost:3001/api/v1/health
```

**Load Test Scenarios**:
1. **Smoke Test**: 1 VU for 1 minute (sanity check)
2. **Load Test**: Ramp to 100 VUs over 5 minutes (normal load)
3. **Stress Test**: Ramp to 300 VUs (breaking point)
4. **Spike Test**: Sudden spike to 500 VUs (traffic surge)

**Performance Thresholds**:
- p95 response time: < 500ms
- p99 response time: < 1000ms
- Error rate: < 1%

See [Load Testing README](apps/api/test/load-testing/README.md) for detailed instructions.

### 4. Security Tests

**Location**: `apps/api/test/security/security.e2e-spec.ts`

**Purpose**: Test security vulnerabilities and protections

**Security Test Categories**:
- SQL Injection protection
- XSS (Cross-Site Scripting) protection
- Authentication & Authorization
- Rate limiting
- Password security
- CORS protection
- Input validation
- Security headers
- Session security
- File upload security

**Running Security Tests**:
```bash
cd apps/api
npm run test:e2e security
```

## Running Tests

### Prerequisites

#### Test Database Setup
```bash
# Create test database
createdb carecircle_test

# Run migrations
DATABASE_URL="postgresql://user:pass@localhost:5432/carecircle_test" npm run migration:run
```

#### Environment Variables
Create `.env.test` in `apps/api/`:
```env
NODE_ENV=test
DATABASE_URL=postgresql://user:pass@localhost:5432/carecircle_test
REDIS_HOST=localhost
REDIS_PORT=6379
JWT_SECRET=test_jwt_secret_min_32_characters_long_for_security
JWT_REFRESH_SECRET=test_jwt_refresh_secret_min_32_characters_long
```

### Running All Tests

```bash
# From root directory
npm run test                # Run all unit tests across all apps

# From API directory
cd apps/api
npm run test                # Unit tests
npm run test:e2e            # E2E tests
npm run test:cov            # Coverage report
```

### Running Specific Tests

```bash
# Run specific test file
npm test auth.service.spec.ts

# Run tests matching pattern
npm test -- --testNamePattern="login"

# Run with verbose output
npm test -- --verbose

# Run in watch mode (great for development)
npm test -- --watch
```

### Debugging Tests

```bash
# Run with Node debugger
node --inspect-brk node_modules/.bin/jest --runInBand

# VS Code launch configuration
{
  "type": "node",
  "request": "launch",
  "name": "Jest Debug",
  "program": "${workspaceFolder}/node_modules/.bin/jest",
  "args": ["--runInBand", "--no-cache"],
  "console": "integratedTerminal",
  "internalConsoleOptions": "neverOpen"
}
```

## Test Coverage

### Generating Coverage Reports

```bash
# Generate coverage report
npm run test:cov

# View HTML coverage report
open coverage/lcov-report/index.html
```

### Coverage Goals

| Component | Target Coverage |
|-----------|----------------|
| Services | 80%+ |
| Controllers | 70%+ |
| Repositories | 75%+ |
| Utilities | 90%+ |
| Overall | 75%+ |

### Current Coverage Status

Run `npm run test:cov` to see current coverage.

**Critical Areas** (100% coverage required):
- Authentication service
- Payment processing
- Security utilities
- Data encryption/decryption

## Writing Tests

### Unit Test Template

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { ServiceName } from './service-name.service';

describe('ServiceName', () => {
  let service: ServiceName;
  let dependencyMock: jest.Mocked<Dependency>;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        ServiceName,
        {
          provide: Dependency,
          useValue: {
            method: jest.fn(),
          },
        },
      ],
    }).compile();

    service = module.get<ServiceName>(ServiceName);
    dependencyMock = module.get(Dependency);
  });

  it('should be defined', () => {
    expect(service).toBeDefined();
  });

  describe('methodName', () => {
    it('should handle success case', async () => {
      dependencyMock.method.mockResolvedValue('result');

      const result = await service.methodName();

      expect(result).toBe('expected');
      expect(dependencyMock.method).toHaveBeenCalledWith('args');
    });

    it('should handle error case', async () => {
      dependencyMock.method.mockRejectedValue(new Error('error'));

      await expect(service.methodName()).rejects.toThrow('error');
    });
  });
});
```

### E2E Test Template

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import * as request from 'supertest';
import { AppModule } from '../src/app.module';
import { PrismaService } from '../src/prisma/prisma.service';

describe('FeatureName API (e2e)', () => {
  let app: INestApplication;
  let prisma: PrismaService;
  let accessToken: string;

  beforeAll(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    prisma = app.get<PrismaService>(PrismaService);
    await app.init();

    // Setup authentication
    const loginRes = await request(app.getHttpServer())
      .post('/auth/login')
      .send({ email: 'test@example.com', password: 'password' });

    accessToken = loginRes.body.accessToken;
  });

  afterAll(async () => {
    await prisma.$disconnect();
    await app.close();
  });

  describe('GET /endpoint', () => {
    it('should return data', () => {
      return request(app.getHttpServer())
        .get('/endpoint')
        .set('Authorization', `Bearer ${accessToken}`)
        .expect(200)
        .expect((res) => {
          expect(res.body).toHaveProperty('data');
        });
    });
  });
});
```

### Best Practices

#### 1. Test Structure (AAA Pattern)
```typescript
it('should do something', () => {
  // Arrange - Set up test data and mocks
  const input = 'test';
  mockService.method.mockResolvedValue('result');

  // Act - Execute the function being tested
  const result = await service.doSomething(input);

  // Assert - Verify the results
  expect(result).toBe('expected');
  expect(mockService.method).toHaveBeenCalledWith(input);
});
```

#### 2. Descriptive Test Names
```typescript
// ❌ Bad
it('works', () => { ... });

// ✅ Good
it('should create user with hashed password when valid data provided', () => { ... });
```

#### 3. Test One Thing
```typescript
// ❌ Bad - tests multiple things
it('should handle user operations', () => {
  // Creates user
  // Updates user
  // Deletes user
});

// ✅ Good - focused tests
it('should create user with valid data', () => { ... });
it('should update user email when authorized', () => { ... });
it('should delete user when admin', () => { ... });
```

#### 4. Use beforeEach for Common Setup
```typescript
describe('UserService', () => {
  let service: UserService;
  let testUser: User;

  beforeEach(async () => {
    // Common setup for all tests
    const module = await Test.createTestingModule({ ... }).compile();
    service = module.get<UserService>(UserService);
    testUser = { id: '1', email: 'test@example.com' };
  });

  // Tests use shared setup
});
```

#### 5. Mock External Dependencies
```typescript
// Mock database calls
mockRepository.findOne.mockResolvedValue(mockUser);

// Mock external APIs
mockHttpService.get.mockResolvedValue({ data: mockResponse });

// Mock time-dependent functions
jest.spyOn(Date, 'now').mockReturnValue(mockTimestamp);
```

#### 6. Test Error Cases
```typescript
it('should throw BadRequestException when email is invalid', async () => {
  await expect(
    service.createUser({ email: 'invalid' })
  ).rejects.toThrow(BadRequestException);
});
```

#### 7. Clean Up After Tests
```typescript
afterEach(async () => {
  // Clean up test data
  await prisma.user.deleteMany({ where: { email: { contains: 'test' } } });
});

afterAll(async () => {
  // Close connections
  await prisma.$disconnect();
  await app.close();
});
```

## CI/CD Integration

### GitHub Actions Example

```yaml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: carecircle_test
        ports:
          - 5432:5432

      redis:
        image: redis:7
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Run unit tests
        run: npm test

      - name: Run E2E tests
        run: npm run test:e2e
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/carecircle_test
          REDIS_HOST: localhost

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
```

## Performance Testing

### Benchmarking Critical Paths

```typescript
describe('Performance', () => {
  it('should handle 1000 medication logs in under 5 seconds', async () => {
    const start = Date.now();

    const promises = [];
    for (let i = 0; i < 1000; i++) {
      promises.push(service.logMedication({ ... }));
    }

    await Promise.all(promises);

    const duration = Date.now() - start;
    expect(duration).toBeLessThan(5000);
  });
});
```

## Troubleshooting

### Common Issues

#### Tests Timeout
```typescript
// Increase timeout for slow tests
jest.setTimeout(30000); // 30 seconds

// Or per test
it('slow test', async () => { ... }, 30000);
```

#### Database Conflicts
```bash
# Reset test database
dropdb carecircle_test
createdb carecircle_test
npm run migration:run
```

#### Port Already in Use
```bash
# Kill process using port 3001
lsof -ti:3001 | xargs kill -9
```

#### Flaky Tests
```typescript
// Add retry logic for flaky tests
jest.retryTimes(3);
```

## Resources

- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [NestJS Testing](https://docs.nestjs.com/fundamentals/testing)
- [Supertest Documentation](https://github.com/visionmedia/supertest)
- [K6 Documentation](https://k6.io/docs/)
- [Artillery Documentation](https://www.artillery.io/docs)
- [Testing Best Practices](https://testingjavascript.com/)

## Continuous Improvement

### Test Metrics to Track

1. **Code Coverage**: Target 75%+ overall
2. **Test Execution Time**: Keep under 5 minutes for full suite
3. **Flaky Test Rate**: Target < 1%
4. **Bug Escape Rate**: Bugs found in production vs caught by tests
5. **Test Maintenance Cost**: Time spent maintaining tests

### Regular Reviews

- **Weekly**: Review failing tests and flaky tests
- **Monthly**: Review coverage reports and identify gaps
- **Quarterly**: Update test data and scenarios
- **Annually**: Review testing strategy and tools

---

**Remember**: Good tests are:
- ✅ Fast
- ✅ Independent
- ✅ Repeatable
- ✅ Self-validating
- ✅ Timely (written with or before the code)

**"If you don't like testing your product, most likely your customers won't like to test it either."**
