# Data Serialization - Complete Guide

> **MUST REMEMBER**: Serialization converts data to bytes for storage/transmission. JSON: human-readable, flexible, verbose. Protobuf: compact, fast, schema required, used in gRPC. Avro: schema evolution, used in Kafka/Hadoop. MessagePack: like JSON but binary. Choose based on: human readability needs, size constraints, schema evolution requirements, tooling ecosystem. Binary formats (Protobuf, Avro) are 2-10x smaller and faster than JSON.

---

## How to Explain Like a Senior Developer

"Serialization is encoding data into bytes so it can be stored or transmitted. JSON is the default for APIs - human-readable, universal support, but verbose and slow. When performance matters (high throughput, large payloads, mobile bandwidth), use binary formats. Protocol Buffers (Protobuf) is Google's format - very compact, extremely fast, requires schema definitions (.proto files), used in gRPC. Avro is popular in data pipelines (Kafka, Hadoop) because it handles schema evolution well - you can add/remove fields without breaking consumers. MessagePack is JSON-compatible but binary, good middle ground. The choice depends on: do humans need to read it? How important is size? Do you need schema evolution? What tools are you using?"

---

## Core Implementation

### JSON - Universal Standard

```typescript
// data_serialization/json-serialization.ts

/**
 * JSON: Human-readable, universal support
 * Pros: Readable, no schema, universal
 * Cons: Verbose, slow, no types
 */

interface User {
  id: string;
  name: string;
  email: string;
  age: number;
  roles: string[];
  metadata: Record<string, unknown>;
  createdAt: Date;
}

class JsonSerializer {
  /**
   * Standard JSON serialization
   */
  serialize(data: unknown): string {
    return JSON.stringify(data);
  }
  
  deserialize<T>(json: string): T {
    return JSON.parse(json);
  }
  
  /**
   * JSON with custom handling (dates, BigInt, etc.)
   */
  serializeWithReplacer(data: unknown): string {
    return JSON.stringify(data, (key, value) => {
      // Handle Date
      if (value instanceof Date) {
        return { __type: 'Date', value: value.toISOString() };
      }
      // Handle BigInt
      if (typeof value === 'bigint') {
        return { __type: 'BigInt', value: value.toString() };
      }
      // Handle Map
      if (value instanceof Map) {
        return { __type: 'Map', value: Array.from(value.entries()) };
      }
      // Handle Set
      if (value instanceof Set) {
        return { __type: 'Set', value: Array.from(value) };
      }
      return value;
    });
  }
  
  deserializeWithReviver<T>(json: string): T {
    return JSON.parse(json, (key, value) => {
      if (value && typeof value === 'object' && '__type' in value) {
        switch (value.__type) {
          case 'Date':
            return new Date(value.value);
          case 'BigInt':
            return BigInt(value.value);
          case 'Map':
            return new Map(value.value);
          case 'Set':
            return new Set(value.value);
        }
      }
      return value;
    });
  }
  
  /**
   * Streaming JSON for large arrays
   */
  async *streamSerialize(items: AsyncIterable<unknown>): AsyncIterable<string> {
    yield '[';
    let first = true;
    
    for await (const item of items) {
      if (!first) yield ',';
      yield JSON.stringify(item);
      first = false;
    }
    
    yield ']';
  }
}

// JSON Lines (JSONL) - one JSON object per line
class JsonLinesSerializer {
  serialize(items: unknown[]): string {
    return items.map(item => JSON.stringify(item)).join('\n');
  }
  
  *deserialize(jsonl: string): Generator<unknown> {
    for (const line of jsonl.split('\n')) {
      if (line.trim()) {
        yield JSON.parse(line);
      }
    }
  }
}
```

### Protocol Buffers (Protobuf)

```protobuf
// data_serialization/schema.proto

syntax = "proto3";

package myapp;

// Message definitions
message User {
  string id = 1;
  string name = 2;
  string email = 3;
  int32 age = 4;
  repeated string roles = 5;
  map<string, string> metadata = 6;
  int64 created_at = 7;  // Unix timestamp
  
  // Nested message
  Address address = 8;
  
  // Enum
  Status status = 9;
}

message Address {
  string street = 1;
  string city = 2;
  string country = 3;
  string postal_code = 4;
}

enum Status {
  STATUS_UNSPECIFIED = 0;
  STATUS_ACTIVE = 1;
  STATUS_INACTIVE = 2;
  STATUS_SUSPENDED = 3;
}

// For streaming/batching
message UserList {
  repeated User users = 1;
}
```

