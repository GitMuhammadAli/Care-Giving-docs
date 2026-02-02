# Stream Processing - Complete Guide

> **MUST REMEMBER**: Stream processing handles data in real-time as it arrives. Key tools: Kafka Streams (native Kafka, stateful), Apache Flink (low latency, exactly-once), Spark Streaming (micro-batch). Key concepts: windowing (tumbling, sliding, session), watermarks (handling late data), state management, exactly-once semantics. Use when: real-time dashboards, fraud detection, alerting, live recommendations.

---

## How to Explain Like a Senior Developer

"Stream processing handles data as it arrives rather than in scheduled batches. Think of it as a continuous pipeline where events flow through transformations. The main challenges are: handling out-of-order data (events arrive late due to network delays), managing state across events (counting, aggregating), and ensuring exactly-once processing (no duplicates, no data loss). We use 'windows' to group events by time - tumbling windows (fixed, non-overlapping), sliding windows (overlapping), or session windows (based on activity gaps). 'Watermarks' tell us when we can consider a time window complete, even with late arrivals. Kafka Streams is great for Kafka-centric architectures, Flink for complex event processing with low latency. The tradeoff vs batch: streaming is more complex but enables real-time insights."

---

## Core Implementation

### Kafka Streams with TypeScript

```typescript
// stream_processing/kafka-streams.ts

import { Kafka, Consumer, Producer, EachMessagePayload } from 'kafkajs';

interface OrderEvent {
  orderId: string;
  userId: string;
  amount: number;
  timestamp: number;
  productId: string;
}

interface AggregatedMetrics {
  windowStart: number;
  windowEnd: number;
  totalOrders: number;
  totalRevenue: number;
  avgOrderValue: number;
}

/**
 * Stream processing with KafkaJS
 * Implements windowed aggregations
 */
class StreamProcessor {
  private kafka: Kafka;
  private consumer: Consumer;
  private producer: Producer;
  
  // In-memory state (use RocksDB/Redis in production)
  private windowState: Map<string, AggregatedMetrics> = new Map();
  private windowDurationMs: number = 60000; // 1 minute windows
  
  constructor(config: { brokers: string[]; groupId: string }) {
    this.kafka = new Kafka({
      clientId: 'stream-processor',
      brokers: config.brokers,
    });
    
    this.consumer = this.kafka.consumer({ 
      groupId: config.groupId,
      sessionTimeout: 30000,
      heartbeatInterval: 3000,
    });
    
    this.producer = this.kafka.producer({
      idempotent: true, // Exactly-once semantics
      maxInFlightRequests: 5,
    });
  }
  
  async start(): Promise<void> {
    await this.consumer.connect();
    await this.producer.connect();
    
    await this.consumer.subscribe({ 
      topic: 'orders',
      fromBeginning: false,
    });
    
    // Process messages
    await this.consumer.run({
      eachMessage: async (payload) => {
        await this.processMessage(payload);
      },
    });
    
    // Emit window results periodically
    this.startWindowEmitter();
  }
  
  private async processMessage(payload: EachMessagePayload): Promise<void> {
    const { message } = payload;
    
    if (!message.value) return;
    
    const event: OrderEvent = JSON.parse(message.value.toString());
    
    // Determine which window this event belongs to
    const windowKey = this.getWindowKey(event.timestamp);
    
    // Update aggregations
    const current = this.windowState.get(windowKey) || {
      windowStart: this.getWindowStart(event.timestamp),
      windowEnd: this.getWindowStart(event.timestamp) + this.windowDurationMs,
      totalOrders: 0,
      totalRevenue: 0,
      avgOrderValue: 0,
    };
    
    current.totalOrders++;
    current.totalRevenue += event.amount;
    current.avgOrderValue = current.totalRevenue / current.totalOrders;
    
    this.windowState.set(windowKey, current);
  }
  
  private getWindowKey(timestamp: number): string {
    const windowStart = this.getWindowStart(timestamp);
    return `window-${windowStart}`;
  }
  
  private getWindowStart(timestamp: number): number {
    return Math.floor(timestamp / this.windowDurationMs) * this.windowDurationMs;
  }
  
  private startWindowEmitter(): void {
    // Emit completed windows every 10 seconds
    setInterval(async () => {
      const now = Date.now();
      const watermark = now - this.windowDurationMs - 10000; // 10s grace period
      
      for (const [key, metrics] of this.windowState.entries()) {
        // Window is complete (watermark passed)
        if (metrics.windowEnd < watermark) {
          await this.emitWindowResult(metrics);
          this.windowState.delete(key);
        }
      }
    }, 10000);
  }
  
  private async emitWindowResult(metrics: AggregatedMetrics): Promise<void> {
    await this.producer.send({
      topic: 'order-metrics',
      messages: [{
        key: `${metrics.windowStart}`,
        value: JSON.stringify(metrics),
        timestamp: `${Date.now()}`,
      }],
    });
    
    console.log(`Emitted window: ${new Date(metrics.windowStart).toISOString()}`);
  }
  
  async stop(): Promise<void> {
    await this.consumer.disconnect();
    await this.producer.disconnect();
  }
}

/**
 * Stateful stream processing with external state store
 */
class StatefulStreamProcessor {
  private redis: any; // Redis client
  
  constructor(redisClient: any) {
    this.redis = redisClient;
  }
  
  /**
   * Count events per user with Redis state
   */
  async processWithState(event: OrderEvent): Promise<number> {
    const key = `user:${event.userId}:order_count`;
    
    // Atomic increment in Redis
    const newCount = await this.redis.incr(key);
    
    // Set expiry for automatic cleanup
    await this.redis.expire(key, 86400); // 24 hours
    
    return newCount;
  }
  
  /**
   * Sliding window count
   */
  async slidingWindowCount(
    event: OrderEvent, 
    windowSizeMs: number
  ): Promise<number> {
    const key = `sliding:${event.userId}:orders`;
    const now = Date.now();
    const windowStart = now - windowSizeMs;
    
    // Add event to sorted set with timestamp as score
    await this.redis.zadd(key, now, `${event.orderId}:${now}`);
    
    // Remove old events outside window
    await this.redis.zremrangebyscore(key, '-inf', windowStart);
    
    // Get count in window
    const count = await this.redis.zcard(key);
    
    // Set expiry
    await this.redis.expire(key, Math.ceil(windowSizeMs / 1000) + 60);
    
    return count;
  }
  
  /**
   * Session window detection
   */
  async detectSession(
    event: OrderEvent,
    sessionGapMs: number
  ): Promise<{ sessionId: string; isNewSession: boolean }> {
    const key = `session:${event.userId}:last_activity`;
    
    const lastActivity = await this.redis.get(key);
    const now = Date.now();
    
    let sessionId: string;
    let isNewSession = false;
    
    if (!lastActivity || (now - parseInt(lastActivity)) > sessionGapMs) {
      // New session
      sessionId = `${event.userId}-${now}`;
      isNewSession = true;
      await this.redis.set(`session:${event.userId}:current`, sessionId);
    } else {
      // Existing session
      sessionId = await this.redis.get(`session:${event.userId}:current`);
    }
    
    // Update last activity
    await this.redis.set(key, now.toString());
    await this.redis.expire(key, Math.ceil(sessionGapMs / 1000) + 60);
    
    return { sessionId, isNewSession };
  }
}

// Usage
async function main() {
  const processor = new StreamProcessor({
    brokers: ['localhost:9092'],
    groupId: 'order-processor',
  });
  
  await processor.start();
  
  // Graceful shutdown
  process.on('SIGTERM', async () => {
    console.log('Shutting down...');
    await processor.stop();
    process.exit(0);
  });
}
```

