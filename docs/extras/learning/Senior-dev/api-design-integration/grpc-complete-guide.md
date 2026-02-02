# ⚡ gRPC - Complete Guide

> A comprehensive guide to gRPC - Protocol Buffers, streaming, service definitions, and when to choose gRPC over REST.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "gRPC is a high-performance, open-source RPC (Remote Procedure Call) framework using Protocol Buffers for serialization and HTTP/2 for transport, enabling efficient communication between services with strong typing and bi-directional streaming."

### The 7 Key Concepts (Remember These!)
```
1. PROTOCOL BUFFERS   → Binary serialization format (.proto files)
2. SERVICE DEFINITION → RPC methods defined in .proto
3. CODE GENERATION    → Generate client/server stubs
4. HTTP/2             → Multiplexing, streaming, header compression
5. UNARY RPC          → Single request, single response
6. STREAMING          → Server, client, or bidirectional streaming
7. INTERCEPTORS       → Middleware for cross-cutting concerns
```

### gRPC vs REST
```
┌─────────────────────────────────────────────────────────────────┐
│                      gRPC vs REST                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Feature          │ gRPC              │ REST                    │
│  ─────────────────┼───────────────────┼─────────────────────── │
│  Protocol         │ HTTP/2            │ HTTP/1.1 (usually)     │
│  Payload          │ Protocol Buffers  │ JSON (usually)         │
│  Contract         │ .proto file       │ OpenAPI (optional)     │
│  Streaming        │ Native support    │ Requires WebSocket     │
│  Browser support  │ Limited (gRPC-Web)│ Full                   │
│  Human readable   │ No (binary)       │ Yes (JSON)             │
│  Payload size     │ Smaller (10x)     │ Larger                 │
│  Performance      │ Faster            │ Slower                 │
│  Learning curve   │ Steeper           │ Gentler                │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  CHOOSE gRPC WHEN:                                             │
│  • Microservices communication                                 │
│  • Low latency required                                        │
│  • Streaming needed                                            │
│  • Strong typing important                                     │
│  • Internal services                                           │
│                                                                 │
│  CHOOSE REST WHEN:                                             │
│  • Public API                                                  │
│  • Browser clients                                             │
│  • Human debugging needed                                      │
│  • Simple CRUD operations                                      │
│  • Team familiarity                                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### gRPC Communication Patterns
```
┌─────────────────────────────────────────────────────────────────┐
│                  gRPC COMMUNICATION PATTERNS                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. UNARY RPC (Request-Response)                               │
│     Client ──────────────────────────────────> Server          │
│              <──────────────────────────────── Response        │
│     Use: Simple queries, CRUD                                  │
│                                                                 │
│  2. SERVER STREAMING                                           │
│     Client ──────────────────────────────────> Server          │
│              <─ Response 1 ─                                   │
│              <─ Response 2 ─                                   │
│              <─ Response N ─                                   │
│     Use: Downloading files, feed updates                       │
│                                                                 │
│  3. CLIENT STREAMING                                           │
│     Client ─ Request 1 ─────────────────────> Server          │
│            ─ Request 2 ─────────────────────>                  │
│            ─ Request N ─────────────────────>                  │
│              <─────────────────────────────── Response         │
│     Use: File upload, aggregation                              │
│                                                                 │
│  4. BIDIRECTIONAL STREAMING                                    │
│     Client <──────────────────────────────────> Server         │
│            ─ Request 1  ─>    <─ Response 1 ─                  │
│            ─ Request 2  ─>    <─ Response 2 ─                  │
│     Use: Chat, real-time sync                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Protocol Buffers"** | "We use Protocol Buffers for efficient serialization" |
| **"HTTP/2 multiplexing"** | "gRPC leverages HTTP/2 multiplexing for concurrent requests" |
| **"Bidirectional streaming"** | "We use bidirectional streaming for real-time updates" |
| **"Service mesh"** | "gRPC integrates well with service mesh (Istio)" |
| **"Interceptors"** | "Cross-cutting concerns handled via interceptors" |
| **"Code generation"** | "Types are generated from .proto definitions" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Payload size | **10x smaller** | vs JSON |
| Latency | **~10x faster** | Binary + HTTP/2 |
| Proto versions | **proto3** | Current standard |
| Default port | **50051** | Convention |

