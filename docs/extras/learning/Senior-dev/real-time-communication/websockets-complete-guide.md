# 🔌 WebSockets Deep Dive - Complete Guide

> A comprehensive guide to WebSockets - native API, Socket.io, scaling, reconnection strategies, heartbeats, authentication, and production patterns.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "WebSockets provide full-duplex, persistent connections over a single TCP connection, enabling real-time bidirectional communication between client and server without the overhead of HTTP request/response cycles."

### The 6 Key Concepts (Remember These!)
```
1. FULL-DUPLEX      → Both sides can send/receive simultaneously
2. PERSISTENT       → Connection stays open (no reconnection overhead)
3. UPGRADE          → Starts as HTTP, upgrades to WebSocket protocol
4. FRAMES           → Data sent in frames (text, binary, control)
5. HEARTBEAT        → Ping/pong to detect dead connections
6. SCALING          → Requires sticky sessions or pub/sub adapter
```

### WebSocket vs HTTP Comparison
```
┌─────────────────────────────────────────────────────────────────┐
│                    HTTP vs WebSocket                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  HTTP (Request/Response):                                       │
│  ┌────────┐         ┌────────┐                                 │
│  │ Client │ ──REQ──▶│ Server │                                 │
│  │        │◀──RES── │        │                                 │
│  └────────┘         └────────┘                                 │
│  • New connection each request                                  │
│  • Headers sent every time (~800 bytes)                         │
│  • Server can't push data                                       │
│                                                                 │
│  WebSocket (Full-Duplex):                                       │
│  ┌────────┐ ══════════════ ┌────────┐                          │
│  │ Client │◀═══ DATA ════▶│ Server │                           │
│  └────────┘ ══════════════ └────────┘                          │
│  • Single persistent connection                                 │
│  • Minimal overhead per message (~2-6 bytes)                    │
│  • Server can push anytime                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Upgrade handshake"** | "The WebSocket connection starts with an HTTP upgrade handshake" |
| **"Full-duplex"** | "Unlike HTTP, WebSockets are full-duplex" |
| **"Sticky sessions"** | "We use sticky sessions for WebSocket load balancing" |
| **"Redis adapter"** | "Socket.io scales horizontally with the Redis adapter" |
| **"Heartbeat/Ping-pong"** | "We detect stale connections with heartbeat ping-pong" |
| **"Backpressure"** | "We implement backpressure to handle slow consumers" |
| **"Binary frames"** | "For efficiency, we send binary frames instead of text" |
| **"Reconnection with backoff"** | "Clients reconnect with exponential backoff" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Frame overhead | **2-14 bytes** | vs ~800 bytes HTTP headers |
| Ping interval | **25-30 seconds** | Below typical proxy timeout |
| Reconnect base | **1-2 seconds** | Initial reconnection delay |
| Max reconnect | **30 seconds** | Cap exponential backoff |
| Max connections/server | **~65K** | Limited by ports (tunable) |
| Browser limit | **~255 per domain** | Varies by browser |

### WebSocket Frame Types
| Opcode | Type | Purpose |
|--------|------|---------|
| 0x1 | Text | UTF-8 text data |
| 0x2 | Binary | Binary data |
| 0x8 | Close | Initiate close |
| 0x9 | Ping | Heartbeat request |
| 0xA | Pong | Heartbeat response |

### The "Wow" Statement (Memorize This!)
> "I architected our real-time system using Socket.io with Redis adapter for horizontal scaling. The native WebSocket API is great for simple cases, but Socket.io adds automatic reconnection with exponential backoff, room-based broadcasting, acknowledgments, and graceful fallback to long-polling. For authentication, we verify JWT during the handshake middleware and attach user context to the socket. We implement heartbeats at 25-second intervals to detect zombie connections before proxy timeouts. For high-throughput scenarios, we use binary frames with MessagePack serialization, reducing payload size by 30-50% compared to JSON."

### Quick Architecture Drawing (Draw This!)
```
                    ┌─────────────────────────────────┐
                    │         Load Balancer           │
                    │    (sticky sessions/IP hash)    │
                    └────────────┬────────────────────┘
                                 │
           ┌─────────────────────┼─────────────────────┐
           │                     │                     │
           ▼                     ▼                     ▼
    ┌─────────────┐       ┌─────────────┐       ┌─────────────┐
    │  Server 1   │       │  Server 2   │       │  Server 3   │
    │  Socket.io  │       │  Socket.io  │       │  Socket.io  │
    └──────┬──────┘       └──────┬──────┘       └──────┬──────┘
           │                     │                     │
           └─────────────────────┼─────────────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │      Redis Pub/Sub      │
                    │   (adapter for scaling) │
                    └─────────────────────────┘
```

### Interview Rapid Fire (Practice These!)

**Q: "What is a WebSocket?"**
> "A protocol providing full-duplex communication over a single TCP connection. Starts as HTTP upgrade, then bidirectional messaging."

**Q: "WebSocket vs HTTP?"**
> "WebSocket: persistent, bidirectional, low overhead. HTTP: request/response, new connection each time, higher overhead."

**Q: "How do you scale WebSockets?"**
> "Sticky sessions at load balancer, plus Redis pub/sub adapter to broadcast across servers."

**Q: "How do you handle disconnections?"**
> "Heartbeat ping/pong to detect dead connections, exponential backoff for reconnection, state reconciliation on reconnect."

**Q: "How do you authenticate WebSockets?"**
> "JWT in handshake query or auth event. Verify in middleware before connection completes. Reject invalid tokens immediately."

**Q: "What's the difference between Socket.io and native WebSocket?"**
> "Socket.io adds: reconnection, rooms, acknowledgments, fallback transports, and multiplexing. More features, slightly more overhead."

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "What are WebSockets?"

**Junior Answer:**
> "WebSockets let you send messages between client and server in real-time."

**Senior Answer:**
> "WebSockets are a protocol that upgrades an HTTP connection to a persistent, full-duplex channel. Unlike HTTP's request-response model, both client and server can send messages at any time without waiting.

The connection starts with an HTTP GET with `Upgrade: websocket` header. If the server supports it, it responds with 101 Switching Protocols, and from that point, both sides communicate using WebSocket frames.

Key advantages:
1. **Low latency** - No connection establishment overhead per message
2. **Low overhead** - Frame headers are 2-14 bytes vs ~800 bytes HTTP headers
3. **Server push** - Server can send data without client requesting
4. **True bidirectional** - Both sides send independently

The tradeoffs are complexity in scaling (need sticky sessions or pub/sub), handling disconnections gracefully, and ensuring proper cleanup of resources."

### When Asked: "How do you scale WebSockets?"

**Junior Answer:**
> "Add more servers behind a load balancer."

**Senior Answer:**
> "WebSocket scaling is trickier than stateless HTTP because connections are persistent and stateful. You need to solve two problems:

**1. Connection Affinity**: A client's WebSocket must stay connected to the same server. Solutions:
- Sticky sessions via IP hash or cookie
- Layer 4 (TCP) load balancing instead of Layer 7

**2. Cross-Server Communication**: When User A on Server 1 sends a message to User B on Server 2:
- Use a pub/sub backbone like Redis
- Socket.io's Redis adapter handles this automatically
- Each server subscribes to channels, broadcasts are replicated

Additional considerations:
- Connection limits per server (~65K ports, but can tune)
- Graceful draining for deployments
- Health checks that account for WebSocket connections"

### Follow-up Questions to Expect:

| They Ask | You Answer |
|----------|------------|
| "What about connection limits?" | "~65K per server by default, tune with SO_REUSEADDR and ulimit. Also consider memory - each connection has buffers." |
| "How do you deploy without dropping connections?" | "Graceful drain: stop accepting new connections, wait for existing to close or migrate, then shutdown." |
| "What if Redis goes down?" | "Connections stay active but can't cross-server broadcast. Use Redis Cluster or Sentinel for HA." |
| "How do you debug WebSocket issues?" | "Chrome DevTools Network tab shows WS frames. Server-side logging of connect/disconnect/message events." |

---

## 📚 Table of Contents

1. [Native WebSocket API](#1-native-websocket-api)
2. [Socket.io Implementation](#2-socketio-implementation)
3. [Authentication & Security](#3-authentication--security)
4. [Scaling WebSockets](#4-scaling-websockets)
5. [Reconnection Strategies](#5-reconnection-strategies)
6. [Heartbeat & Connection Health](#6-heartbeat--connection-health)
7. [Performance Optimization](#7-performance-optimization)
8. [Real-World Patterns](#8-real-world-patterns)
9. [Common Pitfalls](#9-common-pitfalls)
10. [Interview Questions](#10-interview-questions)

---

## 1. Native WebSocket API

### Browser Client

```typescript
// ════════════════════════════════════════════════════════════════
// NATIVE WEBSOCKET CLIENT
// ════════════════════════════════════════════════════════════════

class WebSocketClient {
  private ws: WebSocket | null = null;
  private url: string;
  private reconnectAttempts = 0;
  private maxReconnectAttempts = 10;
  private reconnectDelay = 1000;
  private maxReconnectDelay = 30000;
  private messageHandlers: Map<string, Function[]> = new Map();

  constructor(url: string) {
    this.url = url;
  }

  connect(): Promise<void> {
    return new Promise((resolve, reject) => {
      try {
        this.ws = new WebSocket(this.url);

        this.ws.onopen = () => {
          console.log('WebSocket connected');
          this.reconnectAttempts = 0;
          this.reconnectDelay = 1000;
          resolve();
        };

        this.ws.onmessage = (event) => {
          this.handleMessage(event.data);
        };

        this.ws.onerror = (error) => {
          console.error('WebSocket error:', error);
          reject(error);
        };

        this.ws.onclose = (event) => {
          console.log(`WebSocket closed: ${event.code} ${event.reason}`);
          this.handleClose(event);
        };

      } catch (error) {
        reject(error);
      }
    });
  }

  private handleMessage(data: string) {
    try {
      const message = JSON.parse(data);
      const { type, payload } = message;

      const handlers = this.messageHandlers.get(type) || [];
      handlers.forEach(handler => handler(payload));
    } catch (error) {
      console.error('Failed to parse message:', error);
    }
  }

  private handleClose(event: CloseEvent) {
    // Don't reconnect for normal closure
    if (event.code === 1000) return;

    this.attemptReconnect();
  }

