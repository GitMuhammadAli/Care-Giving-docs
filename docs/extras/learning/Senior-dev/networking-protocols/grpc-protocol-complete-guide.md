# gRPC Protocol - Complete Guide

> **MUST REMEMBER**: gRPC uses HTTP/2 for transport with Protocol Buffers for serialization. Four streaming types: unary (request-response), server streaming, client streaming, bidirectional. Requires .proto schema files defining services and messages. Use deadlines (not timeouts) for request limits. Metadata for headers, status codes for errors. Better for internal microservices; REST better for public APIs.

---

## How to Explain Like a Senior Developer

"gRPC is Google's RPC framework built on HTTP/2 and Protocol Buffers. You define your API in .proto files with services (methods) and messages (data structures). The protoc compiler generates client and server code in any language. Key benefits over REST: strong typing catches errors at compile time, binary protocol is smaller and faster, HTTP/2 gives multiplexing and bidirectional streaming. Four patterns: unary (like REST), server streaming (server sends multiple responses), client streaming (client sends multiple requests), bidirectional (both stream). Use deadlines instead of timeouts - they propagate across service calls. gRPC shines for internal microservice communication but lacks browser support without a proxy, so REST is often better for public APIs."

---

## Core Implementation

### Protocol Buffer Definition

```protobuf
// proto/user.proto
syntax = "proto3";

package user;

option go_package = "github.com/example/user";

// Import for well-known types
import "google/protobuf/timestamp.proto";
import "google/protobuf/empty.proto";

// User service definition
service UserService {
  // Unary RPC - simple request/response
  rpc GetUser(GetUserRequest) returns (User);
  rpc CreateUser(CreateUserRequest) returns (User);
  rpc UpdateUser(UpdateUserRequest) returns (User);
  rpc DeleteUser(DeleteUserRequest) returns (google.protobuf.Empty);
  
  // Server streaming - server sends multiple responses
  rpc ListUsers(ListUsersRequest) returns (stream User);
  rpc WatchUser(WatchUserRequest) returns (stream UserEvent);
  
  // Client streaming - client sends multiple requests
  rpc BatchCreateUsers(stream CreateUserRequest) returns (BatchCreateResponse);
  
  // Bidirectional streaming
  rpc Chat(stream ChatMessage) returns (stream ChatMessage);
}

// Messages
message User {
  string id = 1;
  string email = 2;
  string name = 3;
  UserRole role = 4;
  google.protobuf.Timestamp created_at = 5;
  google.protobuf.Timestamp updated_at = 6;
  
  // Nested message
  Address address = 7;
  
  // Repeated field (array)
  repeated string tags = 8;
  
  // Map field
  map<string, string> metadata = 9;
}

message Address {
  string street = 1;
  string city = 2;
  string country = 3;
  string postal_code = 4;
}

enum UserRole {
  USER_ROLE_UNSPECIFIED = 0;
  USER_ROLE_USER = 1;
  USER_ROLE_ADMIN = 2;
  USER_ROLE_MODERATOR = 3;
}

message GetUserRequest {
  string id = 1;
}

message CreateUserRequest {
  string email = 1;
  string name = 2;
  string password = 3;
  UserRole role = 4;
}

message UpdateUserRequest {
  string id = 1;
  optional string email = 2;
  optional string name = 3;
  optional UserRole role = 4;
}

message DeleteUserRequest {
  string id = 1;
}

message ListUsersRequest {
  int32 page_size = 1;
  string page_token = 2;
  string filter = 3;
}

message WatchUserRequest {
  string user_id = 1;
}

message UserEvent {
  enum EventType {
    EVENT_TYPE_UNSPECIFIED = 0;
    EVENT_TYPE_CREATED = 1;
    EVENT_TYPE_UPDATED = 2;
    EVENT_TYPE_DELETED = 3;
  }
  
  EventType type = 1;
  User user = 2;
  google.protobuf.Timestamp timestamp = 3;
}

message BatchCreateResponse {
  repeated User users = 1;
  int32 success_count = 2;
  int32 failure_count = 3;
}

message ChatMessage {
  string user_id = 1;
  string content = 2;
  google.protobuf.Timestamp timestamp = 3;
}
```

