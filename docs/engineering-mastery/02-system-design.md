# Chapter 02: System Design Principles

> "Architecture is about the important stuff. Whatever that is." - Ralph Johnson

---

## 🎯 The Goal of System Design

Design systems that are:
- **Scalable** - Handle growth (users, data, traffic)
- **Reliable** - Work correctly even when things fail
- **Maintainable** - Easy to understand, modify, extend

---

## 📊 Scalability Fundamentals

### Vertical vs Horizontal Scaling

```
Vertical Scaling (Scale Up):
┌──────────────────────┐
│                      │
│    BIGGER SERVER     │
│    More CPU, RAM     │
│                      │
└──────────────────────┘
Pros: Simple, no code changes
Cons: Hardware limits, single point of failure, expensive

Horizontal Scaling (Scale Out):
┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
│Server 1│ │Server 2│ │Server 3│ │Server N│
└────────┘ └────────┘ └────────┘ └────────┘
Pros: Unlimited scaling, fault tolerant
Cons: Complex, need distributed systems knowledge
```

### Load Balancing Algorithms

```
1. Round Robin:
   Request 1 → Server A
   Request 2 → Server B
   Request 3 → Server C
   Request 4 → Server A (cycle)
   
2. Weighted Round Robin:
   Server A (weight 3): Gets 3x traffic
   Server B (weight 1): Gets 1x traffic
   
3. Least Connections:
   Always route to server with fewest active connections
   
4. IP Hash:
   hash(client_ip) % servers = target server
   Same client always goes to same server (sticky sessions)
   
5. Least Response Time:
   Route to server with fastest response + fewest connections
```

### Load Balancer Types

```
Layer 4 (Transport):
┌─────────────────────────────────────────────────┐
│ Routes based on: IP address, TCP/UDP port       │
│ Fast (no content inspection)                    │
│ Example: AWS NLB                                │
└─────────────────────────────────────────────────┘

Layer 7 (Application):
┌─────────────────────────────────────────────────┐
│ Routes based on: URL, headers, cookies, body    │
│ Slower but smarter                              │
│ Can do: SSL termination, caching, compression  │
│ Example: AWS ALB, Nginx                         │
└─────────────────────────────────────────────────┘
```

---

## ⚖️ CAP Theorem

**In a distributed system, you can only guarantee 2 of 3:**

```
         Consistency
            /\
           /  \
          /    \
         /  CP  \
        /________\
       /\        /\
      /  \  CA  /  \
     / AP \    /    \
    /______\  /______\
Availability ─────── Partition
                     Tolerance
```

### Understanding Each Property

**Consistency (C):**
```
Write X=1 to Node A
Read X from Node B → Must return 1

All nodes see the same data at the same time
```

**Availability (A):**
```
Every request receives a response (success or failure)
System is always operational
```

**Partition Tolerance (P):**
```
   Node A ──X── Node B    (Network partition)
   
System continues working despite network failures
```

### Real-World Choices

| System | Choice | Reasoning |
|--------|--------|-----------|
| **Banking** | CP | Can't have inconsistent balances |
| **Social Media** | AP | Okay to show slightly stale data |
| **E-commerce Inventory** | CP | Can't oversell |
| **User Sessions** | AP | Availability more important |
| **DNS** | AP | Eventually consistent is fine |

### PACELC Theorem (Extended CAP)

```
If Partition:
  Choose: Availability or Consistency (A/C)
Else (normal operation):
  Choose: Latency or Consistency (L/C)

Examples:
- DynamoDB: PA/EL (Available, Low latency)
- PostgreSQL: PC/EC (Consistent always)
- Cassandra: PA/EL (Tunable per query)
```

---

## 🏛️ Architectural Patterns

### 1. Monolith

```
┌─────────────────────────────────────────┐
│              MONOLITH                   │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │  Auth   │ │  Users  │ │ Orders  │   │
│  └─────────┘ └─────────┘ └─────────┘   │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │Payments │ │Inventory│ │ Reports │   │
│  └─────────┘ └─────────┘ └─────────┘   │
│                  │                      │
│            ┌─────┴─────┐               │
│            │  Database │               │
│            └───────────┘               │
└─────────────────────────────────────────┘

Pros: Simple, easy to deploy, easy to debug
Cons: Hard to scale, long deployments, tech lock-in
When: Startups, small teams, MVPs
```

