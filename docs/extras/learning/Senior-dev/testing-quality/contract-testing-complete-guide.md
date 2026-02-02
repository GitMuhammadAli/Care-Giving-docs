# 📜 Contract Testing - Complete Guide

> A comprehensive guide to contract testing - Pact, consumer-driven contracts, API compatibility, and preventing integration failures in microservices.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Contract testing verifies that services can communicate correctly by testing against agreed-upon contracts (schemas) rather than live services - catching breaking API changes before deployment and enabling independent team deployment."

### The 7 Key Concepts (Remember These!)
```
1. CONTRACT        → Formal agreement of request/response format
2. CONSUMER        → Service that calls an API
3. PROVIDER        → Service that provides an API
4. PACT            → Popular contract testing framework
5. CONSUMER-DRIVEN → Consumers define what they need
6. PROVIDER VERIFICATION → Provider verifies it meets contracts
7. PACT BROKER     → Central repository for contracts
```

### Contract Testing vs Integration Testing
```
┌─────────────────────────────────────────────────────────────────┐
│           CONTRACT VS INTEGRATION TESTING                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  INTEGRATION TESTING                                           │
│  ───────────────────                                            │
│  [Consumer] ──────────────────────▶ [Provider]                 │
│              Real network call                                  │
│                                                                 │
│  • Tests against live service                                  │
│  • Slow (network, setup)                                       │
│  • Flaky (service availability)                                │
│  • Tests everything at once                                    │
│  • Hard to test edge cases                                     │
│                                                                 │
│  CONTRACT TESTING                                              │
│  ────────────────                                               │
│  [Consumer] ─▶ [Contract] ◀─ [Provider]                        │
│               (Pact file)                                       │
│                                                                 │
│  • Tests against contract                                      │
│  • Fast (no network)                                           │
│  • Reliable (deterministic)                                    │
│  • Tests consumer needs specifically                           │
│  • Easy to test edge cases                                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Consumer-Driven Contract Flow
```
┌─────────────────────────────────────────────────────────────────┐
│              CONSUMER-DRIVEN CONTRACT FLOW                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. CONSUMER WRITES TEST                                       │
│     ┌──────────────────────────────────────────────┐           │
│     │ Consumer Test:                               │           │
│     │ "When I call GET /users/123, I expect:      │           │
│     │  { id: 123, name: string, email: string }"   │           │
│     └──────────────────────┬───────────────────────┘           │
│                            │                                    │
│                            ▼                                    │
│  2. PACT FILE GENERATED                                        │
│     ┌──────────────────────────────────────────────┐           │
│     │ pact/consumer-provider.json                  │           │
│     │ {                                            │           │
│     │   "consumer": "order-service",               │           │
│     │   "provider": "user-service",                │           │
│     │   "interactions": [...]                      │           │
│     │ }                                            │           │
│     └──────────────────────┬───────────────────────┘           │
│                            │                                    │
│                            ▼                                    │
│  3. PUBLISH TO PACT BROKER                                     │
│     ┌──────────────────────────────────────────────┐           │
│     │ Pact Broker (Central Registry)               │           │
│     │ - Stores all contracts                       │           │
│     │ - Tracks versions                            │           │
│     │ - Shows compatibility matrix                 │           │
│     └──────────────────────┬───────────────────────┘           │
│                            │                                    │
│                            ▼                                    │
│  4. PROVIDER VERIFIES CONTRACT                                 │
│     ┌──────────────────────────────────────────────┐           │
│     │ Provider Test:                               │           │
│     │ "Replay consumer requests against real API"  │           │
│     │ "Verify responses match contract"            │           │
│     └──────────────────────────────────────────────┘           │
│                                                                 │
│  5. CAN-I-DEPLOY                                               │
│     ┌──────────────────────────────────────────────┐           │
│     │ "Are all contracts verified?"                │           │
│     │ If yes → Safe to deploy                      │           │
│     │ If no  → Block deployment                    │           │
│     └──────────────────────────────────────────────┘           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Consumer-driven"** | "We use consumer-driven contracts - consumers define needs" |
| **"Pact Broker"** | "Pact Broker stores contracts and tracks compatibility" |
| **"Can-I-Deploy"** | "can-i-deploy check ensures safe deployments" |
| **"Provider verification"** | "Provider verification runs in CI before deploy" |
| **"Breaking change"** | "Contract tests catch breaking changes before production" |
| **"Pacticipant"** | "Each service is a pacticipant in the contract network" |