```typescript
// data_serialization/protobuf-serialization.ts

import * as protobuf from 'protobufjs';

/**
 * Protobuf: Compact, fast, schema required
 * Pros: Very small, very fast, type-safe
 * Cons: Not human-readable, schema required
 */

class ProtobufSerializer {
  private root: protobuf.Root | null = null;
  private userType: protobuf.Type | null = null;
  
  async initialize(protoPath: string): Promise<void> {
    this.root = await protobuf.load(protoPath);
    this.userType = this.root.lookupType('myapp.User');
  }
  
  /**
   * Serialize to binary
   */
  serialize(user: any): Uint8Array {
    if (!this.userType) throw new Error('Not initialized');
    
    // Validate
    const errMsg = this.userType.verify(user);
    if (errMsg) throw new Error(errMsg);
    
    // Create message and encode
    const message = this.userType.create(user);
    return this.userType.encode(message).finish();
  }
  
  /**
   * Deserialize from binary
   */
  deserialize(buffer: Uint8Array): any {
    if (!this.userType) throw new Error('Not initialized');
    
    const message = this.userType.decode(buffer);
    return this.userType.toObject(message, {
      longs: String,    // Convert Long to string
      enums: String,    // Convert enum to string
      bytes: String,    // Convert bytes to base64
      defaults: true,   // Include default values
    });
  }
  
  /**
   * Compare size with JSON
   */
  compareSizes(data: any): { json: number; protobuf: number; ratio: number } {
    const jsonSize = Buffer.byteLength(JSON.stringify(data));
    const protobufSize = this.serialize(data).length;
    
    return {
      json: jsonSize,
      protobuf: protobufSize,
      ratio: jsonSize / protobufSize,
    };
  }
}

// Dynamic protobuf (without .proto file)
class DynamicProtobuf {
  createUserType(): protobuf.Type {
    const User = new protobuf.Type('User')
      .add(new protobuf.Field('id', 1, 'string'))
      .add(new protobuf.Field('name', 2, 'string'))
      .add(new protobuf.Field('email', 3, 'string'))
      .add(new protobuf.Field('age', 4, 'int32'))
      .add(new protobuf.Field('roles', 5, 'string', 'repeated'));
    
    return User;
  }
}

// Usage
async function protobufExample() {
  const serializer = new ProtobufSerializer();
  await serializer.initialize('./schema.proto');
  
  const user = {
    id: '123',
    name: 'John Doe',
    email: 'john@example.com',
    age: 30,
    roles: ['admin', 'user'],
    metadata: { theme: 'dark' },
    createdAt: Date.now(),
    status: 'STATUS_ACTIVE',
  };
  
  const binary = serializer.serialize(user);
  console.log(`Protobuf size: ${binary.length} bytes`);
  
  const decoded = serializer.deserialize(binary);
  console.log('Decoded:', decoded);
  
  const comparison = serializer.compareSizes(user);
  console.log(`JSON: ${comparison.json}B, Protobuf: ${comparison.protobuf}B`);
  console.log(`Protobuf is ${comparison.ratio.toFixed(1)}x smaller`);
}
```

### Apache Avro