### Node.js gRPC Server

```typescript
// grpc/server.ts
import * as grpc from '@grpc/grpc-js';
import * as protoLoader from '@grpc/proto-loader';
import path from 'path';

// Load proto file
const PROTO_PATH = path.resolve(__dirname, '../proto/user.proto');
const packageDefinition = protoLoader.loadSync(PROTO_PATH, {
  keepCase: true,
  longs: String,
  enums: String,
  defaults: true,
  oneofs: true,
});

const userProto = grpc.loadPackageDefinition(packageDefinition).user as any;

// Mock database
const users: Map<string, any> = new Map();

// Service implementation
const userService = {
  // Unary RPC
  getUser: (
    call: grpc.ServerUnaryCall<any, any>,
    callback: grpc.sendUnaryData<any>
  ) => {
    const userId = call.request.id;
    const user = users.get(userId);
    
    if (!user) {
      callback({
        code: grpc.status.NOT_FOUND,
        message: `User ${userId} not found`,
      });
      return;
    }
    
    callback(null, user);
  },
  
  createUser: (
    call: grpc.ServerUnaryCall<any, any>,
    callback: grpc.sendUnaryData<any>
  ) => {
    const { email, name, role } = call.request;
    
    // Validate
    if (!email || !name) {
      callback({
        code: grpc.status.INVALID_ARGUMENT,
        message: 'Email and name are required',
      });
      return;
    }
    
    const user = {
      id: `user_${Date.now()}`,
      email,
      name,
      role: role || 'USER_ROLE_USER',
      created_at: { seconds: Math.floor(Date.now() / 1000) },
      updated_at: { seconds: Math.floor(Date.now() / 1000) },
    };
    
    users.set(user.id, user);
    callback(null, user);
  },
  
  // Server streaming
  listUsers: (call: grpc.ServerWritableStream<any, any>) => {
    const pageSize = call.request.page_size || 10;
    const filter = call.request.filter;
    
    let count = 0;
    for (const [id, user] of users) {
      // Apply filter if provided
      if (filter && !user.name.includes(filter)) {
        continue;
      }
      
      // Stream each user
      call.write(user);
      count++;
      
      if (count >= pageSize) break;
    }
    
    call.end();
  },
  
  // Server streaming for real-time updates
  watchUser: (call: grpc.ServerWritableStream<any, any>) => {
    const userId = call.request.user_id;
    
    // Set up watcher (simplified)
    const interval = setInterval(() => {
      const user = users.get(userId);
      if (user) {
        call.write({
          type: 'EVENT_TYPE_UPDATED',
          user,
          timestamp: { seconds: Math.floor(Date.now() / 1000) },
        });
      }
    }, 5000);
    
    call.on('cancelled', () => {
      clearInterval(interval);
    });
  },
  
  // Client streaming
  batchCreateUsers: (
    call: grpc.ServerReadableStream<any, any>,
    callback: grpc.sendUnaryData<any>
  ) => {
    const createdUsers: any[] = [];
    let failures = 0;
    
    call.on('data', (request: any) => {
      try {
        const user = {
          id: `user_${Date.now()}_${Math.random()}`,
          email: request.email,
          name: request.name,
          role: request.role || 'USER_ROLE_USER',
          created_at: { seconds: Math.floor(Date.now() / 1000) },
        };
        
        users.set(user.id, user);
        createdUsers.push(user);
      } catch {
        failures++;
      }
    });
    
    call.on('end', () => {
      callback(null, {
        users: createdUsers,
        success_count: createdUsers.length,
        failure_count: failures,
      });
    });
  },
  
  // Bidirectional streaming
  chat: (call: grpc.ServerDuplexStream<any, any>) => {
    call.on('data', (message: any) => {
      // Broadcast to this connection (in real app, broadcast to others)
      call.write({
        user_id: 'server',
        content: `Echo: ${message.content}`,
        timestamp: { seconds: Math.floor(Date.now() / 1000) },
      });
    });
    
    call.on('end', () => {
      call.end();
    });
  },
};

// Create and start server
const server = new grpc.Server();
server.addService(userProto.UserService.service, userService);

server.bindAsync(
  '0.0.0.0:50051',
  grpc.ServerCredentials.createInsecure(),
  (err, port) => {
    if (err) {
      console.error('Server bind error:', err);
      return;
    }
    console.log(`gRPC server running on port ${port}`);
  }
);
```

