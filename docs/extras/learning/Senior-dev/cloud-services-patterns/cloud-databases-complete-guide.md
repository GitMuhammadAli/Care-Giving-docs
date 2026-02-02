# Cloud Databases - Complete Guide

> **MUST REMEMBER**: Managed databases (RDS, Aurora, Cloud SQL) handle backups, patches, failover - you focus on data. DynamoDB/CosmosDB for NoSQL at scale. Choose based on: data model (relational vs document vs key-value), scale requirements, consistency needs, query patterns. Aurora for MySQL/PostgreSQL with 5x performance, DynamoDB for single-digit ms latency at any scale. Managed costs more but reduces ops burden significantly.

---

## How to Explain Like a Senior Developer

"Cloud databases remove the undifferentiated heavy lifting - no more 3am pages for disk space or manual failover. RDS gives you managed MySQL, PostgreSQL, etc. with automated backups and Multi-AZ failover. Aurora is AWS's cloud-native database - MySQL/PostgreSQL compatible but with storage auto-scaling and 15 read replicas. For NoSQL, DynamoDB provides consistent single-digit millisecond latency at any scale with no capacity planning. The decision is: managed vs self-hosted (cost vs control), relational vs NoSQL (query flexibility vs scale), and read/write patterns. For most applications, start with managed PostgreSQL (Aurora or RDS), add a read replica for scale, and consider DynamoDB for high-throughput key-value workloads."

---

## Core Implementation

### RDS - Managed Relational Database

```typescript
// databases/rds.ts
import { Pool, PoolConfig } from 'pg';

/**
 * Best practices for RDS connections:
 * 1. Use connection pooling
 * 2. Use IAM authentication when possible
 * 3. Connect via SSL
 * 4. Use read replicas for read-heavy workloads
 */

interface RdsConfig {
  host: string;
  port: number;
  database: string;
  user: string;
  password: string;
  ssl: boolean;
  maxConnections: number;
  idleTimeout: number;
}

function createRdsPool(config: RdsConfig): Pool {
  const poolConfig: PoolConfig = {
    host: config.host,
    port: config.port,
    database: config.database,
    user: config.user,
    password: config.password,
    max: config.maxConnections,
    idleTimeoutMillis: config.idleTimeout,
    connectionTimeoutMillis: 5000,
  };
  
  if (config.ssl) {
    poolConfig.ssl = {
      rejectUnauthorized: true,
      // AWS RDS requires this
      ca: process.env.RDS_CA_CERT,
    };
  }
  
  const pool = new Pool(poolConfig);
  
  pool.on('error', (err, client) => {
    console.error('Unexpected pool error:', err);
  });
  
  pool.on('connect', (client) => {
    // Set session parameters
    client.query("SET timezone = 'UTC'");
  });
  
  return pool;
}

// Read replica routing
class DatabaseRouter {
  private writer: Pool;
  private readers: Pool[];
  private currentReaderIndex = 0;
  
  constructor(writerConfig: RdsConfig, readerConfigs: RdsConfig[]) {
    this.writer = createRdsPool(writerConfig);
    this.readers = readerConfigs.map(c => createRdsPool(c));
  }
  
  // Write queries go to primary
  async write<T>(query: string, params?: any[]): Promise<T[]> {
    const result = await this.writer.query(query, params);
    return result.rows;
  }
  
  // Read queries go to replicas (round-robin)
  async read<T>(query: string, params?: any[]): Promise<T[]> {
    if (this.readers.length === 0) {
      return this.write(query, params);
    }
    
    const reader = this.readers[this.currentReaderIndex];
    this.currentReaderIndex = (this.currentReaderIndex + 1) % this.readers.length;
    
    const result = await reader.query(query, params);
    return result.rows;
  }
  
  // Transaction always uses writer
  async transaction<T>(fn: (client: any) => Promise<T>): Promise<T> {
    const client = await this.writer.connect();
    
    try {
      await client.query('BEGIN');
      const result = await fn(client);
      await client.query('COMMIT');
      return result;
    } catch (error) {
      await client.query('ROLLBACK');
      throw error;
    } finally {
      client.release();
    }
  }
}

// IAM Authentication
import { RDS } from '@aws-sdk/client-rds';
import { Signer } from '@aws-sdk/rds-signer';

async function getIamAuthToken(config: {
  hostname: string;
  port: number;
  username: string;
  region: string;
}): Promise<string> {
  const signer = new Signer({
    hostname: config.hostname,
    port: config.port,
    username: config.username,
    region: config.region,
  });
  
  return signer.getAuthToken();
}

// Usage with IAM auth (token refreshes automatically)
async function createIamPool(): Promise<Pool> {
  const hostname = process.env.RDS_HOST!;
  const port = parseInt(process.env.RDS_PORT || '5432');
  const username = process.env.RDS_USER!;
  const region = process.env.AWS_REGION!;
  
  return new Pool({
    host: hostname,
    port: port,
    database: process.env.RDS_DATABASE,
    user: username,
    password: async () => {
      // Token is valid for 15 minutes
      return getIamAuthToken({ hostname, port, username, region });
    },
    ssl: { rejectUnauthorized: true },
  });
}
```

