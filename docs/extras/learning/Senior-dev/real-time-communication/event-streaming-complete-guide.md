# 📊 Event Streaming - Complete Guide

> A comprehensive guide to event streaming - Apache Kafka deep dive, partitioning strategies, consumer groups, exactly-once semantics, event replay, and building real-time data pipelines.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Event streaming is a paradigm for capturing, storing, and processing continuous flows of events in real-time using distributed logs (like Kafka), where events are durably stored, partitioned for parallelism, and consumed by consumer groups with exactly-once semantics and replay capability."

### The 7 Key Concepts (Remember These!)
```
1. TOPIC           → Named category/feed of events (like a table)
2. PARTITION       → Parallel unit within topic (ordering guaranteed within)
3. OFFSET          → Position of message within partition (sequential ID)
4. PRODUCER        → Publishes events to topics
5. CONSUMER GROUP  → Coordinated consumers sharing partition load
6. BROKER          → Kafka server storing and serving events
7. COMMIT          → Mark offset as processed (checkpoint)
```

### Kafka Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                    KAFKA ARCHITECTURE                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  PRODUCERS                         KAFKA CLUSTER                │
│  ─────────                         ─────────────                │
│                                                                 │
│  ┌─────────┐                      ┌─────────────────────────┐  │
│  │Producer │──────────────────────▶│  TOPIC: orders         │  │
│  │   A     │                      │  ┌─────────────────────┐│  │
│  └─────────┘                      │  │ Partition 0         ││  │
│                                   │  │ [0][1][2][3][4]     ││  │
│  ┌─────────┐                      │  ├─────────────────────┤│  │
│  │Producer │──────────────────────▶│ │ Partition 1         ││  │
│  │   B     │                      │  │ [0][1][2][3]        ││  │
│  └─────────┘                      │  ├─────────────────────┤│  │
│                                   │  │ Partition 2         ││  │
│                                   │  │ [0][1][2][3][4][5]  ││  │
│                                   │  └─────────────────────┘│  │
│                                   └─────────────────────────┘  │
│                                               │                │
│                                               │                │
│  CONSUMER GROUP: order-processors            │                │
│  ────────────────────────────────            ▼                │
│                                                                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐           │
│  │ Consumer 1  │  │ Consumer 2  │  │ Consumer 3  │           │
│  │ (P0)        │  │ (P1)        │  │ (P2)        │           │
│  └─────────────┘  └─────────────┘  └─────────────┘           │
│                                                                │
│  Each consumer in group gets exclusive partitions              │
│  Max parallelism = number of partitions                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Partition Key Routing
```
┌─────────────────────────────────────────────────────────────────┐
│                   PARTITION KEY ROUTING                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Message: { key: "user-123", value: {...} }                    │
│                                                                 │
│  partition = hash(key) % num_partitions                        │
│            = hash("user-123") % 3                              │
│            = 1                                                 │
│                                                                 │
│  ┌──────────────────────────────────────────────────┐         │
│  │              Topic: user-events                   │         │
│  │  ┌────────────────────────────────────────────┐  │         │
│  │  │ Partition 0   │ Partition 1   │ Partition 2│  │         │
│  │  │               │ ★ user-123   │             │  │         │
│  │  │ user-456      │   user-123   │ user-789   │  │         │
│  │  │ user-456      │   user-123   │ user-012   │  │         │
│  │  └────────────────────────────────────────────┘  │         │
│  └──────────────────────────────────────────────────┘         │
│                                                                 │
│  ✓ Same key → Same partition → Ordering guaranteed             │
│  ✓ All events for user-123 are in partition 1, in order       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Consumer Group Rebalancing
```
┌─────────────────────────────────────────────────────────────────┐
│                  CONSUMER GROUP REBALANCING                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SCENARIO: 3 partitions, consumers join/leave                   │
│                                                                 │
│  Step 1: 1 consumer                                            │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Consumer A: [P0, P1, P2]                                │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  Step 2: Consumer B joins → Rebalance                          │
│  ┌────────────────────────┐  ┌────────────────────────────┐   │
│  │ Consumer A: [P0, P1]   │  │ Consumer B: [P2]           │   │
│  └────────────────────────┘  └────────────────────────────┘   │
│                                                                 │
│  Step 3: Consumer C joins → Rebalance                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │ Consumer A   │  │ Consumer B   │  │ Consumer C   │         │
│  │ [P0]         │  │ [P1]         │  │ [P2]         │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
│                                                                 │
│  Step 4: Consumer B crashes → Rebalance                        │
│  ┌─────────────────────────┐  ┌─────────────────────────┐     │
│  │ Consumer A: [P0, P1]    │  │ Consumer C: [P2]        │     │
│  └─────────────────────────┘  └─────────────────────────┘     │
│                                                                 │
│  Key: Max useful consumers = num partitions                    │
│       Extra consumers sit idle (standby)                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Partition key"** | "We partition by user ID so all user events are ordered" |
| **"Consumer group"** | "Multiple consumers in the same group share the workload" |
| **"Offset commit"** | "We commit offsets after processing to track progress" |
| **"Exactly-once semantics"** | "We use transactions for exactly-once semantics" |
| **"Compacted topic"** | "We use a compacted topic as a changelog for the latest state" |
| **"ISR (In-Sync Replicas)"** | "Messages are durable when written to all ISR brokers" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Max consumers per group | **= num partitions** | Each partition assigned to one consumer |
| Recommended partition count | **10-100** per topic | Balance parallelism vs overhead |
| Message retention default | **7 days** | Configurable, can be infinite |
| Replication factor | **3** (production) | Survive 2 broker failures |
| Batch size | **16KB-1MB** | Balance latency vs throughput |
| Consumer poll timeout | **5-30 seconds** | Before rebalance assumed |

### The "Wow" Statement (Memorize This!)
> "We built a real-time analytics pipeline using Kafka handling 500K events/second. Events are partitioned by customer ID to ensure ordering within each customer's event stream. We have multiple consumer groups: one for real-time dashboards (reads from latest), one for data warehouse loading (batch-oriented), and one for alerting (filters interesting events). We use exactly-once semantics with idempotent producers and transactional commits to prevent duplicates. The compacted changelog topic maintains current state per entity. For replay scenarios (bug fixes, new features), we reset consumer offsets to reprocess historical data. Kafka's durable storage means events are available for 30 days, enabling both real-time and batch analytics on the same data."