### Node.js gRPC Client

```typescript
// grpc/client.ts
import * as grpc from '@grpc/grpc-js';
import * as protoLoader from '@grpc/proto-loader';
import path from 'path';

// Load proto
const PROTO_PATH = path.resolve(__dirname, '../proto/user.proto');
const packageDefinition = protoLoader.loadSync(PROTO_PATH, {
  keepCase: true,
  longs: String,
  enums: String,
  defaults: true,
  oneofs: true,
});

const userProto = grpc.loadPackageDefinition(packageDefinition).user as any;

// Create client
const client = new userProto.UserService(
  'localhost:50051',
  grpc.credentials.createInsecure()
);

// Promisify unary calls
function promisifyUnary<TRequest, TResponse>(
  method: Function
): (request: TRequest, metadata?: grpc.Metadata) => Promise<TResponse> {
  return (request: TRequest, metadata?: grpc.Metadata) => {
    return new Promise((resolve, reject) => {
      method.call(
        client,
        request,
        metadata || new grpc.Metadata(),
        (err: grpc.ServiceError | null, response: TResponse) => {
          if (err) reject(err);
          else resolve(response);
        }
      );
    });
  };
}

// Typed client methods
const userClient = {
  getUser: promisifyUnary<{ id: string }, any>(client.getUser),
  createUser: promisifyUnary<any, any>(client.createUser),
  
  // Server streaming
  listUsers: (request: any): grpc.ClientReadableStream<any> => {
    return client.listUsers(request);
  },
  
  // Client streaming
  batchCreateUsers: (): {
    stream: grpc.ClientWritableStream<any>;
    promise: Promise<any>;
  } => {
    let resolvePromise: (value: any) => void;
    let rejectPromise: (err: any) => void;
    
    const promise = new Promise<any>((resolve, reject) => {
      resolvePromise = resolve;
      rejectPromise = reject;
    });
    
    const stream = client.batchCreateUsers((err: any, response: any) => {
      if (err) rejectPromise(err);
      else resolvePromise(response);
    });
    
    return { stream, promise };
  },
  
  // Bidirectional streaming
  chat: (): grpc.ClientDuplexStream<any, any> => {
    return client.chat();
  },
};

// Usage examples
async function examples(): Promise<void> {
  // Unary call with deadline
  try {
    const metadata = new grpc.Metadata();
    metadata.set('authorization', 'Bearer token123');
    
    // Set deadline (5 seconds from now)
    const deadline = new Date(Date.now() + 5000);
    
    const user = await new Promise((resolve, reject) => {
      client.createUser(
        { email: 'test@example.com', name: 'Test User' },
        metadata,
        { deadline },
        (err: any, response: any) => {
          if (err) reject(err);
          else resolve(response);
        }
      );
    });
    
    console.log('Created user:', user);
  } catch (err: any) {
    if (err.code === grpc.status.DEADLINE_EXCEEDED) {
      console.error('Request timed out');
    } else {
      console.error('Error:', err.message);
    }
  }
  
  // Server streaming
  const listStream = userClient.listUsers({ page_size: 10 });
  
  listStream.on('data', (user: any) => {
    console.log('Received user:', user);
  });
  
  listStream.on('end', () => {
    console.log('List complete');
  });
  
  listStream.on('error', (err: any) => {
    console.error('Stream error:', err);
  });
  
  // Client streaming
  const { stream: batchStream, promise: batchPromise } = userClient.batchCreateUsers();
  
  for (let i = 0; i < 10; i++) {
    batchStream.write({
      email: `user${i}@example.com`,
      name: `User ${i}`,
    });
  }
  batchStream.end();
  
  const batchResult = await batchPromise;
  console.log('Batch result:', batchResult);
  
  // Bidirectional streaming
  const chatStream = userClient.chat();
  
  chatStream.on('data', (message: any) => {
    console.log('Chat message:', message);
  });
  
  chatStream.write({ user_id: 'me', content: 'Hello!' });
  chatStream.write({ user_id: 'me', content: 'How are you?' });
  
  setTimeout(() => {
    chatStream.end();
  }, 5000);
}

examples();
```