### Key Numbers to Remember
| Metric | Target | Notes |
|--------|--------|-------|
| Contract coverage | **All critical APIs** | Between services |
| Verification time | **< 2 minutes** | Per provider |
| Breaking change detection | **100%** | Primary goal |
| Deployment confidence | **High** | With can-i-deploy |

### The "Wow" Statement (Memorize This!)
> "We use Pact for consumer-driven contract testing between our microservices. Each consumer defines what it needs from providers in test code, generating Pact files published to our Pact Broker. Provider teams verify contracts in their CI - if verification fails, deployment is blocked. The can-i-deploy check before deployment ensures we never deploy incompatible versions. This eliminated integration environment failures - we catch breaking changes at PR time. Teams deploy independently because they know contracts are verified. We cover: user service ↔ order service, order service ↔ payment service, etc. Contracts include both happy paths and error scenarios (404s, validation errors). The Pact Broker's compatibility matrix shows exactly which versions work together."

---

## 📚 Table of Contents

1. [Pact Consumer Testing](#1-pact-consumer-testing)
2. [Pact Provider Verification](#2-pact-provider-verification)
3. [Pact Broker](#3-pact-broker)
4. [Advanced Patterns](#4-advanced-patterns)
5. [CI/CD Integration](#5-cicd-integration)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Pact Consumer Testing

```typescript
// ════════════════════════════════════════════════════════════════
// PACT CONSUMER TEST
// tests/contract/user-service.consumer.pact.ts
// ════════════════════════════════════════════════════════════════

import { PactV3, MatchersV3 } from '@pact-foundation/pact';
import { UserClient } from '../../src/clients/user-client';

const { like, eachLike, regex, string, integer } = MatchersV3;

// Configure Pact
const provider = new PactV3({
  consumer: 'OrderService',
  provider: 'UserService',
  logLevel: 'warn',
  dir: './pacts', // Output directory for pact files
});

describe('UserService Client Contract', () => {
  describe('GET /users/:id', () => {
    it('returns user when user exists', async () => {
      // Define the expected interaction
      await provider
        .given('user 123 exists')  // Provider state
        .uponReceiving('a request for user 123')
        .withRequest({
          method: 'GET',
          path: '/users/123',
          headers: {
            Accept: 'application/json',
            Authorization: like('Bearer token123'),
          },
        })
        .willRespondWith({
          status: 200,
          headers: {
            'Content-Type': 'application/json',
          },
          body: {
            id: integer(123),
            name: string('John Doe'),
            email: regex(/^[\w.]+@[\w.]+\.\w+$/, 'john@example.com'),
            status: string('active'),
            createdAt: regex(/^\d{4}-\d{2}-\d{2}/, '2024-01-15'),
          },
        });

      // Execute test
      await provider.executeTest(async (mockServer) => {
        const client = new UserClient(mockServer.url);
        const user = await client.getUser('123', 'token123');

        expect(user).toMatchObject({
          id: 123,
          name: expect.any(String),
          email: expect.any(String),
        });
      });
    });

    it('returns 404 when user does not exist', async () => {
      await provider
        .given('user 999 does not exist')
        .uponReceiving('a request for non-existent user')
        .withRequest({
          method: 'GET',
          path: '/users/999',
          headers: {
            Accept: 'application/json',
          },
        })
        .willRespondWith({
          status: 404,
          headers: {
            'Content-Type': 'application/json',
          },
          body: {
            error: string('User not found'),
            code: string('USER_NOT_FOUND'),
          },
        });

      await provider.executeTest(async (mockServer) => {
        const client = new UserClient(mockServer.url);
        
        await expect(client.getUser('999', 'token')).rejects.toThrow(
          'User not found'
        );
      });
    });
  });

  describe('POST /users', () => {
    it('creates a new user', async () => {
      await provider
        .given('no existing user with email test@example.com')
        .uponReceiving('a request to create user')
        .withRequest({
          method: 'POST',
          path: '/users',
          headers: {
            'Content-Type': 'application/json',
            Authorization: like('Bearer admin-token'),
          },
          body: {
            name: string('Jane Doe'),
            email: string('jane@example.com'),
          },
        })
        .willRespondWith({
          status: 201,
          headers: {
            'Content-Type': 'application/json',
          },
          body: {
            id: integer(),
            name: string('Jane Doe'),
            email: string('jane@example.com'),
            status: string('pending'),
          },
        });

      await provider.executeTest(async (mockServer) => {
        const client = new UserClient(mockServer.url);
        const user = await client.createUser(
          { name: 'Jane Doe', email: 'jane@example.com' },
          'admin-token'
        );

        expect(user.id).toBeDefined();
        expect(user.name).toBe('Jane Doe');
      });
    });

    it('returns 400 for invalid email', async () => {
      await provider
        .given('validation enabled')
        .uponReceiving('a request with invalid email')
        .withRequest({
          method: 'POST',
          path: '/users',
          headers: {
            'Content-Type': 'application/json',
          },
          body: {
            name: string('Test User'),
            email: string('invalid-email'),
          },
        })
        .willRespondWith({
          status: 400,
          body: {
            error: string('Validation failed'),
            details: eachLike({
              field: string('email'),
              message: string('Invalid email format'),
            }),
          },
        });

      await provider.executeTest(async (mockServer) => {
        const client = new UserClient(mockServer.url);
        
        await expect(
          client.createUser({ name: 'Test', email: 'invalid-email' }, 'token')
        ).rejects.toThrow('Validation failed');
      });
    });
  });
});
```

---

## 2. Pact Provider Verification

```typescript
// ════════════════════════════════════════════════════════════════
// PACT PROVIDER VERIFICATION
// tests/contract/user-service.provider.pact.ts
// ════════════════════════════════════════════════════════════════

import { Verifier } from '@pact-foundation/pact';
import { app } from '../../src/app';
import { prisma } from '../../src/db';

describe('UserService Provider Verification', () => {
  let server: any;
  const port = 3001;

  beforeAll(async () => {
    // Start the real provider service
    server = app.listen(port);
  });

  afterAll(async () => {
    server.close();
    await prisma.$disconnect();
  });

  it('validates the expectations of OrderService', async () => {
    const verifier = new Verifier({
      providerBaseUrl: `http://localhost:${port}`,
      provider: 'UserService',
      
      // Get pacts from broker
      pactBrokerUrl: process.env.PACT_BROKER_URL,
      pactBrokerToken: process.env.PACT_BROKER_TOKEN,
      
      // Or from local files
      // pactUrls: ['./pacts/orderservice-userservice.json'],
      
      // Provider version for broker
      providerVersion: process.env.GIT_COMMIT || '1.0.0',
      providerVersionBranch: process.env.GIT_BRANCH || 'main',
      
      // Publish verification results
      publishVerificationResult: process.env.CI === 'true',
      
      // State handlers - set up data for each scenario
      stateHandlers: {
        'user 123 exists': async () => {
          await prisma.user.upsert({
            where: { id: 123 },
            update: {},
            create: {
              id: 123,
              name: 'John Doe',
              email: 'john@example.com',
              status: 'active',
            },
          });
        },
        
        'user 999 does not exist': async () => {
          await prisma.user.deleteMany({ where: { id: 999 }});
        },
        
        'no existing user with email test@example.com': async () => {
          await prisma.user.deleteMany({ 
            where: { email: 'test@example.com' }
          });
        },
        
        'validation enabled': async () => {
          // No setup needed - validation always enabled
        },
      },
      
      // Request filter - add auth headers for verification
      requestFilter: (req, res, next) => {
        // Add default auth if not provided
        if (!req.headers.authorization) {
          req.headers.authorization = 'Bearer test-token';
        }
        next();
      },
    });

    await verifier.verifyProvider();
  });
});