### Quick Architecture Drawing (Draw This!)
```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  EVENT SOURCES              KAFKA                CONSUMERS      │
│  ─────────────              ─────                ─────────      │
│                                                                 │
│  ┌─────────┐              ┌────────────────┐                   │
│  │ Web App │──────────────▶│   Topic:      │    ┌───────────┐ │
│  └─────────┘              │   orders      │───▶│ Analytics │ │
│                           │               │    └───────────┘ │
│  ┌─────────┐              │  [P0][P1][P2] │                   │
│  │ Mobile  │──────────────▶│               │    ┌───────────┐ │
│  └─────────┘              └────────────────┘───▶│ Warehouse │ │
│                                                 └───────────┘ │
│  ┌─────────┐              ┌────────────────┐                   │
│  │ IoT     │──────────────▶│   Topic:      │    ┌───────────┐ │
│  └─────────┘              │   telemetry   │───▶│ Monitoring│ │
│                           └────────────────┘    └───────────┘ │
│                                                                 │
│  Key Benefits:                                                 │
│  • Decoupling (producers don't know consumers)                 │
│  • Durability (events persisted, replayable)                  │
│  • Scalability (add partitions, consumers)                    │
│  • Multiple consumers (same data, different purposes)          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Interview Rapid Fire (Practice These!)

**Q: "What is Apache Kafka?"**
> "Distributed event streaming platform. Durably stores events in topics (partitioned, replicated logs). Producers publish, consumers subscribe in groups. High throughput, fault-tolerant, supports replay. Used for real-time pipelines, event sourcing, log aggregation."

**Q: "How do partitions work?"**
> "Topics are split into partitions for parallelism. Messages with same key go to same partition (ordering preserved). Consumers in a group each get exclusive partitions. More partitions = more parallelism, but max consumers = partitions."

**Q: "What is a consumer group?"**
> "Logical group of consumers that share topic workload. Each partition assigned to one consumer in group. Multiple groups can independently read same topic. Enables parallel processing and fault tolerance."

**Q: "How does Kafka guarantee ordering?"**
> "Ordering is guaranteed within a partition only, not across partitions. Use partition key (e.g., user ID) to ensure related events go to same partition. All events for that key will be ordered."

**Q: "What is offset commit?"**
> "Consumer's progress marker - which message was last processed. Auto-commit: periodic, at-most-once. Manual commit: after processing, at-least-once. Committed offsets survive consumer restarts."

**Q: "Exactly-once vs at-least-once?"**
> "At-least-once: might process duplicates (consumer crashes after process, before commit). Exactly-once: uses idempotent producers + transactions + atomic commits. More complex, slight overhead, but no duplicates."

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "How does Kafka work?"

**Junior Answer:**
> "It's a message queue that stores messages and lets you read them."

**Senior Answer:**
> "Kafka is a distributed commit log optimized for high-throughput event streaming:

1. **Topics and Partitions**: Topics are split into partitions (parallel units). Each partition is an ordered, immutable sequence of records with sequential offsets.

2. **Producers**: Publish records with optional key. Key determines partition (hash(key) % partitions). No key = round-robin.

3. **Consumers**: Poll for records, track progress via offsets. Consumer groups coordinate - each partition goes to one consumer in group.

4. **Durability**: Records replicated across brokers (configurable factor). Persisted to disk, retained for configurable time (or forever for compacted topics).

5. **Performance**: Sequential disk I/O, zero-copy transfers, batching, compression. Achieves millions of messages/second per broker.

The key insight is Kafka trades the flexibility of arbitrary message delivery for the performance and semantics of an ordered log. You get ordering within partition, replay capability, and independent consumer groups."

### When Asked: "How do you design topics and partitions?"

**Junior Answer:**
> "Just create a topic and use it."

**Senior Answer:**
> "Partition design is critical:

1. **Partition count**: Start with expected parallelism × 2. More is better for scaling, but too many adds overhead. Can only increase, never decrease.

2. **Partition key**: Choose carefully - determines ordering guarantees. Common: entity ID (user, order). All events for that entity ordered.

3. **Hotspots**: Avoid keys that create uneven distribution (e.g., country code if 80% from one country). Use composite keys or sampling.

4. **Topic granularity**: One topic per event type (orders, payments) vs one topic with event types in payload. Depends on consumer patterns.

5. **Retention**: Time-based (7 days), size-based, or compaction (keep latest per key). Match business requirements.

For a high-volume event stream, I'd partition by customer/user ID (ordering within customer), set 50-100 partitions (room to scale), 7-day retention, and use schema registry for evolution."

### Follow-up Questions to Expect:

| They Ask | You Answer |
|----------|------------|
| "How do you handle consumer lag?" | "Monitor lag metrics, scale consumers (up to partition count), increase partition count if needed, check for slow consumers." |
| "What about message ordering?" | "Ordering only within partition. Use single partition for global order (limits throughput), or partition key for per-entity ordering." |
| "How do you handle failures?" | "Replication handles broker failures. Consumer failures trigger rebalance. Produce failures retry with idempotent producer. At-least-once is default, exactly-once needs transactions." |
| "Kafka vs RabbitMQ?" | "Kafka: distributed log, high throughput, durability, replay, consumer groups. RabbitMQ: traditional broker, flexible routing, lower latency, simpler for request-reply." |

---

## 📚 Table of Contents

1. [Kafka Producer](#1-kafka-producer)
2. [Kafka Consumer](#2-kafka-consumer)
3. [Consumer Groups](#3-consumer-groups)
4. [Partitioning Strategies](#4-partitioning-strategies)
5. [Exactly-Once Semantics](#5-exactly-once-semantics)
6. [Event Replay](#6-event-replay)
7. [Schema Registry](#7-schema-registry)
8. [Kafka Streams](#8-kafka-streams)
9. [Common Pitfalls](#9-common-pitfalls)
10. [Interview Questions](#10-interview-questions)

---

## 1. Kafka Producer

```typescript
// ════════════════════════════════════════════════════════════════
// KAFKA PRODUCER (Node.js with kafkajs)
// ════════════════════════════════════════════════════════════════

import { Kafka, Producer, Partitioners, CompressionTypes } from 'kafkajs';

// Create Kafka client
const kafka = new Kafka({
  clientId: 'my-app',
  brokers: ['kafka1:9092', 'kafka2:9092', 'kafka3:9092'],
  
  // Connection settings
  connectionTimeout: 3000,
  requestTimeout: 30000,
  
  // Retry settings
  retry: {
    initialRetryTime: 100,
    retries: 8
  },
  
  // SSL (production)
  ssl: process.env.NODE_ENV === 'production',
  
  // SASL authentication (production)
  sasl: process.env.NODE_ENV === 'production' ? {
    mechanism: 'plain',
    username: process.env.KAFKA_USERNAME!,
    password: process.env.KAFKA_PASSWORD!
  } : undefined
});

// Create producer
const producer: Producer = kafka.producer({
  // Idempotent producer (exactly-once within partition)
  idempotent: true,
  
  // Max in-flight requests (set to 1 for strict ordering)
  maxInFlightRequests: 5,
  
  // Partitioner
  createPartitioner: Partitioners.DefaultPartitioner,
  
  // Transaction ID (for exactly-once across partitions)
  // transactionalId: 'my-transactional-producer'
});

// Connect producer
async function connectProducer(): Promise<void> {
  await producer.connect();
  console.log('Producer connected');
}

// ════════════════════════════════════════════════════════════════
// BASIC SENDING
// ════════════════════════════════════════════════════════════════

interface OrderEvent {
  orderId: string;
  customerId: string;
  items: Array<{ productId: string; quantity: number; price: number }>;
  total: number;
  timestamp: string;
}

// Send single message
async function sendOrderEvent(event: OrderEvent): Promise<void> {
  await producer.send({
    topic: 'orders',
    messages: [{
      // Key determines partition (same key = same partition = ordering)
      key: event.customerId,
      value: JSON.stringify(event),
      
      // Optional headers
      headers: {
        'event-type': 'order.created',
        'correlation-id': event.orderId,
        'source': 'order-service'
      },
      
      // Optional: explicit partition (overrides key-based)
      // partition: 0,
      
      // Optional: timestamp (defaults to current time)
      // timestamp: Date.now().toString()
    }]
  });
}

// Send batch of messages (more efficient)
async function sendOrderEvents(events: OrderEvent[]): Promise<void> {
  await producer.send({
    topic: 'orders',
    messages: events.map(event => ({
      key: event.customerId,
      value: JSON.stringify(event)
    })),
    
    // Compression (recommended for production)
    compression: CompressionTypes.GZIP,
    
    // Acknowledgement level
    // acks: 0  - Fire and forget (fastest, may lose)
    // acks: 1  - Leader acknowledged (balanced)
    // acks: -1 - All ISR acknowledged (safest, slowest)
    acks: -1,
    
    // Timeout for acks
    timeout: 30000
  });
}

// Send to multiple topics
async function sendToMultipleTopics(): Promise<void> {
  await producer.sendBatch({
    topicMessages: [
      {
        topic: 'orders',
        messages: [{ key: 'cust-1', value: JSON.stringify({ orderId: '1' }) }]
      },
      {
        topic: 'order-analytics',
        messages: [{ key: 'cust-1', value: JSON.stringify({ orderId: '1' }) }]
      }
    ]
  });
}

// ════════════════════════════════════════════════════════════════
// PRODUCER WITH RETRY AND ERROR HANDLING
// ════════════════════════════════════════════════════════════════

class KafkaProducerService {
  private producer: Producer;
  private connected = false;

