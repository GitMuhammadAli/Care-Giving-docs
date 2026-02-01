# 📬 Message Queues - Complete Guide

> A comprehensive guide to message queues - RabbitMQ, Amazon SQS, Redis queues, BullMQ, job processing, priorities, retries, and dead letter queues.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Message queues are middleware that enable asynchronous communication between services by storing messages until consumers are ready to process them, providing decoupling, load leveling, and guaranteed delivery in distributed systems."

### The 7 Key Concepts (Remember These!)
```
1. PRODUCER       → Sends messages to the queue
2. CONSUMER       → Receives and processes messages
3. BROKER         → The queue server (RabbitMQ, SQS, Redis)
4. ACKNOWLEDGMENT → Consumer confirms message processed
5. DEAD LETTER    → Queue for failed/unprocessable messages
6. VISIBILITY     → Message hidden while being processed
7. AT-LEAST-ONCE  → Messages delivered at least once (may duplicate)
```

### Message Queue Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                  MESSAGE QUEUE ARCHITECTURE                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   PRODUCERS                    BROKER                CONSUMERS  │
│   ─────────                    ──────                ─────────  │
│                                                                 │
│   ┌─────────┐                                      ┌─────────┐ │
│   │ API     │──┐                               ┌──▶│Worker 1 │ │
│   │ Server  │  │   ┌──────────────────────┐   │   └─────────┘ │
│   └─────────┘  │   │                      │   │               │
│                ├──▶│     MESSAGE QUEUE    │───┤   ┌─────────┐ │
│   ┌─────────┐  │   │                      │   └──▶│Worker 2 │ │
│   │ Webhook │──┤   │  ┌─┐ ┌─┐ ┌─┐ ┌─┐    │       └─────────┘ │
│   │ Handler │  │   │  │M│ │M│ │M│ │M│    │                   │
│   └─────────┘  │   │  └─┘ └─┘ └─┘ └─┘    │       ┌─────────┐ │
│                │   └──────────────────────┘   ┌──▶│Worker 3 │ │
│   ┌─────────┐  │                              │   └─────────┘ │
│   │ Cron    │──┘                              │               │
│   │ Job     │       ┌──────────────────────┐  │               │
│   └─────────┘       │   DEAD LETTER QUEUE  │◀─┘               │
│                     │   (failed messages)   │    (after retry │
│                     └──────────────────────┘     failures)    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Queue Types Comparison
```
┌─────────────────────────────────────────────────────────────────┐
│              MESSAGE QUEUE COMPARISON                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  RabbitMQ                                                       │
│  ─────────                                                      │
│  • AMQP protocol (feature-rich)                                │
│  • Exchanges: direct, fanout, topic, headers                   │
│  • Complex routing, message priorities                         │
│  • Best for: Complex routing, enterprise messaging             │
│                                                                 │
│  Amazon SQS                                                     │
│  ──────────                                                     │
│  • Fully managed, infinite scale                               │
│  • Standard (at-least-once) vs FIFO (exactly-once)             │
│  • 256KB max message, 14 day retention                         │
│  • Best for: AWS-native, serverless, simple queuing            │
│                                                                 │
│  Redis (BullMQ/Bull)                                           │
│  ──────────────────                                             │
│  • Uses Redis lists/streams                                    │
│  • Great for job queues, priorities, delayed jobs              │
│  • Dashboard UI (Bull Board)                                   │
│  • Best for: Job processing, already using Redis               │
│                                                                 │
│  Apache Kafka                                                   │
│  ────────────                                                   │
│  • Distributed log, not traditional queue                      │
│  • High throughput, replay capability                          │
│  • Consumer groups, partitioning                               │
│  • Best for: Event streaming, high volume                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"At-least-once delivery"** | "We design for at-least-once delivery with idempotent consumers" |
| **"Visibility timeout"** | "Set visibility timeout longer than max processing time" |
| **"Dead letter queue"** | "Failed messages go to the DLQ for investigation" |
| **"Backpressure"** | "We implement backpressure when consumers can't keep up" |
| **"Competing consumers"** | "We use competing consumers pattern for horizontal scaling" |
| **"Message acknowledgment"** | "Messages are acknowledged only after successful processing" |
| **"Exponential backoff"** | "Retries use exponential backoff to avoid thundering herd" |
| **"Idempotent consumer"** | "Consumers must be idempotent to handle redelivery" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| SQS max message size | **256 KB** | Use S3 for larger payloads |
| SQS retention | **4-14 days** | Default 4, max 14 |
| SQS visibility timeout | **0-12 hours** | Default 30 seconds |
| RabbitMQ message size | **128 MB** | Practical limit |
| BullMQ job attempts | **3 default** | Configurable |
| Typical retry delays | **1s, 5s, 30s, 5m** | Exponential backoff |

### The "Wow" Statement (Memorize This!)
> "We use BullMQ on Redis for our job queue with a carefully designed flow. Jobs have priorities - critical notifications process first. We use exponential backoff for retries: 1s, 10s, 1min, 10min. After 5 failures, jobs go to a dead letter queue where we have alerting and a manual review dashboard. Each job is idempotent - if it runs twice, same result. We use job deduplication to prevent duplicate submissions. For scaling, we run multiple workers with concurrency of 10 each, and use rate limiting to protect downstream APIs. The dashboard shows queue depth, processing rate, and error rate in real-time."

### Quick Architecture Drawing (Draw This!)
```
Producer                    Queue                     Consumer
   │                          │                          │
   │──── enqueue(job) ───────▶│                          │
   │                          │                          │
   │                          │◀──── dequeue() ──────────│
   │                          │                          │
   │                          │         [process]        │
   │                          │                          │
   │                          │◀──── ack(success) ───────│
   │                          │         OR               │
   │                          │◀──── nack(retry) ────────│
   │                          │                          │
   │                          │────▶ [back to queue]     │
   │                          │         OR               │
   │                          │────▶ [to DLQ after max]  │
```

### Interview Rapid Fire (Practice These!)

**Q: "What is a message queue?"**
> "Middleware that stores messages until consumers process them. Decouples services, handles load spikes, enables async processing."

**Q: "At-least-once vs exactly-once?"**
> "At-least-once: message delivered at least once, may duplicate. Exactly-once: harder, needs idempotency or transactions. Most queues are at-least-once."

**Q: "What is a dead letter queue?"**
> "A queue for messages that failed processing after max retries. Used for debugging, alerting, and manual reprocessing."

**Q: "How do you handle poison messages?"**
> "Messages that repeatedly fail. After N retries, move to DLQ. Alert on DLQ depth. Have process to investigate and fix or discard."

**Q: "RabbitMQ vs SQS vs Redis?"**
> "RabbitMQ: complex routing, AMQP. SQS: managed, infinite scale, AWS-native. Redis (BullMQ): job queues, already have Redis, need dashboard."

**Q: "How do you scale consumers?"**
> "Competing consumers pattern - multiple workers pull from same queue. Add workers when queue depth grows. Use auto-scaling based on queue metrics."

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "What is a message queue?"

**Junior Answer:**
> "It's like a to-do list for your app - you put messages in and workers process them."

**Senior Answer:**
> "A message queue is middleware that enables asynchronous communication between services. Producers send messages to the queue, consumers pull and process them. This provides:

1. **Decoupling** - Producer doesn't need to know about consumers
2. **Load leveling** - Absorb traffic spikes, process at sustainable rate
3. **Reliability** - Messages persist until acknowledged
4. **Scalability** - Add consumers to handle more load

The key guarantee is at-least-once delivery - a message will be delivered at least once, but possibly more if there's a failure before acknowledgment. This means consumers must be idempotent. Failed messages typically go to a dead letter queue for investigation."

### When Asked: "How do you choose between RabbitMQ, SQS, and Redis queues?"

**Junior Answer:**
> "I'd just use whatever the team is familiar with."

**Senior Answer:**
> "It depends on requirements:

**RabbitMQ** when you need:
- Complex routing (topic exchanges, headers)
- Message priorities
- AMQP protocol compliance
- On-premise deployment
- Fine-grained control

**Amazon SQS** when you need:
- Managed service, zero ops
- Infinite scalability
- AWS ecosystem integration
- FIFO ordering (SQS FIFO)
- Serverless architecture

**Redis with BullMQ** when you need:
- Job scheduling with delays/cron
- Already using Redis
- Rich dashboard UI
- Rate limiting per queue
- Simple setup for job processing

For most Node.js applications doing background job processing, I'd start with BullMQ - it's well-designed, has great TypeScript support, and the dashboard is excellent for monitoring."

### Follow-up Questions to Expect:

| They Ask | You Answer |
|----------|------------|
| "How do you handle message ordering?" | "SQS FIFO for strict order, or include sequence numbers and reorder in consumer. Most queues don't guarantee order in standard mode." |
| "What about exactly-once delivery?" | "True exactly-once is hard. Use idempotent consumers with deduplication keys. Some systems (Kafka, SQS FIFO) offer exactly-once semantics." |
| "How do you monitor queue health?" | "Track queue depth (backlog), processing rate, error rate, DLQ depth. Alert on growing backlog or DLQ messages. Dashboard for visibility." |
| "What if the queue broker goes down?" | "For RabbitMQ: clustering and mirrored queues. For Redis: Sentinel or Cluster mode. For SQS: it's managed and highly available." |

---

## 📚 Table of Contents

1. [RabbitMQ](#1-rabbitmq)
2. [Amazon SQS](#2-amazon-sqs)
3. [BullMQ (Redis)](#3-bullmq-redis)
4. [Job Processing Patterns](#4-job-processing-patterns)
5. [Retry Strategies](#5-retry-strategies)
6. [Dead Letter Queues](#6-dead-letter-queues)
7. [Scaling Consumers](#7-scaling-consumers)
8. [Monitoring & Alerting](#8-monitoring--alerting)
9. [Common Pitfalls](#9-common-pitfalls)
10. [Interview Questions](#10-interview-questions)

---

## 1. RabbitMQ

### Connection and Basic Operations

```typescript
// ════════════════════════════════════════════════════════════════
// RABBITMQ WITH AMQPLIB
// ════════════════════════════════════════════════════════════════

