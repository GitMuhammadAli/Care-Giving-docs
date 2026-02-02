# Serverless Patterns - Complete Guide

> **MUST REMEMBER**: Serverless = functions as a service (Lambda, Cloud Functions). Key patterns: Function composition (Step Functions, chaining), Event-driven (S3, SQS triggers), Fan-out/Fan-in (parallel processing). Cold starts are the main latency concern - mitigate with provisioned concurrency or keep-warm. Stateless by design - use external storage. Cost model: pay per invocation, not idle time.

---

## How to Explain Like a Senior Developer

"Serverless isn't 'no servers' - it's 'not your servers'. You write functions, the cloud handles scaling, infrastructure, and you pay only for execution time. The challenge is thinking in events and stateless functions. Cold starts happen when a new container spins up - critical for user-facing APIs, less so for background jobs. Patterns emerge: use Step Functions for complex workflows, SQS to decouple and smooth traffic, DynamoDB for fast state storage. The cost model is great for variable workloads but can be expensive at constant high scale. Design for idempotency since functions can be retried, and always set sensible timeouts."

---

## Core Implementation

### Function Composition with Step Functions

```typescript
// serverless/step-functions.ts

/**
 * AWS Step Functions State Machine Definition
 * Orchestrates multiple Lambda functions
 */

const orderProcessingStateMachine = {
  Comment: 'Order processing workflow',
  StartAt: 'ValidateOrder',
  States: {
    ValidateOrder: {
      Type: 'Task',
      Resource: 'arn:aws:lambda:us-east-1:123456789:function:validateOrder',
      Next: 'CheckInventory',
      Catch: [{
        ErrorEquals: ['ValidationError'],
        Next: 'OrderFailed',
      }],
      Retry: [{
        ErrorEquals: ['Lambda.ServiceException'],
        IntervalSeconds: 2,
        MaxAttempts: 3,
        BackoffRate: 2,
      }],
    },
    
    CheckInventory: {
      Type: 'Task',
      Resource: 'arn:aws:lambda:us-east-1:123456789:function:checkInventory',
      Next: 'ProcessPayment',
      Catch: [{
        ErrorEquals: ['OutOfStockError'],
        Next: 'NotifyOutOfStock',
      }],
    },
    
    ProcessPayment: {
      Type: 'Task',
      Resource: 'arn:aws:lambda:us-east-1:123456789:function:processPayment',
      Next: 'ParallelProcessing',
      Catch: [{
        ErrorEquals: ['PaymentDeclinedError'],
        Next: 'OrderFailed',
      }],
    },
    
    // Parallel execution
    ParallelProcessing: {
      Type: 'Parallel',
      Branches: [
        {
          StartAt: 'SendConfirmationEmail',
          States: {
            SendConfirmationEmail: {
              Type: 'Task',
              Resource: 'arn:aws:lambda:us-east-1:123456789:function:sendEmail',
              End: true,
            },
          },
        },
        {
          StartAt: 'UpdateAnalytics',
          States: {
            UpdateAnalytics: {
              Type: 'Task',
              Resource: 'arn:aws:lambda:us-east-1:123456789:function:updateAnalytics',
              End: true,
            },
          },
        },
        {
          StartAt: 'CreateShipment',
          States: {
            CreateShipment: {
              Type: 'Task',
              Resource: 'arn:aws:lambda:us-east-1:123456789:function:createShipment',
              End: true,
            },
          },
        },
      ],
      Next: 'OrderComplete',
    },
    
    OrderComplete: {
      Type: 'Succeed',
    },
    
    OrderFailed: {
      Type: 'Fail',
      Error: 'OrderProcessingFailed',
      Cause: 'Order could not be processed',
    },
    
    NotifyOutOfStock: {
      Type: 'Task',
      Resource: 'arn:aws:lambda:us-east-1:123456789:function:notifyOutOfStock',
      Next: 'OrderFailed',
    },
  },
};

// Lambda handler for Step Functions
import { Context } from 'aws-lambda';

interface OrderInput {
  orderId: string;
  items: Array<{ productId: string; quantity: number }>;
  userId: string;
  paymentMethod: string;
}

export async function validateOrder(
  event: OrderInput,
  context: Context
): Promise<OrderInput & { validated: boolean }> {
  // Validation logic
  if (!event.orderId || !event.items?.length) {
    throw new ValidationError('Invalid order data');
  }
  
  return {
    ...event,
    validated: true,
  };
}

export async function checkInventory(
  event: OrderInput & { validated: boolean },
  context: Context
): Promise<OrderInput & { inventoryReserved: boolean }> {
  // Check and reserve inventory
  for (const item of event.items) {
    const available = await getInventory(item.productId);
    if (available < item.quantity) {
      throw new OutOfStockError(`Product ${item.productId} out of stock`);
    }
  }
  
  // Reserve inventory
  await reserveInventory(event.orderId, event.items);
  
  return {
    ...event,
    inventoryReserved: true,
  };
}

class ValidationError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'ValidationError';
  }
}

class OutOfStockError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'OutOfStockError';
  }
}

async function getInventory(productId: string): Promise<number> { return 100; }
async function reserveInventory(orderId: string, items: any[]): Promise<void> {}
```