  private attemptReconnect() {
    if (this.reconnectAttempts >= this.maxReconnectAttempts) {
      console.error('Max reconnection attempts reached');
      return;
    }

    this.reconnectAttempts++;
    
    // Exponential backoff with jitter
    const jitter = Math.random() * 1000;
    const delay = Math.min(
      this.reconnectDelay * Math.pow(2, this.reconnectAttempts - 1) + jitter,
      this.maxReconnectDelay
    );

    console.log(`Reconnecting in ${delay}ms (attempt ${this.reconnectAttempts})`);

    setTimeout(() => {
      this.connect().catch(() => {
        // Will trigger another reconnect attempt via onclose
      });
    }, delay);
  }

  send(type: string, payload: any): void {
    if (this.ws?.readyState !== WebSocket.OPEN) {
      throw new Error('WebSocket is not connected');
    }

    this.ws.send(JSON.stringify({ type, payload }));
  }

  on(type: string, handler: Function): void {
    const handlers = this.messageHandlers.get(type) || [];
    handlers.push(handler);
    this.messageHandlers.set(type, handlers);
  }

  off(type: string, handler: Function): void {
    const handlers = this.messageHandlers.get(type) || [];
    const index = handlers.indexOf(handler);
    if (index > -1) {
      handlers.splice(index, 1);
    }
  }

  disconnect(): void {
    if (this.ws) {
      this.ws.close(1000, 'Client disconnecting');
      this.ws = null;
    }
  }

  get isConnected(): boolean {
    return this.ws?.readyState === WebSocket.OPEN;
  }
}

// Usage
const client = new WebSocketClient('wss://api.example.com/ws');

client.on('chat:message', (data) => {
  console.log('New message:', data);
});

client.on('user:typing', (data) => {
  console.log('User typing:', data.userId);
});

await client.connect();
client.send('chat:message', { text: 'Hello!', roomId: '123' });
```

### Node.js Server with 'ws' Library

```typescript
// ════════════════════════════════════════════════════════════════
// NATIVE WEBSOCKET SERVER (Node.js with 'ws')
// ════════════════════════════════════════════════════════════════

import { WebSocketServer, WebSocket } from 'ws';
import { createServer } from 'http';
import { parse } from 'url';

interface ExtendedWebSocket extends WebSocket {
  id: string;
  userId?: string;
  isAlive: boolean;
  rooms: Set<string>;
}

class WebSocketManager {
  private wss: WebSocketServer;
  private clients: Map<string, ExtendedWebSocket> = new Map();
  private rooms: Map<string, Set<string>> = new Map(); // roomId -> clientIds
  private heartbeatInterval: NodeJS.Timeout | null = null;

  constructor(server: ReturnType<typeof createServer>) {
    this.wss = new WebSocketServer({ server });
    this.setupConnectionHandler();
    this.startHeartbeat();
  }

  private setupConnectionHandler() {
    this.wss.on('connection', (ws: WebSocket, request) => {
      const extWs = ws as ExtendedWebSocket;
      extWs.id = this.generateId();
      extWs.isAlive = true;
      extWs.rooms = new Set();

      // Parse auth from query string
      const { query } = parse(request.url || '', true);
      const token = query.token as string;

      // Authenticate
      try {
        const user = this.verifyToken(token);
        extWs.userId = user.id;
      } catch (error) {
        ws.close(4001, 'Unauthorized');
        return;
      }

      this.clients.set(extWs.id, extWs);
      console.log(`Client connected: ${extWs.id} (user: ${extWs.userId})`);

      // Handle pong (response to our ping)
      ws.on('pong', () => {
        extWs.isAlive = true;
      });

      ws.on('message', (data) => {
        this.handleMessage(extWs, data.toString());
      });

      ws.on('close', () => {
        this.handleDisconnect(extWs);
      });

      ws.on('error', (error) => {
        console.error(`WebSocket error for ${extWs.id}:`, error);
      });

      // Send welcome message
      this.sendTo(extWs, 'connected', { clientId: extWs.id });
    });
  }

  private handleMessage(client: ExtendedWebSocket, data: string) {
    try {
      const { type, payload } = JSON.parse(data);

      switch (type) {
        case 'join:room':
          this.joinRoom(client, payload.roomId);
          break;
        case 'leave:room':
          this.leaveRoom(client, payload.roomId);
          break;
        case 'room:message':
          this.broadcastToRoom(payload.roomId, 'room:message', {
            ...payload,
            from: client.userId
          }, client.id);
          break;
        case 'direct:message':
          this.sendToUser(payload.toUserId, 'direct:message', {
            ...payload,
            from: client.userId
          });
          break;
        default:
          console.log(`Unknown message type: ${type}`);
      }
    } catch (error) {
      console.error('Failed to handle message:', error);
    }
  }

  private handleDisconnect(client: ExtendedWebSocket) {
    // Leave all rooms
    client.rooms.forEach(roomId => {
      this.leaveRoom(client, roomId);
    });

    this.clients.delete(client.id);
    console.log(`Client disconnected: ${client.id}`);
  }

  // ════════════════════════════════════════════════════════════════
  // ROOM MANAGEMENT
  // ════════════════════════════════════════════════════════════════

  joinRoom(client: ExtendedWebSocket, roomId: string) {
    if (!this.rooms.has(roomId)) {
      this.rooms.set(roomId, new Set());
    }
    this.rooms.get(roomId)!.add(client.id);
    client.rooms.add(roomId);

    // Notify room members
    this.broadcastToRoom(roomId, 'user:joined', {
      userId: client.userId,
      roomId
    });
  }

  leaveRoom(client: ExtendedWebSocket, roomId: string) {
    const room = this.rooms.get(roomId);
    if (room) {
      room.delete(client.id);
      if (room.size === 0) {
        this.rooms.delete(roomId);
      }
    }
    client.rooms.delete(roomId);

    // Notify room members
    this.broadcastToRoom(roomId, 'user:left', {
      userId: client.userId,
      roomId
    });
  }

  // ════════════════════════════════════════════════════════════════
  // MESSAGING
  // ════════════════════════════════════════════════════════════════

  sendTo(client: ExtendedWebSocket, type: string, payload: any) {
    if (client.readyState === WebSocket.OPEN) {
      client.send(JSON.stringify({ type, payload }));
    }
  }

  sendToUser(userId: string, type: string, payload: any) {
    for (const client of this.clients.values()) {
      if (client.userId === userId) {
        this.sendTo(client, type, payload);
      }
    }
  }

  broadcastToRoom(roomId: string, type: string, payload: any, excludeClientId?: string) {
    const room = this.rooms.get(roomId);
    if (!room) return;

    room.forEach(clientId => {
      if (clientId !== excludeClientId) {
        const client = this.clients.get(clientId);
        if (client) {
          this.sendTo(client, type, payload);
        }
      }
    });
  }

  broadcast(type: string, payload: any, excludeClientId?: string) {
    this.clients.forEach((client, clientId) => {
      if (clientId !== excludeClientId) {
        this.sendTo(client, type, payload);
      }
    });
  }

  // ════════════════════════════════════════════════════════════════
  // HEARTBEAT
  // ════════════════════════════════════════════════════════════════

  private startHeartbeat() {
    this.heartbeatInterval = setInterval(() => {
      this.clients.forEach((client, clientId) => {
        if (!client.isAlive) {
          console.log(`Terminating inactive client: ${clientId}`);
          client.terminate();
          this.clients.delete(clientId);
          return;
        }

        client.isAlive = false;
        client.ping();
      });
    }, 30000); // 30 seconds
  }

  // ════════════════════════════════════════════════════════════════
  // UTILITIES
  // ════════════════════════════════════════════════════════════════

  private generateId(): string {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }

  private verifyToken(token: string): { id: string } {
    // Implement your JWT verification logic here
    // This is a placeholder
    if (!token) throw new Error('No token provided');
    return { id: 'user-123' }; // Return decoded user
  }

  getStats() {
    return {
      totalConnections: this.clients.size,
      totalRooms: this.rooms.size,
      roomSizes: Object.fromEntries(
        Array.from(this.rooms.entries()).map(([id, members]) => [id, members.size])
      )
    };
  }

  shutdown() {
    if (this.heartbeatInterval) {
      clearInterval(this.heartbeatInterval);
    }

    this.clients.forEach(client => {
      client.close(1001, 'Server shutting down');
    });
  }
}

// Usage
const server = createServer();
const wsManager = new WebSocketManager(server);

server.listen(3000, () => {
  console.log('Server listening on port 3000');
});

// Graceful shutdown
process.on('SIGTERM', () => {
  wsManager.shutdown();
  server.close();
});
```

---

## 2. Socket.io Implementation

### Server Setup

```typescript
// ════════════════════════════════════════════════════════════════
// SOCKET.IO SERVER
// ════════════════════════════════════════════════════════════════

import { Server, Socket } from 'socket.io';
import { createServer } from 'http';
import { createAdapter } from '@socket.io/redis-adapter';
import { createClient } from 'redis';

interface ServerToClientEvents {
  'chat:message': (data: { userId: string; text: string; timestamp: Date }) => void;
  'user:joined': (data: { userId: string; roomId: string }) => void;
  'user:left': (data: { userId: string; roomId: string }) => void;
  'typing:start': (data: { userId: string }) => void;
  'typing:stop': (data: { userId: string }) => void;
  'error': (data: { message: string; code: string }) => void;
}

interface ClientToServerEvents {
  'chat:send': (data: { roomId: string; text: string }, callback: (ack: { success: boolean }) => void) => void;
  'room:join': (roomId: string, callback: (ack: { success: boolean; members: string[] }) => void) => void;
  'room:leave': (roomId: string) => void;
  'typing:start': (roomId: string) => void;
  'typing:stop': (roomId: string) => void;
}

interface InterServerEvents {
  ping: () => void;
}

interface SocketData {
  userId: string;
  username: string;
  joinedAt: Date;
}

const httpServer = createServer();
const io = new Server<
  ClientToServerEvents,
  ServerToClientEvents,
  InterServerEvents,
  SocketData
>(httpServer, {
  cors: {
    origin: process.env.CORS_ORIGIN || 'http://localhost:3000',
    methods: ['GET', 'POST'],
    credentials: true
  },
  pingInterval: 25000,  // Send ping every 25 seconds
  pingTimeout: 20000,   // Wait 20 seconds for pong
  transports: ['websocket', 'polling'], // Prefer WebSocket
  allowUpgrades: true
});

