# Chapter 05: Message Queues & Event-Driven Architecture

> "Decoupling through messaging is the key to scalable systems."

---

## 🎯 Why Message Queues?

### The Problem

```
Synchronous (tightly coupled):
┌────────┐  1. Order    ┌────────┐  2. Notify  ┌────────┐
│ Client │─────────────►│ Order  │────────────►│ Email  │
│        │              │Service │             │Service │
│        │◄─────────────│        │◄────────────│        │
│        │  5. Response │        │  3. Done    │        │
└────────┘    (3 sec)   └────────┘             └────────┘
                             │
                             │ 4. Inventory
                             ▼
                        ┌────────┐
                        │Inventory│
                        │Service │
                        └────────┘

Problems:
- User waits for ALL services to respond
- If Email service is down, order fails
- Can't scale services independently
- Tight coupling between services
```

### The Solution

```
Asynchronous (loosely coupled):
┌────────┐  1. Order    ┌────────┐  2. Publish  ┌─────────┐
│ Client │─────────────►│ Order  │─────────────►│  Queue  │
│        │◄─────────────│Service │             │(Kafka/  │
│        │  3. Accepted │        │             │ RabbitMQ│
└────────┘    (100ms)   └────────┘             └────┬────┘
                                                    │
                              ┌─────────────────────┼─────────────────────┐
                              │                     │                     │
                              ▼                     ▼                     ▼
                         ┌────────┐           ┌────────┐           ┌────────┐
                         │ Email  │           │Inventory│          │Analytics│
                         │Service │           │Service │           │Service │
                         └────────┘           └────────┘           └────────┘

Benefits:
- User gets immediate response
- Services can fail independently
- Easy to add new consumers
- Services scale independently
```

---

## 📬 Message Queue Patterns

### 1. Point-to-Point (Work Queue)

```
One message → One consumer

Producer ──► ┌─────────────┐ ──► Consumer 1
             │    Queue    │ ──► Consumer 2
             │ [M1][M2][M3]│ ──► Consumer 3
             └─────────────┘

Each message processed by exactly ONE consumer
Work is distributed among consumers
Used for: Task processing, job queues
```

### 2. Publish-Subscribe (Fan-out)

```
One message → All consumers

Producer ──► ┌─────────────┐ ──► Subscriber 1 (gets all)
             │    Topic    │ ──► Subscriber 2 (gets all)
             │ [M1][M2][M3]│ ──► Subscriber 3 (gets all)
             └─────────────┘

All subscribers get ALL messages
Used for: Notifications, event broadcasting
```

### 3. Request-Reply

```
┌──────────┐  Request   ┌─────────────┐
│ Producer │───────────►│Request Queue│
│          │            └──────┬──────┘
│          │                   │
│          │                   ▼
│          │            ┌─────────────┐
│          │            │  Consumer   │
│          │            └──────┬──────┘
│          │                   │
│          │  Response  ┌──────┴──────┐
│          │◄───────────│Response Queue│
└──────────┘            └─────────────┘

Correlation ID links request to response
Used for: RPC over messaging
```

### 4. Dead Letter Queue (DLQ)

```
                         ┌─────────────┐
Producer ──► Main Queue ─┤  Consumer   │
                         │  (fails)    │
                         └──────┬──────┘
                                │ After N retries
                                ▼
                         ┌─────────────┐
                         │Dead Letter  │
                         │   Queue     │
                         └─────────────┘
                                │
                         Manual review/reprocess

Used for: Error handling, debugging failed messages
```

---

## 🐰 RabbitMQ

### Core Concepts

```
┌─────────────────────────────────────────────────────────────────┐
│                           RabbitMQ                              │
│  ┌──────────┐     ┌──────────────┐     ┌─────────────────────┐ │
│  │ Producer │────►│   Exchange   │────►│       Queue         │ │
│  └──────────┘     │   (Router)   │     │                     │ │
│                   └──────────────┘     └──────────┬──────────┘ │
│                                                    │            │
│                                                    ▼            │
│                                             ┌──────────┐        │
│                                             │ Consumer │        │
│                                             └──────────┘        │
└─────────────────────────────────────────────────────────────────┘

Exchange Types:
- Direct: Route by exact routing key match
- Topic: Route by pattern matching (orders.*, *.created)
- Fanout: Broadcast to all bound queues
- Headers: Route by message headers
```

### RabbitMQ Example