### Apache Flink (Java-style concepts in TypeScript)

```typescript
// stream_processing/flink-concepts.ts

/**
 * Flink-style stream processing concepts
 * (Actual Flink uses Java/Scala, but concepts are portable)
 */

type EventTime = number;

interface Event<T> {
  data: T;
  eventTime: EventTime;
  processingTime: EventTime;
}

/**
 * Watermark generator for handling late data
 */
class WatermarkGenerator<T> {
  private maxTimestamp: EventTime = 0;
  private maxOutOfOrderness: number;
  
  constructor(maxOutOfOrdernessMs: number) {
    this.maxOutOfOrderness = maxOutOfOrdernessMs;
  }
  
  /**
   * Update watermark based on observed event
   */
  onEvent(event: Event<T>): void {
    this.maxTimestamp = Math.max(this.maxTimestamp, event.eventTime);
  }
  
  /**
   * Current watermark - events with timestamp <= watermark are considered complete
   */
  getCurrentWatermark(): EventTime {
    return this.maxTimestamp - this.maxOutOfOrderness;
  }
}

/**
 * Window types
 */
enum WindowType {
  TUMBLING = 'tumbling',  // Fixed, non-overlapping
  SLIDING = 'sliding',    // Fixed, overlapping
  SESSION = 'session',    // Variable, based on activity
}

interface Window {
  type: WindowType;
  start: EventTime;
  end: EventTime;
}

class TumblingWindow implements Window {
  type = WindowType.TUMBLING;
  start: EventTime;
  end: EventTime;
  
  constructor(timestamp: EventTime, sizeMs: number) {
    this.start = Math.floor(timestamp / sizeMs) * sizeMs;
    this.end = this.start + sizeMs;
  }
}

class SlidingWindow implements Window {
  type = WindowType.SLIDING;
  start: EventTime;
  end: EventTime;
  
  constructor(timestamp: EventTime, sizeMs: number, slideMs: number) {
    // Event belongs to multiple windows
    const windowStart = Math.floor(timestamp / slideMs) * slideMs;
    this.start = windowStart;
    this.end = windowStart + sizeMs;
  }
  
  /**
   * Get all windows this event belongs to
   */
  static getWindowsForEvent(
    timestamp: EventTime, 
    sizeMs: number, 
    slideMs: number
  ): SlidingWindow[] {
    const windows: SlidingWindow[] = [];
    const lastWindowStart = Math.floor(timestamp / slideMs) * slideMs;
    
    for (let start = lastWindowStart; start > timestamp - sizeMs; start -= slideMs) {
      if (start >= 0) {
        const window = new SlidingWindow(timestamp, sizeMs, slideMs);
        window.start = start;
        window.end = start + sizeMs;
        windows.push(window);
      }
    }
    
    return windows;
  }
}

/**
 * Windowed aggregation operator
 */
class WindowedAggregator<T, ACC, R> {
  private windowState: Map<string, ACC> = new Map();
  private watermarkGenerator: WatermarkGenerator<T>;
  private lateEventHandler: (event: Event<T>) => void;
  
  constructor(
    private windowAssigner: (event: Event<T>) => Window[],
    private accumulator: () => ACC,
    private aggregate: (acc: ACC, event: T) => ACC,
    private finalize: (acc: ACC) => R,
    maxOutOfOrdernessMs: number
  ) {
    this.watermarkGenerator = new WatermarkGenerator(maxOutOfOrdernessMs);
    this.lateEventHandler = () => {}; // Default: drop late events
  }
  
  /**
   * Process incoming event
   */
  process(event: Event<T>): void {
    this.watermarkGenerator.onEvent(event);
    const watermark = this.watermarkGenerator.getCurrentWatermark();
    
    // Check if event is late
    const windows = this.windowAssigner(event);
    
    for (const window of windows) {
      if (window.end <= watermark) {
        // Late event - window already closed
        this.lateEventHandler(event);
        continue;
      }
      
      const key = `${window.start}-${window.end}`;
      let acc = this.windowState.get(key);
      
      if (!acc) {
        acc = this.accumulator();
      }
      
      this.windowState.set(key, this.aggregate(acc, event.data));
    }
  }
  
  /**
   * Emit results for completed windows
   */
  emitCompletedWindows(): R[] {
    const results: R[] = [];
    const watermark = this.watermarkGenerator.getCurrentWatermark();
    
    for (const [key, acc] of this.windowState.entries()) {
      const [start, end] = key.split('-').map(Number);
      
      if (end <= watermark) {
        results.push(this.finalize(acc));
        this.windowState.delete(key);
      }
    }
    
    return results;
  }
  
  /**
   * Handle late events (side output)
   */
  onLateEvent(handler: (event: Event<T>) => void): void {
    this.lateEventHandler = handler;
  }
}

/**
 * Example: Real-time fraud detection
 */
interface Transaction {
  userId: string;
  amount: number;
  merchantId: string;
  location: string;
}

class FraudDetector {
  private userLastLocation: Map<string, { location: string; time: number }> = new Map();
  private userTransactionCount: Map<string, number[]> = new Map(); // timestamps
  
  /**
   * Detect suspicious patterns in real-time
   */
  detect(event: Event<Transaction>): string[] {
    const alerts: string[] = [];
    const { userId, amount, location } = event.data;
    
    // Rule 1: Impossible travel (location change too fast)
    const lastLocation = this.userLastLocation.get(userId);
    if (lastLocation && lastLocation.location !== location) {
      const timeDiff = event.eventTime - lastLocation.time;
      if (timeDiff < 60000) { // Less than 1 minute
        alerts.push(`IMPOSSIBLE_TRAVEL: User ${userId} in ${lastLocation.location} and ${location} within 1 minute`);
      }
    }
    this.userLastLocation.set(userId, { location, time: event.eventTime });
    
    // Rule 2: High frequency (more than 5 transactions in 1 minute)
    let timestamps = this.userTransactionCount.get(userId) || [];
    timestamps = timestamps.filter(t => event.eventTime - t < 60000);
    timestamps.push(event.eventTime);
    this.userTransactionCount.set(userId, timestamps);
    
    if (timestamps.length > 5) {
      alerts.push(`HIGH_FREQUENCY: User ${userId} made ${timestamps.length} transactions in 1 minute`);
    }
    
    // Rule 3: Large amount
    if (amount > 10000) {
      alerts.push(`LARGE_AMOUNT: User ${userId} transaction of ${amount}`);
    }
    
    return alerts;
  }
}
```