// ════════════════════════════════════════════════════════════════
// REDIS ADAPTER FOR SCALING
// ════════════════════════════════════════════════════════════════

async function setupRedisAdapter() {
  const pubClient = createClient({ url: process.env.REDIS_URL });
  const subClient = pubClient.duplicate();

  await Promise.all([pubClient.connect(), subClient.connect()]);

  io.adapter(createAdapter(pubClient, subClient));
  console.log('Redis adapter connected');
}

// ════════════════════════════════════════════════════════════════
// AUTHENTICATION MIDDLEWARE
// ════════════════════════════════════════════════════════════════

io.use(async (socket, next) => {
  try {
    const token = socket.handshake.auth.token || socket.handshake.query.token;
    
    if (!token) {
      return next(new Error('Authentication required'));
    }

    // Verify JWT
    const decoded = await verifyJWT(token as string);
    
    // Attach user data to socket
    socket.data.userId = decoded.userId;
    socket.data.username = decoded.username;
    socket.data.joinedAt = new Date();

    next();
  } catch (error) {
    next(new Error('Invalid token'));
  }
});

// ════════════════════════════════════════════════════════════════
// CONNECTION HANDLER
// ════════════════════════════════════════════════════════════════

io.on('connection', (socket) => {
  const { userId, username } = socket.data;
  console.log(`User connected: ${userId} (${socket.id})`);

  // Join user's personal room (for direct messages)
  socket.join(`user:${userId}`);

  // ══════════════════════════════════════════════════════════════
  // ROOM MANAGEMENT
  // ══════════════════════════════════════════════════════════════

  socket.on('room:join', async (roomId, callback) => {
    try {
      // Check if user can join this room (authorization)
      const canJoin = await checkRoomAccess(userId, roomId);
      if (!canJoin) {
        return callback({ success: false, members: [] });
      }

      // Join the room
      await socket.join(roomId);

      // Get current members
      const sockets = await io.in(roomId).fetchSockets();
      const members = sockets.map(s => s.data.userId);

      // Notify others in the room
      socket.to(roomId).emit('user:joined', { userId, roomId });

      callback({ success: true, members });
    } catch (error) {
      callback({ success: false, members: [] });
    }
  });

  socket.on('room:leave', (roomId) => {
    socket.leave(roomId);
    socket.to(roomId).emit('user:left', { userId, roomId });
  });

  // ══════════════════════════════════════════════════════════════
  // CHAT MESSAGES
  // ══════════════════════════════════════════════════════════════

  socket.on('chat:send', async (data, callback) => {
    try {
      const { roomId, text } = data;

      // Validate message
      if (!text || text.trim().length === 0) {
        return callback({ success: false });
      }

      // Save to database
      const message = await saveMessage({
        roomId,
        userId,
        text: text.trim(),
        timestamp: new Date()
      });

      // Broadcast to room (including sender)
      io.to(roomId).emit('chat:message', {
        userId,
        text: message.text,
        timestamp: message.timestamp
      });

      callback({ success: true });
    } catch (error) {
      callback({ success: false });
    }
  });

  // ══════════════════════════════════════════════════════════════
  // TYPING INDICATORS
  // ══════════════════════════════════════════════════════════════

  socket.on('typing:start', (roomId) => {
    socket.to(roomId).emit('typing:start', { userId });
  });

  socket.on('typing:stop', (roomId) => {
    socket.to(roomId).emit('typing:stop', { userId });
  });

  // ══════════════════════════════════════════════════════════════
  // DISCONNECTION
  // ══════════════════════════════════════════════════════════════

  socket.on('disconnect', (reason) => {
    console.log(`User disconnected: ${userId} (${reason})`);
    
    // Notify all rooms the user was in
    socket.rooms.forEach(roomId => {
      if (roomId !== socket.id && !roomId.startsWith('user:')) {
        socket.to(roomId).emit('user:left', { userId, roomId });
      }
    });
  });
});

// ════════════════════════════════════════════════════════════════
// UTILITY FUNCTIONS
// ════════════════════════════════════════════════════════════════

// Send to specific user (all their connections)
function sendToUser(userId: string, event: keyof ServerToClientEvents, data: any) {
  io.to(`user:${userId}`).emit(event, data);
}

// Get online users in a room
async function getRoomMembers(roomId: string): Promise<string[]> {
  const sockets = await io.in(roomId).fetchSockets();
  return [...new Set(sockets.map(s => s.data.userId))];
}

// Get all rooms a user is in
function getUserRooms(socket: Socket): string[] {
  return Array.from(socket.rooms).filter(
    room => room !== socket.id && !room.startsWith('user:')
  );
}

// Broadcast to all connected clients
function broadcastAll(event: keyof ServerToClientEvents, data: any) {
  io.emit(event, data);
}

// Start server
async function start() {
  await setupRedisAdapter();
  
  httpServer.listen(3000, () => {
    console.log('Socket.io server running on port 3000');
  });
}

start();
```

### Client Setup

```typescript
// ════════════════════════════════════════════════════════════════
// SOCKET.IO CLIENT
// ════════════════════════════════════════════════════════════════

import { io, Socket } from 'socket.io-client';

interface ServerToClientEvents {
  'chat:message': (data: { userId: string; text: string; timestamp: Date }) => void;
  'user:joined': (data: { userId: string; roomId: string }) => void;
  'user:left': (data: { userId: string; roomId: string }) => void;
  'typing:start': (data: { userId: string }) => void;
  'typing:stop': (data: { userId: string }) => void;
  'error': (data: { message: string; code: string }) => void;
}

interface ClientToServerEvents {
  'chat:send': (data: { roomId: string; text: string }, callback: (ack: { success: boolean }) => void) => void;
  'room:join': (roomId: string, callback: (ack: { success: boolean; members: string[] }) => void) => void;
  'room:leave': (roomId: string) => void;
  'typing:start': (roomId: string) => void;
  'typing:stop': (roomId: string) => void;
}

class SocketClient {
  private socket: Socket<ServerToClientEvents, ClientToServerEvents> | null = null;
  private token: string;
  private url: string;

  constructor(url: string, token: string) {
    this.url = url;
    this.token = token;
  }

  connect(): Promise<void> {
    return new Promise((resolve, reject) => {
      this.socket = io(this.url, {
        auth: { token: this.token },
        transports: ['websocket', 'polling'],
        reconnection: true,
        reconnectionAttempts: 10,
        reconnectionDelay: 1000,
        reconnectionDelayMax: 30000,
        timeout: 20000
      });

      this.socket.on('connect', () => {
        console.log('Connected to server');
        resolve();
      });

      this.socket.on('connect_error', (error) => {
        console.error('Connection error:', error.message);
        if (error.message === 'Invalid token') {
          // Handle token refresh or re-authentication
          this.handleAuthError();
        }
        reject(error);
      });

      this.socket.on('disconnect', (reason) => {
        console.log('Disconnected:', reason);
        if (reason === 'io server disconnect') {
          // Server disconnected us, need to reconnect manually
          this.socket?.connect();
        }
        // Otherwise, socket.io will auto-reconnect
      });

      this.setupEventHandlers();
    });
  }

  private setupEventHandlers() {
    if (!this.socket) return;

    this.socket.on('chat:message', (data) => {
      console.log('Message received:', data);
      // Handle incoming message
    });

    this.socket.on('user:joined', (data) => {
      console.log('User joined:', data);
    });

    this.socket.on('user:left', (data) => {
      console.log('User left:', data);
    });

    this.socket.on('typing:start', (data) => {
      console.log('User typing:', data.userId);
    });

    this.socket.on('typing:stop', (data) => {
      console.log('User stopped typing:', data.userId);
    });

    this.socket.on('error', (data) => {
      console.error('Server error:', data);
    });
  }

  private handleAuthError() {
    // Implement token refresh logic
    console.log('Auth error - need to refresh token');
  }

  // ══════════════════════════════════════════════════════════════
  // PUBLIC METHODS
  // ══════════════════════════════════════════════════════════════

  async joinRoom(roomId: string): Promise<{ success: boolean; members: string[] }> {
    return new Promise((resolve) => {
      this.socket?.emit('room:join', roomId, (response) => {
        resolve(response);
      });
    });
  }

  leaveRoom(roomId: string): void {
    this.socket?.emit('room:leave', roomId);
  }

  async sendMessage(roomId: string, text: string): Promise<boolean> {
    return new Promise((resolve) => {
      this.socket?.emit('chat:send', { roomId, text }, (response) => {
        resolve(response.success);
      });
    });
  }

  startTyping(roomId: string): void {
    this.socket?.emit('typing:start', roomId);
  }

  stopTyping(roomId: string): void {
    this.socket?.emit('typing:stop', roomId);
  }

  onMessage(handler: (data: { userId: string; text: string; timestamp: Date }) => void): void {
    this.socket?.on('chat:message', handler);
  }

  disconnect(): void {
    this.socket?.disconnect();
  }

  get isConnected(): boolean {
    return this.socket?.connected ?? false;
  }
}

// Usage
const client = new SocketClient('wss://api.example.com', 'jwt-token-here');

await client.connect();
const { members } = await client.joinRoom('room-123');
console.log('Room members:', members);

client.onMessage((message) => {
  console.log(`${message.userId}: ${message.text}`);
});

await client.sendMessage('room-123', 'Hello everyone!');
```

---

## 3. Authentication & Security

### JWT Authentication in Handshake

```typescript
// ════════════════════════════════════════════════════════════════
// SECURE WEBSOCKET AUTHENTICATION
// ════════════════════════════════════════════════════════════════

import jwt from 'jsonwebtoken';
import { Socket } from 'socket.io';

interface DecodedToken {
  userId: string;
  username: string;
  roles: string[];
  exp: number;
  iat: number;
}