  constructor(kafka: Kafka) {
    this.producer = kafka.producer({
      idempotent: true,
      maxInFlightRequests: 5
    });

    // Handle disconnection
    this.producer.on('producer.disconnect', () => {
      this.connected = false;
      console.log('Producer disconnected');
    });
  }

  async connect(): Promise<void> {
    if (this.connected) return;
    
    await this.producer.connect();
    this.connected = true;
  }

  async send<T>(
    topic: string,
    key: string,
    value: T,
    headers?: Record<string, string>
  ): Promise<void> {
    await this.connect();

    try {
      const result = await this.producer.send({
        topic,
        messages: [{
          key,
          value: JSON.stringify(value),
          headers
        }],
        acks: -1
      });

      console.log(`Message sent to ${topic}:`, result);
    } catch (error: any) {
      console.error('Send failed:', error);

      // Handle specific errors
      if (error.name === 'KafkaJSNumberOfRetriesExceeded') {
        // Retries exhausted - consider dead letter queue
        await this.sendToDeadLetter(topic, key, value, error);
      }

      throw error;
    }
  }

  private async sendToDeadLetter<T>(
    originalTopic: string,
    key: string,
    value: T,
    error: Error
  ): Promise<void> {
    try {
      await this.producer.send({
        topic: 'dead-letter',
        messages: [{
          key,
          value: JSON.stringify({
            originalTopic,
            key,
            value,
            error: error.message,
            timestamp: new Date().toISOString()
          })
        }]
      });
    } catch (dlqError) {
      console.error('Failed to send to dead letter queue:', dlqError);
    }
  }

  async disconnect(): Promise<void> {
    await this.producer.disconnect();
    this.connected = false;
  }
}

// ════════════════════════════════════════════════════════════════
// TRANSACTIONAL PRODUCER (Exactly-once)
// ════════════════════════════════════════════════════════════════

const transactionalProducer = kafka.producer({
  idempotent: true,
  transactionalId: 'order-processor-tx',
  maxInFlightRequests: 1  // Required for transactions
});

async function processWithTransaction(
  inputMessage: { orderId: string; status: string },
  consumer: any  // Consumer reference
): Promise<void> {
  const transaction = await transactionalProducer.transaction();

  try {
    // Send multiple messages atomically
    await transaction.send({
      topic: 'order-updates',
      messages: [{
        key: inputMessage.orderId,
        value: JSON.stringify({ ...inputMessage, processed: true })
      }]
    });

    await transaction.send({
      topic: 'order-analytics',
      messages: [{
        key: inputMessage.orderId,
        value: JSON.stringify({ event: 'processed', orderId: inputMessage.orderId })
      }]
    });

    // Commit consumer offsets as part of transaction
    await transaction.sendOffsets({
      consumerGroupId: 'order-processor',
      topics: [{
        topic: 'orders',
        partitions: [{
          partition: 0,
          offset: '100'  // The offset to commit
        }]
      }]
    });

    // Commit transaction
    await transaction.commit();
    console.log('Transaction committed');

  } catch (error) {
    // Abort transaction on any error
    await transaction.abort();
    console.error('Transaction aborted:', error);
    throw error;
  }
}
```

---

## 2. Kafka Consumer

```typescript
// ════════════════════════════════════════════════════════════════
// KAFKA CONSUMER
// ════════════════════════════════════════════════════════════════

import { Kafka, Consumer, EachMessagePayload, EachBatchPayload } from 'kafkajs';

const consumer: Consumer = kafka.consumer({
  groupId: 'order-processor',
  
  // Session timeout (consumer considered dead if no heartbeat)
  sessionTimeout: 30000,
  
  // Heartbeat interval
  heartbeatInterval: 3000,
  
  // Rebalance timeout
  rebalanceTimeout: 60000,
  
  // Max bytes per partition per fetch
  maxBytesPerPartition: 1048576,  // 1MB
  
  // Min bytes to fetch (wait until this much data)
  minBytes: 1,
  
  // Max wait time for minBytes
  maxWaitTimeInMs: 5000,
  
  // Auto commit (disable for manual control)
  // autoCommit: false
});

// Connect and subscribe
async function startConsumer(): Promise<void> {
  await consumer.connect();
  
  await consumer.subscribe({
    topic: 'orders',
    fromBeginning: false  // Start from latest (true = from beginning)
  });
  
  // Or subscribe to multiple topics
  // await consumer.subscribe({ topics: ['orders', 'payments'] });
  
  // Or pattern
  // await consumer.subscribe({ topics: /order-.*/ });
}

// ════════════════════════════════════════════════════════════════
// MESSAGE-BY-MESSAGE PROCESSING
// ════════════════════════════════════════════════════════════════

async function runEachMessage(): Promise<void> {
  await consumer.run({
    // Process each message individually
    eachMessage: async ({ topic, partition, message, heartbeat }: EachMessagePayload) => {
      const key = message.key?.toString();
      const value = message.value?.toString();
      const headers = message.headers;
      
      console.log({
        topic,
        partition,
        offset: message.offset,
        timestamp: message.timestamp,
        key,
        value: JSON.parse(value || '{}')
      });

      // Process the message
      await processOrder(JSON.parse(value || '{}'));

      // Send heartbeat for long-running processing
      // (prevents consumer from being considered dead)
      await heartbeat();
    }
  });
}

// ════════════════════════════════════════════════════════════════
// BATCH PROCESSING (More efficient)
// ════════════════════════════════════════════════════════════════

async function runEachBatch(): Promise<void> {
  await consumer.run({
    eachBatchAutoResolve: true,  // Auto-commit after batch
    
    eachBatch: async ({
      batch,
      resolveOffset,
      heartbeat,
      commitOffsetsIfNecessary,
      uncommittedOffsets,
      isRunning,
      isStale
    }: EachBatchPayload) => {
      console.log(`Processing batch of ${batch.messages.length} messages`);

      for (const message of batch.messages) {
        // Check if consumer is still running
        if (!isRunning() || isStale()) break;

        const value = JSON.parse(message.value?.toString() || '{}');
        
        await processOrder(value);

        // Mark this offset as processed
        resolveOffset(message.offset);

        // Heartbeat periodically
        await heartbeat();
      }
    }
  });
}

// ════════════════════════════════════════════════════════════════
// MANUAL OFFSET CONTROL
// ════════════════════════════════════════════════════════════════

async function runWithManualCommit(): Promise<void> {
  await consumer.run({
    autoCommit: false,  // Disable auto-commit
    
    eachMessage: async ({ topic, partition, message }) => {
      const value = JSON.parse(message.value?.toString() || '{}');
      
      try {
        // Process message
        await processOrder(value);
        
        // Commit offset after successful processing
        await consumer.commitOffsets([{
          topic,
          partition,
          offset: (parseInt(message.offset) + 1).toString()  // Next offset
        }]);
        
      } catch (error) {
        console.error('Processing failed:', error);
        // Don't commit - message will be reprocessed
        throw error;
      }
    }
  });
}

// ════════════════════════════════════════════════════════════════
// CONSUMER SERVICE WITH GRACEFUL SHUTDOWN
// ════════════════════════════════════════════════════════════════

class KafkaConsumerService {
  private consumer: Consumer;
  private running = false;

  constructor(kafka: Kafka, groupId: string) {
    this.consumer = kafka.consumer({ groupId });
  }

  async start(topics: string[], handler: (message: any) => Promise<void>): Promise<void> {
    await this.consumer.connect();
    
    for (const topic of topics) {
      await this.consumer.subscribe({ topic, fromBeginning: false });
    }

    this.running = true;

    await this.consumer.run({
      eachMessage: async ({ topic, partition, message }) => {
        if (!this.running) return;

        const value = JSON.parse(message.value?.toString() || '{}');
        
        try {
          await handler(value);
        } catch (error) {
          console.error(`Error processing message from ${topic}:${partition}:${message.offset}`, error);
          // Optionally: send to dead letter topic
          throw error;  // Will trigger retry or stop
        }
      }
    });
  }

  async stop(): Promise<void> {
    this.running = false;
    await this.consumer.stop();
    await this.consumer.disconnect();
    console.log('Consumer stopped gracefully');
  }
}

