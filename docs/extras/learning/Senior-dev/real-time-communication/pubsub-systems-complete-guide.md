# 📢 Pub/Sub Systems - Complete Guide

> A comprehensive guide to Publish/Subscribe systems - Redis Pub/Sub, Apache Kafka basics, event streaming, fan-out patterns, and real-world implementations.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Pub/Sub (Publish/Subscribe) is a messaging pattern where publishers send messages to channels without knowing who will receive them, and subscribers listen to channels without knowing who sent the messages - enabling complete decoupling between message producers and consumers."

### The 6 Key Concepts (Remember These!)
```
1. PUBLISHER      → Sends messages to topics/channels
2. SUBSCRIBER     → Receives messages from topics/channels
3. TOPIC/CHANNEL  → Named destination for messages
4. FAN-OUT        → One message delivered to many subscribers
5. DECOUPLING     → Publishers and subscribers don't know each other
6. FIRE-AND-FORGET→ Publisher doesn't wait for delivery confirmation
```

### Pub/Sub vs Message Queue
```
┌─────────────────────────────────────────────────────────────────┐
│              PUB/SUB vs MESSAGE QUEUE                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  MESSAGE QUEUE:                                                 │
│  ──────────────                                                 │
│  Producer ──▶ [Queue] ──▶ Consumer                             │
│                                                                 │
│  • Point-to-point                                              │
│  • One consumer per message                                    │
│  • Messages consumed and removed                               │
│  • Consumer acknowledgment                                     │
│  • Good for: Task distribution, work queues                    │
│                                                                 │
│  PUB/SUB:                                                       │
│  ───────                                                        │
│                     ┌──▶ Subscriber 1                          │
│  Publisher ──▶ Topic┼──▶ Subscriber 2                          │
│                     └──▶ Subscriber 3                          │
│                                                                 │
│  • One-to-many                                                 │
│  • All subscribers receive message                             │
│  • Messages broadcast, not consumed                            │
│  • No acknowledgment (usually)                                 │
│  • Good for: Events, notifications, real-time updates          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Pub/Sub Systems Comparison
```
┌─────────────────────────────────────────────────────────────────┐
│              PUB/SUB SYSTEMS COMPARISON                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  REDIS PUB/SUB                                                  │
│  • Fire-and-forget (no persistence)                            │
│  • Simple, fast, low latency                                   │
│  • No message history, no replay                               │
│  • Best for: Real-time notifications, cache invalidation       │
│                                                                 │
│  REDIS STREAMS                                                  │
│  • Persistent, consumer groups                                 │
│  • Message acknowledgment                                      │
│  • Replay capability                                           │
│  • Best for: Event sourcing lite, reliable messaging           │
│                                                                 │
│  APACHE KAFKA                                                   │
│  • Distributed log, high throughput                            │
│  • Retention (days/weeks), replay                              │
│  • Consumer groups, partitioning                               │
│  • Best for: Event streaming, analytics, high volume           │
│                                                                 │
│  GOOGLE PUB/SUB                                                 │
│  • Managed, global scale                                       │
│  • At-least-once, exactly-once                                 │
│  • Push and pull subscriptions                                 │
│  • Best for: GCP-native, global distribution                   │
│                                                                 │
│  AWS SNS                                                        │
│  • Managed, integrates with AWS                                │
│  • Push to HTTP, SQS, Lambda, email                            │
│  • Message filtering                                           │
│  • Best for: AWS-native, notifications                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Fan-out"** | "We use fan-out to broadcast events to all interested services" |
| **"Topic partitioning"** | "Kafka partitions topics for parallel consumption" |
| **"Consumer group"** | "Consumer groups ensure each message processed once per group" |
| **"At-most-once"** | "Redis Pub/Sub provides at-most-once delivery" |
| **"Event sourcing"** | "Kafka enables event sourcing with its retention capabilities" |
| **"Backpressure"** | "We implement backpressure when subscribers can't keep up" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Redis Pub/Sub latency | **< 1ms** | Fire-and-forget, in-memory |
| Kafka throughput | **Millions/sec** | Distributed, partitioned |
| Kafka retention | **7 days default** | Configurable to forever |
| SNS max message | **256 KB** | Use S3 for larger |
| Kafka partition count | **#consumers** | For parallelism |

### The "Wow" Statement (Memorize This!)
> "We use a hybrid pub/sub architecture. Redis Pub/Sub handles real-time notifications where millisecond latency matters and message loss is acceptable - like typing indicators. For critical events that need durability and replay, we use Kafka. Events are published to topics partitioned by entity ID for ordering guarantees. Consumer groups ensure each microservice processes events exactly once within the group. We retain events for 14 days enabling replay for debugging and new service bootstrapping. For cross-service communication, we follow the 'publish events, not commands' principle - services react to domain events rather than being directly called."

### Quick Architecture Drawing (Draw This!)
```
REDIS PUB/SUB:
Publisher ──▶ Channel ═══════╦═══════╦═══════╗
              "events"       ▼       ▼       ▼
                          Sub 1   Sub 2   Sub 3
                        (online)(online)(offline = missed)

KAFKA:
                         ┌─────────────────────┐
Publisher ──▶ Topic      │ Partition 0: [E1][E2][E3]
              "events"   │ Partition 1: [E4][E5]
                         │ Partition 2: [E6][E7][E8]
                         └─────────────────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              ▼                    ▼                    ▼
         Consumer A           Consumer B           Consumer C
         (Group: svc1)        (Group: svc1)        (Group: svc2)
         
         Each group gets ALL messages
         Within group, messages distributed across consumers
```

### Interview Rapid Fire (Practice These!)

**Q: "What is Pub/Sub?"**
> "Pattern where publishers send to topics, subscribers listen to topics. Complete decoupling - publisher doesn't know subscribers. Enables fan-out broadcasting."

**Q: "Redis Pub/Sub vs Kafka?"**
> "Redis: simple, fast, no persistence, fire-and-forget. Kafka: durable, high throughput, replay capability, consumer groups."

**Q: "What's a consumer group in Kafka?"**
> "Subscribers that share message processing. Each message delivered to one consumer per group. Multiple groups each get all messages."

**Q: "When would you use Pub/Sub vs message queue?"**
> "Pub/Sub: broadcast to many, event notifications. Queue: distribute work, one processor per task, guaranteed processing."

**Q: "What is fan-out?"**
> "One message delivered to multiple subscribers. Classic pub/sub pattern - publish once, everyone subscribed receives it."

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "What is Pub/Sub?"

**Junior Answer:**
> "It's when you publish messages and other services subscribe to get them."

**Senior Answer:**
> "Pub/Sub is a messaging pattern that completely decouples message producers from consumers. Publishers send messages to topics without knowing who, if anyone, will receive them. Subscribers express interest in topics and receive all messages published there.

Key characteristics:
1. **Fan-out**: One message reaches all subscribers
2. **Decoupling**: Publisher and subscriber have no direct dependency
3. **Dynamic**: Subscribers can join/leave without affecting publishers
4. **Asynchronous**: Publisher doesn't wait for subscribers

Different implementations have different guarantees:
- Redis Pub/Sub: At-most-once, no persistence
- Kafka: At-least-once, persistent, replayable
- SNS/GCP Pub/Sub: Managed, various delivery guarantees"

### When Asked: "Redis Pub/Sub vs Kafka - when to use each?"

**Junior Answer:**
> "Kafka for big data, Redis for simple stuff."

**Senior Answer:**
> "They solve different problems:

**Redis Pub/Sub** when you need:
- Real-time notifications (sub-millisecond)
- Cache invalidation across servers
- Presence/typing indicators
- Can tolerate message loss
- Simple implementation

**Kafka** when you need:
- Message durability and replay
- High throughput (millions/sec)
- Multiple consumer groups
- Event sourcing capabilities
- Ordering guarantees within partition
- Auditing/compliance requirements

The key distinction: Redis Pub/Sub is fire-and-forget - if no one is listening, the message is lost. Kafka persists messages for configured retention, enabling replay and ensuring no message loss even if consumers are temporarily down."

### Follow-up Questions to Expect:

| They Ask | You Answer |
|----------|------------|
| "What about Redis Streams?" | "Redis Streams adds persistence and consumer groups to Redis. Middle ground between Pub/Sub and Kafka. Good for moderate throughput with durability." |
| "How do you handle slow subscribers?" | "Depends on system. Kafka: consumers read at own pace from log. Redis Pub/Sub: slow subscribers may miss messages. Implement backpressure or buffering." |
| "How do you ensure ordering?" | "Kafka: partition by key (e.g., user ID) - ordering within partition. Redis Pub/Sub: no ordering guarantees across subscribers." |
| "What's the difference between topic and queue?" | "Topic: broadcast, all subscribers receive. Queue: work distribution, one consumer per message. Different patterns for different use cases." |

---

## 📚 Table of Contents

1. [Redis Pub/Sub](#1-redis-pubsub)
2. [Redis Streams](#2-redis-streams)
3. [Apache Kafka Basics](#3-apache-kafka-basics)
4. [AWS SNS](#4-aws-sns)
5. [Fan-Out Patterns](#5-fan-out-patterns)
6. [Event-Driven Architecture](#6-event-driven-architecture)
7. [Scaling Pub/Sub](#7-scaling-pubsub)
8. [Real-World Patterns](#8-real-world-patterns)
9. [Common Pitfalls](#9-common-pitfalls)
10. [Interview Questions](#10-interview-questions)

---

## 1. Redis Pub/Sub

### Basic Implementation

```typescript
// ════════════════════════════════════════════════════════════════
// REDIS PUB/SUB - BASIC IMPLEMENTATION
// ════════════════════════════════════════════════════════════════

import Redis from 'ioredis';

// Create separate clients for pub and sub (required!)
const publisher = new Redis(process.env.REDIS_URL);
const subscriber = new Redis(process.env.REDIS_URL);

// ════════════════════════════════════════════════════════════════
// PUBLISHER
// ════════════════════════════════════════════════════════════════

async function publish(channel: string, message: any): Promise<number> {
  const payload = JSON.stringify(message);
  // Returns number of subscribers that received the message
  return publisher.publish(channel, payload);
}

// Publish events
await publish('user:events', {
  type: 'user.created',
  userId: '123',
  timestamp: Date.now()
});

await publish('order:events', {
  type: 'order.placed',
  orderId: '456',
  userId: '123'
});

// ════════════════════════════════════════════════════════════════
// SUBSCRIBER
// ════════════════════════════════════════════════════════════════

// Subscribe to specific channel
subscriber.subscribe('user:events', (err, count) => {
  if (err) {
    console.error('Subscribe error:', err);
    return;
  }
  console.log(`Subscribed to ${count} channels`);
});

// Subscribe to multiple channels
subscriber.subscribe('user:events', 'order:events', 'notification:events');

// Pattern subscription (glob patterns)
subscriber.psubscribe('*:events');  // All event channels
subscriber.psubscribe('user:*');    // All user channels

// Handle messages
subscriber.on('message', (channel, message) => {
  const data = JSON.parse(message);
  console.log(`Received on ${channel}:`, data);
  
  // Route to handlers
  handleMessage(channel, data);
});

// Handle pattern messages
subscriber.on('pmessage', (pattern, channel, message) => {
  const data = JSON.parse(message);
  console.log(`Pattern ${pattern} matched ${channel}:`, data);
});

// ════════════════════════════════════════════════════════════════
// TYPED PUB/SUB WRAPPER
// ════════════════════════════════════════════════════════════════

type EventHandler<T> = (data: T) => void | Promise<void>;

class TypedPubSub<Events extends Record<string, any>> {
  private publisher: Redis;
  private subscriber: Redis;
  private handlers: Map<string, Set<EventHandler<any>>> = new Map();

  constructor(redisUrl: string) {
    this.publisher = new Redis(redisUrl);
    this.subscriber = new Redis(redisUrl);
    this.setupSubscriber();
  }

  private setupSubscriber() {
    this.subscriber.on('message', async (channel, message) => {
      const handlers = this.handlers.get(channel);
      if (!handlers) return;

      const data = JSON.parse(message);
      
      for (const handler of handlers) {
        try {
          await handler(data);
        } catch (error) {
          console.error(`Handler error on ${channel}:`, error);
        }
      }
    });
  }

  async publish<K extends keyof Events>(
    channel: K,
    data: Events[K]
  ): Promise<number> {
    return this.publisher.publish(
      String(channel),
      JSON.stringify(data)
    );
  }

  subscribe<K extends keyof Events>(
    channel: K,
    handler: EventHandler<Events[K]>
  ): () => void {
    const channelStr = String(channel);

    if (!this.handlers.has(channelStr)) {
      this.handlers.set(channelStr, new Set());
      this.subscriber.subscribe(channelStr);
    }

    this.handlers.get(channelStr)!.add(handler);

    // Return unsubscribe function
    return () => {
      this.handlers.get(channelStr)?.delete(handler);
      if (this.handlers.get(channelStr)?.size === 0) {
        this.subscriber.unsubscribe(channelStr);
        this.handlers.delete(channelStr);
      }
    };
  }

  async close(): Promise<void> {
    await this.publisher.quit();
    await this.subscriber.quit();
  }
}

// Usage with types
interface AppEvents {
  'user:created': { userId: string; email: string };
  'order:placed': { orderId: string; userId: string; total: number };
  'notification:sent': { notificationId: string; userId: string };
}

const pubsub = new TypedPubSub<AppEvents>(process.env.REDIS_URL!);

// Type-safe subscription
const unsubscribe = pubsub.subscribe('user:created', async (data) => {
  // data is typed as { userId: string; email: string }
  console.log('User created:', data.userId, data.email);
  await sendWelcomeEmail(data.email);
});

// Type-safe publishing
await pubsub.publish('user:created', {
  userId: '123',
  email: 'user@example.com'
});

// Cleanup
unsubscribe();
```

### Real-Time Notifications with Redis Pub/Sub

```typescript
// ════════════════════════════════════════════════════════════════
// REAL-TIME NOTIFICATIONS SYSTEM
// ════════════════════════════════════════════════════════════════

import { Server, Socket } from 'socket.io';

class NotificationService {
  private publisher: Redis;
  private subscriber: Redis;
  private io: Server;
  private userSockets: Map<string, Set<string>> = new Map(); // userId -> socketIds

  constructor(io: Server, redisUrl: string) {
    this.io = io;
    this.publisher = new Redis(redisUrl);
    this.subscriber = new Redis(redisUrl);
    this.setupSubscriber();
    this.setupSocketHandlers();
  }

  private setupSubscriber() {
    // Subscribe to user notification channels with pattern
    this.subscriber.psubscribe('notifications:user:*');

    this.subscriber.on('pmessage', (pattern, channel, message) => {
      // Extract userId from channel: notifications:user:123
      const userId = channel.split(':')[2];
      const notification = JSON.parse(message);
      
      // Send to user's connected sockets
      this.sendToUser(userId, notification);
    });
  }

  private setupSocketHandlers() {
    this.io.on('connection', (socket: Socket) => {
      const userId = socket.data.userId;
      
      // Track user's socket
      if (!this.userSockets.has(userId)) {
        this.userSockets.set(userId, new Set());
      }
      this.userSockets.get(userId)!.add(socket.id);

      socket.on('disconnect', () => {
        this.userSockets.get(userId)?.delete(socket.id);
        if (this.userSockets.get(userId)?.size === 0) {
          this.userSockets.delete(userId);
        }
      });
    });
  }

  private sendToUser(userId: string, notification: any) {
    const socketIds = this.userSockets.get(userId);
    if (!socketIds) return;

    for (const socketId of socketIds) {
      this.io.to(socketId).emit('notification', notification);
    }
  }

  // Call this from any service to send notification
  async notify(userId: string, notification: {
    type: string;
    title: string;
    body: string;
    data?: any;
  }): Promise<void> {
    const channel = `notifications:user:${userId}`;
    const message = {
      id: `notif-${Date.now()}`,
      ...notification,
      timestamp: new Date().toISOString()
    };

    await this.publisher.publish(channel, JSON.stringify(message));
  }

  // Broadcast to all users
  async broadcast(notification: any): Promise<void> {
    await this.publisher.publish('notifications:broadcast', JSON.stringify(notification));
  }
}

// ════════════════════════════════════════════════════════════════
// CACHE INVALIDATION PATTERN
// ════════════════════════════════════════════════════════════════

class DistributedCache {
  private redis: Redis;
  private subscriber: Redis;
  private localCache: Map<string, any> = new Map();
  private readonly CHANNEL = 'cache:invalidation';

  constructor(redisUrl: string) {
    this.redis = new Redis(redisUrl);
    this.subscriber = new Redis(redisUrl);
    this.setupInvalidationListener();
  }

  private setupInvalidationListener() {
    this.subscriber.subscribe(this.CHANNEL);
    
    this.subscriber.on('message', (channel, message) => {
      if (channel !== this.CHANNEL) return;
      
      const { keys, pattern } = JSON.parse(message);
      
      if (keys) {
        keys.forEach((key: string) => this.localCache.delete(key));
      }
      
      if (pattern) {
        const regex = new RegExp(pattern);
        for (const key of this.localCache.keys()) {
          if (regex.test(key)) {
            this.localCache.delete(key);
          }
        }
      }
    });
  }

  async get(key: string): Promise<any> {
    // Check local cache first
    if (this.localCache.has(key)) {
      return this.localCache.get(key);
    }

    // Check Redis
    const value = await this.redis.get(key);
    if (value) {
      const parsed = JSON.parse(value);
      this.localCache.set(key, parsed);
      return parsed;
    }

    return null;
  }

  async set(key: string, value: any, ttlSeconds?: number): Promise<void> {
    const serialized = JSON.stringify(value);
    
    if (ttlSeconds) {
      await this.redis.setex(key, ttlSeconds, serialized);
    } else {
      await this.redis.set(key, serialized);
    }

    // Update local cache
    this.localCache.set(key, value);
  }

  async invalidate(keys: string[]): Promise<void> {
    // Delete from Redis
    if (keys.length > 0) {
      await this.redis.del(...keys);
    }

    // Broadcast invalidation to all instances
    await this.redis.publish(this.CHANNEL, JSON.stringify({ keys }));
  }

  async invalidatePattern(pattern: string): Promise<void> {
    // Find and delete matching keys from Redis
    const keys = await this.redis.keys(pattern);
    if (keys.length > 0) {
      await this.redis.del(...keys);
    }

    // Broadcast pattern invalidation
    await this.redis.publish(this.CHANNEL, JSON.stringify({ pattern }));
  }
}

// Usage
const cache = new DistributedCache(process.env.REDIS_URL!);

// Set cached data
await cache.set('user:123', { name: 'John', email: 'john@example.com' }, 3600);

// When user is updated
await updateUser(123, { name: 'John Doe' });
await cache.invalidate(['user:123']);  // All instances invalidate

// Invalidate all user caches
await cache.invalidatePattern('user:*');
```

---

## 2. Redis Streams

```typescript
// ════════════════════════════════════════════════════════════════
// REDIS STREAMS - PERSISTENT PUB/SUB WITH CONSUMER GROUPS
// ════════════════════════════════════════════════════════════════

import Redis from 'ioredis';

class RedisStreamPubSub {
  private redis: Redis;
  private readonly streamName: string;

  constructor(redisUrl: string, streamName: string) {
    this.redis = new Redis(redisUrl);
    this.streamName = streamName;
  }

  // ══════════════════════════════════════════════════════════════
  // PRODUCER
  // ══════════════════════════════════════════════════════════════

  async publish(data: Record<string, any>): Promise<string> {
    // Convert object to flat array for Redis Streams
    const fields: string[] = [];
    for (const [key, value] of Object.entries(data)) {
      fields.push(key, typeof value === 'object' ? JSON.stringify(value) : String(value));
    }

    // XADD stream * field1 value1 field2 value2 ...
    // * = auto-generate ID
    const messageId = await this.redis.xadd(this.streamName, '*', ...fields);
    return messageId;
  }

  // ══════════════════════════════════════════════════════════════
  // CONSUMER GROUP SETUP
  // ══════════════════════════════════════════════════════════════

  async createConsumerGroup(groupName: string, startFrom: string = '$'): Promise<void> {
    try {
      // $ = only new messages, 0 = from beginning
      await this.redis.xgroup('CREATE', this.streamName, groupName, startFrom, 'MKSTREAM');
      console.log(`Consumer group ${groupName} created`);
    } catch (error: any) {
      if (error.message.includes('BUSYGROUP')) {
        console.log(`Consumer group ${groupName} already exists`);
      } else {
        throw error;
      }
    }
  }

  // ══════════════════════════════════════════════════════════════
  // CONSUMER
  // ══════════════════════════════════════════════════════════════

  async consume(
    groupName: string,
    consumerName: string,
    handler: (id: string, data: Record<string, any>) => Promise<void>,
    options: {
      batchSize?: number;
      blockMs?: number;
    } = {}
  ): Promise<void> {
    const { batchSize = 10, blockMs = 5000 } = options;
    
    console.log(`Consumer ${consumerName} starting in group ${groupName}`);

    while (true) {
      try {
        // XREADGROUP GROUP groupName consumerName COUNT batchSize BLOCK blockMs STREAMS streamName >
        // > = only new messages not yet delivered to this group
        const results = await this.redis.xreadgroup(
          'GROUP', groupName, consumerName,
          'COUNT', batchSize,
          'BLOCK', blockMs,
          'STREAMS', this.streamName,
          '>'
        );

        if (!results) continue;  // Timeout, no new messages

        for (const [stream, messages] of results) {
          for (const [id, fields] of messages) {
            const data = this.parseFields(fields);
            
            try {
              await handler(id, data);
              // Acknowledge successful processing
              await this.redis.xack(this.streamName, groupName, id);
            } catch (error) {
              console.error(`Failed to process message ${id}:`, error);
              // Message will be redelivered later
            }
          }
        }
      } catch (error) {
        console.error('Consumer error:', error);
        await this.sleep(1000);
      }
    }
  }

  // Process pending messages (failed/unacknowledged)
  async processPending(
    groupName: string,
    consumerName: string,
    handler: (id: string, data: Record<string, any>) => Promise<void>,
    minIdleMs: number = 60000
  ): Promise<number> {
    let processed = 0;

    // Get pending messages for this consumer
    const pending = await this.redis.xpending(
      this.streamName, groupName,
      '-', '+', 100,  // min id, max id, count
      consumerName
    );

    for (const [id, consumer, idleTime, deliveryCount] of pending) {
      if (idleTime < minIdleMs) continue;

      // Claim the message
      const claimed = await this.redis.xclaim(
        this.streamName, groupName, consumerName,
        minIdleMs,  // min idle time to claim
        id
      );

      if (claimed.length > 0) {
        const [claimedId, fields] = claimed[0];
        const data = this.parseFields(fields);

        try {
          await handler(claimedId, data);
          await this.redis.xack(this.streamName, groupName, claimedId);
          processed++;
        } catch (error) {
          console.error(`Failed to process pending message ${claimedId}:`, error);
          
          // If too many retries, move to dead letter
          if (deliveryCount > 5) {
            await this.moveToDeadLetter(claimedId, data, error);
            await this.redis.xack(this.streamName, groupName, claimedId);
          }
        }
      }
    }

    return processed;
  }

  private async moveToDeadLetter(id: string, data: any, error: any): Promise<void> {
    await this.redis.xadd(`${this.streamName}:dlq`, '*',
      'originalId', id,
      'data', JSON.stringify(data),
      'error', String(error),
      'timestamp', Date.now().toString()
    );
  }

  private parseFields(fields: string[]): Record<string, any> {
    const result: Record<string, any> = {};
    for (let i = 0; i < fields.length; i += 2) {
      const key = fields[i];
      const value = fields[i + 1];
      try {
        result[key] = JSON.parse(value);
      } catch {
        result[key] = value;
      }
    }
    return result;
  }

  private sleep(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }

  // ══════════════════════════════════════════════════════════════
  // STREAM MANAGEMENT
  // ══════════════════════════════════════════════════════════════

  async trimStream(maxLength: number): Promise<number> {
    // Keep approximately maxLength messages
    return this.redis.xtrim(this.streamName, 'MAXLEN', '~', maxLength);
  }

  async getStreamInfo(): Promise<{
    length: number;
    firstEntry: string;
    lastEntry: string;
    groups: number;
  }> {
    const info = await this.redis.xinfo('STREAM', this.streamName);
    // Parse XINFO response
    return {
      length: info[1],
      firstEntry: info[3],
      lastEntry: info[5],
      groups: info[7]
    };
  }
}

// ════════════════════════════════════════════════════════════════
// USAGE
// ════════════════════════════════════════════════════════════════

const stream = new RedisStreamPubSub(process.env.REDIS_URL!, 'events');

// Create consumer group
await stream.createConsumerGroup('order-service', '0'); // Start from beginning
await stream.createConsumerGroup('notification-service', '$'); // Only new

// Publish event
await stream.publish({
  type: 'order.created',
  orderId: '123',
  userId: '456',
  items: [{ sku: 'ABC', qty: 2 }]
});

// Consumer 1: Order service
stream.consume('order-service', 'worker-1', async (id, data) => {
  console.log(`Processing order event ${id}:`, data);
  await processOrderEvent(data);
});

// Consumer 2: Notification service
stream.consume('notification-service', 'worker-1', async (id, data) => {
  if (data.type === 'order.created') {
    await sendOrderConfirmation(data.userId, data.orderId);
  }
});

// Process pending messages periodically
setInterval(async () => {
  const processed = await stream.processPending(
    'order-service', 'worker-1',
    async (id, data) => {
      console.log(`Reprocessing ${id}`);
      await processOrderEvent(data);
    },
    60000  // Messages idle for 1+ minute
  );
  if (processed > 0) {
    console.log(`Processed ${processed} pending messages`);
  }
}, 30000);
```

---

## 3. Apache Kafka Basics

```typescript
// ════════════════════════════════════════════════════════════════
// KAFKA WITH KAFKAJS
// ════════════════════════════════════════════════════════════════

import { Kafka, Consumer, Producer, EachMessagePayload, logLevel } from 'kafkajs';

// ════════════════════════════════════════════════════════════════
// KAFKA CLIENT SETUP
// ════════════════════════════════════════════════════════════════

const kafka = new Kafka({
  clientId: 'my-app',
  brokers: (process.env.KAFKA_BROKERS || 'localhost:9092').split(','),
  logLevel: logLevel.WARN,
  retry: {
    initialRetryTime: 100,
    retries: 8
  }
});

// ════════════════════════════════════════════════════════════════
// PRODUCER
// ════════════════════════════════════════════════════════════════

class KafkaProducer {
  private producer: Producer;
  private connected = false;

  constructor() {
    this.producer = kafka.producer({
      allowAutoTopicCreation: true,
      transactionTimeout: 30000
    });
  }

  async connect(): Promise<void> {
    await this.producer.connect();
    this.connected = true;
    console.log('Kafka producer connected');
  }

  async send(topic: string, messages: Array<{
    key?: string;
    value: any;
    headers?: Record<string, string>;
  }>): Promise<void> {
    if (!this.connected) {
      await this.connect();
    }

    await this.producer.send({
      topic,
      messages: messages.map(msg => ({
        key: msg.key,
        value: JSON.stringify(msg.value),
        headers: msg.headers
      }))
    });
  }

  // Send single message
  async publish(topic: string, value: any, key?: string): Promise<void> {
    await this.send(topic, [{ key, value }]);
  }

  // Send with partition key (ensures ordering)
  async publishWithKey(topic: string, key: string, value: any): Promise<void> {
    await this.send(topic, [{ key, value }]);
    // Messages with same key go to same partition
  }

  async disconnect(): Promise<void> {
    await this.producer.disconnect();
    this.connected = false;
  }
}

// ════════════════════════════════════════════════════════════════
// CONSUMER
// ════════════════════════════════════════════════════════════════

class KafkaConsumer {
  private consumer: Consumer;
  private handlers: Map<string, (payload: EachMessagePayload) => Promise<void>> = new Map();

  constructor(groupId: string) {
    this.consumer = kafka.consumer({
      groupId,
      sessionTimeout: 30000,
      heartbeatInterval: 3000,
      maxBytesPerPartition: 1048576, // 1MB
      retry: {
        retries: 5
      }
    });
  }

  async connect(): Promise<void> {
    await this.consumer.connect();
    console.log('Kafka consumer connected');
  }

  async subscribe(topics: string[], fromBeginning = false): Promise<void> {
    for (const topic of topics) {
      await this.consumer.subscribe({ topic, fromBeginning });
    }
  }

  onMessage(topic: string, handler: (data: any, metadata: {
    partition: number;
    offset: string;
    key: string | null;
    timestamp: string;
  }) => Promise<void>): void {
    this.handlers.set(topic, async (payload) => {
      const { message, partition, topic: msgTopic } = payload;
      
      const data = JSON.parse(message.value?.toString() || '{}');
      
      await handler(data, {
        partition,
        offset: message.offset,
        key: message.key?.toString() || null,
        timestamp: message.timestamp
      });
    });
  }

  async start(): Promise<void> {
    await this.consumer.run({
      eachMessage: async (payload) => {
        const { topic } = payload;
        const handler = this.handlers.get(topic);
        
        if (handler) {
          try {
            await handler(payload);
          } catch (error) {
            console.error(`Error processing message from ${topic}:`, error);
            // Throw to trigger retry/DLQ
            throw error;
          }
        }
      }
    });
  }

  async pause(topics: Array<{ topic: string; partitions?: number[] }>): Promise<void> {
    this.consumer.pause(topics.map(t => ({
      topic: t.topic,
      partitions: t.partitions
    })));
  }

  async resume(topics: Array<{ topic: string; partitions?: number[] }>): Promise<void> {
    this.consumer.resume(topics.map(t => ({
      topic: t.topic,
      partitions: t.partitions
    })));
  }

  async disconnect(): Promise<void> {
    await this.consumer.disconnect();
  }
}

// ════════════════════════════════════════════════════════════════
// USAGE
// ════════════════════════════════════════════════════════════════

// Producer
const producer = new KafkaProducer();
await producer.connect();

// Publish event with key for ordering
await producer.publishWithKey(
  'orders',
  'user-123',  // All orders for user-123 go to same partition
  {
    type: 'order.created',
    orderId: '456',
    userId: 'user-123',
    items: [{ sku: 'ABC', qty: 2 }]
  }
);

// Consumer
const consumer = new KafkaConsumer('order-processing-service');
await consumer.connect();
await consumer.subscribe(['orders'], false);  // Don't read from beginning

consumer.onMessage('orders', async (data, metadata) => {
  console.log(`Processing order event from partition ${metadata.partition}:`, data);
  
  switch (data.type) {
    case 'order.created':
      await handleOrderCreated(data);
      break;
    case 'order.shipped':
      await handleOrderShipped(data);
      break;
  }
});

await consumer.start();

// Graceful shutdown
process.on('SIGTERM', async () => {
  await consumer.disconnect();
  await producer.disconnect();
});

// ════════════════════════════════════════════════════════════════
// ADMIN OPERATIONS
// ════════════════════════════════════════════════════════════════

const admin = kafka.admin();

async function createTopic(topic: string, partitions: number = 3): Promise<void> {
  await admin.connect();
  
  await admin.createTopics({
    topics: [{
      topic,
      numPartitions: partitions,
      replicationFactor: 1,  // Set to 3 for production
      configEntries: [
        { name: 'retention.ms', value: '604800000' },  // 7 days
        { name: 'cleanup.policy', value: 'delete' }
      ]
    }]
  });

  await admin.disconnect();
}

async function listTopics(): Promise<string[]> {
  await admin.connect();
  const topics = await admin.listTopics();
  await admin.disconnect();
  return topics;
}

async function getTopicOffsets(topic: string): Promise<any> {
  await admin.connect();
  const offsets = await admin.fetchTopicOffsets(topic);
  await admin.disconnect();
  return offsets;
}
```

---

## 4. AWS SNS

```typescript
// ════════════════════════════════════════════════════════════════
// AWS SNS - SIMPLE NOTIFICATION SERVICE
// ════════════════════════════════════════════════════════════════

import {
  SNSClient,
  PublishCommand,
  SubscribeCommand,
  CreateTopicCommand,
  ListSubscriptionsByTopicCommand
} from '@aws-sdk/client-sns';

class SNSPublisher {
  private client: SNSClient;

  constructor(region: string = 'us-east-1') {
    this.client = new SNSClient({ region });
  }

  // ══════════════════════════════════════════════════════════════
  // PUBLISH
  // ══════════════════════════════════════════════════════════════

  async publish(topicArn: string, message: any, options?: {
    subject?: string;
    attributes?: Record<string, { DataType: string; StringValue: string }>;
    deduplicationId?: string;  // For FIFO topics
    groupId?: string;          // For FIFO topics
  }): Promise<string> {
    const command = new PublishCommand({
      TopicArn: topicArn,
      Message: JSON.stringify(message),
      Subject: options?.subject,
      MessageAttributes: options?.attributes,
      MessageDeduplicationId: options?.deduplicationId,
      MessageGroupId: options?.groupId
    });

    const response = await this.client.send(command);
    return response.MessageId!;
  }

  // Publish to multiple targets with filtering
  async publishWithFilter(
    topicArn: string,
    message: any,
    filterAttributes: Record<string, string>
  ): Promise<string> {
    const attributes: Record<string, { DataType: string; StringValue: string }> = {};
    
    for (const [key, value] of Object.entries(filterAttributes)) {
      attributes[key] = {
        DataType: 'String',
        StringValue: value
      };
    }

    return this.publish(topicArn, message, { attributes });
  }

  // ══════════════════════════════════════════════════════════════
  // TOPIC MANAGEMENT
  // ══════════════════════════════════════════════════════════════

  async createTopic(name: string, fifo = false): Promise<string> {
    const command = new CreateTopicCommand({
      Name: fifo ? `${name}.fifo` : name,
      Attributes: fifo ? {
        FifoTopic: 'true',
        ContentBasedDeduplication: 'true'
      } : undefined
    });

    const response = await this.client.send(command);
    return response.TopicArn!;
  }

  async subscribeSqs(topicArn: string, sqsArn: string): Promise<string> {
    const command = new SubscribeCommand({
      TopicArn: topicArn,
      Protocol: 'sqs',
      Endpoint: sqsArn,
      ReturnSubscriptionArn: true
    });

    const response = await this.client.send(command);
    return response.SubscriptionArn!;
  }

  async subscribeLambda(topicArn: string, lambdaArn: string): Promise<string> {
    const command = new SubscribeCommand({
      TopicArn: topicArn,
      Protocol: 'lambda',
      Endpoint: lambdaArn,
      ReturnSubscriptionArn: true
    });

    const response = await this.client.send(command);
    return response.SubscriptionArn!;
  }

  async subscribeHttp(topicArn: string, url: string, https = true): Promise<string> {
    const command = new SubscribeCommand({
      TopicArn: topicArn,
      Protocol: https ? 'https' : 'http',
      Endpoint: url,
      ReturnSubscriptionArn: true
    });

    const response = await this.client.send(command);
    return response.SubscriptionArn!;
  }

  // Subscribe with filter policy
  async subscribeWithFilter(
    topicArn: string,
    protocol: 'sqs' | 'lambda' | 'https',
    endpoint: string,
    filterPolicy: Record<string, string[]>
  ): Promise<string> {
    const command = new SubscribeCommand({
      TopicArn: topicArn,
      Protocol: protocol,
      Endpoint: endpoint,
      Attributes: {
        FilterPolicy: JSON.stringify(filterPolicy)
      },
      ReturnSubscriptionArn: true
    });

    const response = await this.client.send(command);
    return response.SubscriptionArn!;
  }
}

// ════════════════════════════════════════════════════════════════
// SNS + SQS FAN-OUT PATTERN
// ════════════════════════════════════════════════════════════════

/*
Architecture:
                         ┌──────────────────┐
                    ┌───▶│ SQS: orders      │───▶ Order Service
                    │    └──────────────────┘
    ┌──────────┐    │    ┌──────────────────┐
    │   SNS    │────┼───▶│ SQS: inventory   │───▶ Inventory Service
    │  Topic   │    │    └──────────────────┘
    └──────────┘    │    ┌──────────────────┐
         ▲          └───▶│ SQS: analytics   │───▶ Analytics Service
         │               └──────────────────┘
    [Publish once, 
     delivered to all]
*/

class SNSFanOut {
  private sns: SNSPublisher;
  private topicArn: string;

  constructor(topicArn: string) {
    this.sns = new SNSPublisher();
    this.topicArn = topicArn;
  }

  async publish(eventType: string, data: any): Promise<string> {
    return this.sns.publishWithFilter(
      this.topicArn,
      {
        eventType,
        data,
        timestamp: new Date().toISOString()
      },
      {
        eventType  // For filtering
      }
    );
  }

  // Setup subscriptions with filters
  async setupSubscriptions() {
    // Order service gets all order events
    await this.sns.subscribeWithFilter(
      this.topicArn,
      'sqs',
      process.env.ORDERS_QUEUE_ARN!,
      { eventType: ['order.created', 'order.updated', 'order.cancelled'] }
    );

    // Inventory service gets inventory-related events
    await this.sns.subscribeWithFilter(
      this.topicArn,
      'sqs',
      process.env.INVENTORY_QUEUE_ARN!,
      { eventType: ['order.created', 'inventory.updated'] }
    );

    // Analytics service gets everything
    await this.sns.subscribeSqs(
      this.topicArn,
      process.env.ANALYTICS_QUEUE_ARN!
    );
  }
}

// Usage
const fanOut = new SNSFanOut(process.env.SNS_TOPIC_ARN!);

// Publish event - goes to all matching subscribers
await fanOut.publish('order.created', {
  orderId: '123',
  userId: '456',
  items: [{ sku: 'ABC', qty: 2 }]
});
```

---

## 5. Fan-Out Patterns

```typescript
// ════════════════════════════════════════════════════════════════
// FAN-OUT PATTERNS
// ════════════════════════════════════════════════════════════════

// Pattern 1: Simple Fan-Out (broadcast to all)
class SimpleFanOut {
  constructor(private pubsub: TypedPubSub<any>) {}

  async broadcast(event: string, data: any): Promise<void> {
    await this.pubsub.publish(event, data);
    // All subscribers receive
  }
}

// Pattern 2: Filtered Fan-Out (subscribers filter by type)
interface FilteredSubscription<T> {
  filter: (event: T) => boolean;
  handler: (event: T) => Promise<void>;
}

class FilteredFanOut<T extends { type: string }> {
  private subscriptions: FilteredSubscription<T>[] = [];

  subscribe(
    filter: (event: T) => boolean,
    handler: (event: T) => Promise<void>
  ): () => void {
    const subscription = { filter, handler };
    this.subscriptions.push(subscription);
    
    return () => {
      const index = this.subscriptions.indexOf(subscription);
      if (index > -1) {
        this.subscriptions.splice(index, 1);
      }
    };
  }

  async publish(event: T): Promise<void> {
    const matching = this.subscriptions.filter(s => s.filter(event));
    
    await Promise.all(
      matching.map(s => s.handler(event).catch(err => {
        console.error('Handler error:', err);
      }))
    );
  }
}

// Usage
const events = new FilteredFanOut<{ type: string; data: any }>();

// Order service - only order events
events.subscribe(
  e => e.type.startsWith('order.'),
  async (event) => {
    console.log('Order event:', event);
  }
);

// User service - only user events
events.subscribe(
  e => e.type.startsWith('user.'),
  async (event) => {
    console.log('User event:', event);
  }
);

// Audit service - all events
events.subscribe(
  () => true,
  async (event) => {
    await logToAudit(event);
  }
);

// Publish - goes to matching subscribers
await events.publish({ type: 'order.created', data: { orderId: '123' } });

// ════════════════════════════════════════════════════════════════
// Pattern 3: Priority Fan-Out
// ════════════════════════════════════════════════════════════════

interface PrioritySubscription<T> {
  priority: number;
  handler: (event: T) => Promise<boolean>;  // Return false to stop propagation
}

class PriorityFanOut<T> {
  private subscriptions: PrioritySubscription<T>[] = [];

  subscribe(
    priority: number,
    handler: (event: T) => Promise<boolean>
  ): () => void {
    const subscription = { priority, handler };
    this.subscriptions.push(subscription);
    this.subscriptions.sort((a, b) => a.priority - b.priority);  // Lower = higher priority
    
    return () => {
      const index = this.subscriptions.indexOf(subscription);
      if (index > -1) {
        this.subscriptions.splice(index, 1);
      }
    };
  }

  async publish(event: T): Promise<void> {
    for (const subscription of this.subscriptions) {
      try {
        const shouldContinue = await subscription.handler(event);
        if (!shouldContinue) {
          console.log('Event propagation stopped');
          break;
        }
      } catch (error) {
        console.error('Handler error:', error);
      }
    }
  }
}

// ════════════════════════════════════════════════════════════════
// Pattern 4: Topic-Based Fan-Out (Kafka-style)
// ════════════════════════════════════════════════════════════════

class TopicFanOut {
  private topics: Map<string, Set<(data: any) => Promise<void>>> = new Map();

  subscribe(topicPattern: string, handler: (data: any) => Promise<void>): () => void {
    if (!this.topics.has(topicPattern)) {
      this.topics.set(topicPattern, new Set());
    }
    
    this.topics.get(topicPattern)!.add(handler);

    return () => {
      this.topics.get(topicPattern)?.delete(handler);
    };
  }

  async publish(topic: string, data: any): Promise<void> {
    const matchingHandlers: ((data: any) => Promise<void>)[] = [];

    this.topics.forEach((handlers, pattern) => {
      if (this.matchesTopic(topic, pattern)) {
        handlers.forEach(h => matchingHandlers.push(h));
      }
    });

    await Promise.all(
      matchingHandlers.map(h => h(data).catch(console.error))
    );
  }

  private matchesTopic(topic: string, pattern: string): boolean {
    // Simple glob matching
    // orders.* matches orders.created, orders.updated
    // orders.# matches orders.created, orders.us.created
    const regexPattern = pattern
      .replace(/\./g, '\\.')
      .replace(/\*/g, '[^.]+')
      .replace(/#/g, '.*');
    
    return new RegExp(`^${regexPattern}$`).test(topic);
  }
}

// Usage
const topicFanOut = new TopicFanOut();

// Subscribe to specific topic
topicFanOut.subscribe('orders.created', async (data) => {
  console.log('New order:', data);
});

// Subscribe to all order topics
topicFanOut.subscribe('orders.*', async (data) => {
  console.log('Order event:', data);
});

// Subscribe to everything
topicFanOut.subscribe('#', async (data) => {
  console.log('All events:', data);
});

await topicFanOut.publish('orders.created', { orderId: '123' });
```

---

## 6. Event-Driven Architecture

```typescript
// ════════════════════════════════════════════════════════════════
// EVENT-DRIVEN ARCHITECTURE WITH PUB/SUB
// ════════════════════════════════════════════════════════════════

// Domain Event base
interface DomainEvent {
  eventId: string;
  eventType: string;
  aggregateId: string;
  aggregateType: string;
  timestamp: Date;
  version: number;
  payload: any;
  metadata: {
    correlationId: string;
    causationId?: string;
    userId?: string;
  };
}

// Event Factory
class EventFactory {
  static create<T>(
    eventType: string,
    aggregateId: string,
    aggregateType: string,
    payload: T,
    metadata: { correlationId: string; causationId?: string; userId?: string }
  ): DomainEvent {
    return {
      eventId: `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
      eventType,
      aggregateId,
      aggregateType,
      timestamp: new Date(),
      version: 1,
      payload,
      metadata
    };
  }
}

// Event Bus
class EventBus {
  constructor(
    private producer: KafkaProducer,
    private topicPrefix = 'domain-events'
  ) {}

  async publish(event: DomainEvent): Promise<void> {
    const topic = `${this.topicPrefix}.${event.aggregateType}`;
    
    await this.producer.publishWithKey(
      topic,
      event.aggregateId,  // Partition by aggregate for ordering
      event
    );
  }

  async publishBatch(events: DomainEvent[]): Promise<void> {
    const byTopic = new Map<string, DomainEvent[]>();
    
    for (const event of events) {
      const topic = `${this.topicPrefix}.${event.aggregateType}`;
      if (!byTopic.has(topic)) {
        byTopic.set(topic, []);
      }
      byTopic.get(topic)!.push(event);
    }

    for (const [topic, topicEvents] of byTopic) {
      await this.producer.send(
        topic,
        topicEvents.map(e => ({
          key: e.aggregateId,
          value: e
        }))
      );
    }
  }
}

// ════════════════════════════════════════════════════════════════
// EVENT HANDLERS
// ════════════════════════════════════════════════════════════════

type EventHandler<T extends DomainEvent> = (event: T) => Promise<void>;

class EventHandlerRegistry {
  private handlers: Map<string, EventHandler<any>[]> = new Map();

  register<T extends DomainEvent>(
    eventType: string,
    handler: EventHandler<T>
  ): void {
    if (!this.handlers.has(eventType)) {
      this.handlers.set(eventType, []);
    }
    this.handlers.get(eventType)!.push(handler);
  }

  async handle(event: DomainEvent): Promise<void> {
    const handlers = this.handlers.get(event.eventType) || [];
    
    await Promise.all(
      handlers.map(h => h(event).catch(err => {
        console.error(`Handler error for ${event.eventType}:`, err);
        throw err;
      }))
    );
  }
}

// ════════════════════════════════════════════════════════════════
// SERVICE EXAMPLE
// ════════════════════════════════════════════════════════════════

// Order Service - publishes events
class OrderService {
  constructor(
    private eventBus: EventBus,
    private repository: OrderRepository
  ) {}

  async createOrder(
    userId: string,
    items: OrderItem[],
    correlationId: string
  ): Promise<Order> {
    // Business logic
    const order = Order.create(userId, items);
    await this.repository.save(order);

    // Publish event
    const event = EventFactory.create(
      'OrderCreated',
      order.id,
      'Order',
      {
        orderId: order.id,
        userId,
        items,
        total: order.total,
        status: order.status
      },
      { correlationId, userId }
    );

    await this.eventBus.publish(event);

    return order;
  }

  async shipOrder(orderId: string, correlationId: string): Promise<void> {
    const order = await this.repository.findById(orderId);
    order.ship();
    await this.repository.save(order);

    const event = EventFactory.create(
      'OrderShipped',
      orderId,
      'Order',
      {
        orderId,
        shippedAt: new Date()
      },
      { correlationId, causationId: orderId }
    );

    await this.eventBus.publish(event);
  }
}

// Inventory Service - reacts to events
class InventoryEventHandler {
  constructor(
    private inventoryService: InventoryService,
    private registry: EventHandlerRegistry
  ) {
    this.registerHandlers();
  }

  private registerHandlers() {
    this.registry.register('OrderCreated', async (event) => {
      const { items } = event.payload;
      
      for (const item of items) {
        await this.inventoryService.reserve(
          item.sku,
          item.quantity,
          event.metadata.correlationId
        );
      }
    });

    this.registry.register('OrderCancelled', async (event) => {
      const { items } = event.payload;
      
      for (const item of items) {
        await this.inventoryService.release(
          item.sku,
          item.quantity,
          event.metadata.correlationId
        );
      }
    });
  }
}

// Notification Service - reacts to events
class NotificationEventHandler {
  constructor(
    private notificationService: NotificationService,
    private registry: EventHandlerRegistry
  ) {
    this.registerHandlers();
  }

  private registerHandlers() {
    this.registry.register('OrderCreated', async (event) => {
      await this.notificationService.sendOrderConfirmation(
        event.payload.userId,
        event.payload.orderId
      );
    });

    this.registry.register('OrderShipped', async (event) => {
      const order = await this.orderRepository.findById(event.payload.orderId);
      await this.notificationService.sendShipmentNotification(
        order.userId,
        event.payload.orderId
      );
    });
  }
}
```

---

## 7. Scaling Pub/Sub

```typescript
// ════════════════════════════════════════════════════════════════
// SCALING PUB/SUB SYSTEMS
// ════════════════════════════════════════════════════════════════

// 1. REDIS CLUSTER PUB/SUB
// ────────────────────────────────────────────────────────────────

import Redis, { Cluster } from 'ioredis';

// For Redis Cluster, use sharded pub/sub (Redis 7+)
const cluster = new Cluster([
  { host: 'redis-1', port: 6379 },
  { host: 'redis-2', port: 6379 },
  { host: 'redis-3', port: 6379 }
]);

// Sharded pub/sub - messages go to shard owning the channel
await cluster.spublish('channel:123', JSON.stringify(data));

cluster.ssubscribe('channel:123', (err) => {
  if (err) console.error(err);
});

cluster.on('smessage', (channel, message) => {
  console.log(`Received on ${channel}:`, message);
});

// 2. KAFKA PARTITIONING FOR SCALE
// ────────────────────────────────────────────────────────────────

/*
Topic: orders (6 partitions)

Consumer Group: order-service
├── Consumer 1 → Partitions 0, 1
├── Consumer 2 → Partitions 2, 3
└── Consumer 3 → Partitions 4, 5

Consumer Group: analytics-service
├── Consumer A → Partitions 0, 1, 2
└── Consumer B → Partitions 3, 4, 5

Each group gets ALL messages
Within group, load balanced across consumers
*/

// Optimal partition count = max consumers you expect to run
async function createScalableTopic(admin: any, topic: string) {
  await admin.createTopics({
    topics: [{
      topic,
      numPartitions: 12,      // Can have up to 12 consumers per group
      replicationFactor: 3,   // For durability
      configEntries: [
        { name: 'min.insync.replicas', value: '2' }
      ]
    }]
  });
}

// 3. CONSUMER AUTO-SCALING
// ────────────────────────────────────────────────────────────────

class ScalableConsumer {
  private consumer: Consumer;
  private messageCount = 0;
  private lastScaleCheck = Date.now();

  async getConsumerLag(): Promise<number> {
    const admin = kafka.admin();
    await admin.connect();

    const offsets = await admin.fetchTopicOffsets(this.topic);
    const groupOffsets = await admin.fetchOffsets({
      groupId: this.groupId,
      topics: [this.topic]
    });

    let totalLag = 0;
    for (const partition of offsets) {
      const groupOffset = groupOffsets[0].partitions.find(
        p => p.partition === partition.partition
      );
      if (groupOffset) {
        totalLag += parseInt(partition.offset) - parseInt(groupOffset.offset);
      }
    }

    await admin.disconnect();
    return totalLag;
  }

  async monitorAndScale(): Promise<void> {
    setInterval(async () => {
      const lag = await this.getConsumerLag();
      
      if (lag > 10000) {
        console.log(`High lag (${lag}), scale up recommended`);
        await this.requestScaleUp();
      } else if (lag < 100 && this.canScaleDown()) {
        console.log(`Low lag (${lag}), scale down possible`);
        await this.requestScaleDown();
      }
    }, 30000);
  }

  private async requestScaleUp(): Promise<void> {
    // Trigger Kubernetes HPA or similar
  }

  private async requestScaleDown(): Promise<void> {
    // Check if safe to scale down
  }
}

// 4. BACKPRESSURE HANDLING
// ────────────────────────────────────────────────────────────────

class BackpressureHandler {
  private processingCount = 0;
  private maxConcurrency = 100;
  private queue: Array<{ data: any; resolve: () => void }> = [];

  async process(data: any): Promise<void> {
    if (this.processingCount >= this.maxConcurrency) {
      // Wait for slot
      await new Promise<void>(resolve => {
        this.queue.push({ data, resolve });
      });
    }

    this.processingCount++;
    
    try {
      await this.doProcess(data);
    } finally {
      this.processingCount--;
      
      // Process queued item
      const next = this.queue.shift();
      if (next) {
        next.resolve();
      }
    }
  }

  private async doProcess(data: any): Promise<void> {
    // Actual processing
  }
}

// With Kafka - pause/resume partitions
class BackpressureConsumer {
  private consumer: Consumer;
  private paused = false;
  private bufferSize = 0;
  private maxBuffer = 1000;

  async handleMessage(payload: EachMessagePayload): Promise<void> {
    this.bufferSize++;

    if (this.bufferSize > this.maxBuffer && !this.paused) {
      console.log('Buffer full, pausing consumption');
      this.consumer.pause([{ topic: payload.topic }]);
      this.paused = true;
    }

    try {
      await this.process(payload);
    } finally {
      this.bufferSize--;

      if (this.bufferSize < this.maxBuffer / 2 && this.paused) {
        console.log('Buffer drained, resuming');
        this.consumer.resume([{ topic: payload.topic }]);
        this.paused = false;
      }
    }
  }
}
```

---

## 8. Real-World Patterns

### Microservices Event Communication

```typescript
// ════════════════════════════════════════════════════════════════
// MICROSERVICES EVENT COMMUNICATION
// ════════════════════════════════════════════════════════════════

// Event schemas (shared library)
interface UserCreatedEvent {
  type: 'user.created';
  userId: string;
  email: string;
  createdAt: string;
}

interface OrderPlacedEvent {
  type: 'order.placed';
  orderId: string;
  userId: string;
  items: Array<{ sku: string; qty: number; price: number }>;
  total: number;
}

interface PaymentProcessedEvent {
  type: 'payment.processed';
  paymentId: string;
  orderId: string;
  amount: number;
  status: 'success' | 'failed';
}

type DomainEvents = 
  | UserCreatedEvent 
  | OrderPlacedEvent 
  | PaymentProcessedEvent;

// User Service
class UserService {
  constructor(private eventBus: EventBus) {}

  async createUser(email: string, password: string): Promise<User> {
    const user = await this.repository.create({ email, password });

    await this.eventBus.publish({
      type: 'user.created',
      userId: user.id,
      email: user.email,
      createdAt: new Date().toISOString()
    });

    return user;
  }
}

// Order Service - listens for user events, publishes order events
class OrderService {
  constructor(private eventBus: EventBus) {
    this.subscribeToEvents();
  }

  private subscribeToEvents() {
    this.eventBus.subscribe('user.created', async (event: UserCreatedEvent) => {
      // Initialize user's order history
      await this.initializeUserOrderHistory(event.userId);
    });

    this.eventBus.subscribe('payment.processed', async (event: PaymentProcessedEvent) => {
      if (event.status === 'success') {
        await this.confirmOrder(event.orderId);
      } else {
        await this.cancelOrder(event.orderId);
      }
    });
  }

  async placeOrder(userId: string, items: OrderItem[]): Promise<Order> {
    const order = await this.repository.create({ userId, items, status: 'pending' });

    await this.eventBus.publish({
      type: 'order.placed',
      orderId: order.id,
      userId,
      items: items.map(i => ({ sku: i.sku, qty: i.qty, price: i.price })),
      total: order.total
    });

    return order;
  }
}

// Payment Service - listens for order events
class PaymentService {
  constructor(private eventBus: EventBus) {
    this.subscribeToEvents();
  }

  private subscribeToEvents() {
    this.eventBus.subscribe('order.placed', async (event: OrderPlacedEvent) => {
      // Process payment
      const result = await this.processPayment(event.userId, event.total);

      await this.eventBus.publish({
        type: 'payment.processed',
        paymentId: result.id,
        orderId: event.orderId,
        amount: event.total,
        status: result.success ? 'success' : 'failed'
      });
    });
  }
}

// Email Service - listens for all events
class EmailService {
  constructor(private eventBus: EventBus) {
    this.subscribeToEvents();
  }

  private subscribeToEvents() {
    this.eventBus.subscribe('user.created', async (event: UserCreatedEvent) => {
      await this.sendWelcomeEmail(event.email);
    });

    this.eventBus.subscribe('order.placed', async (event: OrderPlacedEvent) => {
      const user = await this.userService.getUser(event.userId);
      await this.sendOrderConfirmation(user.email, event.orderId);
    });

    this.eventBus.subscribe('payment.processed', async (event: PaymentProcessedEvent) => {
      if (event.status === 'failed') {
        const order = await this.orderService.getOrder(event.orderId);
        const user = await this.userService.getUser(order.userId);
        await this.sendPaymentFailedEmail(user.email, event.orderId);
      }
    });
  }
}
```

---

## 9. Common Pitfalls

```typescript
// ════════════════════════════════════════════════════════════════
// PUB/SUB PITFALLS AND SOLUTIONS
// ════════════════════════════════════════════════════════════════

// ❌ PITFALL 1: Assuming message delivery in Redis Pub/Sub
// Problem: Messages lost if no subscribers online

// Bad - assuming all messages received
await redis.publish('events', JSON.stringify(event));
// If no subscribers connected, message lost forever!

// Good - use Redis Streams or Kafka for durability
await redis.xadd('events', '*', 'data', JSON.stringify(event));
// Or
await kafka.send({ topic: 'events', messages: [{ value: event }] });

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 2: Not handling subscriber errors
// Problem: Error in one handler crashes all

// Bad
subscriber.on('message', async (channel, message) => {
  const data = JSON.parse(message);
  await handler1(data);  // If this throws...
  await handler2(data);  // ...this never runs
});

// Good - isolate handlers
subscriber.on('message', async (channel, message) => {
  const data = JSON.parse(message);
  
  const handlers = [handler1, handler2, handler3];
  
  await Promise.allSettled(
    handlers.map(h => h(data).catch(err => {
      console.error(`Handler error:`, err);
      // Don't rethrow - let other handlers run
    }))
  );
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 3: Publishing commands instead of events
// Problem: Tight coupling between services

// Bad - command
await publish('send-email', { to: 'user@example.com', template: 'welcome' });
// Order service knows about email service

// Good - event
await publish('user.created', { userId: '123', email: 'user@example.com' });
// Email service reacts to event, order service doesn't care who listens

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 4: Not considering message ordering
// Problem: Events processed out of order

// Bad - events may arrive out of order
await publish('order.created', { orderId: '123' });
await publish('order.shipped', { orderId: '123' });
// Subscriber might receive 'shipped' before 'created'

// Good - use Kafka with partition key
await kafka.send({
  topic: 'orders',
  messages: [{
    key: '123',  // Same order ID = same partition = ordering guaranteed
    value: JSON.stringify({ type: 'order.created', orderId: '123' })
  }]
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 5: Unbounded subscriber memory
// Problem: Slow consumer accumulates messages in memory

// Bad
const messages: any[] = [];
subscriber.on('message', (channel, message) => {
  messages.push(JSON.parse(message));  // Grows forever!
  processAsync(messages);
});

// Good - implement backpressure
const MAX_BUFFER = 1000;
const buffer: any[] = [];

subscriber.on('message', (channel, message) => {
  if (buffer.length >= MAX_BUFFER) {
    console.warn('Buffer full, dropping message');
    return;  // Or pause subscription
  }
  buffer.push(JSON.parse(message));
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 6: Using same Redis client for pub and sub
// Problem: Subscriber client enters subscription mode

// Bad
const redis = new Redis();
redis.subscribe('channel');  // Client now in subscription mode
await redis.get('key');      // Error! Can't use regular commands

// Good - separate clients
const publisher = new Redis();
const subscriber = new Redis();

subscriber.subscribe('channel');
subscriber.on('message', handleMessage);

await publisher.get('key');  // Works fine
await publisher.publish('channel', data);  // Works fine

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 7: Not handling reconnection
// Problem: Lost subscription after disconnect

// Bad
subscriber.subscribe('channel');
// If connection drops, subscription lost

// Good
subscriber.on('error', (err) => {
  console.error('Subscriber error:', err);
});

subscriber.on('reconnecting', () => {
  console.log('Reconnecting...');
});

subscriber.on('connect', () => {
  console.log('Connected, resubscribing...');
  subscriber.subscribe('channel1', 'channel2');
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 8: Large messages
// Problem: Performance issues, memory pressure

// Bad
await publish('data', { 
  records: hugeArray  // 10MB of data
});

// Good - reference pattern
const s3Key = await uploadToS3(hugeArray);
await publish('data', {
  recordsRef: s3Key,
  recordCount: hugeArray.length
});
```

---

## 10. Interview Questions

### Basic Questions

**Q: "What is Pub/Sub?"**
> "Pub/Sub is a messaging pattern where publishers send messages to topics without knowing who receives them, and subscribers listen to topics without knowing who sent messages. This provides complete decoupling - publishers and subscribers have no direct dependency. It enables fan-out where one message reaches all subscribers."

**Q: "What's the difference between Pub/Sub and message queues?"**
> "Message queues are point-to-point: one consumer per message, messages consumed and removed. Pub/Sub is broadcast: all subscribers receive all messages. Use queues for work distribution, Pub/Sub for event notifications. Some systems like Kafka combine both with consumer groups."

**Q: "What is fan-out?"**
> "Fan-out is when a single message is delivered to multiple subscribers simultaneously. Classic Pub/Sub pattern - publish once, everyone listening receives. Enables building decoupled systems where multiple services react to the same event independently."

### Intermediate Questions

**Q: "Redis Pub/Sub vs Kafka - when to use each?"**
> "**Redis Pub/Sub**: Simple, fast (<1ms), no persistence. Good for real-time notifications, cache invalidation, presence. If no subscriber online, message lost.

**Kafka**: Durable, high throughput, replay capability, consumer groups. Good for event sourcing, audit logs, analytics, high volume.

Key difference: Redis is fire-and-forget, Kafka persists. Choose based on whether you can tolerate message loss."

**Q: "What are Kafka consumer groups?"**
> "Consumer groups enable parallel processing. Within a group, each message delivered to one consumer only (load balancing). Multiple groups each get all messages. Enables both competing consumers (within group) and broadcast (across groups). Partitions assigned to consumers - can't have more consumers than partitions in a group."

**Q: "How do you ensure message ordering in Pub/Sub?"**
> "Kafka: partition by key (e.g., user ID). All messages with same key go to same partition, ordered within partition. Redis Pub/Sub: no ordering guarantees. Redis Streams: XREAD returns in order, but multiple consumers may process out of order. For strict ordering, use single consumer or design for eventual consistency."

### Advanced Questions

**Q: "How would you design an event-driven architecture with Pub/Sub?"**
> "Architecture:
1. **Define domain events** - Past-tense facts about what happened
2. **Event schemas** - Shared schema registry, versioning
3. **Event bus** - Kafka for durability, partition by aggregate ID
4. **Handlers** - Services subscribe to events they care about
5. **Idempotency** - Handle duplicate events gracefully

Principles: Publish events not commands. Events are immutable facts. Services react to events, don't call each other directly. Use correlation IDs for tracing."

**Q: "How do you handle slow consumers in Pub/Sub?"**
> "Options:
1. **Backpressure**: Pause subscription when buffer full, resume when drained
2. **Kafka**: Consumer reads at own pace from log, won't affect others
3. **Redis Pub/Sub**: Slow consumer misses messages (fire-and-forget)
4. **Buffering**: Queue messages locally, process async
5. **Scale out**: Add more consumers (Kafka: add to consumer group)

Monitor consumer lag, alert when falling behind, auto-scale based on lag metrics."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                  PUB/SUB IMPLEMENTATION CHECKLIST               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  DESIGN:                                                        │
│  □ Define event schemas (what happened, not what to do)        │
│  □ Choose delivery guarantee (at-most-once vs at-least-once)   │
│  □ Plan for ordering requirements                              │
│  □ Consider message size limits                                │
│                                                                 │
│  REDIS PUB/SUB:                                                 │
│  □ Use separate clients for pub and sub                        │
│  □ Handle reconnection and resubscription                      │
│  □ Accept message loss for offline subscribers                 │
│  □ Keep messages small                                         │
│                                                                 │
│  KAFKA:                                                         │
│  □ Partition by key for ordering                               │
│  □ Set appropriate partition count for parallelism             │
│  □ Configure retention based on needs                          │
│  □ Use consumer groups appropriately                           │
│                                                                 │
│  RELIABILITY:                                                   │
│  □ Handle subscriber errors gracefully                         │
│  □ Implement idempotent handlers                               │
│  □ Monitor consumer lag                                        │
│  □ Plan for message replay                                     │
│                                                                 │
│  SCALING:                                                       │
│  □ Implement backpressure for slow consumers                   │
│  □ Monitor and auto-scale consumers                            │
│  □ Use appropriate partition count                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

SYSTEM SELECTION:
┌────────────────────────────────────────────────────────────────┐
│ Use Case                  │ Best Choice                        │
├───────────────────────────┼────────────────────────────────────┤
│ Real-time notifications   │ Redis Pub/Sub                      │
│ Cache invalidation        │ Redis Pub/Sub                      │
│ Event sourcing            │ Kafka                              │
│ High throughput           │ Kafka                              │
│ Need replay               │ Kafka, Redis Streams               │
│ AWS native                │ SNS + SQS                          │
│ Serverless                │ SNS, Google Pub/Sub                │
│ Simple, need durability   │ Redis Streams                      │
└────────────────────────────────────────────────────────────────┘

KAFKA CONSUMER GROUPS:
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│  Topic (6 partitions)                                         │
│  ════════════════════                                         │
│                                                                │
│  Consumer Group A (order-service):                            │
│    Consumer 1 → P0, P1    ┐                                   │
│    Consumer 2 → P2, P3    ├── Load balanced within group     │
│    Consumer 3 → P4, P5    ┘                                   │
│                                                                │
│  Consumer Group B (analytics):                                │
│    Consumer X → P0-P5     ←── Gets ALL messages               │
│                                                                │
│  Rule: #consumers ≤ #partitions per group                     │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

