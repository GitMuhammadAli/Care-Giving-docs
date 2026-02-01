# 🏛️ Monolith vs Microservices Complete Guide

> A comprehensive guide to choosing between Monolith and Microservices - decision frameworks, migration strategies, and when to use each architecture.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "A monolith is a single deployable unit where all functionality is tightly coupled; microservices are independently deployable services that communicate over the network. Neither is inherently better - choose based on team size, domain complexity, and scaling needs."

### The Key Trade-offs (Memorize!)
```
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  MONOLITH                        MICROSERVICES                  │
│  ─────────                       ─────────────                  │
│  ✓ Simple to develop             ✓ Independent deployments     │
│  ✓ Easy to debug                 ✓ Scale individual services   │
│  ✓ One deploy, done              ✓ Tech stack flexibility      │
│  ✓ No network latency            ✓ Team autonomy               │
│  ✓ ACID transactions             ✓ Fault isolation             │
│                                                                  │
│  ✗ Scales as one unit            ✗ Network complexity          │
│  ✗ All-or-nothing deploy         ✗ Distributed debugging       │
│  ✗ Tech stack locked             ✗ Data consistency challenges │
│  ✗ Team coupling                 ✗ Operational overhead        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The Decision Heuristics (Memorize!)
```
START WITH MONOLITH WHEN:
├── Team < 10 engineers
├── Domain not well understood
├── Startup / MVP / proving product-market fit
├── Simple scaling needs
└── Need to move fast

CONSIDER MICROSERVICES WHEN:
├── Team > 20-30 engineers
├── Clear bounded contexts
├── Different scaling needs per domain
├── Organizational autonomy needed
└── Parts of system need different tech stacks
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Distributed monolith"** | "If services share a DB, you have a distributed monolith - worst of both worlds" |
| **"Strangler fig"** | "We migrated using strangler fig - wrapping old code, gradually replacing" |
| **"Bounded context"** | "Each microservice should align with a bounded context from DDD" |
| **"Modular monolith"** | "We did modular monolith first - clear boundaries, but single deploy" |
| **"Conway's Law"** | "Architecture reflects team structure - 4 teams = 4 services" |
| **"Premature decomposition"** | "Splitting too early is premature decomposition - you'll get boundaries wrong" |

### The Evolution Path
```
TYPICAL EVOLUTION:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  STAGE 1: Simple Monolith (0-10 engineers)                      │
│  └── One codebase, one database                                │
│  └── Deploy as single unit                                     │
│  └── Fast development, easy debugging                          │
│                                                                  │
│  STAGE 2: Modular Monolith (10-30 engineers)                   │
│  └── Internal boundaries (modules)                             │
│  └── Clear APIs between modules                                │
│  └── Still single deployment                                   │
│  └── Prepares for potential split                              │
│                                                                  │
│  STAGE 3: Hybrid (30-50+ engineers)                            │
│  └── Extract high-value services                               │
│  └── Keep core as monolith                                     │
│  └── Strangler fig pattern                                     │
│                                                                  │
│  STAGE 4: Microservices (50+ engineers, clear domains)         │
│  └── Independent services per bounded context                  │
│  └── Team owns service end-to-end                             │
│  └── Significant operational investment                        │
│                                                                  │
│  ⚠️ MANY COMPANIES STOP AT STAGE 2 - THAT'S FINE!             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The "Wow" Statement (Memorize This!)
> "We consciously chose a modular monolith over microservices. With a team of 15, we didn't have the operational capacity for distributed systems - no dedicated DevOps, no service mesh expertise. Instead, we enforced strict module boundaries: each module has a public API, internal types aren't exported, and cross-module database access is forbidden. This gives us 80% of microservices benefits - team independence, clear ownership - without distributed system complexity. When we extracted our first service (payments for PCI compliance), the clean boundaries made it a 2-week project instead of 3 months. We'll extract more when we hit 30+ engineers or need independent scaling."

### Quick Architecture Drawing (Draw This!)
```
┌─────────────────────────────────────────────────────────────────┐
│               ARCHITECTURE COMPARISON                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  MONOLITH                                                       │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    Single Process                        │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐   │   │
│  │  │  Users   │ │  Orders  │ │ Payments │ │ Products │   │   │
│  │  │  Module  │ │  Module  │ │  Module  │ │  Module  │   │   │
│  │  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘   │   │
│  │       └───────┬────┴───────┬────┴────────────┘         │   │
│  │               ▼            ▼                           │   │
│  │         ┌──────────────────────────────┐               │   │
│  │         │       Shared Database         │               │   │
│  │         └──────────────────────────────┘               │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│  MICROSERVICES                                                  │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐          │
│  │  Users   │ │  Orders  │ │ Payments │ │ Products │          │
│  │ Service  │ │ Service  │ │ Service  │ │ Service  │          │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘          │
│       │            │            │            │                  │
│       ▼            ▼            ▼            ▼                  │
│  ┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐            │
│  │Users DB│   │OrdersDB│   │PaymentsDB│  │ProductsDB│          │
│  └────────┘   └────────┘   └────────┘   └────────┘            │
│       ▲            ▲            ▲            ▲                  │
│       └────────────┴─── API ────┴────────────┘                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Interview Rapid Fire (Practice These!)

**Q: "Monolith or microservices?"**
> "It depends. Small team, unknown domain? Monolith. Large team, clear boundaries, different scaling needs? Microservices. Most startups should start monolith - premature decomposition is worse than a big monolith."

**Q: "What's a distributed monolith?"**
> "Worst of both worlds - services that share a database or require synchronized deploys. You have network complexity but no independence. Avoid at all costs."

**Q: "When should you migrate from monolith?"**
> "When monolith becomes the bottleneck: deploy conflicts, team stepping on each other, can't scale specific parts, or compliance requires isolation. Not because it's trendy."

**Q: "What's the strangler fig pattern?"**
> "Gradually replace parts of a monolith. Wrap old system with new, route traffic to new implementation, keep old as fallback. Named after fig trees that grow around and eventually replace host tree."

**Q: "What's a modular monolith?"**
> "Single deployment, but strict internal boundaries. Modules communicate through defined interfaces, can't access each other's database tables. Best of both worlds for mid-size teams."

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "Should we use microservices?"