```javascript
const amqp = require('amqplib');

// Publisher
async function publishOrder(order) {
  const connection = await amqp.connect('amqp://localhost');
  const channel = await connection.createChannel();
  
  const exchange = 'orders';
  const routingKey = 'order.created';
  
  await channel.assertExchange(exchange, 'topic', { durable: true });
  
  channel.publish(
    exchange,
    routingKey,
    Buffer.from(JSON.stringify(order)),
    { persistent: true }  // Survive broker restart
  );
  
  console.log('Order published');
  await channel.close();
  await connection.close();
}

// Consumer
async function consumeOrders() {
  const connection = await amqp.connect('amqp://localhost');
  const channel = await connection.createChannel();
  
  const queue = 'email-notifications';
  const exchange = 'orders';
  
  await channel.assertQueue(queue, { durable: true });
  await channel.bindQueue(queue, exchange, 'order.*');
  
  // Process one at a time
  channel.prefetch(1);
  
  channel.consume(queue, async (msg) => {
    const order = JSON.parse(msg.content.toString());
    
    try {
      await sendEmail(order);
      channel.ack(msg);  // Acknowledge success
    } catch (error) {
      channel.nack(msg, false, true);  // Requeue for retry
    }
  });
}
```

---

## 📊 Apache Kafka

### Core Concepts

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              Kafka Cluster                              │
│                                                                         │
│  Topic: orders                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │ Partition 0: [msg1] [msg4] [msg7] [msg10]                          │ │
│  │ Partition 1: [msg2] [msg5] [msg8] [msg11]                          │ │
│  │ Partition 2: [msg3] [msg6] [msg9] [msg12]                          │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│  Consumer Group A:                    Consumer Group B:                 │
│  ┌─────────┐ ┌─────────┐             ┌─────────┐                       │
│  │Consumer1│ │Consumer2│             │Consumer1│                       │
│  │(P0, P1) │ │  (P2)   │             │(P0,P1,P2)│                       │
│  └─────────┘ └─────────┘             └─────────┘                       │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘

Key Concepts:
- Topic: Category of messages
- Partition: Ordered, immutable log within a topic
- Offset: Position in partition
- Consumer Group: Set of consumers sharing work
- Replication: Partitions replicated for fault tolerance
```

### Why Kafka?

```
RabbitMQ vs Kafka:

RabbitMQ:
- Message broker (push to consumers)
- Messages deleted after consumption
- Complex routing (exchanges)
- Good for: Task queues, RPC

Kafka:
- Distributed log (consumers pull)
- Messages retained (configurable)
- Ordered within partition
- Replayable (go back in time)
- Good for: Event streaming, event sourcing, analytics
```

### Kafka Example

```javascript
const { Kafka } = require('kafkajs');

const kafka = new Kafka({
  clientId: 'my-app',
  brokers: ['localhost:9092']
});

// Producer
async function publishEvent(event) {
  const producer = kafka.producer();
  await producer.connect();
  
  await producer.send({
    topic: 'user-events',
    messages: [
      {
        key: event.userId,      // For partitioning
        value: JSON.stringify(event),
        headers: {
          'event-type': event.type
        }
      }
    ]
  });
  
  await producer.disconnect();
}

// Consumer
async function consumeEvents() {
  const consumer = kafka.consumer({ groupId: 'analytics-group' });
  await consumer.connect();
  
  await consumer.subscribe({ topic: 'user-events', fromBeginning: false });
  
  await consumer.run({
    eachMessage: async ({ topic, partition, message }) => {
      const event = JSON.parse(message.value.toString());
      console.log({
        partition,
        offset: message.offset,
        key: message.key?.toString(),
        event
      });
      
      // Process event...
    }
  });
}

// Exactly-once semantics
async function processWithTransaction() {
  const producer = kafka.producer({
    transactionalId: 'my-transactional-producer',
    idempotent: true
  });
  
  await producer.connect();
  
  const transaction = await producer.transaction();
  try {
    await transaction.send({ topic: 'topic1', messages: [...] });
    await transaction.send({ topic: 'topic2', messages: [...] });
    await transaction.commit();
  } catch (e) {
    await transaction.abort();
  }
}
```

---

## 🎭 Event-Driven Architecture

### Event Types

```
1. Domain Events (business events):
   - OrderCreated
   - PaymentReceived
   - UserRegistered

2. Integration Events (cross-service):
   - OrderService → InventoryService
   - UserService → EmailService