### Deadlines and Error Handling

```typescript
// grpc/deadlines.ts
import * as grpc from '@grpc/grpc-js';

/**
 * gRPC uses Deadlines, not Timeouts
 * 
 * Deadline = absolute timestamp when request should complete
 * Timeout = relative duration
 * 
 * Deadlines propagate across services automatically
 */

// Set deadline on client call
function callWithDeadline(
  client: any,
  method: string,
  request: any,
  timeoutMs: number
): Promise<any> {
  const deadline = new Date(Date.now() + timeoutMs);
  
  return new Promise((resolve, reject) => {
    client[method](
      request,
      { deadline },
      (err: grpc.ServiceError | null, response: any) => {
        if (err) {
          // Handle specific error codes
          switch (err.code) {
            case grpc.status.DEADLINE_EXCEEDED:
              reject(new Error('Request timed out'));
              break;
            case grpc.status.UNAVAILABLE:
              reject(new Error('Service unavailable'));
              break;
            case grpc.status.NOT_FOUND:
              reject(new Error('Resource not found'));
              break;
            case grpc.status.INVALID_ARGUMENT:
              reject(new Error(`Invalid argument: ${err.message}`));
              break;
            case grpc.status.PERMISSION_DENIED:
              reject(new Error('Permission denied'));
              break;
            case grpc.status.UNAUTHENTICATED:
              reject(new Error('Unauthenticated'));
              break;
            default:
              reject(err);
          }
        } else {
          resolve(response);
        }
      }
    );
  });
}

// Server-side deadline checking
function checkDeadline(call: grpc.ServerUnaryCall<any, any>): boolean {
  const deadline = call.getDeadline();
  
  if (deadline instanceof Date) {
    if (deadline.getTime() < Date.now()) {
      return false; // Already expired
    }
  }
  
  return true;
}

// gRPC Status Codes
const StatusCodes = {
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

export { callWithDeadline, checkDeadline, StatusCodes };
```

### Interceptors (Middleware)

```typescript
// grpc/interceptors.ts
import * as grpc from '@grpc/grpc-js';

/**
 * Interceptors for cross-cutting concerns:
 * - Logging
 * - Authentication
 * - Metrics
 * - Error handling
 */

// Client interceptor
function loggingInterceptor(
  options: any,
  nextCall: any
): grpc.InterceptingCall {
  const method = options.method_definition.path;
  const start = Date.now();
  
  console.log(`[gRPC] Starting ${method}`);
  
  return new grpc.InterceptingCall(nextCall(options), {
    start: function(metadata, listener, next) {
      next(metadata, {
        onReceiveMetadata: function(metadata, next) {
          next(metadata);
        },
        onReceiveMessage: function(message, next) {
          next(message);
        },
        onReceiveStatus: function(status, next) {
          const duration = Date.now() - start;
          console.log(`[gRPC] ${method} completed in ${duration}ms with status ${status.code}`);
          next(status);
        },
      });
    },
  });
}

// Authentication interceptor
function authInterceptor(token: string) {
  return function(
    options: any,
    nextCall: any
  ): grpc.InterceptingCall {
    return new grpc.InterceptingCall(nextCall(options), {
      start: function(metadata, listener, next) {
        metadata.set('authorization', `Bearer ${token}`);
        next(metadata, listener);
      },
    });
  };
}

// Create client with interceptors
function createClientWithInterceptors(
  ServiceClient: any,
  address: string,
  token: string
): any {
  const credentials = grpc.credentials.createInsecure();
  
  return new ServiceClient(address, credentials, {
    interceptors: [
      loggingInterceptor,
      authInterceptor(token),
    ],
  });
}

// Server interceptor (using middleware pattern)
function serverAuthMiddleware(
  call: grpc.ServerUnaryCall<any, any>,
  callback: grpc.sendUnaryData<any>,
  next: () => void
): void {
  const metadata = call.metadata;
  const auth = metadata.get('authorization')[0] as string;
  
  if (!auth || !auth.startsWith('Bearer ')) {
    callback({
      code: grpc.status.UNAUTHENTICATED,
      message: 'Missing or invalid authorization',
    });
    return;
  }
  
  const token = auth.slice(7);
  
  // Validate token (simplified)
  if (token !== 'valid-token') {
    callback({
      code: grpc.status.PERMISSION_DENIED,
      message: 'Invalid token',
    });
    return;
  }
  
  // Add user info to call for handlers
  (call as any).user = { id: 'user123' };
  
  next();
}

export { loggingInterceptor, authInterceptor, createClientWithInterceptors };
```

