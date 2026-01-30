# 🎯 Event-Driven Architecture Complete Guide

> A comprehensive guide to event sourcing, CQRS, message brokers, and building reactive, decoupled systems.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Event-driven architecture uses events as the primary way to communicate state changes between services, enabling loose coupling, scalability, and real-time reactivity."

### The 6 Key Concepts (Remember These!)
```
1. EVENT             → Immutable fact that something happened ("OrderPlaced")
2. EVENT SOURCING    → Store events, not state - rebuild state from events
3. CQRS              → Separate read models from write models
4. MESSAGE BROKER    → Middleware that routes events (Kafka, RabbitMQ)
5. EVENTUAL CONSIST. → Systems sync over time, not immediately
6. BACKPRESSURE      → Slow down instead of crash under heavy load
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Event sourcing"** | "We use event sourcing for complete audit trail" |
| **"CQRS"** | "CQRS lets us optimize reads and writes separately" |
| **"Eventual consistency"** | "The system is eventually consistent" |
| **"Idempotent consumer"** | "Consumers must be idempotent for safe retries" |
| **"Saga pattern"** | "Distributed transactions use choreography with sagas" |
| **"Dead letter queue"** | "Failed events go to dead letter queue for analysis" |
| **"Backpressure"** | "We apply backpressure to prevent system crashes under load" |
| **"Load shedding"** | "Under extreme load, we shed non-critical requests" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Kafka retention | **7 days default** | Can replay events |
| Consumer lag | **< 1000** | Alert if higher |
| Message size | **< 1 MB** | Large = slow |
| Partitions | **#consumers × 2** | For parallelism |

### Event vs Command vs Query
| Type | Purpose | Example | Naming |
|------|---------|---------|--------|
| **Command** | Request action | `CreateOrder` | Imperative |
| **Event** | Fact happened | `OrderCreated` | Past tense |
| **Query** | Read data | `GetOrder` | Question |

### The "Wow" Statement (Memorize This!)
> "Traditional systems store current state and lose history. Event sourcing flips this - we store every event as immutable facts and derive current state by replaying them. This gives us complete audit trail, time travel debugging, and the ability to rebuild any view of data. Combined with CQRS, we can optimize reads and writes independently."

### Quick Architecture Drawing (Draw This!)
```
Command ──▶ Write Model ──▶ Event Store
                               │
                          (publish)
                               ▼
                        Message Broker
                         /    |    \
                        ▼     ▼     ▼
                    Read    Read   Notification
                    Model   Model   Service
```

### Interview Rapid Fire (Practice These!)

**Q: "What is event-driven architecture?"**
> "Services communicate via events - immutable facts. Enables loose coupling and scalability."

**Q: "What is event sourcing?"**
> "Store events instead of state. Rebuild state by replaying events. Complete audit trail."

**Q: "What is CQRS?"**
> "Separate read and write models. Optimize each independently. Often paired with event sourcing."

**Q: "Kafka vs RabbitMQ?"**
> "Kafka: high throughput, retention, replay. RabbitMQ: flexible routing, simpler, push-based."

**Q: "What is backpressure?"**
> "When consumer can't keep up, slow down producer instead of crashing. Better slow than dead."

**Q: "What's the hardest part?"**
> "Schema evolution - changing event structure without breaking consumers."

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "What is Event-Driven Architecture?"

**Junior Answer:**
> "It's when services send messages to each other using events."

**Senior Answer:**
> "Event-driven architecture is a design pattern where services communicate through events - immutable records of things that happened. Instead of Service A directly calling Service B, A publishes an event to a message broker, and any interested service can subscribe.

This gives us three key benefits:
1. **Loose coupling** - Publishers don't know about subscribers
2. **Scalability** - Add consumers without changing producers
3. **Resilience** - If a service is down, events are buffered

The pattern comes in two flavors: simple pub/sub for notifications, and event sourcing where events become the source of truth."

### When Asked: "Explain Event Sourcing"

**Junior Answer:**
> "It's storing events instead of data."

**Senior Answer:**
> "In traditional systems, we store current state - a user has name 'John'. In event sourcing, we store the events that led to that state: 'UserCreated', 'NameChanged to John'.

The current state is derived by replaying events. This seems complex but gives us:
1. **Complete audit trail** - Every change is recorded
2. **Time travel** - Rebuild state at any point in time
3. **Debugging** - See exactly what happened
4. **New projections** - Build new read models from existing events

The trade-off is complexity. You need event versioning, snapshot optimization for performance, and teams must think in events instead of CRUD."

### Follow-up Questions to Expect:

| They Ask | You Answer |
|----------|------------|
| "How do you handle event schema changes?" | "Versioning. Either include version in event, use upcasters to transform old events, or make changes backward compatible." |
| "Isn't replaying slow?" | "Snapshots. Store state periodically, replay only events after snapshot." |
| "How do you query event sourced data?" | "CQRS. Create read-optimized projections from events." |
| "What about GDPR deletion?" | "Crypto-shredding. Encrypt personal data, delete key to 'forget'." |

---

## 📚 Table of Contents

1. [Core Concepts](#1-core-concepts)
2. [Event Sourcing Deep Dive](#2-event-sourcing-deep-dive)
3. [CQRS Pattern](#3-cqrs-pattern)
4. [Message Brokers](#4-message-brokers)
5. [Implementation Patterns](#5-implementation-patterns)
6. [Real-World Scenarios](#6-real-world-scenarios)
7. [Common Pitfalls](#7-common-pitfalls)
8. [Interview Questions](#8-interview-questions)

---

## 1. Core Concepts

### What is an Event?

An **event** is an immutable record of something that happened in the past.

```typescript
// Event structure
interface Event {
  id: string;              // Unique event ID
  type: string;            // Event type (OrderCreated)
  aggregateId: string;     // Entity this event belongs to
  timestamp: Date;         // When it happened
  version: number;         // For ordering
  payload: object;         // Event data
  metadata: object;        // Context (userId, correlationId)
}