### Cold Start Mitigation

```typescript
// serverless/cold-start.ts
import { APIGatewayProxyHandler } from 'aws-lambda';

/**
 * Cold Start Mitigation Strategies:
 * 
 * 1. Provisioned Concurrency (guaranteed warm instances)
 * 2. Keep-warm with CloudWatch Events
 * 3. Optimize bundle size
 * 4. Lazy initialization
 * 5. Use lighter runtimes (Node.js > Java)
 */

// Strategy 1: Move initialization outside handler
let dbConnection: any = null;
let configLoaded = false;
let config: any = {};

async function initializeOnce(): Promise<void> {
  if (!configLoaded) {
    // Load config from Parameter Store
    config = await loadConfig();
    configLoaded = true;
  }
  
  if (!dbConnection) {
    // Reuse connection across invocations
    dbConnection = await createDbConnection();
  }
}

export const handler: APIGatewayProxyHandler = async (event, context) => {
  // Don't wait for event loop to be empty (faster response)
  context.callbackWaitsForEmptyEventLoop = false;
  
  await initializeOnce();
  
  // Handle request...
  return {
    statusCode: 200,
    body: JSON.stringify({ message: 'OK' }),
  };
};

// Strategy 2: Keep-warm handler
export const keepWarm: APIGatewayProxyHandler = async (event) => {
  // Check if this is a keep-warm invocation
  if (event.source === 'serverless-plugin-warmup') {
    console.log('Keep-warm invocation');
    return {
      statusCode: 200,
      body: 'Warmed!',
    };
  }
  
  // Normal processing...
  return {
    statusCode: 200,
    body: JSON.stringify({ message: 'OK' }),
  };
};

// serverless.yml for keep-warm
const warmupConfig = `
plugins:
  - serverless-plugin-warmup

custom:
  warmup:
    default:
      enabled: true
      events:
        - schedule: rate(5 minutes)
      concurrency: 5
      prewarm: true

functions:
  api:
    handler: src/api.handler
    warmup:
      default:
        enabled: true
`;

// Strategy 3: Provisioned Concurrency config
const provisionedConfig = `
functions:
  api:
    handler: src/api.handler
    provisionedConcurrency: 5  # Always 5 warm instances
    events:
      - http:
          path: /
          method: any
`;

async function loadConfig(): Promise<any> { return {}; }
async function createDbConnection(): Promise<any> { return {}; }
```

### Fan-Out/Fan-In Pattern

