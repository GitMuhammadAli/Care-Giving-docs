# CDC (Change Data Capture) - Complete Guide

> **MUST REMEMBER**: CDC captures database changes (INSERT/UPDATE/DELETE) in real-time. Tools: Debezium (open source, Kafka Connect), AWS DMS, Oracle GoldenGate. Sources: database transaction logs (binlog, WAL). Use cases: sync databases, feed data lakes, event sourcing, cache invalidation. Key concepts: log-based (reads DB logs) vs query-based (polling), exactly-once delivery, schema evolution, initial snapshot.

---

## How to Explain Like a Senior Developer

"Change Data Capture is reading the database's transaction log to capture every change in real-time. Instead of polling with SELECT queries (which misses changes and puts load on the DB), CDC reads the same log the database uses for replication. Debezium is the most popular tool - it connects to MySQL's binlog or PostgreSQL's WAL and streams changes to Kafka. Each change becomes an event with before/after values, operation type, and metadata. Use cases: keeping Elasticsearch in sync with your database, feeding a data lake with real-time changes, invalidating caches when data changes, building event-driven systems. The main challenges are: handling the initial snapshot (existing data before CDC starts), managing schema changes, and ensuring exactly-once delivery to downstream systems."

---

## Core Implementation

### Debezium with Kafka Connect

```json
// cdc/debezium-postgres-connector.json

{
  "name": "orders-connector",
  "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "database.hostname": "postgres",
    "database.port": "5432",
    "database.user": "debezium",
    "database.password": "${file:/secrets/postgres-password}",
    "database.dbname": "orders_db",
    "database.server.name": "orders",
    
    "table.include.list": "public.orders,public.order_items,public.customers",
    
    "plugin.name": "pgoutput",
    "publication.name": "dbz_publication",
    "slot.name": "debezium_slot",
    
    "topic.prefix": "cdc",
    
    "transforms": "route,unwrap",
    "transforms.route.type": "org.apache.kafka.connect.transforms.RegexRouter",
    "transforms.route.regex": "([^.]+)\\.([^.]+)\\.([^.]+)",
    "transforms.route.replacement": "$1.$3",
    
    "transforms.unwrap.type": "io.debezium.transforms.ExtractNewRecordState",
    "transforms.unwrap.add.fields": "op,source.ts_ms",
    "transforms.unwrap.delete.handling.mode": "rewrite",
    
    "key.converter": "org.apache.kafka.connect.json.JsonConverter",
    "value.converter": "org.apache.kafka.connect.json.JsonConverter",
    "key.converter.schemas.enable": false,
    "value.converter.schemas.enable": false,
    
    "snapshot.mode": "initial",
    "snapshot.locking.mode": "none",
    
    "heartbeat.interval.ms": 10000,
    "heartbeat.action.query": "UPDATE debezium_heartbeat SET last_heartbeat = NOW()"
  }
}
```

```yaml
# cdc/docker-compose.yml

version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: orders_db
    command:
      - "postgres"
      - "-c"
      - "wal_level=logical"
      - "-c"
      - "max_replication_slots=4"
      - "-c"
      - "max_wal_senders=4"
    ports:
      - "5432:5432"

  kafka:
    image: confluentinc/cp-kafka:7.5.0
    environment:
      KAFKA_NODE_ID: 1
      KAFKA_PROCESS_ROLES: broker,controller
      KAFKA_LISTENERS: PLAINTEXT://kafka:9092,CONTROLLER://kafka:9093
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@kafka:9093
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      CLUSTER_ID: MkU3OEVBNTcwNTJENDM2Qk
    ports:
      - "9092:9092"

  connect:
    image: debezium/connect:2.4
    environment:
      BOOTSTRAP_SERVERS: kafka:9092
      GROUP_ID: connect-cluster
      CONFIG_STORAGE_TOPIC: connect-configs
      OFFSET_STORAGE_TOPIC: connect-offsets
      STATUS_STORAGE_TOPIC: connect-status
      KEY_CONVERTER: org.apache.kafka.connect.json.JsonConverter
      VALUE_CONVERTER: org.apache.kafka.connect.json.JsonConverter
    ports:
      - "8083:8083"
    depends_on:
      - kafka
      - postgres
```

### CDC Event Consumer

