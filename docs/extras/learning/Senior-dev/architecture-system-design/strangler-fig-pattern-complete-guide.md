# 🌿 Strangler Fig Pattern Complete Guide

> A comprehensive guide to the Strangler Fig Pattern - incrementally migrating legacy systems, rewriting applications piece by piece, and avoiding the risks of big-bang rewrites.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "The Strangler Fig Pattern incrementally replaces a legacy system by building new functionality around it, gradually routing traffic to new implementations until the old system can be safely removed - like a strangler fig tree that grows around its host."

### Why "Strangler Fig"?
```
THE METAPHOR:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  In nature, strangler fig trees:                                │
│  1. Grow around an existing host tree                          │
│  2. Gradually wrap and replace the host                        │
│  3. Eventually the host dies, fig tree stands alone            │
│                                                                  │
│  In software:                                                   │
│  1. Build new system around legacy                             │
│  2. Gradually route traffic to new system                      │
│  3. Eventually legacy is unused, can be removed               │
│                                                                  │
│  🌳 Year 1        🌳 Year 3        🌿 Year 5                   │
│  Host tree       Fig wrapping      Fig only                    │
│  (Legacy)        (Migration)       (New system)                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The 3 Phases (Memorize!)
```
STRANGLER FIG PHASES:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  PHASE 1: TRANSFORM                                             │
│  ─────────────────                                              │
│  └── Add facade/proxy in front of legacy                       │
│  └── All traffic flows through facade                          │
│  └── No behavior change yet                                    │
│                                                                  │
│  PHASE 2: CO-EXIST                                              │
│  ────────────────                                               │
│  └── Build new implementation behind facade                    │
│  └── Route some traffic to new (feature flags)                │
│  └── Both systems run in parallel                              │
│  └── Gradually increase traffic to new                         │
│                                                                  │
│  PHASE 3: ELIMINATE                                             │
│  ─────────────────                                              │
│  └── New system handles 100% traffic                           │
│  └── Remove facade routing logic                               │
│  └── Decommission legacy system                                │
│  └── Clean up technical debt                                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Facade/Proxy"** | "We put a facade in front of legacy to intercept all requests" |
| **"Shadow traffic"** | "We shadow traffic to new system to verify behavior before switching" |
| **"Feature toggle"** | "Traffic routing is controlled by feature toggles per endpoint" |
| **"Seam"** | "We identified natural seams in the legacy system to extract" |
| **"Anti-corruption layer"** | "The ACL translates between legacy and new data models" |
| **"Dark launching"** | "We dark launch features - run new code without returning results" |
| **"Parallel running"** | "Both systems process requests; we compare results for validation" |

### Strangler Fig vs Big-Bang Rewrite
```
COMPARISON:
┌────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│  BIG-BANG REWRITE                 STRANGLER FIG                        │
│  ─────────────────                ─────────────                        │
│  ✗ High risk                      ✓ Low risk (incremental)             │
│  ✗ Long time to value             ✓ Early value delivery               │
│  ✗ All-or-nothing                 ✓ Can stop/pause anytime             │
│  ✗ No fallback                    ✓ Legacy as fallback                 │
│  ✗ Team blocked on rewrite        ✓ Continue feature development       │
│  ✗ "Second system syndrome"       ✓ Incremental learning               │
│  ✗ Often fails/abandoned          ✓ Higher success rate                │
│                                                                         │
│  BIG-BANG: 2 years later...                                           │
│  "We're 60% done, need 6 more months" (forever)                       │
│                                                                         │
│  STRANGLER: 2 years later...                                          │
│  "70% of traffic on new system, 30% still on legacy"                  │
│  (Both work, can pivot if needed)                                     │
│                                                                         │
└────────────────────────────────────────────────────────────────────────┘
```

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Initial traffic to new | **1-5%** | Start small, verify behavior |
| Rollout increment | **Double** | 5% → 10% → 20% → 40% → 80% → 100% |
| Migration timeline | **1-3 years** | For large systems |
| Parallel run period | **2-4 weeks** | Before full cutover |
| Rollback time | **< 5 minutes** | Just change routing |

### The "Wow" Statement (Memorize This!)
> "We migrated our 15-year-old .NET monolith to Node.js microservices using the Strangler Fig pattern over 18 months. Instead of a risky rewrite, we put an API gateway in front of the legacy system and started extracting services one by one. Each service went through shadow traffic validation, then 5% canary, then gradual rollout. The beauty is we could still ship features in the legacy system during migration - business wasn't blocked. When we hit issues with the new payments service, we rolled back in 2 minutes by flipping the route. Today, 95% of traffic goes to new services, and we're decommissioning the last legacy components. Zero downtime throughout the entire migration."