---

## Real-World Scenarios

### Scenario 1: Real-time Analytics Dashboard

```typescript
// stream_processing/realtime-analytics.ts

import { Kafka } from 'kafkajs';
import { WebSocket, WebSocketServer } from 'ws';

interface PageViewEvent {
  sessionId: string;
  userId?: string;
  page: string;
  referrer: string;
  timestamp: number;
  userAgent: string;
}

interface RealtimeMetrics {
  activeUsers: number;
  pageViewsPerMinute: number;
  topPages: Array<{ page: string; views: number }>;
  topReferrers: Array<{ referrer: string; count: number }>;
}

class RealtimeAnalyticsPipeline {
  private kafka: Kafka;
  private wss: WebSocketServer;
  
  // Sliding window state (1 minute)
  private pageViews: Map<string, number> = new Map();
  private activeSessions: Set<string> = new Set();
  private referrerCounts: Map<string, number> = new Map();
  private lastCleanup: number = Date.now();
  
  constructor() {
    this.kafka = new Kafka({
      clientId: 'analytics-processor',
      brokers: ['localhost:9092'],
    });
    
    this.wss = new WebSocketServer({ port: 8080 });
  }
  
  async start(): Promise<void> {
    const consumer = this.kafka.consumer({ groupId: 'analytics' });
    await consumer.connect();
    await consumer.subscribe({ topic: 'pageviews' });
    
    // Process events
    await consumer.run({
      eachMessage: async ({ message }) => {
        if (!message.value) return;
        
        const event: PageViewEvent = JSON.parse(message.value.toString());
        this.processEvent(event);
      },
    });
    
    // Broadcast metrics every second
    setInterval(() => this.broadcastMetrics(), 1000);
    
    // Cleanup old data every 10 seconds
    setInterval(() => this.cleanup(), 10000);
  }
  
  private processEvent(event: PageViewEvent): void {
    const now = Date.now();
    
    // Track active sessions
    this.activeSessions.add(event.sessionId);
    
    // Count page views
    const currentCount = this.pageViews.get(event.page) || 0;
    this.pageViews.set(event.page, currentCount + 1);
    
    // Track referrers
    if (event.referrer) {
      const refCount = this.referrerCounts.get(event.referrer) || 0;
      this.referrerCounts.set(event.referrer, refCount + 1);
    }
  }
  
  private cleanup(): void {
    // In production, use proper time-based expiration
    // This is simplified for illustration
    const now = Date.now();
    
    if (now - this.lastCleanup > 60000) {
      // Reset counts every minute (tumbling window)
      this.pageViews.clear();
      this.referrerCounts.clear();
      this.activeSessions.clear();
      this.lastCleanup = now;
    }
  }
  
  private getMetrics(): RealtimeMetrics {
    // Sort pages by views
    const topPages = Array.from(this.pageViews.entries())
      .sort((a, b) => b[1] - a[1])
      .slice(0, 10)
      .map(([page, views]) => ({ page, views }));
    
    // Sort referrers
    const topReferrers = Array.from(this.referrerCounts.entries())
      .sort((a, b) => b[1] - a[1])
      .slice(0, 5)
      .map(([referrer, count]) => ({ referrer, count }));
    
    return {
      activeUsers: this.activeSessions.size,
      pageViewsPerMinute: Array.from(this.pageViews.values())
        .reduce((a, b) => a + b, 0),
      topPages,
      topReferrers,
    };
  }
  
  private broadcastMetrics(): void {
    const metrics = this.getMetrics();
    const message = JSON.stringify(metrics);
    
    this.wss.clients.forEach((client) => {
      if (client.readyState === WebSocket.OPEN) {
        client.send(message);
      }
    });
  }
}
```