// Middleware for initial authentication
io.use(async (socket: Socket, next) => {
  try {
    // Get token from multiple sources
    const token = 
      socket.handshake.auth.token ||           // Preferred: auth object
      socket.handshake.headers.authorization?.split(' ')[1] ||  // Bearer token
      socket.handshake.query.token;            // Query string (fallback)

    if (!token) {
      return next(new Error('AUTHENTICATION_REQUIRED'));
    }

    // Verify JWT
    const decoded = jwt.verify(
      token as string,
      process.env.JWT_SECRET!
    ) as DecodedToken;

    // Check if token is about to expire (refresh needed)
    const expiresIn = decoded.exp - Math.floor(Date.now() / 1000);
    if (expiresIn < 300) { // Less than 5 minutes
      socket.emit('token:expiring', { expiresIn });
    }

    // Check if user is banned or disabled
    const user = await db.users.findUnique({ 
      where: { id: decoded.userId } 
    });
    
    if (!user || user.disabled) {
      return next(new Error('USER_DISABLED'));
    }

    // Attach user data to socket
    socket.data.userId = decoded.userId;
    socket.data.username = decoded.username;
    socket.data.roles = decoded.roles;

    next();
  } catch (error) {
    if (error instanceof jwt.TokenExpiredError) {
      return next(new Error('TOKEN_EXPIRED'));
    }
    if (error instanceof jwt.JsonWebTokenError) {
      return next(new Error('INVALID_TOKEN'));
    }
    return next(new Error('AUTHENTICATION_FAILED'));
  }
});

// ════════════════════════════════════════════════════════════════
// TOKEN REFRESH MECHANISM
// ════════════════════════════════════════════════════════════════

io.on('connection', (socket) => {
  // Handle token refresh
  socket.on('token:refresh', async (newToken: string, callback) => {
    try {
      const decoded = jwt.verify(
        newToken,
        process.env.JWT_SECRET!
      ) as DecodedToken;

      // Verify it's the same user
      if (decoded.userId !== socket.data.userId) {
        return callback({ success: false, error: 'USER_MISMATCH' });
      }

      // Update socket data
      socket.data.roles = decoded.roles;

      callback({ success: true });
    } catch (error) {
      callback({ success: false, error: 'INVALID_TOKEN' });
    }
  });
});

// ════════════════════════════════════════════════════════════════
// AUTHORIZATION MIDDLEWARE FOR EVENTS
// ════════════════════════════════════════════════════════════════

// Role-based event authorization
function requireRole(...roles: string[]) {
  return (socket: Socket, next: (err?: Error) => void) => {
    const userRoles = socket.data.roles || [];
    const hasRole = roles.some(role => userRoles.includes(role));

    if (!hasRole) {
      return next(new Error('INSUFFICIENT_PERMISSIONS'));
    }
    next();
  };
}

// Apply to specific namespace
const adminNamespace = io.of('/admin');
adminNamespace.use(requireRole('admin', 'moderator'));

// ════════════════════════════════════════════════════════════════
// RATE LIMITING
// ════════════════════════════════════════════════════════════════

import { RateLimiterMemory } from 'rate-limiter-flexible';

const rateLimiter = new RateLimiterMemory({
  points: 100,    // Number of events
  duration: 60    // Per 60 seconds
});

const messageRateLimiter = new RateLimiterMemory({
  points: 20,     // Messages
  duration: 60    // Per 60 seconds
});

// Apply rate limiting middleware
io.use(async (socket, next) => {
  try {
    await rateLimiter.consume(socket.handshake.address);
    next();
  } catch {
    next(new Error('RATE_LIMIT_EXCEEDED'));
  }
});

// Rate limit specific events
function rateLimit(limiter: RateLimiterMemory) {
  return async (socket: Socket, args: any[], next: (err?: Error) => void) => {
    try {
      await limiter.consume(socket.data.userId);
      next();
    } catch {
      socket.emit('error', { 
        message: 'Rate limit exceeded', 
        code: 'RATE_LIMIT' 
      });
    }
  };
}

// Usage with socket.io middleware
socket.use(async ([event, ...args], next) => {
  if (event === 'chat:send') {
    try {
      await messageRateLimiter.consume(socket.data.userId);
      next();
    } catch {
      socket.emit('error', { 
        message: 'Too many messages', 
        code: 'MESSAGE_RATE_LIMIT' 
      });
    }
  } else {
    next();
  }
});

// ════════════════════════════════════════════════════════════════
// INPUT VALIDATION
// ════════════════════════════════════════════════════════════════

import { z } from 'zod';

const chatMessageSchema = z.object({
  roomId: z.string().uuid(),
  text: z.string().min(1).max(1000).trim()
});

const roomJoinSchema = z.object({
  roomId: z.string().uuid()
});

// Validation middleware
function validate<T>(schema: z.Schema<T>) {
  return (data: unknown): T => {
    const result = schema.safeParse(data);
    if (!result.success) {
      throw new Error(`Validation failed: ${result.error.message}`);
    }
    return result.data;
  };
}

// Usage in event handler
socket.on('chat:send', async (data, callback) => {
  try {
    const validated = validate(chatMessageSchema)(data);
    // Process validated data...
    callback({ success: true });
  } catch (error) {
    callback({ success: false, error: 'Invalid message format' });
  }
});
```

---

## 4. Scaling WebSockets

### Redis Adapter Deep Dive

```typescript
// ════════════════════════════════════════════════════════════════
// SOCKET.IO REDIS ADAPTER FOR HORIZONTAL SCALING
// ════════════════════════════════════════════════════════════════

import { Server } from 'socket.io';
import { createAdapter } from '@socket.io/redis-adapter';
import { createClient } from 'redis';
import { Emitter } from '@socket.io/redis-emitter';

// Basic setup
async function setupRedisAdapter(io: Server) {
  const pubClient = createClient({
    url: process.env.REDIS_URL,
    socket: {
      reconnectStrategy: (retries) => {
        if (retries > 10) {
          return new Error('Redis connection failed');
        }
        return Math.min(retries * 100, 3000);
      }
    }
  });

  const subClient = pubClient.duplicate();

  // Handle Redis errors
  pubClient.on('error', (err) => console.error('Redis Pub Error:', err));
  subClient.on('error', (err) => console.error('Redis Sub Error:', err));

  await Promise.all([pubClient.connect(), subClient.connect()]);

  io.adapter(createAdapter(pubClient, subClient, {
    key: 'socket.io',           // Redis key prefix
    requestsTimeout: 5000,      // Timeout for adapter requests
    publishOnSpecificResponseChannel: true  // Better for large clusters
  }));

  return { pubClient, subClient };
}

// ════════════════════════════════════════════════════════════════
// EMITTER FOR EXTERNAL SERVICES
// ════════════════════════════════════════════════════════════════

// Emit events from any service (API, workers, etc.)
async function createEmitter() {
  const redisClient = createClient({ url: process.env.REDIS_URL });
  await redisClient.connect();

  const emitter = new Emitter(redisClient, {
    key: 'socket.io'  // Must match adapter key
  });

  return emitter;
}

// Usage in external service (e.g., API server)
const emitter = await createEmitter();

// Broadcast to all clients
emitter.emit('notification', { message: 'Server update' });

// Emit to specific room
emitter.to('room:123').emit('chat:message', { text: 'Hello' });

// Emit to specific user
emitter.to(`user:${userId}`).emit('direct:message', { text: 'Hi' });

// ════════════════════════════════════════════════════════════════
// CLUSTER MODE WITH PM2
// ════════════════════════════════════════════════════════════════

// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'websocket-server',
    script: './dist/server.js',
    instances: 'max',           // Use all CPU cores
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      REDIS_URL: 'redis://localhost:6379'
    },
    // Graceful shutdown
    kill_timeout: 10000,
    wait_ready: true,
    listen_timeout: 10000
  }]
};

// Server code for graceful shutdown
process.on('SIGINT', async () => {
  console.log('Shutting down gracefully...');
  
  // Stop accepting new connections
  io.close(() => {
    console.log('All connections closed');
    process.exit(0);
  });

  // Force close after timeout
  setTimeout(() => {
    console.log('Forcing shutdown');
    process.exit(1);
  }, 10000);
});

// Signal ready for PM2
process.send?.('ready');

// ════════════════════════════════════════════════════════════════
// LOAD BALANCER CONFIGURATION (Nginx)
// ════════════════════════════════════════════════════════════════

/*
# nginx.conf for WebSocket load balancing

upstream websocket_servers {
    # IP hash for sticky sessions
    ip_hash;
    
    server ws1.example.com:3000;
    server ws2.example.com:3000;
    server ws3.example.com:3000;
    
    # Or use least connections
    # least_conn;
}

server {
    listen 443 ssl http2;
    server_name api.example.com;

    ssl_certificate /etc/ssl/certs/api.crt;
    ssl_certificate_key /etc/ssl/private/api.key;

    location /socket.io/ {
        proxy_pass http://websocket_servers;
        proxy_http_version 1.1;
        
        # WebSocket upgrade headers
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        # Forward client info
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
        
        # Buffering
        proxy_buffering off;
        proxy_cache off;
    }
}
*/

// ════════════════════════════════════════════════════════════════
// CONNECTION DISTRIBUTION MONITORING
// ════════════════════════════════════════════════════════════════

async function getClusterStats(io: Server) {
  // Get all sockets across all servers
  const sockets = await io.fetchSockets();

  // Get server-specific stats
  const serverStats = await io.serverSideEmit('getStats');

  return {
    totalConnections: sockets.length,
    serverCount: serverStats.length,
    distribution: serverStats.map((stats, i) => ({
      server: i,
      connections: stats.connections
    }))
  };
}

// Respond to stats request from other servers
io.on('getStats', (callback) => {
  callback({
    connections: io.engine.clientsCount,
    uptime: process.uptime(),
    memory: process.memoryUsage()
  });
});
```

### Kubernetes Deployment

```yaml
# ════════════════════════════════════════════════════════════════
# KUBERNETES DEPLOYMENT FOR WEBSOCKET SERVERS
# ════════════════════════════════════════════════════════════════

# websocket-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: websocket-server
spec:
  replicas: 3
  selector:
    matchLabels:
      app: websocket-server
  template:
    metadata:
      labels:
        app: websocket-server
    spec:
      containers:
        - name: websocket
          image: myapp/websocket-server:latest
          ports:
            - containerPort: 3000
          env:
            - name: REDIS_URL
              valueFrom:
                secretKeyRef:
                  name: redis-secret
                  key: url
          resources:
            requests:
              memory: "256Mi"
              cpu: "200m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          readinessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 15
            periodSeconds: 20
          # Graceful shutdown
          lifecycle:
            preStop:
              exec:
                command: ["/bin/sh", "-c", "sleep 10"]
      terminationGracePeriodSeconds: 30