```typescript
// data_serialization/avro-serialization.ts

import * as avro from 'avsc';

/**
 * Avro: Schema evolution, popular in data pipelines
 * Pros: Schema evolution, self-describing, Kafka integration
 * Cons: More complex than JSON, requires tooling
 */

// Avro schema definition
const userSchema = {
  type: 'record',
  name: 'User',
  namespace: 'com.example',
  fields: [
    { name: 'id', type: 'string' },
    { name: 'name', type: 'string' },
    { name: 'email', type: 'string' },
    { name: 'age', type: 'int' },
    { name: 'roles', type: { type: 'array', items: 'string' } },
    { 
      name: 'status', 
      type: { 
        type: 'enum', 
        name: 'Status', 
        symbols: ['ACTIVE', 'INACTIVE', 'SUSPENDED'] 
      } 
    },
    // Optional field with default
    { 
      name: 'country', 
      type: ['null', 'string'], 
      default: null 
    },
    { name: 'createdAt', type: 'long' },
  ],
};

// Schema evolution: new version with additional field
const userSchemaV2 = {
  type: 'record',
  name: 'User',
  namespace: 'com.example',
  fields: [
    { name: 'id', type: 'string' },
    { name: 'name', type: 'string' },
    { name: 'email', type: 'string' },
    { name: 'age', type: 'int' },
    { name: 'roles', type: { type: 'array', items: 'string' } },
    { 
      name: 'status', 
      type: { 
        type: 'enum', 
        name: 'Status', 
        symbols: ['ACTIVE', 'INACTIVE', 'SUSPENDED'] 
      } 
    },
    { name: 'country', type: ['null', 'string'], default: null },
    { name: 'createdAt', type: 'long' },
    // New field in V2 - with default for backward compatibility
    { name: 'phoneNumber', type: ['null', 'string'], default: null },
  ],
};

class AvroSerializer {
  private type: avro.Type;
  
  constructor(schema: object) {
    this.type = avro.Type.forSchema(schema);
  }
  
  /**
   * Serialize to binary
   */
  serialize(data: any): Buffer {
    return this.type.toBuffer(data);
  }
  
  /**
   * Deserialize from binary
   */
  deserialize(buffer: Buffer): any {
    return this.type.fromBuffer(buffer);
  }
  
  /**
   * Check if data is valid against schema
   */
  isValid(data: any): boolean {
    return this.type.isValid(data);
  }
  
  /**
   * Get validation errors
   */
  getErrors(data: any): string[] {
    const errors: string[] = [];
    this.type.isValid(data, {
      errorHook: (path, any, type) => {
        errors.push(`${path.join('.')}: expected ${type}`);
      },
    });
    return errors;
  }
}

/**
 * Schema evolution handling
 */
class AvroSchemaEvolution {
  /**
   * Read data written with old schema using new schema
   */
  static evolveSchema(
    oldSchema: object,
    newSchema: object,
    oldData: Buffer
  ): any {
    const readerType = avro.Type.forSchema(newSchema);
    const writerType = avro.Type.forSchema(oldSchema);
    
    // Create resolver for schema evolution
    const resolver = readerType.createResolver(writerType);
    
    // Decode with resolver - applies defaults for new fields
    return readerType.fromBuffer(oldData, resolver);
  }
  
  /**
   * Check schema compatibility
   */
  static checkCompatibility(
    readerSchema: object,
    writerSchema: object
  ): { compatible: boolean; errors: string[] } {
    try {
      const readerType = avro.Type.forSchema(readerSchema);
      const writerType = avro.Type.forSchema(writerSchema);
      readerType.createResolver(writerType);
      return { compatible: true, errors: [] };
    } catch (error: any) {
      return { compatible: false, errors: [error.message] };
    }
  }
}

// Usage
function avroExample() {
  const serializer = new AvroSerializer(userSchema);
  
  const user = {
    id: '123',
    name: 'John Doe',
    email: 'john@example.com',
    age: 30,
    roles: ['admin'],
    status: 'ACTIVE',
    country: null,
    createdAt: Date.now(),
  };
  
  const binary = serializer.serialize(user);
  console.log(`Avro size: ${binary.length} bytes`);
  
  const decoded = serializer.deserialize(binary);
  console.log('Decoded:', decoded);
  
  // Schema evolution - read V1 data with V2 schema
  const evolved = AvroSchemaEvolution.evolveSchema(
    userSchema,
    userSchemaV2,
    binary
  );
  console.log('Evolved:', evolved);  // phoneNumber will be null
}
```

### MessagePack