import amqp, { Connection, Channel, ConsumeMessage } from 'amqplib';

class RabbitMQClient {
  private connection: Connection | null = null;
  private channel: Channel | null = null;
  private readonly url: string;

  constructor(url: string = process.env.RABBITMQ_URL || 'amqp://localhost') {
    this.url = url;
  }

  async connect(): Promise<void> {
    this.connection = await amqp.connect(this.url);
    this.channel = await this.connection.createChannel();

    // Handle connection errors
    this.connection.on('error', (err) => {
      console.error('RabbitMQ connection error:', err);
      this.reconnect();
    });

    this.connection.on('close', () => {
      console.log('RabbitMQ connection closed');
      this.reconnect();
    });

    console.log('Connected to RabbitMQ');
  }

  private async reconnect(): Promise<void> {
    console.log('Attempting to reconnect to RabbitMQ...');
    setTimeout(async () => {
      try {
        await this.connect();
      } catch (error) {
        console.error('Reconnection failed:', error);
        this.reconnect();
      }
    }, 5000);
  }

  // ══════════════════════════════════════════════════════════════
  // QUEUE OPERATIONS
  // ══════════════════════════════════════════════════════════════

  async assertQueue(queueName: string, options?: amqp.Options.AssertQueue): Promise<void> {
    await this.channel!.assertQueue(queueName, {
      durable: true,        // Survive broker restart
      ...options
    });
  }

  async sendToQueue(queueName: string, message: any, options?: amqp.Options.Publish): Promise<boolean> {
    const buffer = Buffer.from(JSON.stringify(message));
    
    return this.channel!.sendToQueue(queueName, buffer, {
      persistent: true,     // Survive broker restart
      contentType: 'application/json',
      ...options
    });
  }

  async consume(
    queueName: string,
    handler: (msg: any, originalMsg: ConsumeMessage) => Promise<void>,
    options?: amqp.Options.Consume
  ): Promise<void> {
    await this.channel!.prefetch(1); // Process one at a time

    await this.channel!.consume(queueName, async (msg) => {
      if (!msg) return;

      try {
        const content = JSON.parse(msg.content.toString());
        await handler(content, msg);
        this.channel!.ack(msg);
      } catch (error) {
        console.error('Message processing failed:', error);
        
        // Requeue on first failure, reject after
        const requeue = !msg.fields.redelivered;
        this.channel!.nack(msg, false, requeue);
      }
    }, options);
  }

  // ══════════════════════════════════════════════════════════════
  // EXCHANGE PATTERNS
  // ══════════════════════════════════════════════════════════════

  // Direct Exchange - Route by exact routing key
  async setupDirectExchange(exchangeName: string): Promise<void> {
    await this.channel!.assertExchange(exchangeName, 'direct', { durable: true });
  }

  async publishDirect(exchange: string, routingKey: string, message: any): Promise<boolean> {
    return this.channel!.publish(
      exchange,
      routingKey,
      Buffer.from(JSON.stringify(message)),
      { persistent: true }
    );
  }

  async bindQueueToExchange(queue: string, exchange: string, routingKey: string): Promise<void> {
    await this.channel!.bindQueue(queue, exchange, routingKey);
  }

  // Fanout Exchange - Broadcast to all bound queues
  async setupFanoutExchange(exchangeName: string): Promise<void> {
    await this.channel!.assertExchange(exchangeName, 'fanout', { durable: true });
  }

  async publishFanout(exchange: string, message: any): Promise<boolean> {
    return this.channel!.publish(
      exchange,
      '',  // Routing key ignored for fanout
      Buffer.from(JSON.stringify(message)),
      { persistent: true }
    );
  }

  // Topic Exchange - Route by pattern matching
  async setupTopicExchange(exchangeName: string): Promise<void> {
    await this.channel!.assertExchange(exchangeName, 'topic', { durable: true });
  }

  async publishTopic(exchange: string, routingKey: string, message: any): Promise<boolean> {
    // Routing keys like: orders.created, orders.shipped, users.signup
    return this.channel!.publish(
      exchange,
      routingKey,
      Buffer.from(JSON.stringify(message)),
      { persistent: true }
    );
  }

  // Bind with patterns: 
  // "orders.*" matches orders.created, orders.shipped
  // "*.created" matches orders.created, users.created
  // "#" matches everything

  async close(): Promise<void> {
    await this.channel?.close();
    await this.connection?.close();
  }
}

// ════════════════════════════════════════════════════════════════
// USAGE EXAMPLE
// ════════════════════════════════════════════════════════════════

async function main() {
  const rabbit = new RabbitMQClient();
  await rabbit.connect();

  // Setup queue
  await rabbit.assertQueue('emails');

  // Producer
  await rabbit.sendToQueue('emails', {
    to: 'user@example.com',
    subject: 'Welcome!',
    body: 'Thanks for signing up'
  });

  // Consumer
  await rabbit.consume('emails', async (message) => {
    console.log('Sending email to:', message.to);
    await sendEmail(message);
  });
}

// ════════════════════════════════════════════════════════════════
// TOPIC EXCHANGE EXAMPLE
// ════════════════════════════════════════════════════════════════

async function topicExample() {
  const rabbit = new RabbitMQClient();
  await rabbit.connect();

  // Setup
  await rabbit.setupTopicExchange('events');
  await rabbit.assertQueue('order-notifications');
  await rabbit.assertQueue('audit-log');

  // Bind queues with patterns
  await rabbit.bindQueueToExchange('order-notifications', 'events', 'orders.*');
  await rabbit.bindQueueToExchange('audit-log', 'events', '#'); // All events

  // Publish events
  await rabbit.publishTopic('events', 'orders.created', { orderId: '123' });
  await rabbit.publishTopic('events', 'orders.shipped', { orderId: '123' });
  await rabbit.publishTopic('events', 'users.signup', { userId: '456' });

  // order-notifications receives: orders.created, orders.shipped
  // audit-log receives: all three
}
```

### RabbitMQ with Dead Letter Exchange

```typescript
// ════════════════════════════════════════════════════════════════
// RABBITMQ DEAD LETTER EXCHANGE
// ════════════════════════════════════════════════════════════════

async function setupWithDeadLetter(rabbit: RabbitMQClient, channel: Channel) {
  // Create dead letter exchange
  await channel.assertExchange('dlx', 'direct', { durable: true });
  
  // Create dead letter queue
  await channel.assertQueue('emails-dlq', { durable: true });
  await channel.bindQueue('emails-dlq', 'dlx', 'emails');

  // Create main queue with dead letter config
  await channel.assertQueue('emails', {
    durable: true,
    arguments: {
      'x-dead-letter-exchange': 'dlx',
      'x-dead-letter-routing-key': 'emails',
      'x-message-ttl': 86400000,  // Optional: 24 hour TTL
      'x-max-length': 10000       // Optional: max queue size
    }
  });

  // Consumer with retry logic
  await channel.consume('emails', async (msg) => {
    if (!msg) return;

    const retryCount = (msg.properties.headers?.['x-retry-count'] || 0) as number;
    const maxRetries = 3;

    try {
      const content = JSON.parse(msg.content.toString());
      await processEmail(content);
      channel.ack(msg);
    } catch (error) {
      console.error(`Processing failed (attempt ${retryCount + 1}):`, error);

      if (retryCount < maxRetries) {
        // Retry with delay
        const delay = Math.pow(2, retryCount) * 1000; // Exponential backoff

        setTimeout(() => {
          channel.sendToQueue('emails', msg.content, {
            persistent: true,
            headers: {
              ...msg.properties.headers,
              'x-retry-count': retryCount + 1,
              'x-last-error': (error as Error).message
            }
          });
        }, delay);

        channel.ack(msg); // Ack original
      } else {
        // Max retries - goes to DLQ
        channel.nack(msg, false, false);
      }
    }
  });
}
```

---

## 2. Amazon SQS

### Basic SQS Operations

```typescript
// ════════════════════════════════════════════════════════════════
// AMAZON SQS CLIENT
// ════════════════════════════════════════════════════════════════