```typescript
// serverless/fan-out-fan-in.ts
import {
  SQSClient,
  SendMessageBatchCommand,
} from '@aws-sdk/client-sqs';
import {
  DynamoDBClient,
  PutItemCommand,
  QueryCommand,
} from '@aws-sdk/client-dynamodb';
import { marshall, unmarshall } from '@aws-sdk/util-dynamodb';
import { SQSEvent, SQSHandler } from 'aws-lambda';

const sqs = new SQSClient({});
const dynamodb = new DynamoDBClient({});

const QUEUE_URL = process.env.WORKER_QUEUE_URL!;
const TABLE_NAME = process.env.TABLE_NAME!;

/**
 * Fan-Out: Split large job into smaller tasks
 */
export async function fanOutHandler(event: {
  jobId: string;
  items: string[];
}): Promise<{ taskCount: number }> {
  const { jobId, items } = event;
  
  // Create job record
  await dynamodb.send(new PutItemCommand({
    TableName: TABLE_NAME,
    Item: marshall({
      pk: `JOB#${jobId}`,
      sk: 'META',
      status: 'processing',
      totalTasks: items.length,
      completedTasks: 0,
      createdAt: new Date().toISOString(),
    }),
  }));
  
  // Fan out: Send each item as separate task
  const batches = [];
  for (let i = 0; i < items.length; i += 10) {
    const batch = items.slice(i, i + 10);
    batches.push(batch);
  }
  
  for (const batch of batches) {
    await sqs.send(new SendMessageBatchCommand({
      QueueUrl: QUEUE_URL,
      Entries: batch.map((item, index) => ({
        Id: `${index}`,
        MessageBody: JSON.stringify({
          jobId,
          item,
          taskIndex: items.indexOf(item),
        }),
      })),
    }));
  }
  
  return { taskCount: items.length };
}

/**
 * Worker: Process individual task
 */
export const workerHandler: SQSHandler = async (event: SQSEvent) => {
  for (const record of event.Records) {
    const task = JSON.parse(record.body);
    const { jobId, item, taskIndex } = task;
    
    try {
      // Process the item
      const result = await processItem(item);
      
      // Save result
      await dynamodb.send(new PutItemCommand({
        TableName: TABLE_NAME,
        Item: marshall({
          pk: `JOB#${jobId}`,
          sk: `TASK#${String(taskIndex).padStart(6, '0')}`,
          status: 'completed',
          result,
          completedAt: new Date().toISOString(),
        }),
      }));
      
      // Update completed count (atomic)
      await dynamodb.send(new PutItemCommand({
        TableName: TABLE_NAME,
        Item: marshall({
          pk: `JOB#${jobId}`,
          sk: 'META',
        }),
        // Use UpdateItem with ADD in real implementation
      }));
      
    } catch (error) {
      // Save failure
      await dynamodb.send(new PutItemCommand({
        TableName: TABLE_NAME,
        Item: marshall({
          pk: `JOB#${jobId}`,
          sk: `TASK#${String(taskIndex).padStart(6, '0')}`,
          status: 'failed',
          error: (error as Error).message,
        }),
      }));
    }
  }
};

/**
 * Fan-In: Aggregate results when all tasks complete
 */
export async function fanInHandler(event: {
  jobId: string;
}): Promise<{ results: any[] }> {
  const { jobId } = event;
  
  // Query all task results
  const response = await dynamodb.send(new QueryCommand({
    TableName: TABLE_NAME,
    KeyConditionExpression: 'pk = :pk AND begins_with(sk, :sk)',
    ExpressionAttributeValues: marshall({
      ':pk': `JOB#${jobId}`,
      ':sk': 'TASK#',
    }),
  }));
  
  const results = (response.Items || [])
    .map(item => unmarshall(item))
    .filter(item => item.status === 'completed')
    .map(item => item.result);
  
  // Update job status
  await dynamodb.send(new PutItemCommand({
    TableName: TABLE_NAME,
    Item: marshall({
      pk: `JOB#${jobId}`,
      sk: 'META',
      status: 'completed',
      completedAt: new Date().toISOString(),
    }),
  }));
  
  return { results };
}