// ════════════════════════════════════════════════════════════════
// PROVIDER VERIFICATION WITH SPECIFIC CONSUMERS
// ════════════════════════════════════════════════════════════════

it('validates OrderService contract only', async () => {
  const verifier = new Verifier({
    providerBaseUrl: `http://localhost:${port}`,
    provider: 'UserService',
    pactBrokerUrl: process.env.PACT_BROKER_URL,
    
    // Only verify specific consumer
    consumerVersionSelectors: [
      { consumer: 'OrderService', latest: true },
    ],
    
    stateHandlers: { ... },
  });

  await verifier.verifyProvider();
});

// ════════════════════════════════════════════════════════════════
// VERIFY AGAINST PENDING PACTS (WIP)
// ════════════════════════════════════════════════════════════════

it('validates including pending pacts', async () => {
  const verifier = new Verifier({
    providerBaseUrl: `http://localhost:${port}`,
    provider: 'UserService',
    pactBrokerUrl: process.env.PACT_BROKER_URL,
    
    // Include pacts from WIP branches
    enablePending: true,
    includeWipPactsSince: '2024-01-01',
    
    stateHandlers: { ... },
  });

  await verifier.verifyProvider();
});
```

---

## 3. Pact Broker

```yaml
# ════════════════════════════════════════════════════════════════
# PACT BROKER DOCKER COMPOSE
# ════════════════════════════════════════════════════════════════