### 2. Microservices

```
┌──────────┐  ┌──────────┐  ┌──────────┐
│   Auth   │  │  Users   │  │  Orders  │
│ Service  │  │ Service  │  │ Service  │
│    │     │  │    │     │  │    │     │
│ [Auth DB]│  │[Users DB]│  │[OrdersDB]│
└──────────┘  └──────────┘  └──────────┘
      │             │             │
      └─────────────┼─────────────┘
                    │
            ┌───────┴───────┐
            │  API Gateway  │
            └───────────────┘
            
Pros: Independent scaling, tech flexibility, team autonomy
Cons: Complex, network overhead, distributed transactions
When: Large teams, different scaling needs, polyglot tech
```

### 3. Service-Oriented Architecture (SOA)

```
┌─────────────────────────────────────────────────────┐
│                  Enterprise Service Bus             │
├─────────┬─────────┬─────────┬─────────┬─────────────┤
│Customer │ Order   │Inventory│ Billing │ Shipping    │
│Service  │ Service │ Service │ Service │ Service     │
└─────────┴─────────┴─────────┴─────────┴─────────────┘

Heavier than microservices, enterprise-focused
```

### 4. Event-Driven Architecture

```
┌─────────────┐    ┌─────────────────────────┐
│   Order     │───►│     Event Bus           │
│   Service   │    │  (Kafka/RabbitMQ)       │
└─────────────┘    └───────────┬─────────────┘
                               │
           ┌───────────────────┼───────────────────┐
           │                   │                   │
           ▼                   ▼                   ▼
    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
    │  Inventory  │    │   Email     │    │  Analytics  │
    │   Service   │    │   Service   │    │   Service   │
    └─────────────┘    └─────────────┘    └─────────────┘

Events: OrderCreated, OrderPaid, OrderShipped
Services react to events independently
```

### 5. CQRS (Command Query Responsibility Segregation)

```
               ┌─────────────────┐
               │    Commands     │
               │ (Create, Update)│
               └────────┬────────┘
                        │
                        ▼
               ┌─────────────────┐
               │  Write Model    │
               │  (Normalized)   │
               └────────┬────────┘
                        │
                   Sync/Events
                        │
                        ▼
               ┌─────────────────┐
               │   Read Model    │
               │ (Denormalized)  │
               └────────┬────────┘
                        │
                        ▼
               ┌─────────────────┐
               │    Queries      │
               │   (Read-only)   │
               └─────────────────┘

Separate models for reading and writing
Write: PostgreSQL (normalized)
Read: Elasticsearch (denormalized, fast)
```

---

## 🔄 Communication Patterns

### Synchronous vs Asynchronous

```
Synchronous (HTTP/gRPC):
Client ───request──► Service
Client ◄──response── Service
        └── Waits ──┘

Asynchronous (Message Queue):
Client ───message──► Queue ───message──► Service
Client continues immediately (doesn't wait)
```

### API Design Patterns

**REST:**
```http
GET    /users/123       # Get user
POST   /users           # Create user
PUT    /users/123       # Update user
DELETE /users/123       # Delete user

Stateless, cacheable, widely understood
```

**GraphQL:**
```graphql
query {
  user(id: "123") {
    name
    email
    orders {
      id
      total
    }
  }
}

Single endpoint, client specifies data shape
Good for: Mobile (reduce data), complex relationships
```

**gRPC:**
```protobuf
service UserService {
  rpc GetUser(UserRequest) returns (User);
  rpc CreateUser(CreateUserRequest) returns (User);
}

Binary protocol (Protocol Buffers)
Fast, strongly typed, streaming support
Good for: Internal service communication
```

### Service Discovery