// Graceful shutdown
const consumerService = new KafkaConsumerService(kafka, 'order-processor');

process.on('SIGTERM', async () => {
  console.log('SIGTERM received, shutting down...');
  await consumerService.stop();
  process.exit(0);
});

process.on('SIGINT', async () => {
  console.log('SIGINT received, shutting down...');
  await consumerService.stop();
  process.exit(0);
});

// Start consumer
consumerService.start(['orders'], async (order) => {
  console.log('Processing order:', order.orderId);
  await processOrder(order);
});
```

---

## 3. Consumer Groups

```typescript
// ════════════════════════════════════════════════════════════════
// CONSUMER GROUP PATTERNS
// ════════════════════════════════════════════════════════════════

// Different consumer groups read independently
// Same group shares partitions

// Group 1: Real-time processing
const realtimeConsumer = kafka.consumer({ groupId: 'orders-realtime' });

// Group 2: Analytics (can process same data differently)
const analyticsConsumer = kafka.consumer({ groupId: 'orders-analytics' });

// Group 3: Data warehouse loading
const warehouseConsumer = kafka.consumer({ groupId: 'orders-warehouse' });

// All three groups receive ALL messages (independently)
// Within each group, partitions are distributed among consumers

// ════════════════════════════════════════════════════════════════
// CONSUMER GROUP COORDINATION
// ════════════════════════════════════════════════════════════════

// Multiple instances of same app share group ID
// Kafka coordinates partition assignment

// Instance 1: node index.js  (gets partitions 0, 1, 2)
// Instance 2: node index.js  (gets partitions 3, 4, 5)
// Instance 3: node index.js  (gets partitions 6, 7, 8)

// If Instance 2 dies:
// Instance 1: gets partitions 0, 1, 2, 3, 4
// Instance 3: gets partitions 5, 6, 7, 8

// ════════════════════════════════════════════════════════════════
// PARTITION ASSIGNMENT STRATEGIES
// ════════════════════════════════════════════════════════════════

const consumerWithStrategy = kafka.consumer({
  groupId: 'my-group',
  
  // Partition assignment strategy
  // 'RangeAssignor' - Ranges of partitions per consumer (default)
  // 'RoundRobinAssignor' - Round-robin distribution
  // 'CooperativeStickyAssignor' - Sticky with minimal reassignment
  partitionAssigners: [
    // Custom assigners or built-in
  ]
});

// ════════════════════════════════════════════════════════════════
// HANDLING REBALANCES
// ════════════════════════════════════════════════════════════════

consumer.on('consumer.group_join', ({ payload }) => {
  console.log('Joined consumer group:', payload);
});

consumer.on('consumer.rebalancing', ({ payload }) => {
  console.log('Rebalancing...', payload);
  // Pause processing, commit offsets
});

consumer.on('consumer.stop', () => {
  console.log('Consumer stopped');
});

// ════════════════════════════════════════════════════════════════
// MONITORING CONSUMER LAG
// ════════════════════════════════════════════════════════════════

import { Admin } from 'kafkajs';

const admin: Admin = kafka.admin();

async function getConsumerLag(groupId: string): Promise<void> {
  await admin.connect();
  
  // Get group offsets
  const groupOffsets = await admin.fetchOffsets({
    groupId,
    topics: ['orders']
  });

  // Get topic offsets (latest)
  const topicOffsets = await admin.fetchTopicOffsets('orders');

  // Calculate lag
  for (const partition of groupOffsets[0].partitions) {
    const consumerOffset = parseInt(partition.offset);
    const latestOffset = parseInt(
      topicOffsets.find(t => t.partition === partition.partition)?.offset || '0'
    );
    
    const lag = latestOffset - consumerOffset;
    console.log(`Partition ${partition.partition}: lag = ${lag}`);
  }

  await admin.disconnect();
}

// ════════════════════════════════════════════════════════════════
// SCALING CONSUMERS
// ════════════════════════════════════════════════════════════════

/*
Scaling rules:
1. Max effective consumers = number of partitions
2. More consumers than partitions = some idle (standby)
3. Fewer consumers than partitions = each gets multiple

Example: Topic with 6 partitions

1 consumer:  [P0, P1, P2, P3, P4, P5]
2 consumers: [P0, P1, P2] [P3, P4, P5]
3 consumers: [P0, P1] [P2, P3] [P4, P5]
6 consumers: [P0] [P1] [P2] [P3] [P4] [P5]
8 consumers: [P0] [P1] [P2] [P3] [P4] [P5] [idle] [idle]

To scale beyond 6, increase partitions first.
*/

// Check current assignment
async function describeGroup(groupId: string): Promise<void> {
  await admin.connect();
  
  const groups = await admin.describeGroups([groupId]);
  
  for (const group of groups.groups) {
    console.log(`Group: ${group.groupId}`);
    console.log(`State: ${group.state}`);
    console.log(`Members:`);
    
    for (const member of group.members) {
      const assignment = member.memberAssignment;
      console.log(`  - ${member.memberId}: ${assignment}`);
    }
  }

  await admin.disconnect();
}
```

---

## 4. Partitioning Strategies

```typescript
// ════════════════════════════════════════════════════════════════
// PARTITIONING STRATEGIES
// ════════════════════════════════════════════════════════════════

import { Partitioners } from 'kafkajs';

// Default partitioner: murmur2 hash of key
const defaultPartitioner = Partitioners.DefaultPartitioner;

// Legacy partitioner: Java-compatible
const legacyPartitioner = Partitioners.LegacyPartitioner;

// ════════════════════════════════════════════════════════════════
// CUSTOM PARTITIONER
// ════════════════════════════════════════════════════════════════

import { PartitionerArgs } from 'kafkajs';

// Custom partitioner example
const customPartitioner = () => {
  return ({ topic, partitionMetadata, message }: PartitionerArgs) => {
    const numPartitions = partitionMetadata.length;
    const key = message.key?.toString();
    
    if (!key) {
      // No key: round-robin
      return Math.floor(Math.random() * numPartitions);
    }
    
    // Custom logic based on key structure
    // Example: route by region prefix
    if (key.startsWith('us-')) {
      return 0;  // US events to partition 0
    } else if (key.startsWith('eu-')) {
      return 1;  // EU events to partition 1
    } else if (key.startsWith('asia-')) {
      return 2;  // Asia events to partition 2
    }
    
    // Default: hash-based
    return Math.abs(hashCode(key)) % numPartitions;
  };
};

function hashCode(str: string): number {
  let hash = 0;
  for (let i = 0; i < str.length; i++) {
    hash = ((hash << 5) - hash) + str.charCodeAt(i);
    hash = hash & hash;
  }
  return hash;
}

// Use custom partitioner
const producerWithCustomPartitioner = kafka.producer({
  createPartitioner: customPartitioner
});

// ════════════════════════════════════════════════════════════════
// PARTITION KEY STRATEGIES
// ════════════════════════════════════════════════════════════════

// Strategy 1: Entity ID (most common)
// All events for same entity go to same partition
await producer.send({
  topic: 'orders',
  messages: [{
    key: order.customerId,  // Customer events ordered
    value: JSON.stringify(order)
  }]
});

// Strategy 2: Composite key
// More granular ordering
await producer.send({
  topic: 'user-actions',
  messages: [{
    key: `${userId}:${sessionId}`,  // Per-session ordering
    value: JSON.stringify(action)
  }]
});

// Strategy 3: Time-based partition key
// For time-series data
await producer.send({
  topic: 'metrics',
  messages: [{
    key: `${metric.source}:${getHourBucket(metric.timestamp)}`,
    value: JSON.stringify(metric)
  }]
});

function getHourBucket(timestamp: Date): string {
  const d = new Date(timestamp);
  return `${d.getFullYear()}-${d.getMonth()}-${d.getDate()}-${d.getHours()}`;
}

// Strategy 4: No key (round-robin)
// Maximum parallelism, no ordering guarantee
await producer.send({
  topic: 'logs',
  messages: [{
    // No key - distributed evenly
    value: JSON.stringify(logEntry)
  }]
});