---

## Real-World Scenarios

### Scenario 1: gRPC-Web for Browser Support

```typescript
// grpc/grpc-web.ts

/**
 * Browsers can't make native gRPC calls.
 * Solutions:
 * 1. gRPC-Web (requires proxy like Envoy)
 * 2. Connect protocol (native browser support)
 * 3. REST gateway (grpc-gateway)
 */

// Envoy configuration for gRPC-Web
const envoyConfig = `
static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 8080 }
    filter_chains:
    - filters:
      - name: envoy.filters.network.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
          codec_type: auto
          stat_prefix: ingress_http
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match: { prefix: "/" }
                route:
                  cluster: grpc_service
                  timeout: 0s
                  max_stream_duration:
                    grpc_timeout_header_max: 0s
              cors:
                allow_origin_string_match:
                - prefix: "*"
                allow_methods: GET, PUT, DELETE, POST, OPTIONS
                allow_headers: keep-alive,user-agent,cache-control,content-type,content-transfer-encoding,custom-header-1,x-accept-content-transfer-encoding,x-accept-response-streaming,x-user-agent,x-grpc-web,grpc-timeout
                expose_headers: custom-header-1,grpc-status,grpc-message
          http_filters:
          - name: envoy.filters.http.grpc_web
          - name: envoy.filters.http.cors
          - name: envoy.filters.http.router
  clusters:
  - name: grpc_service
    connect_timeout: 0.25s
    type: logical_dns
    http2_protocol_options: {}
    lb_policy: round_robin
    load_assignment:
      cluster_name: cluster_0
      endpoints:
        - lb_endpoints:
            - endpoint:
                address:
                  socket_address:
                    address: grpc-server
                    port_value: 50051
`;

// Browser client using gRPC-Web
/*
import { UserServiceClient } from './generated/user_grpc_web_pb';
import { GetUserRequest } from './generated/user_pb';

const client = new UserServiceClient('http://localhost:8080');

const request = new GetUserRequest();
request.setId('user123');

client.getUser(request, {}, (err, response) => {
  if (err) {
    console.error('Error:', err.message);
    return;
  }
  console.log('User:', response.toObject());
});
*/
```

### Scenario 2: Load Balancing with gRPC

```typescript
// grpc/load-balancing.ts
import * as grpc from '@grpc/grpc-js';

/**
 * gRPC Load Balancing:
 * 
 * 1. Client-side (grpc built-in)
 *    - round_robin
 *    - pick_first (default)
 * 
 * 2. Proxy-based (Envoy, Nginx)
 *    - More control
 *    - Health checking
 *    - Circuit breaking
 */

// Client-side load balancing
function createLoadBalancedClient(
  ServiceClient: any,
  addresses: string[]
): any {
  // DNS-based discovery
  const target = `dns:///my-service.default.svc.cluster.local:50051`;
  
  // Or static list
  // const target = `static:///${addresses.join(',')}`;
  
  const credentials = grpc.credentials.createInsecure();
  
  return new ServiceClient(target, credentials, {
    // Use round-robin load balancing
    'grpc.service_config': JSON.stringify({
      loadBalancingConfig: [{ round_robin: {} }],
    }),
    // Enable retries
    'grpc.enable_retries': 1,
  });
}