async function processItem(item: string): Promise<any> {
  // Simulate processing
  return { processed: item };
}
```

### Event-Driven Patterns

```typescript
// serverless/event-driven.ts
import { S3Event, S3Handler, DynamoDBStreamEvent, SNSEvent } from 'aws-lambda';
import { S3Client, GetObjectCommand, PutObjectCommand } from '@aws-sdk/client-s3';
import { Readable } from 'stream';

const s3 = new S3Client({});

/**
 * S3 Event Trigger - Image Processing
 */
export const imageProcessor: S3Handler = async (event: S3Event) => {
  for (const record of event.Records) {
    const bucket = record.s3.bucket.name;
    const key = decodeURIComponent(record.s3.object.key.replace(/\+/g, ' '));
    
    // Skip if already processed
    if (key.startsWith('processed/')) continue;
    
    console.log(`Processing: s3://${bucket}/${key}`);
    
    // Get original image
    const original = await s3.send(new GetObjectCommand({
      Bucket: bucket,
      Key: key,
    }));
    
    // Process image (resize, etc.)
    const processed = await processImage(original.Body as Readable);
    
    // Save processed image
    const processedKey = `processed/${key}`;
    await s3.send(new PutObjectCommand({
      Bucket: bucket,
      Key: processedKey,
      Body: processed,
      ContentType: 'image/jpeg',
    }));
    
    console.log(`Saved: s3://${bucket}/${processedKey}`);
  }
};

/**
 * DynamoDB Stream Trigger - Sync to Elasticsearch
 */
export async function dynamoStreamHandler(
  event: DynamoDBStreamEvent
): Promise<void> {
  for (const record of event.Records) {
    const eventName = record.eventName;
    const newImage = record.dynamodb?.NewImage;
    const oldImage = record.dynamodb?.OldImage;
    
    switch (eventName) {
      case 'INSERT':
      case 'MODIFY':
        await indexDocument(newImage);
        break;
      case 'REMOVE':
        await removeDocument(oldImage);
        break;
    }
  }
}

/**
 * SNS Trigger - Multi-subscriber notification
 */
export async function snsHandler(event: SNSEvent): Promise<void> {
  for (const record of event.Records) {
    const message = JSON.parse(record.Sns.Message);
    const messageAttributes = record.Sns.MessageAttributes;
    
    // Route based on message type
    const messageType = messageAttributes['type']?.Value;
    
    switch (messageType) {
      case 'order.created':
        await handleOrderCreated(message);
        break;
      case 'user.signup':
        await handleUserSignup(message);
        break;
      default:
        console.log('Unknown message type:', messageType);
    }
  }
}

async function processImage(stream: Readable): Promise<Buffer> {
  // Image processing logic
  return Buffer.from('');
}

async function indexDocument(doc: any): Promise<void> {}
async function removeDocument(doc: any): Promise<void> {}
async function handleOrderCreated(message: any): Promise<void> {}
async function handleUserSignup(message: any): Promise<void> {}
```

---

## Real-World Scenarios

### Scenario 1: API with Async Processing

```typescript
// serverless/async-api.ts

/**
 * Pattern: Synchronous API returns immediately,
 * actual processing happens asynchronously
 */

import { APIGatewayProxyHandler } from 'aws-lambda';
import { SQSClient, SendMessageCommand } from '@aws-sdk/client-sqs';
import { DynamoDBClient, PutItemCommand, GetItemCommand } from '@aws-sdk/client-dynamodb';
import { marshall, unmarshall } from '@aws-sdk/util-dynamodb';
import { randomUUID } from 'crypto';

const sqs = new SQSClient({});
const dynamodb = new DynamoDBClient({});

// POST /reports - Start report generation
export const createReport: APIGatewayProxyHandler = async (event) => {
  const body = JSON.parse(event.body || '{}');
  const reportId = randomUUID();
  
  // Create job record
  await dynamodb.send(new PutItemCommand({
    TableName: process.env.TABLE_NAME!,
    Item: marshall({
      pk: `REPORT#${reportId}`,
      sk: 'STATUS',
      status: 'pending',
      params: body,
      createdAt: new Date().toISOString(),
    }),
  }));
  
  // Queue async processing
  await sqs.send(new SendMessageCommand({
    QueueUrl: process.env.QUEUE_URL!,
    MessageBody: JSON.stringify({
      reportId,
      params: body,
    }),
  }));
  
  // Return immediately with job ID
  return {
    statusCode: 202, // Accepted
    headers: {
      'Content-Type': 'application/json',
      'Location': `/reports/${reportId}`,
    },
    body: JSON.stringify({
      reportId,
      status: 'pending',
      statusUrl: `/reports/${reportId}`,
    }),
  };
};