---
# Service with session affinity
apiVersion: v1
kind: Service
metadata:
  name: websocket-service
spec:
  selector:
    app: websocket-server
  ports:
    - port: 80
      targetPort: 3000
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 3600
---
# Ingress with WebSocket support
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: websocket-ingress
  annotations:
    nginx.ingress.kubernetes.io/proxy-read-timeout: "3600"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "3600"
    nginx.ingress.kubernetes.io/upstream-hash-by: "$remote_addr"
    nginx.ingress.kubernetes.io/affinity: "cookie"
    nginx.ingress.kubernetes.io/session-cookie-name: "websocket-affinity"
    nginx.ingress.kubernetes.io/session-cookie-max-age: "3600"
spec:
  rules:
    - host: ws.example.com
      http:
        paths:
          - path: /socket.io
            pathType: Prefix
            backend:
              service:
                name: websocket-service
                port:
                  number: 80
```

---

## 5. Reconnection Strategies

```typescript
// ════════════════════════════════════════════════════════════════
// ROBUST RECONNECTION WITH STATE RECOVERY
// ════════════════════════════════════════════════════════════════

interface ConnectionState {
  rooms: string[];
  lastEventId: string;
  pendingMessages: Message[];
}

class RobustWebSocketClient {
  private socket: Socket | null = null;
  private state: ConnectionState = {
    rooms: [],
    lastEventId: '',
    pendingMessages: []
  };
  private reconnecting = false;

  constructor(
    private url: string,
    private getToken: () => Promise<string>
  ) {}

  async connect(): Promise<void> {
    const token = await this.getToken();

    this.socket = io(this.url, {
      auth: { token },
      // Reconnection settings
      reconnection: true,
      reconnectionAttempts: Infinity,  // Never give up
      reconnectionDelay: 1000,
      reconnectionDelayMax: 30000,
      randomizationFactor: 0.5,        // Jitter
      timeout: 20000,
      // Performance
      transports: ['websocket'],
      upgrade: false,                  // Skip polling
      forceNew: false,
      multiplex: true
    });

    this.setupEventHandlers();
    
    return new Promise((resolve, reject) => {
      this.socket!.once('connect', () => resolve());
      this.socket!.once('connect_error', reject);
    });
  }

  private setupEventHandlers() {
    if (!this.socket) return;

    this.socket.on('connect', () => {
      console.log('Connected');
      this.onReconnected();
    });

    this.socket.on('disconnect', (reason) => {
      console.log('Disconnected:', reason);
      
      if (reason === 'io server disconnect') {
        // Server kicked us, try to reconnect
        this.socket?.connect();
      }
      // Other reasons: socket.io handles reconnection
    });

    this.socket.on('reconnect_attempt', (attempt) => {
      console.log(`Reconnection attempt ${attempt}`);
      this.reconnecting = true;
    });

    this.socket.on('reconnect', (attempt) => {
      console.log(`Reconnected after ${attempt} attempts`);
    });

    this.socket.on('reconnect_error', (error) => {
      console.error('Reconnection error:', error);
    });

    this.socket.on('reconnect_failed', () => {
      console.error('Reconnection failed');
      // Implement fallback strategy
      this.handleReconnectionFailed();
    });

    // Handle token expiration during connection
    this.socket.on('connect_error', async (error) => {
      if (error.message === 'TOKEN_EXPIRED') {
        await this.refreshTokenAndReconnect();
      }
    });
  }

  // ══════════════════════════════════════════════════════════════
  // STATE RECOVERY AFTER RECONNECTION
  // ══════════════════════════════════════════════════════════════

  private async onReconnected() {
    if (!this.reconnecting) return;
    this.reconnecting = false;

    console.log('Recovering state...');

    // 1. Rejoin all rooms
    for (const roomId of this.state.rooms) {
      await this.joinRoom(roomId);
    }

    // 2. Fetch missed messages
    await this.syncMissedMessages();

    // 3. Resend pending messages
    await this.resendPendingMessages();

    console.log('State recovered');
  }

  private async syncMissedMessages() {
    if (!this.state.lastEventId) return;

    // Request missed messages from server
    const response = await this.emitWithAck('sync:messages', {
      lastEventId: this.state.lastEventId,
      rooms: this.state.rooms
    });

    if (response.messages) {
      // Process missed messages
      for (const message of response.messages) {
        this.handleMessage(message);
      }
    }
  }

  private async resendPendingMessages() {
    const pending = [...this.state.pendingMessages];
    this.state.pendingMessages = [];

    for (const message of pending) {
      try {
        await this.sendMessage(message.roomId, message.text);
      } catch (error) {
        // Re-queue failed messages
        this.state.pendingMessages.push(message);
      }
    }
  }

  private async refreshTokenAndReconnect() {
    try {
      const newToken = await this.getToken();
      this.socket!.auth = { token: newToken };
      this.socket!.connect();
    } catch (error) {
      console.error('Token refresh failed:', error);
      // Redirect to login
      window.location.href = '/login';
    }
  }

  private handleReconnectionFailed() {
    // Show user notification
    this.showOfflineNotification();
    
    // Try again after longer delay
    setTimeout(() => {
      this.connect();
    }, 60000);
  }

  // ══════════════════════════════════════════════════════════════
  // MESSAGE HANDLING WITH IDEMPOTENCY
  // ══════════════════════════════════════════════════════════════

  async sendMessage(roomId: string, text: string): Promise<boolean> {
    const messageId = this.generateMessageId();
    const message = { id: messageId, roomId, text, timestamp: Date.now() };

    // Add to pending (for retry on disconnect)
    this.state.pendingMessages.push(message);

    try {
      const response = await this.emitWithAck('chat:send', message);
      
      // Remove from pending on success
      this.state.pendingMessages = this.state.pendingMessages.filter(
        m => m.id !== messageId
      );

      return response.success;
    } catch (error) {
      console.error('Send failed, message queued for retry');
      return false;
    }
  }

  private handleMessage(message: any) {
    // Update last event ID for sync
    if (message.eventId) {
      this.state.lastEventId = message.eventId;
    }
    // Emit to UI
    this.emit('message', message);
  }

  // ══════════════════════════════════════════════════════════════
  // ROOM MANAGEMENT WITH STATE TRACKING
  // ══════════════════════════════════════════════════════════════

  async joinRoom(roomId: string): Promise<boolean> {
    const response = await this.emitWithAck('room:join', { roomId });
    
    if (response.success && !this.state.rooms.includes(roomId)) {
      this.state.rooms.push(roomId);
    }

    return response.success;
  }

  leaveRoom(roomId: string) {
    this.socket?.emit('room:leave', { roomId });
    this.state.rooms = this.state.rooms.filter(r => r !== roomId);
  }

  // ══════════════════════════════════════════════════════════════
  // UTILITIES
  // ══════════════════════════════════════════════════════════════

  private emitWithAck(event: string, data: any): Promise<any> {
    return new Promise((resolve, reject) => {
      const timeout = setTimeout(() => {
        reject(new Error('Acknowledgment timeout'));
      }, 10000);

      this.socket?.emit(event, data, (response: any) => {
        clearTimeout(timeout);
        resolve(response);
      });
    });
  }

  private generateMessageId(): string {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }

  private showOfflineNotification() {
    // Implement UI notification
  }

  private emit(event: string, data: any) {
    // Implement event emitter for UI
  }
}
```

---

## 6. Heartbeat & Connection Health

```typescript
// ════════════════════════════════════════════════════════════════
// HEARTBEAT AND CONNECTION HEALTH MONITORING
// ════════════════════════════════════════════════════════════════

// Server-side heartbeat implementation
class HeartbeatManager {
  private sockets: Map<string, {
    lastPing: number;
    lastPong: number;
    latency: number[];
    missedPongs: number;
  }> = new Map();

  private checkInterval: NodeJS.Timeout | null = null;
  
  constructor(
    private io: Server,
    private options = {
      pingInterval: 25000,      // Send ping every 25s
      pongTimeout: 5000,        // Wait 5s for pong
      maxMissedPongs: 3,        // Disconnect after 3 missed
      latencyHistorySize: 10   // Keep last 10 latency measurements
    }
  ) {}

  start() {
    this.io.on('connection', (socket) => {
      this.registerSocket(socket);
    });

    this.checkInterval = setInterval(() => {
      this.pingAll();
    }, this.options.pingInterval);
  }

  private registerSocket(socket: Socket) {
    this.sockets.set(socket.id, {
      lastPing: 0,
      lastPong: 0,
      latency: [],
      missedPongs: 0
    });

    // Handle pong response
    socket.on('pong', (timestamp: number) => {
      this.handlePong(socket.id, timestamp);
    });

    socket.on('disconnect', () => {
      this.sockets.delete(socket.id);
    });
  }

  private pingAll() {
    const now = Date.now();

    this.sockets.forEach((state, socketId) => {
      const socket = this.io.sockets.sockets.get(socketId);
      if (!socket) {
        this.sockets.delete(socketId);
        return;
      }

      // Check for missed pong
      if (state.lastPing > state.lastPong) {
        state.missedPongs++;

        if (state.missedPongs >= this.options.maxMissedPongs) {
          console.log(`Disconnecting ${socketId}: missed ${state.missedPongs} pongs`);
          socket.disconnect(true);
          return;
        }
      }

      // Send ping
      state.lastPing = now;
      socket.emit('ping', now);
    });
  }

  private handlePong(socketId: string, pingTimestamp: number) {
    const state = this.sockets.get(socketId);
    if (!state) return;

    const now = Date.now();
    const latency = now - pingTimestamp;

    state.lastPong = now;
    state.missedPongs = 0;
    
    // Track latency history
    state.latency.push(latency);
    if (state.latency.length > this.options.latencyHistorySize) {
      state.latency.shift();
    }
  }

  getSocketHealth(socketId: string) {
    const state = this.sockets.get(socketId);
    if (!state) return null;

    const avgLatency = state.latency.length > 0
      ? state.latency.reduce((a, b) => a + b, 0) / state.latency.length
      : 0;

    return {
      connected: true,
      lastPing: state.lastPing,
      lastPong: state.lastPong,
      missedPongs: state.missedPongs,
      averageLatency: Math.round(avgLatency),
      latencyHistory: state.latency
    };
  }

