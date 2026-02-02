# AWS Core Services - Complete Guide

> **MUST REMEMBER**: AWS core services: EC2 (compute), S3 (object storage), RDS (managed relational DB), Lambda (serverless), DynamoDB (NoSQL), SQS (message queue), SNS (pub/sub notifications). Choose based on: scaling needs, cost model (on-demand vs reserved), operational overhead. S3 for static files, RDS for relational data, DynamoDB for high-throughput key-value, Lambda for event-driven compute, SQS/SNS for decoupling.

---

## How to Explain Like a Senior Developer

"AWS is a building blocks approach to infrastructure. EC2 gives you virtual servers - full control but you manage everything. S3 is infinitely scalable object storage - perfect for files, backups, static hosting. RDS manages your database (PostgreSQL, MySQL) - handles backups, patches, replicas. Lambda runs code without servers - pay only for execution time, great for event-driven workloads but watch for cold starts. DynamoDB is a fully managed NoSQL database with single-digit millisecond latency at any scale. SQS queues messages between services for decoupling; SNS pushes notifications to multiple subscribers. The art is choosing the right service: managed services cost more but reduce ops burden; EC2 is cheaper at scale but requires more work."

---

## Core Implementation

### EC2 - Elastic Compute Cloud

```typescript
// aws/ec2.ts
import {
  EC2Client,
  RunInstancesCommand,
  DescribeInstancesCommand,
  TerminateInstancesCommand,
  CreateTagsCommand,
  AuthorizeSecurityGroupIngressCommand,
  CreateSecurityGroupCommand,
} from '@aws-sdk/client-ec2';

const ec2 = new EC2Client({ region: 'us-east-1' });

// Launch an EC2 instance
async function launchInstance(config: {
  name: string;
  imageId: string;      // AMI ID
  instanceType: string; // t3.micro, t3.medium, etc.
  keyName: string;      // SSH key pair name
  securityGroupId: string;
  userData?: string;    // Bootstrap script
}): Promise<string> {
  // User data script (runs on first boot)
  const userData = config.userData || `#!/bin/bash
    yum update -y
    yum install -y docker
    service docker start
    usermod -a -G docker ec2-user
  `;
  
  const command = new RunInstancesCommand({
    ImageId: config.imageId,
    InstanceType: config.instanceType,
    MinCount: 1,
    MaxCount: 1,
    KeyName: config.keyName,
    SecurityGroupIds: [config.securityGroupId],
    UserData: Buffer.from(userData).toString('base64'),
    TagSpecifications: [{
      ResourceType: 'instance',
      Tags: [
        { Key: 'Name', Value: config.name },
        { Key: 'Environment', Value: 'production' },
      ],
    }],
    // Enable detailed monitoring
    Monitoring: { Enabled: true },
    // EBS-optimized for better I/O
    EbsOptimized: true,
  });
  
  const response = await ec2.send(command);
  const instanceId = response.Instances?.[0]?.InstanceId;
  
  if (!instanceId) throw new Error('Failed to launch instance');
  
  console.log(`Launched instance: ${instanceId}`);
  return instanceId;
}

// Create security group
async function createSecurityGroup(
  name: string,
  vpcId: string
): Promise<string> {
  const createCommand = new CreateSecurityGroupCommand({
    GroupName: name,
    Description: `Security group for ${name}`,
    VpcId: vpcId,
  });
  
  const response = await ec2.send(createCommand);
  const groupId = response.GroupId!;
  
  // Allow SSH and HTTP
  await ec2.send(new AuthorizeSecurityGroupIngressCommand({
    GroupId: groupId,
    IpPermissions: [
      {
        IpProtocol: 'tcp',
        FromPort: 22,
        ToPort: 22,
        IpRanges: [{ CidrIp: '10.0.0.0/8', Description: 'Internal SSH' }],
      },
      {
        IpProtocol: 'tcp',
        FromPort: 443,
        ToPort: 443,
        IpRanges: [{ CidrIp: '0.0.0.0/0', Description: 'HTTPS' }],
      },
    ],
  }));
  
  return groupId;
}

// Wait for instance to be running
async function waitForInstance(instanceId: string): Promise<void> {
  let running = false;
  
  while (!running) {
    const command = new DescribeInstancesCommand({
      InstanceIds: [instanceId],
    });
    
    const response = await ec2.send(command);
    const state = response.Reservations?.[0]?.Instances?.[0]?.State?.Name;
    
    if (state === 'running') {
      running = true;
    } else {
      console.log(`Instance state: ${state}, waiting...`);
      await new Promise(resolve => setTimeout(resolve, 5000));
    }
  }
}
```