### Scenario 2: Exactly-Once Processing

```typescript
// stream_processing/exactly-once.ts

import { Kafka, Producer, Consumer, EachMessagePayload } from 'kafkajs';

/**
 * Exactly-once semantics with Kafka transactions
 * Guarantees: no duplicates, no data loss
 */
class ExactlyOnceProcessor {
  private kafka: Kafka;
  private producer: Producer;
  private consumer: Consumer;
  
  constructor(brokers: string[]) {
    this.kafka = new Kafka({
      clientId: 'exactly-once-processor',
      brokers,
    });
    
    // Transactional producer
    this.producer = this.kafka.producer({
      transactionalId: 'my-transactional-producer',
      idempotent: true,
      maxInFlightRequests: 1,
    });
    
    // Consumer with read_committed isolation
    this.consumer = this.kafka.consumer({
      groupId: 'exactly-once-group',
      readUncommitted: false, // Only read committed messages
    });
  }
  
  async start(): Promise<void> {
    await this.producer.connect();
    await this.consumer.connect();
    await this.consumer.subscribe({ topic: 'input-topic' });
    
    await this.consumer.run({
      eachBatchAutoResolve: false, // Manual offset management
      eachBatch: async ({ batch, resolveOffset, heartbeat, commitOffsetsIfNecessary }) => {
        for (const message of batch.messages) {
          // Start transaction
          const transaction = await this.producer.transaction();
          
          try {
            // Process message
            const result = await this.processMessage(message);
            
            // Send to output topic within transaction
            await transaction.send({
              topic: 'output-topic',
              messages: [{ value: JSON.stringify(result) }],
            });
            
            // Commit consumer offset within same transaction
            await transaction.sendOffsets({
              consumerGroupId: 'exactly-once-group',
              topics: [{
                topic: batch.topic,
                partitions: [{
                  partition: batch.partition,
                  offset: (Number(message.offset) + 1).toString(),
                }],
              }],
            });
            
            // Commit transaction atomically
            await transaction.commit();
            
            resolveOffset(message.offset);
            await heartbeat();
            
          } catch (error) {
            // Abort transaction on error
            await transaction.abort();
            throw error;
          }
        }
        
        await commitOffsetsIfNecessary();
      },
    });
  }
  
  private async processMessage(message: any): Promise<any> {
    const value = JSON.parse(message.value.toString());
    // Transform
    return {
      ...value,
      processedAt: Date.now(),
      processedBy: 'exactly-once-processor',
    };
  }
}

/**
 * Idempotent processing with deduplication
 */
class IdempotentProcessor {
  private processedIds: Set<string> = new Set();
  private redis: any; // Redis client for distributed dedup
  
  constructor(redisClient: any) {
    this.redis = redisClient;
  }
  
  /**
   * Process with deduplication
   */
  async processIdempotent<T extends { id: string }>(
    event: T,
    processor: (event: T) => Promise<void>
  ): Promise<boolean> {
    const dedupKey = `processed:${event.id}`;
    
    // Check if already processed (atomic operation)
    const wasSet = await this.redis.set(
      dedupKey,
      '1',
      'NX', // Only set if not exists
      'EX', // Set expiry
      86400 // 24 hours
    );
    
    if (!wasSet) {
      console.log(`Duplicate event ${event.id}, skipping`);
      return false;
    }
    
    try {
      await processor(event);
      return true;
    } catch (error) {
      // Remove dedup key on failure to allow retry
      await this.redis.del(dedupKey);
      throw error;
    }
  }
}
```