version: '3'
services:
  pact-broker:
    image: pactfoundation/pact-broker:latest
    ports:
      - "9292:9292"
    environment:
      PACT_BROKER_DATABASE_URL: postgres://postgres:password@db/pact_broker
      PACT_BROKER_BASIC_AUTH_USERNAME: admin
      PACT_BROKER_BASIC_AUTH_PASSWORD: admin
      PACT_BROKER_ALLOW_PUBLIC_READ: 'true'
    depends_on:
      - db

  db:
    image: postgres:13
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: pact_broker
    volumes:
      - pact-db:/var/lib/postgresql/data

volumes:
  pact-db:
```

```bash
# ════════════════════════════════════════════════════════════════
# PACT BROKER CLI COMMANDS
# ════════════════════════════════════════════════════════════════

# Publish pact file to broker
pact-broker publish ./pacts \
  --broker-base-url https://pact-broker.example.com \
  --broker-token $PACT_BROKER_TOKEN \
  --consumer-app-version $GIT_COMMIT \
  --branch $GIT_BRANCH \
  --tag-with-git-branch

# Check if can deploy
pact-broker can-i-deploy \
  --pacticipant OrderService \
  --version $GIT_COMMIT \
  --to-environment production \
  --broker-base-url https://pact-broker.example.com

# Record deployment
pact-broker record-deployment \
  --pacticipant OrderService \
  --version $GIT_COMMIT \
  --environment production

# Create webhook for CI triggers
pact-broker create-webhook https://ci.example.com/trigger \
  --request POST \
  --contract-content-changed \
  --provider UserService

# List latest pact versions
pact-broker list-latest-pact-versions \
  --broker-base-url https://pact-broker.example.com
```

```typescript
// ════════════════════════════════════════════════════════════════
// PUBLISHING PACTS PROGRAMMATICALLY
// ════════════════════════════════════════════════════════════════

import { Publisher } from '@pact-foundation/pact';

async function publishPacts() {
  const publisher = new Publisher({
    pactBroker: process.env.PACT_BROKER_URL!,
    pactBrokerToken: process.env.PACT_BROKER_TOKEN,
    pactFilesOrDirs: ['./pacts'],
    consumerVersion: process.env.GIT_COMMIT!,
    branch: process.env.GIT_BRANCH,
    tags: [process.env.GIT_BRANCH!, 'ci'],
  });

  await publisher.publish();
  console.log('Pacts published successfully');
}

publishPacts().catch(console.error);
```

---

## 4. Advanced Patterns

```typescript
// ════════════════════════════════════════════════════════════════
// MESSAGE PACTS (Event-Driven)
// ════════════════════════════════════════════════════════════════

import { MessageConsumerPact, synchronousBodyHandler } from '@pact-foundation/pact';

describe('Order Events Consumer', () => {
  const messagePact = new MessageConsumerPact({
    consumer: 'NotificationService',
    provider: 'OrderService',
    dir: './pacts',
  });

  it('handles OrderCreated event', async () => {
    await messagePact
      .given('order is created')
      .expectsToReceive('an order created event')
      .withContent({
        eventType: 'OrderCreated',
        orderId: like('order-123'),
        userId: like('user-456'),
        total: like(99.99),
        items: eachLike({
          productId: like('prod-1'),
          quantity: like(2),
        }),
        timestamp: like('2024-01-15T10:00:00Z'),
      })
      .verify(synchronousBodyHandler(async (message) => {
        // Test your message handler
        const handler = new OrderEventHandler();
        await handler.handle(message);
        
        // Verify side effects
        expect(handler.notificationsSent).toBe(1);
      }));
  });
});

// Provider verification for messages
describe('OrderService Message Provider', () => {
  it('produces OrderCreated event correctly', async () => {
    const verifier = new MessageProviderPact({
      provider: 'OrderService',
      pactBrokerUrl: process.env.PACT_BROKER_URL,
      
      messageProviders: {
        'an order created event': async () => {
          // Return the actual message your service produces
          return createOrderCreatedEvent({
            orderId: 'order-123',
            userId: 'user-456',
            total: 99.99,
            items: [{ productId: 'prod-1', quantity: 2 }],
          });
        },
      },
      
      stateHandlers: {
        'order is created': async () => {
          // Set up state if needed
        },
      },
    });

    await verifier.verify();
  });
});