### S3 - Simple Storage Service

```typescript
// aws/s3.ts
import {
  S3Client,
  PutObjectCommand,
  GetObjectCommand,
  DeleteObjectCommand,
  ListObjectsV2Command,
  CopyObjectCommand,
  HeadObjectCommand,
} from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';
import { Readable } from 'stream';

const s3 = new S3Client({ region: 'us-east-1' });
const BUCKET = 'my-app-bucket';

// Upload file
async function uploadFile(
  key: string,
  body: Buffer | Readable | string,
  contentType: string,
  options?: {
    isPublic?: boolean;
    metadata?: Record<string, string>;
    cacheControl?: string;
  }
): Promise<string> {
  const command = new PutObjectCommand({
    Bucket: BUCKET,
    Key: key,
    Body: body,
    ContentType: contentType,
    ACL: options?.isPublic ? 'public-read' : 'private',
    Metadata: options?.metadata,
    CacheControl: options?.cacheControl || 'max-age=31536000', // 1 year
  });
  
  await s3.send(command);
  
  return options?.isPublic
    ? `https://${BUCKET}.s3.amazonaws.com/${key}`
    : key;
}

// Generate presigned URL for upload
async function getUploadUrl(
  key: string,
  contentType: string,
  expiresIn: number = 3600
): Promise<string> {
  const command = new PutObjectCommand({
    Bucket: BUCKET,
    Key: key,
    ContentType: contentType,
  });
  
  return getSignedUrl(s3, command, { expiresIn });
}

// Generate presigned URL for download
async function getDownloadUrl(
  key: string,
  expiresIn: number = 3600
): Promise<string> {
  const command = new GetObjectCommand({
    Bucket: BUCKET,
    Key: key,
  });
  
  return getSignedUrl(s3, command, { expiresIn });
}

// Download file
async function downloadFile(key: string): Promise<Buffer> {
  const command = new GetObjectCommand({
    Bucket: BUCKET,
    Key: key,
  });
  
  const response = await s3.send(command);
  const stream = response.Body as Readable;
  
  const chunks: Buffer[] = [];
  for await (const chunk of stream) {
    chunks.push(chunk);
  }
  
  return Buffer.concat(chunks);
}

// List objects with pagination
async function listObjects(
  prefix: string,
  maxKeys: number = 1000
): Promise<string[]> {
  const keys: string[] = [];
  let continuationToken: string | undefined;
  
  do {
    const command = new ListObjectsV2Command({
      Bucket: BUCKET,
      Prefix: prefix,
      MaxKeys: maxKeys,
      ContinuationToken: continuationToken,
    });
    
    const response = await s3.send(command);
    
    for (const obj of response.Contents || []) {
      if (obj.Key) keys.push(obj.Key);
    }
    
    continuationToken = response.NextContinuationToken;
  } while (continuationToken);
  
  return keys;
}

// Copy object (for moving/renaming)
async function copyObject(sourceKey: string, destKey: string): Promise<void> {
  await s3.send(new CopyObjectCommand({
    Bucket: BUCKET,
    CopySource: `${BUCKET}/${sourceKey}`,
    Key: destKey,
  }));
}

// Check if object exists
async function objectExists(key: string): Promise<boolean> {
  try {
    await s3.send(new HeadObjectCommand({
      Bucket: BUCKET,
      Key: key,
    }));
    return true;
  } catch (error: any) {
    if (error.name === 'NotFound') return false;
    throw error;
  }
}
```

### Lambda - Serverless Functions

```typescript
// aws/lambda-handler.ts
import { 
  APIGatewayProxyEvent, 
  APIGatewayProxyResult,
  S3Event,
  SQSEvent,
  Context,
} from 'aws-lambda';