// Example events
const orderCreated: Event = {
  id: "evt-123",
  type: "OrderCreated",
  aggregateId: "order-456",
  timestamp: new Date(),
  version: 1,
  payload: {
    customerId: "cust-789",
    items: [{ productId: "prod-1", quantity: 2 }],
    total: 99.99
  },
  metadata: {
    userId: "user-123",
    correlationId: "req-abc",
    causationId: "cmd-xyz"
  }
};
```

### Event Naming Conventions

```
✅ Good Event Names (Past Tense, Business Language)
- OrderPlaced
- PaymentReceived
- UserEmailVerified
- ShipmentDispatched

❌ Bad Event Names
- CreateOrder (command, not event)
- OrderData (not descriptive)
- UpdateUserEmail (CRUD thinking)
```

### Types of Events

```
1. DOMAIN EVENTS
   └── Business facts: OrderPlaced, PaymentFailed
   └── Rich in business meaning
   └── Published by aggregates

2. INTEGRATION EVENTS
   └── Cross-service communication
   └── May be transformed from domain events
   └── Include only necessary data

3. SYSTEM EVENTS
   └── Technical: ServiceStarted, HealthCheckFailed
   └── Infrastructure concerns
```

### Event-Driven vs Request-Driven

| Aspect | Request-Driven | Event-Driven |
|--------|----------------|--------------|
| **Coupling** | Tight (knows target) | Loose (publish to topic) |
| **Communication** | Synchronous | Asynchronous |
| **Failure** | Cascades | Isolated |
| **Scaling** | Scale together | Scale independently |
| **Debugging** | Simpler | More complex |

---

## 2. Event Sourcing Deep Dive

### Traditional vs Event Sourcing

```
TRADITIONAL (State-Based):
┌─────────────────────────────────┐
│ Orders Table                    │
├─────────────────────────────────┤
│ id: 123                         │
│ status: "shipped"               │
│ total: $99.99                   │
│ updated_at: 2024-01-15          │
└─────────────────────────────────┘
→ Only current state, history lost

EVENT SOURCING:
┌─────────────────────────────────┐
│ Event Store                     │
├─────────────────────────────────┤
│ 1. OrderCreated      2024-01-10 │
│ 2. ItemAdded         2024-01-10 │
│ 3. PaymentReceived   2024-01-12 │
│ 4. OrderShipped      2024-01-15 │
└─────────────────────────────────┘
→ Complete history, derive any state
```

### Event Store Implementation

```typescript
// Event Store interface
interface EventStore {
  // Append events to a stream
  append(
    streamId: string, 
    events: Event[], 
    expectedVersion: number
  ): Promise<void>;
  
  // Read events from a stream
  read(
    streamId: string, 
    fromVersion?: number
  ): Promise<Event[]>;
  
  // Subscribe to new events
  subscribe(
    streamId: string, 
    handler: (event: Event) => void
  ): Subscription;
}

// Simple in-memory implementation
class InMemoryEventStore implements EventStore {
  private streams: Map<string, Event[]> = new Map();

  async append(
    streamId: string, 
    events: Event[], 
    expectedVersion: number
  ): Promise<void> {
    const stream = this.streams.get(streamId) || [];
    
    // Optimistic concurrency check
    if (stream.length !== expectedVersion) {
      throw new ConcurrencyError(
        `Expected version ${expectedVersion}, but found ${stream.length}`
      );
    }
    
    // Append with version numbers
    const versionedEvents = events.map((e, i) => ({
      ...e,
      version: expectedVersion + i + 1
    }));
    
    this.streams.set(streamId, [...stream, ...versionedEvents]);
  }

  async read(streamId: string, fromVersion = 0): Promise<Event[]> {
    const stream = this.streams.get(streamId) || [];
    return stream.filter(e => e.version > fromVersion);
  }
}
```

### Aggregate with Event Sourcing

```typescript
// Order Aggregate
class Order {
  private id: string;
  private status: string;
  private items: OrderItem[] = [];
  private total: number = 0;
  
  // Uncommitted events (not yet persisted)
  private uncommittedEvents: Event[] = [];
  
  // Current version (for concurrency)
  private version: number = 0;

  // COMMAND: Place order
  placeOrder(customerId: string, items: OrderItem[]): void {
    // Business validation
    if (items.length === 0) {
      throw new Error("Order must have at least one item");
    }
    
    // Raise event (don't mutate state directly!)
    this.raise({
      type: "OrderPlaced",
      payload: { customerId, items, total: this.calculateTotal(items) }
    });
  }

  // COMMAND: Ship order
  ship(trackingNumber: string): void {
    if (this.status !== "paid") {
      throw new Error("Can only ship paid orders");
    }
    
    this.raise({
      type: "OrderShipped",
      payload: { trackingNumber, shippedAt: new Date() }
    });
  }

  // Apply event to state (called during replay too)
  private apply(event: Event): void {
    switch (event.type) {
      case "OrderPlaced":
        this.id = event.aggregateId;
        this.status = "placed";
        this.items = event.payload.items;
        this.total = event.payload.total;
        break;
        
      case "PaymentReceived":
        this.status = "paid";
        break;
        
      case "OrderShipped":
        this.status = "shipped";
        break;
    }
    this.version++;
  }

  // Raise event (apply + store for persistence)
  private raise(eventData: Partial<Event>): void {
    const event: Event = {
      ...eventData,
      id: generateId(),
      aggregateId: this.id,
      timestamp: new Date(),
      version: this.version + 1
    } as Event;
    
    this.apply(event);
    this.uncommittedEvents.push(event);
  }

  // Reconstitute from events (called by repository)
  static fromEvents(events: Event[]): Order {
    const order = new Order();
    events.forEach(e => order.apply(e));
    return order;
  }

  getUncommittedEvents(): Event[] {
    return this.uncommittedEvents;
  }

  clearUncommittedEvents(): void {
    this.uncommittedEvents = [];
  }
}
```

### Repository Pattern with Event Sourcing

```typescript
class OrderRepository {
  constructor(private eventStore: EventStore) {}

  async save(order: Order): Promise<void> {
    const events = order.getUncommittedEvents();
    if (events.length === 0) return;

    await this.eventStore.append(
      `order-${order.id}`,
      events,
      order.version - events.length // Expected version before new events
    );

    order.clearUncommittedEvents();
  }

  async getById(orderId: string): Promise<Order | null> {
    const events = await this.eventStore.read(`order-${orderId}`);
    if (events.length === 0) return null;
    
    return Order.fromEvents(events);
  }
}