// ════════════════════════════════════════════════════════════════
// MATCHING RULES
// ════════════════════════════════════════════════════════════════

import { MatchersV3 } from '@pact-foundation/pact';

const {
  like,           // Match type, any value
  eachLike,       // Array with at least one element matching
  regex,          // Match regex pattern
  string,         // Must be string
  integer,        // Must be integer
  decimal,        // Must be decimal
  boolean,        // Must be boolean
  datetime,       // ISO datetime format
  uuid,           // UUID format
  includes,       // Contains substring
  nullValue,      // Explicit null
} = MatchersV3;

// Example usage
const responseBody = {
  id: uuid(),
  name: string('John'),
  email: regex(/^[\w.]+@[\w.]+\.\w+$/, 'test@example.com'),
  age: integer(25),
  balance: decimal(100.50),
  active: boolean(true),
  createdAt: datetime("yyyy-MM-dd'T'HH:mm:ss'Z'"),
  tags: eachLike(string('tag1')),
  metadata: like({
    source: string('web'),
    version: integer(1),
  }),
};
```

---

## 5. CI/CD Integration

```yaml
# ════════════════════════════════════════════════════════════════
# GITHUB ACTIONS - CONSUMER WORKFLOW
# ════════════════════════════════════════════════════════════════

name: Consumer Contract Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  contract-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run contract tests
        run: npm run test:contract:consumer

      - name: Publish pacts to broker
        if: github.ref == 'refs/heads/main' || github.event_name == 'pull_request'
        run: |
          npx pact-broker publish ./pacts \
            --broker-base-url ${{ secrets.PACT_BROKER_URL }} \
            --broker-token ${{ secrets.PACT_BROKER_TOKEN }} \
            --consumer-app-version ${{ github.sha }} \
            --branch ${{ github.head_ref || github.ref_name }}

      - name: Can I Deploy?
        run: |
          npx pact-broker can-i-deploy \
            --pacticipant OrderService \
            --version ${{ github.sha }} \
            --to-environment production \
            --broker-base-url ${{ secrets.PACT_BROKER_URL }} \
            --broker-token ${{ secrets.PACT_BROKER_TOKEN }}

# ════════════════════════════════════════════════════════════════
# GITHUB ACTIONS - PROVIDER WORKFLOW
# ════════════════════════════════════════════════════════════════

name: Provider Contract Verification

on:
  push:
    branches: [main]
  workflow_dispatch:
  repository_dispatch:
    types: [pact-changed]  # Triggered by Pact Broker webhook

jobs:
  verify-contracts:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm ci

      - name: Start provider service
        run: |
          npm run start:test &
          sleep 10

      - name: Verify contracts
        run: |
          npm run test:contract:provider
        env:
          PACT_BROKER_URL: ${{ secrets.PACT_BROKER_URL }}
          PACT_BROKER_TOKEN: ${{ secrets.PACT_BROKER_TOKEN }}
          GIT_COMMIT: ${{ github.sha }}
          GIT_BRANCH: ${{ github.ref_name }}
          CI: true

      - name: Can I Deploy?
        run: |
          npx pact-broker can-i-deploy \
            --pacticipant UserService \
            --version ${{ github.sha }} \
            --to-environment production \
            --broker-base-url ${{ secrets.PACT_BROKER_URL }}

      - name: Record deployment
        if: github.ref == 'refs/heads/main'
        run: |
          npx pact-broker record-deployment \
            --pacticipant UserService \
            --version ${{ github.sha }} \
            --environment production \
            --broker-base-url ${{ secrets.PACT_BROKER_URL }}
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# CONTRACT TESTING PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Over-specifying contracts
# Bad
body: {
  id: 123,           # Exact value
  name: "John Doe",  # Exact value
  timestamp: "2024-01-15T10:00:00Z"  # Exact value
}
# Breaks if any value differs

# Good
body: {
  id: integer(123),
  name: string("John Doe"),
  timestamp: datetime("yyyy-MM-dd'T'HH:mm:ss'Z'"),
}
# Matches structure and type, not exact values

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Not testing error scenarios
# Bad
# Only test happy paths
# Consumer breaks when provider returns errors