// API Gateway handler
export async function apiHandler(
  event: APIGatewayProxyEvent,
  context: Context
): Promise<APIGatewayProxyResult> {
  // Log for CloudWatch
  console.log('Request:', JSON.stringify(event, null, 2));
  
  try {
    const { httpMethod, path, body, queryStringParameters } = event;
    
    // Route handling
    if (httpMethod === 'GET' && path === '/users') {
      const users = await getUsers();
      return response(200, { users });
    }
    
    if (httpMethod === 'POST' && path === '/users') {
      const data = JSON.parse(body || '{}');
      const user = await createUser(data);
      return response(201, { user });
    }
    
    return response(404, { error: 'Not Found' });
    
  } catch (error: any) {
    console.error('Error:', error);
    return response(500, { error: error.message });
  }
}

// S3 trigger handler
export async function s3Handler(event: S3Event): Promise<void> {
  for (const record of event.Records) {
    const bucket = record.s3.bucket.name;
    const key = decodeURIComponent(record.s3.object.key.replace(/\+/g, ' '));
    
    console.log(`Processing: s3://${bucket}/${key}`);
    
    // Process uploaded file (e.g., resize image, extract text)
    if (key.endsWith('.jpg') || key.endsWith('.png')) {
      await processImage(bucket, key);
    }
  }
}

// SQS trigger handler
export async function sqsHandler(event: SQSEvent): Promise<void> {
  // Process messages in batch
  const failures: string[] = [];
  
  for (const record of event.Records) {
    try {
      const message = JSON.parse(record.body);
      await processMessage(message);
    } catch (error) {
      console.error(`Failed to process ${record.messageId}:`, error);
      failures.push(record.messageId);
    }
  }
  
  // Partial batch failure reporting
  if (failures.length > 0) {
    throw new Error(`Failed to process: ${failures.join(', ')}`);
  }
}

// Helper functions
function response(
  statusCode: number,
  body: object
): APIGatewayProxyResult {
  return {
    statusCode,
    headers: {
      'Content-Type': 'application/json',
      'Access-Control-Allow-Origin': '*',
    },
    body: JSON.stringify(body),
  };
}

async function getUsers(): Promise<any[]> { return []; }
async function createUser(data: any): Promise<any> { return data; }
async function processImage(bucket: string, key: string): Promise<void> {}
async function processMessage(message: any): Promise<void> {}
```

### DynamoDB - NoSQL Database

```typescript
// aws/dynamodb.ts
import {
  DynamoDBClient,
  PutItemCommand,
  GetItemCommand,
  QueryCommand,
  UpdateItemCommand,
  DeleteItemCommand,
  BatchWriteItemCommand,
  TransactWriteItemsCommand,
} from '@aws-sdk/client-dynamodb';
import { marshall, unmarshall } from '@aws-sdk/util-dynamodb';

const dynamodb = new DynamoDBClient({ region: 'us-east-1' });
const TABLE_NAME = 'Users';

interface User {
  pk: string;  // Partition key: USER#<id>
  sk: string;  // Sort key: PROFILE
  id: string;
  email: string;
  name: string;
  createdAt: string;
}

// Put item
async function createUser(user: Omit<User, 'pk' | 'sk'>): Promise<User> {
  const item: User = {
    pk: `USER#${user.id}`,
    sk: 'PROFILE',
    ...user,
    createdAt: new Date().toISOString(),
  };
  
  await dynamodb.send(new PutItemCommand({
    TableName: TABLE_NAME,
    Item: marshall(item),
    ConditionExpression: 'attribute_not_exists(pk)', // Prevent overwrite
  }));
  
  return item;
}

// Get item
async function getUser(userId: string): Promise<User | null> {
  const response = await dynamodb.send(new GetItemCommand({
    TableName: TABLE_NAME,
    Key: marshall({
      pk: `USER#${userId}`,
      sk: 'PROFILE',
    }),
  }));
  
  if (!response.Item) return null;
  return unmarshall(response.Item) as User;
}

// Query with partition key
async function getUserOrders(userId: string): Promise<any[]> {
  const response = await dynamodb.send(new QueryCommand({
    TableName: TABLE_NAME,
    KeyConditionExpression: 'pk = :pk AND begins_with(sk, :sk)',
    ExpressionAttributeValues: marshall({
      ':pk': `USER#${userId}`,
      ':sk': 'ORDER#',
    }),
    ScanIndexForward: false, // Descending order
    Limit: 10,
  }));
  
  return (response.Items || []).map(item => unmarshall(item));
}