// ════════════════════════════════════════════════════════════════
// AVOIDING HOT PARTITIONS
// ════════════════════════════════════════════════════════════════

// Problem: Uneven key distribution
// If 80% of events have key="USA", partition for USA is overloaded

// Solution 1: Add random suffix for high-volume keys
function getBalancedKey(entityId: string, highVolumeEntities: Set<string>): string {
  if (highVolumeEntities.has(entityId)) {
    // Add random suffix to distribute across partitions
    const suffix = Math.floor(Math.random() * 10);
    return `${entityId}:${suffix}`;
  }
  return entityId;
}

// Solution 2: Salting with sub-keys
function getSaltedKey(customerId: string, orderId: string): string {
  // Distribute within customer by order
  return `${customerId}:${hashCode(orderId) % 10}`;
}

// Solution 3: Separate topics for high-volume entities
async function routeEvent(event: any): Promise<void> {
  const highVolumeCustomers = ['customer-big-corp', 'customer-enterprise'];
  
  if (highVolumeCustomers.includes(event.customerId)) {
    // Dedicated topic with more partitions
    await producer.send({
      topic: 'orders-high-volume',
      messages: [{ key: event.orderId, value: JSON.stringify(event) }]
    });
  } else {
    await producer.send({
      topic: 'orders',
      messages: [{ key: event.customerId, value: JSON.stringify(event) }]
    });
  }
}
```

---

## 5. Exactly-Once Semantics

```typescript
// ════════════════════════════════════════════════════════════════
// EXACTLY-ONCE SEMANTICS (EOS)
// ════════════════════════════════════════════════════════════════

/*
Three levels of delivery semantics:

1. AT-MOST-ONCE (may lose messages)
   - Fire and forget
   - acks=0
   
2. AT-LEAST-ONCE (may duplicate messages)  [Default]
   - Retry on failure
   - Consumer processes, then commits
   - If crash after process, before commit: reprocess
   
3. EXACTLY-ONCE (no loss, no duplicates)
   - Idempotent producer + transactions
   - Atomic read-process-write
*/

// ════════════════════════════════════════════════════════════════
// IDEMPOTENT PRODUCER
// ════════════════════════════════════════════════════════════════

// Idempotent producer prevents duplicates within single partition
const idempotentProducer = kafka.producer({
  idempotent: true,  // Enable idempotent producer
  maxInFlightRequests: 5  // Up to 5 with idempotent
});

// With idempotent producer:
// - Each message gets sequence number
// - Broker detects and rejects duplicates
// - Safe retries

// ════════════════════════════════════════════════════════════════
// TRANSACTIONAL PRODUCER (Cross-partition EOS)
// ════════════════════════════════════════════════════════════════

const transactionalProducer = kafka.producer({
  idempotent: true,
  transactionalId: 'order-processor-tx-1',  // Unique per producer instance
  maxInFlightRequests: 1  // Required for transactions
});

await transactionalProducer.connect();

async function processOrderTransactionally(order: any): Promise<void> {
  const transaction = await transactionalProducer.transaction();
  
  try {
    // All or nothing: these messages are atomic
    await transaction.send({
      topic: 'processed-orders',
      messages: [{ key: order.orderId, value: JSON.stringify(order) }]
    });
    
    await transaction.send({
      topic: 'inventory-updates',
      messages: [{ 
        key: order.productId, 
        value: JSON.stringify({ action: 'decrement', quantity: order.quantity })
      }]
    });
    
    await transaction.send({
      topic: 'notifications',
      messages: [{ 
        key: order.customerId, 
        value: JSON.stringify({ type: 'order_confirmed', orderId: order.orderId })
      }]
    });
    
    await transaction.commit();
    
  } catch (error) {
    await transaction.abort();
    throw error;
  }
}

// ════════════════════════════════════════════════════════════════
// CONSUME-TRANSFORM-PRODUCE PATTERN
// ════════════════════════════════════════════════════════════════

// Exactly-once: read from input, process, write to output atomically

const inputConsumer = kafka.consumer({
  groupId: 'processor-group',
  // Read committed: only see committed transactional messages
  readUncommitted: false
});

const outputProducer = kafka.producer({
  idempotent: true,
  transactionalId: 'processor-tx'
});

await inputConsumer.connect();
await outputProducer.connect();

await inputConsumer.subscribe({ topic: 'input-events', fromBeginning: false });

await inputConsumer.run({
  autoCommit: false,  // Manual commit as part of transaction
  
  eachMessage: async ({ topic, partition, message }) => {
    const transaction = await outputProducer.transaction();
    
    try {
      // Process message
      const input = JSON.parse(message.value?.toString() || '{}');
      const output = await processEvent(input);
      
      // Write output
      await transaction.send({
        topic: 'output-events',
        messages: [{ key: input.id, value: JSON.stringify(output) }]
      });
      
      // Commit consumer offset as part of transaction
      await transaction.sendOffsets({
        consumerGroupId: 'processor-group',
        topics: [{
          topic,
          partitions: [{
            partition,
            offset: (parseInt(message.offset) + 1).toString()
          }]
        }]
      });
      
      await transaction.commit();
      
    } catch (error) {
      await transaction.abort();
      throw error;
    }
  }
});

// ════════════════════════════════════════════════════════════════
// IDEMPOTENT CONSUMER (Alternative approach)
// ════════════════════════════════════════════════════════════════

// If transactional overhead too high, make consumer idempotent

interface ProcessedMessage {
  id: string;
  processedAt: Date;
}

const processedMessages = new Map<string, ProcessedMessage>();

async function idempotentProcess(message: any): Promise<void> {
  const messageId = message.id;
  
  // Check if already processed
  if (processedMessages.has(messageId)) {
    console.log(`Duplicate message ${messageId}, skipping`);
    return;
  }
  
  // Process
  await processEvent(message);
  
  // Mark as processed (in production, use Redis or database)
  processedMessages.set(messageId, {
    id: messageId,
    processedAt: new Date()
  });
}

// Database-backed idempotency
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();

async function idempotentProcessWithDb(message: any): Promise<void> {
  const messageId = message.id;
  
  // Use upsert for idempotency
  const result = await prisma.processedMessage.upsert({
    where: { messageId },
    update: {},  // No update if exists
    create: {
      messageId,
      processedAt: new Date()
    }
  });
  
  // If newly created, process
  if (result.createdAt.getTime() === result.updatedAt?.getTime()) {
    await processEvent(message);
  }
}
```

---

## 6. Event Replay

```typescript
// ════════════════════════════════════════════════════════════════
// EVENT REPLAY CAPABILITIES
// ════════════════════════════════════════════════════════════════

import { Admin, SeekEntry } from 'kafkajs';

const admin = kafka.admin();

// ════════════════════════════════════════════════════════════════
// RESET CONSUMER OFFSETS
// ════════════════════════════════════════════════════════════════

// Reset to beginning (replay all events)
async function resetToBeginning(groupId: string, topic: string): Promise<void> {
  await admin.connect();
  
  // Get earliest offsets
  const offsets = await admin.fetchTopicOffsets(topic);
  const earliestOffsets = offsets.map(o => ({
    partition: o.partition,
    offset: o.low  // Earliest available offset
  }));
  
  // Reset consumer group offsets
  await admin.setOffsets({
    groupId,
    topic,
    partitions: earliestOffsets
  });
  
  console.log(`Reset ${groupId} to beginning of ${topic}`);
  await admin.disconnect();
}

// Reset to specific timestamp
async function resetToTimestamp(
  groupId: string, 
  topic: string, 
  timestamp: number
): Promise<void> {
  await admin.connect();
  
  // Get offsets for timestamp
  const offsets = await admin.fetchTopicOffsetsByTimestamp(topic, timestamp);
  
  const partitionOffsets = offsets.map(o => ({
    partition: o.partition,
    offset: o.offset
  }));
  
  await admin.setOffsets({
    groupId,
    topic,
    partitions: partitionOffsets
  });
  
  console.log(`Reset ${groupId} to timestamp ${new Date(timestamp).toISOString()}`);
  await admin.disconnect();
}