  getAllHealth() {
    const stats: any[] = [];
    this.sockets.forEach((state, socketId) => {
      stats.push({
        socketId,
        ...this.getSocketHealth(socketId)
      });
    });
    return stats;
  }

  stop() {
    if (this.checkInterval) {
      clearInterval(this.checkInterval);
    }
  }
}

// ════════════════════════════════════════════════════════════════
// CLIENT-SIDE HEARTBEAT
// ════════════════════════════════════════════════════════════════

class ClientHeartbeat {
  private latencyHistory: number[] = [];
  private connectionQuality: 'good' | 'degraded' | 'poor' = 'good';
  private lastPongTime = Date.now();
  private missedPings = 0;

  constructor(private socket: Socket) {
    this.setupHandlers();
  }

  private setupHandlers() {
    // Respond to server pings
    this.socket.on('ping', (timestamp: number) => {
      this.socket.emit('pong', timestamp);
      
      // Track our own latency
      const latency = Date.now() - timestamp;
      this.updateLatency(latency);
    });

    // Application-level heartbeat (on top of Socket.io's)
    this.socket.on('heartbeat', () => {
      this.lastPongTime = Date.now();
      this.missedPings = 0;
      this.socket.emit('heartbeat:ack');
    });
  }

  private updateLatency(latency: number) {
    this.latencyHistory.push(latency);
    if (this.latencyHistory.length > 10) {
      this.latencyHistory.shift();
    }

    // Update connection quality
    const avgLatency = this.getAverageLatency();
    if (avgLatency < 100) {
      this.connectionQuality = 'good';
    } else if (avgLatency < 300) {
      this.connectionQuality = 'degraded';
    } else {
      this.connectionQuality = 'poor';
    }

    // Emit event for UI
    this.socket.emit('connection:quality', {
      quality: this.connectionQuality,
      latency: avgLatency
    });
  }

  getAverageLatency(): number {
    if (this.latencyHistory.length === 0) return 0;
    return Math.round(
      this.latencyHistory.reduce((a, b) => a + b, 0) / this.latencyHistory.length
    );
  }

  getConnectionQuality() {
    return {
      quality: this.connectionQuality,
      latency: this.getAverageLatency(),
      history: this.latencyHistory
    };
  }
}

// ════════════════════════════════════════════════════════════════
// CONNECTION HEALTH DASHBOARD DATA
// ════════════════════════════════════════════════════════════════

// Health endpoint for monitoring
app.get('/health/websocket', async (req, res) => {
  const sockets = await io.fetchSockets();
  
  const health = {
    status: 'healthy',
    connections: {
      total: sockets.length,
      byServer: await getConnectionsByServer()
    },
    rooms: {
      total: io.sockets.adapter.rooms.size,
      sizes: getRoomSizes()
    },
    latency: {
      average: getAverageLatency(),
      p95: getP95Latency(),
      p99: getP99Latency()
    },
    errors: {
      connectionErrors: getConnectionErrorCount(),
      messageErrors: getMessageErrorCount()
    }
  };

  // Determine overall status
  if (health.latency.p95 > 500 || health.errors.connectionErrors > 10) {
    health.status = 'degraded';
  }
  if (health.latency.p99 > 1000 || health.errors.connectionErrors > 50) {
    health.status = 'unhealthy';
  }

  res.json(health);
});
```

---

## 7. Performance Optimization

```typescript
// ════════════════════════════════════════════════════════════════
// WEBSOCKET PERFORMANCE OPTIMIZATION
// ════════════════════════════════════════════════════════════════

// 1. BINARY DATA WITH MESSAGEPACK
// ════════════════════════════════════════════════════════════════

import { encode, decode } from '@msgpack/msgpack';

// Server-side
io.engine.on('connection', (rawSocket) => {
  // Use binary parser
  rawSocket.binaryType = 'arraybuffer';
});

// Custom parser for binary messages
const parser = {
  encode(packet: any): ArrayBuffer {
    return encode(packet);
  },
  decode(data: ArrayBuffer): any {
    return decode(new Uint8Array(data));
  }
};

// Client configuration
const socket = io(url, {
  parser: {
    encode: (packet) => encode(packet),
    decode: (data) => decode(new Uint8Array(data))
  }
});

// Comparison: JSON vs MessagePack
const data = { 
  type: 'message', 
  payload: { userId: '123', text: 'Hello', timestamp: Date.now() } 
};

console.log('JSON size:', JSON.stringify(data).length);           // ~80 bytes
console.log('MessagePack size:', encode(data).byteLength);        // ~55 bytes
// ~30% smaller!

// 2. MESSAGE BATCHING
// ════════════════════════════════════════════════════════════════

class MessageBatcher {
  private queue: any[] = [];
  private flushTimeout: NodeJS.Timeout | null = null;
  private maxBatchSize = 100;
  private maxWaitMs = 50;

  constructor(
    private socket: Socket,
    private event: string
  ) {}

  add(message: any) {
    this.queue.push(message);

    if (this.queue.length >= this.maxBatchSize) {
      this.flush();
    } else if (!this.flushTimeout) {
      this.flushTimeout = setTimeout(() => this.flush(), this.maxWaitMs);
    }
  }

  private flush() {
    if (this.flushTimeout) {
      clearTimeout(this.flushTimeout);
      this.flushTimeout = null;
    }

    if (this.queue.length === 0) return;

    const batch = this.queue.splice(0, this.maxBatchSize);
    this.socket.emit(this.event, { batch });
  }
}

// Usage
const batcher = new MessageBatcher(socket, 'messages:batch');

// Instead of:
// socket.emit('message', msg1);
// socket.emit('message', msg2);

// Do:
batcher.add(msg1);
batcher.add(msg2);
// Sends as single batch after 50ms or when batch is full

// 3. COMPRESSION
// ════════════════════════════════════════════════════════════════

import { Server } from 'socket.io';

const io = new Server(httpServer, {
  perMessageDeflate: {
    threshold: 1024,  // Only compress messages > 1KB
    zlibDeflateOptions: {
      chunkSize: 16 * 1024  // 16KB chunks
    },
    zlibInflateOptions: {
      chunkSize: 16 * 1024
    },
    clientNoContextTakeover: true,
    serverNoContextTakeover: true,
    serverMaxWindowBits: 10,
    concurrencyLimit: 10,
    threshold: 1024
  }
});

// 4. EFFICIENT ROOM BROADCASTING
// ════════════════════════════════════════════════════════════════

// Bad: Iterating over all sockets
async function broadcastSlow(roomId: string, event: string, data: any) {
  const sockets = await io.in(roomId).fetchSockets();
  for (const socket of sockets) {
    socket.emit(event, data);
  }
}

// Good: Use built-in room broadcast
function broadcastFast(roomId: string, event: string, data: any) {
  io.to(roomId).emit(event, data);  // Single operation
}

// Even better for large rooms: volatile emit (UDP-like)
function broadcastVolatile(roomId: string, event: string, data: any) {
  io.to(roomId).volatile.emit(event, data);
  // Messages may be dropped if client is slow, but won't block
}

// 5. CONNECTION POOLING FOR EXTERNAL SERVICES
// ════════════════════════════════════════════════════════════════

import { createPool, Pool } from 'generic-pool';

// Pool for database connections in WebSocket handlers
const dbPool: Pool<any> = createPool({
  create: async () => {
    return new DatabaseConnection();
  },
  destroy: async (conn) => {
    await conn.close();
  }
}, {
  min: 2,
  max: 10,
  acquireTimeoutMillis: 5000
});

// Use pool in handlers
socket.on('getData', async (params, callback) => {
  const conn = await dbPool.acquire();
  try {
    const data = await conn.query(params);
    callback({ success: true, data });
  } finally {
    dbPool.release(conn);
  }
});

// 6. MEMORY MANAGEMENT
// ════════════════════════════════════════════════════════════════

// Limit message size
io.on('connection', (socket) => {
  socket.use(([event, ...args], next) => {
    const size = JSON.stringify(args).length;
    if (size > 1024 * 100) { // 100KB limit
      return next(new Error('Message too large'));
    }
    next();
  });
});

// Clean up listeners to prevent memory leaks
socket.on('disconnect', () => {
  socket.removeAllListeners();
});

// Monitor memory usage
setInterval(() => {
  const usage = process.memoryUsage();
  if (usage.heapUsed > 500 * 1024 * 1024) { // 500MB
    console.warn('High memory usage:', usage);
    // Trigger garbage collection if exposed
    if (global.gc) global.gc();
  }
}, 60000);

// 7. BACKPRESSURE HANDLING
// ════════════════════════════════════════════════════════════════

class BackpressureHandler {
  private writeBufferSize = 0;
  private maxBufferSize = 1024 * 1024; // 1MB
  private paused = false;

  constructor(private socket: Socket) {
    this.monitorBuffer();
  }

  private monitorBuffer() {
    // Check buffer periodically
    setInterval(() => {
      // Socket.io internal buffer
      const bufferedAmount = (this.socket as any).conn?.transport?.writable?.writableLength || 0;
      
      if (bufferedAmount > this.maxBufferSize && !this.paused) {
        this.pause();
      } else if (bufferedAmount < this.maxBufferSize / 2 && this.paused) {
        this.resume();
      }
    }, 100);
  }

  private pause() {
    this.paused = true;
    this.socket.emit('flow:pause');
    console.log('Backpressure: pausing client');
  }

  private resume() {
    this.paused = false;
    this.socket.emit('flow:resume');
    console.log('Backpressure: resuming client');
  }

  canSend(): boolean {
    return !this.paused;
  }
}
```

---

## 8. Real-World Patterns

### Chat Application

```typescript
// ════════════════════════════════════════════════════════════════
// PRODUCTION CHAT APPLICATION
// ════════════════════════════════════════════════════════════════

interface ChatRoom {
  id: string;
  name: string;
  members: Set<string>;
  typingUsers: Set<string>;
}

class ChatServer {
  private rooms: Map<string, ChatRoom> = new Map();
  private userRooms: Map<string, Set<string>> = new Map(); // userId -> roomIds

  constructor(private io: Server) {
    this.setupHandlers();
  }