# Good
# Test 400, 404, 500 responses
# Consumer handles errors gracefully
.willRespondWith({ status: 404, body: { error: "Not found" }})

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: Shared state between interactions
# Bad
# Interaction 1 creates user
# Interaction 2 assumes user exists
# Order-dependent, flaky

# Good
# Each interaction uses state handlers
.given('user 123 exists')
# State setup is explicit and independent

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Testing provider implementation
# Bad
# Contract tests internal details
# "Field should be fetched from table X"

# Good
# Contract tests interface only
# "Response should include id, name, email"
# Provider can change implementation freely

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Not publishing to broker
# Bad
# Pact files in repo only
# No version tracking
# Can't use can-i-deploy

# Good
# Publish to broker in CI
# Version with git commit
# Enable can-i-deploy checks

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Manual state setup
# Bad
# "Go to staging and create user 123"
# Fragile, not reproducible

# Good
stateHandlers: {
  'user 123 exists': async () => {
    await createUser({ id: 123, ... });
  }
}
# Automated, reproducible
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is contract testing?"**
> "Testing that services can communicate correctly by verifying against agreed-upon contracts. Consumer defines expectations (request/response), provider verifies it meets them. Catches API breaking changes without requiring running services together."

**Q: "What is consumer-driven contracts?"**
> "Consumers define what they need from providers. Consumer tests generate contracts, providers verify against them. If consumer needs fields A, B, C, provider must return those. Provider can add field D without breaking contract."

**Q: "How is it different from integration testing?"**
> "Integration tests: Call real services over network, test everything together. Slow, flaky. Contract tests: Test against contracts (JSON files), no network. Fast, reliable. Contract tests verify compatibility, integration tests verify behavior."

### Intermediate Questions

**Q: "What is a Pact Broker?"**
> "Central repository for contracts. Stores pact files with versions, tracks which versions are verified, provides can-i-deploy check. Shows compatibility matrix across services. Enables webhooks to trigger provider verification when contracts change."

**Q: "What is can-i-deploy?"**
> "Pact Broker check that verifies deployment safety. Checks if your version's contracts are verified against target environment. If OrderService v2 is verified against UserService in production, can-i-deploy returns success. Blocks deployments with unverified contracts."

**Q: "How do you handle breaking changes?"**
> "1) Consumer publishes new contract with changed requirement. 2) Provider verification fails. 3) Provider team is notified. 4) Provider adds support for new contract. 5) Verification passes. 6) Both can deploy. Contracts enable discussion before breaking production."

### Advanced Questions

**Q: "How do you handle async/event-driven contracts?"**
> "Pact supports message contracts. Consumer defines expected message format, provider verifies it produces messages matching format. Same flow: consumer publishes, provider verifies. Works for Kafka, RabbitMQ, SNS. Message producers are 'providers', consumers are 'consumers'."

**Q: "How do you manage contract testing at scale?"**
> "Pact Broker for centralization. Webhooks to trigger provider verification on contract changes. Consumer version selectors to verify against specific branches/environments. Pending pacts for WIP features. Tags for environments (dev, staging, prod). can-i-deploy in all pipelines."

**Q: "Contract testing vs API schema validation?"**
> "Schema validation (OpenAPI): Validates request/response structure. Provider-driven - provider defines schema. Contract testing: Consumer-driven - consumers define needs. Tests actual usage, not theoretical schema. Both are valuable - schema for documentation, contracts for compatibility."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│               CONTRACT TESTING CHECKLIST                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CONSUMER SIDE:                                                 │
│  □ Write tests defining expected requests/responses            │
│  □ Use matchers (like, regex) not exact values                 │
│  □ Test success AND error scenarios                            │
│  □ Publish pacts to broker with version                        │
│                                                                 │
│  PROVIDER SIDE:                                                 │
│  □ Implement state handlers for test scenarios                 │
│  □ Verify against broker in CI                                 │
│  □ Publish verification results                                │
│                                                                 │
│  CI/CD:                                                         │
│  □ Publish pacts on consumer PR/merge                          │
│  □ Verify contracts on provider changes                        │
│  □ can-i-deploy before deployment                              │
│  □ Record deployments to broker                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

PACT FLOW:
┌────────────────────────────────────────────────────────────────┐
│ 1. Consumer: Write test → Generate pact file                   │
│ 2. Consumer: Publish pact to broker                            │
│ 3. Provider: Fetch pacts from broker                           │
│ 4. Provider: Verify against real API                           │
│ 5. Provider: Publish verification results                      │
│ 6. Both: can-i-deploy before release                           │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