// Reset to latest (skip all existing)
async function resetToLatest(groupId: string, topic: string): Promise<void> {
  await admin.connect();
  
  const offsets = await admin.fetchTopicOffsets(topic);
  const latestOffsets = offsets.map(o => ({
    partition: o.partition,
    offset: o.high  // Latest offset
  }));
  
  await admin.setOffsets({
    groupId,
    topic,
    partitions: latestOffsets
  });
  
  await admin.disconnect();
}

// ════════════════════════════════════════════════════════════════
// PROGRAMMATIC SEEK
// ════════════════════════════════════════════════════════════════

// Seek within running consumer
const seekableConsumer = kafka.consumer({ groupId: 'replay-group' });

await seekableConsumer.connect();
await seekableConsumer.subscribe({ topic: 'orders', fromBeginning: false });

// Before running, seek to specific offsets
seekableConsumer.seek({
  topic: 'orders',
  partition: 0,
  offset: '1000'  // Start from offset 1000
});

await seekableConsumer.run({
  eachMessage: async ({ message, partition }) => {
    console.log(`Processing: partition=${partition}, offset=${message.offset}`);
    // Process message
  }
});

// ════════════════════════════════════════════════════════════════
// REPLAY SERVICE
// ════════════════════════════════════════════════════════════════

interface ReplayRequest {
  groupId: string;
  topic: string;
  startTimestamp?: number;
  endTimestamp?: number;
  startOffset?: string;
  endOffset?: string;
}

class EventReplayService {
  private admin: Admin;
  
  constructor(kafka: Kafka) {
    this.admin = kafka.admin();
  }

  async replayFromTimestamp(
    groupId: string,
    topic: string,
    startTimestamp: number,
    endTimestamp?: number
  ): Promise<void> {
    await this.admin.connect();

    // Create new consumer group for replay
    const replayGroupId = `${groupId}-replay-${Date.now()}`;
    
    // Get start offsets
    const startOffsets = await this.admin.fetchTopicOffsetsByTimestamp(
      topic, 
      startTimestamp
    );

    // Set initial offsets
    await this.admin.setOffsets({
      groupId: replayGroupId,
      topic,
      partitions: startOffsets.map(o => ({
        partition: o.partition,
        offset: o.offset
      }))
    });

    // Consumer for replay
    const replayConsumer = kafka.consumer({ groupId: replayGroupId });
    await replayConsumer.connect();
    await replayConsumer.subscribe({ topic });

    // Get end offsets if specified
    let endOffsets: Map<number, string> | undefined;
    if (endTimestamp) {
      const ends = await this.admin.fetchTopicOffsetsByTimestamp(topic, endTimestamp);
      endOffsets = new Map(ends.map(o => [o.partition, o.offset]));
    }

    await replayConsumer.run({
      eachMessage: async ({ topic, partition, message }) => {
        // Check if we've reached end
        if (endOffsets) {
          const endOffset = endOffsets.get(partition);
          if (endOffset && parseInt(message.offset) >= parseInt(endOffset)) {
            console.log(`Reached end of replay for partition ${partition}`);
            return;
          }
        }

        // Process for replay
        await this.processReplayMessage(message);
      }
    });

    await this.admin.disconnect();
  }

  private async processReplayMessage(message: any): Promise<void> {
    const value = JSON.parse(message.value?.toString() || '{}');
    console.log('Replaying:', value);
    // Reprocess the event
  }
}

// ════════════════════════════════════════════════════════════════
// COMPACTED TOPICS (State/Changelog)
// ════════════════════════════════════════════════════════════════

// Compacted topics keep only the latest value per key
// Useful for: changelogs, state snapshots, lookup tables

// Create compacted topic
await admin.createTopics({
  topics: [{
    topic: 'user-profiles',
    numPartitions: 10,
    replicationFactor: 3,
    configEntries: [
      { name: 'cleanup.policy', value: 'compact' },
      { name: 'min.cleanable.dirty.ratio', value: '0.5' },
      { name: 'segment.ms', value: '86400000' }  // 24 hours
    ]
  }]
});

// Usage: Update user profile
await producer.send({
  topic: 'user-profiles',
  messages: [{
    key: 'user-123',  // Same key = latest wins after compaction
    value: JSON.stringify({ 
      userId: 'user-123',
      name: 'John Doe',
      email: 'john@example.com',
      updatedAt: new Date().toISOString()
    })
  }]
});

// Delete user (tombstone)
await producer.send({
  topic: 'user-profiles',
  messages: [{
    key: 'user-123',
    value: null  // Tombstone - will be removed after compaction
  }]
});

// Consumer reading compacted topic gets latest state
await consumer.subscribe({ topic: 'user-profiles', fromBeginning: true });
// On restart, consumer replays all keys with their latest values
```

---

## 7. Schema Registry

```typescript
// ════════════════════════════════════════════════════════════════
// SCHEMA REGISTRY (Avro/Protobuf/JSON Schema)
// ════════════════════════════════════════════════════════════════

// Schema Registry ensures:
// - Producers and consumers agree on schema
// - Schema evolution (backward/forward compatibility)
// - Efficient serialization (schema ID instead of full schema)

import { SchemaRegistry, SchemaType } from '@kafkajs/confluent-schema-registry';

const registry = new SchemaRegistry({
  host: 'http://schema-registry:8081'
});

// ════════════════════════════════════════════════════════════════
// AVRO SCHEMA
// ════════════════════════════════════════════════════════════════

const avroSchema = {
  type: 'record',
  name: 'Order',
  namespace: 'com.example.orders',
  fields: [
    { name: 'orderId', type: 'string' },
    { name: 'customerId', type: 'string' },
    { name: 'amount', type: 'double' },
    { name: 'currency', type: 'string', default: 'USD' },
    { name: 'timestamp', type: 'long', logicalType: 'timestamp-millis' },
    { 
      name: 'items', 
      type: { 
        type: 'array', 
        items: {
          type: 'record',
          name: 'OrderItem',
          fields: [
            { name: 'productId', type: 'string' },
            { name: 'quantity', type: 'int' },
            { name: 'price', type: 'double' }
          ]
        }
      }
    }
  ]
};

// Register schema
async function registerSchema(): Promise<number> {
  const { id } = await registry.register({
    type: SchemaType.AVRO,
    schema: JSON.stringify(avroSchema)
  }, {
    subject: 'orders-value'  // topic-value convention
  });
  
  return id;
}

// ════════════════════════════════════════════════════════════════
// PRODUCER WITH SCHEMA REGISTRY
// ════════════════════════════════════════════════════════════════

async function sendWithSchema(order: any): Promise<void> {
  // Encode with schema registry
  const encodedValue = await registry.encode(schemaId, order);
  
  await producer.send({
    topic: 'orders',
    messages: [{
      key: order.customerId,
      value: encodedValue  // Binary Avro
    }]
  });
}

// Auto-register schema and send
async function sendWithAutoRegister(order: any): Promise<void> {
  const schemaId = await registry.register({
    type: SchemaType.AVRO,
    schema: JSON.stringify(avroSchema)
  }, { subject: 'orders-value' });
  
  const encodedValue = await registry.encode(schemaId.id, order);
  
  await producer.send({
    topic: 'orders',
    messages: [{ key: order.customerId, value: encodedValue }]
  });
}

// ════════════════════════════════════════════════════════════════
// CONSUMER WITH SCHEMA REGISTRY
// ════════════════════════════════════════════════════════════════

await consumer.run({
  eachMessage: async ({ message }) => {
    // Decode automatically (schema ID embedded in message)
    const order = await registry.decode(message.value!);
    
    console.log('Received order:', order);
    // order is typed according to schema
  }
});

// ════════════════════════════════════════════════════════════════
// SCHEMA EVOLUTION
// ════════════════════════════════════════════════════════════════