import {
  SQSClient,
  SendMessageCommand,
  ReceiveMessageCommand,
  DeleteMessageCommand,
  GetQueueAttributesCommand,
  CreateQueueCommand,
  Message
} from '@aws-sdk/client-sqs';

class SQSQueue {
  private client: SQSClient;
  private queueUrl: string;

  constructor(queueUrl: string, region: string = 'us-east-1') {
    this.queueUrl = queueUrl;
    this.client = new SQSClient({ region });
  }

  // ══════════════════════════════════════════════════════════════
  // SEND MESSAGE
  // ══════════════════════════════════════════════════════════════

  async send(message: any, options: {
    delaySeconds?: number;
    deduplicationId?: string;  // Required for FIFO
    groupId?: string;          // Required for FIFO
  } = {}): Promise<string> {
    const command = new SendMessageCommand({
      QueueUrl: this.queueUrl,
      MessageBody: JSON.stringify(message),
      DelaySeconds: options.delaySeconds,
      MessageDeduplicationId: options.deduplicationId,
      MessageGroupId: options.groupId,
      MessageAttributes: {
        'Timestamp': {
          DataType: 'Number',
          StringValue: Date.now().toString()
        }
      }
    });

    const response = await this.client.send(command);
    return response.MessageId!;
  }

  // Send batch (up to 10 messages)
  async sendBatch(messages: any[]): Promise<void> {
    const { SendMessageBatchCommand } = await import('@aws-sdk/client-sqs');

    // SQS allows max 10 messages per batch
    const batches = this.chunk(messages, 10);

    for (const batch of batches) {
      const command = new SendMessageBatchCommand({
        QueueUrl: this.queueUrl,
        Entries: batch.map((msg, i) => ({
          Id: `msg-${i}`,
          MessageBody: JSON.stringify(msg)
        }))
      });

      await this.client.send(command);
    }
  }

  // ══════════════════════════════════════════════════════════════
  // RECEIVE MESSAGES
  // ══════════════════════════════════════════════════════════════

  async receive(options: {
    maxMessages?: number;       // 1-10, default 1
    waitTimeSeconds?: number;   // 0-20 for long polling
    visibilityTimeout?: number; // How long to hide message
  } = {}): Promise<Message[]> {
    const command = new ReceiveMessageCommand({
      QueueUrl: this.queueUrl,
      MaxNumberOfMessages: options.maxMessages || 10,
      WaitTimeSeconds: options.waitTimeSeconds || 20,  // Long polling
      VisibilityTimeout: options.visibilityTimeout || 30,
      MessageAttributeNames: ['All'],
      AttributeNames: ['All']
    });

    const response = await this.client.send(command);
    return response.Messages || [];
  }

  // ══════════════════════════════════════════════════════════════
  // DELETE MESSAGE (Acknowledge)
  // ══════════════════════════════════════════════════════════════

  async delete(receiptHandle: string): Promise<void> {
    const command = new DeleteMessageCommand({
      QueueUrl: this.queueUrl,
      ReceiptHandle: receiptHandle
    });

    await this.client.send(command);
  }

  async deleteBatch(receiptHandles: string[]): Promise<void> {
    const { DeleteMessageBatchCommand } = await import('@aws-sdk/client-sqs');

    const batches = this.chunk(receiptHandles, 10);

    for (const batch of batches) {
      const command = new DeleteMessageBatchCommand({
        QueueUrl: this.queueUrl,
        Entries: batch.map((handle, i) => ({
          Id: `msg-${i}`,
          ReceiptHandle: handle
        }))
      });

      await this.client.send(command);
    }
  }

  // ══════════════════════════════════════════════════════════════
  // QUEUE STATS
  // ══════════════════════════════════════════════════════════════

  async getStats(): Promise<{
    messagesAvailable: number;
    messagesInFlight: number;
    messagesDelayed: number;
  }> {
    const command = new GetQueueAttributesCommand({
      QueueUrl: this.queueUrl,
      AttributeNames: [
        'ApproximateNumberOfMessages',
        'ApproximateNumberOfMessagesNotVisible',
        'ApproximateNumberOfMessagesDelayed'
      ]
    });

    const response = await this.client.send(command);
    
    return {
      messagesAvailable: parseInt(response.Attributes?.ApproximateNumberOfMessages || '0'),
      messagesInFlight: parseInt(response.Attributes?.ApproximateNumberOfMessagesNotVisible || '0'),
      messagesDelayed: parseInt(response.Attributes?.ApproximateNumberOfMessagesDelayed || '0')
    };
  }

  private chunk<T>(array: T[], size: number): T[][] {
    const chunks: T[][] = [];
    for (let i = 0; i < array.length; i += size) {
      chunks.push(array.slice(i, i + size));
    }
    return chunks;
  }
}

// ════════════════════════════════════════════════════════════════
// SQS CONSUMER
// ════════════════════════════════════════════════════════════════

class SQSConsumer {
  private running = false;
  private queue: SQSQueue;

  constructor(
    private queueUrl: string,
    private handler: (message: any) => Promise<void>,
    private options: {
      batchSize?: number;
      visibilityTimeout?: number;
      pollingWaitTime?: number;
    } = {}
  ) {
    this.queue = new SQSQueue(queueUrl);
  }

  async start(): Promise<void> {
    this.running = true;
    console.log('SQS Consumer started');

    while (this.running) {
      try {
        const messages = await this.queue.receive({
          maxMessages: this.options.batchSize || 10,
          waitTimeSeconds: this.options.pollingWaitTime || 20,
          visibilityTimeout: this.options.visibilityTimeout || 30
        });

        for (const message of messages) {
          await this.processMessage(message);
        }
      } catch (error) {
        console.error('Consumer error:', error);
        await this.sleep(5000);
      }
    }
  }

  private async processMessage(message: Message): Promise<void> {
    try {
      const body = JSON.parse(message.Body || '{}');
      await this.handler(body);
      await this.queue.delete(message.ReceiptHandle!);
    } catch (error) {
      console.error('Message processing failed:', error);
      // Message will become visible again after visibility timeout
      // After max receives, goes to DLQ if configured
    }
  }

  stop(): void {
    this.running = false;
    console.log('SQS Consumer stopping...');
  }