```typescript
// cdc/cdc-consumer.ts

import { Kafka, EachMessagePayload } from 'kafkajs';

/**
 * Debezium change event structure
 */
interface DebeziumEvent<T = any> {
  before: T | null;
  after: T | null;
  source: {
    version: string;
    connector: string;
    name: string;
    ts_ms: number;
    snapshot: string;
    db: string;
    schema: string;
    table: string;
    txId: number;
    lsn: number;
  };
  op: 'c' | 'u' | 'd' | 'r';  // create, update, delete, read (snapshot)
  ts_ms: number;
}

/**
 * Simplified event after ExtractNewRecordState transform
 */
interface SimplifiedEvent<T = any> {
  payload: T;
  __op: 'c' | 'u' | 'd' | 'r';
  __source_ts_ms: number;
  __deleted: boolean;
}

interface Order {
  id: number;
  customer_id: number;
  total_amount: number;
  status: string;
  created_at: string;
  updated_at: string;
}

/**
 * CDC consumer that processes database changes
 */
class CDCConsumer {
  private kafka: Kafka;
  private handlers: Map<string, (event: DebeziumEvent) => Promise<void>> = new Map();
  
  constructor(brokers: string[]) {
    this.kafka = new Kafka({
      clientId: 'cdc-consumer',
      brokers,
    });
  }
  
  /**
   * Register handler for a table's changes
   */
  onTable(table: string, handler: (event: DebeziumEvent) => Promise<void>): this {
    this.handlers.set(table, handler);
    return this;
  }
  
  async start(): Promise<void> {
    const consumer = this.kafka.consumer({ groupId: 'cdc-processors' });
    
    await consumer.connect();
    
    // Subscribe to all CDC topics
    const topics = Array.from(this.handlers.keys()).map(table => `cdc.${table}`);
    await consumer.subscribe({ topics, fromBeginning: false });
    
    await consumer.run({
      eachMessage: async (payload: EachMessagePayload) => {
        const { topic, message } = payload;
        
        if (!message.value) return;
        
        const event: DebeziumEvent = JSON.parse(message.value.toString());
        const table = topic.replace('cdc.', '');
        
        const handler = this.handlers.get(table);
        if (handler) {
          await handler(event);
        }
      },
    });
  }
}

/**
 * Example: Sync to Elasticsearch
 */
class ElasticsearchSync {
  private esClient: any;  // Elasticsearch client
  
  constructor(esClient: any) {
    this.esClient = esClient;
  }
  
  async handleOrderChange(event: DebeziumEvent<Order>): Promise<void> {
    const { op, before, after } = event;
    
    switch (op) {
      case 'c':  // Create
      case 'r':  // Read (snapshot)
        await this.esClient.index({
          index: 'orders',
          id: after!.id.toString(),
          body: after,
        });
        console.log(`Indexed order ${after!.id}`);
        break;
      
      case 'u':  // Update
        await this.esClient.update({
          index: 'orders',
          id: after!.id.toString(),
          body: { doc: after },
        });
        console.log(`Updated order ${after!.id}`);
        break;
      
      case 'd':  // Delete
        await this.esClient.delete({
          index: 'orders',
          id: before!.id.toString(),
        });
        console.log(`Deleted order ${before!.id}`);
        break;
    }
  }
}

/**
 * Example: Cache invalidation
 */
class CacheInvalidator {
  private redis: any;  // Redis client
  
  constructor(redisClient: any) {
    this.redis = redisClient;
  }
  
  async handleCustomerChange(event: DebeziumEvent): Promise<void> {
    const customerId = (event.after || event.before)?.id;
    
    // Invalidate all cache keys related to this customer
    const pattern = `customer:${customerId}:*`;
    const keys = await this.redis.keys(pattern);
    
    if (keys.length > 0) {
      await this.redis.del(...keys);
      console.log(`Invalidated ${keys.length} cache keys for customer ${customerId}`);
    }
  }
}

// Usage
async function main() {
  const esSync = new ElasticsearchSync(/* es client */);
  const cacheInvalidator = new CacheInvalidator(/* redis client */);
  
  const consumer = new CDCConsumer(['localhost:9092'])
    .onTable('orders', (e) => esSync.handleOrderChange(e))
    .onTable('customers', (e) => cacheInvalidator.handleCustomerChange(e));
  
  await consumer.start();
}
```