// Compatibility modes:
// - BACKWARD: New schema can read old data (add fields with defaults)
// - FORWARD: Old schema can read new data (remove optional fields)
// - FULL: Both backward and forward compatible
// - NONE: No compatibility check

// Set compatibility mode
async function setCompatibility(subject: string): Promise<void> {
  await fetch(`http://schema-registry:8081/config/${subject}`, {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ compatibility: 'BACKWARD' })
  });
}

// Evolved schema (backward compatible)
const evolvedSchema = {
  type: 'record',
  name: 'Order',
  namespace: 'com.example.orders',
  fields: [
    { name: 'orderId', type: 'string' },
    { name: 'customerId', type: 'string' },
    { name: 'amount', type: 'double' },
    { name: 'currency', type: 'string', default: 'USD' },
    { name: 'timestamp', type: 'long', logicalType: 'timestamp-millis' },
    { name: 'items', type: { type: 'array', items: 'OrderItem' }},
    // NEW FIELD: Has default, so backward compatible
    { name: 'shippingAddress', type: ['null', 'string'], default: null },
    // NEW FIELD: Has default
    { name: 'priority', type: 'string', default: 'normal' }
  ]
};

// ════════════════════════════════════════════════════════════════
// JSON SCHEMA (Alternative)
// ════════════════════════════════════════════════════════════════

const jsonSchema = {
  $schema: 'http://json-schema.org/draft-07/schema#',
  title: 'Order',
  type: 'object',
  required: ['orderId', 'customerId', 'amount'],
  properties: {
    orderId: { type: 'string' },
    customerId: { type: 'string' },
    amount: { type: 'number' },
    currency: { type: 'string', default: 'USD' },
    timestamp: { type: 'integer' }
  }
};

await registry.register({
  type: SchemaType.JSON,
  schema: JSON.stringify(jsonSchema)
}, { subject: 'orders-json-value' });
```

---

## 8. Kafka Streams

```typescript
// ════════════════════════════════════════════════════════════════
// KAFKA STREAMS-LIKE PROCESSING (Node.js patterns)
// ════════════════════════════════════════════════════════════════

// Note: KafkaJS doesn't have full Kafka Streams API
// For true Kafka Streams, use Java or ksqlDB
// Below are patterns for stream processing in Node.js

// ════════════════════════════════════════════════════════════════
// STATELESS TRANSFORMATIONS
// ════════════════════════════════════════════════════════════════

// Filter
async function filterOrders(minAmount: number): Promise<void> {
  await consumer.run({
    eachMessage: async ({ message }) => {
      const order = JSON.parse(message.value?.toString() || '{}');
      
      // Filter: only process high-value orders
      if (order.amount >= minAmount) {
        await producer.send({
          topic: 'high-value-orders',
          messages: [{ key: order.orderId, value: JSON.stringify(order) }]
        });
      }
    }
  });
}

// Map/Transform
async function enrichOrders(): Promise<void> {
  await consumer.run({
    eachMessage: async ({ message }) => {
      const order = JSON.parse(message.value?.toString() || '{}');
      
      // Enrich with additional data
      const customer = await fetchCustomer(order.customerId);
      const enrichedOrder = {
        ...order,
        customerName: customer.name,
        customerTier: customer.tier,
        enrichedAt: new Date().toISOString()
      };
      
      await producer.send({
        topic: 'enriched-orders',
        messages: [{ key: order.orderId, value: JSON.stringify(enrichedOrder) }]
      });
    }
  });
}

// ════════════════════════════════════════════════════════════════
// STATEFUL AGGREGATIONS
// ════════════════════════════════════════════════════════════════

// Count by key
import { Redis } from 'ioredis';
const redis = new Redis();

async function countOrdersByCustomer(): Promise<void> {
  await consumer.run({
    eachMessage: async ({ message }) => {
      const order = JSON.parse(message.value?.toString() || '{}');
      
      // Increment count in Redis
      const count = await redis.incr(`order-count:${order.customerId}`);
      
      // Emit updated count
      await producer.send({
        topic: 'customer-order-counts',
        messages: [{
          key: order.customerId,
          value: JSON.stringify({
            customerId: order.customerId,
            orderCount: count,
            updatedAt: new Date().toISOString()
          })
        }]
      });
    }
  });
}

// Windowed aggregation
interface WindowedCount {
  windowStart: number;
  windowEnd: number;
  key: string;
  count: number;
  sum: number;
}

async function windowedAggregation(windowSizeMs: number): Promise<void> {
  const windows = new Map<string, WindowedCount>();
  
  await consumer.run({
    eachMessage: async ({ message }) => {
      const order = JSON.parse(message.value?.toString() || '{}');
      const timestamp = parseInt(message.timestamp);
      
      // Calculate window
      const windowStart = Math.floor(timestamp / windowSizeMs) * windowSizeMs;
      const windowEnd = windowStart + windowSizeMs;
      const windowKey = `${order.customerId}:${windowStart}`;
      
      // Update window
      let window = windows.get(windowKey);
      if (!window) {
        window = {
          windowStart,
          windowEnd,
          key: order.customerId,
          count: 0,
          sum: 0
        };
        windows.set(windowKey, window);
      }
      
      window.count++;
      window.sum += order.amount;
      
      // Check if window is complete (time-based, simplified)
      if (Date.now() > windowEnd) {
        // Emit completed window
        await producer.send({
          topic: 'order-aggregates-hourly',
          messages: [{
            key: windowKey,
            value: JSON.stringify(window)
          }]
        });
        
        windows.delete(windowKey);
      }
    }
  });
}

// ════════════════════════════════════════════════════════════════
// JOIN PATTERNS
// ════════════════════════════════════════════════════════════════

// Simple join using lookup table (compacted topic)
const userCache = new Map<string, any>();

// First, populate cache from compacted topic
async function loadUserCache(): Promise<void> {
  const cacheConsumer = kafka.consumer({ groupId: 'cache-loader' });
  await cacheConsumer.connect();
  await cacheConsumer.subscribe({ topic: 'users', fromBeginning: true });
  
  await cacheConsumer.run({
    eachMessage: async ({ message }) => {
      const userId = message.key?.toString();
      if (message.value) {
        userCache.set(userId!, JSON.parse(message.value.toString()));
      } else {
        userCache.delete(userId!);  // Tombstone
      }
    }
  });
}

// Then, join orders with users
async function joinOrdersWithUsers(): Promise<void> {
  await consumer.run({
    eachMessage: async ({ message }) => {
      const order = JSON.parse(message.value?.toString() || '{}');
      const user = userCache.get(order.customerId);
      
      const joinedOrder = {
        ...order,
        user: user || null
      };
      
      await producer.send({
        topic: 'orders-with-users',
        messages: [{
          key: order.orderId,
          value: JSON.stringify(joinedOrder)
        }]
      });
    }
  });
}
```

---

## 9. Common Pitfalls

```typescript
// ════════════════════════════════════════════════════════════════
// KAFKA PITFALLS
// ════════════════════════════════════════════════════════════════

// ❌ PITFALL 1: Expecting global ordering
// Problem: Kafka only guarantees ordering within partition

// Bad: Expecting all events in order
await producer.send({
  topic: 'events',
  messages: [
    { value: 'event-1' },  // May go to partition 0
    { value: 'event-2' },  // May go to partition 1
    { value: 'event-3' }   // May go to partition 2
  ]
});
// Consumer may receive: event-2, event-1, event-3

// Good: Use partition key for related events
await producer.send({
  topic: 'events',
  messages: [
    { key: 'user-1', value: 'event-1' },  // Partition determined by key
    { key: 'user-1', value: 'event-2' },  // Same partition as event-1
    { key: 'user-1', value: 'event-3' }   // Same partition
  ]
});
// Events for user-1 are ordered

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 2: Auto-commit with slow processing
// Problem: Offset committed before processing complete

// Bad
await consumer.run({
  autoCommit: true,  // Default
  autoCommitInterval: 5000,
  
  eachMessage: async ({ message }) => {
    await slowDatabaseOperation(message);  // Takes 10 seconds
    // Offset may be committed before this finishes
    // If consumer crashes, message is lost
  }
});

