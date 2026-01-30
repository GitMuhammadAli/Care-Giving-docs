# 🏗️ Microservices Architecture Complete Guide

> A comprehensive guide to designing, building, and operating microservices-based systems.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Microservices decompose an application into small, autonomous services organized around business capabilities, each owning its data and deployable independently."

### The 6 Key Concepts (Remember These!)
```
1. BOUNDED CONTEXT   → Each service owns one business domain
2. DATABASE PER SVC  → Services don't share databases
3. API COMMUNICATION → Services talk via REST/gRPC/events
4. INDEPENDENT DEPLOY → Each service has its own CI/CD
5. FAILURE ISOLATION → One service failure doesn't crash others
6. BACKPRESSURE      → Slow down instead of crash under heavy load
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Bounded context"** | "Services align with bounded contexts from DDD" |
| **"Conway's Law"** | "Team structure should match service boundaries - Conway's Law" |
| **"Saga pattern"** | "Distributed transactions use the Saga pattern" |
| **"Circuit breaker"** | "We prevent cascading failures with circuit breakers" |
| **"Backpressure"** | "We apply backpressure to slow down instead of crash" |
| **"Load shedding"** | "Under extreme load, we shed non-critical requests" |
| **"Event sourcing"** | "Event sourcing gives us an audit trail" |
| **"Distributed monolith"** | "Wrong boundaries create a distributed monolith" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Team size for microservices | **20+ developers** | Below this, monolith is simpler |
| Service timeout | **5-30 seconds** | Don't wait forever |
| Circuit breaker threshold | **5 failures** | Then open circuit |
| Retry attempts | **3** | With exponential backoff |

### The "Wow" Statement (Memorize This!)
> "Most microservices failures come from premature decomposition. Teams split before understanding the domain and create a distributed monolith - all the complexity with none of the benefits. Start with a modular monolith, identify seams, extract services when there's genuine need."

### Quick Architecture Drawing (Draw This!)
```
            ┌─── User Service ─── [DB]
            │
API Gateway ├─── Order Service ─── [DB]
            │          │
            │          ▼ (events)
            └─── Payment Service ─── [DB]
```

### Interview Rapid Fire (Practice These!)

**Q: "What are microservices?"**
> "Small, autonomous services around business capabilities, own their data, deploy independently."

**Q: "When NOT to use microservices?"**
> "Small team, simple domain, early startup, unclear requirements."

**Q: "How do you handle distributed transactions?"**
> "Saga pattern - choreography for simple flows, orchestration for complex ones."

**Q: "What if a service is down?"**
> "Circuit breaker, retry with backoff, fallback response, graceful degradation."

**Q: "What is backpressure?"**
> "When overloaded, slow down responses instead of crashing. Better slow than dead."

**Q: "What is load shedding?"**
> "Under extreme load, drop non-critical requests to keep critical ones working."

**Q: "What's the hardest part?"**
> "Getting service boundaries right. Wrong boundaries = distributed monolith."

### Microservices vs Monolith (Memorize This Table!)
| | Monolith | Microservices |
|--|----------|---------------|
| **Team** | < 20 devs | 20+ devs |
| **Deploy** | All together | Independent |
| **Scale** | Everything | Per service |
| **Data** | Shared DB | DB per service |
| **Complexity** | Lower | Higher |

---

## 🎯 How to Explain Like a Senior Developer

### The Perfect Answer: "What are Microservices?"

**❌ Junior Answer:**
> "Microservices is when you split your application into smaller services that communicate over HTTP."

**❌ Textbook Answer (sounds memorized):**
> "Microservices is an architectural style that structures an application as a collection of loosely coupled services."

**✅ Senior Answer:**
> "Microservices is an architectural approach where you decompose an application into small, autonomous services - each owning a specific business capability and its data.
>
> The key insight is organizing around business domains, not technical layers. Instead of having a 'database team' or 'UI team', you have a 'payments team' that owns the entire payment service end-to-end.
>
> The tradeoff is complexity - you're trading in-process method calls for network calls, ACID transactions for eventual consistency, and simple deployments for orchestration. It really only makes sense when you have a large team or genuinely different scaling requirements.
>
> I've seen teams adopt microservices too early and create a 'distributed monolith' - all the complexity with none of the benefits. The rule of thumb is: if you can't split your team, don't split your service."

---

### Structure Your Answer

```
┌─────────────────────────────────────────────────────────────┐
│           THE SENIOR DEVELOPER ANSWER STRUCTURE              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. WHAT (Clear definition)                                 │
│     "Microservices is an architectural approach where..."   │
│                                                              │
│  2. KEY INSIGHT (The 'aha' moment)                          │
│     "The key insight is organizing around business          │
│      domains, not technical layers..."                      │
│                                                              │
│  3. TRADEOFFS (Shows maturity)                              │
│     "The tradeoff is... You're trading X for Y"             │
│                                                              │
│  4. WHEN TO USE (Practical wisdom)                          │
│     "It makes sense when... It doesn't make sense when..."  │
│                                                              │
│  5. REAL EXPERIENCE (Credibility)                           │
│     "I've seen teams..." or "In my experience..."           │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

### Follow-up Questions They WILL Ask

| They Ask | Your Answer Should Cover |
|----------|--------------------------|
| "Monolith vs Microservices?" | Team size, scaling needs, deployment frequency, complexity |
| "How do services communicate?" | Sync (REST/gRPC) vs Async (events/queues), when to use each |
| "How do you handle transactions?" | Saga pattern (choreography vs orchestration), eventual consistency |
| "How do you handle failures?" | Circuit breaker, retry, timeout, bulkhead, fallback |
| "What about data consistency?" | Database per service, event-driven sync, CQRS if needed |
| "How do you deploy?" | Kubernetes, CI/CD per service, blue-green, canary |
| "What about debugging?" | Distributed tracing (Jaeger), correlation IDs, centralized logging |
| "When should you NOT use microservices?" | Small team, simple domain, early startup, unclear requirements |

---

### Explain to Different Audiences

**To a Non-Technical Person (CEO/PM):**
> "Instead of one big application, we have many small applications that work together - like a team of specialists instead of one person doing everything. Each team owns one piece completely. This lets us move faster and scale specific parts when needed."

**To a Junior Developer:**
> "Think of it like splitting a restaurant into specialized stations - one for grilling, one for salads, one for desserts. Each station has its own chef and equipment. They communicate through orders, not by sharing the same pan. This way, if the grill station gets busy, you can add more grills without changing the salad station."

**To a Senior Developer / Architect:**
> "It's domain-driven decomposition with independent deployment and data ownership. Services communicate via async events for loose coupling, sync APIs when we need immediate consistency. We use Sagas for distributed transactions, circuit breakers for fault tolerance, and the API gateway handles cross-cutting concerns. The key is defining bounded contexts correctly - Conway's Law matters here."

**To an Interviewer (Show depth + invite questions):**
> Give the senior answer, then: "The success really depends on getting the service boundaries right - I like to start with a modular monolith and extract services when the domain is clear. Would you like me to walk through how I'd design a specific system?"

---

### Real Conversation Example

**Interviewer:** "What are microservices and when would you use them?"

**You:** "Microservices is an architectural approach where you decompose an application into small, autonomous services - each owning a specific business capability and its data.

The key characteristics are:
1. **Independent deployment** - each service has its own CI/CD pipeline
2. **Database per service** - no shared database, services own their data
3. **Organized by business capability** - a 'payment service' not a 'database service'
4. **Loose coupling** - services communicate via APIs or events, not shared code

I'd use microservices when:
- Team is large enough (20+ developers) that coordination becomes a bottleneck
- Different parts of the system have different scaling needs
- We need different technology stacks for different components
- We want to deploy frequently and independently

I'd avoid microservices when:
- Small team or early-stage startup - the overhead isn't worth it
- Domain isn't well understood yet - hard to draw the right boundaries
- Simple CRUD application - unnecessary complexity"

**Interviewer:** "How do you handle a transaction that spans multiple services?"

**You:** "Since ACID transactions don't work across service boundaries, we use the Saga pattern.

There are two approaches:

**Choreography** - Each service publishes events and others react. Order service publishes 'OrderCreated', payment service listens, processes payment, publishes 'PaymentCompleted', inventory service listens and reserves stock, etc. It's decentralized but can be hard to follow the flow.

**Orchestration** - A saga coordinator controls the flow. It calls each service in sequence, tracks state, and if anything fails, it calls compensating transactions in reverse order - like refunding a payment if inventory reservation fails.

I prefer orchestration for complex flows because it's easier to understand and debug, even though it introduces a central coordinator."

**Interviewer:** "What if the payment service is down?"

**You:** "This is where resilience patterns come in.

**Circuit Breaker** - After a threshold of failures, we stop trying and fail fast. This prevents resource exhaustion and gives the payment service time to recover.

**Retry with exponential backoff** - For transient failures, we retry with increasing delays: 1s, 2s, 4s, etc., plus some random jitter to prevent thundering herd.

**Fallback** - Maybe we can't process payment now, but we can queue the order as 'pending payment' and process it later.

**Timeout** - We never wait forever. If payment service doesn't respond in 5 seconds, we timeout and handle it.

The key is designing for failure from the start, not as an afterthought."

---

### Power Phrases That Show Expertise

| Phrase | What It Shows |
|--------|---------------|
| "Bounded context" | You know DDD |
| "Conway's Law" | You understand org dynamics |
| "Distributed monolith" | You've seen failures |
| "Eventual consistency" | You understand tradeoffs |
| "Saga pattern" | You know distributed transactions |
| "Circuit breaker" | You know fault tolerance |
| "Database per service" | You know data ownership |
| "Event-driven" | You know async patterns |
| "Strangler fig" | You know migration strategies |
| "API composition" | You know data aggregation |
| "Choreography vs orchestration" | Deep saga knowledge |

---

### Common Interview Traps & How to Handle

**Trap 1: "So microservices are always better than monoliths?"**

**Your response:**
> "No, it's about tradeoffs. Monoliths are simpler to develop, deploy, and debug. They're great for small teams and when the domain isn't well understood. I'd always start with a well-structured modular monolith and extract services only when there's a clear need - like a specific module needs independent scaling or a separate team will own it."

---

**Trap 2: "How small should a microservice be?"**

**Your response:**
> "There's no perfect size. The heuristic I use is: one service per bounded context, owned by one team. If you can't fit the service in your head, it's too big. If you're making constant changes across multiple services for a single feature, your boundaries are wrong. Amazon's 'two-pizza team' rule is a good proxy - if one team can't own it, split it."

---

**Trap 3: "Have you implemented microservices?"**

**If yes:**
> "Yes, I worked on [project]. We had [X] services. The key challenges were [Y]. One thing I'd do differently is [Z]."

**If no (be honest, show knowledge):**
> "I haven't built a production microservices system from scratch, but I've worked with systems that use microservice patterns. I've studied the architecture deeply - the patterns like Saga, CQRS, event sourcing - and I understand both the benefits and the operational complexity involved. Given my experience with [relevant skills], I'm confident I could contribute effectively."