### DynamoDB - NoSQL at Scale

```typescript
// databases/dynamodb.ts
import {
  DynamoDBClient,
  QueryCommand,
  GetItemCommand,
  PutItemCommand,
  UpdateItemCommand,
  DeleteItemCommand,
  TransactWriteItemsCommand,
  BatchGetItemCommand,
  BatchWriteItemCommand,
} from '@aws-sdk/client-dynamodb';
import {
  DynamoDBDocumentClient,
  QueryCommandInput,
} from '@aws-sdk/lib-dynamodb';
import { marshall, unmarshall } from '@aws-sdk/util-dynamodb';

const client = new DynamoDBClient({
  region: process.env.AWS_REGION,
});

const docClient = DynamoDBDocumentClient.from(client, {
  marshallOptions: {
    removeUndefinedValues: true,
  },
});

/**
 * Single-Table Design Pattern
 * 
 * Table: MyApp
 * PK: Partition key (e.g., USER#123, ORDER#456)
 * SK: Sort key (e.g., PROFILE, ORDER#2024-01-01#abc)
 * GSI1PK: For alternative access patterns
 * GSI1SK: For alternative access patterns
 */

const TABLE_NAME = process.env.DYNAMODB_TABLE!;

// Entity types
interface BaseEntity {
  pk: string;
  sk: string;
  entityType: string;
  createdAt: string;
  updatedAt: string;
}

interface User extends BaseEntity {
  userId: string;
  email: string;
  name: string;
  gsi1pk: string; // EMAIL#user@example.com
  gsi1sk: string;
}

interface Order extends BaseEntity {
  orderId: string;
  userId: string;
  status: string;
  total: number;
  items: Array<{ productId: string; quantity: number; price: number }>;
  gsi1pk: string; // USER#123
  gsi1sk: string; // ORDER#2024-01-01
}

// Repository pattern
class DynamoRepository<T extends BaseEntity> {
  constructor(
    private entityType: string,
    private pkPrefix: string,
    private skPrefix: string
  ) {}
  
  async create(entity: Omit<T, 'pk' | 'sk' | 'createdAt' | 'updatedAt' | 'entityType'>): Promise<T> {
    const now = new Date().toISOString();
    const id = (entity as any)[`${this.entityType.toLowerCase()}Id`];
    
    const item: T = {
      ...entity,
      pk: `${this.pkPrefix}#${id}`,
      sk: this.skPrefix,
      entityType: this.entityType,
      createdAt: now,
      updatedAt: now,
    } as T;
    
    await docClient.send(new PutItemCommand({
      TableName: TABLE_NAME,
      Item: marshall(item),
      ConditionExpression: 'attribute_not_exists(pk)',
    }));
    
    return item;
  }
  
  async get(id: string): Promise<T | null> {
    const response = await docClient.send(new GetItemCommand({
      TableName: TABLE_NAME,
      Key: marshall({
        pk: `${this.pkPrefix}#${id}`,
        sk: this.skPrefix,
      }),
    }));
    
    if (!response.Item) return null;
    return unmarshall(response.Item) as T;
  }
  
  async update(id: string, updates: Partial<T>): Promise<T> {
    const updateExpressions: string[] = ['#updatedAt = :updatedAt'];
    const names: Record<string, string> = { '#updatedAt': 'updatedAt' };
    const values: Record<string, any> = { ':updatedAt': new Date().toISOString() };
    
    Object.entries(updates).forEach(([key, value], i) => {
      if (['pk', 'sk', 'entityType', 'createdAt'].includes(key)) return;
      updateExpressions.push(`#field${i} = :value${i}`);
      names[`#field${i}`] = key;
      values[`:value${i}`] = value;
    });
    
    const response = await docClient.send(new UpdateItemCommand({
      TableName: TABLE_NAME,
      Key: marshall({
        pk: `${this.pkPrefix}#${id}`,
        sk: this.skPrefix,
      }),
      UpdateExpression: `SET ${updateExpressions.join(', ')}`,
      ExpressionAttributeNames: names,
      ExpressionAttributeValues: marshall(values),
      ReturnValues: 'ALL_NEW',
    }));
    
    return unmarshall(response.Attributes!) as T;
  }
  
  async delete(id: string): Promise<void> {
    await docClient.send(new DeleteItemCommand({
      TableName: TABLE_NAME,
      Key: marshall({
        pk: `${this.pkPrefix}#${id}`,
        sk: this.skPrefix,
      }),
    }));
  }
}