// Query with GSI
async function getUserByEmail(email: string): Promise<User | null> {
  const response = await dynamodb.send(new QueryCommand({
    TableName: TABLE_NAME,
    IndexName: 'email-index',
    KeyConditionExpression: 'email = :email',
    ExpressionAttributeValues: marshall({ ':email': email }),
    Limit: 1,
  }));
  
  if (!response.Items?.length) return null;
  return unmarshall(response.Items[0]) as User;
}

// Update item
async function updateUser(
  userId: string,
  updates: Partial<User>
): Promise<User> {
  const updateExpressions: string[] = [];
  const expressionAttributeNames: Record<string, string> = {};
  const expressionAttributeValues: Record<string, any> = {};
  
  Object.entries(updates).forEach(([key, value], index) => {
    if (key === 'pk' || key === 'sk') return;
    
    updateExpressions.push(`#field${index} = :value${index}`);
    expressionAttributeNames[`#field${index}`] = key;
    expressionAttributeValues[`:value${index}`] = value;
  });
  
  const response = await dynamodb.send(new UpdateItemCommand({
    TableName: TABLE_NAME,
    Key: marshall({
      pk: `USER#${userId}`,
      sk: 'PROFILE',
    }),
    UpdateExpression: `SET ${updateExpressions.join(', ')}`,
    ExpressionAttributeNames: expressionAttributeNames,
    ExpressionAttributeValues: marshall(expressionAttributeValues),
    ReturnValues: 'ALL_NEW',
  }));
  
  return unmarshall(response.Attributes!) as User;
}

// Transaction (atomic multi-item operations)
async function createOrder(
  userId: string,
  orderId: string,
  amount: number
): Promise<void> {
  await dynamodb.send(new TransactWriteItemsCommand({
    TransactItems: [
      // Create order
      {
        Put: {
          TableName: TABLE_NAME,
          Item: marshall({
            pk: `USER#${userId}`,
            sk: `ORDER#${orderId}`,
            orderId,
            amount,
            status: 'pending',
            createdAt: new Date().toISOString(),
          }),
        },
      },
      // Update user order count
      {
        Update: {
          TableName: TABLE_NAME,
          Key: marshall({
            pk: `USER#${userId}`,
            sk: 'PROFILE',
          }),
          UpdateExpression: 'ADD orderCount :inc',
          ExpressionAttributeValues: marshall({ ':inc': 1 }),
        },
      },
    ],
  }));
}

// Batch write (up to 25 items)
async function batchCreateUsers(users: User[]): Promise<void> {
  const chunks = [];
  for (let i = 0; i < users.length; i += 25) {
    chunks.push(users.slice(i, i + 25));
  }
  
  for (const chunk of chunks) {
    await dynamodb.send(new BatchWriteItemCommand({
      RequestItems: {
        [TABLE_NAME]: chunk.map(user => ({
          PutRequest: {
            Item: marshall(user),
          },
        })),
      },
    }));
  }
}
```

### SQS & SNS - Messaging

```typescript
// aws/messaging.ts
import {
  SQSClient,
  SendMessageCommand,
  ReceiveMessageCommand,
  DeleteMessageCommand,
  SendMessageBatchCommand,
} from '@aws-sdk/client-sqs';
import {
  SNSClient,
  PublishCommand,
  SubscribeCommand,
} from '@aws-sdk/client-sns';

const sqs = new SQSClient({ region: 'us-east-1' });
const sns = new SNSClient({ region: 'us-east-1' });

const QUEUE_URL = 'https://sqs.us-east-1.amazonaws.com/123456789/my-queue';
const TOPIC_ARN = 'arn:aws:sns:us-east-1:123456789:my-topic';

// SQS - Send message
async function sendMessage(
  body: object,
  options?: {
    delaySeconds?: number;
    messageGroupId?: string; // For FIFO queues
    deduplicationId?: string;
  }
): Promise<string> {
  const command = new SendMessageCommand({
    QueueUrl: QUEUE_URL,
    MessageBody: JSON.stringify(body),
    DelaySeconds: options?.delaySeconds,
    MessageGroupId: options?.messageGroupId,
    MessageDeduplicationId: options?.deduplicationId,
    MessageAttributes: {
      'type': {
        DataType: 'String',
        StringValue: 'order-created',
      },
    },
  });
  
  const response = await sqs.send(command);
  return response.MessageId!;
}