---

### What NOT to Say

| ❌ Don't Say | ✅ Say Instead |
|-------------|----------------|
| "Split by technical layers (API, DB, Logic)" | "Split by business domain (payments, users, orders)" |
| "REST is the only way" | "REST for sync, events/queues for async" |
| "Each service is tiny" | "Service size matches team and domain boundaries" |
| "Microservices are always better" | "It depends on team size, complexity, and scaling needs" |
| "Just use Docker and you're done" | "Containerization is one piece; you need service discovery, observability, CI/CD" |
| "We'll figure out the boundaries later" | "Getting boundaries wrong creates a distributed monolith" |

---

### The "Wow" Answers That Impress

**On why microservices fail:**
> "Most microservices failures I've seen come from premature decomposition. Teams split before they understand the domain, create wrong boundaries, and end up with a distributed monolith - all the complexity of microservices with none of the benefits. The better approach is starting with a modular monolith, identifying clear seams over time, and extracting services when there's a genuine need."

**On testing:**
> "Testing microservices requires a different approach. Unit tests for business logic, contract tests for service boundaries - these catch breaking changes before deployment. The key is testing the contracts, not mock implementations. Tools like Pact make this manageable."

**On the hardest part:**
> "The hardest part isn't the technology - it's getting the boundaries right. Technical challenges like distributed transactions and eventual consistency have known solutions. But if you draw the wrong service boundaries, you'll be fighting that decision forever. That's why I like starting with domain experts, doing Event Storming sessions, and being willing to refactor boundaries early."

---