// Query patterns
class OrderRepository {
  // Get orders for a user (using GSI)
  async getByUser(userId: string, limit: number = 20): Promise<Order[]> {
    const response = await docClient.send(new QueryCommand({
      TableName: TABLE_NAME,
      IndexName: 'gsi1',
      KeyConditionExpression: 'gsi1pk = :pk',
      ExpressionAttributeValues: marshall({
        ':pk': `USER#${userId}`,
      }),
      ScanIndexForward: false, // Descending (newest first)
      Limit: limit,
    }));
    
    return (response.Items || []).map(item => unmarshall(item) as Order);
  }
  
  // Get orders in date range
  async getByDateRange(
    userId: string,
    startDate: string,
    endDate: string
  ): Promise<Order[]> {
    const response = await docClient.send(new QueryCommand({
      TableName: TABLE_NAME,
      IndexName: 'gsi1',
      KeyConditionExpression: 'gsi1pk = :pk AND gsi1sk BETWEEN :start AND :end',
      ExpressionAttributeValues: marshall({
        ':pk': `USER#${userId}`,
        ':start': `ORDER#${startDate}`,
        ':end': `ORDER#${endDate}`,
      }),
    }));
    
    return (response.Items || []).map(item => unmarshall(item) as Order);
  }
}
```

### Aurora - Cloud-Native Database

```typescript
// databases/aurora.ts

/**
 * Aurora benefits over RDS:
 * - 5x performance of MySQL, 3x of PostgreSQL
 * - Auto-scaling storage (10GB to 128TB)
 * - Up to 15 read replicas (vs 5 for RDS)
 * - Global database (cross-region replication)
 * - Serverless option (auto-scaling compute)
 */

import { Pool } from 'pg';

// Aurora PostgreSQL with read scaling
class AuroraCluster {
  private writer: Pool;
  private reader: Pool;
  
  constructor() {
    // Writer endpoint - all writes and read-after-write consistency
    this.writer = new Pool({
      host: process.env.AURORA_WRITER_ENDPOINT,
      port: 5432,
      database: process.env.DB_NAME,
      user: process.env.DB_USER,
      password: process.env.DB_PASSWORD,
      max: 20,
      ssl: { rejectUnauthorized: true },
    });
    
    // Reader endpoint - load balanced across replicas
    this.reader = new Pool({
      host: process.env.AURORA_READER_ENDPOINT,
      port: 5432,
      database: process.env.DB_NAME,
      user: process.env.DB_USER,
      password: process.env.DB_PASSWORD,
      max: 50, // More connections for reads
      ssl: { rejectUnauthorized: true },
    });
  }
  
  // Write operations
  async execute(query: string, params?: any[]): Promise<any> {
    return this.writer.query(query, params);
  }
  
  // Read operations (eventually consistent, uses replicas)
  async query(query: string, params?: any[]): Promise<any> {
    return this.reader.query(query, params);
  }
  
  // Read after write (uses writer for consistency)
  async readAfterWrite(query: string, params?: any[]): Promise<any> {
    return this.writer.query(query, params);
  }
}