### Quick Architecture Drawing (Draw This!)
```
┌─────────────────────────────────────────────────────────────────┐
│               STRANGLER FIG PATTERN EVOLUTION                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  BEFORE (Direct to Legacy)                                      │
│  ┌──────────┐                                                   │
│  │ Clients  │──────────────────────►┌──────────┐               │
│  └──────────┘                        │  Legacy  │               │
│                                      │  System  │               │
│                                      └──────────┘               │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  PHASE 1: Add Facade                                            │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐               │
│  │ Clients  │────►│  Facade  │────►│  Legacy  │               │
│  └──────────┘     │  (Proxy) │     │  System  │               │
│                   └──────────┘     └──────────┘               │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  PHASE 2: Route Some Traffic                                    │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐               │
│  │ Clients  │────►│  Facade  │─10%►│   New    │               │
│  └──────────┘     │  (Route) │     │ Service  │               │
│                   │          │─90%►┌──────────┐               │
│                   └──────────┘     │  Legacy  │               │
│                                    └──────────┘               │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  PHASE 3: Majority on New                                       │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐               │
│  │ Clients  │────►│  Facade  │─95%►│   New    │               │
│  └──────────┘     │          │     │ Services │               │
│                   │          │─5%─►┌──────────┐               │
│                   └──────────┘     │  Legacy  │               │
│                                    │ (backup) │               │
│                                    └──────────┘               │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  AFTER: Legacy Removed                                          │
│  ┌──────────┐                      ┌──────────┐               │
│  │ Clients  │─────────────────────►│   New    │               │
│  └──────────┘                      │ Services │               │
│                                    └──────────┘               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Interview Rapid Fire (Practice These!)

**Q: "What is the Strangler Fig Pattern?"**
> "Incremental migration strategy where you build new system around legacy, gradually route traffic to new, and eventually decommission legacy. Named after strangler fig trees that grow around host trees."

**Q: "Why not just rewrite?"**
> "Big-bang rewrites are high risk - often fail, take too long, block feature development. Strangler fig is incremental - you can stop anytime, legacy is fallback, deliver value early."

**Q: "How do you route traffic?"**
> "Facade/proxy in front of legacy. Use feature flags or routing rules to split traffic. Start at 5%, increase gradually. Can route by endpoint, user segment, or percentage."

**Q: "What if new system has bugs?"**
> "Roll back by changing routing - takes seconds. Legacy still works. That's the beauty - you always have a fallback. Shadow traffic and parallel running catch issues before full cutover."

**Q: "How long does migration take?"**
> "Depends on system size. Small: 6 months. Large: 1-3 years. The key is it's safe to take that long - business continues, features ship, no deadline pressure."

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "How do you migrate a legacy system?"

**Junior Answer:**
> "Rewrite it in a modern stack."

**Senior Answer:**
> "I'd use the Strangler Fig pattern for incremental migration:

**1. Assess & Plan**
- Identify natural seams in legacy (bounded contexts)
- Prioritize what to migrate first (pain points, business value)
- Set up facade/proxy infrastructure

**2. Extract Incrementally**
- Build new service for one feature
- Shadow traffic to validate behavior
- Gradual rollout (5% → 100%)
- Keep legacy as fallback

**3. Data Strategy**
- Initially: new service reads from legacy DB
- Then: sync data to new service's DB
- Finally: new service owns its data

**4. Continuous Value**
- Business doesn't wait for migration
- Features can ship in legacy or new
- Risk is controlled at each step

Trade-offs:
- Longer total timeline than (theoretical) rewrite
- Running two systems temporarily
- Need to maintain facade/routing
- Worth it for reduced risk"

### Follow-up Questions to Expect:

| They Ask | You Answer |
|----------|------------|
| "What if legacy has no API?" | "Create one - wrap legacy with REST/GraphQL API as first step. This becomes your seam." |
| "How do you handle data?" | "Start with shared DB, then sync, then separate. Event sourcing helps. Accept temporary duplication." |
| "What about testing?" | "Shadow traffic compares old vs new responses. Parallel run for critical paths. Feature flags for gradual exposure." |
| "How do you prioritize what to migrate?" | "Pain points, business value, clear boundaries, low coupling. Don't start with the hardest part." |

---

## 📚 Table of Contents

1. [Core Concepts](#1-core-concepts)
2. [Implementation Strategies](#2-implementation-strategies)
3. [Routing & Traffic Management](#3-routing--traffic-management)
4. [Data Migration](#4-data-migration)
5. [Real-World Examples](#5-real-world-examples)
6. [When to Use / Not Use](#6-when-to-use--not-use)
7. [Interview Questions](#7-interview-questions)

---

## 1. Core Concepts

### Finding the Seams

```
IDENTIFYING SEAMS (Where to Split):
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  GOOD SEAMS (Easy to Extract):                                  │
│  ─────────────────────────────                                  │
│  • Clear API boundary already exists                           │
│  • Minimal shared state with other parts                       │
│  • Well-defined input/output                                   │
│  • Limited database tables                                     │
│  • Low coupling with rest of system                            │
│                                                                  │
│  Examples:                                                      │
│  ├── Authentication/Authorization                              │
│  ├── Email/Notification sending                                │
│  ├── File upload/processing                                    │
│  ├── Search functionality                                      │
│  ├── Reporting/Analytics                                       │
│  └── Payment processing                                        │
│                                                                  │
│  BAD SEAMS (Hard to Extract):                                  │
│  ────────────────────────────                                  │
│  • Deep integration with core logic                            │
│  • Many database joins with other domains                      │
│  • Shared mutable state                                        │
│  • Circular dependencies                                       │
│                                                                  │
│  FINDING SEAMS:                                                │
│  1. Analyze code dependencies (module coupling)                │
│  2. Analyze database relationships (foreign keys)              │
│  3. Talk to business (what are distinct capabilities?)        │
│  4. Check team structure (Conway's Law)                       │
│  5. Look at change patterns (what changes together?)          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The Facade Layer

```typescript
// ════════════════════════════════════════════════════════════════
// FACADE/PROXY: The Strangler's Key Component
// ════════════════════════════════════════════════════════════════

// The facade sits between clients and backend, routing to old or new

interface RouteConfig {
  path: string;
  method: string;
  target: 'legacy' | 'new' | 'split';
  splitPercentage?: number;  // For gradual rollout
  featureFlag?: string;      // For targeting
}

const routes: RouteConfig[] = [
  // Fully migrated
  { path: '/api/users/*', method: 'GET', target: 'new' },
  
  // Still on legacy
  { path: '/api/orders/*', method: '*', target: 'legacy' },
  
  // Gradual rollout (20% to new)
  { path: '/api/products/*', method: 'GET', target: 'split', splitPercentage: 20 },
  
  // Feature flag controlled
  { path: '/api/search/*', method: 'GET', target: 'split', featureFlag: 'new-search' },
];

class StranglerFacade {
  constructor(
    private legacyClient: HttpClient,
    private newServiceClients: Map<string, HttpClient>,
    private featureFlags: FeatureFlagService,
  ) {}
  
  async handleRequest(req: Request): Promise<Response> {
    const route = this.matchRoute(req);
    const target = await this.determineTarget(route, req);
    
    if (target === 'new') {
      return this.routeToNew(req);
    } else {
      return this.routeToLegacy(req);
    }
  }
  
  private async determineTarget(route: RouteConfig, req: Request): Promise<'legacy' | 'new'> {
    if (route.target === 'legacy') return 'legacy';
    if (route.target === 'new') return 'new';
    
    // Split traffic
    if (route.featureFlag) {
      const userId = this.extractUserId(req);
      return this.featureFlags.isEnabled(route.featureFlag, { userId }) ? 'new' : 'legacy';
    }
    
    if (route.splitPercentage) {
      const bucket = this.hashRequest(req) % 100;
      return bucket < route.splitPercentage ? 'new' : 'legacy';
    }
    
    return 'legacy';  // Default to legacy
  }
  
  private hashRequest(req: Request): number {
    // Consistent hashing ensures same user gets same result
    const userId = this.extractUserId(req);
    return hashCode(userId + req.path);
  }
}
```

---

## 2. Implementation Strategies

### Strategy 1: API-Level Strangling

```typescript
// ════════════════════════════════════════════════════════════════
// API-LEVEL: Route entire endpoints to new service
// ════════════════════════════════════════════════════════════════

// Best for: REST/GraphQL APIs with clear endpoint boundaries

// nginx.conf - API Gateway routing
/*
upstream legacy {
    server legacy-app:8080;
}

upstream users_service {
    server users-service:3000;
}

upstream orders_service {
    server orders-service:3001;
}

server {
    listen 80;
    
    # Migrated to new service
    location /api/users {
        proxy_pass http://users_service;
    }
    
    # Partially migrated (split traffic)
    location /api/orders {
        # Use split_clients for consistent user routing
        split_clients "${request_uri}${remote_addr}" $backend {
            20%     orders_service;
            *       legacy;
        }
        proxy_pass http://$backend;
    }
    
    # Still on legacy
    location / {
        proxy_pass http://legacy;
    }
}
*/

// Or with Express.js as facade
import express from 'express';
import { createProxyMiddleware } from 'http-proxy-middleware';

const app = express();

// Migrated endpoints → New service
app.use('/api/users', createProxyMiddleware({
  target: 'http://users-service:3000',
  changeOrigin: true,
}));

// Gradual rollout with feature flag
app.use('/api/orders', async (req, res, next) => {
  const useNew = await shouldUseNewService(req, 'orders-service-migration');
  
  if (useNew) {
    createProxyMiddleware({
      target: 'http://orders-service:3001',
      changeOrigin: true,
    })(req, res, next);
  } else {
    createProxyMiddleware({
      target: 'http://legacy-app:8080',
      changeOrigin: true,
    })(req, res, next);
  }
});

// Everything else → Legacy
app.use('/', createProxyMiddleware({
  target: 'http://legacy-app:8080',
  changeOrigin: true,
}));
```

### Strategy 2: Asset Capture (UI Migration)

```typescript
// ════════════════════════════════════════════════════════════════
// ASSET CAPTURE: Migrate UI piece by piece
// ════════════════════════════════════════════════════════════════

// Best for: Migrating frontend from legacy (Rails, PHP) to SPA (React)

// Scenario: Legacy PHP renders HTML, new React app for specific pages

// nginx.conf
/*
server {
    listen 80;
    
    # New React app handles these routes
    location /app/dashboard {
        proxy_pass http://react-app:3000;
    }
    
    location /app/settings {
        proxy_pass http://react-app:3000;
    }
    
    # Static assets for React app
    location /static {
        proxy_pass http://react-app:3000;
    }
    
    # Everything else → Legacy PHP
    location / {
        proxy_pass http://legacy-php:80;
    }
}
*/

// React app uses same auth cookie as legacy
// User navigates between old and new pages seamlessly

// Anti-corruption layer for API calls
class LegacyApiAdapter {
  // Transform legacy API responses to new format
  async getUser(id: string): Promise<User> {
    const legacyResponse = await fetch(`/legacy-api/users/${id}`);
    const legacy = await legacyResponse.json();
    
    // Map legacy format to new format
    return {
      id: legacy.user_id,
      email: legacy.email_address,
      name: `${legacy.first_name} ${legacy.last_name}`,
      createdAt: new Date(legacy.created_timestamp * 1000),
      // Legacy uses different status codes
      status: this.mapStatus(legacy.status_code),
    };
  }
  
  private mapStatus(legacyStatus: number): UserStatus {
    const mapping: Record<number, UserStatus> = {
      0: 'inactive',
      1: 'active',
      2: 'suspended',
      99: 'deleted',
    };
    return mapping[legacyStatus] || 'unknown';
  }
}
```

### Strategy 3: Branch by Abstraction

```typescript
// ════════════════════════════════════════════════════════════════
// BRANCH BY ABSTRACTION: Swap implementations behind interface
// ════════════════════════════════════════════════════════════════

// Best for: Internal components, when you can't use a proxy

// Step 1: Create abstraction
interface PaymentProcessor {
  charge(amount: number, customerId: string): Promise<PaymentResult>;
  refund(paymentId: string): Promise<RefundResult>;
}

// Step 2: Legacy implementation (existing code, wrapped)
class LegacyPaymentProcessor implements PaymentProcessor {
  constructor(private legacyModule: LegacyPaymentModule) {}
  
  async charge(amount: number, customerId: string): Promise<PaymentResult> {
    // Existing legacy code
    const result = await this.legacyModule.processPayment({
      amt: amount,
      cust_id: customerId,
    });
    
    // Transform to new interface
    return {
      id: result.transaction_id,
      status: result.success ? 'completed' : 'failed',
      amount,
    };
  }
  
  async refund(paymentId: string): Promise<RefundResult> {
    // Wrap legacy
    return this.legacyModule.refundPayment(paymentId);
  }
}

// Step 3: New implementation
class NewPaymentProcessor implements PaymentProcessor {
  constructor(private stripeClient: StripeClient) {}
  
  async charge(amount: number, customerId: string): Promise<PaymentResult> {
    const charge = await this.stripeClient.charges.create({
      amount: amount * 100,  // Stripe uses cents
      currency: 'usd',
      customer: customerId,
    });
    
    return {
      id: charge.id,
      status: charge.status === 'succeeded' ? 'completed' : 'failed',
      amount,
    };
  }
  
  async refund(paymentId: string): Promise<RefundResult> {
    const refund = await this.stripeClient.refunds.create({
      charge: paymentId,
    });
    return { id: refund.id, status: refund.status };
  }
}

// Step 4: Router that switches between implementations
class PaymentProcessorRouter implements PaymentProcessor {
  constructor(
    private legacy: LegacyPaymentProcessor,
    private newProcessor: NewPaymentProcessor,
    private featureFlags: FeatureFlagService,
  ) {}
  
  async charge(amount: number, customerId: string): Promise<PaymentResult> {
    if (await this.featureFlags.isEnabled('new-payment-processor', { customerId })) {
      return this.newProcessor.charge(amount, customerId);
    }
    return this.legacy.charge(amount, customerId);
  }
  
  async refund(paymentId: string): Promise<RefundResult> {
    // Refunds go to whichever system processed the original charge
    const payment = await this.getPaymentInfo(paymentId);
    if (payment.processor === 'new') {
      return this.newProcessor.refund(paymentId);
    }
    return this.legacy.refund(paymentId);
  }
}
```

### Strategy 4: Event Interception

```typescript
// ════════════════════════════════════════════════════════════════
// EVENT INTERCEPTION: Capture events from legacy for new services
// ════════════════════════════════════════════════════════════════

// Best for: Legacy can't be modified, but you can intercept DB/events

// Option A: Database triggers (CDC - Change Data Capture)
/*
PostgreSQL trigger on legacy orders table:

CREATE OR REPLACE FUNCTION notify_order_change()
RETURNS TRIGGER AS $$
BEGIN
  PERFORM pg_notify('order_changes', json_build_object(
    'operation', TG_OP,
    'id', NEW.id,
    'data', row_to_json(NEW)
  )::text);
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER order_change_trigger
AFTER INSERT OR UPDATE ON orders
FOR EACH ROW EXECUTE FUNCTION notify_order_change();
*/

// Node.js listener for CDC events
import { Client } from 'pg';

const pgClient = new Client(process.env.LEGACY_DB_URL);

async function listenToLegacyChanges() {
  await pgClient.connect();
  await pgClient.query('LISTEN order_changes');
  
  pgClient.on('notification', async (msg) => {
    const change = JSON.parse(msg.payload);
    
    // Process in new service
    if (change.operation === 'INSERT') {
      await newOrdersService.handleNewOrder(change.data);
    } else if (change.operation === 'UPDATE') {
      await newOrdersService.handleOrderUpdate(change.id, change.data);
    }
  });
}

// Option B: Debezium for CDC (production-ready)
/*
Debezium captures database changes and publishes to Kafka:

Legacy DB → Debezium → Kafka → New Service

This allows new services to build their own data stores
from legacy database changes without modifying legacy code.
*/

// Kafka consumer in new service
import { Kafka } from 'kafkajs';

const kafka = new Kafka({ brokers: ['kafka:9092'] });
const consumer = kafka.consumer({ groupId: 'new-orders-service' });

async function consumeLegacyChanges() {
  await consumer.subscribe({ topic: 'legacy.public.orders' });
  
  await consumer.run({
    eachMessage: async ({ message }) => {
      const change = JSON.parse(message.value.toString());
      
      // Sync to new service's database
      await prisma.order.upsert({
        where: { legacyId: change.id },
        create: mapLegacyToNew(change),
        update: mapLegacyToNew(change),
      });
    },
  });
}
```

---

## 3. Routing & Traffic Management

### Gradual Rollout Strategies

```
ROLLOUT STRATEGIES:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. PERCENTAGE-BASED                                            │
│     └── 5% → 10% → 25% → 50% → 100%                           │
│     └── Consistent hashing (same user, same result)            │
│     └── Monitor errors at each stage                           │
│                                                                  │
│  2. USER SEGMENT-BASED                                          │
│     └── Internal employees first                               │
│     └── Beta users next                                        │
│     └── New signups (no legacy baggage)                        │
│     └── Finally, all users                                     │
│                                                                  │
│  3. GEOGRAPHIC                                                  │
│     └── One region first (e.g., EU)                           │
│     └── Expand to other regions                                │
│     └── Good for compliance testing                            │
│                                                                  │
│  4. FEATURE-BASED                                               │
│     └── Migrate read operations first                          │
│     └── Then write operations                                  │
│     └── Safer: reads are idempotent                            │
│                                                                  │
│  5. CUSTOMER TIER-BASED                                        │
│     └── Free tier first (lower stakes)                        │
│     └── Enterprise last (higher expectations)                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Shadow Traffic (Dark Launch)

```typescript
// ════════════════════════════════════════════════════════════════
// SHADOW TRAFFIC: Test new system without affecting users
// ════════════════════════════════════════════════════════════════

class ShadowTrafficRouter {
  constructor(
    private legacy: LegacyService,
    private newService: NewService,
    private comparator: ResponseComparator,
    private metrics: MetricsService,
  ) {}
  
  async handleRequest(req: Request): Promise<Response> {
    // Always return legacy response to user
    const legacyPromise = this.legacy.handle(req);
    
    // Shadow request to new service (fire and forget)
    this.shadowToNewService(req, legacyPromise);
    
    // User always gets legacy response
    return legacyPromise;
  }
  
  private async shadowToNewService(
    req: Request,
    legacyPromise: Promise<Response>
  ): Promise<void> {
    try {
      // Run both in parallel
      const [legacyResponse, newResponse] = await Promise.all([
        legacyPromise,
        this.newService.handle(req),
      ]);
      
      // Compare responses
      const comparison = await this.comparator.compare(
        legacyResponse,
        newResponse
      );
      
      // Log differences (don't affect user)
      if (!comparison.match) {
        this.metrics.recordMismatch({
          endpoint: req.path,
          legacyResponse: comparison.legacy,
          newResponse: comparison.new,
          differences: comparison.differences,
        });
      }
      
      // Track latency comparison
      this.metrics.recordLatency({
        endpoint: req.path,
        legacyMs: comparison.legacyLatency,
        newMs: comparison.newLatency,
      });
      
    } catch (error) {
      // New service failure doesn't affect user
      this.metrics.recordShadowError({
        endpoint: req.path,
        error: error.message,
      });
    }
  }
}

// Response comparator
class ResponseComparator {
  async compare(legacy: Response, newResp: Response): Promise<ComparisonResult> {
    const legacyBody = await legacy.clone().json();
    const newBody = await newResp.clone().json();
    
    // Normalize and compare
    const normalizedLegacy = this.normalize(legacyBody);
    const normalizedNew = this.normalize(newBody);
    
    const differences = this.findDifferences(normalizedLegacy, normalizedNew);
    
    return {
      match: differences.length === 0,
      legacy: normalizedLegacy,
      new: normalizedNew,
      differences,
      legacyLatency: legacy.headers.get('x-response-time'),
      newLatency: newResp.headers.get('x-response-time'),
    };
  }
  
  private normalize(body: any): any {
    // Ignore fields that are expected to differ
    const copy = JSON.parse(JSON.stringify(body));
    delete copy.timestamp;
    delete copy.requestId;
    // Sort arrays for consistent comparison
    // ... normalization logic
    return copy;
  }
}
```

### Parallel Running (Dual Write)

```typescript
// ════════════════════════════════════════════════════════════════
// PARALLEL RUNNING: Both systems process, compare results
// ════════════════════════════════════════════════════════════════

class ParallelRunner {
  constructor(
    private legacy: LegacyService,
    private newService: NewService,
    private config: ParallelConfig,
  ) {}
  
  async handleWrite(req: WriteRequest): Promise<Response> {
    // For critical operations, run both and compare
    const legacyPromise = this.legacy.write(req);
    const newPromise = this.newService.write(req);
    
    try {
      const [legacyResult, newResult] = await Promise.all([
        legacyPromise,
        newPromise,
      ]);
      
      // Compare results
      if (!this.resultsMatch(legacyResult, newResult)) {
        await this.handleMismatch(req, legacyResult, newResult);
      }
      
      // Return based on config
      if (this.config.primarySystem === 'legacy') {
        return legacyResult;
      }
      return newResult;
      
    } catch (error) {
      // If one fails, use the other
      return this.handlePartialFailure(legacyPromise, newPromise);
    }
  }
  
  private async handleMismatch(
    req: WriteRequest,
    legacy: Response,
    newResp: Response
  ): Promise<void> {
    // Log for investigation
    console.error('MISMATCH DETECTED', {
      request: req,
      legacy: await legacy.clone().json(),
      new: await newResp.clone().json(),
    });
    
    // Optionally reconcile
    if (this.config.reconcileOnMismatch) {
      await this.reconciler.reconcile(legacy, newResp);
    }
    
    // Alert if critical
    if (this.config.alertOnMismatch) {
      await this.alerting.send({
        severity: 'high',
        message: 'Parallel running mismatch detected',
      });
    }
  }
}
```

### Rollback Strategy

```typescript
// ════════════════════════════════════════════════════════════════
// INSTANT ROLLBACK: Switch traffic back to legacy
// ════════════════════════════════════════════════════════════════

class MigrationController {
  constructor(
    private featureFlags: FeatureFlagService,
    private metrics: MetricsService,
    private alerting: AlertingService,
  ) {}
  
  // Automated rollback based on error rates
  async monitorAndRollback(serviceName: string): Promise<void> {
    const errorRate = await this.metrics.getErrorRate(serviceName, '5m');
    const latencyP99 = await this.metrics.getLatencyP99(serviceName, '5m');
    
    const thresholds = {
      maxErrorRate: 0.02,      // 2%
      maxLatencyP99: 1000,     // 1 second
    };
    
    if (errorRate > thresholds.maxErrorRate || latencyP99 > thresholds.maxLatencyP99) {
      await this.triggerRollback(serviceName, { errorRate, latencyP99 });
    }
  }
  
  async triggerRollback(serviceName: string, reason: any): Promise<void> {
    console.log(`ROLLING BACK ${serviceName}`, reason);
    
    // Disable feature flag → traffic goes to legacy
    await this.featureFlags.update(`use-new-${serviceName}`, {
      enabled: false,
    });
    
    // Alert team
    await this.alerting.send({
      severity: 'high',
      message: `Automatic rollback triggered for ${serviceName}`,
      details: reason,
    });
    
    // Log for post-mortem
    await this.logRollback(serviceName, reason);
  }
  
  // Manual rollback (emergency)
  async emergencyRollback(serviceName: string): Promise<void> {
    // Immediately disable all traffic to new service
    await this.featureFlags.update(`use-new-${serviceName}`, {
      enabled: false,
      percentage: 0,
    });
    
    console.log(`EMERGENCY ROLLBACK: ${serviceName}`);
  }
}

// Circuit breaker for automatic protection
class MigrationCircuitBreaker {
  private failures = 0;
  private isOpen = false;
  
  async execute<T>(
    newServiceFn: () => Promise<T>,
    legacyFn: () => Promise<T>
  ): Promise<T> {
    if (this.isOpen) {
      // Circuit open: use legacy
      return legacyFn();
    }
    
    try {
      const result = await newServiceFn();
      this.failures = 0;
      return result;
    } catch (error) {
      this.failures++;
      
      if (this.failures >= 5) {
        this.isOpen = true;
        // Auto-reset after 5 minutes
        setTimeout(() => {
          this.isOpen = false;
          this.failures = 0;
        }, 5 * 60 * 1000);
      }
      
      // Fallback to legacy
      return legacyFn();
    }
  }
}
```

---

## 4. Data Migration

### Data Migration Strategies

```
DATA MIGRATION APPROACHES:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  PHASE 1: SHARED DATABASE (Temporary)                           │
│  ─────────────────────────────────────                          │
│  New service reads from legacy database                         │
│  Quick to implement, but creates coupling                       │
│  Use as stepping stone only                                    │
│                                                                  │
│  [Legacy] ──┐                                                   │
│             ├──► [Shared Database]                              │
│  [New Svc]──┘                                                   │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  PHASE 2: SYNC + SEPARATE                                       │
│  ─────────────────────────                                      │
│  New service has own database                                  │
│  Data synced from legacy via CDC/events                        │
│  Both databases in sync                                        │
│                                                                  │
│  [Legacy] ──► [Legacy DB] ──CDC──► [New DB] ◄── [New Svc]      │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  PHASE 3: NEW SERVICE OWNS DATA                                │
│  ─────────────────────────────                                  │
│  New service is source of truth                                │
│  Legacy syncs FROM new service (if needed)                     │
│  Ready to decommission legacy                                  │
│                                                                  │
│  [Legacy] ◄──sync── [New DB] ◄── [New Svc]                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Implementing Data Sync

```typescript
// ════════════════════════════════════════════════════════════════
// DATA SYNC: Legacy → New Service
// ════════════════════════════════════════════════════════════════

// Strategy 1: Initial bulk load + CDC for ongoing changes

class DataMigrationService {
  constructor(
    private legacyDb: LegacyDatabase,
    private newDb: NewDatabase,
    private cdc: CDCService,
  ) {}
  
  async migrate(): Promise<void> {
    // Step 1: Initial bulk load
    console.log('Starting initial data migration...');
    await this.bulkLoad();
    
    // Step 2: Start CDC to capture changes during migration
    console.log('Starting CDC stream...');
    await this.startCDC();
    
    // Step 3: Verify data consistency
    console.log('Verifying data consistency...');
    await this.verifyConsistency();
  }
  
  private async bulkLoad(): Promise<void> {
    // Migrate in batches to avoid memory issues
    const batchSize = 1000;
    let offset = 0;
    
    while (true) {
      const legacyRecords = await this.legacyDb.query(
        `SELECT * FROM orders ORDER BY id LIMIT ${batchSize} OFFSET ${offset}`
      );
      
      if (legacyRecords.length === 0) break;
      
      // Transform and insert
      const transformed = legacyRecords.map(this.transformRecord);
      await this.newDb.orders.createMany({ data: transformed });
      
      offset += batchSize;
      console.log(`Migrated ${offset} records...`);
    }
  }
  
  private transformRecord(legacy: LegacyOrder): NewOrder {
    return {
      id: legacy.order_id,
      customerId: legacy.cust_id,
      total: legacy.total_amount / 100,  // Legacy stores cents
      status: mapStatus(legacy.status_code),
      createdAt: new Date(legacy.created_ts * 1000),
      items: JSON.parse(legacy.items_json),
    };
  }
  
  private async startCDC(): Promise<void> {
    // Listen for changes in legacy database
    this.cdc.subscribe('orders', async (change) => {
      if (change.operation === 'INSERT' || change.operation === 'UPDATE') {
        const transformed = this.transformRecord(change.data);
        await this.newDb.orders.upsert({
          where: { id: transformed.id },
          create: transformed,
          update: transformed,
        });
      } else if (change.operation === 'DELETE') {
        await this.newDb.orders.delete({
          where: { id: change.data.order_id },
        });
      }
    });
  }
  
  private async verifyConsistency(): Promise<void> {
    // Spot check records
    const sampleIds = await this.legacyDb.query(
      'SELECT order_id FROM orders ORDER BY RANDOM() LIMIT 100'
    );
    
    for (const { order_id } of sampleIds) {
      const legacy = await this.legacyDb.query(
        `SELECT * FROM orders WHERE order_id = '${order_id}'`
      );
      const newRecord = await this.newDb.orders.findUnique({
        where: { id: order_id },
      });
      
      if (!this.recordsMatch(legacy[0], newRecord)) {
        console.error(`MISMATCH: ${order_id}`);
      }
    }
  }
}
```

### Handling Write Operations During Migration

```typescript
// ════════════════════════════════════════════════════════════════
// DUAL WRITE: Write to both systems during migration
// ════════════════════════════════════════════════════════════════

class DualWriteOrderService {
  constructor(
    private legacyDb: LegacyDatabase,
    private newDb: NewDatabase,
    private featureFlags: FeatureFlagService,
  ) {}
  
  async createOrder(data: CreateOrderInput): Promise<Order> {
    const migrationPhase = await this.featureFlags.getString('order-migration-phase');
    
    switch (migrationPhase) {
      case 'legacy-primary':
        // Write to legacy first, then sync to new
        return this.legacyPrimaryWrite(data);
        
      case 'dual-write':
        // Write to both, legacy is source of truth
        return this.dualWrite(data);
        
      case 'new-primary':
        // Write to new first, sync to legacy for compatibility
        return this.newPrimaryWrite(data);
        
      case 'new-only':
        // Only write to new system
        return this.newOnlyWrite(data);
        
      default:
        return this.legacyPrimaryWrite(data);
    }
  }
  
  private async legacyPrimaryWrite(data: CreateOrderInput): Promise<Order> {
    // Write to legacy
    const legacyOrder = await this.legacyDb.insertOrder(data);
    
    // Async sync to new (don't block response)
    this.syncToNew(legacyOrder).catch(err => {
      console.error('Sync to new failed', err);
      // Queue for retry
    });
    
    return this.transformToOrder(legacyOrder);
  }
  
  private async dualWrite(data: CreateOrderInput): Promise<Order> {
    // Write to both in parallel
    const [legacyOrder, newOrder] = await Promise.all([
      this.legacyDb.insertOrder(data),
      this.newDb.orders.create({ data: this.transformToNew(data) }),
    ]);
    
    // Verify they match (for monitoring)
    if (!this.ordersMatch(legacyOrder, newOrder)) {
      console.error('Dual write mismatch', { legacyOrder, newOrder });
    }
    
    // Return legacy as source of truth
    return this.transformToOrder(legacyOrder);
  }
  
  private async newPrimaryWrite(data: CreateOrderInput): Promise<Order> {
    // Write to new first
    const newOrder = await this.newDb.orders.create({
      data: this.transformToNew(data),
    });
    
    // Sync to legacy for backward compatibility
    this.syncToLegacy(newOrder).catch(err => {
      console.error('Sync to legacy failed', err);
    });
    
    return newOrder;
  }
  
  private async newOnlyWrite(data: CreateOrderInput): Promise<Order> {
    // Only new system
    return this.newDb.orders.create({
      data: this.transformToNew(data),
    });
  }
}

// ════════════════════════════════════════════════════════════════
// ANTI-CORRUPTION LAYER: Translate between legacy and new models
// ════════════════════════════════════════════════════════════════

class OrderAntiCorruptionLayer {
  // Legacy → New
  legacyToNew(legacy: LegacyOrder): NewOrder {
    return {
      id: legacy.order_id,
      customerId: legacy.cust_id,
      status: this.mapStatus(legacy.status_code),
      total: {
        amount: legacy.total_amount,
        currency: legacy.currency || 'USD',
      },
      items: this.parseItems(legacy.items_json),
      createdAt: new Date(legacy.created_ts * 1000),
      metadata: {
        legacyId: legacy.order_id,
        migratedAt: new Date(),
      },
    };
  }
  
  // New → Legacy (for backward compatibility)
  newToLegacy(order: NewOrder): LegacyOrder {
    return {
      order_id: order.id,
      cust_id: order.customerId,
      status_code: this.reverseMapStatus(order.status),
      total_amount: order.total.amount,
      currency: order.total.currency,
      items_json: JSON.stringify(order.items),
      created_ts: Math.floor(order.createdAt.getTime() / 1000),
    };
  }
  
  private mapStatus(code: number): OrderStatus {
    const map: Record<number, OrderStatus> = {
      0: 'pending',
      1: 'confirmed',
      2: 'shipped',
      3: 'delivered',
      9: 'cancelled',
    };
    return map[code] || 'unknown';
  }
}
```

---

## 5. Real-World Examples

### Example 1: E-commerce Platform Migration

```
SCENARIO: Migrate 10-year-old PHP monolith to Node.js microservices
─────────────────────────────────────────────────────────────────────

LEGACY SYSTEM:
• PHP 5.6 monolith (no longer supported)
• MySQL database with 500+ tables
• 50+ developers, 10M users
• Can't do big-bang: business can't stop

MIGRATION TIMELINE: 24 months

PHASE 1: Foundation (Months 1-3)
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  • Deploy API Gateway (Kong) in front of PHP                   │
│  • All traffic now flows through Gateway                       │
│  • Set up observability (Datadog, Jaeger)                     │
│  • No behavior change - just routing through Gateway          │
│                                                                  │
│  [Clients] ──► [API Gateway] ──► [PHP Monolith]               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

PHASE 2: First Service - Search (Months 4-6)
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  WHY SEARCH FIRST:                                              │
│  • Clear boundary (read-only)                                  │
│  • Needs Elasticsearch (PHP can't scale)                       │
│  • Low risk (search errors are tolerable)                      │
│                                                                  │
│  MIGRATION:                                                     │
│  1. Build Node.js search service                               │
│  2. Index data from MySQL to Elasticsearch                     │
│  3. Shadow traffic: compare results                            │
│  4. Gradual rollout: 5% → 25% → 100%                          │
│  5. Remove search code from PHP                                │
│                                                                  │
│  [Gateway] ──search──► [Search Service (Node.js)]              │
│            ──other───► [PHP Monolith]                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

PHASE 3: User Service (Months 7-9)
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  WHY USERS:                                                     │
│  • Auth is shared - extract once, reuse everywhere             │
│  • Clear API boundary                                          │
│  • Enables future services to not depend on PHP               │
│                                                                  │
│  CHALLENGES:                                                    │
│  • Session management: migrate from PHP sessions to JWT        │
│  • Password hashes: support both old (md5) and new (bcrypt)   │
│  • Dual write during migration                                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

PHASE 4: Product Catalog (Months 10-12)
PHASE 5: Cart & Checkout (Months 13-16)
PHASE 6: Order Management (Months 17-20)
PHASE 7: Payments - External Service (Months 21-22)
PHASE 8: Decommission PHP (Months 23-24)

RESULTS:
• Zero downtime during entire migration
• Features continued shipping throughout
• PHP finally shut down month 24
• 3x improvement in page load time
• Team can now hire Node.js developers
```

### Example 2: Monolith to Serverless

```
SCENARIO: Migrate Rails monolith to AWS Lambda
─────────────────────────────────────────────────────────────────────

LEGACY: Rails 4 on EC2
TARGET: Lambda + API Gateway + DynamoDB

STRATEGY: Event-driven extraction

PHASE 1: Add Event Bus
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Before extracting services, add event publishing               │
│  Rails publishes domain events to EventBridge                  │
│                                                                  │
│  class Order < ApplicationRecord                               │
│    after_create do                                             │
│      EventPublisher.publish('order.created', self.to_event)   │
│    end                                                         │
│  end                                                           │
│                                                                  │
│  [Rails] ──events──► [EventBridge] ──► (future services)       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

PHASE 2: Extract Notification Service to Lambda
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Lambda subscribes to events, sends emails                     │
│  Rails no longer handles email sending                         │
│                                                                  │
│  [Rails] ──events──► [EventBridge] ──► [Lambda: Notifications] │
│                                                                  │
│  # Lambda handler                                              │
│  def handler(event, context):                                  │
│      for record in event['Records']:                          │
│          event_data = json.loads(record['body'])              │
│          if event_data['type'] == 'order.created':            │
│              send_order_confirmation(event_data)              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

PHASE 3: Extract Image Processing
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Heavy CPU task → perfect for Lambda                           │
│                                                                  │
│  [S3 Upload] ──trigger──► [Lambda: Image Processor]            │
│                              │                                  │
│                              ▼                                  │
│                          [S3: Thumbnails]                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

PHASE 4: API Migration (Gradual)
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  API Gateway routes some endpoints to Lambda                   │
│                                                                  │
│  [API Gateway]                                                 │
│      │                                                         │
│      ├── /api/users/*    ──► [Lambda: Users]                  │
│      ├── /api/products/* ──► [Lambda: Products]               │
│      └── /api/*          ──► [Rails] (remainder)               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

RESULTS:
• 70% cost reduction (pay-per-use vs always-on EC2)
• Auto-scaling (no capacity planning)
• Rails still runs for complex features
• Migrated over 18 months
```

### Example 3: Database Migration

```
SCENARIO: Oracle to PostgreSQL migration
─────────────────────────────────────────────────────────────────────

CHALLENGE: 500GB Oracle database, complex PL/SQL procedures

STRATEGY: Gradual data migration with dual reads

PHASE 1: Read Replication
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Set up real-time replication from Oracle to PostgreSQL        │
│                                                                  │
│  [App] ──writes──► [Oracle]                                    │
│                       │                                         │
│                   [CDC/Debezium]                                │
│                       │                                         │
│                       ▼                                         │
│                   [PostgreSQL] (read replica)                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

PHASE 2: Read Traffic Shift
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Gradually move read queries to PostgreSQL                     │
│                                                                  │
│  [App] ──writes────────────► [Oracle]                          │
│        ──reads (90%)────────► [Oracle]                         │
│        ──reads (10%)────────► [PostgreSQL]                     │
│                                                                  │
│  Monitor: query results match? latency acceptable?             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

PHASE 3: Write Cutover
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Move writes to PostgreSQL                                     │
│                                                                  │
│  [App] ──writes──► [PostgreSQL]                                │
│        ──reads───► [PostgreSQL]                                │
│                       │                                         │
│                   [Sync to Oracle] (for legacy apps)           │
│                       │                                         │
│                       ▼                                         │
│                   [Oracle] (read-only, sunset)                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

PHASE 4: Decommission Oracle
• Stop sync
• Shutdown Oracle
• Save $200K/year on licensing

TOOLS USED:
• AWS DMS (Database Migration Service)
• Debezium for CDC
• pg_loader for bulk transfer
• Custom validation scripts
```

### Migration Checklist

```
STRANGLER FIG MIGRATION CHECKLIST:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  BEFORE STARTING:                                               │
│  □ Map legacy system (APIs, databases, dependencies)           │
│  □ Identify natural seams                                      │
│  □ Prioritize: what to migrate first                          │
│  □ Set up facade/proxy infrastructure                         │
│  □ Establish monitoring & observability                       │
│  □ Define rollback procedures                                 │
│  □ Get stakeholder buy-in for timeline                        │
│                                                                  │
│  FOR EACH SERVICE EXTRACTION:                                  │
│  □ Build new service                                          │
│  □ Implement anti-corruption layer                            │
│  □ Set up data sync (if needed)                              │
│  □ Shadow traffic - compare results                           │
│  □ Fix discrepancies                                          │
│  □ Gradual rollout (5% → 100%)                               │
│  □ Monitor at each stage                                      │
│  □ Remove old code from legacy                                │
│  □ Update documentation                                       │
│                                                                  │
│  AFTER MIGRATION:                                              │
│  □ Decommission legacy system                                 │
│  □ Clean up sync infrastructure                               │
│  □ Remove facade routing (direct to new services)            │
│  □ Retrospective: what worked, what didn't                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. When to Use / Not Use

### When Strangler Fig is PERFECT ✅

```
IDEAL SCENARIOS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  ✅ Large legacy system (big-bang too risky)                   │
│  ✅ Business can't stop (need continuous delivery)            │
│  ✅ Clear API boundaries exist                                 │
│  ✅ Team needs time to learn new tech                         │
│  ✅ Gradual budget/resource allocation                        │
│  ✅ Need to prove value incrementally                         │
│  ✅ Multiple teams can work in parallel                       │
│  ✅ Legacy has decent documentation/understanding             │
│  ✅ New system can co-exist with old                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### When NOT to Use ❌

```
AVOID STRANGLER FIG WHEN:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  ❌ SMALL SYSTEM                                                │
│     └── If migration takes < 1 month, just rewrite             │
│     └── Overhead of facade not worth it                        │
│                                                                  │
│  ❌ COMPLETE TECH STACK CHANGE WITH NO API                     │
│     └── COBOL mainframe with no HTTP interface                │
│     └── Can't put facade in front                              │
│     └── May need big-bang with parallel run                    │
│                                                                  │
│  ❌ TIGHT COUPLING EVERYWHERE                                   │
│     └── No natural seams to extract                            │
│     └── Everything depends on everything                       │
│     └── Refactor first, then strangle                         │
│                                                                  │
│  ❌ REGULATORY "FREEZE"                                         │
│     └── Legacy must remain unchanged for compliance            │
│     └── Can't add facade layer                                 │
│                                                                  │
│  ❌ LEGACY IS TRULY DYING                                       │
│     └── Hardware failing, vendor support ended                 │
│     └── No time for gradual - need emergency replacement      │
│                                                                  │
│  ❌ TEAM WANTS TO LEARN BY BUILDING                            │
│     └── Strangling requires deep legacy understanding          │
│     └── If team doesn't know legacy, hard to match behavior    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Common Pitfalls

```
PITFALLS TO AVOID:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. NEVER FINISHING ("ETERNAL MIGRATION")                       │
│     ─────────────────────────────────────                       │
│     Problem: 80% migrated, last 20% never done                 │
│     Legacy becomes permanent "legacy corner"                   │
│                                                                  │
│     Solution: Set deadlines for legacy decommission            │
│     Make it painful to add features to legacy                  │
│                                                                  │
│  2. TWO SYSTEMS FOREVER                                        │
│     ────────────────────                                        │
│     Problem: Both systems maintained indefinitely              │
│     Double the maintenance cost                                │
│                                                                  │
│     Solution: Aggressive timeline for removal                  │
│     Track "lines of legacy code" metric                        │
│                                                                  │
│  3. FACADE BECOMES BOTTLENECK                                  │
│     ─────────────────────────                                   │
│     Problem: All traffic through single facade                 │
│     Facade goes down = everything down                         │
│                                                                  │
│     Solution: Scale facade horizontally                        │
│     Eventually remove facade (direct to services)              │
│                                                                  │
│  4. DATA SYNC NIGHTMARES                                        │
│     ──────────────────────                                      │
│     Problem: Data out of sync between old and new             │
│     Debugging where data lives is hell                         │
│                                                                  │
│     Solution: Clear ownership boundaries                       │
│     One source of truth at any time                            │
│                                                                  │
│  5. STARTING WITH THE HARDEST PART                             │
│     ─────────────────────────────                               │
│     Problem: Core domain extracted first, fails               │
│     Team loses confidence, project cancelled                   │
│                                                                  │
│     Solution: Start with easy wins (search, auth, email)       │
│     Build confidence, then tackle core                         │
│                                                                  │
│  6. NO ROLLBACK PLAN                                           │
│     ────────────────────                                        │
│     Problem: New service has bugs, can't go back              │
│                                                                  │
│     Solution: Always keep legacy running until confident       │
│     Rollback = flip routing back to legacy                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. Interview Questions

### Conceptual Questions

**Q: "What is the Strangler Fig Pattern?"**
> "It's an incremental migration strategy where you build new functionality around an existing legacy system, gradually routing more traffic to the new system until the legacy can be completely decommissioned. Named after strangler fig trees that grow around host trees. Key benefit: you can stop at any point - legacy still works as fallback."

**Q: "Why use Strangler Fig instead of a big-bang rewrite?"**
> "Big-bang rewrites have high failure rates because:
> 1. **Risk** - all-or-nothing, no fallback if it fails
> 2. **Time** - often takes years, business requirements change
> 3. **Value** - no ROI until complete, stakeholders lose patience
> 4. **Feature freeze** - can't add new features during rewrite
>
> Strangler Fig mitigates all of these - incremental value, always have fallback, can continue feature development."

**Q: "What's the first step in a Strangler Fig migration?"**
> "Put a facade (proxy or API gateway) in front of the legacy system. All traffic flows through it. Initially, 100% routes to legacy - no behavior change. This gives you control over routing to selectively send traffic to new services later."

**Q: "How do you decide what to migrate first?"**
> "Look for:
> 1. **Clear boundaries** - well-defined API, limited DB tables
> 2. **Low risk** - failure tolerable (search vs payments)
> 3. **High value** - pain points, performance issues
> 4. **Low coupling** - minimal dependencies on other parts
>
> Good first candidates: authentication, search, notifications, file processing. Avoid: core business logic, highly coupled modules."

### Implementation Questions

**Q: "How do you handle data during migration?"**
> "Three phases:
> 1. **Shared DB** - new service reads from legacy DB (temporary)
> 2. **Sync** - CDC streams data to new service's own DB
> 3. **Ownership** - new service owns data, legacy syncs if needed
>
> Use anti-corruption layer to translate between data models. Accept temporary duplication - it's part of the process."

**Q: "How do you route traffic between legacy and new?"**
> "Feature flags + routing rules:
> - **Percentage split** - 5% to new, 95% to legacy
> - **User segments** - internal users first, then beta, then all
> - **Geographic** - one region at a time
> - **Endpoint-based** - some APIs on new, others on legacy
>
> Consistent hashing ensures same user always hits same system (important for sessions, caching)."

**Q: "How do you validate the new system works correctly?"**
> "Three techniques:
> 1. **Shadow traffic** - send copies of requests to new system, compare responses (don't return to user)
> 2. **Parallel running** - both process, compare results, return one
> 3. **Canary release** - small % of real traffic, monitor for errors
>
> Look for: response differences, latency changes, error rates."

**Q: "What if the new service has bugs after rollout?"**
> "Rollback = change routing back to legacy. Takes seconds. That's the beauty of Strangler Fig:
> - Legacy is always there as fallback
> - No data loss (writes still went to legacy or were synced)
> - Set up monitoring to auto-rollback if error rates spike
>
> Circuit breaker pattern: if new service fails 5 times, automatically route to legacy."

### Scenario Questions

**Q: "You're migrating a 10-year-old PHP monolith to microservices. Walk me through your approach."**
> "1. **Assessment** - map the monolith: APIs, database schema, team knowledge
> 2. **Infrastructure** - deploy API gateway in front of PHP
> 3. **First service** - extract something simple like authentication
>    - Build new auth service
>    - Shadow traffic, compare results
>    - Gradual rollout: 5% → 25% → 100%
> 4. **Continue** - extract search, then user profiles, then orders
> 5. **Data** - start with shared DB, then sync, then separate
> 6. **Timeline** - probably 18-24 months for large monolith
> 7. **Decommission** - when all traffic on new, shutdown PHP
>
> Key: business continues throughout, features keep shipping, legacy is fallback."

**Q: "The migration is 80% done but the team wants to abandon the last 20%. What do you do?"**
> "This is 'eternal migration' - common pitfall. I'd:
> 1. **Quantify** - what's the cost of maintaining both systems?
> 2. **Prioritize** - what's blocking the last 20%? Can we simplify?
> 3. **Deadline** - set hard date for legacy shutdown
> 4. **Incentivize** - make it painful to add features to legacy
> 5. **Scope cut** - maybe some features aren't worth migrating, retire them
>
> If truly can't finish, at least clean up the boundary so legacy is isolated and maintainable."

**Q: "How do you handle sessions/authentication when users navigate between legacy and new parts?"**
> "Options:
> 1. **Shared session store** - both systems read/write to Redis
> 2. **JWT tokens** - stateless, both systems validate same token
> 3. **Session migration** - on first new-system hit, create JWT from PHP session
> 4. **Single sign-on** - external auth service both systems use
>
> Critical: users shouldn't notice they're crossing system boundaries. Same cookie/token works everywhere."

### Edge Cases & Traps

**Q: "What if legacy system has no API?"**
> "Create one. This is actually step zero:
> - Wrap legacy with thin API layer
> - Expose key operations as REST endpoints
> - This becomes the seam for extraction
> - Modern systems can now interact with legacy via API"

**Q: "What about background jobs and cron tasks?"**
> "Often forgotten! Strategy:
> 1. **Event-driven** - convert crons to event handlers
> 2. **Dual execution** - both systems run job, one is passive
> 3. **Migrate last** - jobs usually depend on data, migrate after data
>
> Use feature flags to control which system runs the job."

**Q: "Legacy database has stored procedures. How do you handle them?"**
> "Stored procedures are the hardest part:
> 1. **Replicate logic** - rewrite in application code
> 2. **Shadow test** - run both, compare results
> 3. **Gradual** - call app code for some operations, sproc for others
> 4. **Last resort** - keep sproc, call from new service
>
> Document all sproc behaviors first - they often have undocumented edge cases."

---

## Quick Reference

```
STRANGLER FIG CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  PHASES:                                                        │
│  1. TRANSFORM - Add facade in front of legacy                   │
│  2. CO-EXIST - Build new behind facade, route some traffic      │
│  3. ELIMINATE - All traffic to new, remove legacy               │
│                                                                  │
│  STRATEGIES:                                                    │
│  • API-level strangling (route entire endpoints)                │
│  • Asset capture (UI migration)                                 │
│  • Branch by abstraction (swap implementations)                 │
│  • Event interception (CDC from legacy DB)                      │
│                                                                  │
│  TRAFFIC ROUTING:                                               │
│  • Percentage split (5% → 100%)                                │
│  • User segments (internal → beta → all)                        │
│  • Geographic (one region at a time)                            │
│  • Feature flags for fine control                               │
│                                                                  │
│  DATA MIGRATION:                                                │
│  1. Shared DB (temporary)                                       │
│  2. Sync via CDC                                                │
│  3. New service owns data                                       │
│                                                                  │
│  VALIDATION:                                                    │
│  • Shadow traffic - compare without affecting users             │
│  • Parallel running - both process, compare                     │
│  • Canary - small % real traffic                               │
│                                                                  │
│  ROLLBACK:                                                      │
│  • Change routing back to legacy (seconds)                      │
│  • Circuit breaker for auto-rollback                            │
│  • Legacy always available as fallback                          │
│                                                                  │
│  PITFALLS:                                                      │
│  ✗ Never finishing (eternal migration)                          │
│  ✗ Maintaining two systems forever                              │
│  ✗ Facade becoming bottleneck                                   │
│  ✗ Data sync nightmares                                         │
│  ✗ Starting with hardest part                                   │
│                                                                  │
│  WHEN TO USE:                                                   │
│  ✓ Large legacy, big-bang too risky                            │
│  ✓ Business can't stop                                          │
│  ✓ Clear API boundaries exist                                   │
│                                                                  │
│  WHEN NOT TO USE:                                               │
│  ✗ Small system (just rewrite)                                  │
│  ✗ No API to strangle                                           │
│  ✗ Tight coupling everywhere                                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Final "Wow" Statement for Interviews

> "The Strangler Fig pattern is about **managing risk** in legacy migration. The name comes from strangler fig trees that grow around host trees - we do the same with software. Instead of a risky big-bang rewrite where everything might fail, we build new services around the legacy system. We put a facade in front, extract one capability at a time, validate with shadow traffic, then gradually shift real traffic. The legacy system is always there as a fallback - if the new service has bugs, we flip traffic back in seconds. I've used this to migrate a 15-year-old monolith over 18 months with zero downtime. The business kept shipping features throughout, and we decommissioned the legacy system when we were confident. The key insight is that **you don't need to finish to get value** - every extracted service is an improvement, and you can stop anytime."

---

*Guide created following Martin Fowler's original Strangler Fig Application pattern and battle-tested migration experiences from companies like Amazon, Netflix, and Shopify.*