3. Event Notifications (tell, don't carry data):
   { "type": "OrderCreated", "orderId": "123" }
   Consumers fetch data if needed

4. Event-Carried State Transfer (carry data):
   { "type": "OrderCreated", "order": { ... full order ... } }
   Consumers have all data needed
```

### Event Design

```javascript
// Good event structure
{
  "eventId": "uuid-123",           // Unique ID for idempotency
  "eventType": "OrderCreated",     // Type for routing
  "timestamp": "2024-01-15T10:30:00Z",
  "version": 1,                    // Schema version
  "source": "order-service",       // Origin
  "correlationId": "request-456",  // For tracing
  "causationId": "event-789",      // What caused this
  "data": {
    "orderId": "order-123",
    "userId": "user-456",
    "items": [...]
  }
}

// Event naming conventions:
// PastTense: OrderCreated, PaymentReceived
// Specific: UserEmailChanged (not UserUpdated)
// Business language: ItemAddedToCart
```

### Saga Pattern (Distributed Transactions)

```
Problem:
Order spans multiple services (Order, Payment, Inventory, Shipping)
Can't use traditional transactions

Solution: Saga (sequence of local transactions)

Choreography (event-driven):
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│  Order      Payment     Inventory    Shipping                          │
│ Service     Service      Service     Service                           │
│    │           │            │           │                              │
│    │ OrderCreated          │           │                              │
│    ├──────────►│            │           │                              │
│    │           │ PaymentReceived       │                              │
│    │           ├───────────►│           │                              │
│    │           │            │ InventoryReserved                        │
│    │           │            ├──────────►│                              │
│    │           │            │           │ ShippingScheduled            │
│    │◄──────────┴────────────┴───────────┤                              │
│    │ OrderCompleted                     │                              │
│                                                                         │
│ If Payment fails:                                                       │
│    │ PaymentFailed                      │                              │
│    ├──────────►│ (compensate)          │                              │
│    │ OrderCancelled                     │                              │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘

Orchestration (centralized coordinator):
┌─────────────────────────────────────────────────────────────────────────┐
│                         Saga Orchestrator                               │
│    ┌────────────────────────────────────────────────────────────┐      │
│    │ 1. Create Order                                            │      │
│    │ 2. Reserve Payment                                         │      │
│    │ 3. Reserve Inventory                                       │      │
│    │ 4. Schedule Shipping                                       │      │
│    │ 5. Complete Order                                          │      │
│    └────────────────────────────────────────────────────────────┘      │
│         │            │            │            │                        │
│         ▼            ▼            ▼            ▼                        │
│    ┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐                   │
│    │ Order  │   │Payment │   │Inventory│  │Shipping│                   │
│    │Service │   │Service │   │Service │   │Service │                   │
│    └────────┘   └────────┘   └────────┘   └────────┘                   │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📈 Event Sourcing

```
Traditional (state storage):
┌─────────────────────────────────────────┐
│ accounts table                          │
│ id: 123, balance: 500                   │  ← Only current state
└─────────────────────────────────────────┘

Event Sourcing (event storage):
┌─────────────────────────────────────────┐
│ account_events table                    │
│ 1. AccountCreated(id: 123, balance: 0)  │
│ 2. MoneyDeposited(id: 123, amount: 1000)│
│ 3. MoneyWithdrawn(id: 123, amount: 300) │
│ 4. MoneyWithdrawn(id: 123, amount: 200) │
│ → Current balance: 500                  │
└─────────────────────────────────────────┘

Benefits:
- Complete audit trail
- Debug production (replay events)
- Temporal queries (balance at any point)
- Event-driven projections
```

```javascript
// Event Sourcing implementation
class Account {
  constructor() {
    this.balance = 0;
    this.events = [];
  }
  
  // Apply events to rebuild state
  apply(event) {
    switch (event.type) {
      case 'AccountCreated':
        this.balance = event.initialBalance;
        break;
      case 'MoneyDeposited':
        this.balance += event.amount;
        break;
      case 'MoneyWithdrawn':
        this.balance -= event.amount;
        break;
    }
    this.events.push(event);
  }
  
  // Commands produce events
  deposit(amount) {
    if (amount <= 0) throw new Error('Invalid amount');
    this.apply({
      type: 'MoneyDeposited',
      amount,
      timestamp: new Date()
    });
  }
  
  withdraw(amount) {
    if (amount > this.balance) throw new Error('Insufficient funds');
    this.apply({
      type: 'MoneyWithdrawn',
      amount,
      timestamp: new Date()
    });
  }
  
  // Load from event store
  static fromEvents(events) {
    const account = new Account();
    events.forEach(e => account.apply(e));
    return account;
  }
}
```

---

## 📖 Further Reading

- "Enterprise Integration Patterns" by Hohpe
- "Designing Event-Driven Systems" (Confluent)
- "Building Event-Driven Microservices"

---

**Next:** [Chapter 06: Distributed Systems →](./06-distributed-systems.md)