  private setupHandlers() {
    this.io.on('connection', (socket) => {
      const { userId } = socket.data;

      // ════════════════════════════════════════════════════════════
      // JOIN ROOM
      // ════════════════════════════════════════════════════════════
      socket.on('room:join', async ({ roomId }, callback) => {
        try {
          // Authorization check
          const canJoin = await this.canUserJoinRoom(userId, roomId);
          if (!canJoin) {
            return callback({ success: false, error: 'Access denied' });
          }

          // Join Socket.io room
          await socket.join(roomId);

          // Track membership
          if (!this.rooms.has(roomId)) {
            this.rooms.set(roomId, {
              id: roomId,
              name: roomId,
              members: new Set(),
              typingUsers: new Set()
            });
          }
          this.rooms.get(roomId)!.members.add(userId);

          if (!this.userRooms.has(userId)) {
            this.userRooms.set(userId, new Set());
          }
          this.userRooms.get(userId)!.add(roomId);

          // Notify others
          socket.to(roomId).emit('user:joined', {
            userId,
            roomId,
            timestamp: new Date()
          });

          // Get recent messages
          const messages = await this.getRecentMessages(roomId, 50);
          const members = Array.from(this.rooms.get(roomId)!.members);

          callback({ success: true, messages, members });
        } catch (error) {
          callback({ success: false, error: 'Failed to join room' });
        }
      });

      // ════════════════════════════════════════════════════════════
      // SEND MESSAGE
      // ════════════════════════════════════════════════════════════
      socket.on('message:send', async ({ roomId, text, replyTo }, callback) => {
        try {
          // Validate
          if (!text?.trim()) {
            return callback({ success: false, error: 'Empty message' });
          }

          // Check room membership
          if (!this.rooms.get(roomId)?.members.has(userId)) {
            return callback({ success: false, error: 'Not in room' });
          }

          // Save message
          const message = await this.saveMessage({
            id: this.generateId(),
            roomId,
            userId,
            text: text.trim(),
            replyTo,
            timestamp: new Date()
          });

          // Broadcast to room
          this.io.to(roomId).emit('message:new', message);

          // Stop typing indicator
          this.handleTypingStop(socket, roomId);

          callback({ success: true, messageId: message.id });
        } catch (error) {
          callback({ success: false, error: 'Failed to send' });
        }
      });

      // ════════════════════════════════════════════════════════════
      // TYPING INDICATORS
      // ════════════════════════════════════════════════════════════
      const typingTimeouts: Map<string, NodeJS.Timeout> = new Map();

      socket.on('typing:start', ({ roomId }) => {
        const room = this.rooms.get(roomId);
        if (!room?.members.has(userId)) return;

        room.typingUsers.add(userId);
        socket.to(roomId).emit('typing:update', {
          roomId,
          users: Array.from(room.typingUsers)
        });

        // Auto-stop after 5 seconds
        const key = `${roomId}:${userId}`;
        if (typingTimeouts.has(key)) {
          clearTimeout(typingTimeouts.get(key)!);
        }
        typingTimeouts.set(key, setTimeout(() => {
          this.handleTypingStop(socket, roomId);
        }, 5000));
      });

      socket.on('typing:stop', ({ roomId }) => {
        this.handleTypingStop(socket, roomId);
      });

      // ════════════════════════════════════════════════════════════
      // READ RECEIPTS
      // ════════════════════════════════════════════════════════════
      socket.on('message:read', async ({ roomId, messageId }) => {
        await this.markMessageRead(roomId, messageId, userId);
        
        socket.to(roomId).emit('message:read', {
          roomId,
          messageId,
          userId,
          timestamp: new Date()
        });
      });

      // ════════════════════════════════════════════════════════════
      // DISCONNECT
      // ════════════════════════════════════════════════════════════
      socket.on('disconnect', () => {
        // Clean up typing indicators
        this.userRooms.get(userId)?.forEach(roomId => {
          this.handleTypingStop(socket, roomId);
          
          const room = this.rooms.get(roomId);
          if (room) {
            room.members.delete(userId);
            socket.to(roomId).emit('user:left', { userId, roomId });
          }
        });

        this.userRooms.delete(userId);
      });
    });
  }

  private handleTypingStop(socket: Socket, roomId: string) {
    const { userId } = socket.data;
    const room = this.rooms.get(roomId);
    if (!room) return;

    room.typingUsers.delete(userId);
    socket.to(roomId).emit('typing:update', {
      roomId,
      users: Array.from(room.typingUsers)
    });
  }

  // Database operations (implement these)
  private async canUserJoinRoom(userId: string, roomId: string): Promise<boolean> {
    // Check permissions
    return true;
  }

  private async getRecentMessages(roomId: string, limit: number): Promise<any[]> {
    // Fetch from database
    return [];
  }

  private async saveMessage(message: any): Promise<any> {
    // Save to database
    return message;
  }

  private async markMessageRead(roomId: string, messageId: string, userId: string): Promise<void> {
    // Update read receipt
  }

  private generateId(): string {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }
}
```

### Live Notifications System

```typescript
// ════════════════════════════════════════════════════════════════
// REAL-TIME NOTIFICATION SYSTEM
// ════════════════════════════════════════════════════════════════

interface Notification {
  id: string;
  userId: string;
  type: string;
  title: string;
  body: string;
  data: any;
  read: boolean;
  createdAt: Date;
}

class NotificationService {
  constructor(
    private io: Server,
    private redis: Redis
  ) {}

  // Send notification to user (from any service)
  async send(userId: string, notification: Omit<Notification, 'id' | 'createdAt' | 'read'>) {
    const fullNotification: Notification = {
      ...notification,
      id: this.generateId(),
      read: false,
      createdAt: new Date()
    };

    // Save to database
    await this.saveNotification(fullNotification);

    // Publish to Redis (picked up by all Socket.io servers)
    await this.redis.publish('notifications', JSON.stringify({
      userId,
      notification: fullNotification
    }));

    return fullNotification;
  }

  // Setup socket handlers
  setupHandlers(socket: Socket) {
    const { userId } = socket.data;

    // User joins their notification room
    socket.join(`notifications:${userId}`);

    // Get unread count
    socket.on('notifications:getUnread', async (callback) => {
      const count = await this.getUnreadCount(userId);
      callback({ count });
    });

    // Mark as read
    socket.on('notifications:markRead', async ({ notificationId }, callback) => {
      await this.markAsRead(userId, notificationId);
      
      // Update unread count for all user's connections
      const count = await this.getUnreadCount(userId);
      this.io.to(`notifications:${userId}`).emit('notifications:countUpdate', { count });
      
      callback({ success: true });
    });

    // Mark all as read
    socket.on('notifications:markAllRead', async (callback) => {
      await this.markAllAsRead(userId);
      this.io.to(`notifications:${userId}`).emit('notifications:countUpdate', { count: 0 });
      callback({ success: true });
    });
  }

  // Subscribe to Redis notifications
  async subscribeToNotifications() {
    const subscriber = this.redis.duplicate();
    await subscriber.subscribe('notifications');

    subscriber.on('message', (channel, message) => {
      if (channel === 'notifications') {
        const { userId, notification } = JSON.parse(message);
        
        // Emit to user's notification room
        this.io.to(`notifications:${userId}`).emit('notification:new', notification);
      }
    });
  }

  private async saveNotification(notification: Notification): Promise<void> {
    // Database save
  }

  private async getUnreadCount(userId: string): Promise<number> {
    // Database query
    return 0;
  }

  private async markAsRead(userId: string, notificationId: string): Promise<void> {
    // Database update
  }

  private async markAllAsRead(userId: string): Promise<void> {
    // Database update
  }