// Usage
const orderRepo = new OrderRepository(eventStore);

// Create order
const order = new Order();
order.placeOrder("cust-123", [{ productId: "prod-1", quantity: 2 }]);
await orderRepo.save(order);

// Later: load and modify
const loadedOrder = await orderRepo.getById("order-123");
loadedOrder.ship("TRACK-456");
await orderRepo.save(loadedOrder);
```

### Snapshots for Performance

```typescript
interface Snapshot {
  aggregateId: string;
  version: number;
  state: any;
  createdAt: Date;
}

class OrderRepository {
  private snapshotInterval = 100; // Snapshot every 100 events

  async getById(orderId: string): Promise<Order | null> {
    // Try to load from snapshot first
    const snapshot = await this.loadSnapshot(orderId);
    
    let events: Event[];
    let order: Order;
    
    if (snapshot) {
      // Load events after snapshot
      events = await this.eventStore.read(
        `order-${orderId}`, 
        snapshot.version
      );
      order = Order.fromSnapshot(snapshot.state);
      events.forEach(e => order.apply(e));
    } else {
      // Load all events
      events = await this.eventStore.read(`order-${orderId}`);
      if (events.length === 0) return null;
      order = Order.fromEvents(events);
    }

    // Create snapshot if needed
    if (order.version % this.snapshotInterval === 0) {
      await this.saveSnapshot(order);
    }

    return order;
  }

  private async saveSnapshot(order: Order): Promise<void> {
    const snapshot: Snapshot = {
      aggregateId: order.id,
      version: order.version,
      state: order.toSnapshot(),
      createdAt: new Date()
    };
    await this.snapshotStore.save(snapshot);
  }
}
```

---

## 3. CQRS Pattern

### What is CQRS?

**Command Query Responsibility Segregation** - separate models for reading and writing.

```
TRADITIONAL:
┌─────────────────────────────────┐
│         Single Model            │
│  (Handles both reads & writes)  │
└─────────────────────────────────┘

CQRS:
┌─────────────────┐    ┌─────────────────┐
│  Write Model    │    │   Read Model    │
│  (Commands)     │    │   (Queries)     │
│  Normalized     │    │  Denormalized   │
│  Consistent     │    │  Eventually     │
└─────────────────┘    └─────────────────┘
```

### Why CQRS?

| Problem | CQRS Solution |
|---------|---------------|
| Reads need joins across tables | Denormalized read models |
| Writes need validation | Focused write model |
| Different scaling needs | Scale reads/writes independently |
| Complex queries slow down writes | Separate databases |

### CQRS Implementation

```typescript
// WRITE SIDE (Commands)
interface CreateOrderCommand {
  type: "CreateOrder";
  customerId: string;
  items: OrderItem[];
}

class OrderCommandHandler {
  constructor(
    private orderRepo: OrderRepository,
    private eventBus: EventBus
  ) {}

  async handle(command: CreateOrderCommand): Promise<string> {
    // Create aggregate
    const order = new Order();
    order.placeOrder(command.customerId, command.items);
    
    // Save (appends events)
    await this.orderRepo.save(order);
    
    // Publish events for read model update
    for (const event of order.getUncommittedEvents()) {
      await this.eventBus.publish(event);
    }
    
    return order.id;
  }
}

// READ SIDE (Queries)
interface OrderView {
  id: string;
  customerName: string;    // Denormalized!
  items: Array<{
    productName: string;   // Denormalized!
    quantity: number;
    price: number;
  }>;
  status: string;
  total: number;
  createdAt: Date;
}

class OrderProjection {
  constructor(private readDb: Database) {}

  // Event handlers update read model
  async onOrderPlaced(event: OrderPlacedEvent): Promise<void> {
    const customer = await this.getCustomer(event.payload.customerId);
    const items = await this.enrichItems(event.payload.items);
    
    await this.readDb.orders.insert({
      id: event.aggregateId,
      customerName: customer.name,
      items: items,
      status: "placed",
      total: event.payload.total,
      createdAt: event.timestamp
    });
  }

  async onOrderShipped(event: OrderShippedEvent): Promise<void> {
    await this.readDb.orders.update(
      { id: event.aggregateId },
      { status: "shipped", shippedAt: event.payload.shippedAt }
    );
  }
}

// Query handler
class OrderQueryHandler {
  constructor(private readDb: Database) {}

  async getOrder(orderId: string): Promise<OrderView> {
    // Simple read from denormalized store
    return this.readDb.orders.findOne({ id: orderId });
  }

  async getCustomerOrders(customerId: string): Promise<OrderView[]> {
    // No joins needed!
    return this.readDb.orders.find({ customerId });
  }

  async getOrdersForDashboard(): Promise<DashboardView> {
    // Read model optimized for this exact query
    return this.readDb.dashboardView.findOne({});
  }
}
```

### Multiple Read Models

```typescript
// Same events, different projections

// Projection 1: Customer's order history
class CustomerOrdersProjection {
  async onOrderPlaced(event: OrderPlacedEvent): Promise<void> {
    await this.db.customerOrders.upsert({
      customerId: event.payload.customerId,
      $push: { orders: { id: event.aggregateId, total: event.payload.total } }
    });
  }
}

// Projection 2: Analytics dashboard
class AnalyticsProjection {
  async onOrderPlaced(event: OrderPlacedEvent): Promise<void> {
    await this.db.dailyStats.upsert({
      date: toDateString(event.timestamp),
      $inc: { orderCount: 1, revenue: event.payload.total }
    });
  }
}

// Projection 3: Search index
class SearchProjection {
  async onOrderPlaced(event: OrderPlacedEvent): Promise<void> {
    await this.elasticsearch.index({
      index: 'orders',
      id: event.aggregateId,
      body: {
        customerId: event.payload.customerId,
        items: event.payload.items.map(i => i.productName),
        total: event.payload.total
      }
    });
  }
}
```

### CQRS Without Event Sourcing

```typescript
// You can use CQRS without event sourcing!

// Write model (traditional database)
class OrderService {
  async createOrder(data: CreateOrderData): Promise<Order> {
    const order = await this.db.orders.create(data);
    
    // Publish event for read model sync
    await this.eventBus.publish({
      type: "OrderCreated",
      payload: order
    });
    
    return order;
  }
}