// SQS - Send batch (up to 10 messages)
async function sendMessageBatch(
  messages: Array<{ id: string; body: object }>
): Promise<void> {
  const command = new SendMessageBatchCommand({
    QueueUrl: QUEUE_URL,
    Entries: messages.map(msg => ({
      Id: msg.id,
      MessageBody: JSON.stringify(msg.body),
    })),
  });
  
  const response = await sqs.send(command);
  
  if (response.Failed?.length) {
    console.error('Failed messages:', response.Failed);
  }
}

// SQS - Receive and process messages
async function pollMessages(): Promise<void> {
  const command = new ReceiveMessageCommand({
    QueueUrl: QUEUE_URL,
    MaxNumberOfMessages: 10,
    WaitTimeSeconds: 20, // Long polling
    VisibilityTimeout: 30,
    MessageAttributeNames: ['All'],
  });
  
  const response = await sqs.send(command);
  
  for (const message of response.Messages || []) {
    try {
      const body = JSON.parse(message.Body!);
      await processMessage(body);
      
      // Delete after successful processing
      await sqs.send(new DeleteMessageCommand({
        QueueUrl: QUEUE_URL,
        ReceiptHandle: message.ReceiptHandle,
      }));
    } catch (error) {
      console.error(`Failed to process ${message.MessageId}:`, error);
      // Message will return to queue after visibility timeout
    }
  }
}

// SNS - Publish to topic
async function publishNotification(
  subject: string,
  message: object | string
): Promise<string> {
  const command = new PublishCommand({
    TopicArn: TOPIC_ARN,
    Subject: subject,
    Message: typeof message === 'string' ? message : JSON.stringify(message),
    MessageAttributes: {
      'event': {
        DataType: 'String',
        StringValue: 'user-signup',
      },
    },
  });
  
  const response = await sns.send(command);
  return response.MessageId!;
}

// SNS - Publish with different formats per protocol
async function publishMultiFormat(
  subject: string,
  messages: {
    default: string;
    email?: string;
    sms?: string;
    lambda?: string;
  }
): Promise<string> {
  const command = new PublishCommand({
    TopicArn: TOPIC_ARN,
    Subject: subject,
    Message: JSON.stringify(messages),
    MessageStructure: 'json', // Enable per-protocol messages
  });
  
  const response = await sns.send(command);
  return response.MessageId!;
}

// SNS - Subscribe endpoint
async function subscribeToTopic(
  protocol: 'email' | 'sms' | 'lambda' | 'sqs' | 'https',
  endpoint: string
): Promise<string> {
  const command = new SubscribeCommand({
    TopicArn: TOPIC_ARN,
    Protocol: protocol,
    Endpoint: endpoint,
  });
  
  const response = await sns.send(command);
  return response.SubscriptionArn!;
}

async function processMessage(body: any): Promise<void> {
  console.log('Processing:', body);
}
```

---

## Real-World Scenarios

### Scenario 1: Complete Serverless API Architecture

```typescript
// serverless/architecture.ts

/**
 * Serverless API Architecture:
 * 
 * API Gateway → Lambda → DynamoDB
 *                  ↓
 *               S3 (files)
 *                  ↓
 *               SQS (async tasks)
 *                  ↓
 *            Lambda (worker)
 *                  ↓
 *               SNS (notifications)
 */