---

## Common Pitfalls

### 1. Ignoring Late Data

```typescript
// ❌ BAD: Dropping all late events silently
if (event.timestamp < watermark) {
  return; // Lost data!
}

// ✅ GOOD: Handle late data appropriately
if (event.timestamp < watermark) {
  // Option 1: Side output for manual handling
  await sendToLateDataTopic(event);
  
  // Option 2: Update aggregations if possible
  if (canUpdatePreviousWindow(event)) {
    await updatePreviousWindowResult(event);
  }
  
  // Option 3: Allow late data within grace period
  if (event.timestamp > watermark - gracePeriod) {
    processNormally(event);
  }
}
```

### 2. State Growing Unboundedly

```typescript
// ❌ BAD: State grows forever
class BadProcessor {
  private state: Map<string, any> = new Map();
  
  process(event: any) {
    this.state.set(event.key, event); // Never cleaned up!
  }
}

// ✅ GOOD: Implement state TTL
class GoodProcessor {
  private state: Map<string, { data: any; expiry: number }> = new Map();
  
  process(event: any) {
    const expiry = Date.now() + 3600000; // 1 hour TTL
    this.state.set(event.key, { data: event, expiry });
    
    // Periodic cleanup
    this.cleanup();
  }
  
  private cleanup() {
    const now = Date.now();
    for (const [key, value] of this.state.entries()) {
      if (value.expiry < now) {
        this.state.delete(key);
      }
    }
  }
}
```