// Aurora Serverless v2 configuration
const auroraServerlessConfig = `
resource "aws_rds_cluster" "aurora_serverless" {
  cluster_identifier = "my-aurora-cluster"
  engine             = "aurora-postgresql"
  engine_mode        = "provisioned"
  engine_version     = "14.6"
  database_name      = "mydb"
  master_username    = "admin"
  master_password    = var.db_password

  serverlessv2_scaling_configuration {
    min_capacity = 0.5  # 0.5 ACU minimum (cost savings)
    max_capacity = 16   # Scale up to 16 ACU
  }

  storage_encrypted = true
  
  vpc_security_group_ids = [aws_security_group.aurora.id]
  db_subnet_group_name   = aws_db_subnet_group.aurora.name
}

resource "aws_rds_cluster_instance" "aurora_serverless" {
  cluster_identifier   = aws_rds_cluster.aurora_serverless.id
  instance_class       = "db.serverless"
  engine               = aws_rds_cluster.aurora_serverless.engine
  engine_version       = aws_rds_cluster.aurora_serverless.engine_version
  publicly_accessible  = false
}
`;
```

---

## Real-World Scenarios

### Scenario 1: Multi-Region with DynamoDB Global Tables

```typescript
// databases/global-tables.ts

/**
 * DynamoDB Global Tables:
 * - Active-active replication across regions
 * - Sub-second replication latency
 * - Automatic conflict resolution (last-writer-wins)
 * - Great for global applications
 */

import { DynamoDBClient } from '@aws-sdk/client-dynamodb';

// Route to nearest region
function getDynamoClient(): DynamoDBClient {
  // Determine region based on latency or user location
  const region = determineClosestRegion();
  
  return new DynamoDBClient({ region });
}

function determineClosestRegion(): string {
  // In practice, use CloudFront or Route 53 latency routing
  // Or measure RTT to each region
  const regions = ['us-east-1', 'eu-west-1', 'ap-southeast-1'];
  
  // For Lambda, use the region where the function runs
  return process.env.AWS_REGION || 'us-east-1';
}

// Handling conflicts with conditional writes
async function updateWithConflictResolution(
  client: DynamoDBClient,
  key: { pk: string; sk: string },
  updates: Record<string, any>,
  expectedVersion: number
): Promise<boolean> {
  try {
    await client.send(new UpdateItemCommand({
      TableName: TABLE_NAME,
      Key: marshall(key),
      UpdateExpression: 'SET #data = :data, #version = :newVersion',
      ConditionExpression: '#version = :expectedVersion',
      ExpressionAttributeNames: {
        '#data': 'data',
        '#version': 'version',
      },
      ExpressionAttributeValues: marshall({
        ':data': updates,
        ':newVersion': expectedVersion + 1,
        ':expectedVersion': expectedVersion,
      }),
    }));
    return true;
  } catch (error: any) {
    if (error.name === 'ConditionalCheckFailedException') {
      // Version conflict - another write happened
      return false;
    }
    throw error;
  }
}

import { UpdateItemCommand } from '@aws-sdk/client-dynamodb';
import { marshall } from '@aws-sdk/util-dynamodb';
const TABLE_NAME = '';
```

### Scenario 2: Read-Heavy Workload with Caching

```typescript
// databases/caching-layer.ts
import Redis from 'ioredis';
import { Pool } from 'pg';

/**
 * Pattern: Cache-Aside with read replicas
 * 
 * 1. Check cache first
 * 2. If miss, read from replica
 * 3. Populate cache
 * 4. Invalidate on writes
 */

class CachedDatabaseRepository<T> {
  private cache: Redis;
  private reader: Pool;
  private writer: Pool;
  private ttl: number;
  
  constructor(
    cache: Redis,
    reader: Pool,
    writer: Pool,
    ttl: number = 300 // 5 minutes default
  ) {
    this.cache = cache;
    this.reader = reader;
    this.writer = writer;
    this.ttl = ttl;
  }
  
  async get(id: string, query: string): Promise<T | null> {
    const cacheKey = `entity:${id}`;
    
    // 1. Check cache
    const cached = await this.cache.get(cacheKey);
    if (cached) {
      return JSON.parse(cached);
    }
    
    // 2. Read from replica
    const result = await this.reader.query(query, [id]);
    if (result.rows.length === 0) return null;
    
    const entity = result.rows[0] as T;
    
    // 3. Populate cache
    await this.cache.setex(cacheKey, this.ttl, JSON.stringify(entity));
    
    return entity;
  }
  
  async update(id: string, query: string, params: any[]): Promise<T> {
    // Write to primary
    const result = await this.writer.query(query, params);
    const entity = result.rows[0] as T;
    
    // Invalidate cache
    await this.cache.del(`entity:${id}`);
    
    return entity;
  }
  