```typescript
// data_serialization/messagepack-serialization.ts

import * as msgpack from '@msgpack/msgpack';

/**
 * MessagePack: Binary JSON
 * Pros: JSON-compatible, smaller than JSON, faster
 * Cons: Not as compact as Protobuf, less tooling
 */

class MessagePackSerializer {
  /**
   * Serialize to binary
   */
  serialize(data: unknown): Uint8Array {
    return msgpack.encode(data);
  }
  
  /**
   * Deserialize from binary
   */
  deserialize<T>(buffer: Uint8Array): T {
    return msgpack.decode(buffer) as T;
  }
  
  /**
   * Streaming encode for large data
   */
  async *encodeStream(items: AsyncIterable<unknown>): AsyncIterable<Uint8Array> {
    for await (const item of items) {
      yield msgpack.encode(item);
    }
  }
  
  /**
   * Custom extension for types not in JSON
   */
  serializeWithExtensions(data: unknown): Uint8Array {
    const extensionCodec = new msgpack.ExtensionCodec();
    
    // Custom handler for Date
    extensionCodec.register({
      type: 0,
      encode: (object: unknown): Uint8Array | null => {
        if (object instanceof Date) {
          return msgpack.encode(object.getTime());
        }
        return null;
      },
      decode: (data: Uint8Array): Date => {
        return new Date(msgpack.decode(data) as number);
      },
    });
    
    // Custom handler for RegExp
    extensionCodec.register({
      type: 1,
      encode: (object: unknown): Uint8Array | null => {
        if (object instanceof RegExp) {
          return msgpack.encode({ source: object.source, flags: object.flags });
        }
        return null;
      },
      decode: (data: Uint8Array): RegExp => {
        const { source, flags } = msgpack.decode(data) as { source: string; flags: string };
        return new RegExp(source, flags);
      },
    });
    
    return msgpack.encode(data, { extensionCodec });
  }
}

// Benchmark comparison
async function benchmark() {
  const data = {
    id: '550e8400-e29b-41d4-a716-446655440000',
    name: 'John Doe',
    email: 'john@example.com',
    age: 30,
    roles: ['admin', 'user', 'moderator'],
    metadata: {
      theme: 'dark',
      language: 'en',
      timezone: 'UTC',
    },
    createdAt: Date.now(),
  };
  
  const iterations = 100000;
  
  // JSON
  let start = Date.now();
  for (let i = 0; i < iterations; i++) {
    const json = JSON.stringify(data);
    JSON.parse(json);
  }
  const jsonTime = Date.now() - start;
  const jsonSize = Buffer.byteLength(JSON.stringify(data));
  
  // MessagePack
  start = Date.now();
  const msgpackSerializer = new MessagePackSerializer();
  for (let i = 0; i < iterations; i++) {
    const binary = msgpackSerializer.serialize(data);
    msgpackSerializer.deserialize(binary);
  }
  const msgpackTime = Date.now() - start;
  const msgpackSize = msgpackSerializer.serialize(data).length;
  
  console.log(`JSON: ${jsonTime}ms, ${jsonSize} bytes`);
  console.log(`MessagePack: ${msgpackTime}ms, ${msgpackSize} bytes`);
  console.log(`MessagePack is ${(jsonTime / msgpackTime).toFixed(1)}x faster`);
  console.log(`MessagePack is ${(jsonSize / msgpackSize).toFixed(1)}x smaller`);
}
```

---

## Real-World Scenarios

### Scenario 1: Choosing Format by Use Case