// Good: Manual commit after processing
await consumer.run({
  autoCommit: false,
  
  eachMessage: async ({ topic, partition, message }) => {
    await slowDatabaseOperation(message);
    
    // Commit after successful processing
    await consumer.commitOffsets([{
      topic,
      partition,
      offset: (parseInt(message.offset) + 1).toString()
    }]);
  }
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 3: Too many partitions
// Problem: More consumers than partitions = idle consumers

// Bad: 100 partitions, but rarely more than 3 consumers
const producer = kafka.producer();
// 97 partitions unused, overhead for nothing

// Good: Plan for realistic scaling
// Rule of thumb: 2-3x expected peak consumers
// 10 partitions for 3-5 consumers is usually enough

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 4: Not handling rebalances
// Problem: Processing duplicate/missing messages during rebalance

// Bad: No handling
await consumer.run({
  eachMessage: async ({ message }) => {
    // During rebalance, partition assignment changes
    // May process same message twice or miss messages
  }
});

// Good: Handle rebalance events
consumer.on('consumer.rebalancing', async () => {
  // Commit processed offsets before rebalance
  await commitPendingOffsets();
  // Pause processing
  isPaused = true;
});

consumer.on('consumer.group_join', () => {
  // Resume processing
  isPaused = false;
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 5: Large messages
// Problem: Kafka has message size limits

// Bad: Sending huge payloads
await producer.send({
  topic: 'files',
  messages: [{
    value: JSON.stringify(hugeFile)  // 50MB
  }]
});
// Error: Message size larger than max.message.bytes

// Good: Store large data elsewhere, send reference
await producer.send({
  topic: 'files',
  messages: [{
    value: JSON.stringify({
      fileId: 'file-123',
      s3Url: 's3://bucket/files/file-123',
      size: 50000000
    })
  }]
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 6: Synchronous processing blocking consumer
// Problem: Consumer falls behind, appears dead

// Bad: Blocking processing
await consumer.run({
  eachMessage: async ({ message }) => {
    await verySlowOperation(message);  // 30 seconds
    // No heartbeat sent, consumer may be kicked from group
  }
});

// Good: Send heartbeats during long operations
await consumer.run({
  eachMessage: async ({ message, heartbeat }) => {
    for (const chunk of chunks) {
      await processChunk(chunk);
      await heartbeat();  // Keep consumer alive
    }
  }
});

// Better: Process asynchronously, commit later
const queue: any[] = [];
await consumer.run({
  eachMessage: async ({ message }) => {
    queue.push(JSON.parse(message.value?.toString() || '{}'));
    // Return quickly
  }
});

// Process queue in background
setInterval(async () => {
  while (queue.length > 0) {
    const item = queue.shift();
    await processItem(item);
  }
}, 100);

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 7: Not using idempotent producer
// Problem: Network issues cause duplicate messages

// Bad: Default producer
const producer = kafka.producer();
// Network error + retry = potential duplicate

// Good: Enable idempotence
const producer = kafka.producer({
  idempotent: true
});
// Kafka deduplicates based on producer ID and sequence number
```

---

## 10. Interview Questions

### Basic Questions

**Q: "What is Apache Kafka?"**
> "Distributed event streaming platform. Topics store ordered sequences of events in partitions. Producers publish, consumers subscribe in groups. High throughput (millions/sec), durable (replicated), supports replay. Used for real-time pipelines, event sourcing, log aggregation."

**Q: "What is a partition?"**
> "Topics are split into partitions for parallelism. Each partition is ordered, immutable log with sequential offsets. Messages with same key go to same partition. More partitions = more parallelism, but max consumers per group = partitions."

**Q: "What is a consumer group?"**
> "Coordinated set of consumers sharing topic workload. Each partition assigned to one consumer in group. Multiple groups independently consume same data. Enables parallel processing and fault tolerance through rebalancing."

### Intermediate Questions

**Q: "How does Kafka guarantee ordering?"**
> "Ordering only within partition, not across. Use partition key (e.g., user ID, order ID) - same key goes to same partition. Global ordering requires single partition (limits throughput). Design around per-entity ordering."

**Q: "What is offset and how is it managed?"**
> "Offset is message position in partition (sequential number). Consumer tracks progress via committed offsets. Auto-commit: periodic, simpler but at-most-once. Manual commit: after processing, at-least-once. Committed offsets survive restarts."

**Q: "Explain exactly-once semantics."**
> "Three levels: at-most-once (may lose), at-least-once (may duplicate, default), exactly-once (neither). EOS uses: idempotent producer (dedup within partition), transactions (atomic writes across partitions + offset commit). Higher complexity and slight overhead."

### Advanced Questions

**Q: "How would you design a real-time analytics pipeline with Kafka?"**
> "Events from sources → Kafka topics (partitioned by entity). Multiple consumer groups: real-time processing, batch loading, alerting. Schema Registry for evolution. Exactly-once for financial data. Compacted topics for state/lookups. Monitor lag, scale consumers to match partitions. 30-day retention for replay."

**Q: "How do you handle consumer lag?"**
> "Monitor lag metrics (offset diff). Causes: slow processing, too few consumers, insufficient partitions. Solutions: scale consumers (up to partition count), optimize processing (batching, async), increase partitions if needed. For temporary spikes: accept lag, catch up later. Alerts on sustained lag."

**Q: "Kafka vs other messaging systems?"**
> "vs RabbitMQ: Kafka = distributed log (ordered, durable, replay), high throughput. RabbitMQ = traditional broker (flexible routing, lower latency). vs Kinesis: Similar to Kafka, managed AWS service. vs Pulsar: Multi-tenancy, tiered storage. Choose Kafka for: high volume, event sourcing, multiple consumers on same data, replay."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                     KAFKA QUICK REFERENCE                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  PRODUCER:                                                      │
│  □ Use partition key for ordering                              │
│  □ Enable idempotent producer (avoid duplicates)               │
│  □ acks=-1 for durability, acks=1 for balanced                 │
│  □ Compression for high throughput                             │
│                                                                 │
│  CONSUMER:                                                      │
│  □ Consumer group for parallel processing                      │
│  □ Manual commit for at-least-once                             │
│  □ Send heartbeats during long processing                      │
│  □ Handle rebalances gracefully                                │
│                                                                 │
│  PARTITIONING:                                                  │
│  □ Same key → Same partition → Ordering                        │
│  □ Max consumers = num partitions                              │
│  □ Avoid hot partitions (uneven keys)                          │
│                                                                 │
│  RELIABILITY:                                                   │
│  □ Replication factor = 3 (production)                         │
│  □ min.insync.replicas = 2                                     │
│  □ acks = -1 (all replicas)                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

DELIVERY SEMANTICS:
┌────────────────────────────────────────────────────────────────┐
│ At-most-once   - acks=0, may lose                              │
│ At-least-once  - retry + commit after, may duplicate [default] │
│ Exactly-once   - idempotent + transactions, no loss/dup        │
└────────────────────────────────────────────────────────────────┘

TOPIC CONFIGURATION:
┌────────────────────────────────────────────────────────────────┐
│ partitions         - Parallelism unit (can only increase)      │
│ replication.factor - Copies across brokers (3 for prod)        │
│ retention.ms       - How long to keep (default 7 days)         │
│ cleanup.policy     - delete (default) or compact               │
└────────────────────────────────────────────────────────────────┘

CONSUMER GROUP RULES:
┌────────────────────────────────────────────────────────────────┐
│ • Each partition → one consumer in group                       │
│ • More consumers than partitions → some idle                   │
│ • Consumer dies → partitions redistributed                     │
│ • Multiple groups → each reads all messages                    │
└────────────────────────────────────────────────────────────────┘

OFFSET MANAGEMENT:
┌────────────────────────────────────────────────────────────────┐
│ Auto-commit: Periodic, simpler, at-most-once risk              │
│ Manual commit: After processing, at-least-once                 │
│ Transactional: Atomic with produce, exactly-once               │
│ Reset: earliest (replay), latest (skip), timestamp             │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