// Read model (separate, denormalized)
class OrderReadService {
  async onOrderCreated(event: OrderCreatedEvent): Promise<void> {
    // Update denormalized read store
    await this.readDb.orderViews.insert(
      this.enrichOrder(event.payload)
    );
  }
}
```

---

## 4. Message Brokers

### Comparison: Kafka vs RabbitMQ vs Others

| Feature | Kafka | RabbitMQ | AWS SQS | Redis Streams |
|---------|-------|----------|---------|---------------|
| **Model** | Log-based | Queue-based | Queue | Log-based |
| **Throughput** | Very High (M/s) | High (K/s) | Medium | High |
| **Retention** | Configurable | Until consumed | 14 days | Configurable |
| **Replay** | ✅ Yes | ❌ No | ❌ No | ✅ Yes |
| **Ordering** | Per partition | Per queue | FIFO option | Per stream |
| **Push/Pull** | Pull | Push | Pull | Both |
| **Use Case** | Event streaming | Task queues | Simple queuing | Real-time |

### When to Use What

```
KAFKA - Choose when you need:
├── High throughput (millions/sec)
├── Event replay / audit trail
├── Multiple consumers for same event
├── Stream processing
└── Long retention

RABBITMQ - Choose when you need:
├── Complex routing (topic, headers)
├── Flexible patterns (pub/sub, work queue)
├── Message acknowledgment
├── Lower latency
└── Simpler operations

AWS SQS - Choose when you need:
├── Managed service (no ops)
├── Simple point-to-point
├── AWS integration
└── Pay-per-use

REDIS STREAMS - Choose when you need:
├── Real-time with low latency
├── Already using Redis
├── Simpler than Kafka
└── Consumer groups
```

### Kafka Deep Dive

```typescript
// Kafka concepts
// TOPIC    → Category of messages (like "orders")
// PARTITION → Ordered log within topic (parallelism)
// OFFSET   → Position in partition (for replay)
// CONSUMER GROUP → Load balancing across consumers

import { Kafka } from 'kafkajs';

const kafka = new Kafka({
  clientId: 'order-service',
  brokers: ['kafka1:9092', 'kafka2:9092']
});

// Producer
const producer = kafka.producer();

async function publishOrderCreated(order: Order): Promise<void> {
  await producer.send({
    topic: 'orders',
    messages: [{
      key: order.customerId,  // Same customer → same partition → ordered
      value: JSON.stringify({
        type: 'OrderCreated',
        payload: order,
        timestamp: Date.now()
      }),
      headers: {
        'correlation-id': order.correlationId
      }
    }]
  });
}

// Consumer
const consumer = kafka.consumer({ groupId: 'order-processor' });

async function startConsumer(): Promise<void> {
  await consumer.connect();
  await consumer.subscribe({ topic: 'orders', fromBeginning: false });
  
  await consumer.run({
    eachMessage: async ({ topic, partition, message }) => {
      const event = JSON.parse(message.value.toString());
      
      console.log({
        topic,
        partition,
        offset: message.offset,
        event: event.type
      });
      
      await processEvent(event);
    }
  });
}

// Consumer with manual commits (for exactly-once semantics)
await consumer.run({
  autoCommit: false,
  eachMessage: async ({ topic, partition, message }) => {
    try {
      await processEvent(JSON.parse(message.value.toString()));
      
      // Commit only after successful processing
      await consumer.commitOffsets([{
        topic,
        partition,
        offset: (parseInt(message.offset) + 1).toString()
      }]);
    } catch (error) {
      // Don't commit - will retry on restart
      console.error('Processing failed:', error);
    }
  }
});
```

### RabbitMQ Patterns

```typescript
import amqp from 'amqplib';

// Setup
const connection = await amqp.connect('amqp://localhost');
const channel = await connection.createChannel();

// PATTERN 1: Direct Exchange (point-to-point)
await channel.assertExchange('orders', 'direct', { durable: true });
await channel.assertQueue('order-processor', { durable: true });
await channel.bindQueue('order-processor', 'orders', 'order.created');

channel.publish('orders', 'order.created', Buffer.from(JSON.stringify(order)));

// PATTERN 2: Fanout Exchange (broadcast)
await channel.assertExchange('notifications', 'fanout', { durable: true });

// All bound queues receive the message
channel.publish('notifications', '', Buffer.from(JSON.stringify(notification)));

// PATTERN 3: Topic Exchange (pattern matching)
await channel.assertExchange('events', 'topic', { durable: true });

// Bindings with wildcards
await channel.bindQueue('analytics', 'events', 'order.*');      // order.created, order.shipped
await channel.bindQueue('shipping', 'events', 'order.paid');    // only order.paid
await channel.bindQueue('audit', 'events', '#');                // everything

channel.publish('events', 'order.created', Buffer.from(JSON.stringify(event)));

// Consumer with acknowledgment
channel.consume('order-processor', async (msg) => {
  try {
    const event = JSON.parse(msg.content.toString());
    await processEvent(event);
    
    channel.ack(msg);  // Success - remove from queue
  } catch (error) {
    if (isRetryable(error)) {
      channel.nack(msg, false, true);  // Retry - requeue
    } else {
      channel.nack(msg, false, false); // Dead letter
    }
  }
});

// Dead Letter Queue setup
await channel.assertQueue('order-processor', {
  durable: true,
  deadLetterExchange: 'dead-letters',
  deadLetterRoutingKey: 'order-processor.dead'
});
```

---

## 5. Implementation Patterns

### Pattern 1: Idempotent Consumers

```typescript
// PROBLEM: Messages can be delivered multiple times
// SOLUTION: Make processing idempotent

class IdempotentConsumer {
  constructor(
    private processedStore: ProcessedEventStore,
    private handler: EventHandler
  ) {}

  async handle(event: Event): Promise<void> {
    // Check if already processed
    const processed = await this.processedStore.isProcessed(event.id);
    if (processed) {
      console.log(`Event ${event.id} already processed, skipping`);
      return;
    }

    // Process event
    await this.handler.handle(event);

    // Mark as processed
    await this.processedStore.markProcessed(event.id);
  }
}