### Custom CDC with PostgreSQL Logical Replication

```typescript
// cdc/postgres-logical-replication.ts

import { Client } from 'pg';
import { LogicalReplicationService, PgoutputPlugin } from 'pg-logical-replication';

/**
 * Direct PostgreSQL logical replication (without Kafka)
 * Useful for simpler setups or when Kafka is overkill
 */
class PostgresCDC {
  private client: Client;
  private service: LogicalReplicationService;
  
  constructor(connectionString: string) {
    this.client = new Client(connectionString);
    this.service = new LogicalReplicationService(
      { connectionString },
      { acknowledge: { auto: true, timeoutSeconds: 10 } }
    );
  }
  
  async setup(): Promise<void> {
    await this.client.connect();
    
    // Create publication (if not exists)
    await this.client.query(`
      DO $$
      BEGIN
        IF NOT EXISTS (SELECT 1 FROM pg_publication WHERE pubname = 'cdc_publication') THEN
          CREATE PUBLICATION cdc_publication FOR ALL TABLES;
        END IF;
      END
      $$;
    `);
    
    // Create replication slot (if not exists)
    try {
      await this.client.query(`
        SELECT pg_create_logical_replication_slot('cdc_slot', 'pgoutput');
      `);
    } catch (e: any) {
      if (!e.message.includes('already exists')) throw e;
    }
  }
  
  async subscribe(
    onInsert: (table: string, data: any) => Promise<void>,
    onUpdate: (table: string, oldData: any, newData: any) => Promise<void>,
    onDelete: (table: string, data: any) => Promise<void>
  ): Promise<void> {
    const plugin = new PgoutputPlugin({
      protoVersion: 1,
      publicationNames: ['cdc_publication'],
    });
    
    this.service.on('data', async (lsn: string, log: any) => {
      if (log.tag === 'insert') {
        await onInsert(log.relation.name, this.parseRow(log.new));
      } else if (log.tag === 'update') {
        await onUpdate(
          log.relation.name,
          log.old ? this.parseRow(log.old) : null,
          this.parseRow(log.new)
        );
      } else if (log.tag === 'delete') {
        await onDelete(log.relation.name, this.parseRow(log.old));
      }
    });
    
    this.service.on('error', (err: Error) => {
      console.error('Replication error:', err);
    });
    
    await this.service.subscribe(plugin, 'cdc_slot');
  }
  
  private parseRow(row: any[]): any {
    // Convert array of {name, value} to object
    return row.reduce((obj, col) => {
      obj[col.name] = col.value;
      return obj;
    }, {});
  }
  
  async stop(): Promise<void> {
    await this.service.stop();
    await this.client.end();
  }
}

// Usage
async function directCDC() {
  const cdc = new PostgresCDC('postgresql://user:pass@localhost/db');
  await cdc.setup();
  
  await cdc.subscribe(
    async (table, data) => {
      console.log(`INSERT on ${table}:`, data);
    },
    async (table, oldData, newData) => {
      console.log(`UPDATE on ${table}:`, { old: oldData, new: newData });
    },
    async (table, data) => {
      console.log(`DELETE on ${table}:`, data);
    }
  );
}
```

---

## Real-World Scenarios

### Scenario 1: Database to Data Lake Pipeline