// With service config for retries
const serviceConfig = {
  loadBalancingConfig: [{ round_robin: {} }],
  methodConfig: [{
    name: [{ service: 'user.UserService' }],
    retryPolicy: {
      maxAttempts: 3,
      initialBackoff: '0.1s',
      maxBackoff: '1s',
      backoffMultiplier: 2,
      retryableStatusCodes: ['UNAVAILABLE', 'DEADLINE_EXCEEDED'],
    },
  }],
};
```

---

## Common Pitfalls

### 1. Not Setting Deadlines

```typescript
// ❌ BAD: No deadline
client.getUser(request, (err, response) => {
  // Request may hang forever!
});

// ✅ GOOD: Always set deadline
const deadline = new Date(Date.now() + 5000);
client.getUser(request, { deadline }, (err, response) => {
  // Will timeout after 5 seconds
});
```

### 2. Ignoring Backpressure in Streams

```typescript
// ❌ BAD: Writing without checking
for (const item of largeArray) {
  stream.write(item); // May overflow buffer!
}

// ✅ GOOD: Respect backpressure
async function writeWithBackpressure(
  stream: grpc.ClientWritableStream<any>,
  items: any[]
): Promise<void> {
  for (const item of items) {
    const canContinue = stream.write(item);
    if (!canContinue) {
      await new Promise(resolve => stream.once('drain', resolve));
    }
  }
  stream.end();
}
```

### 3. Using REST Patterns in gRPC

```protobuf
// ❌ BAD: REST-style resource paths in gRPC
service UserService {
  rpc GetUserById(GetUserByIdRequest) returns (User);
  rpc GetUserByEmail(GetUserByEmailRequest) returns (User);
  rpc GetUserByName(GetUserByNameRequest) returns (User);
}

// ✅ GOOD: Single method with query options
service UserService {
  rpc GetUser(GetUserRequest) returns (User);
}

message GetUserRequest {
  oneof identifier {
    string id = 1;
    string email = 2;
    string name = 3;
  }
}
```

---

## Interview Questions

### Q1: How does gRPC differ from REST?

**A:** gRPC uses HTTP/2 (multiplexing, streaming) vs HTTP/1.1. Protocol Buffers (binary, typed) vs JSON (text, dynamic). Code generation from .proto vs manual client/server code. Built-in streaming vs long-polling/WebSocket. Better for internal microservices; REST better for public APIs due to browser support.

### Q2: What are the four gRPC streaming types?

**A:** 1) **Unary**: single request, single response (like REST). 2) **Server streaming**: single request, stream of responses (real-time updates). 3) **Client streaming**: stream of requests, single response (batch upload). 4) **Bidirectional**: both sides stream independently (chat, gaming).

### Q3: What are gRPC deadlines and how do they differ from timeouts?

**A:** Deadlines are absolute timestamps when a request must complete. Timeouts are relative durations. Deadlines propagate across service calls - if service A calls B with 5s deadline, and A already used 2s, B gets 3s remaining. This prevents cascading timeouts in microservices.

### Q4: How do you handle versioning in gRPC?

**A:** 1) Add new fields (backward compatible - old clients ignore them). 2) Never reuse field numbers. 3) Deprecate fields instead of removing. 4) Use oneof for mutually exclusive fields. 5) Version the package/service name for breaking changes (`user.v2.UserService`).

---

## Quick Reference Checklist

### Proto Design
- [ ] Use proto3 syntax
- [ ] Define clear service and message boundaries
- [ ] Use well-known types (Timestamp, Empty)
- [ ] Document with comments
- [ ] Never reuse field numbers

### Server Implementation
- [ ] Handle all RPC types you define
- [ ] Return appropriate status codes
- [ ] Check deadlines in long operations
- [ ] Clean up resources on stream end/cancel

### Client Implementation
- [ ] Always set deadlines
- [ ] Handle all status codes
- [ ] Use interceptors for cross-cutting concerns
- [ ] Implement retry with backoff

### Production
- [ ] Use TLS in production
- [ ] Set up health checking
- [ ] Configure load balancing
- [ ] Monitor with metrics/tracing

---

*Last updated: February 2026*