// Alternative: Use database constraints
async function handleOrderCreated(event: OrderCreatedEvent): Promise<void> {
  try {
    // eventId as unique constraint prevents duplicates
    await db.orderProjection.insert({
      id: event.aggregateId,
      eventId: event.id,  // UNIQUE constraint
      ...event.payload
    });
  } catch (error) {
    if (error.code === 'DUPLICATE_KEY') {
      // Already processed - idempotent!
      return;
    }
    throw error;
  }
}
```

### Pattern 2: Outbox Pattern (Transactional Messaging)

```typescript
// PROBLEM: Database write + event publish must be atomic
// SOLUTION: Store events in database, publish separately

// Step 1: Write to database + outbox in same transaction
async function createOrder(data: CreateOrderData): Promise<Order> {
  return await db.transaction(async (tx) => {
    // Save order
    const order = await tx.orders.insert(data);
    
    // Save to outbox (same transaction!)
    await tx.outbox.insert({
      id: generateId(),
      aggregateType: 'Order',
      aggregateId: order.id,
      eventType: 'OrderCreated',
      payload: JSON.stringify(order),
      createdAt: new Date(),
      publishedAt: null
    });
    
    return order;
  });
}

// Step 2: Background job publishes from outbox
class OutboxPublisher {
  async run(): Promise<void> {
    while (true) {
      const unpublished = await db.outbox.find({
        publishedAt: null,
        createdAt: { $lt: new Date(Date.now() - 1000) }  // 1 sec delay
      });

      for (const event of unpublished) {
        try {
          await messageBroker.publish(event.eventType, event.payload);
          await db.outbox.update(
            { id: event.id },
            { publishedAt: new Date() }
          );
        } catch (error) {
          console.error(`Failed to publish ${event.id}:`, error);
        }
      }

      await sleep(100);  // Poll interval
    }
  }
}
```

### Pattern 3: Saga Pattern (Distributed Transactions)

```typescript
// PROBLEM: Multi-service transactions
// SOLUTION: Saga with compensating actions

// CHOREOGRAPHY: Services react to events
// Order Service → OrderCreated
// Payment Service listens → PaymentProcessed OR PaymentFailed
// Inventory Service listens → InventoryReserved OR InventoryFailed
// Order Service listens → Updates order status or rolls back

// ORCHESTRATION: Central coordinator
class OrderSaga {
  private steps = [
    { service: 'payment', action: 'process', compensation: 'refund' },
    { service: 'inventory', action: 'reserve', compensation: 'release' },
    { service: 'shipping', action: 'schedule', compensation: 'cancel' }
  ];

  async execute(orderId: string): Promise<void> {
    const completedSteps: string[] = [];

    for (const step of this.steps) {
      try {
        await this.executeStep(step, orderId);
        completedSteps.push(step.service);
      } catch (error) {
        // Compensate completed steps in reverse order
        await this.compensate(completedSteps.reverse(), orderId);
        throw new SagaFailedError(step.service, error);
      }
    }
  }

  private async compensate(
    services: string[], 
    orderId: string
  ): Promise<void> {
    for (const service of services) {
      const step = this.steps.find(s => s.service === service);
      try {
        await this.executeCompensation(step, orderId);
      } catch (error) {
        // Log and continue - best effort
        console.error(`Compensation failed for ${service}:`, error);
      }
    }
  }
}
```

### Pattern 4: Event Versioning

```typescript
// PROBLEM: Event schemas evolve over time
// SOLUTION: Version events and use upcasters

// Version 1
interface OrderCreatedV1 {
  type: 'OrderCreated';
  version: 1;
  payload: {
    customerId: string;
    items: string[];  // Just product IDs
    total: number;
  };
}

// Version 2 (breaking change)
interface OrderCreatedV2 {
  type: 'OrderCreated';
  version: 2;
  payload: {
    customerId: string;
    items: Array<{     // Now includes quantity
      productId: string;
      quantity: number;
      price: number;
    }>;
    total: number;
    currency: string;   // New field
  };
}

// Upcaster transforms old events to new format
class OrderCreatedUpcaster {
  canUpcast(event: Event): boolean {
    return event.type === 'OrderCreated' && event.version === 1;
  }

  upcast(event: OrderCreatedV1): OrderCreatedV2 {
    return {
      ...event,
      version: 2,
      payload: {
        customerId: event.payload.customerId,
        items: event.payload.items.map(productId => ({
          productId,
          quantity: 1,     // Default
          price: 0         // Unknown
        })),
        total: event.payload.total,
        currency: 'USD'    // Default
      }
    };
  }
}

// Event store applies upcasters when reading
class EventStoreWithUpcasting {
  constructor(
    private store: EventStore,
    private upcasters: Upcaster[]
  ) {}

  async read(streamId: string): Promise<Event[]> {
    const events = await this.store.read(streamId);
    return events.map(e => this.upcast(e));
  }

  private upcast(event: Event): Event {
    let current = event;
    for (const upcaster of this.upcasters) {
      if (upcaster.canUpcast(current)) {
        current = upcaster.upcast(current);
      }
    }
    return current;
  }
}
```

### Pattern 5: Backpressure (Prevent Crash Under Load)

```typescript
// PROBLEM: Consumer can't keep up with producer → memory fills → crash
// SOLUTION: Slow down instead of crashing

// ============================================
// STRATEGY 1: RATE LIMITING (Slow Responses)
// ============================================

class RateLimitedConsumer {
  private processing = 0;
  private maxConcurrent = 100;  // Don't process more than 100 at a time
  private queue: Message[] = [];

  async onMessage(message: Message): Promise<void> {
    if (this.processing >= this.maxConcurrent) {
      // BACKPRESSURE: Queue instead of process immediately
      this.queue.push(message);
      console.log(`Backpressure applied. Queue size: ${this.queue.length}`);
      return;
    }

    this.processing++;
    try {
      await this.process(message);
    } finally {
      this.processing--;
      this.processQueued();  // Process queued messages
    }
  }
}

// ============================================
// STRATEGY 2: LOAD SHEDDING (Drop Non-Critical)
// ============================================

class LoadSheddingHandler {
  private highWaterMark = 1000;  // Start shedding at 1000 pending
  private lowWaterMark = 500;   // Stop shedding at 500 pending
  private shedding = false;