```typescript
// cdc/db-to-lake.ts

import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3';
import { Kafka } from 'kafkajs';

/**
 * Stream database changes to data lake (S3)
 * Micro-batch for efficiency
 */
class DatabaseToLakePipeline {
  private kafka: Kafka;
  private s3: S3Client;
  private buffer: Map<string, any[]> = new Map();
  private bufferSize = 10000;
  private flushInterval = 60000; // 1 minute
  
  constructor(kafkaBrokers: string[], s3Region: string) {
    this.kafka = new Kafka({ brokers: kafkaBrokers, clientId: 'db-to-lake' });
    this.s3 = new S3Client({ region: s3Region });
  }
  
  async start(tables: string[]): Promise<void> {
    const consumer = this.kafka.consumer({ groupId: 'lake-writer' });
    await consumer.connect();
    
    const topics = tables.map(t => `cdc.${t}`);
    await consumer.subscribe({ topics });
    
    // Periodic flush
    setInterval(() => this.flushAll(), this.flushInterval);
    
    await consumer.run({
      eachMessage: async ({ topic, message }) => {
        if (!message.value) return;
        
        const event = JSON.parse(message.value.toString());
        const table = topic.replace('cdc.', '');
        
        // Buffer events
        if (!this.buffer.has(table)) {
          this.buffer.set(table, []);
        }
        
        this.buffer.get(table)!.push(this.transformEvent(event));
        
        // Flush if buffer full
        if (this.buffer.get(table)!.length >= this.bufferSize) {
          await this.flush(table);
        }
      },
    });
  }
  
  private transformEvent(event: any): any {
    // Transform Debezium event to lake format
    return {
      ...event.after || event.before,
      _cdc_operation: event.op,
      _cdc_timestamp: event.ts_ms,
      _cdc_source: event.source.table,
    };
  }
  
  private async flush(table: string): Promise<void> {
    const events = this.buffer.get(table);
    if (!events || events.length === 0) return;
    
    // Write as newline-delimited JSON
    const content = events.map(e => JSON.stringify(e)).join('\n');
    const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
    const key = `cdc/${table}/year=${new Date().getFullYear()}/month=${String(new Date().getMonth() + 1).padStart(2, '0')}/${table}-${timestamp}.json`;
    
    await this.s3.send(new PutObjectCommand({
      Bucket: 'data-lake-bucket',
      Key: key,
      Body: content,
      ContentType: 'application/x-ndjson',
    }));
    
    console.log(`Flushed ${events.length} events to s3://${key}`);
    
    // Clear buffer
    this.buffer.set(table, []);
  }
  
  private async flushAll(): Promise<void> {
    for (const table of this.buffer.keys()) {
      await this.flush(table);
    }
  }
}
```

### Scenario 2: Real-time Materialized View

```typescript
// cdc/materialized-view.ts

import { Kafka } from 'kafkajs';
import { Pool } from 'pg';

/**
 * Maintain denormalized view in real-time using CDC
 * Example: Order with customer details and item count
 */
interface OrderView {
  orderId: number;
  orderTotal: number;
  orderStatus: string;
  customerName: string;
  customerEmail: string;
  itemCount: number;
  lastUpdated: Date;
}

class RealtimeMaterializedView {
  private kafka: Kafka;
  private targetDb: Pool;
  
  constructor(kafkaBrokers: string[], targetDbUrl: string) {
    this.kafka = new Kafka({ brokers: kafkaBrokers, clientId: 'materialized-view' });
    this.targetDb = new Pool({ connectionString: targetDbUrl });
  }
  
  async start(): Promise<void> {
    const consumer = this.kafka.consumer({ groupId: 'order-view-builder' });
    await consumer.connect();
    
    // Subscribe to all relevant tables
    await consumer.subscribe({
      topics: ['cdc.orders', 'cdc.customers', 'cdc.order_items'],
    });
    
    await consumer.run({
      eachMessage: async ({ topic, message }) => {
        if (!message.value) return;
        
        const event = JSON.parse(message.value.toString());
        const table = topic.replace('cdc.', '');
        
        switch (table) {
          case 'orders':
            await this.handleOrderChange(event);
            break;
          case 'customers':
            await this.handleCustomerChange(event);
            break;
          case 'order_items':
            await this.handleOrderItemChange(event);
            break;
        }
      },
    });
  }
  
  private async handleOrderChange(event: any): Promise<void> {
    const order = event.after || event.before;
    
    if (event.op === 'd') {
      // Delete from view
      await this.targetDb.query(
        'DELETE FROM order_view WHERE order_id = $1',
        [order.id]
      );
    } else {
      // Upsert order fields
      await this.targetDb.query(`
        INSERT INTO order_view (order_id, order_total, order_status, last_updated)
        VALUES ($1, $2, $3, NOW())
        ON CONFLICT (order_id) DO UPDATE SET
          order_total = EXCLUDED.order_total,
          order_status = EXCLUDED.order_status,
          last_updated = NOW()
      `, [order.id, order.total_amount, order.status]);
    }
  }
  
  private async handleCustomerChange(event: any): Promise<void> {
    const customer = event.after || event.before;
    
    // Update all orders for this customer
    await this.targetDb.query(`
      UPDATE order_view
      SET customer_name = $1, customer_email = $2, last_updated = NOW()
      WHERE order_id IN (
        SELECT id FROM orders WHERE customer_id = $3
      )
    `, [customer.name, customer.email, customer.id]);
  }
  