// GET /reports/:id - Check status
export const getReport: APIGatewayProxyHandler = async (event) => {
  const reportId = event.pathParameters?.id;
  
  const response = await dynamodb.send(new GetItemCommand({
    TableName: process.env.TABLE_NAME!,
    Key: marshall({
      pk: `REPORT#${reportId}`,
      sk: 'STATUS',
    }),
  }));
  
  if (!response.Item) {
    return {
      statusCode: 404,
      body: JSON.stringify({ error: 'Report not found' }),
    };
  }
  
  const report = unmarshall(response.Item);
  
  return {
    statusCode: 200,
    body: JSON.stringify({
      reportId,
      status: report.status,
      downloadUrl: report.downloadUrl,
      createdAt: report.createdAt,
      completedAt: report.completedAt,
    }),
  };
};
```

### Scenario 2: Cost-Optimized Batch Processing

```typescript
// serverless/batch-processing.ts

/**
 * Cost optimization strategies:
 * 1. Batch records together
 * 2. Use reserved concurrency to limit costs
 * 3. Process during off-peak for spot pricing
 * 4. Use S3 Select for filtering before Lambda
 */

import { S3Event, SQSEvent } from 'aws-lambda';

// Process S3 objects in batch (triggered by S3 inventory or scheduled)
export async function batchS3Processor(event: {
  bucket: string;
  prefix: string;
}): Promise<{ processed: number }> {
  const { bucket, prefix } = event;
  
  // Use S3 Select to filter data before processing
  // Reduces data transfer costs
  const filteredData = await s3SelectQuery(bucket, prefix, `
    SELECT * FROM s3object s 
    WHERE s.status = 'pending'
  `);
  
  let processed = 0;
  
  for (const record of filteredData) {
    await processRecord(record);
    processed++;
  }
  
  return { processed };
}

// SQS batch processing with partial failure reporting
export async function sqsBatchProcessor(event: SQSEvent): Promise<{
  batchItemFailures: Array<{ itemIdentifier: string }>;
}> {
  const failures: string[] = [];
  
  // Process all messages
  await Promise.all(
    event.Records.map(async (record) => {
      try {
        const body = JSON.parse(record.body);
        await processRecord(body);
      } catch (error) {
        console.error(`Failed ${record.messageId}:`, error);
        failures.push(record.messageId);
      }
    })
  );
  
  // Report partial failures - only failed messages return to queue
  return {
    batchItemFailures: failures.map(id => ({ itemIdentifier: id })),
  };
}

// serverless.yml for cost-optimized batch
const batchConfig = `
functions:
  batchProcessor:
    handler: src/batch.sqsBatchProcessor
    timeout: 900  # 15 minutes max
    memorySize: 256
    reservedConcurrency: 10  # Limit concurrent executions = limit cost
    events:
      - sqs:
          arn: !GetAtt BatchQueue.Arn
          batchSize: 100  # Process 100 messages per invocation
          maximumBatchingWindow: 30  # Wait up to 30s to fill batch
          functionResponseType: ReportBatchItemFailures
`;

async function s3SelectQuery(bucket: string, prefix: string, query: string): Promise<any[]> {
  return [];
}
async function processRecord(record: any): Promise<void> {}
```

---

## Common Pitfalls

### 1. Not Handling Idempotency

```typescript
// ❌ BAD: Non-idempotent handler
export async function handler(event: any) {
  // Creates duplicate if retried!
  await createOrder(event.order);
}