  async onMessage(message: Message): Promise<void> {
    const pending = await this.getPendingCount();

    // Start shedding when overloaded
    if (pending > this.highWaterMark) {
      this.shedding = true;
    } else if (pending < this.lowWaterMark) {
      this.shedding = false;
    }

    if (this.shedding && !this.isCritical(message)) {
      // DROP non-critical messages
      console.log(`Shedding message: ${message.type}`);
      await this.sendToDeadLetter(message, 'load_shed');
      return;
    }

    await this.process(message);
  }

  private isCritical(message: Message): boolean {
    // Always process payments, critical events
    return ['PaymentReceived', 'OrderCancelled', 'EmergencyAlert']
      .includes(message.type);
  }
}

// ============================================
// STRATEGY 3: SLOW DOWN PRODUCER (Tell upstream)
// ============================================

// Kafka: Consumer automatically applies backpressure via pull model
// Producer can't push faster than consumer pulls

// RabbitMQ: Prefetch limits
channel.prefetch(100);  // Only get 100 unacked messages at a time
// If consumer is slow, RabbitMQ holds messages, slowing producer

// HTTP API: Return 429 or slow response
class BackpressureMiddleware {
  private activeRequests = 0;
  private maxActive = 500;
  private responseDelay = 0;

  async handle(req: Request, res: Response, next: NextFunction) {
    this.activeRequests++;

    // STRATEGY A: Reject with 429
    if (this.activeRequests > this.maxActive) {
      this.activeRequests--;
      return res.status(429).json({
        error: 'Too Many Requests',
        retryAfter: 5,
        message: 'Server under heavy load, please retry'
      });
    }

    // STRATEGY B: Slow down responses (graceful degradation)
    if (this.activeRequests > this.maxActive * 0.8) {
      // 80% capacity: start slowing down
      this.responseDelay = Math.min(this.responseDelay + 100, 2000);
      await sleep(this.responseDelay);
    } else {
      this.responseDelay = Math.max(this.responseDelay - 50, 0);
    }

    try {
      await next();
    } finally {
      this.activeRequests--;
    }
  }
}

// ============================================
// STRATEGY 4: CIRCUIT BREAKER (Stop calling failing service)
// ============================================

class CircuitBreaker {
  private failures = 0;
  private threshold = 5;
  private state: 'closed' | 'open' | 'half-open' = 'closed';
  private openTime?: Date;
  private timeout = 30000;  // 30 seconds