  private sleep(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// ════════════════════════════════════════════════════════════════
// USAGE
// ════════════════════════════════════════════════════════════════

async function main() {
  const queue = new SQSQueue(process.env.SQS_QUEUE_URL!);

  // Send message
  await queue.send({
    type: 'order.created',
    orderId: '12345',
    userId: '67890'
  });

  // Send with delay
  await queue.send(
    { type: 'reminder' },
    { delaySeconds: 60 }  // Deliver after 1 minute
  );

  // Consumer
  const consumer = new SQSConsumer(
    process.env.SQS_QUEUE_URL!,
    async (message) => {
      console.log('Processing:', message);
      await processOrder(message);
    }
  );

  consumer.start();

  // Graceful shutdown
  process.on('SIGTERM', () => consumer.stop());
}
```

### SQS with Dead Letter Queue

```typescript
// ════════════════════════════════════════════════════════════════
// SQS WITH DEAD LETTER QUEUE (CDK/CloudFormation)
// ════════════════════════════════════════════════════════════════

// Infrastructure as Code (CDK TypeScript)
import * as sqs from 'aws-cdk-lib/aws-sqs';
import { Duration } from 'aws-cdk-lib';

// Create DLQ
const dlq = new sqs.Queue(this, 'OrdersDLQ', {
  queueName: 'orders-dlq',
  retentionPeriod: Duration.days(14)
});

// Create main queue with DLQ
const ordersQueue = new sqs.Queue(this, 'OrdersQueue', {
  queueName: 'orders',
  visibilityTimeout: Duration.seconds(30),
  retentionPeriod: Duration.days(7),
  deadLetterQueue: {
    queue: dlq,
    maxReceiveCount: 3  // After 3 failures, send to DLQ
  }
});

// ════════════════════════════════════════════════════════════════
// PROCESSING DLQ MESSAGES
// ════════════════════════════════════════════════════════════════

class DLQProcessor {
  constructor(
    private dlqUrl: string,
    private mainQueueUrl: string
  ) {}

  async processDeadLetters(): Promise<void> {
    const dlq = new SQSQueue(this.dlqUrl);
    const mainQueue = new SQSQueue(this.mainQueueUrl);

    const messages = await dlq.receive({ maxMessages: 10 });

    for (const message of messages) {
      const body = JSON.parse(message.Body || '{}');
      const receiveCount = parseInt(
        message.Attributes?.ApproximateReceiveCount || '0'
      );

      console.log(`DLQ message (received ${receiveCount} times):`, body);

      // Decide what to do with message
      const action = await this.analyzeAndDecide(body);

      switch (action) {
        case 'retry':
          // Send back to main queue
          await mainQueue.send(body);
          await dlq.delete(message.ReceiptHandle!);
          break;

        case 'discard':
          // Just delete it
          await dlq.delete(message.ReceiptHandle!);
          break;

        case 'alert':
          // Alert and keep in DLQ
          await this.sendAlert(body);
          break;
      }
    }
  }

  private async analyzeAndDecide(message: any): Promise<'retry' | 'discard' | 'alert'> {
    // Implement your logic
    // e.g., check if it's a known error type that's now fixed
    return 'alert';
  }

  private async sendAlert(message: any): Promise<void> {
    // Send to Slack, PagerDuty, etc.
  }
}
```

---

## 3. BullMQ (Redis)

### Complete BullMQ Setup

```typescript
// ════════════════════════════════════════════════════════════════
// BULLMQ - PRODUCTION SETUP
// ════════════════════════════════════════════════════════════════

import { Queue, Worker, Job, QueueScheduler, QueueEvents } from 'bullmq';
import Redis from 'ioredis';

// Redis connection
const connection = new Redis({
  host: process.env.REDIS_HOST || 'localhost',
  port: parseInt(process.env.REDIS_PORT || '6379'),
  password: process.env.REDIS_PASSWORD,
  maxRetriesPerRequest: null  // Required for BullMQ
});

// ════════════════════════════════════════════════════════════════
// QUEUE SETUP
// ════════════════════════════════════════════════════════════════

interface EmailJobData {
  to: string;
  subject: string;
  body: string;
  attachments?: string[];
}

interface EmailJobResult {
  messageId: string;
  sentAt: Date;
}

const emailQueue = new Queue<EmailJobData, EmailJobResult>('emails', {
  connection,
  defaultJobOptions: {
    attempts: 5,
    backoff: {
      type: 'exponential',
      delay: 1000  // 1s, 2s, 4s, 8s, 16s
    },
    removeOnComplete: {
      count: 1000,  // Keep last 1000 completed
      age: 24 * 60 * 60  // Remove after 24 hours
    },
    removeOnFail: {
      count: 5000   // Keep more failures for debugging
    }
  }
});

// ════════════════════════════════════════════════════════════════
// WORKER
// ════════════════════════════════════════════════════════════════

const emailWorker = new Worker<EmailJobData, EmailJobResult>(
  'emails',
  async (job: Job<EmailJobData>) => {
    console.log(`Processing email job ${job.id} to ${job.data.to}`);

    // Update progress
    await job.updateProgress(10);

    // Simulate email sending
    const result = await sendEmail(job.data);

    await job.updateProgress(100);

    return {
      messageId: result.messageId,
      sentAt: new Date()
    };
  },
  {
    connection,
    concurrency: 10,  // Process 10 jobs in parallel
    limiter: {
      max: 100,       // Max 100 jobs
      duration: 60000 // Per minute (rate limiting)
    }
  }
);

// ════════════════════════════════════════════════════════════════
// EVENT HANDLERS
// ════════════════════════════════════════════════════════════════

emailWorker.on('completed', (job, result) => {
  console.log(`Job ${job.id} completed:`, result);
});

emailWorker.on('failed', (job, error) => {
  console.error(`Job ${job?.id} failed:`, error.message);
  
  if (job && job.attemptsMade >= (job.opts.attempts || 5)) {
    // Final failure - alert
    alertOnJobFailure(job, error);
  }
});

emailWorker.on('progress', (job, progress) => {
  console.log(`Job ${job.id} progress: ${progress}%`);
});

emailWorker.on('error', (error) => {
  console.error('Worker error:', error);
});

// ════════════════════════════════════════════════════════════════
// QUEUE EVENTS (Global)
// ════════════════════════════════════════════════════════════════

const queueEvents = new QueueEvents('emails', { connection });

queueEvents.on('waiting', ({ jobId }) => {
  console.log(`Job ${jobId} is waiting`);
});

queueEvents.on('active', ({ jobId }) => {
  console.log(`Job ${jobId} is active`);
});

queueEvents.on('completed', ({ jobId, returnvalue }) => {
  console.log(`Job ${jobId} completed with:`, returnvalue);
});

queueEvents.on('failed', ({ jobId, failedReason }) => {
  console.log(`Job ${jobId} failed:`, failedReason);
});

// ════════════════════════════════════════════════════════════════
// ADDING JOBS
// ════════════════════════════════════════════════════════════════

// Simple job
await emailQueue.add('welcome-email', {
  to: 'user@example.com',
  subject: 'Welcome!',
  body: 'Thanks for signing up'
});

// Job with options
await emailQueue.add('important-notification', 
  {
    to: 'admin@example.com',
    subject: 'Alert!',
    body: 'Something needs attention'
  },
  {
    priority: 1,  // Lower = higher priority
    delay: 5000,  // Wait 5 seconds
    attempts: 10, // Override default attempts
    jobId: 'unique-alert-123'  // Custom ID for deduplication
  }
);

// Bulk add
await emailQueue.addBulk([
  { name: 'email', data: { to: 'user1@example.com', subject: 'Hi', body: 'Hello' } },
  { name: 'email', data: { to: 'user2@example.com', subject: 'Hi', body: 'Hello' } },
  { name: 'email', data: { to: 'user3@example.com', subject: 'Hi', body: 'Hello' } }
]);

// ════════════════════════════════════════════════════════════════
// DELAYED AND SCHEDULED JOBS
// ════════════════════════════════════════════════════════════════

// Delayed job
await emailQueue.add('reminder', 
  { to: 'user@example.com', subject: 'Reminder', body: 'Don\'t forget!' },
  { delay: 60 * 60 * 1000 }  // 1 hour from now
);

// Repeating job (cron)
await emailQueue.add('daily-report',
  { to: 'team@example.com', subject: 'Daily Report', body: '' },
  {
    repeat: {
      pattern: '0 9 * * *',  // Every day at 9 AM
      tz: 'America/New_York'
    }
  }
);

// Repeating job (interval)
await emailQueue.add('health-check',
  { to: 'ops@example.com', subject: 'Health Check', body: '' },
  {
    repeat: {
      every: 5 * 60 * 1000  // Every 5 minutes
    }
  }
);

// ════════════════════════════════════════════════════════════════
// JOB MANAGEMENT
// ════════════════════════════════════════════════════════════════

// Get job by ID
const job = await emailQueue.getJob('job-id');

// Get job status
const state = await job?.getState();  // waiting, active, completed, failed, delayed

// Cancel a job
await job?.remove();

// Retry a failed job
await job?.retry();

// Get waiting jobs
const waiting = await emailQueue.getWaiting(0, 100);

// Get failed jobs
const failed = await emailQueue.getFailed(0, 100);

// Clean up old jobs
await emailQueue.clean(
  24 * 60 * 60 * 1000,  // Older than 24 hours
  100,                   // Limit
  'completed'            // Type: completed, failed, wait, delayed, active
);

// Pause/Resume
await emailQueue.pause();
await emailQueue.resume();

// ════════════════════════════════════════════════════════════════
// GRACEFUL SHUTDOWN
// ════════════════════════════════════════════════════════════════

async function shutdown() {
  console.log('Shutting down...');

  // Close worker - waits for current jobs
  await emailWorker.close();

  // Close queue
  await emailQueue.close();

  // Close events
  await queueEvents.close();

  // Close Redis
  await connection.quit();

  console.log('Shutdown complete');
  process.exit(0);
}

process.on('SIGTERM', shutdown);
process.on('SIGINT', shutdown);
```

### BullMQ Dashboard Setup

```typescript
// ════════════════════════════════════════════════════════════════
// BULL BOARD DASHBOARD
// ════════════════════════════════════════════════════════════════

import { createBullBoard } from '@bull-board/api';
import { BullMQAdapter } from '@bull-board/api/bullMQAdapter';
import { ExpressAdapter } from '@bull-board/express';
import express from 'express';

// Create Express adapter
const serverAdapter = new ExpressAdapter();
serverAdapter.setBasePath('/admin/queues');

// Create dashboard
createBullBoard({
  queues: [
    new BullMQAdapter(emailQueue),
    new BullMQAdapter(notificationQueue),
    new BullMQAdapter(processingQueue)
  ],
  serverAdapter
});

// Mount dashboard
const app = express();
app.use('/admin/queues', serverAdapter.getRouter());

// Add auth middleware for production
app.use('/admin/queues', basicAuth({
  users: { admin: process.env.ADMIN_PASSWORD || 'password' },
  challenge: true
}));

app.listen(3000, () => {
  console.log('Dashboard at http://localhost:3000/admin/queues');
});
```

---

## 4. Job Processing Patterns

### Priority Queue Pattern

```typescript
// ════════════════════════════════════════════════════════════════
// PRIORITY QUEUE PATTERN
// ════════════════════════════════════════════════════════════════

enum JobPriority {
  CRITICAL = 1,
  HIGH = 2,
  NORMAL = 5,
  LOW = 10
}

interface PrioritizedJob {
  type: string;
  data: any;
  priority: JobPriority;
}

class PriorityJobQueue {
  constructor(private queue: Queue) {}

  async addCritical(data: any): Promise<Job> {
    return this.queue.add('job', data, { priority: JobPriority.CRITICAL });
  }

  async addHigh(data: any): Promise<Job> {
    return this.queue.add('job', data, { priority: JobPriority.HIGH });
  }

  async addNormal(data: any): Promise<Job> {
    return this.queue.add('job', data, { priority: JobPriority.NORMAL });
  }

  async addLow(data: any): Promise<Job> {
    return this.queue.add('job', data, { priority: JobPriority.LOW });
  }
}

// Usage
const priorityQueue = new PriorityJobQueue(emailQueue);

// Critical: Password reset
await priorityQueue.addCritical({
  type: 'password-reset',
  email: 'user@example.com'
});

// Normal: Welcome email
await priorityQueue.addNormal({
  type: 'welcome',
  email: 'user@example.com'
});

// Low: Marketing newsletter
await priorityQueue.addLow({
  type: 'newsletter',
  email: 'user@example.com'
});
```

### Job Chaining Pattern

```typescript
// ════════════════════════════════════════════════════════════════
// JOB CHAINING / FLOW PATTERN
// ════════════════════════════════════════════════════════════════

import { FlowProducer, FlowJob } from 'bullmq';

const flowProducer = new FlowProducer({ connection });

// Define a workflow
async function createOrderWorkflow(orderId: string) {
  const flow = await flowProducer.add({
    name: 'notify-completion',
    queueName: 'notifications',
    data: { orderId, type: 'order-complete' },
    children: [
      {
        name: 'send-invoice',
        queueName: 'invoices',
        data: { orderId },
        children: [
          {
            name: 'charge-payment',
            queueName: 'payments',
            data: { orderId },
            children: [
              {
                name: 'validate-order',
                queueName: 'orders',
                data: { orderId }
              }
            ]
          }
        ]
      },
      {
        name: 'update-inventory',
        queueName: 'inventory',
        data: { orderId }
      }
    ]
  });

  return flow;
}

/*
Execution order:
1. validate-order (must complete first)
2. charge-payment (waits for validate-order)
3. send-invoice AND update-inventory (parallel, wait for charge-payment)
4. notify-completion (waits for both children)
*/

// Workers for each queue
const ordersWorker = new Worker('orders', async (job) => {
  console.log('Validating order:', job.data.orderId);
  return { validated: true };
});

const paymentsWorker = new Worker('payments', async (job) => {
  console.log('Charging payment:', job.data.orderId);
  return { charged: true, amount: 99.99 };
});

const invoicesWorker = new Worker('invoices', async (job) => {
  console.log('Sending invoice:', job.data.orderId);
  return { invoiceId: 'INV-123' };
});

const inventoryWorker = new Worker('inventory', async (job) => {
  console.log('Updating inventory:', job.data.orderId);
  return { updated: true };
});

const notificationsWorker = new Worker('notifications', async (job) => {
  console.log('Sending notification:', job.data.orderId);
  return { notified: true };
});
```

### Saga Pattern

```typescript
// ════════════════════════════════════════════════════════════════
// SAGA PATTERN WITH COMPENSATION
// ════════════════════════════════════════════════════════════════

interface SagaStep {
  name: string;
  execute: (context: any) => Promise<any>;
  compensate: (context: any) => Promise<void>;
}

class SagaOrchestrator {
  private steps: SagaStep[] = [];
  private completedSteps: string[] = [];

  addStep(step: SagaStep): this {
    this.steps.push(step);
    return this;
  }

  async execute(initialContext: any): Promise<any> {
    let context = { ...initialContext };

    try {
      for (const step of this.steps) {
        console.log(`Executing step: ${step.name}`);
        const result = await step.execute(context);
        context = { ...context, [step.name]: result };
        this.completedSteps.push(step.name);
      }

      return context;

    } catch (error) {
      console.error('Saga failed, compensating...');
      await this.compensate(context);
      throw error;
    }
  }

  private async compensate(context: any): Promise<void> {
    // Execute compensations in reverse order
    for (const stepName of this.completedSteps.reverse()) {
      const step = this.steps.find(s => s.name === stepName);
      if (step) {
        console.log(`Compensating step: ${step.name}`);
        try {
          await step.compensate(context);
        } catch (error) {
          console.error(`Compensation failed for ${step.name}:`, error);
          // Log for manual intervention
        }
      }
    }
  }
}

// Usage
const orderSaga = new SagaOrchestrator()
  .addStep({
    name: 'reserveInventory',
    execute: async (ctx) => {
      const reservation = await inventoryService.reserve(ctx.items);
      return reservation;
    },
    compensate: async (ctx) => {
      await inventoryService.cancelReservation(ctx.reserveInventory.id);
    }
  })
  .addStep({
    name: 'chargePayment',
    execute: async (ctx) => {
      const charge = await paymentService.charge(ctx.userId, ctx.amount);
      return charge;
    },
    compensate: async (ctx) => {
      await paymentService.refund(ctx.chargePayment.transactionId);
    }
  })
  .addStep({
    name: 'createShipment',
    execute: async (ctx) => {
      const shipment = await shippingService.create(ctx.address, ctx.items);
      return shipment;
    },
    compensate: async (ctx) => {
      await shippingService.cancel(ctx.createShipment.id);
    }
  });

// Execute saga
try {
  const result = await orderSaga.execute({
    userId: '123',
    items: [{ sku: 'ABC', qty: 2 }],
    amount: 99.99,
    address: '123 Main St'
  });
  console.log('Order completed:', result);
} catch (error) {
  console.error('Order failed, compensations executed');
}
```

---

## 5. Retry Strategies

```typescript
// ════════════════════════════════════════════════════════════════
// RETRY STRATEGIES
// ════════════════════════════════════════════════════════════════

// 1. EXPONENTIAL BACKOFF
const exponentialBackoff = {
  attempts: 5,
  backoff: {
    type: 'exponential',
    delay: 1000  // 1s, 2s, 4s, 8s, 16s
  }
};

// 2. FIXED DELAY
const fixedDelay = {
  attempts: 5,
  backoff: {
    type: 'fixed',
    delay: 5000  // Always 5 seconds
  }
};

// 3. CUSTOM BACKOFF
const customBackoff = {
  attempts: 5,
  backoff: {
    type: 'custom'
  }
};

// Custom backoff function
const worker = new Worker('queue', processor, {
  settings: {
    backoffStrategy: (attemptsMade: number) => {
      // Custom delays: 1s, 5s, 30s, 5min, 30min
      const delays = [1000, 5000, 30000, 300000, 1800000];
      return delays[Math.min(attemptsMade - 1, delays.length - 1)];
    }
  }
});

// 4. RETRY WITH JITTER
function getRetryDelayWithJitter(attemptsMade: number): number {
  const baseDelay = 1000;
  const maxDelay = 60000;
  
  // Exponential backoff
  const exponentialDelay = baseDelay * Math.pow(2, attemptsMade - 1);
  
  // Add jitter (±25%)
  const jitter = exponentialDelay * 0.25 * (Math.random() * 2 - 1);
  
  return Math.min(exponentialDelay + jitter, maxDelay);
}

// ════════════════════════════════════════════════════════════════
// CONDITIONAL RETRY
// ════════════════════════════════════════════════════════════════

class RetryableError extends Error {
  constructor(message: string, public shouldRetry: boolean = true) {
    super(message);
    this.name = 'RetryableError';
  }
}

const smartWorker = new Worker('queue', async (job) => {
  try {
    await processJob(job.data);
  } catch (error) {
    // Don't retry certain errors
    if (error instanceof ValidationError) {
      throw new RetryableError(error.message, false);
    }
    if (error instanceof RateLimitError) {
      // Wait longer for rate limits
      throw new RetryableError(error.message, true);
    }
    throw error;
  }
}, {
  settings: {
    backoffStrategy: (attemptsMade, type, err) => {
      if (err instanceof RetryableError && !err.shouldRetry) {
        return -1;  // Don't retry
      }
      if (err instanceof RateLimitError) {
        return 60000;  // Wait 1 minute for rate limits
      }
      return 1000 * Math.pow(2, attemptsMade);
    }
  }
});

// ════════════════════════════════════════════════════════════════
// RETRY BUDGET
// ════════════════════════════════════════════════════════════════

class RetryBudget {
  private attempts = 0;
  private failures = 0;
  private windowStart = Date.now();
  private readonly windowMs = 60000;  // 1 minute window
  private readonly maxFailureRate = 0.5;  // 50%
  private readonly minAttempts = 10;

  shouldRetry(): boolean {
    this.resetWindowIfNeeded();
    
    // Always retry if we haven't hit minimum attempts
    if (this.attempts < this.minAttempts) {
      return true;
    }
    
    // Check failure rate
    const failureRate = this.failures / this.attempts;
    return failureRate < this.maxFailureRate;
  }

  recordSuccess(): void {
    this.attempts++;
  }

  recordFailure(): void {
    this.attempts++;
    this.failures++;
  }

  private resetWindowIfNeeded(): void {
    if (Date.now() - this.windowStart > this.windowMs) {
      this.attempts = 0;
      this.failures = 0;
      this.windowStart = Date.now();
    }
  }
}
```

---

## 6. Dead Letter Queues

```typescript
// ════════════════════════════════════════════════════════════════
// DEAD LETTER QUEUE IMPLEMENTATION
// ════════════════════════════════════════════════════════════════

// BullMQ doesn't have built-in DLQ, implement manually

class DeadLetterQueue {
  private dlq: Queue;
  private mainQueue: Queue;

  constructor(
    private queueName: string,
    private connection: Redis
  ) {
    this.mainQueue = new Queue(queueName, { connection });
    this.dlq = new Queue(`${queueName}-dlq`, { connection });
  }

  async moveToDeadLetter(job: Job, reason: string): Promise<void> {
    await this.dlq.add('dead-letter', {
      originalJob: {
        id: job.id,
        name: job.name,
        data: job.data,
        attemptsMade: job.attemptsMade,
        failedReason: job.failedReason
      },
      movedAt: new Date().toISOString(),
      reason
    }, {
      removeOnComplete: false  // Keep for investigation
    });

    await job.remove();
  }

  async reprocessDeadLetters(filter?: (job: any) => boolean): Promise<number> {
    const deadLetters = await this.dlq.getJobs(['waiting', 'delayed']);
    let reprocessed = 0;

    for (const dlJob of deadLetters) {
      if (!filter || filter(dlJob.data.originalJob)) {
        // Add back to main queue
        await this.mainQueue.add(
          dlJob.data.originalJob.name,
          dlJob.data.originalJob.data,
          {
            jobId: `retry-${dlJob.data.originalJob.id}`
          }
        );

        await dlJob.remove();
        reprocessed++;
      }
    }

    return reprocessed;
  }

  async getDeadLetterStats(): Promise<{
    total: number;
    byReason: Record<string, number>;
    oldest: Date | null;
  }> {
    const deadLetters = await this.dlq.getJobs(['waiting', 'delayed', 'completed', 'failed']);
    
    const byReason: Record<string, number> = {};
    let oldest: Date | null = null;

    for (const job of deadLetters) {
      const reason = job.data.reason || 'unknown';
      byReason[reason] = (byReason[reason] || 0) + 1;

      const movedAt = new Date(job.data.movedAt);
      if (!oldest || movedAt < oldest) {
        oldest = movedAt;
      }
    }

    return {
      total: deadLetters.length,
      byReason,
      oldest
    };
  }

  async purgeOldDeadLetters(maxAgeMs: number): Promise<number> {
    const cutoff = Date.now() - maxAgeMs;
    const deadLetters = await this.dlq.getJobs(['waiting', 'delayed']);
    let purged = 0;

    for (const job of deadLetters) {
      const movedAt = new Date(job.data.movedAt).getTime();
      if (movedAt < cutoff) {
        await job.remove();
        purged++;
      }
    }

    return purged;
  }
}

// ════════════════════════════════════════════════════════════════
// WORKER WITH DLQ INTEGRATION
// ════════════════════════════════════════════════════════════════

function createWorkerWithDLQ(
  queueName: string,
  processor: (job: Job) => Promise<any>,
  connection: Redis
) {
  const dlq = new DeadLetterQueue(queueName, connection);

  const worker = new Worker(queueName, processor, {
    connection,
    settings: {
      backoffStrategy: (attemptsMade) => {
        return Math.min(1000 * Math.pow(2, attemptsMade), 60000);
      }
    }
  });

  worker.on('failed', async (job, error) => {
    if (!job) return;

    const maxAttempts = job.opts.attempts || 5;
    
    if (job.attemptsMade >= maxAttempts) {
      console.log(`Moving job ${job.id} to DLQ after ${job.attemptsMade} attempts`);
      await dlq.moveToDeadLetter(job, error.message);
      
      // Alert
      await alertOnDeadLetter(job, error);
    }
  });

  return { worker, dlq };
}

// ════════════════════════════════════════════════════════════════
// DLQ MONITORING
// ════════════════════════════════════════════════════════════════

async function monitorDeadLetterQueues(queues: DeadLetterQueue[]) {
  setInterval(async () => {
    for (const dlq of queues) {
      const stats = await dlq.getDeadLetterStats();
      
      // Log metrics
      console.log('DLQ Stats:', stats);

      // Alert if DLQ is growing
      if (stats.total > 100) {
        await alertDLQThreshold(stats);
      }

      // Purge old messages (older than 7 days)
      const purged = await dlq.purgeOldDeadLetters(7 * 24 * 60 * 60 * 1000);
      if (purged > 0) {
        console.log(`Purged ${purged} old dead letters`);
      }
    }
  }, 60000);  // Check every minute
}
```

---

## 7. Scaling Consumers

```typescript
// ════════════════════════════════════════════════════════════════
// SCALING CONSUMERS
// ════════════════════════════════════════════════════════════════

// 1. CONCURRENCY WITHIN WORKER
const worker = new Worker('queue', processor, {
  connection,
  concurrency: 10  // Process 10 jobs simultaneously
});

// 2. MULTIPLE WORKER INSTANCES
// Run multiple processes with PM2 or Kubernetes

// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'queue-worker',
    script: './worker.js',
    instances: 4,         // 4 worker processes
    exec_mode: 'cluster'
  }]
};