**Junior Answer:**
> "Yes, microservices are modern and scalable."

**Senior Answer:**
> "Let me ask some questions first:

**1. Team Size & Structure**
- How many engineers? (<20: probably monolith)
- How are teams organized? (One team = one service works)
- Do teams need to deploy independently?

**2. Domain Clarity**
- Are bounded contexts clear?
- Has the domain stabilized? (Early = monolith)
- Where are the natural seams?

**3. Scaling Requirements**
- Do different parts need different scaling?
- What's the expected load pattern?
- Are there hot spots?

**4. Operational Readiness**
- Do you have DevOps expertise?
- Monitoring, tracing, service mesh ready?
- Can you handle distributed debugging?

**My recommendation for most teams:**
Start with a modular monolith. Enforce boundaries now, extract services later when needed. You can always split a well-designed monolith; you can't easily merge poorly designed microservices."

### Follow-up Questions to Expect:

| They Ask | You Answer |
|----------|------------|
| "But Netflix uses microservices!" | "Netflix has 2000+ engineers and clear domain boundaries. They started monolith, migrated over years. Copy their evolution, not their current state." |
| "Isn't monolith technical debt?" | "No. A well-designed monolith with clear boundaries is not debt. A poorly designed microservices system IS debt - distributed complexity without benefits." |
| "What about scaling?" | "Monoliths scale horizontally too. Run multiple instances. Extract specific services when you identify bottlenecks, not before." |
| "How do we prevent spaghetti?" | "Enforce module boundaries: no cross-module database access, public APIs only, lint rules, code reviews. Modular monolith is the answer." |

---

## 📚 Table of Contents