  async invalidate(id: string): Promise<void> {
    await this.cache.del(`entity:${id}`);
  }
  
  async warmCache(ids: string[], query: string): Promise<void> {
    // Batch load for cache warming
    const result = await this.reader.query(query, [ids]);
    
    const pipeline = this.cache.pipeline();
    for (const row of result.rows) {
      const key = `entity:${(row as any).id}`;
      pipeline.setex(key, this.ttl, JSON.stringify(row));
    }
    
    await pipeline.exec();
  }
}

// Usage
const userRepo = new CachedDatabaseRepository<{ id: string; name: string }>(
  new Redis(process.env.REDIS_URL!),
  readerPool,
  writerPool,
  600 // 10 minute cache
);

const readerPool = new Pool({});
const writerPool = new Pool({});
```

---

## Common Pitfalls

### 1. Opening Too Many Connections

```typescript
// ❌ BAD: New connection per request
async function handleRequest(req: Request) {
  const client = new Client(config);
  await client.connect();
  // ... query ...
  await client.end();
}

// ✅ GOOD: Connection pooling
const pool = new Pool({ max: 20 });

async function handleRequest(req: Request) {
  const client = await pool.connect();
  try {
    // ... query ...
  } finally {
    client.release();
  }
}

import { Client, Pool } from 'pg';
type Request = any;
const config = {};
```

### 2. Not Using Read Replicas

```typescript
// ❌ BAD: All reads hit primary
const result = await primaryPool.query('SELECT * FROM users WHERE ...');

// ✅ GOOD: Route reads to replicas
const result = await replicaPool.query('SELECT * FROM users WHERE ...');
// Use primary only for writes and read-after-write

const primaryPool = new Pool({});
const replicaPool = new Pool({});
```

### 3. DynamoDB Hot Partitions

```typescript
// ❌ BAD: Sequential timestamps cause hot partition
const item = {
  pk: 'LOGS',  // All logs in one partition!
  sk: new Date().toISOString(),
};

// ✅ GOOD: Distribute with sharding
const shard = Math.floor(Math.random() * 10);
const item = {
  pk: `LOGS#${shard}`,  // Spread across 10 partitions
  sk: new Date().toISOString(),
};
```

---

## Interview Questions

### Q1: When would you choose Aurora over RDS?

**A:** Aurora for: higher performance needs (5x MySQL), auto-scaling storage, more read replicas (15 vs 5), global distribution, serverless workloads. RDS for: lower cost for small workloads, specific engine features not in Aurora, simpler setup. Aurora costs ~20% more but provides significant performance and scalability benefits.

### Q2: How do you design a DynamoDB table?

**A:** Start with access patterns, not entities. Design keys to support queries without scans. Use composite keys (pk + sk) for one-to-many. Use GSIs for alternative access patterns. Denormalize - store data together that's queried together. Use single-table design for related entities to enable transactions.

### Q3: How do you handle database migrations in the cloud?

**A:** Use migration tools (Flyway, Liquibase). Test in staging first. For zero-downtime: 1) Deploy code that works with old and new schema. 2) Run migration. 3) Deploy code for new schema only. For large tables, consider pt-online-schema-change or Aurora's parallel query. Always have rollback plan.

### Q4: When would you choose DynamoDB over Aurora?

**A:** **DynamoDB** for: extreme scale (millions of requests/sec), predictable single-digit ms latency, simple access patterns, serverless/event-driven apps. **Aurora** for: complex queries, joins, transactions across entities, ad-hoc reporting, existing SQL skills. Many apps use both: Aurora for core data, DynamoDB for sessions/caching.

---

## Quick Reference Checklist

### RDS/Aurora
- [ ] Enable Multi-AZ for production
- [ ] Set up read replicas for scale
- [ ] Configure automated backups
- [ ] Use connection pooling
- [ ] Enable Performance Insights

### DynamoDB
- [ ] Design for access patterns
- [ ] Set appropriate capacity mode
- [ ] Configure GSIs as needed
- [ ] Enable point-in-time recovery
- [ ] Monitor throttling metrics

### General
- [ ] Use IAM authentication when possible
- [ ] Enable encryption at rest
- [ ] Configure VPC security groups
- [ ] Set up monitoring and alerting
- [ ] Test disaster recovery

---

*Last updated: February 2026*