// 3. AUTO-SCALING BASED ON QUEUE DEPTH
class AutoScaler {
  private currentWorkers = 1;
  private readonly minWorkers = 1;
  private readonly maxWorkers = 10;
  private readonly scaleUpThreshold = 1000;  // Jobs in queue
  private readonly scaleDownThreshold = 100;

  constructor(private queue: Queue) {
    this.startMonitoring();
  }

  private async startMonitoring() {
    setInterval(async () => {
      const waiting = await this.queue.getWaitingCount();
      const delayed = await this.queue.getDelayedCount();
      const total = waiting + delayed;

      console.log(`Queue depth: ${total}, Workers: ${this.currentWorkers}`);

      if (total > this.scaleUpThreshold && this.currentWorkers < this.maxWorkers) {
        await this.scaleUp();
      } else if (total < this.scaleDownThreshold && this.currentWorkers > this.minWorkers) {
        await this.scaleDown();
      }
    }, 10000);  // Check every 10 seconds
  }

  private async scaleUp(): Promise<void> {
    this.currentWorkers++;
    console.log(`Scaling up to ${this.currentWorkers} workers`);
    // In practice: spawn new process, Kubernetes pod, or ECS task
    await this.updateDesiredCount(this.currentWorkers);
  }

  private async scaleDown(): Promise<void> {
    this.currentWorkers--;
    console.log(`Scaling down to ${this.currentWorkers} workers`);
    await this.updateDesiredCount(this.currentWorkers);
  }