```typescript
// data_serialization/format-selection.ts

type UseCase = 
  | 'rest_api'
  | 'grpc_service'
  | 'kafka_events'
  | 'file_storage'
  | 'browser_cache'
  | 'mobile_app';

interface FormatRecommendation {
  format: 'json' | 'protobuf' | 'avro' | 'messagepack';
  reasons: string[];
}

function recommendFormat(useCase: UseCase): FormatRecommendation {
  switch (useCase) {
    case 'rest_api':
      return {
        format: 'json',
        reasons: [
          'Universal browser/client support',
          'Human-readable for debugging',
          'No schema required',
          'Standard for REST APIs',
        ],
      };
    
    case 'grpc_service':
      return {
        format: 'protobuf',
        reasons: [
          'Native gRPC format',
          'Smallest size',
          'Fastest serialization',
          'Strong typing with .proto files',
        ],
      };
    
    case 'kafka_events':
      return {
        format: 'avro',
        reasons: [
          'Schema registry integration',
          'Schema evolution support',
          'Self-describing messages',
          'Kafka ecosystem standard',
        ],
      };
    
    case 'file_storage':
      return {
        format: 'avro',  // or Parquet for columnar
        reasons: [
          'Schema embedded in file',
          'Compression support',
          'Splittable for parallel processing',
          'Compatible with Spark/Hadoop',
        ],
      };
    
    case 'browser_cache':
      return {
        format: 'json',
        reasons: [
          'Native browser support',
          'Works with localStorage/IndexedDB',
          'Easy debugging in DevTools',
        ],
      };
    
    case 'mobile_app':
      return {
        format: 'protobuf',
        reasons: [
          'Minimal bandwidth usage',
          'Fast parsing (important for battery)',
          'Type-safe code generation',
        ],
      };
  }
}
```

### Scenario 2: Schema Registry with Avro

```typescript
// data_serialization/schema-registry.ts

import axios from 'axios';

interface SchemaInfo {
  id: number;
  schema: string;
  version: number;
}

/**
 * Confluent Schema Registry client
 * Used with Kafka for Avro schema management
 */
class SchemaRegistryClient {
  private baseUrl: string;
  private schemaCache: Map<number, object> = new Map();
  
  constructor(registryUrl: string) {
    this.baseUrl = registryUrl;
  }
  
  /**
   * Register new schema
   */
  async registerSchema(subject: string, schema: object): Promise<number> {
    const response = await axios.post(
      `${this.baseUrl}/subjects/${subject}/versions`,
      { schema: JSON.stringify(schema) },
      { headers: { 'Content-Type': 'application/vnd.schemaregistry.v1+json' } }
    );
    
    return response.data.id;
  }
  
  /**
   * Get schema by ID
   */
  async getSchemaById(id: number): Promise<object> {
    // Check cache
    if (this.schemaCache.has(id)) {
      return this.schemaCache.get(id)!;
    }
    
    const response = await axios.get(`${this.baseUrl}/schemas/ids/${id}`);
    const schema = JSON.parse(response.data.schema);
    
    // Cache for reuse
    this.schemaCache.set(id, schema);
    
    return schema;
  }
  
  /**
   * Get latest schema for subject
   */
  async getLatestSchema(subject: string): Promise<SchemaInfo> {
    const response = await axios.get(
      `${this.baseUrl}/subjects/${subject}/versions/latest`
    );
    
    return {
      id: response.data.id,
      schema: response.data.schema,
      version: response.data.version,
    };
  }
  
  /**
   * Check compatibility before registering
   */
  async checkCompatibility(
    subject: string,
    schema: object
  ): Promise<{ compatible: boolean }> {
    try {
      const response = await axios.post(
        `${this.baseUrl}/compatibility/subjects/${subject}/versions/latest`,
        { schema: JSON.stringify(schema) },
        { headers: { 'Content-Type': 'application/vnd.schemaregistry.v1+json' } }
      );
      
      return { compatible: response.data.is_compatible };
    } catch (error) {
      return { compatible: false };
    }
  }
}

/**
 * Kafka Avro producer/consumer with schema registry
 */
class KafkaAvroClient {
  private registry: SchemaRegistryClient;
  
  constructor(registryUrl: string) {
    this.registry = new SchemaRegistryClient(registryUrl);
  }
  
  /**
   * Serialize with schema ID prefix (Confluent wire format)
   */
  async serialize(
    subject: string,
    data: object,
    schema: object
  ): Promise<Buffer> {
    // Register/get schema ID
    const schemaId = await this.registry.registerSchema(subject, schema);
    
    // Serialize data
    const avroType = require('avsc').Type.forSchema(schema);
    const avroBuffer = avroType.toBuffer(data);
    
    // Confluent wire format: [magic byte][4-byte schema ID][avro data]
    const result = Buffer.alloc(1 + 4 + avroBuffer.length);
    result.writeUInt8(0, 0);  // Magic byte
    result.writeUInt32BE(schemaId, 1);  // Schema ID
    avroBuffer.copy(result, 5);  // Avro data
    
    return result;
  }
  
  /**
   * Deserialize with schema lookup
   */
  async deserialize(buffer: Buffer): Promise<object> {
    // Read magic byte
    const magic = buffer.readUInt8(0);
    if (magic !== 0) throw new Error('Invalid magic byte');
    
    // Read schema ID
    const schemaId = buffer.readUInt32BE(1);
    
    // Get schema from registry
    const schema = await this.registry.getSchemaById(schemaId);
    
    // Deserialize
    const avroType = require('avsc').Type.forSchema(schema);
    return avroType.fromBuffer(buffer.slice(5));
  }
}
```