// serverless.yml (Serverless Framework)
const serverlessConfig = `
service: my-api

provider:
  name: aws
  runtime: nodejs18.x
  region: us-east-1
  environment:
    TABLE_NAME: \${self:service}-\${sls:stage}
    BUCKET_NAME: \${self:service}-uploads-\${sls:stage}
    QUEUE_URL: !Ref TaskQueue
  iam:
    role:
      statements:
        - Effect: Allow
          Action:
            - dynamodb:*
          Resource:
            - !GetAtt Table.Arn
            - !Sub "\${Table.Arn}/index/*"
        - Effect: Allow
          Action:
            - s3:*
          Resource:
            - !Sub "arn:aws:s3:::\${self:provider.environment.BUCKET_NAME}/*"
        - Effect: Allow
          Action:
            - sqs:*
          Resource:
            - !GetAtt TaskQueue.Arn

functions:
  api:
    handler: src/api.handler
    events:
      - http:
          path: /{proxy+}
          method: ANY
          cors: true
    timeout: 29
    memorySize: 512
  
  worker:
    handler: src/worker.handler
    events:
      - sqs:
          arn: !GetAtt TaskQueue.Arn
          batchSize: 10
    timeout: 300
    memorySize: 1024
  
  s3Processor:
    handler: src/s3.handler
    events:
      - s3:
          bucket: \${self:provider.environment.BUCKET_NAME}
          event: s3:ObjectCreated:*
          rules:
            - prefix: uploads/

resources:
  Resources:
    Table:
      Type: AWS::DynamoDB::Table
      Properties:
        TableName: \${self:provider.environment.TABLE_NAME}
        BillingMode: PAY_PER_REQUEST
        AttributeDefinitions:
          - AttributeName: pk
            AttributeType: S
          - AttributeName: sk
            AttributeType: S
          - AttributeName: gsi1pk
            AttributeType: S
          - AttributeName: gsi1sk
            AttributeType: S
        KeySchema:
          - AttributeName: pk
            KeyType: HASH
          - AttributeName: sk
            KeyType: RANGE
        GlobalSecondaryIndexes:
          - IndexName: gsi1
            KeySchema:
              - AttributeName: gsi1pk
                KeyType: HASH
              - AttributeName: gsi1sk
                KeyType: RANGE
            Projection:
              ProjectionType: ALL
    
    TaskQueue:
      Type: AWS::SQS::Queue
      Properties:
        QueueName: \${self:service}-tasks-\${sls:stage}
        VisibilityTimeout: 300
        RedrivePolicy:
          deadLetterTargetArn: !GetAtt DeadLetterQueue.Arn
          maxReceiveCount: 3
    
    DeadLetterQueue:
      Type: AWS::SQS::Queue
      Properties:
        QueueName: \${self:service}-dlq-\${sls:stage}
`;
```

### Scenario 2: Cost-Optimized Multi-Tier Architecture

```typescript
// architecture/multi-tier.ts

/**
 * Cost-Optimized Architecture:
 * 
 * Tier 1: CloudFront CDN + S3 (static assets)
 * Tier 2: ALB + EC2 Auto Scaling (API)
 * Tier 3: RDS Multi-AZ (database)
 * Tier 4: ElastiCache (caching)
 * 
 * Cost optimizations:
 * - Reserved instances for predictable workloads
 * - Spot instances for fault-tolerant workers
 * - S3 lifecycle policies for old data
 * - CloudWatch alarms for rightsizing
 */

import { 
  AutoScalingClient, 
  CreateAutoScalingGroupCommand,
  PutScalingPolicyCommand,
} from '@aws-sdk/client-auto-scaling';

const autoScaling = new AutoScalingClient({ region: 'us-east-1' });

// Create Auto Scaling Group with mixed instances
async function createAutoScalingGroup(): Promise<void> {
  // Auto Scaling Group
  await autoScaling.send(new CreateAutoScalingGroupCommand({
    AutoScalingGroupName: 'my-api-asg',
    LaunchTemplate: {
      LaunchTemplateId: 'lt-xxx',
      Version: '$Latest',
    },
    MinSize: 2,
    MaxSize: 10,
    DesiredCapacity: 2,
    VPCZoneIdentifier: 'subnet-xxx,subnet-yyy', // Multiple AZs
    TargetGroupARNs: ['arn:aws:elasticloadbalancing:...'],
    HealthCheckType: 'ELB',
    HealthCheckGracePeriod: 300,
    // Mixed instances for cost optimization
    MixedInstancesPolicy: {
      InstancesDistribution: {
        OnDemandBaseCapacity: 1, // 1 on-demand for stability
        OnDemandPercentageAboveBaseCapacity: 25, // 25% on-demand
        SpotAllocationStrategy: 'capacity-optimized',
      },
      LaunchTemplate: {
        LaunchTemplateSpecification: {
          LaunchTemplateId: 'lt-xxx',
          Version: '$Latest',
        },
        Overrides: [
          { InstanceType: 't3.medium' },
          { InstanceType: 't3a.medium' },
          { InstanceType: 't2.medium' },
        ],
      },
    },
  }));
  
  // Target tracking scaling
  await autoScaling.send(new PutScalingPolicyCommand({
    AutoScalingGroupName: 'my-api-asg',
    PolicyName: 'cpu-target-tracking',
    PolicyType: 'TargetTrackingScaling',
    TargetTrackingConfiguration: {
      PredefinedMetricSpecification: {
        PredefinedMetricType: 'ASGAverageCPUUtilization',
      },
      TargetValue: 70,
      ScaleInCooldown: 300,
      ScaleOutCooldown: 60,
    },
  }));
}
```

---

## Common Pitfalls

### 1. Lambda Cold Starts

```typescript
// ❌ BAD: Heavy initialization in handler
export async function handler(event: any) {
  const db = await connectToDatabase(); // Cold start penalty!
  return processRequest(event, db);
}