  private async updateDesiredCount(count: number): Promise<void> {
    // Implement based on your infrastructure
    // e.g., Kubernetes HPA, AWS ECS desired count, PM2 scale
  }
}

// ════════════════════════════════════════════════════════════════
// KUBERNETES HPA CONFIG
// ════════════════════════════════════════════════════════════════

/*
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: queue-worker-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: queue-worker
  minReplicas: 1
  maxReplicas: 10
  metrics:
    - type: External
      external:
        metric:
          name: queue_depth
          selector:
            matchLabels:
              queue: emails
        target:
          type: AverageValue
          averageValue: 100  # Scale up when >100 messages per pod
*/

// ════════════════════════════════════════════════════════════════
// RATE LIMITING
// ════════════════════════════════════════════════════════════════

// Worker-level rate limiting
const rateLimitedWorker = new Worker('api-calls', processor, {
  connection,
  limiter: {
    max: 100,        // Max 100 jobs
    duration: 60000  // Per minute
  }
});

// Job-level rate limiting (per key)
const queue = new Queue('api-calls', {
  connection,
  limiter: {
    max: 10,
    duration: 1000,
    groupKey: 'apiKey'  // Rate limit per API key
  }
});

// Add jobs with group key
await queue.add('call', { apiKey: 'key-1', endpoint: '/users' });
await queue.add('call', { apiKey: 'key-2', endpoint: '/orders' });
// Each API key has its own rate limit
```

---

## 8. Monitoring & Alerting

```typescript
// ════════════════════════════════════════════════════════════════
// QUEUE MONITORING
// ════════════════════════════════════════════════════════════════