### The "Wow" Statement (Memorize This!)
> "We use gRPC for service-to-service communication in our microservices. Protocol Buffers give us 10x smaller payloads than JSON and strong typing across languages. HTTP/2 provides multiplexing, reducing connection overhead. We define services in .proto files, generating TypeScript clients and Go servers. For real-time features like order tracking, we use server streaming. Bidirectional streaming powers our chat. Interceptors handle auth, logging, and metrics consistently. We expose REST via gRPC-Gateway for external clients while keeping gRPC internally. The strict contract (.proto) catches breaking changes at compile time."

---

## 📚 Table of Contents

1. [Protocol Buffers](#1-protocol-buffers)
2. [Service Definition](#2-service-definition)
3. [Implementation](#3-implementation)
4. [Streaming](#4-streaming)
5. [Best Practices](#5-best-practices)
6. [Common Pitfalls](#6-common-pitfalls)
7. [Interview Questions](#7-interview-questions)

---

## 1. Protocol Buffers

```protobuf
// ════════════════════════════════════════════════════════════════
// PROTOCOL BUFFER BASICS (.proto file)
// ════════════════════════════════════════════════════════════════

syntax = "proto3";

package user.v1;

option go_package = "github.com/example/user/v1";
option java_package = "com.example.user.v1";

// Import other proto files
import "google/protobuf/timestamp.proto";
import "google/protobuf/empty.proto";

// ════════════════════════════════════════════════════════════════
// MESSAGE DEFINITIONS
// ════════════════════════════════════════════════════════════════

// Basic message
message User {
  string id = 1;           // Field number (1-15 use 1 byte, 16-2047 use 2 bytes)
  string name = 2;
  string email = 3;
  UserStatus status = 4;
  Role role = 5;
  google.protobuf.Timestamp created_at = 6;
  google.protobuf.Timestamp updated_at = 7;
  
  // Nested message
  Profile profile = 8;
  
  // Repeated field (array)
  repeated string tags = 9;
  
  // Map field
  map<string, string> metadata = 10;
  
  // Optional field (proto3 - all fields are optional by default)
  optional string phone = 11;
  
  // Oneof - only one field can be set
  oneof contact_method {
    string phone_number = 12;
    string slack_id = 13;
  }
}

// Enum definition
enum UserStatus {
  USER_STATUS_UNSPECIFIED = 0;  // Must have 0 value
  USER_STATUS_ACTIVE = 1;
  USER_STATUS_INACTIVE = 2;
  USER_STATUS_PENDING = 3;
}

enum Role {
  ROLE_UNSPECIFIED = 0;
  ROLE_USER = 1;
  ROLE_ADMIN = 2;
  ROLE_MODERATOR = 3;
}

// Nested message
message Profile {
  string bio = 1;
  string avatar_url = 2;
  repeated SocialLink social_links = 3;
}

message SocialLink {
  string platform = 1;
  string url = 2;
}

// ════════════════════════════════════════════════════════════════
// REQUEST/RESPONSE MESSAGES
// ════════════════════════════════════════════════════════════════

message CreateUserRequest {
  string name = 1;
  string email = 2;
  string password = 3;
  Role role = 4;
}

message CreateUserResponse {
  User user = 1;
}

message GetUserRequest {
  string id = 1;
}

message GetUserResponse {
  User user = 1;
}

message ListUsersRequest {
  int32 page_size = 1;
  string page_token = 2;
  UserStatus status_filter = 3;
}

message ListUsersResponse {
  repeated User users = 1;
  string next_page_token = 2;
  int32 total_count = 3;
}

message UpdateUserRequest {
  string id = 1;
  string name = 2;
  string email = 3;
  UserStatus status = 4;
  // Use field mask for partial updates
  google.protobuf.FieldMask update_mask = 5;
}

message DeleteUserRequest {
  string id = 1;
}

// ════════════════════════════════════════════════════════════════
// SCALAR TYPES
// ════════════════════════════════════════════════════════════════

/*
  Type        Default   Notes
  ─────────────────────────────────────────────
  double      0         64-bit float
  float       0         32-bit float
  int32       0         Variable-length
  int64       0         Variable-length
  uint32      0         Variable-length
  uint64      0         Variable-length
  sint32      0         Signed, efficient for negatives
  sint64      0         Signed, efficient for negatives
  fixed32     0         Always 4 bytes
  fixed64     0         Always 8 bytes
  sfixed32    0         Always 4 bytes, signed
  sfixed64    0         Always 8 bytes, signed
  bool        false     Boolean
  string      ""        UTF-8 or ASCII
  bytes       empty     Arbitrary bytes
*/
```

---

## 2. Service Definition

```protobuf
// ════════════════════════════════════════════════════════════════
// SERVICE DEFINITION
// ════════════════════════════════════════════════════════════════

syntax = "proto3";

package user.v1;

import "google/protobuf/empty.proto";
import "user/v1/user.proto";

// Service definition
service UserService {
  // Unary RPC - single request, single response
  rpc CreateUser(CreateUserRequest) returns (CreateUserResponse);
  rpc GetUser(GetUserRequest) returns (GetUserResponse);
  rpc UpdateUser(UpdateUserRequest) returns (User);
  rpc DeleteUser(DeleteUserRequest) returns (google.protobuf.Empty);
  
  // Server streaming - client sends one request, server sends multiple responses
  rpc ListUsers(ListUsersRequest) returns (stream User);
  
  // Client streaming - client sends multiple requests, server sends one response
  rpc BatchCreateUsers(stream CreateUserRequest) returns (BatchCreateUsersResponse);
  
  // Bidirectional streaming - both sides send multiple messages
  rpc UserChat(stream ChatMessage) returns (stream ChatMessage);
}

message BatchCreateUsersResponse {
  repeated User users = 1;
  int32 success_count = 2;
  int32 failure_count = 3;
}

message ChatMessage {
  string user_id = 1;
  string content = 2;
  google.protobuf.Timestamp timestamp = 3;
}

// ════════════════════════════════════════════════════════════════
// ORDER SERVICE EXAMPLE
// ════════════════════════════════════════════════════════════════

service OrderService {
  // Create order
  rpc CreateOrder(CreateOrderRequest) returns (Order);
  
  // Get order
  rpc GetOrder(GetOrderRequest) returns (Order);
  
  // Stream order status updates (server streaming)
  rpc WatchOrder(WatchOrderRequest) returns (stream OrderStatusUpdate);
  
  // Bulk order import (client streaming)
  rpc ImportOrders(stream Order) returns (ImportOrdersResponse);
  
  // Real-time order matching (bidirectional)
  rpc MatchOrders(stream OrderMatch) returns (stream OrderMatch);
}

message Order {
  string id = 1;
  string user_id = 2;
  repeated OrderItem items = 3;
  OrderStatus status = 4;
  Money total = 5;
  google.protobuf.Timestamp created_at = 6;
}

message OrderItem {
  string product_id = 1;
  int32 quantity = 2;
  Money price = 3;
}

message Money {
  string currency_code = 1;  // ISO 4217
  int64 units = 2;           // Whole units
  int32 nanos = 3;           // Nano units (10^-9)
}

enum OrderStatus {
  ORDER_STATUS_UNSPECIFIED = 0;
  ORDER_STATUS_PENDING = 1;
  ORDER_STATUS_PROCESSING = 2;
  ORDER_STATUS_SHIPPED = 3;
  ORDER_STATUS_DELIVERED = 4;
  ORDER_STATUS_CANCELLED = 5;
}

message WatchOrderRequest {
  string order_id = 1;
}

message OrderStatusUpdate {
  string order_id = 1;
  OrderStatus old_status = 2;
  OrderStatus new_status = 3;
  google.protobuf.Timestamp updated_at = 4;
  string message = 5;
}
```

---

## 3. Implementation

```typescript
// ════════════════════════════════════════════════════════════════
// NODE.js gRPC SERVER IMPLEMENTATION
// ════════════════════════════════════════════════════════════════

import * as grpc from '@grpc/grpc-js';
import * as protoLoader from '@grpc/proto-loader';
import { ProtoGrpcType } from './generated/user';
import { UserServiceHandlers } from './generated/user/v1/UserService';

// Load proto file
const packageDefinition = protoLoader.loadSync('./proto/user/v1/user.proto', {
  keepCase: true,
  longs: String,
  enums: String,
  defaults: true,
  oneofs: true,
});

const proto = grpc.loadPackageDefinition(packageDefinition) as unknown as ProtoGrpcType;

// Implement service handlers
const userServiceHandlers: UserServiceHandlers = {
  // Unary RPC
  async CreateUser(call, callback) {
    try {
      const { name, email, password, role } = call.request;
      
      // Validate
      if (!name || !email) {
        return callback({
          code: grpc.status.INVALID_ARGUMENT,
          message: 'Name and email are required',
        });
      }

      // Create user
      const user = await userRepository.create({ name, email, password, role });
      
      callback(null, { user });
    } catch (error) {
      callback({
        code: grpc.status.INTERNAL,
        message: error.message,
      });
    }
  },

  async GetUser(call, callback) {
    try {
      const { id } = call.request;
      const user = await userRepository.findById(id);
      
      if (!user) {
        return callback({
          code: grpc.status.NOT_FOUND,
          message: `User ${id} not found`,
        });
      }
      
      callback(null, { user });
    } catch (error) {
      callback({
        code: grpc.status.INTERNAL,
        message: error.message,
      });
    }
  },

  // Server streaming
  ListUsers(call) {
    const { page_size, status_filter } = call.request;
    
    const stream = userRepository.streamUsers({ status: status_filter });
    
    stream.on('data', (user) => {
      call.write(user);
    });
    
    stream.on('end', () => {
      call.end();
    });
    
    stream.on('error', (error) => {
      call.destroy(error);
    });
  },

  // Client streaming
  BatchCreateUsers(call, callback) {
    const users: User[] = [];
    let successCount = 0;
    let failureCount = 0;

    call.on('data', async (request) => {
      try {
        const user = await userRepository.create(request);
        users.push(user);
        successCount++;
      } catch (error) {
        failureCount++;
      }
    });

    call.on('end', () => {
      callback(null, {
        users,
        success_count: successCount,
        failure_count: failureCount,
      });
    });

    call.on('error', (error) => {
      callback(error);
    });
  },

  // Bidirectional streaming
  UserChat(call) {
    call.on('data', async (message) => {
      // Broadcast to other users
      const response = {
        user_id: message.user_id,
        content: `Echo: ${message.content}`,
        timestamp: new Date().toISOString(),
      };
      call.write(response);
    });

    call.on('end', () => {
      call.end();
    });
  },
};

// Create and start server
const server = new grpc.Server();

server.addService(proto.user.v1.UserService.service, userServiceHandlers);

server.bindAsync(
  '0.0.0.0:50051',
  grpc.ServerCredentials.createInsecure(),
  (error, port) => {
    if (error) {
      console.error('Server failed to start:', error);
      return;
    }
    console.log(`Server running on port ${port}`);
    server.start();
  }
);

// ════════════════════════════════════════════════════════════════
// NODE.js gRPC CLIENT
// ════════════════════════════════════════════════════════════════

import * as grpc from '@grpc/grpc-js';

// Create client
const client = new proto.user.v1.UserService(
  'localhost:50051',
  grpc.credentials.createInsecure()
);

// Unary call
async function createUser(name: string, email: string): Promise<User> {
  return new Promise((resolve, reject) => {
    client.CreateUser({ name, email, password: 'temp' }, (error, response) => {
      if (error) reject(error);
      else resolve(response.user);
    });
  });
}

// With async/await wrapper
import { promisify } from 'util';

const createUserAsync = promisify(client.CreateUser).bind(client);
const getUserAsync = promisify(client.GetUser).bind(client);

// Usage
const user = await createUserAsync({ name: 'John', email: 'john@example.com', password: 'secret' });

// Server streaming
function listUsers(): AsyncIterable<User> {
  const call = client.ListUsers({ page_size: 100 });
  
  return {
    async *[Symbol.asyncIterator]() {
      for await (const user of call) {
        yield user;
      }
    },
  };
}

// Usage
for await (const user of listUsers()) {
  console.log(user);
}

// Client streaming
async function batchCreateUsers(users: CreateUserRequest[]): Promise<BatchCreateUsersResponse> {
  return new Promise((resolve, reject) => {
    const call = client.BatchCreateUsers((error, response) => {
      if (error) reject(error);
      else resolve(response);
    });

    for (const user of users) {
      call.write(user);
    }
    
    call.end();
  });
}
```

---

## 4. Streaming

```typescript
// ════════════════════════════════════════════════════════════════
// SERVER STREAMING - Real-time Order Updates
// ════════════════════════════════════════════════════════════════

// Server implementation
const orderServiceHandlers: OrderServiceHandlers = {
  WatchOrder(call) {
    const { order_id } = call.request;
    
    // Subscribe to order updates
    const unsubscribe = orderEventEmitter.on(`order:${order_id}`, (update) => {
      call.write(update);
    });

    // Handle client disconnect
    call.on('cancelled', () => {
      unsubscribe();
    });

    // Keep connection open until client cancels
    // or order reaches terminal state
  },
};

// Client usage
function watchOrder(orderId: string): void {
  const call = client.WatchOrder({ order_id: orderId });

  call.on('data', (update: OrderStatusUpdate) => {
    console.log(`Order ${update.order_id}: ${update.old_status} -> ${update.new_status}`);
    updateUI(update);
  });

  call.on('end', () => {
    console.log('Order tracking ended');
  });

  call.on('error', (error) => {
    console.error('Tracking error:', error);
    // Implement reconnection logic
    setTimeout(() => watchOrder(orderId), 5000);
  });

  // Cancel tracking
  // call.cancel();
}

// ════════════════════════════════════════════════════════════════
// CLIENT STREAMING - File Upload
// ════════════════════════════════════════════════════════════════

// Proto
/*
service FileService {
  rpc UploadFile(stream FileChunk) returns (UploadResponse);
}

message FileChunk {
  bytes content = 1;
  string filename = 2;  // Only in first chunk
  int64 offset = 3;
}

message UploadResponse {
  string file_id = 1;
  int64 size = 2;
  string checksum = 3;
}
*/

// Server implementation
const fileServiceHandlers: FileServiceHandlers = {
  UploadFile(call, callback) {
    let filename = '';
    let totalSize = 0;
    const chunks: Buffer[] = [];

    call.on('data', (chunk: FileChunk) => {
      if (chunk.filename) {
        filename = chunk.filename;
      }
      chunks.push(Buffer.from(chunk.content));
      totalSize += chunk.content.length;
    });

    call.on('end', async () => {
      const fileBuffer = Buffer.concat(chunks);
      const fileId = await fileStorage.save(filename, fileBuffer);
      const checksum = calculateChecksum(fileBuffer);

      callback(null, {
        file_id: fileId,
        size: totalSize,
        checksum,
      });
    });

    call.on('error', (error) => {
      callback(error);
    });
  },
};

// Client implementation
async function uploadFile(filePath: string): Promise<UploadResponse> {
  return new Promise((resolve, reject) => {
    const call = client.UploadFile((error, response) => {
      if (error) reject(error);
      else resolve(response);
    });

    const fileStream = fs.createReadStream(filePath, { highWaterMark: 64 * 1024 }); // 64KB chunks
    const filename = path.basename(filePath);
    let isFirst = true;

    fileStream.on('data', (chunk: Buffer) => {
      call.write({
        content: chunk,
        filename: isFirst ? filename : undefined,
        offset: isFirst ? 0 : undefined,
      });
      isFirst = false;
    });

    fileStream.on('end', () => {
      call.end();
    });

    fileStream.on('error', (error) => {
      call.destroy(error);
      reject(error);
    });
  });
}

// ════════════════════════════════════════════════════════════════
// BIDIRECTIONAL STREAMING - Chat
// ════════════════════════════════════════════════════════════════

// Proto
/*
service ChatService {
  rpc Chat(stream ChatMessage) returns (stream ChatMessage);
}

message ChatMessage {
  string room_id = 1;
  string user_id = 2;
  string content = 3;
  google.protobuf.Timestamp timestamp = 4;
}
*/

// Server implementation
const chatRooms = new Map<string, Set<grpc.ServerDuplexStream<ChatMessage, ChatMessage>>>();

const chatServiceHandlers: ChatServiceHandlers = {
  Chat(call) {
    let roomId: string | null = null;

    call.on('data', (message: ChatMessage) => {
      roomId = message.room_id;
      
      // Join room
      if (!chatRooms.has(roomId)) {
        chatRooms.set(roomId, new Set());
      }
      chatRooms.get(roomId)!.add(call);

      // Broadcast to all clients in room
      const clients = chatRooms.get(roomId)!;
      for (const client of clients) {
        client.write({
          ...message,
          timestamp: new Date().toISOString(),
        });
      }
    });

    call.on('end', () => {
      // Leave room
      if (roomId && chatRooms.has(roomId)) {
        chatRooms.get(roomId)!.delete(call);
      }
      call.end();
    });

    call.on('error', () => {
      if (roomId && chatRooms.has(roomId)) {
        chatRooms.get(roomId)!.delete(call);
      }
    });
  },
};

// Client implementation
class ChatClient {
  private call: grpc.ClientDuplexStream<ChatMessage, ChatMessage>;
  private roomId: string;
  private userId: string;

  constructor(roomId: string, userId: string) {
    this.roomId = roomId;
    this.userId = userId;
    this.call = client.Chat();
    this.setupListeners();
  }

  private setupListeners() {
    this.call.on('data', (message: ChatMessage) => {
      console.log(`[${message.user_id}]: ${message.content}`);
      this.onMessage?.(message);
    });

    this.call.on('end', () => {
      console.log('Chat ended');
    });

    this.call.on('error', (error) => {
      console.error('Chat error:', error);
      this.reconnect();
    });
  }

  send(content: string) {
    this.call.write({
      room_id: this.roomId,
      user_id: this.userId,
      content,
    });
  }

  disconnect() {
    this.call.end();
  }

  onMessage?: (message: ChatMessage) => void;

  private reconnect() {
    setTimeout(() => {
      this.call = client.Chat();
      this.setupListeners();
    }, 5000);
  }
}

// Usage
const chat = new ChatClient('room-1', 'user-123');
chat.onMessage = (msg) => updateChatUI(msg);
chat.send('Hello everyone!');
```

---

## 5. Best Practices

```typescript
// ════════════════════════════════════════════════════════════════
// gRPC BEST PRACTICES
// ════════════════════════════════════════════════════════════════

// 1. INTERCEPTORS FOR CROSS-CUTTING CONCERNS

// Server interceptor (middleware)
function loggingInterceptor(
  methodDescriptor: grpc.MethodDefinition<any, any>,
  call: grpc.ServerUnaryCall<any, any>
): grpc.ServerUnaryCall<any, any> {
  const start = Date.now();
  
  const originalCallback = call.callback;
  call.callback = (error, response) => {
    const duration = Date.now() - start;
    console.log(`${methodDescriptor.path} - ${duration}ms - ${error ? 'ERROR' : 'OK'}`);
    originalCallback(error, response);
  };
  
  return call;
}

// Client interceptor
function authInterceptor(options: any, nextCall: any) {
  return new grpc.InterceptingCall(nextCall(options), {
    start(metadata, listener, next) {
      metadata.add('authorization', `Bearer ${getToken()}`);
      next(metadata, listener);
    },
  });
}

// 2. ERROR HANDLING

// Use standard gRPC status codes
const grpcStatusCodes = {
  OK: 0,
  CANCELLED: 1,
  UNKNOWN: 2,
  INVALID_ARGUMENT: 3,
  DEADLINE_EXCEEDED: 4,
  NOT_FOUND: 5,
  ALREADY_EXISTS: 6,
  PERMISSION_DENIED: 7,
  RESOURCE_EXHAUSTED: 8,
  FAILED_PRECONDITION: 9,
  ABORTED: 10,
  OUT_OF_RANGE: 11,
  UNIMPLEMENTED: 12,
  INTERNAL: 13,
  UNAVAILABLE: 14,
  DATA_LOSS: 15,
  UNAUTHENTICATED: 16,
};

// Rich error details
import { Status } from '@grpc/grpc-js/build/src/constants';

function createError(code: Status, message: string, details?: any): grpc.ServiceError {
  const error: grpc.ServiceError = {
    code,
    message,
    details: JSON.stringify(details),
    metadata: new grpc.Metadata(),
  };
  return error;
}

// Usage
callback(createError(
  grpc.status.INVALID_ARGUMENT,
  'Validation failed',
  { errors: [{ field: 'email', message: 'Invalid format' }] }
));

// 3. DEADLINES AND TIMEOUTS

// Client with deadline
const deadline = new Date();
deadline.setSeconds(deadline.getSeconds() + 30); // 30 second timeout

client.GetUser({ id: '123' }, { deadline }, (error, response) => {
  if (error?.code === grpc.status.DEADLINE_EXCEEDED) {
    console.log('Request timed out');
  }
});

// 4. HEALTH CHECKS

// Proto
/*
syntax = "proto3";

package grpc.health.v1;

service Health {
  rpc Check(HealthCheckRequest) returns (HealthCheckResponse);
  rpc Watch(HealthCheckRequest) returns (stream HealthCheckResponse);
}

message HealthCheckRequest {
  string service = 1;
}

message HealthCheckResponse {
  enum ServingStatus {
    UNKNOWN = 0;
    SERVING = 1;
    NOT_SERVING = 2;
  }
  ServingStatus status = 1;
}
*/

// 5. GRACEFUL SHUTDOWN

async function gracefulShutdown(server: grpc.Server) {
  return new Promise<void>((resolve) => {
    server.tryShutdown(() => {
      console.log('Server shut down gracefully');
      resolve();
    });
  });
}

process.on('SIGTERM', async () => {
  await gracefulShutdown(server);
  process.exit(0);
});
```

---

## 6. Common Pitfalls

```yaml
# ════════════════════════════════════════════════════════════════
# gRPC PITFALLS
# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 1: Changing field numbers
# Bad
message User {
  string id = 1;
  string name = 2;  # Was email before!
}

# Good - Never change field numbers
# Add new fields with new numbers
# Mark old fields as reserved

message User {
  string id = 1;
  reserved 2;  # Was email
  string email = 3;  # New field number
  string name = 4;
}

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 2: Not handling stream errors
# Bad
const call = client.ListUsers({});
call.on('data', (user) => console.log(user));
# Missing error handler!

# Good
call.on('data', (user) => console.log(user));
call.on('error', (error) => handleError(error));
call.on('end', () => console.log('Stream ended'));

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 3: No deadlines
# Bad - Request can hang forever
client.GetUser({ id: '123' }, callback);

# Good - Always set deadlines
const deadline = new Date(Date.now() + 30000);
client.GetUser({ id: '123' }, { deadline }, callback);

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 4: Exposing gRPC directly to browsers
# Bad - gRPC uses HTTP/2 which browsers can't use directly

# Good - Use gRPC-Web with Envoy proxy
# Or provide REST API via gRPC-Gateway

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 5: Large messages
# Bad - Sending entire file in one message
message FileContent {
  bytes content = 1;  # Could be GBs!
}

# Good - Use streaming for large data
rpc UploadFile(stream FileChunk) returns (UploadResponse);
message FileChunk {
  bytes content = 1;  # Small chunks (64KB-1MB)
}

# ════════════════════════════════════════════════════════════════

# ❌ PITFALL 6: Not versioning protos
# Bad
package user;  # No version

# Good - Include version in package
package user.v1;
# Later: package user.v2;
```

---

## 7. Interview Questions

### Basic Questions

**Q: "What is gRPC?"**
> "gRPC is a high-performance RPC framework by Google. Uses Protocol Buffers for serialization (binary, ~10x smaller than JSON) and HTTP/2 for transport (multiplexing, streaming). Define services in .proto files, generate client/server code. Great for microservices communication."

**Q: "gRPC vs REST - when to use each?"**
> "gRPC: Microservices (internal), low latency critical, streaming needed, polyglot environments. REST: Public APIs, browser clients, human debugging, simpler setup. gRPC is faster but less accessible. Many use both: gRPC internal, REST external via gRPC-Gateway."

**Q: "What are Protocol Buffers?"**
> "Protocol Buffers (protobuf) is a language-neutral, platform-neutral serialization format. Define structure in .proto files, generate code for any language. Binary format is smaller and faster than JSON. Strongly typed with schema evolution support."

### Intermediate Questions

**Q: "What are the gRPC communication patterns?"**
> "Four patterns: 1) Unary - single request/response (like REST). 2) Server streaming - one request, multiple responses (live updates). 3) Client streaming - multiple requests, one response (file upload). 4) Bidirectional - both sides stream independently (chat). All use single HTTP/2 connection."

**Q: "How do you handle errors in gRPC?"**
> "gRPC has standard status codes (OK, NOT_FOUND, INVALID_ARGUMENT, etc.) similar to HTTP but more specific. Return status code and message. For rich errors, use metadata or Google's error details proto. Always handle stream errors on client. Use DEADLINE_EXCEEDED for timeouts."

**Q: "How does gRPC achieve better performance?"**
> "Multiple factors: 1) Binary Protocol Buffers vs text JSON (~10x smaller). 2) HTTP/2 multiplexing - multiple requests on one connection. 3) HTTP/2 header compression. 4) Persistent connections. 5) Code generation - no reflection/parsing overhead. Combined: significantly lower latency and bandwidth."

### Advanced Questions

**Q: "How do you handle versioning in gRPC?"**
> "Proto supports backward-compatible evolution: add optional fields, never change field numbers, mark removed fields as reserved. For breaking changes: new package version (user.v1 → user.v2), run both versions, migrate clients. Use semantic versioning. Test compatibility with buf breaking."

**Q: "How do you use gRPC with browsers?"**
> "Browsers can't use HTTP/2 trailers needed by gRPC. Solutions: 1) gRPC-Web - modified protocol, needs Envoy proxy. 2) gRPC-Gateway - generates REST API from proto, best for public APIs. 3) WebSocket transport. Most common: internal gRPC, external REST via gateway."

**Q: "How do you handle authentication in gRPC?"**
> "Multiple approaches: 1) Metadata - add auth token in interceptor, like HTTP headers. 2) SSL/TLS - mutual TLS for service-to-service. 3) Token-based - JWT in metadata, validate in server interceptor. 4) Google's ALTS for GCP. Use interceptors for consistent auth across all RPCs."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                     gRPC CHECKLIST                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  PROTO DESIGN:                                                  │
│  □ Use proto3 syntax                                           │
│  □ Version in package name                                     │
│  □ Never change field numbers                                  │
│  □ Use reserved for removed fields                             │
│                                                                 │
│  IMPLEMENTATION:                                                │
│  □ Interceptors for cross-cutting concerns                     │
│  □ Always set deadlines                                        │
│  □ Handle stream errors                                        │
│  □ Implement health checks                                     │
│  □ Graceful shutdown                                           │
│                                                                 │
│  PRODUCTION:                                                    │
│  □ TLS enabled                                                 │
│  □ Load balancing configured                                   │
│  □ Retries with backoff                                        │
│  □ Monitoring (latency, errors)                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

COMMUNICATION PATTERNS:
┌────────────────────────────────────────────────────────────────┐
│ Unary:         Request ──> Response                            │
│ Server Stream: Request ──> Response, Response, Response...     │
│ Client Stream: Request, Request, Request... ──> Response       │
│ Bidirectional: Requests <──> Responses (independent streams)   │
└────────────────────────────────────────────────────────────────┘

gRPC STATUS CODES:
┌────────────────────────────────────────────────────────────────┐
│ OK(0)  INVALID_ARGUMENT(3)  NOT_FOUND(5)  ALREADY_EXISTS(6)   │
│ PERMISSION_DENIED(7)  UNAUTHENTICATED(16)  DEADLINE_EXCEEDED(4)│
│ UNAVAILABLE(14)  INTERNAL(13)  UNIMPLEMENTED(12)              │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