// ✅ GOOD: Idempotent with deduplication
export async function handler(event: any) {
  const idempotencyKey = event.order.orderId;
  
  // Check if already processed
  const existing = await getOrder(idempotencyKey);
  if (existing) {
    return existing; // Return cached result
  }
  
  const result = await createOrder(event.order);
  await cacheResult(idempotencyKey, result);
  return result;
}

async function createOrder(order: any): Promise<any> { return order; }
async function getOrder(id: string): Promise<any> { return null; }
async function cacheResult(key: string, result: any): Promise<void> {}
```

### 2. Ignoring Concurrency Limits

```typescript
// ❌ BAD: Unbounded concurrency can exhaust downstream resources
// Default Lambda concurrency is 1000 per region!

// ✅ GOOD: Set reserved concurrency
const config = `
functions:
  dbWriter:
    handler: src/writer.handler
    reservedConcurrency: 10  # Match DB connection pool
`;

// Or use SQS to control throughput
const sqsConfig = `
functions:
  processor:
    handler: src/processor.handler
    reservedConcurrency: 5
    events:
      - sqs:
          arn: !GetAtt Queue.Arn
          batchSize: 10
`;
```

### 3. Large Payloads in Lambda

```typescript
// ❌ BAD: Passing large data between functions
stepFunction.startExecution({
  input: JSON.stringify({
    largeData: hugeArray, // 256KB limit!
  }),
});

// ✅ GOOD: Store in S3, pass reference
const key = `data/${executionId}.json`;
await s3.putObject({
  Bucket: bucket,
  Key: key,
  Body: JSON.stringify(hugeArray),
});

stepFunction.startExecution({
  input: JSON.stringify({
    dataLocation: { bucket, key },
  }),
});

const stepFunction = { startExecution: (input: any) => {} };
const s3 = { putObject: (params: any) => {} };
const hugeArray: any[] = [];
const executionId = '';
const bucket = '';
```

---

## Interview Questions

### Q1: How do you handle cold starts in serverless?

**A:** 1) Provisioned concurrency (guaranteed warm instances). 2) Keep-warm with scheduled invocations. 3) Smaller bundle size (less to load). 4) Initialize outside handler (reuse across invocations). 5) Choose faster runtimes (Node.js, Python over Java). 6) Use VPC only when necessary (adds latency).

### Q2: When would you use Step Functions vs direct Lambda chaining?

**A:** **Step Functions** for: complex workflows, error handling/retry, parallel execution, long-running processes, visual debugging, audit requirements. **Direct chaining** for: simple A→B flows, low latency requirements, cost sensitivity. Step Functions add latency and cost but provide reliability and observability.

### Q3: How do you handle state in serverless applications?

**A:** Functions are stateless by design. Use: DynamoDB for fast key-value state, ElastiCache for caching, S3 for large objects, Step Functions for workflow state. Design for: idempotency (retries), eventual consistency, storing minimal state in functions (connection reuse).

### Q4: How do you debug serverless applications?

**A:** 1) CloudWatch Logs for function logs. 2) X-Ray for distributed tracing. 3) Local testing with SAM/Serverless offline. 4) Structured logging with correlation IDs. 5) CloudWatch Insights for log queries. 6) Step Functions visual workflow. The key is good logging and tracing since you can't SSH into a Lambda.

---

## Quick Reference Checklist

### Cold Start Optimization
- [ ] Initialize outside handler
- [ ] Minimize bundle size
- [ ] Consider provisioned concurrency
- [ ] Keep-warm for critical functions

### Reliability
- [ ] Design for idempotency
- [ ] Handle partial failures
- [ ] Set appropriate timeouts
- [ ] Configure DLQ for failures

### Cost Optimization
- [ ] Right-size memory allocation
- [ ] Use reserved concurrency
- [ ] Batch processing where possible
- [ ] Monitor and alert on costs

### Observability
- [ ] Enable X-Ray tracing
- [ ] Structured JSON logging
- [ ] Correlation IDs across services
- [ ] Dashboard for key metrics

---

*Last updated: February 2026*