### 3. Not Handling Backpressure

```typescript
// ❌ BAD: No backpressure handling
async function badProcessor(events: AsyncIterable<Event>) {
  for await (const event of events) {
    await heavyProcessing(event); // Overwhelms downstream
  }
}

// ✅ GOOD: Implement backpressure
class BackpressureHandler {
  private queue: Event[] = [];
  private maxQueueSize = 10000;
  private processing = false;
  
  async enqueue(event: Event): Promise<boolean> {
    if (this.queue.length >= this.maxQueueSize) {
      // Apply backpressure - reject or buffer to disk
      console.warn('Queue full, applying backpressure');
      return false;
    }
    
    this.queue.push(event);
    this.processQueue();
    return true;
  }
  
  private async processQueue() {
    if (this.processing) return;
    this.processing = true;
    
    while (this.queue.length > 0) {
      const batch = this.queue.splice(0, 100);
      await this.processBatch(batch);
    }
    
    this.processing = false;
  }
}
```

---

## Interview Questions

### Q1: What are the differences between event time and processing time?

**A:** **Event time** is when the event actually occurred (embedded in the event). **Processing time** is when the system processes the event. They differ due to network delays, buffering, retries. Use event time for accurate analytics (e.g., "sales per hour"). Use processing time for operational metrics (e.g., "throughput"). Watermarks track event time progress and trigger window computations.

### Q2: Explain exactly-once semantics in stream processing.

**A:** Exactly-once means each event affects the output exactly once - no duplicates (at-least-once) and no loss (at-most-once). Achieved through: 1) **Idempotent operations** (same input always produces same output). 2) **Transactions** (atomic writes to multiple systems). 3) **Deduplication** (track processed event IDs). Kafka achieves this with transactional producers and consumer offset commits in the same transaction.

### Q3: When would you use stream processing vs batch processing?

**A:** Use **streaming** when: real-time insights needed (dashboards, alerts), continuous data flow, low latency required (fraud detection), event-driven architecture. Use **batch** when: latency tolerance (hours), complete dataset analysis, complex multi-pass algorithms, cost-sensitive (batch is cheaper). Many systems use **Lambda architecture** (batch + stream) or **Kappa architecture** (stream only).

### Q4: How do you handle out-of-order events?

**A:** 1) **Watermarks**: Define expected delay, process windows when watermark passes. 2) **Allowed lateness**: Accept late events within grace period. 3) **Retractions**: Update previous results when late data arrives. 4) **Session windows**: Naturally handle out-of-order within session gap. 5) **Event time indexing**: Store by event time, not arrival time. Trade-off: longer wait for accuracy vs lower latency.

---

## Quick Reference Checklist

### Window Types
- [ ] Tumbling: Fixed size, non-overlapping
- [ ] Sliding: Fixed size, overlapping
- [ ] Session: Variable, activity-based

### State Management
- [ ] Use external store for large state
- [ ] Implement TTL for cleanup
- [ ] Handle state recovery on failure

### Delivery Guarantees
- [ ] At-most-once: Fire and forget
- [ ] At-least-once: Retry until ACK
- [ ] Exactly-once: Transactions + dedup

### Operations
- [ ] Monitor lag (consumer behind producer)
- [ ] Handle backpressure
- [ ] Plan for late data
- [ ] Test failure scenarios

---

*Last updated: February 2026*