---

## Common Pitfalls

### 1. Using JSON for High-Throughput Systems

```typescript
// ❌ BAD: JSON for millions of messages/second
for (const event of events) {
  const json = JSON.stringify(event);  // Slow!
  await kafka.send({ value: json });   // Large payloads
}

// ✅ GOOD: Binary format for throughput
const avroType = avro.Type.forSchema(eventSchema);
for (const event of events) {
  const binary = avroType.toBuffer(event);  // 5x faster, 3x smaller
  await kafka.send({ value: binary });
}
```

### 2. Ignoring Schema Evolution

```typescript
// ❌ BAD: Breaking change without version handling
// V1: { name: string }
// V2: { fullName: string }  // Renamed field - breaks consumers!

// ✅ GOOD: Additive changes with defaults
// V1: { name: string }
// V2: { name: string, fullName?: string }  // Add new field with default
// Eventually: migrate consumers, deprecate 'name'
```

### 3. Not Validating Before Serialization

```typescript
// ❌ BAD: Serialize invalid data
const binary = serializer.serialize(untrustedInput);  // May fail or produce garbage

// ✅ GOOD: Validate first
const errors = serializer.validate(untrustedInput);
if (errors.length > 0) {
  throw new ValidationError(errors);
}
const binary = serializer.serialize(untrustedInput);
```

---

## Interview Questions

### Q1: When would you choose Protobuf over JSON?

**A:** Choose **Protobuf** when: 1) Performance critical (high throughput, low latency). 2) Bandwidth constrained (mobile, IoT). 3) Strong typing needed (code generation from .proto). 4) Using gRPC. 5) Large payloads where size matters. Stick with **JSON** when: human readability needed, dynamic schemas, browser consumption, simpler tooling preferred.

### Q2: What is schema evolution and why does it matter?

**A:** Schema evolution is changing data structure over time (adding/removing fields) without breaking producers/consumers. Matters because: systems evolve, can't update all services simultaneously, need backward compatibility. Avro handles this well with default values for new fields. Strategies: only add optional fields, never remove required fields, use schema registry to track versions.

### Q3: How does Avro differ from Protobuf?

**A:** Key differences: 1) **Schema storage**: Avro embeds schema or uses registry, Protobuf requires shared .proto files. 2) **Evolution**: Avro has better schema evolution with reader/writer schemas. 3) **Ecosystem**: Avro integrates with Hadoop/Kafka, Protobuf with gRPC. 4) **Size**: Protobuf slightly smaller (no field names). 5) **Tooling**: Protobuf has better code generation.

### Q4: What is the Confluent wire format?

**A:** Binary format for Kafka messages with schema registry: `[magic byte (0x00)][4-byte schema ID][serialized data]`. Enables: consumers to look up schema by ID, schema evolution without message changes, centralized schema management. Used with Avro, JSON Schema, and Protobuf in Confluent ecosystem.

---

## Quick Reference Checklist

### Format Selection
- [ ] Human-readable needed → JSON
- [ ] Performance critical → Protobuf/Avro
- [ ] Schema evolution → Avro
- [ ] gRPC → Protobuf
- [ ] Kafka → Avro + Schema Registry

### Implementation
- [ ] Define schemas explicitly
- [ ] Validate before serialization
- [ ] Handle schema evolution
- [ ] Benchmark for your use case

### Operations
- [ ] Version schemas
- [ ] Test compatibility
- [ ] Monitor message sizes
- [ ] Document format choices

---

*Last updated: February 2026*