import { Gauge, Counter, Histogram, Registry } from 'prom-client';

const register = new Registry();

// Metrics
const queueDepth = new Gauge({
  name: 'queue_depth',
  help: 'Number of jobs in queue',
  labelNames: ['queue', 'state'],
  registers: [register]
});

const jobsProcessed = new Counter({
  name: 'jobs_processed_total',
  help: 'Total jobs processed',
  labelNames: ['queue', 'status'],
  registers: [register]
});

const jobDuration = new Histogram({
  name: 'job_duration_seconds',
  help: 'Job processing duration',
  labelNames: ['queue'],
  buckets: [0.1, 0.5, 1, 5, 10, 30, 60],
  registers: [register]
});

// Metrics collector
class QueueMetricsCollector {
  constructor(private queues: Map<string, Queue>) {
    this.startCollecting();
  }

  private async startCollecting() {
    setInterval(async () => {
      for (const [name, queue] of this.queues) {
        const counts = await queue.getJobCounts();
        
        queueDepth.set({ queue: name, state: 'waiting' }, counts.waiting);
        queueDepth.set({ queue: name, state: 'active' }, counts.active);
        queueDepth.set({ queue: name, state: 'delayed' }, counts.delayed);
        queueDepth.set({ queue: name, state: 'failed' }, counts.failed);
      }
    }, 5000);
  }

  trackJobComplete(queue: string, duration: number) {
    jobsProcessed.inc({ queue, status: 'completed' });
    jobDuration.observe({ queue }, duration);
  }

  trackJobFailed(queue: string) {
    jobsProcessed.inc({ queue, status: 'failed' });
  }
}

// Worker with metrics
function createMetricedWorker(
  queueName: string,
  processor: (job: Job) => Promise<any>,
  metrics: QueueMetricsCollector
) {
  const worker = new Worker(queueName, async (job) => {
    const start = Date.now();
    try {
      const result = await processor(job);
      metrics.trackJobComplete(queueName, (Date.now() - start) / 1000);
      return result;
    } catch (error) {
      metrics.trackJobFailed(queueName);
      throw error;
    }
  });

  return worker;
}

// ════════════════════════════════════════════════════════════════
// ALERTING
// ════════════════════════════════════════════════════════════════

interface AlertConfig {
  queueDepthThreshold: number;
  dlqThreshold: number;
  processingTimeThreshold: number;  // seconds
  errorRateThreshold: number;       // percentage
}

class QueueAlerter {
  private lastAlertTime: Map<string, number> = new Map();
  private alertCooldown = 5 * 60 * 1000;  // 5 minutes

  constructor(
    private queues: Map<string, Queue>,
    private config: AlertConfig
  ) {
    this.startMonitoring();
  }

  private async startMonitoring() {
    setInterval(async () => {
      for (const [name, queue] of this.queues) {
        await this.checkQueue(name, queue);
      }
    }, 30000);  // Check every 30 seconds
  }

  private async checkQueue(name: string, queue: Queue) {
    const counts = await queue.getJobCounts();

    // Check queue depth
    if (counts.waiting > this.config.queueDepthThreshold) {
      await this.alert(`Queue ${name} depth high: ${counts.waiting}`, name);
    }

    // Check failed jobs (potential DLQ)
    if (counts.failed > this.config.dlqThreshold) {
      await this.alert(`Queue ${name} has ${counts.failed} failed jobs`, name);
    }
  }

  private async alert(message: string, key: string): Promise<void> {
    const lastAlert = this.lastAlertTime.get(key) || 0;
    if (Date.now() - lastAlert < this.alertCooldown) {
      return;  // Still in cooldown
    }

    this.lastAlertTime.set(key, Date.now());

    // Send alert
    console.error('ALERT:', message);
    await this.sendToSlack(message);
    await this.sendToPagerDuty(message);
  }

  private async sendToSlack(message: string): Promise<void> {
    // Implement Slack webhook
  }

  private async sendToPagerDuty(message: string): Promise<void> {
    // Implement PagerDuty integration
  }
}

// ════════════════════════════════════════════════════════════════
// METRICS ENDPOINT
// ════════════════════════════════════════════════════════════════

app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.send(await register.metrics());
});

// Health check
app.get('/health', async (req, res) => {
  const health = {
    status: 'healthy',
    queues: {}
  };

  for (const [name, queue] of queues) {
    const counts = await queue.getJobCounts();
    health.queues[name] = {
      waiting: counts.waiting,
      active: counts.active,
      failed: counts.failed
    };

    if (counts.waiting > 10000 || counts.failed > 100) {
      health.status = 'degraded';
    }
  }

  res.status(health.status === 'healthy' ? 200 : 503).json(health);
});
```

---

## 9. Common Pitfalls

```typescript
// ════════════════════════════════════════════════════════════════
// MESSAGE QUEUE PITFALLS
// ════════════════════════════════════════════════════════════════

// ❌ PITFALL 1: Not making consumers idempotent
// Problem: Duplicate processing on retry

// Bad
async function processOrder(order) {
  await chargeCustomer(order.customerId, order.amount);
  await sendEmail(order.email);
  // If email fails, customer charged twice on retry!
}

// Good - Use idempotency keys
async function processOrder(order) {
  const idempotencyKey = `order-${order.id}`;
  
  // Check if already processed
  const processed = await redis.get(`processed:${idempotencyKey}`);
  if (processed) {
    console.log('Already processed, skipping');
    return;
  }
  
  await chargeCustomer(order.customerId, order.amount, idempotencyKey);
  await sendEmail(order.email);
  
  // Mark as processed
  await redis.set(`processed:${idempotencyKey}`, '1', 'EX', 86400);
}

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 2: Not acknowledging messages properly
// Problem: Messages lost or reprocessed indefinitely

// Bad
channel.consume('queue', async (msg) => {
  processMessage(msg);
  // No ack! Message stays invisible forever or redelivered
});

