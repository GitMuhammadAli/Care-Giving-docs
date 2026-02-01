# ⏳ Long Polling - Complete Guide

> A comprehensive guide to long polling - fallback strategies, timeout handling, implementation patterns, and when to use it.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Long polling is a technique where the client makes an HTTP request and the server holds it open until new data is available, then responds and the client immediately makes a new request - simulating server push using standard HTTP request/response."

### The 5 Key Concepts (Remember These!)
```
1. HELD REQUEST      → Server doesn't respond immediately, waits for data
2. IMMEDIATE RECONNECT → Client makes new request right after receiving response
3. TIMEOUT HANDLING  → Both client and server must handle request timeouts
4. FALLBACK          → Works everywhere HTTP works (corporate firewalls, old browsers)
5. HIGHER OVERHEAD   → More HTTP headers than WebSocket/SSE, but simpler
```

### Long Polling Flow Diagram
```
┌─────────────────────────────────────────────────────────────────┐
│                    LONG POLLING FLOW                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CLIENT                              SERVER                     │
│  ──────                              ──────                     │
│                                                                 │
│  1. ────── GET /poll ─────────────▶  [HOLD REQUEST]            │
│            (request held...)         │                         │
│                                      │ (waiting for data...)   │
│                                      │                         │
│                                      │ [EVENT OCCURS]          │
│  2. ◀──── Response with data ────── │                         │
│                                                                 │
│  3. ────── GET /poll ─────────────▶  [HOLD REQUEST]            │
│            (immediately reconnect)   │                         │
│                                      │ (waiting...)            │
│                                      │                         │
│            [30s TIMEOUT]             │                         │
│  4. ◀──── 204 No Content ────────── │ (no data, timeout)      │
│                                                                 │
│  5. ────── GET /poll ─────────────▶  [HOLD REQUEST]            │
│            (immediately reconnect)   │                         │
│                                      ...                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Long Polling vs Other Real-Time Methods
```
┌─────────────────────────────────────────────────────────────────┐
│           COMPARISON: POLLING vs LONG POLLING vs SSE            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  TRADITIONAL POLLING:                                           │
│  ───────────────────                                            │
│  Client: GET? GET? GET? GET? GET? GET? GET?                    │
│  Server: no   no   no   DATA no   no   DATA                    │
│  • Wasteful - many empty requests                              │
│  • Delay between data availability and delivery                │
│  • Configurable interval (e.g., every 5 seconds)               │
│                                                                 │
│  LONG POLLING:                                                  │
│  ────────────                                                   │
│  Client: GET ──────────────────────▶ DATA                      │
│  Client: GET ─────────▶ DATA                                   │
│  Client: GET ──────────────────────────────▶ TIMEOUT           │
│  Client: GET ───▶ DATA                                         │
│  • Efficient - only responds when data available               │
│  • Near real-time delivery                                     │
│  • Server holds connection                                     │
│                                                                 │
│  SERVER-SENT EVENTS:                                            │
│  ───────────────────                                            │
│  Client: GET ═══════════════════════════════▶                  │
│  Server:     DATA  DATA    DATA DATA    DATA                   │
│  • Single persistent connection                                │
│  • Most efficient for server-to-client                         │
│  • Native browser support                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Comet"** | "Long polling is part of the Comet family of techniques" |
| **"Hanging GET"** | "We use a hanging GET pattern for real-time updates" |
| **"Request timeout"** | "We set request timeout to 30 seconds to prevent indefinite hangs" |
| **"Immediate reconnect"** | "Client immediately reconnects after each response" |
| **"Graceful degradation"** | "Long polling provides graceful degradation when WebSocket fails" |
| **"Connection throttling"** | "We implement connection throttling to prevent overwhelming the server" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Server timeout | **30-60 seconds** | Below most proxy timeouts |
| Client timeout | **35-65 seconds** | Slightly longer than server |
| Reconnect delay | **0-100ms** | Immediate, with small jitter |
| Error backoff start | **1 second** | Initial retry delay |
| Max backoff | **30 seconds** | Cap for error retries |
| HTTP header overhead | **~500-800 bytes** | Per request/response |

### The "Wow" Statement (Memorize This!)
> "We use long polling as a fallback transport when WebSockets are unavailable - behind corporate firewalls or older infrastructure that strips upgrade headers. The client sends a request, the server holds it up to 30 seconds waiting for data. If data arrives, respond immediately and client reconnects. If timeout, respond with empty body and client reconnects. We track 'last event ID' on client to handle missed messages during reconnection, and implement exponential backoff for error scenarios. Socket.io uses this exact pattern as its fallback transport."

### Quick Architecture Drawing (Draw This!)
```
CLIENT                                SERVER
┌──────────────────┐                 ┌──────────────────┐
│                  │                 │                  │
│  Poll Loop       │    Request      │  Request Queue   │
│  ┌────────────┐  │ ───────────────▶│  ┌────────────┐  │
│  │ Make       │  │                 │  │ Hold       │  │
│  │ Request    │──┼───┐             │  │ Request    │  │
│  └────────────┘  │   │             │  └─────┬──────┘  │
│        ▲         │   │             │        │         │
│        │         │   │             │        ▼         │
│  ┌─────┴──────┐  │   │             │  ┌────────────┐  │
│  │ Handle     │  │   │   Response  │  │ Event      │  │
│  │ Response   │◀─┼───┼─────────────│◀─│ Emitter    │  │
│  └────────────┘  │   │             │  └────────────┘  │
│                  │   │             │                  │
└──────────────────┘   │             └──────────────────┘
                       │
                    [Timeout or Data]
                       │
                    Reconnect immediately
```

### Interview Rapid Fire (Practice These!)

**Q: "What is long polling?"**
> "Server holds HTTP request until data is available or timeout. Client immediately reconnects after each response, simulating real-time push over HTTP."

**Q: "Why use long polling over WebSockets?"**
> "Compatibility. Works through all proxies, firewalls, and older browsers. WebSocket may be blocked or stripped. Long polling is the reliable fallback."

**Q: "What's the downside of long polling?"**
> "Higher overhead - HTTP headers on every reconnection. More server resources holding connections. Slight latency on reconnection. But it works everywhere."

**Q: "How do you handle timeouts?"**
> "Server times out after 30 seconds with empty response. Client has slightly longer timeout. Client immediately reconnects after any response. Exponential backoff on errors."

**Q: "How do you prevent missed messages?"**
> "Track last event ID on client. Send it with each poll request. Server sends events since that ID. Same pattern as SSE's Last-Event-ID."

**Q: "How does Socket.io use long polling?"**
> "Socket.io starts with long polling for fast initial connection, then upgrades to WebSocket. If upgrade fails, continues with long polling. Best of both worlds."

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "What is long polling?"

**Junior Answer:**
> "It's when the server doesn't respond right away and waits until there's new data."

**Senior Answer:**
> "Long polling is a technique that simulates server push using standard HTTP. The client sends a request, and instead of responding immediately, the server holds the connection open until either:
1. New data is available to send
2. A timeout period expires (typically 30-60 seconds)

After receiving any response, the client immediately opens a new request. This creates a near-real-time stream while using only standard HTTP, which works through all proxies and firewalls.

The key difference from regular polling is efficiency. Regular polling might send a request every 5 seconds, most getting empty responses. Long polling only completes when there's actual data, reducing unnecessary traffic while delivering updates faster."

### When Asked: "When would you use long polling over WebSocket?"

**Junior Answer:**
> "When WebSocket doesn't work."

**Senior Answer:**
> "Long polling is the universal fallback. Use it when:

1. **Corporate environments** - Many corporate proxies strip WebSocket upgrade headers or block non-HTTP ports
2. **Load balancer limitations** - Some LBs don't support WebSocket or have poor support
3. **Initial connection** - Socket.io uses long polling first for faster initial connection, then upgrades
4. **Browser compatibility** - Though rare now, some older browsers don't support WebSocket
5. **Debugging** - Easier to debug with standard HTTP tools