  private generateId(): string {
    return `notif-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }
}

// Usage from external services (API, workers, etc.)
const notificationService = new NotificationService(io, redis);

// When something happens that needs notification
await notificationService.send('user-123', {
  userId: 'user-123',
  type: 'message',
  title: 'New Message',
  body: 'You have a new message from John',
  data: { messageId: 'msg-456', senderId: 'user-789' }
});
```

---

## 9. Common Pitfalls

```typescript
// ════════════════════════════════════════════════════════════════
// COMMON WEBSOCKET PITFALLS AND SOLUTIONS
// ════════════════════════════════════════════════════════════════

// ❌ PITFALL 1: Not handling disconnections properly
// Problem: Assuming socket is always connected

// Bad
socket.emit('message', data);  // May fail silently

// Good
function safeSend(socket: Socket, event: string, data: any): boolean {
  if (socket.connected) {
    socket.emit(event, data);
    return true;
  }
  console.warn('Socket not connected, queuing message');
  queueMessage(event, data);
  return false;
}

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 2: Memory leaks from unremoved listeners
// Problem: Adding listeners without cleanup

// Bad
socket.on('connect', () => {
  socket.on('data', handleData);  // New listener each reconnect!
});

// Good
const handleData = (data: any) => { /* ... */ };
socket.on('data', handleData);

socket.on('disconnect', () => {
  socket.off('data', handleData);
});

// Or use .once() for one-time handlers
socket.once('ready', () => { /* ... */ });

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 3: Not implementing reconnection properly
// Problem: Users stuck in disconnected state

// Bad: Relying only on Socket.io's auto-reconnect
const socket = io(url);  // May give up after attempts

// Good: Custom reconnection logic
const socket = io(url, {
  reconnection: true,
  reconnectionAttempts: Infinity,
  reconnectionDelay: 1000,
  reconnectionDelayMax: 30000
});

socket.on('reconnect_failed', () => {
  // Manual retry with longer delay
  setTimeout(() => socket.connect(), 60000);
  showOfflineUI();
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 4: Not authenticating WebSocket connections
// Problem: Anyone can connect

// Bad
io.on('connection', (socket) => {
  // No auth check!
  handleSocket(socket);
});

// Good
io.use((socket, next) => {
  const token = socket.handshake.auth.token;
  try {
    const user = verifyToken(token);
    socket.data.user = user;
    next();
  } catch {
    next(new Error('Authentication failed'));
  }
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 5: Blocking the event loop
// Problem: Long operations block all sockets

// Bad
socket.on('process', (data) => {
  const result = heavyComputation(data);  // Blocks!
  socket.emit('result', result);
});

// Good: Use workers or async
socket.on('process', async (data) => {
  const result = await workerPool.exec('heavyComputation', [data]);
  socket.emit('result', result);
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 6: Not implementing rate limiting
// Problem: Clients can flood the server

// Bad
socket.on('message', (data) => {
  broadcast(data);  // No limit!
});

// Good
const limiter = new RateLimiterMemory({ points: 10, duration: 1 });

socket.on('message', async (data) => {
  try {
    await limiter.consume(socket.data.userId);
    broadcast(data);
  } catch {
    socket.emit('error', { code: 'RATE_LIMIT' });
  }
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 7: Not handling large messages
// Problem: Large messages crash server

// Bad
socket.on('upload', (data) => {
  saveToDatabase(data);  // Could be gigabytes!
});

// Good
const MAX_MESSAGE_SIZE = 1024 * 100;  // 100KB

socket.use(([event, data], next) => {
  const size = JSON.stringify(data).length;
  if (size > MAX_MESSAGE_SIZE) {
    return next(new Error('Message too large'));
  }
  next();
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 8: Not using rooms efficiently
// Problem: Broadcasting to all sockets

// Bad: Iterating all sockets
io.sockets.sockets.forEach(socket => {
  if (socket.data.roomId === roomId) {
    socket.emit('message', data);
  }
});

// Good: Use rooms
io.to(roomId).emit('message', data);  // Single operation

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 9: Not handling scaling properly
// Problem: Messages don't reach users on other servers

// Bad: In-memory state only
const users = new Map();  // Lost when server restarts

// Good: Use Redis adapter + shared state
io.adapter(createAdapter(pubClient, subClient));

// Store user sessions in Redis
await redis.hset(`user:${userId}:sessions`, socketId, serverId);

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 10: Ignoring error handling
// Problem: Errors crash the server

// Bad
socket.on('action', async (data) => {
  const result = await riskyOperation(data);  // May throw!
  socket.emit('result', result);
});

// Good
socket.on('action', async (data, callback) => {
  try {
    const result = await riskyOperation(data);
    callback({ success: true, result });
  } catch (error) {
    console.error('Action failed:', error);
    callback({ success: false, error: 'Operation failed' });
  }
});
```

---

## 10. Interview Questions

### Basic Questions

**Q: "What is the WebSocket protocol?"**
> "WebSocket is a protocol that provides full-duplex, bidirectional communication over a single TCP connection. It starts with an HTTP upgrade handshake, then both client and server can send messages at any time. Unlike HTTP's request-response model, either side can initiate communication. It's designed for real-time applications like chat, gaming, and live updates."

**Q: "What's the difference between WebSocket and HTTP?"**
> "HTTP is request-response: client asks, server answers, connection may close. WebSocket is persistent and bidirectional: once established, both sides can send messages anytime. HTTP has high overhead (headers every request), WebSocket has minimal overhead (2-14 byte frame headers). HTTP is stateless, WebSocket connections are stateful."

**Q: "What is the WebSocket handshake?"**
> "The handshake is how a WebSocket connection is established. Client sends an HTTP GET with `Upgrade: websocket` and `Connection: Upgrade` headers, plus a random `Sec-WebSocket-Key`. Server responds with 101 Switching Protocols and a `Sec-WebSocket-Accept` header (derived from the client's key). After this, both sides communicate using WebSocket frames."

**Q: "What are WebSocket frames?"**
> "Frames are the data units in WebSocket protocol. Each frame has a small header (2-14 bytes) and payload. Frame types include text (opcode 0x1), binary (0x2), close (0x8), ping (0x9), and pong (0xA). The header contains flags like FIN (final fragment), opcode, mask bit, and payload length."

### Intermediate Questions

**Q: "How does Socket.io differ from native WebSocket?"**
> "Socket.io is a library built on top of WebSocket that adds: automatic reconnection with exponential backoff, room-based broadcasting, event acknowledgments, binary support, fallback to long-polling if WebSocket fails, and multiplexing. It has slightly more overhead but significantly more features for production use."

**Q: "How do you handle WebSocket authentication?"**
> "Authenticate during the handshake, not after connection. Pass JWT token in handshake auth object or query string. Verify in middleware before connection completes. Reject invalid tokens immediately with close code. For token refresh, emit a special event to update credentials without reconnecting. Never trust user data without verification."

**Q: "What is a WebSocket ping/pong?"**
> "Ping/pong is the heartbeat mechanism. Server sends ping frame, client responds with pong. This detects dead connections (zombie connections where TCP thinks it's alive but client is gone). Typically done every 25-30 seconds to stay under proxy timeouts. If pong not received within timeout, connection is considered dead and closed."

**Q: "How do you handle reconnection properly?"**
> "Implement exponential backoff with jitter: start at 1 second, double each attempt, cap at 30 seconds, add random jitter to prevent thundering herd. Track state before disconnect (rooms, pending messages). On reconnect, rejoin rooms, sync missed messages using last event ID, and resend queued messages. Handle auth token refresh if expired."

### Advanced Questions

**Q: "How do you scale WebSocket servers horizontally?"**
> "Two main challenges: connection affinity and cross-server communication.

For affinity, use sticky sessions via IP hash or cookie at load balancer level. Alternatively, use Layer 4 load balancing.

For cross-server communication, use a pub/sub backbone like Redis. Socket.io's Redis adapter handles this: when you broadcast to a room, it publishes to Redis, all servers receive and emit to their local clients.

Also consider: connection limits per server, graceful draining for deploys, and health checks that account for WebSocket state."

**Q: "How would you implement a real-time collaborative editing feature?"**
> "Use Operational Transformation (OT) or CRDTs for conflict resolution. Each client has local state and sends operations to server. Server transforms concurrent operations to maintain consistency. Broadcast transformed operations to all clients.

WebSocket provides the transport: low latency for character-by-character sync. Implement cursor position sharing for awareness. Handle offline with operation queuing and reconciliation on reconnect. Consider libraries like Yjs or ShareDB that handle the complex conflict resolution."

**Q: "How do you handle backpressure in WebSocket?"**
> "Backpressure occurs when server sends faster than client can receive. Monitor send buffer size. When buffer exceeds threshold, pause sending and notify client to slow down. Implement flow control: server emits 'pause' event, client stops sending, server emits 'resume' when caught up. For volatile data (like live prices), use volatile emit to drop messages rather than queue."

**Q: "What are the security considerations for WebSocket?"**
> "1. Always use WSS (WebSocket Secure) - encrypted connection.
2. Authenticate during handshake, not after.
3. Validate and sanitize all incoming messages.
4. Implement rate limiting per connection.
5. Set maximum message size limits.
6. Use CORS to restrict origins.
7. Implement authorization for room joins.
8. Don't expose sensitive data in connection URLs.
9. Handle token refresh securely.
10. Log and monitor for anomalies."

**Q: "How do you debug WebSocket issues in production?"**
> "1. Chrome DevTools Network tab shows WebSocket frames.
2. Implement structured logging: connection, disconnection, message events with correlation IDs.
3. Add client-side logging with reconnection attempts and latency.
4. Monitor metrics: connection count, message rate, error rate, latency percentiles.
5. Use distributed tracing to follow messages across services.
6. Implement health endpoints that show connection stats.
7. For intermittent issues, capture and replay traffic using tools like Charles Proxy."

**Q: "Explain the difference between ws, Socket.io, and uWebSockets."**
> "**ws**: Native WebSocket implementation for Node.js. Minimal, fast, no extra features. Good for simple use cases or as base for custom implementation.

**Socket.io**: Full-featured library with reconnection, rooms, acknowledgments, fallback transports. More overhead but production-ready features. Has both client and server libraries.

**uWebSockets**: Extremely high-performance WebSocket implementation in C++ with Node.js bindings. Handles millions of connections. Used when performance is critical, but less feature-rich.

Choose based on needs: ws for simple cases, Socket.io for typical apps, uWebSockets for extreme scale."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│              WEBSOCKET IMPLEMENTATION CHECKLIST                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SETUP:                                                         │
│  □ Choose library (native ws, Socket.io, uWebSockets)          │
│  □ Configure CORS properly                                      │
│  □ Set up Redis adapter for scaling                            │
│  □ Configure load balancer (sticky sessions)                    │
│                                                                 │
│  AUTHENTICATION:                                                │
│  □ Verify JWT in handshake middleware                          │
│  □ Attach user data to socket                                  │
│  □ Handle token expiration/refresh                             │
│  □ Implement authorization for rooms/events                    │
│                                                                 │
│  CONNECTION MANAGEMENT:                                         │
│  □ Implement heartbeat (25-30s interval)                       │
│  □ Configure reconnection with exponential backoff             │
│  □ Handle graceful disconnection                               │
│  □ Track connection state                                      │
│                                                                 │
│  SCALING:                                                       │
│  □ Use Redis pub/sub adapter                                   │
│  □ Configure sticky sessions                                   │
│  □ Implement graceful draining for deploys                     │
│  □ Monitor connection distribution                             │
│                                                                 │
│  SECURITY:                                                      │
│  □ Use WSS (TLS)                                               │
│  □ Rate limit connections and messages                         │
│  □ Validate all incoming data                                  │
│  □ Set maximum message size                                    │
│                                                                 │
│  PERFORMANCE:                                                   │
│  □ Use binary format (MessagePack) for large payloads          │
│  □ Implement message batching                                  │
│  □ Enable compression for large messages                       │
│  □ Use volatile emit for non-critical data                     │
│                                                                 │
│  MONITORING:                                                    │
│  □ Log connections/disconnections                              │
│  □ Track latency metrics                                       │
│  □ Monitor error rates                                         │
│  □ Set up alerts for connection spikes                         │
│                                                                 │
│  RECOVERY:                                                      │
│  □ Implement state recovery on reconnect                       │
│  □ Queue messages during disconnect                            │
│  □ Sync missed events                                          │
│  □ Handle duplicate message detection                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

FRAME OVERHEAD COMPARISON:
┌──────────────────────────────────────────┐
│ HTTP:      ~800 bytes headers per request │
│ WebSocket: 2-14 bytes per frame           │
│ Savings:   98%+ for small messages        │
└──────────────────────────────────────────┘

WHEN TO USE WEBSOCKETS:
✅ Real-time chat/messaging
✅ Live notifications
✅ Collaborative editing
✅ Gaming
✅ Live dashboards/feeds
✅ IoT device communication

WHEN NOT TO USE:
❌ Simple CRUD operations
❌ Infrequent updates (use polling)
❌ One-way server→client only (use SSE)
❌ When connection persistence isn't needed
```

---

*Last updated: February 2026*