1. [Introduction](#introduction)
2. [Core Concepts](#core-concepts)
3. [When to Use Microservices](#when-to-use-microservices)
4. [Service Design Principles](#service-design-principles)
5. [Communication Patterns](#communication-patterns)
6. [Data Management](#data-management)
7. [Service Discovery](#service-discovery)
8. [API Gateway](#api-gateway)
9. [Resilience Patterns](#resilience-patterns)
10. [Deployment Strategies](#deployment-strategies)
11. [Monitoring & Observability](#monitoring--observability)
12. [Security](#security)
13. [Testing Strategies](#testing-strategies)
14. [Migration from Monolith](#migration-from-monolith)
15. [Common Pitfalls](#common-pitfalls)
16. [Interview Questions](#interview-questions)

---

## Introduction

### What are Microservices?

Microservices architecture is a design approach where an application is built as a collection of small, independent services that:

- Run in their own process
- Communicate via lightweight protocols (HTTP/REST, gRPC, messaging)
- Are independently deployable
- Are organized around business capabilities
- Can be written in different programming languages
- Can use different data storage technologies

### Monolith vs Microservices

```
┌─────────────────────────────────────────────────────────────────┐
│                        MONOLITH                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                    Single Application                    │   │
│   │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐   │   │
│   │  │   User   │ │  Order   │ │ Payment  │ │ Inventory│   │   │
│   │  │  Module  │ │  Module  │ │  Module  │ │  Module  │   │   │
│   │  └──────────┘ └──────────┘ └──────────┘ └──────────┘   │   │
│   │                                                         │   │
│   │  ┌─────────────────────────────────────────────────┐   │   │
│   │  │              Shared Database                     │   │   │
│   │  └─────────────────────────────────────────────────┘   │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                      MICROSERVICES                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐    │
│   │   User   │   │  Order   │   │ Payment  │   │Inventory │    │
│   │ Service  │   │ Service  │   │ Service  │   │ Service  │    │
│   └────┬─────┘   └────┬─────┘   └────┬─────┘   └────┬─────┘    │
│        │              │              │              │           │
│   ┌────┴────┐    ┌────┴────┐   ┌────┴────┐   ┌────┴────┐      │
│   │   DB    │    │   DB    │   │   DB    │   │   DB    │      │
│   └─────────┘    └─────────┘   └─────────┘   └─────────┘      │
│                                                                  │
│              Communication via APIs / Messages                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Comparison Table

| Aspect | Monolith | Microservices |
|--------|----------|---------------|
| **Deployment** | All or nothing | Independent per service |
| **Scaling** | Scale entire app | Scale specific services |
| **Technology** | Single stack | Polyglot (multiple stacks) |
| **Team Structure** | One large team | Small autonomous teams |
| **Data** | Shared database | Database per service |
| **Complexity** | Simpler initially | Complex from start |
| **Latency** | In-process calls | Network calls |
| **Consistency** | ACID transactions | Eventual consistency |
| **Debugging** | Easier | Harder (distributed) |
| **Development Speed** | Slower at scale | Faster per service |

---

## Core Concepts

### 1. Service Boundaries

Services should be organized around **business capabilities**, not technical layers.

```
❌ BAD: Technical Boundaries
├── UI Service
├── Business Logic Service
└── Data Access Service

✅ GOOD: Business Boundaries
├── User Service (accounts, profiles, auth)
├── Order Service (cart, checkout, orders)
├── Payment Service (transactions, refunds)
├── Inventory Service (stock, warehouses)
└── Notification Service (email, SMS, push)
```

### 2. Bounded Context (Domain-Driven Design)

Each service owns a specific domain with clear boundaries.

```javascript
// User Service - owns user-related concepts
class User {
  id: string;
  email: string;
  profile: UserProfile;
  // User service doesn't know about orders
}

// Order Service - has its own view of user
class OrderCustomer {
  userId: string;      // Reference only
  shippingAddress: Address;
  // Order service doesn't need full user profile
}

// Payment Service - minimal user info
class PaymentCustomer {
  userId: string;
  billingDetails: BillingInfo;
  // Just what's needed for payments
}
```

### 3. Single Responsibility

Each service should do one thing well.

```
Order Service responsibilities:
✅ Create orders
✅ Update order status
✅ Cancel orders
✅ Order history

❌ NOT Order Service responsibility:
❌ Process payments (Payment Service)
❌ Update inventory (Inventory Service)
❌ Send notifications (Notification Service)
❌ Manage user accounts (User Service)
```

### 4. Loose Coupling

Services should be independent with minimal dependencies.

```javascript
// ❌ Tight Coupling - Direct database access
class OrderService {
  async createOrder(data) {
    // Directly accessing user database - BAD!
    const user = await userDatabase.query('SELECT * FROM users WHERE id = ?', data.userId);
    // Directly accessing inventory database - BAD!
    await inventoryDatabase.query('UPDATE products SET stock = stock - 1');
  }
}

// ✅ Loose Coupling - API calls
class OrderService {
  async createOrder(data) {
    // Call User service API
    const user = await this.userClient.getUser(data.userId);
    // Call Inventory service API
    await this.inventoryClient.reserveStock(data.productId, data.quantity);
  }
}
```

### 5. High Cohesion

Related functionality should be grouped together.

```javascript
// ✅ High Cohesion - Payment Service
class PaymentService {
  processPayment() { }
  refundPayment() { }
  getPaymentHistory() { }
  validatePaymentMethod() { }
  // All payment-related
}

// ❌ Low Cohesion - Mixed responsibilities
class MixedService {
  processPayment() { }
  sendEmail() { }        // Should be in Notification Service
  updateInventory() { }  // Should be in Inventory Service
}
```

---

## When to Use Microservices

### ✅ Use Microservices When:

| Scenario | Why Microservices Help |
|----------|----------------------|
| **Large team (20+ developers)** | Teams can work independently |
| **Different scaling needs** | Scale hot services independently |
| **Multiple tech requirements** | Use best tool for each job |
| **High deployment frequency** | Deploy services independently |
| **Complex domain** | Clear boundaries reduce complexity |
| **High availability required** | Failure isolation |

### ❌ Avoid Microservices When:

| Scenario | Why Monolith is Better |
|----------|----------------------|
| **Small team (< 10 developers)** | Overhead not worth it |
| **Early-stage startup** | Need to iterate quickly |
| **Simple domain** | Unnecessary complexity |
| **Unclear requirements** | Hard to define boundaries |
| **Limited DevOps expertise** | Operational complexity |
| **Tight budget** | Infrastructure costs higher |

### Decision Framework

```
┌─────────────────────────────────────────────────────────────┐
│                  SHOULD I USE MICROSERVICES?                 │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   Team Size > 20?                                           │
│       YES ──────────────────────────────────┐               │
│       NO ───┐                               │               │
│             │                               │               │
│             ▼                               ▼               │
│   Need Different Scaling?            Consider Microservices │
│       YES ──────────────────────────────────┘               │
│       NO ───┐                                               │
│             │                                               │
│             ▼                                               │
│   Complex Domain?                                           │
│       YES ──────────────────────────────────┐               │
│       NO ───┐                               │               │
│             │                               ▼               │
│             ▼                         Start with Modular    │
│   Stick with Monolith                 Monolith, Migrate     │
│   (Well-structured)                   When Needed           │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Service Design Principles

### 1. Design for Failure

Assume any service can fail at any time.

```javascript
class OrderService {
  async createOrder(orderData) {
    try {
      // Attempt to reserve inventory
      const reserved = await this.inventoryClient.reserve(orderData.items, {
        timeout: 5000,
        retries: 3
      });
      
      if (!reserved) {
        return { success: false, error: 'INVENTORY_UNAVAILABLE' };
      }
      
      // Attempt payment
      const payment = await this.paymentClient.process(orderData.payment, {
        timeout: 10000,
        retries: 2
      });
      
      if (!payment.success) {
        // Rollback inventory reservation
        await this.inventoryClient.release(orderData.items);
        return { success: false, error: 'PAYMENT_FAILED' };
      }
      
      return { success: true, orderId: order.id };
      
    } catch (error) {
      // Handle timeout, network errors, etc.
      await this.handleFailure(orderData, error);
      return { success: false, error: 'SERVICE_ERROR' };
    }
  }
}
```

### 2. Design for Scalability

```javascript
// Stateless services - can scale horizontally
class ProductService {
  // No instance state - any instance can handle any request
  async getProduct(id) {
    return this.repository.findById(id);
  }
}

// Externalize state
class SessionService {
  constructor(redisClient) {
    this.redis = redisClient; // State in Redis, not in memory
  }
  
  async getSession(sessionId) {
    return this.redis.get(`session:${sessionId}`);
  }
}
```

### 3. API First Design

Design APIs before implementation.

```yaml
# OpenAPI Specification
openapi: 3.0.0
info:
  title: Order Service API
  version: 1.0.0

paths:
  /orders:
    post:
      summary: Create a new order
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateOrderRequest'
      responses:
        '201':
          description: Order created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
        '400':
          description: Invalid request
        '503':
          description: Service unavailable

components:
  schemas:
    CreateOrderRequest:
      type: object
      required:
        - customerId
        - items
      properties:
        customerId:
          type: string
        items:
          type: array
          items:
            $ref: '#/components/schemas/OrderItem'
```

### 4. Backward Compatibility

Never break existing consumers.

```javascript
// Version 1 response
{
  "user": {
    "name": "John Doe",
    "email": "john@example.com"
  }
}

// Version 2 - ADD fields, don't remove or rename
{
  "user": {
    "name": "John Doe",           // Keep for backward compat
    "firstName": "John",          // New field
    "lastName": "Doe",            // New field
    "email": "john@example.com",
    "emailVerified": true         // New field
  }
}

// Use API versioning for breaking changes
// GET /api/v1/users - old format
// GET /api/v2/users - new format
```

---

## Communication Patterns

### 1. Synchronous Communication (REST/gRPC)

```
┌──────────┐         ┌──────────┐         ┌──────────┐
│  Client  │──HTTP──▶│  Order   │──HTTP──▶│ Payment  │
│          │◀────────│ Service  │◀────────│ Service  │
└──────────┘         └──────────┘         └──────────┘
                          │
                          │ HTTP
                          ▼
                     ┌──────────┐
                     │Inventory │
                     │ Service  │
                     └──────────┘
```

**REST Example:**

```javascript
// Order Service calling Payment Service
class PaymentClient {
  constructor(baseUrl) {
    this.baseUrl = baseUrl;
  }
  
  async processPayment(paymentData) {
    const response = await fetch(`${this.baseUrl}/payments`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(paymentData),
      timeout: 5000
    });
    
    if (!response.ok) {
      throw new PaymentError(response.status, await response.json());
    }
    
    return response.json();
  }
}
```

**gRPC Example:**

```protobuf
// payment.proto
syntax = "proto3";

service PaymentService {
  rpc ProcessPayment(PaymentRequest) returns (PaymentResponse);
  rpc RefundPayment(RefundRequest) returns (RefundResponse);
}

message PaymentRequest {
  string order_id = 1;
  double amount = 2;
  string currency = 3;
  PaymentMethod method = 4;
}

message PaymentResponse {
  string transaction_id = 1;
  PaymentStatus status = 2;
}
```

```javascript
// gRPC client
const client = new PaymentServiceClient('payment-service:50051');

const response = await client.processPayment({
  orderId: 'order-123',
  amount: 99.99,
  currency: 'USD',
  method: PaymentMethod.CREDIT_CARD
});
```

### 2. Asynchronous Communication (Message Queue)

```
┌──────────┐         ┌──────────┐         ┌──────────┐
│  Order   │──Pub───▶│  Message │──Sub───▶│ Payment  │
│ Service  │         │  Broker  │         │ Service  │
└──────────┘         │ (Kafka/  │         └──────────┘
                     │ RabbitMQ)│
                     │          │──Sub───▶┌──────────┐
                     │          │         │Inventory │
                     │          │         │ Service  │
                     │          │         └──────────┘
                     │          │
                     │          │──Sub───▶┌──────────┐
                     └──────────┘         │  Email   │
                                          │ Service  │
                                          └──────────┘
```

**Event Publishing:**

```javascript
// Order Service publishes event
class OrderService {
  constructor(messageBroker) {
    this.broker = messageBroker;
  }
  
  async createOrder(orderData) {
    // Save order to database
    const order = await this.repository.save(orderData);
    
    // Publish event (fire and forget)
    await this.broker.publish('orders', {
      type: 'ORDER_CREATED',
      data: {
        orderId: order.id,
        customerId: order.customerId,
        items: order.items,
        totalAmount: order.totalAmount,
        timestamp: new Date().toISOString()
      }
    });
    
    return order;
  }
}
```

**Event Consuming:**

```javascript
// Payment Service subscribes to events
class PaymentEventHandler {
  constructor(messageBroker, paymentService) {
    this.broker = messageBroker;
    this.paymentService = paymentService;
    
    this.broker.subscribe('orders', this.handleOrderEvent.bind(this));
  }
  
  async handleOrderEvent(event) {
    switch (event.type) {
      case 'ORDER_CREATED':
        await this.paymentService.initiatePayment(event.data);
        break;
      case 'ORDER_CANCELLED':
        await this.paymentService.refundPayment(event.data.orderId);
        break;
    }
  }
}
```

### 3. Event-Driven Architecture

```javascript
// Domain Events
const OrderEvents = {
  ORDER_CREATED: 'order.created',
  ORDER_PAID: 'order.paid',
  ORDER_SHIPPED: 'order.shipped',
  ORDER_DELIVERED: 'order.delivered',
  ORDER_CANCELLED: 'order.cancelled'
};

// Event Structure
interface DomainEvent {
  id: string;           // Unique event ID
  type: string;         // Event type
  aggregateId: string;  // Entity ID
  aggregateType: string;// Entity type
  data: any;            // Event payload
  metadata: {
    timestamp: string;
    version: number;
    correlationId: string;
    causationId: string;
  };
}

// Example Event
{
  "id": "evt-123456",
  "type": "order.created",
  "aggregateId": "order-789",
  "aggregateType": "Order",
  "data": {
    "customerId": "cust-456",
    "items": [...],
    "totalAmount": 99.99
  },
  "metadata": {
    "timestamp": "2024-01-15T10:30:00Z",
    "version": 1,
    "correlationId": "req-abc",
    "causationId": null
  }
}
```

### Comparison: Sync vs Async

| Aspect | Synchronous | Asynchronous |
|--------|-------------|--------------|
| **Latency** | Adds to request time | No impact on request |
| **Coupling** | Temporal coupling | Loose coupling |
| **Reliability** | Dependent on all services | Resilient to failures |
| **Complexity** | Simpler to implement | More complex (queues, etc.) |
| **Debugging** | Easier to trace | Harder to trace |
| **Use Case** | Need immediate response | Fire and forget |

---

## Data Management

### Database Per Service Pattern

Each service owns its data and exposes it only through APIs.

```
┌─────────────────────────────────────────────────────────────┐
│                    DATABASE PER SERVICE                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   ┌──────────────┐    ┌──────────────┐    ┌──────────────┐ │
│   │ User Service │    │Order Service │    │Payment Svc   │ │
│   └──────┬───────┘    └──────┬───────┘    └──────┬───────┘ │
│          │                   │                   │          │
│          ▼                   ▼                   ▼          │
│   ┌──────────────┐    ┌──────────────┐    ┌──────────────┐ │
│   │  PostgreSQL  │    │   MongoDB    │    │  PostgreSQL  │ │
│   │   (Users)    │    │   (Orders)   │    │  (Payments)  │ │
│   └──────────────┘    └──────────────┘    └──────────────┘ │
│                                                              │
│   ❌ Services CANNOT directly access other service's DB     │
│   ✅ Services communicate via APIs only                     │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Saga Pattern (Distributed Transactions)

Since we can't use ACID transactions across services, use Sagas.

**Choreography-based Saga:**

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Order   │───▶│ Payment  │───▶│Inventory │───▶│ Shipping │
│ Service  │    │ Service  │    │ Service  │    │ Service  │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
     │               │               │               │
     │ OrderCreated  │ PaymentDone   │ StockReserved │ ShipmentCreated
     ▼               ▼               ▼               ▼
   Event ─────────▶ Event ─────────▶ Event ─────────▶ Event
   
   On Failure: Each service listens for failure events and compensates
```

```javascript
// Choreography Saga - Each service reacts to events
class OrderService {
  async createOrder(data) {
    const order = await this.repository.save({ ...data, status: 'PENDING' });
    await this.eventBus.publish('OrderCreated', order);
    return order;
  }
  
  // Listen for payment events
  @Subscribe('PaymentFailed')
  async handlePaymentFailed(event) {
    await this.repository.updateStatus(event.orderId, 'PAYMENT_FAILED');
    await this.eventBus.publish('OrderCancelled', { orderId: event.orderId });
  }
}

class PaymentService {
  @Subscribe('OrderCreated')
  async handleOrderCreated(event) {
    try {
      const payment = await this.processPayment(event.order);
      await this.eventBus.publish('PaymentCompleted', { 
        orderId: event.order.id,
        paymentId: payment.id 
      });
    } catch (error) {
      await this.eventBus.publish('PaymentFailed', { 
        orderId: event.order.id,
        reason: error.message 
      });
    }
  }
}
```

**Orchestration-based Saga:**

```
┌─────────────────────────────────────────────────────────────┐
│                      SAGA ORCHESTRATOR                       │
│                                                              │
│   ┌──────────────────────────────────────────────────────┐  │
│   │                  Order Saga                          │  │
│   │                                                      │  │
│   │   1. Create Order ─────────────────────────────────┐ │  │
│   │   2. Reserve Inventory ◀───────────────────────────┤ │  │
│   │   3. Process Payment ◀─────────────────────────────┤ │  │
│   │   4. Confirm Order ◀───────────────────────────────┘ │  │
│   │                                                      │  │
│   │   On Failure: Execute compensating transactions     │  │
│   │   - Cancel Payment                                  │  │
│   │   - Release Inventory                               │  │
│   │   - Cancel Order                                    │  │
│   └──────────────────────────────────────────────────────┘  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

```javascript
// Orchestration Saga
class OrderSaga {
  constructor(orderService, inventoryService, paymentService) {
    this.orderService = orderService;
    this.inventoryService = inventoryService;
    this.paymentService = paymentService;
  }
  
  async execute(orderData) {
    const sagaLog = [];
    
    try {
      // Step 1: Create Order
      const order = await this.orderService.create(orderData);
      sagaLog.push({ step: 'ORDER_CREATED', data: order });
      
      // Step 2: Reserve Inventory
      const reservation = await this.inventoryService.reserve(order.items);
      sagaLog.push({ step: 'INVENTORY_RESERVED', data: reservation });
      
      // Step 3: Process Payment
      const payment = await this.paymentService.process(order);
      sagaLog.push({ step: 'PAYMENT_PROCESSED', data: payment });
      
      // Step 4: Confirm Order
      await this.orderService.confirm(order.id);
      sagaLog.push({ step: 'ORDER_CONFIRMED' });
      
      return { success: true, order };
      
    } catch (error) {
      // Compensate in reverse order
      await this.compensate(sagaLog);
      return { success: false, error: error.message };
    }
  }
  
  async compensate(sagaLog) {
    for (const entry of sagaLog.reverse()) {
      switch (entry.step) {
        case 'PAYMENT_PROCESSED':
          await this.paymentService.refund(entry.data.id);
          break;
        case 'INVENTORY_RESERVED':
          await this.inventoryService.release(entry.data.reservationId);
          break;
        case 'ORDER_CREATED':
          await this.orderService.cancel(entry.data.id);
          break;
      }
    }
  }
}
```

### CQRS (Command Query Responsibility Segregation)

Separate read and write models.

```
┌─────────────────────────────────────────────────────────────┐
│                         CQRS                                 │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│                      ┌──────────────┐                       │
│                      │   Commands   │                       │
│                      │ (Write Ops)  │                       │
│                      └──────┬───────┘                       │
│                             │                                │
│                             ▼                                │
│   ┌─────────────────────────────────────────────────────┐   │
│   │              Write Model (Domain)                   │   │
│   │   - Complex business logic                          │   │
│   │   - Validations                                     │   │
│   │   - Domain events                                   │   │
│   └─────────────────────────────────────────────────────┘   │
│                             │                                │
│                             │ Events                         │
│                             ▼                                │
│   ┌─────────────────────────────────────────────────────┐   │
│   │              Read Model (Projections)               │   │
│   │   - Denormalized for queries                        │   │
│   │   - Optimized views                                 │   │
│   │   - Multiple projections                            │   │
│   └─────────────────────────────────────────────────────┘   │
│                             │                                │
│                             ▼                                │
│                      ┌──────────────┐                       │
│                      │   Queries    │                       │
│                      │ (Read Ops)   │                       │
│                      └──────────────┘                       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

```javascript
// Write Model - Complex domain logic
class OrderAggregate {
  constructor() {
    this.events = [];
  }
  
  createOrder(data) {
    // Validations
    if (!data.items.length) throw new Error('Order must have items');
    if (data.totalAmount <= 0) throw new Error('Invalid amount');
    
    // Apply business rules
    const order = {
      id: generateId(),
      ...data,
      status: 'CREATED',
      createdAt: new Date()
    };
    
    // Emit event
    this.events.push({
      type: 'OrderCreated',
      data: order
    });
    
    return order;
  }
}

// Read Model - Optimized for queries
class OrderReadModel {
  constructor(db) {
    this.db = db;
  }
  
  // Denormalized view
  async getOrderWithDetails(orderId) {
    return this.db.query(`
      SELECT 
        o.*,
        c.name as customer_name,
        c.email as customer_email,
        json_agg(oi.*) as items
      FROM order_projections o
      JOIN customer_projections c ON o.customer_id = c.id
      JOIN order_item_projections oi ON o.id = oi.order_id
      WHERE o.id = $1
      GROUP BY o.id, c.id
    `, [orderId]);
  }
  
  // Event handler to update projection
  async onOrderCreated(event) {
    await this.db.insert('order_projections', event.data);
  }
}
```

---

## Service Discovery

### Why Service Discovery?

In microservices, services need to find each other dynamically as instances scale up/down.

```
Without Service Discovery:
┌──────────┐                    ┌──────────┐
│  Order   │──── hardcoded ────▶│ Payment  │
│ Service  │     IP:PORT        │ Service  │
└──────────┘                    └──────────┘
❌ Breaks when Payment Service moves/scales

With Service Discovery:
┌──────────┐     ┌──────────┐     ┌──────────┐
│  Order   │────▶│ Service  │────▶│ Payment  │
│ Service  │     │ Registry │     │ Service  │
└──────────┘     └──────────┘     └──────────┘
✅ Dynamic lookup
```

### Client-Side Discovery

```javascript
// Service registers itself
class PaymentService {
  async start() {
    await this.serviceRegistry.register({
      name: 'payment-service',
      address: process.env.HOST,
      port: process.env.PORT,
      healthCheck: '/health'
    });
    
    // Send heartbeats
    setInterval(() => {
      this.serviceRegistry.heartbeat('payment-service');
    }, 10000);
  }
}

// Client discovers and load balances
class ServiceClient {
  constructor(serviceRegistry) {
    this.registry = serviceRegistry;
    this.cache = new Map();
  }
  
  async call(serviceName, endpoint, options) {
    // Get available instances
    const instances = await this.registry.getInstances(serviceName);
    
    if (instances.length === 0) {
      throw new Error(`No instances of ${serviceName} available`);
    }
    
    // Load balance (round-robin)
    const instance = this.selectInstance(instances);
    
    return fetch(`http://${instance.address}:${instance.port}${endpoint}`, options);
  }
  
  selectInstance(instances) {
    // Round-robin
    const index = this.counter++ % instances.length;
    return instances[index];
  }
}
```

### Server-Side Discovery (Load Balancer)

```
┌──────────┐     ┌──────────────┐     ┌──────────┐
│  Order   │────▶│    Load      │────▶│ Payment  │
│ Service  │     │   Balancer   │     │ Service  │
└──────────┘     │   (Nginx/    │     │ Instance1│
                 │    AWS ALB)  │     └──────────┘
                 │              │     ┌──────────┐
                 │              │────▶│ Payment  │
                 │              │     │ Instance2│
                 └──────────────┘     └──────────┘
```

### Tools

| Tool | Type | Features |
|------|------|----------|
| **Consul** | Service Mesh | Discovery, health checks, KV store |
| **Eureka** | Registry | Netflix OSS, Java-focused |
| **etcd** | KV Store | Distributed, consistent |
| **Kubernetes** | Platform | Built-in DNS-based discovery |
| **AWS Cloud Map** | Managed | AWS native |

### Kubernetes Service Discovery

```yaml
# Service definition
apiVersion: v1
kind: Service
metadata:
  name: payment-service
spec:
  selector:
    app: payment
  ports:
    - port: 80
      targetPort: 8080

---
# Other services can reach it at:
# http://payment-service (within same namespace)
# http://payment-service.namespace.svc.cluster.local (full DNS)
```

---

## API Gateway

### What is an API Gateway?

Single entry point for all client requests.

```
┌─────────────────────────────────────────────────────────────┐
│                       API GATEWAY                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   ┌──────────┐                                              │
│   │  Mobile  │──┐                                           │
│   │   App    │  │      ┌───────────────────────────────┐   │
│   └──────────┘  │      │        API Gateway            │   │
│                 │      │                               │   │
│   ┌──────────┐  ├─────▶│  • Authentication            │   │
│   │   Web    │──┤      │  • Rate Limiting             │   │
│   │   App    │  │      │  • Request Routing           │   │
│   └──────────┘  │      │  • Load Balancing            │   │
│                 │      │  • Response Caching          │   │
│   ┌──────────┐  │      │  • Request/Response Transform│   │
│   │  Third   │──┘      │  • API Composition           │   │
│   │  Party   │         │  • Monitoring/Logging        │   │
│   └──────────┘         └───────────────────────────────┘   │
│                                    │                        │
│                    ┌───────────────┼───────────────┐       │
│                    ▼               ▼               ▼       │
│              ┌──────────┐   ┌──────────┐   ┌──────────┐   │
│              │   User   │   │  Order   │   │ Payment  │   │
│              │ Service  │   │ Service  │   │ Service  │   │
│              └──────────┘   └──────────┘   └──────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Gateway Implementation

```javascript
// Simple API Gateway with Express
const express = require('express');
const { createProxyMiddleware } = require('http-proxy-middleware');
const rateLimit = require('express-rate-limit');

const app = express();

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
});
app.use(limiter);

// Authentication middleware
app.use(async (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }
  
  try {
    const user = await verifyToken(token);
    req.user = user;
    next();
  } catch {
    res.status(401).json({ error: 'Invalid token' });
  }
});

// Route to services
app.use('/api/users', createProxyMiddleware({
  target: 'http://user-service:3001',
  changeOrigin: true,
  pathRewrite: { '^/api/users': '' }
}));

app.use('/api/orders', createProxyMiddleware({
  target: 'http://order-service:3002',
  changeOrigin: true,
  pathRewrite: { '^/api/orders': '' }
}));

app.use('/api/payments', createProxyMiddleware({
  target: 'http://payment-service:3003',
  changeOrigin: true,
  pathRewrite: { '^/api/payments': '' }
}));

// API Composition - aggregate data from multiple services
app.get('/api/dashboard', async (req, res) => {
  const [user, orders, notifications] = await Promise.all([
    fetch('http://user-service:3001/profile').then(r => r.json()),
    fetch('http://order-service:3002/recent').then(r => r.json()),
    fetch('http://notification-service:3004/unread').then(r => r.json())
  ]);
  
  res.json({ user, orders, notifications });
});
```

### Backend for Frontend (BFF)

Different gateways for different clients.

```
┌──────────┐     ┌─────────────┐
│  Mobile  │────▶│ Mobile BFF  │──┐
│   App    │     │ (optimized) │  │
└──────────┘     └─────────────┘  │
                                  │    ┌──────────┐
┌──────────┐     ┌─────────────┐  ├───▶│ Services │
│   Web    │────▶│   Web BFF   │──┤    └──────────┘
│   App    │     │ (full data) │  │
└──────────┘     └─────────────┘  │
                                  │
┌──────────┐     ┌─────────────┐  │
│  Third   │────▶│ Public API  │──┘
│  Party   │     │   Gateway   │
└──────────┘     └─────────────┘
```

### Popular API Gateways

| Gateway | Type | Best For |
|---------|------|----------|
| **Kong** | Open Source | Full-featured, plugins |
| **AWS API Gateway** | Managed | AWS ecosystem |
| **Nginx** | Reverse Proxy | Performance, simplicity |
| **Traefik** | Cloud Native | Kubernetes, auto-discovery |
| **Express Gateway** | Node.js | Node.js ecosystem |

---

## Resilience Patterns

### 1. Circuit Breaker

Prevent cascading failures by failing fast.

```
┌─────────────────────────────────────────────────────────────┐
│                    CIRCUIT BREAKER                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   CLOSED ────────────▶ OPEN ────────────▶ HALF-OPEN        │
│     │                    │                    │             │
│     │ Failure            │ Timeout            │ Success     │
│     │ threshold          │ expires            │             │
│     │ reached            │                    │             │
│     │                    │                    ▼             │
│     │                    │              Try one request     │
│     │                    │                    │             │
│     │                    │         ┌──────────┴──────────┐  │
│     │                    │         │                     │  │
│     │                    │      Success               Failure│
│     │                    │         │                     │  │
│     │                    │         ▼                     ▼  │
│     ▼                    ◀───── CLOSED               OPEN   │
│   Normal                                                    │
│   operation                                                 │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

```javascript
class CircuitBreaker {
  constructor(options = {}) {
    this.failureThreshold = options.failureThreshold || 5;
    this.resetTimeout = options.resetTimeout || 30000;
    this.state = 'CLOSED';
    this.failures = 0;
    this.lastFailure = null;
  }
  
  async call(fn) {
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailure > this.resetTimeout) {
        this.state = 'HALF-OPEN';
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }
    
    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  onSuccess() {
    this.failures = 0;
    this.state = 'CLOSED';
  }
  
  onFailure() {
    this.failures++;
    this.lastFailure = Date.now();
    
    if (this.failures >= this.failureThreshold) {
      this.state = 'OPEN';
    }
  }
}

// Usage
const breaker = new CircuitBreaker({ failureThreshold: 3, resetTimeout: 10000 });

async function callPaymentService(data) {
  return breaker.call(() => paymentClient.process(data));
}
```

### 2. Retry with Backoff

```javascript
async function retryWithBackoff(fn, options = {}) {
  const { maxRetries = 3, baseDelay = 1000, maxDelay = 30000 } = options;
  
  let lastError;
  
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;
      
      // Don't retry on client errors
      if (error.status >= 400 && error.status < 500) {
        throw error;
      }
      
      // Calculate delay with exponential backoff + jitter
      const delay = Math.min(
        baseDelay * Math.pow(2, attempt) + Math.random() * 1000,
        maxDelay
      );
      
      console.log(`Retry ${attempt + 1}/${maxRetries} in ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  
  throw lastError;
}
```

### 3. Bulkhead Pattern

Isolate failures to prevent system-wide impact.

```javascript
// Separate thread pools/connection pools for different services
class BulkheadManager {
  constructor() {
    this.pools = new Map();
  }
  
  createPool(name, maxConcurrent) {
    this.pools.set(name, {
      maxConcurrent,
      current: 0,
      queue: []
    });
  }
  
  async execute(poolName, fn) {
    const pool = this.pools.get(poolName);
    
    if (pool.current >= pool.maxConcurrent) {
      // Queue or reject
      return new Promise((resolve, reject) => {
        pool.queue.push({ fn, resolve, reject });
      });
    }
    
    pool.current++;
    
    try {
      return await fn();
    } finally {
      pool.current--;
      this.processQueue(poolName);
    }
  }
  
  processQueue(poolName) {
    const pool = this.pools.get(poolName);
    if (pool.queue.length > 0 && pool.current < pool.maxConcurrent) {
      const { fn, resolve, reject } = pool.queue.shift();
      this.execute(poolName, fn).then(resolve).catch(reject);
    }
  }
}

// Usage - isolate payment calls from inventory calls
const bulkhead = new BulkheadManager();
bulkhead.createPool('payment', 10);   // Max 10 concurrent payment calls
bulkhead.createPool('inventory', 20); // Max 20 concurrent inventory calls

await bulkhead.execute('payment', () => paymentService.process(data));
```

### 4. Timeout Pattern

```javascript
function withTimeout(promise, ms) {
  const timeout = new Promise((_, reject) => {
    setTimeout(() => reject(new Error('Timeout')), ms);
  });
  
  return Promise.race([promise, timeout]);
}

// Usage
try {
  const result = await withTimeout(paymentService.process(data), 5000);
} catch (error) {
  if (error.message === 'Timeout') {
    // Handle timeout
  }
}
```

### 5. Fallback Pattern

```javascript
async function getProductWithFallback(productId) {
  try {
    // Try primary service
    return await productService.getProduct(productId);
  } catch (error) {
    console.log('Primary service failed, using fallback');
    
    try {
      // Try cache
      return await cache.get(`product:${productId}`);
    } catch {
      // Return default/static data
      return {
        id: productId,
        name: 'Product Unavailable',
        available: false
      };
    }
  }
}
```

### 6. Backpressure Pattern (Slow Down Instead of Crash)

> **"It's better to be slow than dead."** When under heavy load, slow down responses instead of crashing.

```javascript
// STRATEGY 1: Rate limit concurrent requests
class BackpressureHandler {
  constructor(maxConcurrent = 100) {
    this.maxConcurrent = maxConcurrent;
    this.active = 0;
    this.responseDelay = 0;
  }

  async middleware(req, res, next) {
    this.active++;

    // At 80% capacity: start slowing down
    if (this.active > this.maxConcurrent * 0.8) {
      this.responseDelay = Math.min(this.responseDelay + 100, 2000);
      await this.sleep(this.responseDelay);
      console.log(`Backpressure: ${this.active} active, delay ${this.responseDelay}ms`);
    } else {
      this.responseDelay = Math.max(this.responseDelay - 50, 0);
    }

    // At 100% capacity: reject new requests
    if (this.active > this.maxConcurrent) {
      this.active--;
      return res.status(429).json({
        error: 'Too Many Requests',
        message: 'Server under heavy load, please retry',
        retryAfter: 5
      });
    }

    try {
      await next();
    } finally {
      this.active--;
    }
  }

  sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// Usage
const backpressure = new BackpressureHandler(500);
app.use((req, res, next) => backpressure.middleware(req, res, next));
```

```javascript
// STRATEGY 2: Adaptive throttling based on latency
class AdaptiveThrottler {
  constructor() {
    this.targetLatency = 100;  // Target 100ms response
    this.currentRate = 1000;   // Start at 1000 req/s
    this.minRate = 10;
    this.maxRate = 10000;
  }

  adjustRate(actualLatency) {
    if (actualLatency > this.targetLatency * 2) {
      // Way too slow - cut rate in half
      this.currentRate = Math.max(this.currentRate * 0.5, this.minRate);
    } else if (actualLatency > this.targetLatency) {
      // Slow - reduce by 10%
      this.currentRate = Math.max(this.currentRate * 0.9, this.minRate);
    } else if (actualLatency < this.targetLatency * 0.5) {
      // Fast - increase by 10%
      this.currentRate = Math.min(this.currentRate * 1.1, this.maxRate);
    }
    console.log(`Rate adjusted to ${this.currentRate}/s`);
  }
}
```

### 7. Load Shedding Pattern (Drop Non-Critical Under Extreme Load)

> When overwhelmed, **drop non-critical requests** to keep critical functions working.

```javascript
class LoadSheddingMiddleware {
  constructor() {
    this.highWaterMark = 1000;  // Start shedding
    this.lowWaterMark = 500;    // Stop shedding
    this.shedding = false;
    this.activeRequests = 0;
  }

  async middleware(req, res, next) {
    this.activeRequests++;

    // Determine shedding state
    if (this.activeRequests > this.highWaterMark) {
      this.shedding = true;
    } else if (this.activeRequests < this.lowWaterMark) {
      this.shedding = false;
    }

    // When shedding, drop non-critical requests
    if (this.shedding && !this.isCritical(req)) {
      this.activeRequests--;
      console.log(`Load shedding: Dropped ${req.method} ${req.path}`);
      return res.status(503).json({
        error: 'Service Temporarily Unavailable',
        message: 'System under heavy load, non-critical requests temporarily disabled',
        retryAfter: 10
      });
    }

    try {
      await next();
    } finally {
      this.activeRequests--;
    }
  }

  isCritical(req) {
    // Always allow critical endpoints
    const criticalPaths = [
      '/api/payments',
      '/api/auth',
      '/api/health',
      '/api/emergency'
    ];
    return criticalPaths.some(path => req.path.startsWith(path));
  }
}

// Usage
const loadShedder = new LoadSheddingMiddleware();
app.use((req, res, next) => loadShedder.middleware(req, res, next));
```

### Resilience Patterns Quick Reference

| Pattern | What It Does | When It Kicks In |
|---------|--------------|------------------|
| **Circuit Breaker** | Stop calling failing service | After N failures |
| **Retry** | Try again on failure | Temporary failures |
| **Timeout** | Don't wait forever | Slow responses |
| **Bulkhead** | Isolate resources | Always (preventive) |
| **Fallback** | Return backup data | When primary fails |
| **Backpressure** | Slow down responses | At 80%+ capacity |
| **Load Shedding** | Drop non-critical | At 100%+ capacity |

### Key Insight (Memorize This!)
> "A crashed system requires manual intervention. A slow system recovers automatically when load decreases. Always prefer graceful degradation over hard failures."

---

## Deployment Strategies

### 1. Blue-Green Deployment

```
┌─────────────────────────────────────────────────────────────┐
│                   BLUE-GREEN DEPLOYMENT                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   Before:                                                    │
│   ┌──────────┐     ┌───────────────┐                       │
│   │  Users   │────▶│ Load Balancer │────▶ Blue (v1) ✓      │
│   └──────────┘     └───────────────┘      Green (v2) idle  │
│                                                              │
│   After switch:                                             │
│   ┌──────────┐     ┌───────────────┐                       │
│   │  Users   │────▶│ Load Balancer │────▶ Green (v2) ✓     │
│   └──────────┘     └───────────────┘      Blue (v1) standby│
│                                                              │
│   Rollback: Just switch back to Blue                        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 2. Canary Deployment

```
┌─────────────────────────────────────────────────────────────┐
│                    CANARY DEPLOYMENT                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   Phase 1: 5% traffic to canary                             │
│   ┌──────────┐     ┌───────────────┐     ┌────────────┐    │
│   │  Users   │────▶│ Load Balancer │──95%─▶│  v1 (old) │    │
│   └──────────┘     └───────────────┘       └────────────┘    │
│                            │                                 │
│                           5%──────────────▶┌────────────┐    │
│                                            │v2 (canary) │    │
│                                            └────────────┘    │
│                                                              │
│   Phase 2: Monitor metrics, gradually increase              │
│   Phase 3: 100% to v2, retire v1                            │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 3. Rolling Deployment

```javascript
// Kubernetes rolling update
/*
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1  # Max pods that can be unavailable
      maxSurge: 1        # Max pods above desired count
  template:
    spec:
      containers:
      - name: order-service
        image: order-service:v2
*/
```

### Container Orchestration (Kubernetes)

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
  labels:
    app: order-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
    spec:
      containers:
      - name: order-service
        image: myregistry/order-service:1.0.0
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: order-secrets
              key: database-url
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health/live
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5

---
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: order-service
spec:
  selector:
    app: order-service
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP

---
# hpa.yaml (Horizontal Pod Autoscaler)
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: order-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

---

## Monitoring & Observability

### The Three Pillars

```
┌─────────────────────────────────────────────────────────────┐
│                  THREE PILLARS OF OBSERVABILITY              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│   │    LOGS      │  │   METRICS    │  │   TRACES     │     │
│   │              │  │              │  │              │     │
│   │ What happened│  │ What's the   │  │ How requests │     │
│   │ (events)     │  │ state (num)  │  │ flow (path)  │     │
│   │              │  │              │  │              │     │
│   │ • Errors     │  │ • CPU usage  │  │ • Latency    │     │
│   │ • Requests   │  │ • Memory     │  │ • Service    │     │
│   │ • Actions    │  │ • Req/sec    │  │   calls      │     │
│   │              │  │ • Error rate │  │ • Bottleneck │     │
│   └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                              │
│   Tools:                                                    │
│   • ELK Stack     • Prometheus     • Jaeger                │
│   • Loki          • Grafana        • Zipkin                │
│   • Datadog       • Datadog        • OpenTelemetry         │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Structured Logging

```javascript
const logger = require('pino')({
  level: 'info',
  formatters: {
    level: (label) => ({ level: label })
  }
});

// Middleware to add request context
app.use((req, res, next) => {
  req.log = logger.child({
    requestId: req.headers['x-request-id'] || uuid(),
    userId: req.user?.id,
    service: 'order-service'
  });
  next();
});

// Logging in handlers
app.post('/orders', async (req, res) => {
  req.log.info({ body: req.body }, 'Creating order');
  
  try {
    const order = await orderService.create(req.body);
    req.log.info({ orderId: order.id }, 'Order created successfully');
    res.json(order);
  } catch (error) {
    req.log.error({ error: error.message, stack: error.stack }, 'Failed to create order');
    res.status(500).json({ error: 'Internal error' });
  }
});
```

### Distributed Tracing

```javascript
const { NodeTracerProvider } = require('@opentelemetry/sdk-trace-node');
const { SimpleSpanProcessor } = require('@opentelemetry/sdk-trace-base');
const { JaegerExporter } = require('@opentelemetry/exporter-jaeger');
const { registerInstrumentations } = require('@opentelemetry/instrumentation');
const { HttpInstrumentation } = require('@opentelemetry/instrumentation-http');
const { ExpressInstrumentation } = require('@opentelemetry/instrumentation-express');

// Setup tracing
const provider = new NodeTracerProvider();
provider.addSpanProcessor(
  new SimpleSpanProcessor(
    new JaegerExporter({ endpoint: 'http://jaeger:14268/api/traces' })
  )
);
provider.register();

registerInstrumentations({
  instrumentations: [
    new HttpInstrumentation(),
    new ExpressInstrumentation()
  ]
});

// Manual spans for business operations
const tracer = provider.getTracer('order-service');

async function createOrder(data) {
  const span = tracer.startSpan('createOrder');
  
  try {
    span.setAttribute('customerId', data.customerId);
    span.setAttribute('itemCount', data.items.length);
    
    // Child span for database operation
    const dbSpan = tracer.startSpan('saveToDatabase', { parent: span });
    const order = await db.orders.create(data);
    dbSpan.end();
    
    // Child span for event publishing
    const eventSpan = tracer.startSpan('publishEvent', { parent: span });
    await eventBus.publish('OrderCreated', order);
    eventSpan.end();
    
    span.setStatus({ code: SpanStatusCode.OK });
    return order;
  } catch (error) {
    span.setStatus({ code: SpanStatusCode.ERROR, message: error.message });
    throw error;
  } finally {
    span.end();
  }
}
```

### Health Checks

```javascript
app.get('/health/live', (req, res) => {
  // Basic liveness - is the process running?
  res.status(200).json({ status: 'alive' });
});

app.get('/health/ready', async (req, res) => {
  // Readiness - can the service handle requests?
  const checks = {
    database: await checkDatabase(),
    redis: await checkRedis(),
    dependencies: await checkDependencies()
  };
  
  const allHealthy = Object.values(checks).every(c => c.healthy);
  
  res.status(allHealthy ? 200 : 503).json({
    status: allHealthy ? 'ready' : 'not ready',
    checks
  });
});

async function checkDatabase() {
  try {
    await db.query('SELECT 1');
    return { healthy: true };
  } catch (error) {
    return { healthy: false, error: error.message };
  }
}
```

---

## Security

### Service-to-Service Authentication

```
┌─────────────────────────────────────────────────────────────┐
│              SERVICE-TO-SERVICE AUTH                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   Option 1: Mutual TLS (mTLS)                               │
│   ┌──────────┐         ┌──────────┐                        │
│   │ Service A│◄──TLS──►│ Service B│                        │
│   │ (cert)   │         │ (cert)   │                        │
│   └──────────┘         └──────────┘                        │
│   Both services verify each other's certificates            │
│                                                              │
│   Option 2: API Keys / Tokens                               │
│   ┌──────────┐  API Key  ┌──────────┐                      │
│   │ Service A│──────────►│ Service B│                       │
│   └──────────┘  Header   └──────────┘                       │
│                                                              │
│   Option 3: JWT with Service Identity                       │
│   ┌──────────┐   JWT     ┌──────────┐                      │
│   │ Service A│──────────►│ Service B│                       │
│   │ (signed) │  claims   │ (verify) │                       │
│   └──────────┘           └──────────┘                       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

```javascript
// JWT-based service authentication
class ServiceAuthClient {
  constructor(privateKey, serviceId) {
    this.privateKey = privateKey;
    this.serviceId = serviceId;
  }
  
  generateToken() {
    return jwt.sign(
      {
        iss: this.serviceId,
        aud: 'microservices',
        iat: Date.now(),
        exp: Date.now() + 60000 // 1 minute
      },
      this.privateKey,
      { algorithm: 'RS256' }
    );
  }
  
  async call(url, options = {}) {
    const token = this.generateToken();
    
    return fetch(url, {
      ...options,
      headers: {
        ...options.headers,
        'Authorization': `Bearer ${token}`,
        'X-Service-Id': this.serviceId
      }
    });
  }
}

// Middleware to verify service tokens
function verifyServiceToken(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];
  
  try {
    const decoded = jwt.verify(token, publicKey, { algorithms: ['RS256'] });
    req.serviceId = decoded.iss;
    next();
  } catch {
    res.status(401).json({ error: 'Invalid service token' });
  }
}
```

### API Gateway Security

```javascript
// Security middleware stack
const securityMiddleware = [
  // 1. Rate limiting
  rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 100,
    keyGenerator: (req) => req.user?.id || req.ip
  }),
  
  // 2. Input validation
  (req, res, next) => {
    const { error } = validateRequest(req);
    if (error) return res.status(400).json({ error: error.message });
    next();
  },
  
  // 3. Authentication
  passport.authenticate('jwt', { session: false }),
  
  // 4. Authorization
  (req, res, next) => {
    if (!hasPermission(req.user, req.path, req.method)) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    next();
  },
  
  // 5. Security headers
  helmet(),
  
  // 6. CORS
  cors({
    origin: process.env.ALLOWED_ORIGINS.split(','),
    credentials: true
  })
];
```

### Secrets Management

```javascript
// Using HashiCorp Vault
const vault = require('node-vault')({
  endpoint: process.env.VAULT_ADDR,
  token: process.env.VAULT_TOKEN
});

class SecretsManager {
  constructor() {
    this.cache = new Map();
    this.ttl = 3600000; // 1 hour
  }
  
  async getSecret(path) {
    const cached = this.cache.get(path);
    if (cached && cached.expiresAt > Date.now()) {
      return cached.value;
    }
    
    const result = await vault.read(path);
    this.cache.set(path, {
      value: result.data,
      expiresAt: Date.now() + this.ttl
    });
    
    return result.data;
  }
  
  async getDatabaseCredentials() {
    return this.getSecret('secret/data/order-service/database');
  }
}

// Environment-based (simpler)
const config = {
  database: {
    host: process.env.DB_HOST,
    password: process.env.DB_PASSWORD // From K8s secret
  }
};
```

---

## Testing Strategies

### Testing Pyramid for Microservices

```
┌─────────────────────────────────────────────────────────────┐
│                    TESTING PYRAMID                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│                         /\                                  │
│                        /  \                                 │
│                       / E2E\    Few, slow, expensive        │
│                      /──────\                               │
│                     /Contract\   Service boundaries         │
│                    /──────────\                             │
│                   / Integration\  Database, external APIs   │
│                  /──────────────\                           │
│                 /     Unit       \  Many, fast, cheap       │
│                /──────────────────\                         │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Unit Tests

```javascript
// order.service.test.js
describe('OrderService', () => {
  let orderService;
  let mockRepository;
  let mockEventBus;
  
  beforeEach(() => {
    mockRepository = {
      save: jest.fn(),
      findById: jest.fn()
    };
    mockEventBus = {
      publish: jest.fn()
    };
    orderService = new OrderService(mockRepository, mockEventBus);
  });
  
  describe('createOrder', () => {
    it('should create order and publish event', async () => {
      const orderData = {
        customerId: 'cust-123',
        items: [{ productId: 'prod-1', quantity: 2 }]
      };
      
      mockRepository.save.mockResolvedValue({ id: 'order-1', ...orderData });
      
      const result = await orderService.createOrder(orderData);
      
      expect(mockRepository.save).toHaveBeenCalledWith(
        expect.objectContaining(orderData)
      );
      expect(mockEventBus.publish).toHaveBeenCalledWith(
        'OrderCreated',
        expect.objectContaining({ id: 'order-1' })
      );
      expect(result.id).toBe('order-1');
    });
    
    it('should throw error for empty items', async () => {
      await expect(
        orderService.createOrder({ customerId: 'cust-123', items: [] })
      ).rejects.toThrow('Order must have at least one item');
    });
  });
});
```

### Integration Tests

```javascript
// order.integration.test.js
describe('Order API Integration', () => {
  let app;
  let db;
  
  beforeAll(async () => {
    // Start test database (use testcontainers)
    db = await startTestDatabase();
    app = createApp({ database: db });
  });
  
  afterAll(async () => {
    await db.close();
  });
  
  beforeEach(async () => {
    await db.clear();
  });
  
  it('should create and retrieve order', async () => {
    // Create order
    const createResponse = await request(app)
      .post('/orders')
      .send({
        customerId: 'cust-123',
        items: [{ productId: 'prod-1', quantity: 2, price: 10.00 }]
      })
      .expect(201);
    
    const orderId = createResponse.body.id;
    
    // Retrieve order
    const getResponse = await request(app)
      .get(`/orders/${orderId}`)
      .expect(200);
    
    expect(getResponse.body).toMatchObject({
      id: orderId,
      customerId: 'cust-123',
      status: 'CREATED'
    });
  });
});
```

### Contract Tests (Pact)

```javascript
// Consumer test (Order Service testing Payment Service contract)
const { Pact } = require('@pact-foundation/pact');

describe('Payment Service Contract', () => {
  const provider = new Pact({
    consumer: 'OrderService',
    provider: 'PaymentService',
    port: 1234
  });
  
  beforeAll(() => provider.setup());
  afterAll(() => provider.finalize());
  afterEach(() => provider.verify());
  
  it('should process payment', async () => {
    // Define expected interaction
    await provider.addInteraction({
      state: 'payment method exists',
      uponReceiving: 'a request to process payment',
      withRequest: {
        method: 'POST',
        path: '/payments',
        headers: { 'Content-Type': 'application/json' },
        body: {
          orderId: '123',
          amount: 99.99,
          currency: 'USD'
        }
      },
      willRespondWith: {
        status: 201,
        body: {
          transactionId: like('txn-123'),
          status: 'COMPLETED'
        }
      }
    });
    
    // Test the client
    const client = new PaymentClient(`http://localhost:1234`);
    const result = await client.processPayment({
      orderId: '123',
      amount: 99.99,
      currency: 'USD'
    });
    
    expect(result.status).toBe('COMPLETED');
  });
});
```

### E2E Tests

```javascript
// e2e/order-flow.test.js
describe('Order Flow E2E', () => {
  it('should complete full order flow', async () => {
    // 1. Create user
    const user = await api.post('/users', { email: 'test@example.com' });
    
    // 2. Add items to cart
    await api.post(`/cart/${user.id}/items`, { productId: 'prod-1', quantity: 2 });
    
    // 3. Create order
    const order = await api.post('/orders', { 
      customerId: user.id,
      paymentMethod: 'card'
    });
    
    expect(order.status).toBe('PENDING');
    
    // 4. Wait for payment processing
    await waitFor(async () => {
      const updated = await api.get(`/orders/${order.id}`);
      return updated.status === 'PAID';
    }, { timeout: 10000 });
    
    // 5. Verify inventory updated
    const product = await api.get('/products/prod-1');
    expect(product.stock).toBe(98); // Reduced by 2
    
    // 6. Verify notification sent
    const notifications = await api.get(`/notifications?userId=${user.id}`);
    expect(notifications).toContainEqual(
      expect.objectContaining({ type: 'ORDER_CONFIRMED' })
    );
  });
});
```

---

## Migration from Monolith

### Strangler Fig Pattern

```
┌─────────────────────────────────────────────────────────────┐
│                   STRANGLER FIG PATTERN                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   Phase 1: Add Facade                                       │
│   ┌──────────┐     ┌──────────┐     ┌──────────────────┐   │
│   │  Client  │────▶│  Facade  │────▶│    Monolith      │   │
│   └──────────┘     └──────────┘     │  ┌────┬────┬────┐│   │
│                                      │  │User│Ordr│Pay ││   │
│                                      │  └────┴────┴────┘│   │
│                                      └──────────────────┘   │
│                                                              │
│   Phase 2: Extract First Service                            │
│   ┌──────────┐     ┌──────────┐     ┌──────────────────┐   │
│   │  Client  │────▶│  Facade  │──┬─▶│    Monolith      │   │
│   └──────────┘     └──────────┘  │  │  ┌────┬────┐    │   │
│                                   │  │  │Ordr│Pay │    │   │
│                                   │  │  └────┴────┘    │   │
│                                   │  └──────────────────┘   │
│                                   │                         │
│                                   └─▶┌──────────┐          │
│                                      │   User   │          │
│                                      │ Service  │          │
│                                      └──────────┘          │
│                                                              │
│   Phase N: Monolith Eliminated                              │
│   ┌──────────┐     ┌──────────┐                            │
│   │  Client  │────▶│  Facade  │──┬─▶ User Service         │
│   └──────────┘     │ (Gateway)│  ├─▶ Order Service        │
│                    └──────────┘  └─▶ Payment Service       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Migration Steps

```javascript
// Step 1: Identify bounded contexts in monolith
/*
Monolith modules:
├── users/         → User Service
├── orders/        → Order Service
├── payments/      → Payment Service
├── inventory/     → Inventory Service
└── notifications/ → Notification Service
*/

// Step 2: Create anti-corruption layer
class UserAntiCorruptionLayer {
  constructor(legacyUserModule, newUserService) {
    this.legacy = legacyUserModule;
    this.new = newUserService;
    this.useNewService = process.env.USE_NEW_USER_SERVICE === 'true';
  }
  
  async getUser(id) {
    if (this.useNewService) {
      return this.new.getUser(id);
    }
    return this.legacy.getUser(id);
  }
  
  // Gradually migrate methods
  async createUser(data) {
    // Write to both during migration
    const legacyUser = await this.legacy.createUser(data);
    
    try {
      await this.new.createUser({ ...data, legacyId: legacyUser.id });
    } catch (error) {
      console.error('Failed to sync to new service', error);
    }
    
    return legacyUser;
  }
}

// Step 3: Data migration
async function migrateUsers() {
  const batchSize = 100;
  let offset = 0;
  
  while (true) {
    const users = await legacyDb.query(
      'SELECT * FROM users LIMIT ? OFFSET ?',
      [batchSize, offset]
    );
    
    if (users.length === 0) break;
    
    for (const user of users) {
      await newUserService.create({
        id: user.id,
        email: user.email,
        profile: transformProfile(user),
        migratedAt: new Date()
      });
    }
    
    offset += batchSize;
    console.log(`Migrated ${offset} users`);
  }
}

// Step 4: Verify and switch
async function verifyMigration() {
  const sampleIds = await getSampleUserIds(1000);
  
  for (const id of sampleIds) {
    const legacy = await legacyDb.getUser(id);
    const new_ = await newUserService.getUser(id);
    
    if (!deepEqual(transformForComparison(legacy), new_)) {
      console.error(`Mismatch for user ${id}`);
      return false;
    }
  }
  
  return true;
}
```

---

## Common Pitfalls

### 1. Distributed Monolith

```
❌ BAD: Services tightly coupled
┌──────────┐     ┌──────────┐     ┌──────────┐
│ Service A│────▶│ Service B│────▶│ Service C│
│          │◀────│          │◀────│          │
└──────────┘     └──────────┘     └──────────┘
All services must deploy together = Distributed Monolith

✅ GOOD: Loosely coupled services
┌──────────┐     ┌──────────┐     ┌──────────┐
│ Service A│     │ Service B│     │ Service C│
└────┬─────┘     └────┬─────┘     └────┬─────┘
     │                │                │
     └────────────────┼────────────────┘
                      ▼
              Event Bus (Async)
```

### 2. Wrong Service Boundaries

```
❌ BAD: Technical boundaries
├── API Service
├── Business Logic Service
└── Database Service

❌ BAD: Too fine-grained (nano-services)
├── User Create Service
├── User Update Service
├── User Delete Service
└── User Query Service

✅ GOOD: Business domain boundaries
├── User Management Service
├── Order Management Service
├── Payment Processing Service
└── Inventory Service
```

### 3. Shared Database

```
❌ BAD: Multiple services sharing one database
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Service A│  │ Service B│  │ Service C│
└────┬─────┘  └────┬─────┘  └────┬─────┘
     │             │             │
     └─────────────┼─────────────┘
                   ▼
            ┌──────────┐
            │ Shared DB│
            └──────────┘
• Changes affect all services
• Can't scale independently
• Tight coupling

✅ GOOD: Database per service
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Service A│  │ Service B│  │ Service C│
└────┬─────┘  └────┬─────┘  └────┬─────┘
     │             │             │
     ▼             ▼             ▼
┌────────┐   ┌────────┐   ┌────────┐
│  DB A  │   │  DB B  │   │  DB C  │
└────────┘   └────────┘   └────────┘
```

### 4. No Observability

```
❌ Without observability:
"Something is slow but we don't know what"
"Errors are happening somewhere"
"Which service is the bottleneck?"

✅ With observability:
• Distributed traces show request flow
• Metrics identify bottlenecks
• Logs provide context
• Alerts notify issues early
```

### 5. Synchronous Everything

```
❌ BAD: All sync calls
Order Service
  → Payment Service (wait)
    → Fraud Service (wait)
      → Bank Service (wait)
  → Inventory Service (wait)
  → Notification Service (wait)
Total latency: Sum of all services

✅ GOOD: Async where possible
Order Service
  → Publish "OrderCreated" event
  → Return immediately

Async handlers:
  • Payment Service processes
  • Inventory Service reserves
  • Notification Service notifies
```

---

## Interview Questions & Answers

### Basic Level

#### Q1: What are microservices?

**Answer:**
Microservices is an architectural style where an application is built as a collection of small, independent services that:

- Run in their own process
- Communicate over network (HTTP, gRPC, messaging)
- Are independently deployable
- Are organized around business capabilities
- Own their own data

**Key characteristics:**
- Single responsibility
- Loose coupling
- High cohesion
- Independent deployment
- Decentralized data management

---

#### Q2: What is the difference between monolith and microservices?

**Answer:**

| Aspect | Monolith | Microservices |
|--------|----------|---------------|
| **Deployment** | Single unit | Independent services |
| **Scaling** | Scale everything | Scale specific services |
| **Technology** | Single stack | Multiple stacks possible |
| **Database** | Single shared DB | Database per service |
| **Team** | Large team, shared code | Small teams, service ownership |
| **Failure** | Can bring down whole app | Isolated to service |
| **Complexity** | Simpler initially | Complex from start |
| **Communication** | In-process | Network calls |

**When to use Monolith:**
- Small team
- Simple domain
- Early-stage startup
- Unclear requirements

**When to use Microservices:**
- Large team (20+)
- Complex domain
- Different scaling needs
- High deployment frequency

---

#### Q3: How do microservices communicate?

**Answer:**

**1. Synchronous (Request-Response):**
- REST APIs
- gRPC
- GraphQL

```javascript
// REST
const user = await fetch('http://user-service/users/123');

// gRPC
const user = await userClient.getUser({ id: '123' });
```

**2. Asynchronous (Event-Driven):**
- Message queues (RabbitMQ, SQS)
- Event streaming (Kafka)
- Pub/Sub

```javascript
// Publish event
await eventBus.publish('OrderCreated', { orderId: '123' });

// Subscribe
eventBus.subscribe('OrderCreated', handleOrderCreated);
```

**When to use which:**
- **Sync:** Need immediate response, simple request-response
- **Async:** Fire-and-forget, long processing, event-driven flows

---

### Intermediate Level

#### Q4: Explain the Saga pattern.

**Answer:**
Saga is a pattern for managing distributed transactions across multiple services without using traditional ACID transactions.

**Two types:**

**1. Choreography:** Each service publishes events, others react
```
Order Created → Payment Service listens → Payment Done → Inventory listens → Stock Reserved
```

**2. Orchestration:** Central coordinator manages the flow
```
Saga Orchestrator:
1. Call Order Service
2. Call Payment Service  
3. Call Inventory Service
4. On failure: Execute compensating transactions
```

**Compensating transactions:** Undo operations if saga fails
- Payment failed → Cancel order, release inventory
- Unlike rollback, they're new transactions

```javascript
// Orchestration example
class OrderSaga {
  async execute(order) {
    try {
      await orderService.create(order);
      await paymentService.charge(order);
      await inventoryService.reserve(order);
    } catch (error) {
      // Compensate in reverse order
      await inventoryService.release(order);
      await paymentService.refund(order);
      await orderService.cancel(order);
    }
  }
}
```

---

#### Q5: What is an API Gateway?

**Answer:**
API Gateway is a single entry point for all client requests that handles cross-cutting concerns.

**Responsibilities:**
- Request routing
- Authentication/Authorization
- Rate limiting
- Load balancing
- Request/Response transformation
- Caching
- Logging/Monitoring
- API composition

**Benefits:**
- Clients have single endpoint
- Offloads common functionality from services
- Can provide different APIs for different clients (BFF pattern)

**Popular gateways:** Kong, AWS API Gateway, Nginx, Traefik

```
Client → API Gateway → Service A
                    → Service B
                    → Service C
```

---

#### Q6: Explain the Circuit Breaker pattern.

**Answer:**
Circuit Breaker prevents cascading failures by failing fast when a service is unhealthy.

**States:**
1. **CLOSED:** Normal operation, requests pass through
2. **OPEN:** Service unhealthy, requests fail immediately
3. **HALF-OPEN:** Testing if service recovered

**Flow:**
```
CLOSED --[failures > threshold]--> OPEN --[timeout]--> HALF-OPEN
                                     ↑                     |
                                     |                     |
                                     +--[failure]----------+
                                     |
CLOSED <--[success]------------------+
```

```javascript
const breaker = new CircuitBreaker({
  failureThreshold: 5,    // Open after 5 failures
  resetTimeout: 30000     // Try again after 30s
});

try {
  await breaker.call(() => paymentService.process());
} catch (error) {
  if (error.message === 'Circuit breaker is OPEN') {
    return fallbackResponse();
  }
}
```

---

#### Q7: What is service discovery?

**Answer:**
Service discovery allows services to find each other dynamically without hardcoded addresses.

**Why needed:**
- Services scale up/down dynamically
- IP addresses change
- Multiple instances behind load balancer

**Types:**

**1. Client-side discovery:**
- Client queries registry
- Client does load balancing
- Tools: Eureka, Consul

**2. Server-side discovery:**
- Load balancer queries registry
- Client calls load balancer
- Tools: Kubernetes, AWS ALB

```javascript
// Client-side
const instances = await registry.getInstances('payment-service');
const instance = loadBalancer.choose(instances);
await fetch(`http://${instance.host}:${instance.port}/pay`);

// Server-side (Kubernetes)
await fetch('http://payment-service/pay'); // DNS resolves to service
```

---

### Advanced Level

#### Q8: How do you handle distributed transactions?

**Answer:**

**Problem:** ACID transactions don't work across services.

**Solutions:**

**1. Saga Pattern:**
- Choreography or Orchestration
- Compensating transactions for rollback

**2. Two-Phase Commit (2PC):**
- Coordinator asks all to prepare
- If all ready, coordinator commits all
- Rarely used in microservices (blocking, slow)

**3. Event Sourcing:**
- Store events, not state
- Replay events to rebuild state
- Natural audit trail

**4. Eventual Consistency:**
- Accept that data won't be immediately consistent
- Design for it

**Best practices:**
- Prefer async communication
- Design idempotent operations
- Use correlation IDs for tracking
- Implement retry with backoff
- Have dead letter queues for failed messages

---

#### Q9: How do you ensure data consistency across services?

**Answer:**

**Challenge:** Each service owns its data, can't use joins across databases.

**Patterns:**

**1. Event-Driven Synchronization:**
```javascript
// Order Service publishes event
await publish('OrderCreated', { orderId, customerId, items });

// Inventory Service subscribes and updates its local data
subscribe('OrderCreated', async (event) => {
  await inventoryDB.reserveStock(event.items);
});
```

**2. API Composition:**
```javascript
// Gateway aggregates data from multiple services
async function getOrderDetails(orderId) {
  const [order, customer, payments] = await Promise.all([
    orderService.getOrder(orderId),
    userService.getUser(order.customerId),
    paymentService.getPayments(orderId)
  ]);
  
  return { ...order, customer, payments };
}
```

**3. CQRS:**
- Write model: normalized, in owning service
- Read model: denormalized, optimized for queries

**4. Eventual Consistency:**
- Accept temporary inconsistency
- Design UI to handle it
- Use background reconciliation jobs

---

#### Q10: How would you migrate a monolith to microservices?

**Answer:**

**Strangler Fig Pattern:**

**Step 1:** Add facade (API Gateway) in front of monolith

**Step 2:** Identify bounded contexts
- Analyze domain
- Find seams in code
- Map dependencies

**Step 3:** Extract services incrementally
- Start with least coupled module
- Create anti-corruption layer
- Dual-write during migration
- Verify data consistency

**Step 4:** Migrate data
- Copy data to new service DB
- Sync changes during transition
- Switch reads then writes

**Step 5:** Remove from monolith
- Redirect all traffic to new service
- Delete old code
- Repeat for next service

```javascript
// Anti-corruption layer during migration
class UserService {
  async getUser(id) {
    if (featureFlags.useNewUserService) {
      return newUserService.getUser(id);
    }
    return legacyMonolith.getUser(id);
  }
}
```

**Tips:**
- Don't rewrite, extract
- Start small, gain experience
- Invest in CI/CD and observability first
- Keep the monolith working during migration

---

#### Q11: How do you handle service failures?

**Answer:**

**Patterns:**

**1. Circuit Breaker:** Fail fast when service is down

**2. Retry with Backoff:** Retry transient failures
```javascript
const delay = Math.min(baseDelay * Math.pow(2, attempt), maxDelay);
```

**3. Bulkhead:** Isolate resources per service
```javascript
// Separate connection pools
const paymentPool = createPool({ max: 10 });
const inventoryPool = createPool({ max: 20 });
```

**4. Timeout:** Don't wait forever
```javascript
const result = await Promise.race([
  serviceCall(),
  timeout(5000)
]);
```

**5. Fallback:** Provide degraded functionality
```javascript
try {
  return await recommendationService.get();
} catch {
  return defaultRecommendations; // Cache or static data
}
```

**6. Health Checks:** Detect failures early
- Liveness: Is process running?
- Readiness: Can handle requests?

**7. Graceful Degradation:** Partial functionality better than total failure

---

#### Q12: How do you test microservices?

**Answer:**

**Testing Pyramid:**

**1. Unit Tests (70%):**
- Test business logic
- Mock dependencies
- Fast, many tests

**2. Integration Tests (20%):**
- Test with real database
- Test API endpoints
- Use test containers

**3. Contract Tests:**
- Verify service interfaces
- Consumer-driven contracts (Pact)
- Prevent breaking changes

**4. E2E Tests (10%):**
- Test full flow
- Slower, fewer tests
- Use staging environment

**Additional:**
- **Component tests:** Single service in isolation
- **Chaos testing:** Inject failures
- **Performance tests:** Load testing

```javascript
// Contract test example
await provider.addInteraction({
  state: 'user exists',
  uponReceiving: 'get user request',
  withRequest: { method: 'GET', path: '/users/1' },
  willRespondWith: {
    status: 200,
    body: { id: '1', name: like('John') }
  }
});
```

---

### Scenario-Based Questions

#### Q13: Design an e-commerce system using microservices.

**Answer:**

**Services:**

```
┌─────────────────────────────────────────────────────────────┐
│                     E-COMMERCE SYSTEM                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│   │   User   │  │  Product │  │   Cart   │  │  Order   │  │
│   │ Service  │  │ Service  │  │ Service  │  │ Service  │  │
│   └──────────┘  └──────────┘  └──────────┘  └──────────┘  │
│                                                              │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│   │ Payment  │  │Inventory │  │ Shipping │  │  Search  │  │
│   │ Service  │  │ Service  │  │ Service  │  │ Service  │  │
│   └──────────┘  └──────────┘  └──────────┘  └──────────┘  │
│                                                              │
│   ┌──────────┐  ┌──────────┐                               │
│   │  Review  │  │  Notif   │                               │
│   │ Service  │  │ Service  │                               │
│   └──────────┘  └──────────┘                               │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Data ownership:**
- User Service: Users, addresses, preferences
- Product Service: Products, categories
- Inventory Service: Stock levels, warehouses
- Order Service: Orders, order items
- Payment Service: Transactions, refunds

**Order flow (Saga):**
```
1. Order Service: Create order (PENDING)
2. Inventory Service: Reserve stock
3. Payment Service: Process payment
4. Order Service: Confirm order (CONFIRMED)
5. Notification Service: Send confirmation
6. Shipping Service: Schedule delivery
```

**Communication:**
- Sync: Cart → Product (get prices)
- Async: Order created → Inventory, Payment, Notification

---

#### Q14: A service is failing. How do you debug it?

**Answer:**

**Step 1: Check observability tools**
- Metrics: Error rate, latency spikes
- Traces: Find slow/failing requests
- Logs: Error messages, stack traces

**Step 2: Identify scope**
- All requests or specific ones?
- All instances or specific?
- Started when? (deployment, config change)

**Step 3: Check dependencies**
- Database connectivity
- External service health
- Message queue status

**Step 4: Analyze patterns**
- Correlation with traffic
- Memory/CPU usage
- Connection pool exhaustion

**Step 5: Review recent changes**
- Code deployments
- Config changes
- Infrastructure changes

**Tools:**
- Grafana/Prometheus for metrics
- Jaeger/Zipkin for traces
- ELK/Loki for logs
- Sentry for errors

```javascript
// Add correlation ID for tracing
app.use((req, res, next) => {
  req.correlationId = req.headers['x-correlation-id'] || uuid();
  res.setHeader('x-correlation-id', req.correlationId);
  next();
});
```

---

#### Q15: How would you handle a service that needs data from 5 other services?

**Answer:**

**Options:**

**1. API Composition (Gateway):**
```javascript
// Gateway aggregates
async function getDashboard(userId) {
  const [user, orders, notifications, recommendations, activity] = 
    await Promise.all([
      userService.getUser(userId),
      orderService.getRecentOrders(userId),
      notificationService.getUnread(userId),
      recommendationService.getFor(userId),
      activityService.getRecent(userId)
    ]);
  
  return { user, orders, notifications, recommendations, activity };
}
```

**2. CQRS with Read Model:**
- Create denormalized read model
- Update via events from all services
- Single query for dashboard

**3. GraphQL Federation:**
- Each service defines its schema
- Gateway federates schemas
- Client queries what it needs

**4. Backend for Frontend (BFF):**
- Dedicated service for this client
- Handles aggregation
- Optimized for client needs

**Best practices:**
- Use parallel calls (Promise.all)
- Set timeouts
- Have fallbacks for each service
- Cache where appropriate
- Consider if all data is really needed

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────┐
│              MICROSERVICES CHEAT SHEET                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  PRINCIPLES:                                                │
│  • Single responsibility per service                        │
│  • Database per service                                     │
│  • Smart endpoints, dumb pipes                              │
│  • Design for failure                                       │
│  • Decentralized governance                                 │
│                                                              │
│  COMMUNICATION:                                              │
│  • REST/gRPC → Sync, request-response                       │
│  • Message Queue → Async, fire-and-forget                   │
│  • Events → Loose coupling, reactive                        │
│                                                              │
│  PATTERNS:                                                   │
│  • API Gateway → Single entry point                         │
│  • Circuit Breaker → Fail fast                              │
│  • Saga → Distributed transactions                          │
│  • CQRS → Separate read/write                               │
│  • Event Sourcing → Store events                            │
│                                                              │
│  RESILIENCE:                                                 │
│  • Retry with backoff                                       │
│  • Timeout                                                  │
│  • Bulkhead                                                 │
│  • Fallback                                                 │
│                                                              │
│  DATA:                                                       │
│  • Eventual consistency                                     │
│  • Event-driven sync                                        │
│  • API composition                                          │
│                                                              │
│  OBSERVABILITY:                                              │
│  • Distributed tracing (Jaeger)                             │
│  • Centralized logging (ELK)                                │
│  • Metrics (Prometheus)                                     │
│  • Health checks                                            │
│                                                              │
│  AVOID:                                                      │
│  • Shared databases                                         │
│  • Synchronous chains                                       │
│  • Distributed monolith                                     │
│  • Too fine-grained services                                │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Resources

### Books
- "Building Microservices" by Sam Newman
- "Microservices Patterns" by Chris Richardson
- "Domain-Driven Design" by Eric Evans

### Tools
| Category | Tools |
|----------|-------|
| **API Gateway** | Kong, AWS API Gateway, Traefik |
| **Service Mesh** | Istio, Linkerd, Consul Connect |
| **Message Queue** | RabbitMQ, Kafka, AWS SQS |
| **Orchestration** | Kubernetes, Docker Swarm |
| **Monitoring** | Prometheus, Grafana, Datadog |
| **Tracing** | Jaeger, Zipkin, AWS X-Ray |
| **Service Discovery** | Consul, Eureka, Kubernetes DNS |

### Further Reading
- [microservices.io](https://microservices.io) - Patterns and best practices
- [12factor.net](https://12factor.net) - 12-Factor App methodology
- Martin Fowler's microservices articles

---

*Last updated: January 2026*