1. [Core Concepts](#1-core-concepts)
2. [Decision Framework](#2-decision-framework)
3. [Migration Strategies](#3-migration-strategies)
4. [Patterns & Pitfalls](#4-patterns--pitfalls)
5. [Real-World Scenarios](#5-real-world-scenarios)
6. [Interview Questions](#6-interview-questions)

---

## 1. Core Concepts

### What is a Monolith?

```
MONOLITH CHARACTERISTICS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  SINGLE DEPLOYABLE UNIT                                         │
│  └── All code in one repository                                │
│  └── One build, one artifact                                   │
│  └── Deploy entire application at once                         │
│                                                                  │
│  SHARED RUNTIME                                                 │
│  └── All modules run in same process                          │
│  └── In-memory function calls (fast)                          │
│  └── Shared memory, shared state                               │
│                                                                  │
│  SHARED DATABASE                                                │
│  └── Single database schema                                    │
│  └── ACID transactions across domains                          │
│  └── Joins across any tables                                   │
│                                                                  │
│  TYPES OF MONOLITHS:                                           │
│  ├── Ball of Mud: No structure, everything coupled            │
│  ├── Layered: UI → Business → Data (horizontal)               │
│  └── Modular: Clear vertical slices (GOOD!)                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### What are Microservices?

```
MICROSERVICES CHARACTERISTICS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  INDEPENDENTLY DEPLOYABLE                                       │
│  └── Each service deployed separately                          │
│  └── Different release cycles                                  │
│  └── Can use different tech stacks                             │
│                                                                  │
│  OWN THEIR DATA                                                │
│  └── Database per service                                      │
│  └── No shared database access                                 │
│  └── Data duplication is okay                                  │
│                                                                  │
│  COMMUNICATE OVER NETWORK                                       │
│  └── REST, gRPC, messaging                                     │
│  └── Network is unreliable                                     │
│  └── Latency is real                                           │
│                                                                  │
│  ORGANIZED AROUND BUSINESS CAPABILITIES                        │
│  └── One service = one bounded context                         │
│  └── Team owns service end-to-end                             │
│  └── Autonomy over technology choices                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The Modular Monolith (Best of Both?)

```typescript
// ════════════════════════════════════════════════════════════════
// MODULAR MONOLITH STRUCTURE
// ════════════════════════════════════════════════════════════════

/*
src/
├── modules/
│   ├── users/
│   │   ├── api/           # Public API (what others can use)
│   │   │   ├── index.ts   # Only exported interface
│   │   │   └── types.ts   # Public types
│   │   ├── internal/      # Private implementation
│   │   │   ├── services/
│   │   │   ├── repositories/
│   │   │   └── entities/
│   │   └── index.ts       # Module entry point
│   │
│   ├── orders/
│   │   ├── api/
│   │   ├── internal/
│   │   └── index.ts
│   │
│   ├── payments/
│   │   ├── api/
│   │   ├── internal/
│   │   └── index.ts
│   │
│   └── products/
│       ├── api/
│       ├── internal/
│       └── index.ts
│
├── shared/                # Truly shared utilities
│   ├── database/
│   ├── logging/
│   └── auth/
│
└── main.ts
*/

// Module public API (users/api/index.ts)
export interface UsersModule {
  getUser(id: string): Promise<User>;
  createUser(data: CreateUserInput): Promise<User>;
  validateCredentials(email: string, password: string): Promise<User | null>;
}

// Module implementation (users/index.ts)
import { UsersModule } from './api';
import { UserService } from './internal/services/UserService';
import { UserRepository } from './internal/repositories/UserRepository';

export function createUsersModule(db: Database): UsersModule {
  const repository = new UserRepository(db);
  const service = new UserService(repository);
  
  return {
    getUser: (id) => service.getById(id),
    createUser: (data) => service.create(data),
    validateCredentials: (email, pass) => service.validateCredentials(email, pass),
  };
}

// RULES (enforced via lint/review):
// 1. Modules can only import from other modules' /api
// 2. No direct database access across modules
// 3. No importing from internal/
// 4. Shared code is truly shared (logging, auth)
```

---

## 2. Decision Framework

### The Decision Matrix

```
WHEN TO USE WHAT:
┌────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│  FACTOR              │ MONOLITH          │ MICROSERVICES               │
│  ────────────────────────────────────────────────────────────────────  │
│  Team Size           │ < 20 engineers    │ > 30 engineers              │
│  Domain Knowledge    │ Still learning    │ Well understood             │
│  Product Stage       │ MVP, finding fit  │ Mature, scaling             │
│  Deploy Frequency    │ Weekly/monthly    │ Multiple times/day          │
│  Scaling Needs       │ Uniform           │ Different per service       │
│  Tech Stack          │ One is fine       │ Need flexibility            │
│  Ops Maturity        │ Basic             │ Platform team, DevOps       │
│  Data Consistency    │ Strong needed     │ Eventual okay               │
│  Budget              │ Limited           │ Can invest in infra         │
│                                                                         │
└────────────────────────────────────────────────────────────────────────┘
```

### The Decision Checklist

```
✅ CHOOSE MONOLITH IF:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  □ Team is smaller than 15-20 engineers                        │
│  □ You're building an MVP or early-stage product               │
│  □ Domain boundaries are not yet clear                         │
│  □ You need strong ACID transactions                           │
│  □ You don't have dedicated DevOps/Platform team               │
│  □ Simple deployment is a priority                             │
│  □ You want to move fast and iterate                           │
│  □ Budget for infrastructure is limited                        │
│                                                                  │
│  ⚠️ If you checked 5+, start with monolith                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

✅ CHOOSE MICROSERVICES IF:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  □ Team is larger than 30+ engineers                           │
│  □ Clear bounded contexts exist                                │
│  □ Different services need different scaling                   │
│  □ Teams need to deploy independently                          │
│  □ You have platform/DevOps expertise                          │
│  □ Domain is mature and well-understood                        │
│  □ Different tech stacks needed for different problems         │
│  □ Parts of system have different reliability requirements     │
│  □ Regulatory compliance requires isolation (payments, etc.)   │
│                                                                  │
│  ⚠️ If you checked 5+, consider microservices                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Conway's Law in Practice

```
CONWAY'S LAW:
"Organizations design systems that mirror their communication structure."

┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  TEAM STRUCTURE → ARCHITECTURE                                  │
│  ─────────────────────────────                                  │
│                                                                  │
│  ONE TEAM                   →  MONOLITH                         │
│  ┌────────────────────┐       ┌────────────────────┐           │
│  │   Full-Stack Team  │  →    │     Single App     │           │
│  └────────────────────┘       └────────────────────┘           │
│                                                                  │
│  ═══════════════════════════════════════════════════════════   │
│                                                                  │
│  MULTIPLE TEAMS             →  MICROSERVICES                   │
│  ┌──────────┐ ┌──────────┐    ┌──────────┐ ┌──────────┐       │
│  │ Users    │ │ Orders   │ →  │ Users    │ │ Orders   │       │
│  │ Team     │ │ Team     │    │ Service  │ │ Service  │       │
│  └──────────┘ └──────────┘    └──────────┘ └──────────┘       │
│  ┌──────────┐ ┌──────────┐    ┌──────────┐ ┌──────────┐       │
│  │ Payments │ │ Products │ →  │ Payments │ │ Products │       │
│  │ Team     │ │ Team     │    │ Service  │ │ Service  │       │
│  └──────────┘ └──────────┘    └──────────┘ └──────────┘       │
│                                                                  │
│  INVERSE CONWAY MANEUVER:                                       │
│  Structure teams around desired architecture, not vice versa.   │
│  If you want microservices, organize teams by service first.   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The Signals to Migrate

```
🚨 SIGNS YOU NEED TO MIGRATE FROM MONOLITH:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. DEPLOYMENT CONFLICTS                                        │
│     └── Teams blocking each other's releases                   │
│     └── Merge conflicts are constant                           │
│     └── "We can't deploy because X team isn't ready"           │
│                                                                  │
│  2. SCALING BOTTLENECKS                                        │
│     └── One component needs 10x more resources                 │
│     └── Search needs 8 servers, but whole app gets scaled      │
│     └── Can't optimize one part without affecting others       │
│                                                                  │
│  3. TEAM FRICTION                                              │
│     └── 50+ engineers stepping on each other                   │
│     └── Changes in one area break another                      │
│     └── Nobody owns anything                                   │
│                                                                  │
│  4. TECHNOLOGY CONSTRAINTS                                     │
│     └── Stuck on old framework, can't upgrade                  │
│     └── Need Python ML but app is Java                         │
│     └── Legacy dependencies blocking innovation                │
│                                                                  │
│  5. COMPLIANCE REQUIREMENTS                                    │
│     └── Payments needs PCI isolation                           │
│     └── Healthcare data must be separate                       │
│     └── Different security requirements                        │
│                                                                  │
│  ⚠️ DON'T MIGRATE JUST BECAUSE:                               │
│     - It's trendy                                              │
│     - Other companies do it                                    │
│     - You want to learn microservices                         │
│     - Monolith feels "old"                                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Cost Comparison

```
TOTAL COST OF OWNERSHIP:
┌────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│  COST FACTOR           │ MONOLITH        │ MICROSERVICES               │
│  ────────────────────────────────────────────────────────────────────  │
│  Infrastructure        │ $               │ $$$                         │
│  (each service = more) │                 │ (k8s, load balancers, etc.) │
│                        │                 │                              │
│  Development Speed     │ Fast initially  │ Slow initially              │
│  (setup overhead)      │                 │ (contracts, APIs, infra)    │
│                        │                 │                              │
│  Operational Overhead  │ Low             │ High                        │
│  (monitoring, deploy)  │ (one thing)     │ (N things to monitor)       │
│                        │                 │                              │
│  Debugging             │ Easy            │ Hard                        │
│  (stack traces vs      │ (single process)│ (distributed tracing)       │
│  distributed traces)   │                 │                              │
│                        │                 │                              │
│  Team Required         │ Developers      │ Developers + DevOps +       │
│                        │                 │ Platform engineers          │
│                        │                 │                              │
│  Break-even Point      │ N/A             │ ~30-50 engineers,           │
│                        │                 │ significant scale           │
│                                                                         │
└────────────────────────────────────────────────────────────────────────┘

ROUGH MONTHLY COST ESTIMATE (hypothetical):

Monolith:
- 3 app servers: $500
- 1 database: $200
- Basic monitoring: $50
- Total: ~$750/month

Microservices (8 services):
- 16 containers (2 per service): $800
- 8 databases: $1,600
- Kubernetes cluster: $500
- Service mesh: $200
- APM/Tracing: $500
- Message broker: $200
- API Gateway: $100
- Total: ~$3,900/month

⚠️ 5x more infrastructure cost before you add a single feature
```

---

## 3. Migration Strategies

### Strangler Fig Pattern

```
STRANGLER FIG PATTERN:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Named after strangler fig trees that grow around host trees,   │
│  eventually replacing them.                                     │
│                                                                  │
│  PHASE 1: Wrap                                                  │
│  ┌────────────────────────────────────────────────────────┐    │
│  │                        Facade                           │    │
│  │                          │                              │    │
│  │              ┌───────────┴───────────┐                 │    │
│  │              ▼                       ▼                 │    │
│  │        ┌──────────┐           ┌──────────┐            │    │
│  │        │ New Svc  │           │ Monolith │            │    │
│  │        │  (10%)   │           │  (90%)   │            │    │
│  │        └──────────┘           └──────────┘            │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                  │
│  PHASE 2: Migrate                                               │
│  ┌────────────────────────────────────────────────────────┐    │
│  │                        Facade                           │    │
│  │                          │                              │    │
│  │              ┌───────────┴───────────┐                 │    │
│  │              ▼                       ▼                 │    │
│  │        ┌──────────┐           ┌──────────┐            │    │
│  │        │ New Svc  │           │ Monolith │            │    │
│  │        │  (50%)   │           │  (50%)   │            │    │
│  │        └──────────┘           └──────────┘            │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                  │
│  PHASE 3: Complete                                              │
│  ┌────────────────────────────────────────────────────────┐    │
│  │                        Facade                           │    │
│  │                          │                              │    │
│  │                          ▼                              │    │
│  │                   ┌──────────┐                          │    │
│  │                   │ New Svc  │   Monolith               │    │
│  │                   │  (100%)  │   (deleted)              │    │
│  │                   └──────────┘                          │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

```typescript
// ════════════════════════════════════════════════════════════════
// STRANGLER FIG IMPLEMENTATION
// ════════════════════════════════════════════════════════════════

// 1. Create facade that routes to old or new implementation
class OrdersFacade {
  constructor(
    private monolithClient: MonolithClient,
    private newOrdersService: OrdersServiceClient,
    private featureFlags: FeatureFlagService,
  ) {}
  
  async createOrder(data: CreateOrderInput): Promise<Order> {
    // Check if user should use new service
    if (await this.shouldUseNewService(data.userId)) {
      try {
        return await this.newOrdersService.createOrder(data);
      } catch (error) {
        // Fallback to monolith on error (optional)
        console.error('New service failed, falling back to monolith', error);
        return await this.monolithClient.createOrder(data);
      }
    }
    
    return await this.monolithClient.createOrder(data);
  }
  
  private async shouldUseNewService(userId: string): Promise<boolean> {
    return this.featureFlags.isEnabled('use-new-orders-service', { userId });
  }
}

// 2. Gradually increase traffic to new service
// Week 1: 1% (canary)
// Week 2: 10%
// Week 3: 25%
// Week 4: 50%
// Week 5: 100%

// 3. Once at 100%, remove monolith code
```

### Branch by Abstraction

```typescript
// ════════════════════════════════════════════════════════════════
// BRANCH BY ABSTRACTION
// ════════════════════════════════════════════════════════════════

// Step 1: Create abstraction over existing code
interface NotificationService {
  sendEmail(to: string, subject: string, body: string): Promise<void>;
  sendSMS(to: string, message: string): Promise<void>;
  sendPush(userId: string, notification: PushNotification): Promise<void>;
}

// Step 2: Existing monolith implementation
class MonolithNotificationService implements NotificationService {
  async sendEmail(to: string, subject: string, body: string): Promise<void> {
    // Existing monolith code
    await this.emailModule.send({ to, subject, body });
  }
  
  async sendSMS(to: string, message: string): Promise<void> {
    await this.smsModule.send({ to, message });
  }
  
  async sendPush(userId: string, notification: PushNotification): Promise<void> {
    await this.pushModule.send(userId, notification);
  }
}

// Step 3: New microservice implementation
class MicroserviceNotificationService implements NotificationService {
  constructor(private httpClient: HttpClient) {}
  
  async sendEmail(to: string, subject: string, body: string): Promise<void> {
    await this.httpClient.post('http://notifications-service/email', {
      to, subject, body,
    });
  }
  
  async sendSMS(to: string, message: string): Promise<void> {
    await this.httpClient.post('http://notifications-service/sms', {
      to, message,
    });
  }
  
  async sendPush(userId: string, notification: PushNotification): Promise<void> {
    await this.httpClient.post('http://notifications-service/push', {
      userId, notification,
    });
  }
}

// Step 4: Switch implementation via feature flag
function createNotificationService(flags: FeatureFlags): NotificationService {
  if (flags.isEnabled('use-notification-microservice')) {
    return new MicroserviceNotificationService(httpClient);
  }
  return new MonolithNotificationService();
}
```

### Database Migration Strategies

```typescript
// ════════════════════════════════════════════════════════════════
// DATABASE DECOMPOSITION STRATEGIES
// ════════════════════════════════════════════════════════════════

/*
STRATEGY 1: Shared Database (Temporary)
────────────────────────────────────────
- Both monolith and new service access same DB
- Quick to implement
- DANGEROUS: Creates coupling
- Use only as stepping stone

  [Monolith] ──┐
               ├──► [Shared Database]
  [New Service]┘
*/

/*
STRATEGY 2: Database View (Bridge)
────────────────────────────────────────
- New service accesses DB through view
- View provides abstraction
- Can change schema under the view

  [Monolith] ──────► [Database]
                          │
                       [View]
                          │
  [New Service] ──────────┘
*/

/*
STRATEGY 3: Synchronize Databases (Eventual)
────────────────────────────────────────
- New service has own database
- Data synced via events/CDC
- Eventual consistency

  [Monolith] ──► [Main DB] ──► [CDC/Events] ──► [Service DB] ◄── [New Service]
*/

// Example: CDC (Change Data Capture) sync
class OrdersSyncHandler {
  constructor(
    private monolithDb: Database,
    private ordersServiceDb: Database,
  ) {}
  
  // Listen for changes in monolith database
  async handleOrderChange(change: DatabaseChange) {
    if (change.table === 'orders') {
      switch (change.operation) {
        case 'INSERT':
          await this.ordersServiceDb.orders.create(change.newData);
          break;
        case 'UPDATE':
          await this.ordersServiceDb.orders.update({
            where: { id: change.newData.id },
            data: change.newData,
          });
          break;
        case 'DELETE':
          await this.ordersServiceDb.orders.delete({
            where: { id: change.oldData.id },
          });
          break;
      }
    }
  }
}

/*
STRATEGY 4: API-based Access (Clean)
────────────────────────────────────────
- No direct database access
- All data access via APIs
- Cleanest but slowest migration

  [Monolith] ◄──API──► [New Service]
       │                    │
       ▼                    ▼
  [Main DB]            [Service DB]
*/
```

### Migration Checklist

```
MIGRATION CHECKLIST:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  BEFORE MIGRATION:                                              │
│  □ Clear bounded contexts identified                           │
│  □ API contracts defined                                       │
│  □ Monitoring/observability in place                           │
│  □ Rollback strategy defined                                   │
│  □ Team trained on microservices patterns                      │
│  □ CI/CD for multiple services ready                           │
│  □ Service discovery/registry set up                           │
│                                                                  │
│  DURING MIGRATION:                                              │
│  □ Extract highest-value service first                         │
│  □ Use feature flags for gradual rollout                       │
│  □ Keep monolith functional as fallback                        │
│  □ Sync data between old and new                              │
│  □ Monitor error rates, latency                               │
│  □ Document API changes                                        │
│                                                                  │
│  AFTER EXTRACTION:                                              │
│  □ Remove old code from monolith                              │
│  □ Remove data sync (once confident)                          │
│  □ Update documentation                                        │
│  □ Retrospective: what worked, what didn't                    │
│                                                                  │
│  TIMELINE (per service):                                       │
│  • Planning: 1-2 weeks                                         │
│  • Implementation: 2-4 weeks                                   │
│  • Gradual rollout: 2-4 weeks                                  │
│  • Cleanup: 1 week                                             │
│  • Total: 6-11 weeks per service                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Patterns & Pitfalls

### The Distributed Monolith (Anti-Pattern)

```
⚠️ DISTRIBUTED MONOLITH - WORST OF BOTH WORLDS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  SYMPTOMS:                                                      │
│  ─────────                                                      │
│  • Services share a database                                   │
│  • Services must deploy together                               │
│  • Changing one service breaks others                          │
│  • Circular dependencies between services                      │
│  • One team owns multiple "services"                           │
│                                                                  │
│  WHY IT HAPPENS:                                                │
│  ───────────────                                                │
│  • Split too early (domain not understood)                     │
│  • Split wrong (boundaries don't match business)              │
│  • Shared database for "convenience"                           │
│  • No clear ownership                                          │
│                                                                  │
│  EXAMPLE (BAD):                                                 │
│                                                                  │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐               │
│  │  Users   │────►│  Orders  │────►│ Payments │               │
│  │ Service  │◄────│ Service  │◄────│ Service  │               │
│  └────┬─────┘     └────┬─────┘     └────┬─────┘               │
│       │                │                │                      │
│       └────────────────┴────────────────┘                      │
│                        │                                        │
│                        ▼                                        │
│                 ┌──────────────┐                               │
│                 │ SHARED       │  ← All services               │
│                 │ DATABASE     │    access same tables!       │
│                 └──────────────┘                               │
│                                                                  │
│  RESULT:                                                        │
│  • Network latency of microservices                            │
│  • No deployment independence                                  │
│  • No scaling independence                                     │
│  • Harder debugging than monolith                              │
│  • More infrastructure cost                                    │
│                                                                  │
│  SOLUTION:                                                      │
│  • Either commit to true microservices (separate DBs)          │
│  • Or go back to modular monolith                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Common Pitfalls

```
MICROSERVICES PITFALLS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. PREMATURE DECOMPOSITION                                    │
│     └── Splitting before understanding domain                  │
│     └── Getting boundaries wrong                               │
│     └── Fix: Start monolith, extract later                     │
│                                                                  │
│  2. WRONG BOUNDARIES                                           │
│     └── Services split by technical layer (API, DB, etc.)     │
│     └── Should split by business capability                   │
│     └── Fix: Use DDD bounded contexts                          │
│                                                                  │
│  3. TOO FINE-GRAINED                                           │
│     └── Nano-services for every function                       │
│     └── Massive network overhead                               │
│     └── Fix: One service per bounded context                   │
│                                                                  │
│  4. SHARED DATABASE                                            │
│     └── "Just for now" becomes forever                        │
│     └── Creates tight coupling                                 │
│     └── Fix: Each service owns its data                       │
│                                                                  │
│  5. SYNCHRONOUS EVERYWHERE                                     │
│     └── REST calls for everything                             │
│     └── Cascading failures                                    │
│     └── Fix: Use async messaging where appropriate            │
│                                                                  │
│  6. NO OBSERVABILITY                                           │
│     └── Can't trace requests across services                  │
│     └── Debugging is nightmare                                │
│     └── Fix: Distributed tracing, centralized logging         │
│                                                                  │
│  7. IGNORING DATA CONSISTENCY                                  │
│     └── Expecting ACID across services                        │
│     └── Data inconsistencies                                  │
│     └── Fix: Embrace eventual consistency, saga pattern       │
│                                                                  │
│  8. UNDERESTIMATING OPERATIONAL COMPLEXITY                    │
│     └── No CI/CD for multiple services                        │
│     └── No monitoring strategy                                │
│     └── Fix: Invest in platform/DevOps                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Monolith Pitfalls

```
MONOLITH PITFALLS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. BIG BALL OF MUD                                            │
│     └── No internal structure                                  │
│     └── Everything depends on everything                       │
│     └── Fix: Modular monolith with clear boundaries           │
│                                                                  │
│  2. GOD CLASSES/SERVICES                                       │
│     └── One class does everything                             │
│     └── 5000-line files                                       │
│     └── Fix: Single responsibility, proper abstraction        │
│                                                                  │
│  3. DATABASE COUPLING                                          │
│     └── Any code can access any table                         │
│     └── Schema changes break everything                       │
│     └── Fix: Module-owned tables, repository pattern          │
│                                                                  │
│  4. SHARED MUTABLE STATE                                       │
│     └── Global variables, singletons everywhere               │
│     └── Race conditions, hard to test                         │
│     └── Fix: Dependency injection, explicit state             │
│                                                                  │
│  5. LONG BUILDS/TESTS                                          │
│     └── 30-minute builds                                      │
│     └── Developers avoid running tests                        │
│     └── Fix: Module-level tests, incremental builds           │
│                                                                  │
│  6. SCALING LIMITATIONS                                        │
│     └── Can't scale parts independently                       │
│     └── One component bottlenecks whole app                   │
│     └── Fix: Identify bottlenecks, consider extraction        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Good Patterns for Both

```typescript
// ════════════════════════════════════════════════════════════════
// PATTERNS THAT WORK FOR BOTH ARCHITECTURES
// ════════════════════════════════════════════════════════════════

// 1. CLEAN BOUNDARIES (works for modules or services)
interface OrdersModule {
  // Only expose what others need
  createOrder(data: CreateOrderInput): Promise<Order>;
  getOrder(id: string): Promise<Order | null>;
  cancelOrder(id: string): Promise<void>;
}

// 2. DEPENDENCY INVERSION (decouple from implementations)
interface PaymentGateway {
  charge(amount: number, paymentMethod: PaymentMethod): Promise<ChargeResult>;
  refund(chargeId: string): Promise<RefundResult>;
}

// Module/service doesn't care if payment is internal or external
class OrderService {
  constructor(private paymentGateway: PaymentGateway) {}
  
  async checkout(orderId: string): Promise<void> {
    const order = await this.getOrder(orderId);
    await this.paymentGateway.charge(order.total, order.paymentMethod);
  }
}

// 3. ANTI-CORRUPTION LAYER (protect from external changes)
class ExternalInventoryAdapter implements InventoryService {
  constructor(private externalApi: ExternalInventoryApi) {}
  
  async checkStock(productId: string): Promise<number> {
    // Translate external format to internal
    const external = await this.externalApi.getProduct(productId);
    return external.qty_available; // External uses different naming
  }
}

// 4. EVENT-DRIVEN (works for both, enables future extraction)
interface DomainEvent {
  type: string;
  timestamp: Date;
  payload: unknown;
}

class OrderPlacedEvent implements DomainEvent {
  type = 'order.placed';
  timestamp = new Date();
  
  constructor(public payload: { orderId: string; userId: string; total: number }) {}
}

// In monolith: in-memory event bus
// In microservices: message broker (Kafka, RabbitMQ)
class EventBus {
  private handlers: Map<string, Function[]> = new Map();
  
  publish(event: DomainEvent): void {
    const handlers = this.handlers.get(event.type) || [];
    handlers.forEach(handler => handler(event));
  }
  
  subscribe(eventType: string, handler: Function): void {
    const handlers = this.handlers.get(eventType) || [];
    handlers.push(handler);
    this.handlers.set(eventType, handlers);
  }
}

// 5. CIRCUIT BREAKER (resilience for any architecture)
class CircuitBreaker {
  private failures = 0;
  private isOpen = false;
  
  async execute<T>(fn: () => Promise<T>): Promise<T> {
    if (this.isOpen) {
      throw new Error('Circuit is open');
    }
    
    try {
      const result = await fn();
      this.failures = 0;
      return result;
    } catch (error) {
      this.failures++;
      if (this.failures >= 5) {
        this.isOpen = true;
        setTimeout(() => this.isOpen = false, 30000);
      }
      throw error;
    }
  }
}
```

---

## 5. Real-World Scenarios

### Scenario 1: E-commerce Startup (10 Engineers)

```
SCENARIO: Early-stage e-commerce, finding product-market fit
─────────────────────────────────────────────────────────────────

Team: 10 engineers
Stage: Series A, still iterating on product
Domain: Products, orders, users, payments, inventory

RECOMMENDATION: Modular Monolith

WHY:
• Team small enough to coordinate
• Domain still evolving (will get boundaries wrong)
• Need to ship fast, iterate quickly
• Don't have DevOps expertise for microservices
• Single database makes transactions easy

STRUCTURE:
┌─────────────────────────────────────────────────────────────────┐
│                    E-COMMERCE MONOLITH                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  src/modules/                                                   │
│  ├── catalog/      (products, categories, search)              │
│  ├── orders/       (cart, checkout, order history)             │
│  ├── payments/     (payment processing, refunds)               │
│  ├── users/        (auth, profiles, addresses)                 │
│  ├── inventory/    (stock, warehouses)                         │
│  └── shipping/     (rates, tracking)                           │
│                                                                  │
│  Rules enforced:                                                │
│  • Each module has public API (api/index.ts)                   │
│  • No cross-module database access                             │
│  • Domain events for cross-module communication                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

FUTURE: Extract payments service for PCI compliance when needed.
```

### Scenario 2: Growing SaaS (40 Engineers)

```
SCENARIO: B2B SaaS platform, scaling rapidly
─────────────────────────────────────────────────────────────────

Team: 40 engineers in 6 teams
Stage: Series C, scaling customers
Domain: Users, workspaces, documents, billing, notifications, analytics

RECOMMENDATION: Hybrid (Modular Monolith + Strategic Microservices)

WHY:
• Teams stepping on each other in core monolith
• Billing needs PCI compliance (isolate)
• Analytics needs different tech (Python/ML)
• Search needs independent scaling
• Rest of app fine as modular monolith

ARCHITECTURE:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  EXTRACTED SERVICES:                                            │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐               │
│  │  Billing   │  │  Search    │  │ Analytics  │               │
│  │ (PCI req)  │  │ (Elastic)  │  │  (Python)  │               │
│  └────────────┘  └────────────┘  └────────────┘               │
│       ▲               ▲               ▲                        │
│       │               │               │                        │
│       └───────────────┴───────┬───────┘                        │
│                               │ API calls                      │
│                               ▼                                │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                    CORE MONOLITH                         │  │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐           │  │
│  │  │ Users  │ │Workspcs│ │  Docs  │ │ Notifs │           │  │
│  │  └────────┘ └────────┘ └────────┘ └────────┘           │  │
│  └─────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

MIGRATION TIMELINE:
• Month 1-2: Extract billing (compliance driver)
• Month 3-4: Extract search (scaling driver)
• Month 5-6: Extract analytics (tech stack driver)
• Core monolith remains, further splits only if needed
```

### Scenario 3: Enterprise (200+ Engineers)

```
SCENARIO: Large enterprise platform
─────────────────────────────────────────────────────────────────

Team: 200+ engineers, 20+ teams
Stage: Mature, stable domains
Domain: Multiple product lines, clear boundaries

RECOMMENDATION: Full Microservices

WHY:
• Clear bounded contexts after years of domain modeling
• Teams need autonomous deploy cycles
• Different parts need vastly different scaling
• Can invest in platform engineering team
• Regulatory requirements for isolation

ARCHITECTURE:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  PLATFORM LAYER:                                                │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ API Gateway │ Service Mesh │ Observability │ CI/CD      │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│  PRODUCT A:                    PRODUCT B:                       │
│  ┌───────┐ ┌───────┐          ┌───────┐ ┌───────┐             │
│  │Orders │ │Catalog│          │Booking│ │Pricing│             │
│  └───────┘ └───────┘          └───────┘ └───────┘             │
│  ┌───────┐ ┌───────┐          ┌───────┐ ┌───────┐             │
│  │Payment│ │Shipng │          │Payment│ │AvailPP│             │
│  └───────┘ └───────┘          └───────┘ └───────┘             │
│                                                                  │
│  SHARED SERVICES:                                               │
│  ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐           │
│  │ Auth  │ │Notifs │ │ Files │ │Search │ │Logging│           │
│  └───────┘ └───────┘ └───────┘ └───────┘ └───────┘           │
│                                                                  │
│  REQUIREMENTS:                                                  │
│  • Dedicated platform team (5-10 engineers)                    │
│  • Service mesh (Istio/Linkerd)                               │
│  • Distributed tracing (Jaeger/Zipkin)                        │
│  • CI/CD per service                                          │
│  • Service catalog and documentation                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### What to Extract First

```
EXTRACTION PRIORITY:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. COMPLIANCE-DRIVEN (extract first)                          │
│     └── Payments (PCI DSS)                                     │
│     └── Healthcare data (HIPAA)                                │
│     └── Personal data (GDPR - data residency)                  │
│                                                                  │
│  2. SCALING-DRIVEN (clear bottlenecks)                         │
│     └── Search (needs Elasticsearch, heavy load)              │
│     └── Media processing (CPU intensive)                       │
│     └── Real-time features (WebSockets)                        │
│                                                                  │
│  3. TECHNOLOGY-DRIVEN (different stack needed)                 │
│     └── ML/AI features (Python)                                │
│     └── Real-time analytics (streaming)                        │
│     └── Legacy system replacement                              │
│                                                                  │
│  4. TEAM-DRIVEN (organizational needs)                         │
│     └── Feature team wants autonomy                           │
│     └── Acquired company integration                          │
│     └── External team/vendor ownership                        │
│                                                                  │
│  DON'T EXTRACT:                                                │
│  ✗ Core domain logic (keep close to data)                     │
│  ✗ Tightly coupled features                                   │
│  ✗ Things that need ACID transactions together                │
│  ✗ Just because it "feels" like a service                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Famous Company Examples

```
REAL-WORLD EXAMPLES:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  AMAZON                                                         │
│  └── Started as monolith (1995-2001)                          │
│  └── Gradually extracted services (2001+)                     │
│  └── Now: 100s of services                                    │
│  └── Key: Service-oriented architecture mandate (2002)        │
│                                                                  │
│  NETFLIX                                                        │
│  └── Started as monolith (2008)                               │
│  └── Migrated to AWS + microservices (2009-2016)              │
│  └── Now: 700+ microservices                                  │
│  └── Key: Had to rebuild for streaming scale                  │
│                                                                  │
│  SHOPIFY                                                        │
│  └── Started as monolith (2004)                               │
│  └── Still largely monolith (2024)                            │
│  └── Modular monolith with some extracted services           │
│  └── Key: Works for their scale, no need to change           │
│                                                                  │
│  BASECAMP/HEY                                                   │
│  └── Monolith by choice                                       │
│  └── Profitable, small team, fast shipping                    │
│  └── Key: Right architecture for their context                │
│                                                                  │
│  SEGMENT                                                        │
│  └── Microservices → Back to monolith (2017)                 │
│  └── Premature decomposition caused problems                  │
│  └── Key: Not afraid to reverse course                        │
│                                                                  │
│  LESSON: Success isn't about the architecture pattern.         │
│          It's about choosing right for YOUR context.           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. Interview Questions & Answers

### Basic Questions

**Q1: Monolith vs Microservices - what's the difference?**
> **A:** 
> - **Monolith**: Single deployable unit, all code in one process, shared database, in-memory calls. Simple, fast development, easy debugging.
> - **Microservices**: Independent deployable services, each owns its data, communicate over network. Enables team autonomy, independent scaling, but adds complexity.
>
> Neither is inherently better - choose based on team size, domain clarity, and operational maturity.

**Q2: When should you use microservices?**
> **A:** When you have:
> - Large team (30+ engineers) that can't coordinate in one codebase
> - Clear bounded contexts (domain is well understood)
> - Different scaling needs per domain
> - Need for independent deployment
> - Platform/DevOps expertise to manage complexity
> - Budget for operational overhead
>
> DON'T use just because it's trendy or "modern."

**Q3: What is a distributed monolith?**
> **A:** Worst of both worlds - services that appear separate but are tightly coupled. Signs: shared database, synchronized deploys required, circular dependencies. You get network complexity without independence benefits. Fix: either true microservices (separate DBs) or consolidate back to monolith.

**Q4: What is a modular monolith?**
> **A:** Single deployment with strict internal boundaries. Modules have public APIs, can't access each other's database tables, communicate through defined interfaces. Gives 80% of microservices benefits (team independence, clear ownership) without distributed system complexity. Best choice for most mid-size teams.

### Intermediate Questions

**Q5: How do you migrate from monolith to microservices?**
> **A:** Strangler Fig pattern:
> 1. Identify clear bounded context to extract
> 2. Create facade/proxy in front of monolith
> 3. Implement new service behind facade
> 4. Gradually route traffic to new service (feature flags)
> 5. Once 100%, remove old code from monolith
>
> Start with compliance-driven or scaling-driven services. Don't try to migrate everything at once.

**Q6: What should you extract first?**
> **A:** Priority order:
> 1. **Compliance-driven**: Payments (PCI), healthcare data (HIPAA)
> 2. **Scaling-driven**: Clear bottlenecks (search, media processing)
> 3. **Technology-driven**: Need different stack (ML in Python)
> 4. **Team-driven**: External team, acquired company
>
> DON'T extract: core domain logic, tightly coupled features, things needing ACID together.

**Q7: How do you handle data in microservices?**
> **A:** Each service owns its data:
> - No shared database access
> - Data duplication is okay (denormalization)
> - Sync via events/CDC for eventual consistency
> - Saga pattern for distributed transactions
>
> During migration: can temporarily share DB with views, then fully separate.

**Q8: What's Conway's Law and why does it matter?**
> **A:** "Organizations design systems that mirror their communication structure." If you have 4 teams, you'll likely end up with 4 services. 
>
> **Inverse Conway**: Structure teams around desired architecture first. Want microservices? Create team per service before splitting the code. Don't fight Conway's Law.

### Advanced Questions

**Q9: How do you prevent a modular monolith from becoming a big ball of mud?**
> **A:** Enforce boundaries:
> - Lint rules: only import from module's public API
> - Code reviews: reject cross-module database access
> - Module-owned tables: enforce via naming convention
> - Domain events for cross-module communication
> - Dependency graphs: fail build on circular deps
> - Architecture tests: verify dependency rules
>
> Discipline is key - technical enforcement helps.

**Q10: What's the cost of microservices?**
> **A:**
> - **Infrastructure**: 3-5x more (each service needs its own DB, containers, load balancers)
> - **Team**: Need DevOps/Platform engineers (not just developers)
> - **Cognitive**: Distributed debugging, eventual consistency
> - **Development**: Service contracts, API versioning, network code
>
> Break-even: ~30-50 engineers with clear domains. Before that, the overhead isn't worth it.

**Q11: Should services share code?**
> **A:** Minimize sharing, but some is okay:
> - **Shared libraries**: DTOs, utilities, clients (versioned)
> - **Shared contracts**: API schemas (OpenAPI, protobuf)
> - **Shared infrastructure**: Auth, logging, tracing SDKs
>
> **DON'T share**: Business logic, domain models (leads to coupling). Each service should own its domain completely.

**Q12: How do you handle transactions across services?**
> **A:** You can't have ACID across services. Options:
> - **Saga pattern**: Choreography (events) or orchestration (coordinator)
> - **Eventual consistency**: Accept that data syncs eventually
> - **Compensation**: Undo operations if later steps fail
>
> If you need strong consistency, maybe those aren't separate services.

### Scenario Questions

**Q13: Startup with 8 engineers asks: should we use microservices?**
> **A:** No. Here's why:
> - Team too small to benefit from independence
> - Domain probably not understood yet (will get boundaries wrong)
> - No DevOps expertise to manage complexity
> - Need to move fast, not manage infrastructure
>
> Start with modular monolith. Enforce boundaries now, extract later when you have clear bottlenecks or hit 30+ engineers.

**Q14: Team has a "distributed monolith" - what should they do?**
> **A:** Two options:
> 
> **Option A: Consolidate** (if boundaries were wrong)
> - Merge back into modular monolith
> - Remove network overhead
> - Re-evaluate boundaries
> - Try again when domain is clearer
>
> **Option B: Fix** (if boundaries are right)
> - Separate databases (one per service)
> - Remove synchronous coupling
> - Add async communication (events)
> - Accept eventual consistency
>
> Hint: If deployment is still coordinated, Option A is likely better.

**Q15: Design an architecture for growing e-commerce (current: 20 engineers, projected: 50)**
> **A:** Phased approach:
> 
> **Now (20 engineers)**: Modular monolith
> - Clear module boundaries (catalog, orders, payments, users, inventory)
> - Enforce separation (no cross-module DB access)
> - Domain events for communication
>
> **At 30 engineers**: Extract payments
> - PCI compliance requirement
> - Use strangler fig pattern
> - Keep rest as monolith
>
> **At 40+ engineers**: Evaluate further
> - Is search a bottleneck? Extract.
> - Do teams need independence? Consider more extraction.
> - Always: justify each extraction with clear reason

---

## 🎓 Key Takeaways

1. **Start with monolith** - you can always extract later, but can't easily merge
2. **Modular monolith** is often the sweet spot for mid-size teams
3. **Microservices are expensive** - 3-5x infrastructure, need DevOps expertise
4. **Conway's Law is real** - architecture follows team structure
5. **Distributed monolith is worst of both** - avoid at all costs
6. **Extract for clear reasons** - compliance, scaling, technology, not trend
7. **Strangler fig** for migration - gradual, with fallback
8. **Each service owns its data** - no shared databases
9. **Team size matters** - <20: monolith, 30+: consider microservices
10. **Shopify proves monoliths work** - right choice for right context

---

## 📚 Resources

### Books
- "Building Microservices" by Sam Newman
- "Monolith to Microservices" by Sam Newman
- "Domain-Driven Design" by Eric Evans (for bounded contexts)

### Articles
- [Martin Fowler: Microservices](https://martinfowler.com/articles/microservices.html)
- [Martin Fowler: Monolith First](https://martinfowler.com/bliki/MonolithFirst.html)
- [Segment: Goodbye Microservices](https://segment.com/blog/goodbye-microservices/)

### Patterns
- [Strangler Fig Pattern](https://martinfowler.com/bliki/StranglerFigApplication.html)
- [Branch by Abstraction](https://martinfowler.com/bliki/BranchByAbstraction.html)
- [Saga Pattern](https://microservices.io/patterns/data/saga.html)