The tradeoff is higher overhead (HTTP headers per message) and slightly higher latency on reconnection. But reliability often trumps efficiency - a working connection beats a fast connection that doesn't work."

### Follow-up Questions to Expect:

| They Ask | You Answer |
|----------|------------|
| "How much overhead does long polling add?" | "500-800 bytes per request in HTTP headers. For low-frequency updates (few per minute), negligible. For high-frequency, WebSocket/SSE is 10-100x more efficient." |
| "What about mobile connections?" | "Long polling works but more battery-intensive due to radio wake-ups. Consider batching responses and longer poll intervals on mobile." |
| "How do you scale long polling?" | "Each held request consumes a connection. Use async/non-blocking server (Node.js, Go, async Python). Redis pub/sub for multi-server. Monitor connection counts." |
| "What's Comet?" | "Old umbrella term for push techniques including long polling, streaming, and hidden iframe. Long polling is the most common Comet implementation." |

---

## 📚 Table of Contents

1. [Basic Implementation](#1-basic-implementation)
2. [Server Implementation](#2-server-implementation)
3. [Client Implementation](#3-client-implementation)
4. [Handling Timeouts](#4-handling-timeouts)
5. [Message Ordering & Recovery](#5-message-ordering--recovery)
6. [Scaling Long Polling](#6-scaling-long-polling)
7. [Transport Upgrade Pattern](#7-transport-upgrade-pattern)
8. [Real-World Patterns](#8-real-world-patterns)
9. [Common Pitfalls](#9-common-pitfalls)
10. [Interview Questions](#10-interview-questions)

---

## 1. Basic Implementation

### Simple Long Polling Example

```typescript
// ════════════════════════════════════════════════════════════════
// BASIC LONG POLLING - SERVER (Express)
// ════════════════════════════════════════════════════════════════

import express from 'express';

const app = express();
const POLL_TIMEOUT = 30000; // 30 seconds

// Pending requests waiting for data
const pendingRequests: Map<string, express.Response> = new Map();

// Message queue (in production, use Redis)
const messageQueue: Array<{ id: number; data: any; timestamp: number }> = [];
let messageIdCounter = 0;

// Long polling endpoint
app.get('/poll', (req, res) => {
  const clientId = req.query.clientId as string || generateClientId();
  const lastEventId = parseInt(req.query.lastEventId as string) || 0;

  // Check for pending messages first
  const pendingMessages = messageQueue.filter(m => m.id > lastEventId);
  
  if (pendingMessages.length > 0) {
    // Immediate response with pending messages
    return res.json({
      messages: pendingMessages,
      lastEventId: pendingMessages[pendingMessages.length - 1].id
    });
  }

  // No pending messages - hold the request
  pendingRequests.set(clientId, res);

  // Set timeout to prevent indefinite hanging
  const timeout = setTimeout(() => {
    if (pendingRequests.has(clientId)) {
      pendingRequests.delete(clientId);
      // Empty response on timeout
      res.json({ messages: [], lastEventId });
    }
  }, POLL_TIMEOUT);

  // Clean up on client disconnect
  req.on('close', () => {
    clearTimeout(timeout);
    pendingRequests.delete(clientId);
  });
});

// Endpoint to push new messages
app.post('/message', express.json(), (req, res) => {
  const message = {
    id: ++messageIdCounter,
    data: req.body,
    timestamp: Date.now()
  };

  // Add to queue
  messageQueue.push(message);

  // Clean old messages (keep last 1000)
  while (messageQueue.length > 1000) {
    messageQueue.shift();
  }

  // Notify all waiting clients
  pendingRequests.forEach((response, clientId) => {
    response.json({
      messages: [message],
      lastEventId: message.id
    });
    pendingRequests.delete(clientId);
  });

  res.json({ success: true, messageId: message.id });
});

function generateClientId(): string {
  return `client-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
}

app.listen(3000, () => console.log('Long polling server on port 3000'));

// ════════════════════════════════════════════════════════════════
// BASIC LONG POLLING - CLIENT (Browser)
// ════════════════════════════════════════════════════════════════

class LongPollingClient {
  private polling = false;
  private lastEventId = 0;
  private clientId: string;
  private onMessage: (messages: any[]) => void;

  constructor(
    private baseUrl: string,
    onMessage: (messages: any[]) => void
  ) {
    this.clientId = `client-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
    this.onMessage = onMessage;
  }

  start() {
    this.polling = true;
    this.poll();
  }

  stop() {
    this.polling = false;
  }

  private async poll() {
    while (this.polling) {
      try {
        const url = `${this.baseUrl}/poll?clientId=${this.clientId}&lastEventId=${this.lastEventId}`;
        
        const response = await fetch(url, {
          method: 'GET',
          headers: { 'Content-Type': 'application/json' }
        });

        if (!response.ok) {
          throw new Error(`HTTP ${response.status}`);
        }

        const data = await response.json();

        if (data.messages && data.messages.length > 0) {
          this.onMessage(data.messages);
          this.lastEventId = data.lastEventId;
        }

        // Immediately poll again
        // (small delay to prevent tight loop on errors)
        await this.delay(10);

      } catch (error) {
        console.error('Poll error:', error);
        // Wait before retrying on error
        await this.delay(1000);
      }
    }
  }

  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// Usage
const client = new LongPollingClient('http://localhost:3000', (messages) => {
  messages.forEach(msg => {
    console.log('Received:', msg);
  });
});

client.start();

// Later: client.stop();
```

---

## 2. Server Implementation

### Production-Ready Server

```typescript
// ════════════════════════════════════════════════════════════════
// PRODUCTION LONG POLLING SERVER
// ════════════════════════════════════════════════════════════════

import express from 'express';
import { EventEmitter } from 'events';
import { createClient, RedisClientType } from 'redis';

interface PendingRequest {
  res: express.Response;
  userId: string;
  channels: string[];
  lastEventId: number;
  timeout: NodeJS.Timeout;
  connectedAt: number;
}

interface Message {
  id: number;
  channel: string;
  data: any;
  timestamp: number;
}

class LongPollingServer {
  private pendingRequests: Map<string, PendingRequest> = new Map();
  private messageEmitter = new EventEmitter();
  private redis: RedisClientType;
  private subscriber: RedisClientType;
  private messageIdCounter = 0;

  private readonly POLL_TIMEOUT = 30000;
  private readonly MESSAGE_RETENTION = 5 * 60 * 1000; // 5 minutes
  private readonly MAX_MESSAGES = 1000;

  constructor(private app: express.Application) {
    this.setupRoutes();
    this.setupRedis();
    this.messageEmitter.setMaxListeners(10000); // Handle many waiting requests
  }

  private async setupRedis() {
    this.redis = createClient({ url: process.env.REDIS_URL });
    this.subscriber = this.redis.duplicate();

    await Promise.all([
      this.redis.connect(),
      this.subscriber.connect()
    ]);

    // Subscribe to messages channel
    await this.subscriber.subscribe('longpoll:messages', (message) => {
      const parsed = JSON.parse(message);
      this.handleNewMessage(parsed);
    });
  }

  private setupRoutes() {
    // ══════════════════════════════════════════════════════════════
    // POLL ENDPOINT
    // ══════════════════════════════════════════════════════════════
    this.app.get('/api/poll', this.authenticate, async (req, res) => {
      const requestId = this.generateId();
      const userId = (req as any).user?.id || 'anonymous';
      const channels = this.parseChannels(req.query.channels as string);
      const lastEventId = parseInt(req.query.lastEventId as string) || 0;

      // Check for pending messages first
      const pendingMessages = await this.getPendingMessages(channels, lastEventId);
      
      if (pendingMessages.length > 0) {
        return res.json({
          success: true,
          messages: pendingMessages,
          lastEventId: pendingMessages[pendingMessages.length - 1].id
        });
      }

      // Hold the request
      const timeout = setTimeout(() => {
        this.completeRequest(requestId, []);
      }, this.POLL_TIMEOUT);

      const pending: PendingRequest = {
        res,
        userId,
        channels,
        lastEventId,
        timeout,
        connectedAt: Date.now()
      };

      this.pendingRequests.set(requestId, pending);

      // Listen for new messages
      const handler = (message: Message) => {
        if (this.shouldReceive(pending, message)) {
          this.completeRequest(requestId, [message]);
        }
      };

      this.messageEmitter.on('message', handler);

      // Cleanup on disconnect
      req.on('close', () => {
        this.messageEmitter.off('message', handler);
        clearTimeout(timeout);
        this.pendingRequests.delete(requestId);
      });
    });

    // ══════════════════════════════════════════════════════════════
    // SEND MESSAGE ENDPOINT
    // ══════════════════════════════════════════════════════════════
    this.app.post('/api/messages', this.authenticate, express.json(), async (req, res) => {
      const { channel, data } = req.body;

      if (!channel || !data) {
        return res.status(400).json({ error: 'channel and data required' });
      }

      const message = await this.publishMessage(channel, data);
      res.json({ success: true, messageId: message.id });
    });

    // ══════════════════════════════════════════════════════════════
    // HEALTH CHECK
    // ══════════════════════════════════════════════════════════════
    this.app.get('/api/poll/health', (req, res) => {
      res.json({
        pendingConnections: this.pendingRequests.size,
        status: 'healthy'
      });
    });
  }

  private completeRequest(requestId: string, messages: Message[]) {
    const pending = this.pendingRequests.get(requestId);
    if (!pending) return;

    clearTimeout(pending.timeout);
    this.pendingRequests.delete(requestId);

    const lastEventId = messages.length > 0 
      ? messages[messages.length - 1].id 
      : pending.lastEventId;

    try {
      pending.res.json({
        success: true,
        messages,
        lastEventId
      });
    } catch (error) {
      // Response already sent or connection closed
    }
  }

  private shouldReceive(pending: PendingRequest, message: Message): boolean {
    // Check channel subscription
    if (!pending.channels.includes('*') && !pending.channels.includes(message.channel)) {
      return false;
    }

    // Check event ID
    if (message.id <= pending.lastEventId) {
      return false;
    }

    return true;
  }

  private async publishMessage(channel: string, data: any): Promise<Message> {
    const id = ++this.messageIdCounter;
    
    const message: Message = {
      id,
      channel,
      data,
      timestamp: Date.now()
    };

    // Store in Redis for recovery
    await this.redis.zAdd('longpoll:messages', {
      score: id,
      value: JSON.stringify(message)
    });

    // Cleanup old messages
    const cutoffId = id - this.MAX_MESSAGES;
    await this.redis.zRemRangeByScore('longpoll:messages', '-inf', cutoffId);

    // Publish to all servers
    await this.redis.publish('longpoll:messages', JSON.stringify(message));

    return message;
  }

  private handleNewMessage(message: Message) {
    // Emit to all local pending requests
    this.messageEmitter.emit('message', message);
  }

  private async getPendingMessages(channels: string[], lastEventId: number): Promise<Message[]> {
    const messages = await this.redis.zRangeByScore(
      'longpoll:messages',
      lastEventId + 1,
      '+inf'
    );

    return messages
      .map(m => JSON.parse(m) as Message)
      .filter(m => channels.includes('*') || channels.includes(m.channel));
  }

  private parseChannels(channels: string | undefined): string[] {
    if (!channels) return ['*']; // Subscribe to all
    return channels.split(',').map(c => c.trim());
  }

  private authenticate = (req: express.Request, res: express.Response, next: express.NextFunction) => {
    // Implement your authentication logic
    // For example, verify JWT from query string or cookie
    const token = req.query.token as string || req.cookies?.token;
    
    if (!token) {
      return res.status(401).json({ error: 'Authentication required' });
    }

    try {
      (req as any).user = this.verifyToken(token);
      next();
    } catch {
      res.status(401).json({ error: 'Invalid token' });
    }
  };

  private verifyToken(token: string): { id: string } {
    // Implement JWT verification
    return { id: 'user-123' };
  }

  private generateId(): string {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }

  // ══════════════════════════════════════════════════════════════
  // PUBLIC API
  // ══════════════════════════════════════════════════════════════

  async broadcast(channel: string, data: any): Promise<Message> {
    return this.publishMessage(channel, data);
  }

  async sendToUser(userId: string, data: any): Promise<Message> {
    return this.publishMessage(`user:${userId}`, data);
  }

  getStats() {
    return {
      pendingConnections: this.pendingRequests.size,
      connectionsByUser: this.getConnectionsByUser()
    };
  }

  private getConnectionsByUser(): Map<string, number> {
    const counts = new Map<string, number>();
    this.pendingRequests.forEach(p => {
      counts.set(p.userId, (counts.get(p.userId) || 0) + 1);
    });
    return counts;
  }
}

// Usage
const app = express();
const longPollServer = new LongPollingServer(app);

app.listen(3000, () => {
  console.log('Long polling server running on port 3000');
});

// Broadcast from anywhere
longPollServer.broadcast('notifications', { message: 'Hello everyone!' });
longPollServer.sendToUser('user-123', { message: 'Hello user!' });
```

---

## 3. Client Implementation

### Robust Client with Retry Logic

```typescript
// ════════════════════════════════════════════════════════════════
// PRODUCTION LONG POLLING CLIENT
// ════════════════════════════════════════════════════════════════

interface LongPollConfig {
  baseUrl: string;
  channels?: string[];
  token?: string;
  pollTimeout?: number;
  errorRetryDelay?: number;
  maxErrorRetryDelay?: number;
  onMessage: (message: any) => void;
  onConnect?: () => void;
  onDisconnect?: () => void;
  onError?: (error: Error) => void;
}

class RobustLongPollingClient {
  private config: Required<LongPollConfig>;
  private polling = false;
  private lastEventId = 0;
  private errorRetryCount = 0;
  private abortController: AbortController | null = null;
  private connected = false;

  constructor(config: LongPollConfig) {
    this.config = {
      channels: ['*'],
      token: '',
      pollTimeout: 35000, // Slightly longer than server timeout
      errorRetryDelay: 1000,
      maxErrorRetryDelay: 30000,
      onConnect: () => {},
      onDisconnect: () => {},
      onError: () => {},
      ...config
    };
  }

  start() {
    if (this.polling) return;
    this.polling = true;
    this.poll();
  }

  stop() {
    this.polling = false;
    this.abortController?.abort();
    
    if (this.connected) {
      this.connected = false;
      this.config.onDisconnect();
    }
  }

  private async poll() {
    while (this.polling) {
      try {
        const messages = await this.doRequest();
        
        // Successful request - reset error counter
        this.errorRetryCount = 0;

        if (!this.connected) {
          this.connected = true;
          this.config.onConnect();
        }

        // Process messages
        if (messages && messages.length > 0) {
          messages.forEach((msg: any) => {
            try {
              this.config.onMessage(msg);
            } catch (error) {
              console.error('Error in message handler:', error);
            }
          });
        }

        // Small delay to prevent tight loop
        await this.delay(10);

      } catch (error) {
        if (!this.polling) break; // Stopped intentionally

        this.handleError(error as Error);
        
        // Exponential backoff
        const delay = Math.min(
          this.config.errorRetryDelay * Math.pow(2, this.errorRetryCount),
          this.config.maxErrorRetryDelay
        );
        this.errorRetryCount++;

        console.log(`Poll error, retrying in ${delay}ms...`);
        await this.delay(delay);
      }
    }
  }

  private async doRequest(): Promise<any[]> {
    this.abortController = new AbortController();

    const params = new URLSearchParams({
      lastEventId: this.lastEventId.toString(),
      channels: this.config.channels.join(',')
    });

    if (this.config.token) {
      params.set('token', this.config.token);
    }

    const url = `${this.config.baseUrl}/api/poll?${params}`;

    const response = await fetch(url, {
      method: 'GET',
      signal: this.abortController.signal,
      credentials: 'include',
      headers: {
        'Accept': 'application/json'
      }
    });

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }

    const data = await response.json();

    if (data.lastEventId) {
      this.lastEventId = data.lastEventId;
    }

    return data.messages || [];
  }

  private handleError(error: Error) {
    if (this.connected) {
      this.connected = false;
      this.config.onDisconnect();
    }

    this.config.onError(error);
  }

  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }

  // ══════════════════════════════════════════════════════════════
  // PUBLIC API
  // ══════════════════════════════════════════════════════════════

  get isConnected(): boolean {
    return this.connected;
  }

  get isPolling(): boolean {
    return this.polling;
  }

  updateToken(token: string) {
    this.config.token = token;
  }

  updateChannels(channels: string[]) {
    this.config.channels = channels;
  }

  // Reset state (e.g., after re-login)
  reset() {
    this.lastEventId = 0;
    this.errorRetryCount = 0;
  }
}

// ════════════════════════════════════════════════════════════════
// REACT HOOK
// ════════════════════════════════════════════════════════════════

import { useEffect, useRef, useState, useCallback } from 'react';

interface UseLongPollingOptions {
  url: string;
  channels?: string[];
  token?: string;
  enabled?: boolean;
}

interface UseLongPollingReturn<T> {
  messages: T[];
  isConnected: boolean;
  error: Error | null;
  clearMessages: () => void;
}

function useLongPolling<T = any>(options: UseLongPollingOptions): UseLongPollingReturn<T> {
  const [messages, setMessages] = useState<T[]>([]);
  const [isConnected, setIsConnected] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  const clientRef = useRef<RobustLongPollingClient | null>(null);

  const clearMessages = useCallback(() => {
    setMessages([]);
  }, []);

  useEffect(() => {
    if (options.enabled === false) {
      clientRef.current?.stop();
      return;
    }

    const client = new RobustLongPollingClient({
      baseUrl: options.url,
      channels: options.channels,
      token: options.token,
      onMessage: (message) => {
        setMessages(prev => [...prev, message as T]);
      },
      onConnect: () => {
        setIsConnected(true);
        setError(null);
      },
      onDisconnect: () => {
        setIsConnected(false);
      },
      onError: (err) => {
        setError(err);
      }
    });

    clientRef.current = client;
    client.start();

    return () => {
      client.stop();
    };
  }, [options.url, options.channels?.join(','), options.token, options.enabled]);

  // Update token without reconnecting
  useEffect(() => {
    if (options.token && clientRef.current) {
      clientRef.current.updateToken(options.token);
    }
  }, [options.token]);

  return { messages, isConnected, error, clearMessages };
}

// Usage
function NotificationList() {
  const { messages, isConnected, error } = useLongPolling<Notification>({
    url: 'https://api.example.com',
    channels: ['notifications'],
    token: localStorage.getItem('token') || undefined
  });

  return (
    <div>
      <div className={`status ${isConnected ? 'connected' : 'disconnected'}`}>
        {isConnected ? '🟢 Connected' : '🔴 Disconnected'}
      </div>
      {error && <div className="error">{error.message}</div>}
      <ul>
        {messages.map((msg, i) => (
          <li key={i}>{msg.data.message}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

## 4. Handling Timeouts

```typescript
// ════════════════════════════════════════════════════════════════
// TIMEOUT HANDLING PATTERNS
// ════════════════════════════════════════════════════════════════

// Server-side timeout configuration
const TIMEOUT_CONFIG = {
  // Server holds request for this long
  serverTimeout: 30000,
  
  // Grace period for network latency
  gracePeriod: 5000,
  
  // Client should set timeout to this
  clientTimeout: 35000,
  
  // Early timeout response for slow clients
  slowClientThreshold: 25000
};

// ════════════════════════════════════════════════════════════════
// SERVER TIMEOUT HANDLING
// ════════════════════════════════════════════════════════════════

class TimeoutManager {
  private pendingRequests: Map<string, {
    res: express.Response;
    timeout: NodeJS.Timeout;
    startTime: number;
  }> = new Map();

  holdRequest(requestId: string, res: express.Response, onTimeout: () => void) {
    const timeout = setTimeout(() => {
      this.completeWithTimeout(requestId);
      onTimeout();
    }, TIMEOUT_CONFIG.serverTimeout);

    this.pendingRequests.set(requestId, {
      res,
      timeout,
      startTime: Date.now()
    });

    // Also set up early timeout for potentially slow clients
    this.setupEarlyTimeout(requestId);
  }

  private setupEarlyTimeout(requestId: string) {
    // Check connection health before main timeout
    setTimeout(() => {
      const pending = this.pendingRequests.get(requestId);
      if (!pending) return;

      // Try to write a small chunk to detect dead connections
      try {
        pending.res.write(''); // Empty write to check connection
      } catch {
        // Connection dead, clean up
        this.cancel(requestId);
      }
    }, TIMEOUT_CONFIG.slowClientThreshold);
  }

  completeWithData(requestId: string, data: any) {
    const pending = this.pendingRequests.get(requestId);
    if (!pending) return;

    clearTimeout(pending.timeout);
    this.pendingRequests.delete(requestId);

    try {
      pending.res.json(data);
    } catch {
      // Connection already closed
    }
  }

  completeWithTimeout(requestId: string) {
    const pending = this.pendingRequests.get(requestId);
    if (!pending) return;

    clearTimeout(pending.timeout);
    this.pendingRequests.delete(requestId);

    const duration = Date.now() - pending.startTime;

    try {
      pending.res.json({
        success: true,
        messages: [],
        timeout: true,
        duration
      });
    } catch {
      // Connection already closed
    }
  }

  cancel(requestId: string) {
    const pending = this.pendingRequests.get(requestId);
    if (!pending) return;

    clearTimeout(pending.timeout);
    this.pendingRequests.delete(requestId);
  }
}

// ════════════════════════════════════════════════════════════════
// CLIENT TIMEOUT HANDLING
// ════════════════════════════════════════════════════════════════

class TimeoutAwarePollClient {
  private async pollWithTimeout(): Promise<any> {
    const controller = new AbortController();
    
    // Set client-side timeout
    const timeoutId = setTimeout(() => {
      controller.abort();
    }, TIMEOUT_CONFIG.clientTimeout);

    try {
      const response = await fetch(this.pollUrl, {
        signal: controller.signal
      });

      clearTimeout(timeoutId);
      return response.json();

    } catch (error) {
      clearTimeout(timeoutId);

      if (error instanceof Error && error.name === 'AbortError') {
        // Timeout - this is expected, reconnect
        console.log('Poll timeout, reconnecting...');
        return { messages: [], timeout: true };
      }

      throw error;
    }
  }
}

// ════════════════════════════════════════════════════════════════
// ADAPTIVE TIMEOUT
// ════════════════════════════════════════════════════════════════

class AdaptiveTimeoutClient {
  private baseTimeout = 30000;
  private currentTimeout = 30000;
  private recentLatencies: number[] = [];
  private readonly maxLatencyHistory = 10;

  private adjustTimeout(latency: number) {
    this.recentLatencies.push(latency);
    
    if (this.recentLatencies.length > this.maxLatencyHistory) {
      this.recentLatencies.shift();
    }

    // Calculate P95 latency
    const sorted = [...this.recentLatencies].sort((a, b) => a - b);
    const p95Index = Math.floor(sorted.length * 0.95);
    const p95Latency = sorted[p95Index] || sorted[sorted.length - 1];

    // Adjust timeout based on latency
    // If latency is high, increase timeout
    // If latency is low, decrease toward base
    if (p95Latency > this.baseTimeout * 0.8) {
      this.currentTimeout = Math.min(this.currentTimeout * 1.2, 60000);
    } else if (p95Latency < this.baseTimeout * 0.3) {
      this.currentTimeout = Math.max(this.currentTimeout * 0.9, this.baseTimeout);
    }
  }

  private async poll() {
    const startTime = Date.now();
    
    try {
      const data = await this.pollWithTimeout(this.currentTimeout);
      const latency = Date.now() - startTime;
      this.adjustTimeout(latency);
      return data;
    } catch (error) {
      // On timeout, increase timeout for next request
      this.currentTimeout = Math.min(this.currentTimeout * 1.5, 60000);
      throw error;
    }
  }
}
```

---

## 5. Message Ordering & Recovery

```typescript
// ════════════════════════════════════════════════════════════════
// MESSAGE ORDERING AND RECOVERY
// ════════════════════════════════════════════════════════════════

interface OrderedMessage {
  id: number;          // Sequential ID for ordering
  uuid: string;        // Unique ID for deduplication
  channel: string;
  data: any;
  timestamp: number;
}

// ════════════════════════════════════════════════════════════════
// SERVER: MESSAGE STORE WITH ORDERING
// ════════════════════════════════════════════════════════════════

class MessageStore {
  private redis: Redis;
  private readonly keyPrefix = 'longpoll:messages';
  private readonly retentionMs = 5 * 60 * 1000; // 5 minutes
  private readonly maxMessages = 10000;

  constructor(redis: Redis) {
    this.redis = redis;
    this.startCleanup();
  }

  async publish(channel: string, data: any): Promise<OrderedMessage> {
    // Get next ID atomically
    const id = await this.redis.incr(`${this.keyPrefix}:counter`);
    
    const message: OrderedMessage = {
      id,
      uuid: `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
      channel,
      data,
      timestamp: Date.now()
    };

    // Store with score = id for ordering
    await this.redis.zAdd(`${this.keyPrefix}:${channel}`, {
      score: id,
      value: JSON.stringify(message)
    });

    // Also store in global stream
    await this.redis.zAdd(`${this.keyPrefix}:all`, {
      score: id,
      value: JSON.stringify(message)
    });

    return message;
  }

  async getMessages(
    channels: string[],
    afterId: number,
    limit = 100
  ): Promise<OrderedMessage[]> {
    if (channels.includes('*')) {
      // Get from global stream
      const raw = await this.redis.zRangeByScore(
        `${this.keyPrefix}:all`,
        afterId + 1,
        '+inf',
        { LIMIT: { offset: 0, count: limit } }
      );
      return raw.map(r => JSON.parse(r));
    }

    // Get from specific channels and merge
    const results = await Promise.all(
      channels.map(ch =>
        this.redis.zRangeByScore(
          `${this.keyPrefix}:${ch}`,
          afterId + 1,
          '+inf'
        )
      )
    );

    const messages = results
      .flat()
      .map(r => JSON.parse(r) as OrderedMessage)
      .sort((a, b) => a.id - b.id)
      .slice(0, limit);

    return messages;
  }

  async getLastId(): Promise<number> {
    const id = await this.redis.get(`${this.keyPrefix}:counter`);
    return id ? parseInt(id) : 0;
  }

  private startCleanup() {
    // Clean old messages every minute
    setInterval(async () => {
      const cutoffTime = Date.now() - this.retentionMs;
      
      // Get all channel keys
      const keys = await this.redis.keys(`${this.keyPrefix}:*`);
      
      for (const key of keys) {
        if (key.endsWith(':counter')) continue;
        
        // Remove messages older than retention
        // This requires storing timestamp as score, or iterating
        // For simplicity, we'll keep by count
        const count = await this.redis.zCard(key);
        if (count > this.maxMessages) {
          await this.redis.zRemRangeByRank(key, 0, count - this.maxMessages - 1);
        }
      }
    }, 60000);
  }
}

// ════════════════════════════════════════════════════════════════
// CLIENT: DEDUPLICATION AND ORDERING
// ════════════════════════════════════════════════════════════════

class OrderedMessageClient {
  private lastId = 0;
  private seenIds = new Set<string>(); // Track UUIDs for deduplication
  private readonly maxSeenIds = 1000;
  private messageBuffer: OrderedMessage[] = [];

  constructor(private onMessage: (message: OrderedMessage) => void) {}

  handleResponse(messages: OrderedMessage[]) {
    for (const message of messages) {
      // Deduplication
      if (this.seenIds.has(message.uuid)) {
        console.log('Duplicate message skipped:', message.uuid);
        continue;
      }

      // Track seen
      this.seenIds.add(message.uuid);
      if (this.seenIds.size > this.maxSeenIds) {
        // Remove oldest (convert to array, remove first half)
        const arr = Array.from(this.seenIds);
        arr.slice(0, this.maxSeenIds / 2).forEach(id => this.seenIds.delete(id));
      }

      // Add to buffer
      this.messageBuffer.push(message);
    }

    // Sort buffer by ID
    this.messageBuffer.sort((a, b) => a.id - b.id);

    // Deliver in-order messages
    this.deliverOrdered();
  }

  private deliverOrdered() {
    while (this.messageBuffer.length > 0) {
      const next = this.messageBuffer[0];

      // Check if this is the next expected message
      if (next.id === this.lastId + 1) {
        this.messageBuffer.shift();
        this.lastId = next.id;
        this.onMessage(next);
      } else if (next.id <= this.lastId) {
        // Already delivered (shouldn't happen with dedup)
        this.messageBuffer.shift();
      } else {
        // Gap detected - wait for missing messages
        console.log(`Gap detected: expected ${this.lastId + 1}, got ${next.id}`);
        break;
      }
    }
  }

  // Call this on reconnection to handle gaps
  syncFromLastId(): number {
    return this.lastId;
  }

  // Force skip gap if messages are lost
  skipToId(id: number) {
    this.lastId = id - 1;
    this.deliverOrdered();
  }
}

// ════════════════════════════════════════════════════════════════
// GAP DETECTION AND RECOVERY
// ════════════════════════════════════════════════════════════════

class GapRecoveryClient {
  private expectedNextId = 1;
  private gapTimeout: NodeJS.Timeout | null = null;
  private readonly gapTimeoutMs = 5000;

  handleMessages(messages: OrderedMessage[]) {
    for (const msg of messages) {
      if (msg.id < this.expectedNextId) {
        // Already seen
        continue;
      }

      if (msg.id > this.expectedNextId) {
        // Gap detected
        this.handleGap(this.expectedNextId, msg.id - 1);
      }

      this.deliverMessage(msg);
      this.expectedNextId = msg.id + 1;
    }
  }

  private handleGap(fromId: number, toId: number) {
    console.warn(`Message gap detected: ${fromId} to ${toId}`);

    // Clear existing gap timeout
    if (this.gapTimeout) {
      clearTimeout(this.gapTimeout);
    }

    // Try to recover missing messages
    this.requestMissingMessages(fromId, toId);

    // Set timeout to skip if recovery fails
    this.gapTimeout = setTimeout(() => {
      console.warn(`Skipping unrecovered messages: ${fromId} to ${toId}`);
      this.expectedNextId = toId + 1;
    }, this.gapTimeoutMs);
  }

  private async requestMissingMessages(fromId: number, toId: number) {
    try {
      // Request specific message range from server
      const response = await fetch(
        `/api/messages/range?from=${fromId}&to=${toId}`
      );
      const { messages } = await response.json();
      
      // Process recovered messages
      for (const msg of messages) {
        this.deliverMessage(msg);
      }

      if (this.gapTimeout) {
        clearTimeout(this.gapTimeout);
        this.gapTimeout = null;
      }
    } catch (error) {
      console.error('Failed to recover messages:', error);
    }
  }

  private deliverMessage(msg: OrderedMessage) {
    // Deliver to application
    console.log('Delivering message:', msg.id);
  }
}
```

---

## 6. Scaling Long Polling

```typescript
// ════════════════════════════════════════════════════════════════
// SCALING LONG POLLING ACROSS MULTIPLE SERVERS
// ════════════════════════════════════════════════════════════════

import { createClient, RedisClientType } from 'redis';
import express from 'express';
import cluster from 'cluster';
import os from 'os';

// ════════════════════════════════════════════════════════════════
// MULTI-SERVER ARCHITECTURE
// ════════════════════════════════════════════════════════════════

class ScalableLongPollServer {
  private redis: RedisClientType;
  private subscriber: RedisClientType;
  private pendingRequests: Map<string, PendingRequest> = new Map();
  private serverId: string;

  constructor(private app: express.Application) {
    this.serverId = `server-${process.pid}-${Date.now()}`;
    this.init();
  }

  private async init() {
    // Setup Redis
    this.redis = createClient({ url: process.env.REDIS_URL });
    this.subscriber = this.redis.duplicate();

    await Promise.all([
      this.redis.connect(),
      this.subscriber.connect()
    ]);

    // Subscribe to broadcast channel
    await this.subscriber.subscribe('longpoll:broadcast', (message) => {
      this.handleBroadcast(JSON.parse(message));
    });

    // Subscribe to server-specific channel for targeted messages
    await this.subscriber.subscribe(`longpoll:server:${this.serverId}`, (message) => {
      this.handleDirectMessage(JSON.parse(message));
    });

    // Register server in Redis
    await this.registerServer();

    this.setupRoutes();
  }

  private async registerServer() {
    // Store server info with TTL
    await this.redis.hSet('longpoll:servers', this.serverId, JSON.stringify({
      pid: process.pid,
      startedAt: Date.now(),
      host: os.hostname()
    }));

    // Keep-alive ping
    setInterval(async () => {
      await this.redis.hSet('longpoll:servers', this.serverId, JSON.stringify({
        pid: process.pid,
        connections: this.pendingRequests.size,
        lastPing: Date.now()
      }));
    }, 10000);
  }

  private setupRoutes() {
    this.app.get('/api/poll', async (req, res) => {
      const requestId = `${this.serverId}-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
      const userId = req.query.userId as string;
      const lastEventId = parseInt(req.query.lastEventId as string) || 0;

      // Store which server this user is connected to
      if (userId) {
        await this.redis.hSet('longpoll:user-servers', userId, this.serverId);
      }

      // Check for pending messages
      const messages = await this.getPendingMessages(userId, lastEventId);
      if (messages.length > 0) {
        return res.json({ messages, lastEventId: messages[messages.length - 1].id });
      }

      // Hold request
      const timeout = setTimeout(() => {
        this.completeRequest(requestId, []);
      }, 30000);

      this.pendingRequests.set(requestId, {
        res,
        userId,
        lastEventId,
        timeout
      });

      req.on('close', () => {
        clearTimeout(timeout);
        this.pendingRequests.delete(requestId);
        if (userId) {
          this.redis.hDel('longpoll:user-servers', userId);
        }
      });
    });
  }

  private handleBroadcast(data: { channel: string; message: any }) {
    // Send to all local pending requests
    this.pendingRequests.forEach((pending, requestId) => {
      if (this.shouldReceive(pending, data.channel)) {
        this.completeRequest(requestId, [data.message]);
      }
    });
  }

  private handleDirectMessage(data: { userId: string; message: any }) {
    // Send to specific user's local connection
    this.pendingRequests.forEach((pending, requestId) => {
      if (pending.userId === data.userId) {
        this.completeRequest(requestId, [data.message]);
      }
    });
  }

  // ══════════════════════════════════════════════════════════════
  // PUBLIC BROADCAST METHODS
  // ══════════════════════════════════════════════════════════════

  async broadcast(channel: string, data: any) {
    const message = {
      id: await this.redis.incr('longpoll:message-id'),
      channel,
      data,
      timestamp: Date.now()
    };

    // Store message for recovery
    await this.storeMessage(message);

    // Publish to all servers
    await this.redis.publish('longpoll:broadcast', JSON.stringify({
      channel,
      message
    }));
  }

  async sendToUser(userId: string, data: any) {
    const message = {
      id: await this.redis.incr('longpoll:message-id'),
      channel: `user:${userId}`,
      data,
      timestamp: Date.now()
    };

    await this.storeMessage(message);

    // Find which server the user is connected to
    const serverId = await this.redis.hGet('longpoll:user-servers', userId);
    
    if (serverId) {
      // Send directly to that server
      await this.redis.publish(`longpoll:server:${serverId}`, JSON.stringify({
        userId,
        message
      }));
    }
    // If not connected, message is stored for when they poll next
  }

  private async storeMessage(message: any) {
    await this.redis.zAdd('longpoll:messages', {
      score: message.id,
      value: JSON.stringify(message)
    });

    // Cleanup old messages
    const count = await this.redis.zCard('longpoll:messages');
    if (count > 10000) {
      await this.redis.zRemRangeByRank('longpoll:messages', 0, count - 10001);
    }
  }

  private async getPendingMessages(userId: string | undefined, lastEventId: number): Promise<any[]> {
    const messages = await this.redis.zRangeByScore(
      'longpoll:messages',
      lastEventId + 1,
      '+inf'
    );

    return messages
      .map(m => JSON.parse(m))
      .filter(m => 
        m.channel === '*' || 
        !userId || 
        m.channel === `user:${userId}`
      );
  }

  private shouldReceive(pending: PendingRequest, channel: string): boolean {
    return channel === '*' || channel === `user:${pending.userId}`;
  }

  private completeRequest(requestId: string, messages: any[]) {
    const pending = this.pendingRequests.get(requestId);
    if (!pending) return;

    clearTimeout(pending.timeout);
    this.pendingRequests.delete(requestId);

    try {
      pending.res.json({
        messages,
        lastEventId: messages.length > 0 ? messages[messages.length - 1].id : pending.lastEventId
      });
    } catch {
      // Connection closed
    }
  }
}

// ════════════════════════════════════════════════════════════════
// CLUSTER MODE
// ════════════════════════════════════════════════════════════════

if (cluster.isPrimary) {
  const numCPUs = os.cpus().length;

  console.log(`Primary ${process.pid} is running`);
  console.log(`Forking ${numCPUs} workers...`);

  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died. Restarting...`);
    cluster.fork();
  });
} else {
  const app = express();
  const server = new ScalableLongPollServer(app);

  app.listen(3000, () => {
    console.log(`Worker ${process.pid} started on port 3000`);
  });
}
```

---

## 7. Transport Upgrade Pattern

```typescript
// ════════════════════════════════════════════════════════════════
// TRANSPORT UPGRADE: LONG POLLING → WEBSOCKET
// ════════════════════════════════════════════════════════════════

// This is how Socket.io handles transport fallback

type TransportType = 'websocket' | 'longpoll';

interface TransportConfig {
  upgradeTimeout: number;
  pingInterval: number;
  pingTimeout: number;
}

class UniversalTransport {
  private transport: TransportType = 'longpoll';
  private ws: WebSocket | null = null;
  private longPollClient: RobustLongPollingClient | null = null;
  private onMessage: (data: any) => void;
  private upgradeAttempted = false;

  constructor(
    private baseUrl: string,
    onMessage: (data: any) => void,
    private config: TransportConfig = {
      upgradeTimeout: 10000,
      pingInterval: 25000,
      pingTimeout: 5000
    }
  ) {
    this.onMessage = onMessage;
  }

  async connect() {
    // Start with long polling (reliable, works everywhere)
    this.startLongPolling();

    // Attempt WebSocket upgrade after initial connection
    setTimeout(() => {
      this.attemptUpgrade();
    }, 1000);
  }

  private startLongPolling() {
    this.longPollClient = new RobustLongPollingClient({
      baseUrl: this.baseUrl,
      onMessage: (msg) => this.handleMessage(msg),
      onConnect: () => console.log('Long polling connected'),
      onDisconnect: () => this.handleDisconnect()
    });

    this.longPollClient.start();
    this.transport = 'longpoll';
  }

  private async attemptUpgrade() {
    if (this.upgradeAttempted) return;
    this.upgradeAttempted = true;

    const wsUrl = this.baseUrl
      .replace('http://', 'ws://')
      .replace('https://', 'wss://') + '/ws';

    try {
      const ws = new WebSocket(wsUrl);

      const upgradeTimeout = setTimeout(() => {
        if (ws.readyState !== WebSocket.OPEN) {
          ws.close();
          console.log('WebSocket upgrade timeout, staying on long polling');
        }
      }, this.config.upgradeTimeout);

      ws.onopen = () => {
        clearTimeout(upgradeTimeout);
        this.completeUpgrade(ws);
      };

      ws.onerror = () => {
        clearTimeout(upgradeTimeout);
        console.log('WebSocket not available, continuing with long polling');
      };

    } catch (error) {
      console.log('WebSocket upgrade failed:', error);
    }
  }

  private completeUpgrade(ws: WebSocket) {
    // Stop long polling
    this.longPollClient?.stop();
    this.longPollClient = null;

    // Switch to WebSocket
    this.ws = ws;
    this.transport = 'websocket';

    console.log('Upgraded to WebSocket');

    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      this.handleMessage(data);
    };

    ws.onclose = () => {
      console.log('WebSocket closed, falling back to long polling');
      this.ws = null;
      this.transport = 'longpoll';
      this.upgradeAttempted = false;
      this.startLongPolling();
    };

    ws.onerror = () => {
      ws.close();
    };

    // Setup ping-pong for WebSocket
    this.setupPingPong(ws);
  }

  private setupPingPong(ws: WebSocket) {
    let pingTimeout: NodeJS.Timeout | null = null;

    const ping = setInterval(() => {
      if (ws.readyState !== WebSocket.OPEN) {
        clearInterval(ping);
        return;
      }

      ws.send(JSON.stringify({ type: 'ping' }));

      pingTimeout = setTimeout(() => {
        console.log('Ping timeout, closing WebSocket');
        ws.close();
      }, this.config.pingTimeout);
    }, this.config.pingInterval);

    // Listen for pong
    const originalOnMessage = ws.onmessage;
    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      if (data.type === 'pong') {
        if (pingTimeout) clearTimeout(pingTimeout);
        return;
      }
      originalOnMessage?.call(ws, event);
    };
  }

  private handleMessage(data: any) {
    this.onMessage(data);
  }

  private handleDisconnect() {
    if (this.transport === 'longpoll') {
      // Long polling will auto-reconnect
    }
  }

  send(data: any) {
    if (this.transport === 'websocket' && this.ws?.readyState === WebSocket.OPEN) {
      this.ws.send(JSON.stringify(data));
    } else {
      // Send via HTTP POST
      fetch(`${this.baseUrl}/api/send`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data)
      });
    }
  }

  disconnect() {
    this.longPollClient?.stop();
    this.ws?.close();
  }

  get currentTransport(): TransportType {
    return this.transport;
  }
}

// Usage
const transport = new UniversalTransport(
  'https://api.example.com',
  (message) => {
    console.log('Received:', message);
  }
);

transport.connect();

// Later
console.log('Current transport:', transport.currentTransport);
// Will be 'longpoll' initially, then 'websocket' after upgrade
```

---

## 8. Real-World Patterns

### Chat Application with Long Polling Fallback

```typescript
// ════════════════════════════════════════════════════════════════
// CHAT WITH LONG POLLING FALLBACK
// ════════════════════════════════════════════════════════════════

class ChatClient {
  private transport: UniversalTransport;
  private roomId: string | null = null;
  private onMessageCallback: ((message: ChatMessage) => void) | null = null;

  constructor(private baseUrl: string) {
    this.transport = new UniversalTransport(
      baseUrl,
      (data) => this.handleIncoming(data)
    );
  }

  async connect() {
    await this.transport.connect();
  }

  joinRoom(roomId: string) {
    this.roomId = roomId;
    this.transport.send({
      type: 'join',
      roomId
    });
  }

  leaveRoom() {
    if (this.roomId) {
      this.transport.send({
        type: 'leave',
        roomId: this.roomId
      });
      this.roomId = null;
    }
  }

  sendMessage(text: string) {
    if (!this.roomId) {
      throw new Error('Not in a room');
    }

    this.transport.send({
      type: 'message',
      roomId: this.roomId,
      text,
      timestamp: Date.now()
    });
  }

  onMessage(callback: (message: ChatMessage) => void) {
    this.onMessageCallback = callback;
  }

  private handleIncoming(data: any) {
    switch (data.type) {
      case 'message':
        this.onMessageCallback?.(data as ChatMessage);
        break;
      case 'user-joined':
        console.log('User joined:', data.userId);
        break;
      case 'user-left':
        console.log('User left:', data.userId);
        break;
    }
  }

  disconnect() {
    this.leaveRoom();
    this.transport.disconnect();
  }
}

// Server-side handler
app.get('/api/poll', authenticate, async (req, res) => {
  const userId = req.user.id;
  const roomId = req.query.roomId as string;
  const lastEventId = parseInt(req.query.lastEventId as string) || 0;

  // Get messages for user's room
  const messages = await getMessagesForRoom(roomId, lastEventId);
  
  if (messages.length > 0) {
    return res.json({ messages, lastEventId: messages[messages.length - 1].id });
  }

  // Hold request...
  holdRequest(userId, roomId, res);
});

app.post('/api/send', authenticate, express.json(), async (req, res) => {
  const userId = req.user.id;
  const { type, roomId, text } = req.body;

  switch (type) {
    case 'message':
      await broadcastToRoom(roomId, {
        type: 'message',
        userId,
        text,
        timestamp: Date.now()
      });
      break;
    case 'join':
      await addUserToRoom(userId, roomId);
      await broadcastToRoom(roomId, { type: 'user-joined', userId });
      break;
    case 'leave':
      await removeUserFromRoom(userId, roomId);
      await broadcastToRoom(roomId, { type: 'user-left', userId });
      break;
  }

  res.json({ success: true });
});
```

---

## 9. Common Pitfalls

```typescript
// ════════════════════════════════════════════════════════════════
// LONG POLLING PITFALLS AND SOLUTIONS
// ════════════════════════════════════════════════════════════════

// ❌ PITFALL 1: Not handling client disconnection
// Problem: Server holds references to closed connections

// Bad
const pending = new Map();
app.get('/poll', (req, res) => {
  pending.set(req.id, res);
  // Never cleaned up!
});

// Good
app.get('/poll', (req, res) => {
  const id = generateId();
  pending.set(id, res);
  
  req.on('close', () => {
    pending.delete(id); // Clean up!
  });
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 2: No timeout on held requests
// Problem: Requests hang forever

// Bad
app.get('/poll', (req, res) => {
  pending.set(req.id, res);
  // Waits forever for data
});

// Good
app.get('/poll', (req, res) => {
  const timeout = setTimeout(() => {
    res.json({ messages: [] }); // Empty response
    pending.delete(req.id);
  }, 30000);
  
  pending.set(req.id, { res, timeout });
  
  req.on('close', () => {
    clearTimeout(timeout);
    pending.delete(req.id);
  });
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 3: Not tracking message IDs
// Problem: Messages lost during reconnection

// Bad - No way to know what was missed
app.get('/poll', (req, res) => {
  // Just returns latest data, no history
});

// Good - Track and send missed messages
app.get('/poll', (req, res) => {
  const lastEventId = req.query.lastEventId;
  const missed = getMessagesSince(lastEventId);
  if (missed.length > 0) {
    return res.json({ messages: missed });
  }
  // Hold for new messages...
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 4: Blocking event loop
// Problem: Holding requests blocks other requests

// Bad - Synchronous operations
app.get('/poll', (req, res) => {
  while (!hasNewData()) {
    // Busy wait - blocks everything!
  }
  res.json(getData());
});

// Good - Non-blocking with event emitter
const emitter = new EventEmitter();

app.get('/poll', (req, res) => {
  const handler = (data) => {
    res.json(data);
  };
  
  emitter.once('data', handler);
  
  setTimeout(() => {
    emitter.off('data', handler);
    res.json({ messages: [] });
  }, 30000);
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 5: No error handling on response write
// Problem: Server crashes when writing to closed connection

// Bad
function broadcast(data) {
  pending.forEach(({ res }) => {
    res.json(data); // May throw if connection closed!
  });
}

// Good
function broadcast(data) {
  pending.forEach(({ res }, id) => {
    try {
      if (!res.writableEnded) {
        res.json(data);
      }
    } catch (error) {
      console.error('Write failed:', error);
    } finally {
      pending.delete(id);
    }
  });
}

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 6: Client not immediately reconnecting
// Problem: Delay between receiving response and new request

// Bad - Waiting before reconnecting
async function poll() {
  const data = await fetch('/poll');
  handleData(data);
  await sleep(1000); // Unnecessary delay!
  poll();
}

// Good - Immediate reconnection
async function poll() {
  const data = await fetch('/poll');
  handleData(data);
  poll(); // Immediately reconnect (or use setImmediate)
}

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 7: No exponential backoff on errors
// Problem: Hammering server during outage

// Bad
async function poll() {
  try {
    await fetch('/poll');
  } catch {
    poll(); // Immediate retry - hammers server!
  }
}

// Good
let retryDelay = 1000;
const maxDelay = 30000;

async function poll() {
  try {
    await fetch('/poll');
    retryDelay = 1000; // Reset on success
  } catch {
    await sleep(retryDelay);
    retryDelay = Math.min(retryDelay * 2, maxDelay);
  }
  poll();
}

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 8: Not handling concurrent requests per user
// Problem: User opens multiple tabs, each polling

// Bad - Each tab creates separate connection
// User has 5 tabs = 5 held connections

// Good - Deduplicate or use SharedWorker
// Option 1: Server limits connections per user
const userConnections = new Map<string, number>();
const MAX_PER_USER = 2;

app.get('/poll', (req, res) => {
  const userId = req.user.id;
  const count = userConnections.get(userId) || 0;
  
  if (count >= MAX_PER_USER) {
    return res.status(429).json({ error: 'Too many connections' });
  }
  
  userConnections.set(userId, count + 1);
  // ...
});

// Option 2: Client uses SharedWorker (browser)
// Single connection shared across tabs

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 9: Sending too much data per response
// Problem: Large responses slow down client

// Bad - Sending all history
app.get('/poll', (req, res) => {
  const allMessages = getAllMessages();
  res.json({ messages: allMessages }); // Could be thousands!
});

// Good - Paginate and limit
app.get('/poll', (req, res) => {
  const lastId = req.query.lastEventId;
  const messages = getMessagesSince(lastId, { limit: 100 });
  res.json({ messages, hasMore: messages.length === 100 });
});
```

---

## 10. Interview Questions

### Basic Questions

**Q: "What is long polling?"**
> "Long polling is a technique where the client sends an HTTP request and the server holds it open until new data is available or a timeout occurs. When the server responds, the client immediately sends a new request. This simulates real-time server push using standard HTTP, working through all proxies and firewalls where WebSocket might be blocked."

**Q: "How is long polling different from regular polling?"**
> "Regular polling sends requests at fixed intervals (e.g., every 5 seconds), most returning empty. Long polling holds the request until data is available, responding only when there's something to send. This is more efficient - fewer requests, lower latency when data arrives, but the server must hold connections open."

**Q: "What are the drawbacks of long polling?"**
> "Higher overhead than WebSocket - HTTP headers on every reconnection (500-800 bytes vs 2-6 bytes for WebSocket frame). More server resources holding connections open. Slight latency on reconnection. More complex error handling. But it works everywhere HTTP works, which is its main advantage."

### Intermediate Questions

**Q: "How do you handle message ordering and lost messages?"**
> "Track sequential message IDs server-side. Client sends last received ID with each poll. Server sends all messages since that ID. On reconnection, client requests from its last ID, server replays missed messages. For critical ordering, buffer messages client-side and deliver in-order, detecting gaps for recovery requests."

**Q: "How do you handle timeouts?"**
> "Server timeout at 30 seconds (below typical proxy timeouts), responds with empty body. Client timeout slightly longer (35 seconds) to account for network latency. Client immediately reconnects after any response. Exponential backoff (1s, 2s, 4s, max 30s) on errors. Server cleanup on req.close event."

**Q: "How do you scale long polling?"**
> "Each held request is a connection, so use non-blocking server (Node.js, Go). Store messages in Redis for multi-server broadcasting. Use Redis pub/sub - publish messages, all servers subscribe and notify their local clients. Track which server each user is connected to for targeted messages."

### Advanced Questions

**Q: "How does Socket.io use long polling?"**
> "Socket.io starts with HTTP long polling for fast initial connection (no handshake delay). It then attempts WebSocket upgrade. If upgrade succeeds, switches to WebSocket. If upgrade fails (proxy issues, firewall), continues with long polling. This gives best of both worlds - reliability of long polling, performance of WebSocket when available."

**Q: "How would you implement a transport upgrade pattern?"**
> "Start with long polling (works everywhere). After initial connection, attempt WebSocket. Set upgrade timeout (10 seconds). If WebSocket opens successfully, stop long polling and switch. If WebSocket fails or closes, fall back to long polling. Track last message ID across transport switches for seamless transition. Implement ping-pong on WebSocket to detect dead connections."

**Q: "What are the performance considerations?"**
> "Each held connection consumes memory. Monitor connection count and memory usage. Use connection limits per user. Implement backpressure - if server is overloaded, respond immediately with retry-after header. Batch messages when possible to reduce reconnection frequency. For high-frequency data, consider upgrading to WebSocket or SSE."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│               LONG POLLING IMPLEMENTATION CHECKLIST             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SERVER:                                                        │
│  □ Hold requests until data or timeout (30s)                   │
│  □ Clean up on req.close event                                 │
│  □ Clear timeout when responding                               │
│  □ Handle write errors gracefully                              │
│  □ Track message IDs for recovery                              │
│  □ Store messages for reconnection replay                      │
│                                                                 │
│  CLIENT:                                                        │
│  □ Immediately reconnect after response                        │
│  □ Track last event ID                                         │
│  □ Send last ID with each request                              │
│  □ Exponential backoff on errors                               │
│  □ Handle timeout (client-side) slightly > server              │
│  □ Deduplication of messages                                   │
│                                                                 │
│  SCALING:                                                       │
│  □ Use non-blocking server (Node.js, Go)                       │
│  □ Redis pub/sub for multi-server                              │
│  □ Track user→server mapping                                   │
│  □ Monitor connection counts                                   │
│  □ Implement connection limits                                 │
│                                                                 │
│  RELIABILITY:                                                   │
│  □ Message ID sequence for ordering                            │
│  □ Store messages for recovery window                          │
│  □ Gap detection and recovery                                  │
│  □ Graceful degradation on overload                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

TIMEOUT RECOMMENDATIONS:
┌────────────────────────────────────────┐
│ Server hold timeout:  30 seconds       │
│ Client timeout:       35 seconds       │
│ Error retry start:    1 second         │
│ Error retry max:      30 seconds       │
│ Reconnect delay:      0-100ms (jitter) │
└────────────────────────────────────────┘

WHEN TO USE LONG POLLING:
✅ WebSocket blocked (corporate firewalls)
✅ Load balancer doesn't support WebSocket
✅ Universal fallback needed
✅ Simple implementation required
✅ Browser compatibility (legacy)

WHEN NOT TO USE:
❌ High-frequency updates (use WebSocket)
❌ Binary data (use WebSocket)
❌ Bidirectional real-time (use WebSocket)
❌ Modern infrastructure with WS support
```

---

*Last updated: February 2026*