// ✅ GOOD: Initialize outside handler
let db: any;

async function getDb() {
  if (!db) {
    db = await connectToDatabase();
  }
  return db;
}

export async function handler(event: any) {
  const database = await getDb(); // Reused in warm starts
  return processRequest(event, database);
}

async function connectToDatabase(): Promise<any> { return {}; }
async function processRequest(event: any, db: any): Promise<any> { return {}; }
```

### 2. DynamoDB Hot Partitions

```typescript
// ❌ BAD: Sequential IDs cause hot partition
const item = {
  pk: 'ORDER#1', // All traffic to one partition!
  sk: 'DETAIL',
};

// ✅ GOOD: Distribute with random prefix or user-based key
const item = {
  pk: `USER#${userId}`, // Distribute across users
  sk: `ORDER#${Date.now()}#${randomId}`,
};

const userId = '';
const randomId = '';
```

### 3. Not Using SQS Dead Letter Queue

```typescript
// ❌ BAD: Failed messages lost forever
const queue = {
  QueueName: 'my-queue',
  // No DLQ!
};

// ✅ GOOD: DLQ for failed message inspection
const queue = {
  QueueName: 'my-queue',
  RedrivePolicy: JSON.stringify({
    deadLetterTargetArn: 'arn:aws:sqs:...:my-dlq',
    maxReceiveCount: 3, // Send to DLQ after 3 failures
  }),
};
```

---

## Interview Questions

### Q1: When would you choose Lambda vs EC2?

**A:** **Lambda**: event-driven workloads, variable traffic, short-running tasks (<15min), pay-per-use, no server management. **EC2**: long-running processes, consistent high traffic, need specific runtime/OS, GPU workloads, lower cost at scale. Lambda for APIs with variable traffic; EC2 for always-on services or when you need more control.

### Q2: How do you design DynamoDB tables for efficient queries?

**A:** Start with access patterns, not data structure. Use composite keys (pk + sk) for one-to-many relationships. Use GSIs for alternative access patterns. Denormalize data - store together what's queried together. Avoid scans; use queries with partition key. Consider single-table design for related entities.

### Q3: How does SQS differ from SNS?

**A:** **SQS** is a queue (one consumer per message) - messages are pulled, persist until processed, good for work distribution. **SNS** is pub/sub (many subscribers per message) - messages are pushed, no persistence, good for fan-out notifications. Often used together: SNS to fan-out to multiple SQS queues.

### Q4: How do you handle Lambda concurrency limits?

**A:** Default 1000 concurrent executions per region. Solutions: 1) Request limit increase. 2) Reserved concurrency for critical functions. 3) SQS to buffer and smooth traffic. 4) Provisioned concurrency for predictable latency. 5) Step Functions for workflow orchestration.

---

## Quick Reference Checklist

### EC2
- [ ] Choose right instance type for workload
- [ ] Use Auto Scaling for variable loads
- [ ] Enable detailed monitoring
- [ ] Use EBS-optimized instances

### S3
- [ ] Enable versioning for important buckets
- [ ] Set lifecycle policies
- [ ] Use presigned URLs for secure access
- [ ] Enable encryption at rest

### Lambda
- [ ] Minimize cold start impact
- [ ] Set appropriate memory/timeout
- [ ] Use environment variables for config
- [ ] Enable X-Ray tracing

### DynamoDB
- [ ] Design for access patterns
- [ ] Use partition key wisely
- [ ] Set up GSIs for queries
- [ ] Enable point-in-time recovery

### SQS/SNS
- [ ] Configure dead letter queues
- [ ] Use long polling for SQS
- [ ] Set appropriate visibility timeout
- [ ] Enable encryption

---

*Last updated: February 2026*