```
How does Service A find Service B?

1. Hardcoded (bad):
   const serviceB = "http://192.168.1.50:3000"
   
2. DNS-based:
   const serviceB = "http://service-b.internal"
   
3. Service Registry (Consul, etcd):
   ┌─────────────┐
   │  Registry   │
   │  service-b: │
   │  - 10.0.0.1 │
   │  - 10.0.0.2 │
   └─────────────┘
   
4. Kubernetes Service:
   service-b.namespace.svc.cluster.local
```

---

## 🔄 Data Flow Patterns

### Request-Response

```
┌────────┐  request   ┌────────┐
│ Client │ ─────────► │ Server │
│        │ ◄───────── │        │
└────────┘  response  └────────┘
```

### Publish-Subscribe

```
              ┌────────────┐
        ┌────►│Subscriber 1│
        │     └────────────┘
┌───────┴───┐ ┌────────────┐
│  Pub/Sub  │─┤Subscriber 2│
│   Topic   │ └────────────┘
└───────┬───┘ ┌────────────┐
        └────►│Subscriber 3│
              └────────────┘
              
Publisher doesn't know subscribers
Subscribers don't know each other
```

### Event Sourcing

```
Instead of storing current state:
┌──────────────┐
│ balance: 100 │  (current state only)
└──────────────┘

Store all events:
┌────────────────────────────────┐
│ 1. AccountCreated(balance: 0)  │
│ 2. Deposited(amount: 150)      │
│ 3. Withdrawn(amount: 50)       │
│ → Current balance: 100         │
└────────────────────────────────┘

Benefits:
- Complete audit trail
- Replay to any point in time
- Debug production issues
- Event-driven reactions
```

---

## 📊 System Design Template

When designing a system, follow this structure:

### 1. Requirements Clarification
```
Functional:
- What features are needed?
- Who are the users?
- What's the expected behavior?

Non-functional:
- Scale: How many users? Requests/sec?
- Latency: What's acceptable response time?
- Availability: What's the uptime requirement?
- Consistency: Strong or eventual?
```

### 2. Back-of-Envelope Estimation
```
Example: Twitter-like system

Users: 500 million
DAU: 100 million (20%)
Tweets/day: 100 million
Reads/day: 10 billion (100:1 read/write)

Storage:
- Tweet: 140 chars + metadata = ~500 bytes
- 100M tweets/day × 500 bytes = 50 GB/day
- 5 years: 50GB × 365 × 5 = 91 TB

Traffic:
- Writes: 100M / 86400 = 1,157 tweets/sec
- Reads: 10B / 86400 = 115,740 reads/sec
- Peak: 3× average = 350K reads/sec
```

### 3. High-Level Design
```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│    Client    │───►│ Load Balancer│───►│  API Server  │
└──────────────┘    └──────────────┘    └──────┬───────┘
                                               │
         ┌─────────────────────────────────────┼─────────────────┐
         │                                     │                 │
         ▼                                     ▼                 ▼
┌──────────────┐                    ┌──────────────┐    ┌──────────────┐
│    Cache     │                    │   Database   │    │    Queue     │
│   (Redis)    │                    │ (PostgreSQL) │    │   (Kafka)    │
└──────────────┘                    └──────────────┘    └──────────────┘
```

### 4. Deep Dive
- Database schema
- API endpoints
- Caching strategy
- Data partitioning
- Replication

### 5. Trade-offs
- Consistency vs Availability
- Cost vs Performance
- Complexity vs Features

---

## 🎯 Common System Design Questions

| System | Key Challenges |
|--------|----------------|
| **URL Shortener** | Hash generation, redirection speed, analytics |
| **Twitter** | Feed generation, celebrity problem, real-time |
| **Instagram** | Image storage, CDN, recommendation |
| **Uber** | Real-time location, matching, surge pricing |
| **WhatsApp** | Messaging delivery, presence, encryption |
| **YouTube** | Video storage, encoding, streaming |
| **Google Search** | Web crawling, indexing, ranking |
| **Dropbox** | File sync, chunking, deduplication |

---

## 📖 Further Reading

- "Designing Data-Intensive Applications" by Martin Kleppmann
- "System Design Interview" by Alex Xu (Vol 1 & 2)
- "Building Microservices" by Sam Newman

---

**Next:** [Chapter 03: Database Engineering →](./03-database-engineering.md)