  async call<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === 'open') {
      if (Date.now() - this.openTime!.getTime() > this.timeout) {
        this.state = 'half-open';  // Try one request
      } else {
        throw new Error('Circuit is OPEN - service unavailable');
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

  private onSuccess(): void {
    this.failures = 0;
    this.state = 'closed';
  }

  private onFailure(): void {
    this.failures++;
    if (this.failures >= this.threshold) {
      this.state = 'open';
      this.openTime = new Date();
      console.log('Circuit OPENED - stopping requests to prevent cascade');
    }
  }
}

// ============================================
// STRATEGY 5: ADAPTIVE THROTTLING
// ============================================

class AdaptiveThrottler {
  private targetLatency = 100;  // Target 100ms response time
  private currentRate = 1000;   // Requests per second
  private minRate = 10;
  private maxRate = 10000;

  async adjustRate(actualLatency: number): Promise<void> {
    if (actualLatency > this.targetLatency * 2) {
      // Way too slow - cut rate in half
      this.currentRate = Math.max(this.currentRate * 0.5, this.minRate);
    } else if (actualLatency > this.targetLatency) {
      // Slightly slow - reduce by 10%
      this.currentRate = Math.max(this.currentRate * 0.9, this.minRate);
    } else if (actualLatency < this.targetLatency * 0.5) {
      // Fast - increase by 10%
      this.currentRate = Math.min(this.currentRate * 1.1, this.maxRate);
    }

    console.log(`Adjusted rate to ${this.currentRate}/s (latency: ${actualLatency}ms)`);
  }
}
```

### Backpressure Quick Reference

| Strategy | When to Use | Implementation |
|----------|-------------|----------------|
| **Rate Limiting** | Control processing speed | Queue + max concurrent |
| **Load Shedding** | Extreme overload | Drop non-critical messages |
| **429 Response** | HTTP APIs | Return "Too Many Requests" |
| **Slow Responses** | Gradual degradation | Add delay under load |
| **Circuit Breaker** | Failing downstream | Stop calling + recover |
| **Prefetch Limit** | Message queues | RabbitMQ prefetch(N) |
| **Pull Model** | Event streaming | Kafka consumer pulls |

### Key Insight (Memorize This!)
> "It's better to be slow than dead. A system that responds slowly under load will recover when load decreases. A system that crashes under load requires manual intervention and may lose data."

---

### Pattern 6: Projection Rebuild

```typescript
// Rebuild read model from scratch (after bug fix or new projection)

class ProjectionRebuilder {
  async rebuild(
    projection: Projection,
    eventStore: EventStore
  ): Promise<void> {
    console.log(`Rebuilding ${projection.name}...`);
    
    // Clear existing data
    await projection.clear();
    
    // Get all events
    const allStreams = await eventStore.getAllStreamIds();
    let processed = 0;
    
    for (const streamId of allStreams) {
      const events = await eventStore.read(streamId);
      
      for (const event of events) {
        await projection.apply(event);
        processed++;
        
        if (processed % 1000 === 0) {
          console.log(`Processed ${processed} events...`);
        }
      }
    }
    
    console.log(`Rebuild complete. ${processed} events processed.`);
  }
}

// Catchup subscription (process missed events then subscribe live)
class CatchupSubscription {
  async start(
    projection: Projection,
    eventStore: EventStore
  ): Promise<void> {
    // Get last processed position
    const lastPosition = await projection.getLastPosition();
    
    // Process all events since last position
    const events = await eventStore.readAll(lastPosition);
    for (const event of events) {
      await projection.apply(event);
      await projection.savePosition(event.position);
    }
    
    // Subscribe to live events
    eventStore.subscribeToAll(lastPosition, async (event) => {
      await projection.apply(event);
      await projection.savePosition(event.position);
    });
  }
}
```

---

## 6. Real-World Scenarios

### Scenario 1: E-Commerce Order Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    E-Commerce Event Flow                        │
└─────────────────────────────────────────────────────────────────┘

User places order
       │
       ▼
┌─────────────┐    OrderCreated    ┌─────────────┐
│ Order       │ ──────────────────▶│   Kafka     │
│ Service     │                    │   Topic     │
└─────────────┘                    └─────────────┘
                                         │
           ┌───────────────┬─────────────┼─────────────┬───────────┐
           ▼               ▼             ▼             ▼           ▼
    ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
    │  Payment    │ │ Inventory   │ │ Notification│ │  Analytics  │
    │  Service    │ │  Service    │ │  Service    │ │  Service    │
    └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘
           │               │             │
           ▼               ▼             ▼
    PaymentProcessed InventoryReserved  Email Sent
           │
           ▼
    ┌─────────────┐
    │  Shipping   │ ───▶ OrderShipped
    │  Service    │
    └─────────────┘
```

### Scenario 2: Banking with Event Sourcing

```typescript
// Account aggregate
class BankAccount {
  private balance: number = 0;
  private dailyWithdrawals: number = 0;

  deposit(amount: number): void {
    if (amount <= 0) throw new Error("Invalid amount");
    
    this.raise({
      type: "MoneyDeposited",
      payload: { amount, newBalance: this.balance + amount }
    });
  }

  withdraw(amount: number): void {
    if (amount <= 0) throw new Error("Invalid amount");
    if (amount > this.balance) throw new Error("Insufficient funds");
    if (this.dailyWithdrawals + amount > 10000) {
      throw new Error("Daily limit exceeded");
    }
    
    this.raise({
      type: "MoneyWithdrawn",
      payload: { amount, newBalance: this.balance - amount }
    });
  }

  private apply(event: Event): void {
    switch (event.type) {
      case "MoneyDeposited":
        this.balance += event.payload.amount;
        break;
      case "MoneyWithdrawn":
        this.balance -= event.payload.amount;
        this.dailyWithdrawals += event.payload.amount;
        break;
    }
  }
}

// Time travel debugging
async function investigateIssue(accountId: string, date: Date): Promise<void> {
  const events = await eventStore.read(`account-${accountId}`);
  
  // Filter events up to the date
  const eventsAtDate = events.filter(e => e.timestamp <= date);
  
  // Rebuild state at that point
  const account = BankAccount.fromEvents(eventsAtDate);
  
  console.log(`Account state at ${date}:`, account.getState());
  console.log('Events that day:', eventsAtDate.filter(
    e => e.timestamp.toDateString() === date.toDateString()
  ));
}
```

### Scenario 3: Real-Time Collaboration (CQRS)

```typescript
// Document editing with CQRS

// Write side: Commands
interface EditDocumentCommand {
  documentId: string;
  userId: string;
  operations: DocumentOperation[];  // CRDTs for conflict-free merging
}

class DocumentCommandHandler {
  async handle(command: EditDocumentCommand): Promise<void> {
    const document = await this.repo.getById(command.documentId);
    document.applyOperations(command.operations);
    await this.repo.save(document);
    
    // Publish for real-time sync
    await this.eventBus.publish({
      type: "DocumentEdited",
      documentId: command.documentId,
      operations: command.operations,
      userId: command.userId
    });
  }
}

// Read side: Multiple projections
class DocumentProjections {
  // Projection 1: Full document for editing
  async onDocumentEdited(event: DocumentEditedEvent): Promise<void> {
    await this.redis.publish(
      `document:${event.documentId}`,
      JSON.stringify(event)  // Real-time to connected clients
    );
  }

  // Projection 2: Document list for dashboard
  async onDocumentEdited(event: DocumentEditedEvent): Promise<void> {
    await this.db.documentList.update(
      { id: event.documentId },
      { 
        lastEditedAt: event.timestamp,
        lastEditedBy: event.userId 
      }
    );
  }

  // Projection 3: Search index
  async onDocumentEdited(event: DocumentEditedEvent): Promise<void> {
    const content = await this.getDocumentContent(event.documentId);
    await this.elasticsearch.update({
      index: 'documents',
      id: event.documentId,
      body: { content, updatedAt: event.timestamp }
    });
  }
}
```

---

## 7. Common Pitfalls

### ❌ Pitfall 1: Not Making Consumers Idempotent

```typescript
// BAD: Will double-charge on retry
async function handlePaymentRequested(event: PaymentRequestedEvent) {
  await paymentGateway.charge(event.amount);  // Duplicate charges!
}

// GOOD: Idempotent with deduplication
async function handlePaymentRequested(event: PaymentRequestedEvent) {
  const existing = await db.payments.findOne({ eventId: event.id });
  if (existing) return;  // Already processed
  
  await paymentGateway.charge(event.amount, { idempotencyKey: event.id });
  await db.payments.insert({ eventId: event.id, status: 'processed' });
}
```

### ❌ Pitfall 2: Large Events

```typescript
// BAD: Huge event payload
const event = {
  type: "OrderCreated",
  payload: {
    customer: fullCustomerObject,      // 100+ fields
    products: fullProductCatalog,       // Megabytes of data
    attachments: base64EncodedFiles    // OMG
  }
};

// GOOD: Minimal event, reference IDs
const event = {
  type: "OrderCreated",
  payload: {
    orderId: "order-123",
    customerId: "cust-456",
    productIds: ["prod-1", "prod-2"],
    total: 99.99
  }
};
// Consumers fetch additional data if needed
```

### ❌ Pitfall 3: Synchronous Event Handling

```typescript
// BAD: Blocking on event processing
await eventBus.publish(event);  // Waits for all handlers

// GOOD: Fire and forget, process async
eventBus.publish(event);  // Returns immediately
// Or use message broker for true async
```

### ❌ Pitfall 4: Missing Event Schema Versioning

```typescript
// BAD: Breaking change with no versioning
// V1: { items: ["prod-1", "prod-2"] }
// V2: { items: [{ productId: "prod-1", qty: 1 }] }
// Old consumers crash!

// GOOD: Version and migrate
const event = {
  type: "OrderCreated",
  version: 2,
  payload: { ... }
};

// Consumer handles multiple versions
function handle(event: OrderCreatedEvent) {
  const normalized = event.version === 1 
    ? migrateV1toV2(event) 
    : event;
  // Process normalized event
}
```

### ❌ Pitfall 5: Ignoring Event Ordering

```typescript
// BAD: Processing out of order
// Events: OrderCreated → OrderCancelled
// If processed: OrderCancelled → OrderCreated
// Result: Order exists but should be cancelled!

// GOOD: Ensure ordering per aggregate
// Kafka: Use aggregateId as partition key
await producer.send({
  topic: 'orders',
  messages: [{
    key: order.id,  // Same order → same partition → ordered
    value: JSON.stringify(event)
  }]
});
```

### ❌ Pitfall 6: Not Handling Poison Messages

```typescript
// BAD: One bad message blocks everything
consumer.on('message', async (msg) => {
  await process(msg);  // Throws on invalid message, retries forever
});

// GOOD: Dead letter queue after retries
consumer.on('message', async (msg) => {
  const retryCount = msg.headers['x-retry-count'] || 0;
  
  try {
    await process(msg);
  } catch (error) {
    if (retryCount < 3) {
      await republish(msg, retryCount + 1);
    } else {
      await sendToDeadLetter(msg, error);
    }
  }
});
```

---

## 8. Interview Questions & Answers

### Basic Questions

**Q: What is the difference between a command and an event?**
> **A:** A command is a request to do something (imperative: "CreateOrder"). It can be rejected. An event is a fact that something happened (past tense: "OrderCreated"). It cannot be rejected - it already happened. Commands represent intent, events represent facts.

**Q: What is eventual consistency?**
> **A:** In event-driven systems, different services may see different states at any given moment, but will eventually converge to the same state. After an event is published, consumers process it at different speeds. We accept temporary inconsistency for the benefits of decoupling and scalability.

**Q: Why use a message broker instead of direct HTTP calls?**
> **A:** 
> 1. **Decoupling** - Publisher doesn't need to know subscribers
> 2. **Resilience** - Messages are buffered if consumer is down
> 3. **Scalability** - Add consumers without changing producer
> 4. **Async processing** - Don't block on slow operations

### Intermediate Questions

**Q: Explain the Outbox Pattern.**
> **A:** The outbox pattern solves the dual-write problem - we need to update the database AND publish an event atomically. Solution: Write both the business data and the event to the database in the same transaction. A separate process reads from the outbox table and publishes to the message broker. This ensures we never publish an event without the database change, or vice versa.

**Q: How do you handle a consumer that's slower than the producer?**
> **A:**
> 1. **Scale consumers** - Add more instances (Kafka partitions enable this)
> 2. **Backpressure** - Slow down producer if possible
> 3. **Drop or batch** - For analytics, aggregate before processing
> 4. **Prioritize** - Process important events first
> 5. **Monitor lag** - Alert when consumer falls behind

**Q: What's the difference between choreography and orchestration in sagas?**
> **A:**
> - **Choreography**: Each service reacts to events and publishes its own. No central coordinator. Simpler but harder to understand the full flow.
> - **Orchestration**: A central saga orchestrator tells each service what to do and handles failures. More control but single point of coordination.
> 
> Use choreography for simple flows, orchestration for complex ones.

### Advanced Questions

**Q: How do you handle GDPR "right to be forgotten" with event sourcing?**
> **A:** You can't delete events (immutability), so use **crypto-shredding**: 
> 1. Encrypt personal data in events with a per-user key
> 2. Store the key separately
> 3. To "forget" a user, delete their key
> 4. Events remain but personal data is unreadable
> 
> Alternative: Store personal data references (userId), not actual data. Delete from reference store.

**Q: How do you rebuild a projection that takes hours?**
> **A:**
> 1. **Build new projection alongside old** - Don't delete old until new is ready
> 2. **Use snapshots** - Checkpoint progress, resume if interrupted
> 3. **Parallel processing** - Process multiple streams concurrently
> 4. **Blue-green deployment** - New projection in new database, switch when ready
> 5. **Incremental replay** - If possible, only replay affected events

**Q: How do you ensure exactly-once processing?**
> **A:** True exactly-once is impossible in distributed systems. We achieve effectively exactly-once through:
> 1. **At-least-once delivery** (broker guarantees) + **Idempotent processing** (deduplication)
> 2. **Transactional outbox** - Database + event in one transaction
> 3. **Idempotency keys** - Include event ID in downstream calls
> 
> The combination gives us exactly-once semantics even though individual components only guarantee at-least-once.

**Q: When would you NOT use event sourcing?**
> **A:**
> 1. **Simple CRUD** - Overhead isn't worth it
> 2. **Frequent updates to same data** - Event stream grows huge
> 3. **Team unfamiliar** - Steep learning curve
> 4. **Need for ad-hoc queries** - Can't easily query event store
> 5. **Strict consistency required** - Eventual consistency unacceptable
> 6. **Regulatory deletion requirements** - Hard with immutable events

---

## 📦 Recommended Tools & Libraries

### Event Stores
| Tool | Language | Notes |
|------|----------|-------|
| **EventStoreDB** | Any | Purpose-built, projections built-in |
| **Marten** | .NET | PostgreSQL-based |
| **Axon Framework** | Java | Full CQRS/ES framework |
| **eventstore** | Node.js | EventStoreDB client |

### Message Brokers
| Tool | Best For | Notes |
|------|----------|-------|
| **Kafka** | High throughput | Industry standard |
| **RabbitMQ** | Flexible routing | Good for task queues |
| **AWS SQS/SNS** | AWS apps | Managed, serverless |
| **Redis Streams** | Real-time | Simple, fast |
| **NATS** | Cloud native | Lightweight |

### CQRS Frameworks
| Tool | Language | Notes |
|------|----------|-------|
| **MediatR** | .NET | Command/Query dispatching |
| **NestJS CQRS** | Node.js | Built-in module |
| **Lagom** | Java/Scala | Full microservices framework |

---

## 🎓 Key Takeaways

1. **Events are facts** - Immutable, past tense, can't be rejected
2. **Event sourcing = events as source of truth** - Store events, derive state
3. **CQRS = separate read/write** - Optimize each independently
4. **Message brokers decouple services** - Async, resilient, scalable
5. **Idempotency is critical** - Messages can be delivered multiple times
6. **Eventual consistency is the trade-off** - Accept for benefits of decoupling
7. **Start simple** - Don't adopt event sourcing unless you need it
```