// Good
channel.consume('queue', async (msg) => {
  try {
    await processMessage(msg);
    channel.ack(msg);  // Success - remove from queue
  } catch (error) {
    channel.nack(msg, false, true);  // Failure - requeue
  }
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 3: Setting visibility timeout too short
// Problem: Message processed twice simultaneously

// Bad
await sqs.receive({ VisibilityTimeout: 5 });  // Only 5 seconds!
// If processing takes 10 seconds, message becomes visible again

// Good
const processingTimeEstimate = 30;  // seconds
const buffer = 30;  // extra buffer
await sqs.receive({ VisibilityTimeout: processingTimeEstimate + buffer });

// Or extend visibility during processing
async function processWithExtension(message) {
  const interval = setInterval(async () => {
    await sqs.changeMessageVisibility({
      ReceiptHandle: message.ReceiptHandle,
      VisibilityTimeout: 30
    });
  }, 15000);  // Extend every 15 seconds
  
  try {
    await longRunningProcess(message);
  } finally {
    clearInterval(interval);
  }
}

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 4: No dead letter queue
// Problem: Poison messages block the queue

// Bad - Retry forever
const worker = new Worker('queue', processor, {
  attempts: Infinity  // Never stops retrying
});

// Good - Limit retries, use DLQ
const worker = new Worker('queue', processor, {
  attempts: 5,
  backoff: { type: 'exponential', delay: 1000 }
});

worker.on('failed', async (job, error) => {
  if (job.attemptsMade >= 5) {
    await moveToDeadLetterQueue(job, error);
  }
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 5: Not handling connection failures
// Problem: Application crashes or hangs

// Bad
const channel = await connection.createChannel();
// No error handling, connection drops = crash

// Good
connection.on('error', (err) => {
  console.error('RabbitMQ connection error:', err);
  reconnect();
});

connection.on('close', () => {
  console.log('RabbitMQ connection closed, reconnecting...');
  reconnect();
});

async function reconnect() {
  let attempts = 0;
  while (attempts < 10) {
    try {
      await connect();
      return;
    } catch {
      attempts++;
      await sleep(Math.min(1000 * Math.pow(2, attempts), 30000));
    }
  }
  process.exit(1);  // Give up
}

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 6: Messages too large
// Problem: Performance issues, SQS rejection

// Bad
await queue.send({
  data: hugeObject,  // 5MB of data
  image: base64EncodedImage
});

// Good - Store large data externally
const s3Key = await uploadToS3(hugeObject);
await queue.send({
  dataRef: s3Key,  // Just the reference
  type: 'large-data'
});

// Consumer
const dataRef = message.dataRef;
const data = await downloadFromS3(dataRef);

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 7: No backpressure handling
// Problem: Consumer overwhelmed, crashes

// Bad
const worker = new Worker('queue', processor, {
  concurrency: 100  // Too many!
});

// Good - Start conservative, monitor, adjust
const worker = new Worker('queue', processor, {
  concurrency: 10,
  limiter: {
    max: 100,
    duration: 60000  // Rate limit
  }
});

// Monitor and adjust
setInterval(async () => {
  const memory = process.memoryUsage();
  if (memory.heapUsed > 500 * 1024 * 1024) {
    await worker.pause();
    // Force GC if available
    global.gc?.();
    await worker.resume();
  }
}, 10000);

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 8: Not monitoring queue depth
// Problem: Backlog grows unnoticed

// Good - Monitor and alert
setInterval(async () => {
  const depth = await queue.getWaitingCount();
  
  if (depth > 10000) {
    alert('Queue depth critical', depth);
  } else if (depth > 1000) {
    warn('Queue depth high', depth);
  }
  
  metrics.gauge('queue_depth', depth);
}, 30000);
```

---

## 10. Interview Questions

### Basic Questions

**Q: "What is a message queue?"**
> "A message queue is middleware that enables asynchronous communication between services. Producers send messages, the queue stores them, consumers pull and process them. This provides decoupling (producer doesn't know about consumers), load leveling (absorb spikes), and reliability (messages persist until processed)."

**Q: "What is at-least-once delivery?"**
> "At-least-once means a message is guaranteed to be delivered at least once, but possibly more times if there's a failure before acknowledgment. This is the default for most queues. It requires consumers to be idempotent - processing the same message twice should have the same effect as once."

**Q: "What is a dead letter queue?"**
> "A DLQ stores messages that failed processing after maximum retries. Instead of losing them or blocking the queue, they're moved to a separate queue for investigation. You can analyze, fix issues, and reprocess them. Essential for debugging and maintaining system reliability."

### Intermediate Questions

**Q: "How would you choose between RabbitMQ, SQS, and BullMQ?"**
> "**RabbitMQ**: Need complex routing (topic exchanges), message priorities, on-premise, or AMQP compliance.

**SQS**: AWS-native, want managed service with zero ops, need infinite scale, serverless architecture, FIFO ordering.

**BullMQ**: Already using Redis, need job scheduling/delays, want rich dashboard, Node.js job processing.

For most Node.js apps doing background jobs, I'd start with BullMQ. For complex enterprise messaging, RabbitMQ. For AWS serverless, SQS."

**Q: "How do you handle poison messages?"**
> "Poison messages are those that repeatedly fail processing. Strategy:
1. Limit retry attempts (e.g., 5)
2. After max retries, move to DLQ
3. Alert on DLQ messages
4. Investigate: is it data issue, bug, or external dependency?
5. Fix the issue
6. Reprocess from DLQ or discard if invalid

The key is not letting poison messages block the main queue while still capturing them for investigation."

**Q: "How do you scale message consumers?"**
> "Competing consumers pattern - multiple workers pull from the same queue. Scaling approaches:
1. **Concurrency**: Process multiple messages per worker (e.g., concurrency: 10)
2. **Horizontal**: Run multiple worker processes (PM2 cluster, Kubernetes replicas)
3. **Auto-scaling**: Scale based on queue depth metrics (Kubernetes HPA, AWS auto-scaling)

Monitor queue depth and processing lag. Scale up when backlog grows, scale down when caught up."

### Advanced Questions

**Q: "How would you implement exactly-once processing?"**
> "True exactly-once is difficult. Practical approaches:

1. **Idempotent consumers**: Use idempotency keys stored in database. Check before processing, mark after processing.

2. **Transactional outbox**: Write to database and queue atomically. Consumer uses database transaction with idempotency check.

3. **Deduplication window**: Track message IDs in Redis with TTL. Reject duplicates within window.

4. **SQS FIFO**: Has built-in deduplication with MessageDeduplicationId.

The key insight is that exactly-once is really 'at-least-once delivery + idempotent consumer'."

**Q: "How would you design a job processing system for sending millions of emails?"**
> "Architecture:
1. **Ingestion**: API accepts email requests, validates, adds to queue with deduplication ID
2. **Queue**: BullMQ with priorities (transactional > marketing), rate limiting per domain
3. **Workers**: Multiple workers with concurrency 10 each, respecting ESP rate limits
4. **Retry**: Exponential backoff, separate handling for bounce vs rate limit
5. **DLQ**: Failed after 5 attempts, daily review job
6. **Monitoring**: Queue depth, send rate, bounce rate, latency percentiles

Key considerations: rate limits per ESP, domain reputation, bounce handling, unsubscribe processing."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│              MESSAGE QUEUE IMPLEMENTATION CHECKLIST             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  PRODUCER:                                                      │
│  □ Set message persistence/durability                          │
│  □ Include idempotency key if needed                           │
│  □ Handle send failures (retry, circuit breaker)               │
│  □ Keep messages small (reference large data)                  │
│                                                                 │
│  CONSUMER:                                                      │
│  □ Make processing idempotent                                  │
│  □ Acknowledge only after successful processing                │
│  □ Set appropriate visibility/lock timeout                     │
│  □ Handle errors gracefully (retry vs reject)                  │
│  □ Implement graceful shutdown                                 │
│                                                                 │
│  RELIABILITY:                                                   │
│  □ Configure retry with exponential backoff                    │
│  □ Set up dead letter queue                                    │
│  □ Alert on DLQ messages                                       │
│  □ Have DLQ reprocessing capability                            │
│                                                                 │
│  SCALING:                                                       │
│  □ Set appropriate concurrency                                 │
│  □ Implement rate limiting if needed                           │
│  □ Monitor queue depth                                         │
│  □ Auto-scale based on metrics                                 │
│                                                                 │
│  MONITORING:                                                    │
│  □ Track queue depth (waiting, active, failed)                 │
│  □ Track processing rate and latency                           │
│  □ Alert on backlog growth                                     │
│  □ Dashboard for visibility                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

COMMON RETRY DELAYS:
┌────────────────────────────────────────┐
│ Attempt 1: 1 second                    │
│ Attempt 2: 5 seconds                   │
│ Attempt 3: 30 seconds                  │
│ Attempt 4: 5 minutes                   │
│ Attempt 5: 30 minutes → DLQ            │
└────────────────────────────────────────┘

QUEUE SELECTION GUIDE:
┌────────────────────────────────────────────────────────────┐
│ Need         │ RabbitMQ │ SQS │ BullMQ │ Kafka           │
├──────────────┼──────────┼─────┼────────┼─────────────────┤
│ Job queue    │ ✓        │ ✓   │ ✓✓     │                 │
│ Pub/Sub      │ ✓✓       │     │        │ ✓✓              │
│ Event stream │          │     │        │ ✓✓              │
│ Managed      │          │ ✓✓  │        │ (Confluent)     │
│ Complex route│ ✓✓       │     │        │                 │
│ Dashboard    │ ✓        │     │ ✓✓     │ ✓               │
└────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