  private async handleOrderItemChange(event: any): Promise<void> {
    const item = event.after || event.before;
    
    // Recalculate item count for the order
    const result = await this.targetDb.query(
      'SELECT COUNT(*) as count FROM order_items WHERE order_id = $1',
      [item.order_id]
    );
    
    await this.targetDb.query(`
      UPDATE order_view
      SET item_count = $1, last_updated = NOW()
      WHERE order_id = $2
    `, [result.rows[0].count, item.order_id]);
  }
}
```

---

## Common Pitfalls

### 1. Not Handling Initial Snapshot

```typescript
// ❌ BAD: Start CDC without existing data
// New CDC captures changes but misses existing records

// ✅ GOOD: Handle snapshot mode
const event = JSON.parse(message.value.toString());

if (event.op === 'r') {
  // Snapshot record - treat as initial load
  await handleInitialLoad(event.after);
} else {
  // Regular change
  await handleChange(event);
}

// Or use Debezium snapshot.mode: initial
```

### 2. Ignoring Schema Changes

```typescript
// ❌ BAD: Assume schema never changes
const name = event.after.name;  // Breaks if column renamed

// ✅ GOOD: Handle schema evolution
// 1. Use Debezium schema history topic
// 2. Subscribe to schema changes
// 3. Version your event handlers

interface SchemaVersion {
  version: number;
  columns: string[];
}

const handlers: Record<number, (event: any) => void> = {
  1: (e) => handleV1(e),
  2: (e) => handleV2(e),  // New schema
};
```

### 3. Not Idempotent Processing

```typescript
// ❌ BAD: Duplicate processing causes duplicates
await db.insert(event.after);  // Duplicate if replayed!

// ✅ GOOD: Idempotent operations
await db.query(`
  INSERT INTO orders (id, ...)
  VALUES ($1, ...)
  ON CONFLICT (id) DO UPDATE SET
    ... = EXCLUDED....
`, [event.after.id, ...]);

// Or track processed LSN
await db.query(`
  INSERT INTO processed_events (lsn, processed_at)
  VALUES ($1, NOW())
  ON CONFLICT DO NOTHING
`, [event.source.lsn]);
```

---

## Interview Questions

### Q1: What is the difference between log-based and query-based CDC?

**A:** **Log-based CDC** reads the database's transaction log (binlog, WAL) - captures all changes, no missed events, minimal DB impact. **Query-based CDC** polls with SELECT queries using timestamps - simpler setup but can miss rapid changes, requires timestamp columns, puts load on DB. Log-based is preferred for most use cases.

### Q2: How does Debezium handle the initial snapshot?

**A:** Debezium starts with a snapshot of existing data before streaming changes. Modes: **initial** (snapshot then stream), **schema_only** (schema only, no data), **never** (skip snapshot), **when_needed** (snapshot if slot lost). During snapshot, it reads tables in consistent order while still tracking new changes. Snapshot records have `op: r` (read).

### Q3: What happens if the CDC consumer falls behind?

**A:** If consumer lag grows: 1) Database may run out of WAL space (PostgreSQL) or binlog retention (MySQL). 2) Replication slot prevents WAL cleanup. 3) Need to either catch up or re-snapshot. Mitigations: monitor consumer lag, set appropriate retention, scale consumers horizontally, use backpressure.

### Q4: How do you handle CDC for multiple downstream systems?

**A:** Best practice: CDC to Kafka, then multiple consumers. Each consumer group processes independently. Benefits: 1) Single connection to source DB. 2) Consumers can replay from Kafka. 3) Different processing speeds OK. 4) Add new consumers without affecting source. Alternative: fan-out at CDC level but less flexible.

---

## Quick Reference Checklist

### Setup
- [ ] Enable logical replication on source DB
- [ ] Create replication slot/publication
- [ ] Plan for initial snapshot
- [ ] Set up heartbeat for idle tables

### Processing
- [ ] Handle all operation types (c/u/d/r)
- [ ] Make handlers idempotent
- [ ] Plan for schema evolution
- [ ] Monitor consumer lag

### Operations
- [ ] Monitor replication slot
- [ ] Set retention policies
- [ ] Handle connector failures
- [ ] Test disaster recovery

---

*Last updated: February 2026*

