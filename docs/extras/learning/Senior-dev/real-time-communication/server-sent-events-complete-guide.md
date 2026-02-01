# 📡 Server-Sent Events (SSE) - Complete Guide

> A comprehensive guide to Server-Sent Events - when to use vs WebSockets, EventSource API, implementation patterns, and real-world use cases.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Server-Sent Events (SSE) is a server push technology enabling one-way real-time updates from server to client over a single HTTP connection, using a simple text-based protocol that's natively supported by browsers through the EventSource API."

### The 6 Key Concepts (Remember These!)
```
1. ONE-WAY         → Server to client only (client uses HTTP for requests)
2. HTTP-BASED      → Uses standard HTTP, works with existing infrastructure
3. AUTO-RECONNECT  → Browser handles reconnection automatically
4. TEXT PROTOCOL   → Simple text format (event: data: id:)
5. LAST-EVENT-ID   → Built-in resume capability after disconnection
6. EVENTSOURCE     → Native browser API, no library needed
```

### SSE vs WebSocket Comparison
```
┌─────────────────────────────────────────────────────────────────┐
│              SSE vs WebSocket - Quick Comparison                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Feature              │ SSE              │ WebSocket            │
│  ─────────────────────┼──────────────────┼─────────────────────│
│  Direction            │ Server→Client    │ Bidirectional        │
│  Protocol             │ HTTP             │ WebSocket (ws://)    │
│  Connection           │ HTTP/1.1+        │ Upgrade required     │
│  Auto-reconnect       │ ✅ Built-in      │ ❌ Manual            │
│  Event IDs            │ ✅ Built-in      │ ❌ Manual            │
│  Binary data          │ ❌ Text only     │ ✅ Supported         │
│  Browser support      │ All modern       │ All modern           │
│  Max connections      │ 6 per domain*    │ ~255 per domain      │
│  Proxy friendly       │ ✅ Yes           │ ⚠️ Sometimes         │
│  HTTP/2 multiplexing  │ ✅ Yes           │ ❌ No                │
│  Complexity           │ Simple           │ More complex         │
│                                                                 │
│  * HTTP/2 removes this limit via multiplexing                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"EventSource API"** | "Browsers natively support SSE via the EventSource API" |
| **"Last-Event-ID"** | "SSE uses Last-Event-ID header for automatic reconnection resume" |
| **"Text/event-stream"** | "The MIME type for SSE is text/event-stream" |
| **"Retry directive"** | "We set the retry directive to control reconnection timing" |
| **"Named events"** | "We use named events to separate different message types" |
| **"HTTP/2 multiplexing"** | "With HTTP/2, SSE connections don't count against the connection limit" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Max connections (HTTP/1.1) | **6 per domain** | Browser limit (use HTTP/2!) |
| Default retry | **3 seconds** | Browser reconnect delay |
| Keep-alive comment | **Every 15-30s** | Prevent proxy timeouts |
| Max event size | **No hard limit** | But memory considerations |
| Connection timeout | **Varies** | Depends on proxy/server config |

### SSE Message Format
```
┌─────────────────────────────────────────────────────────────────┐
│                    SSE MESSAGE FORMAT                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  id: 12345                    ← Event ID (for reconnection)    │
│  event: notification          ← Event type/name                 │
│  data: {"msg": "Hello"}       ← Payload (can be multi-line)    │
│  retry: 5000                  ← Reconnect delay in ms           │
│                               ← Empty line = end of event       │
│                                                                 │
│  SIMPLE MESSAGE:                                                │
│  data: Hello World                                              │
│                                                                 │
│  MULTI-LINE DATA:                                               │
│  data: First line                                               │
│  data: Second line                                              │
│  data: Third line                                               │
│                                                                 │
│  JSON PAYLOAD:                                                  │
│  data: {"user": "john", "action": "login"}                     │
│                                                                 │
│  NAMED EVENT:                                                   │
│  event: user-joined                                             │
│  data: {"userId": "123", "name": "John"}                       │
│                                                                 │
│  KEEP-ALIVE (comment):                                          │
│  : ping                       ← Lines starting with : ignored  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### The "Wow" Statement (Memorize This!)
> "I choose SSE over WebSockets when we only need server-to-client streaming - like live notifications, real-time feeds, or dashboard updates. SSE is simpler: it's just HTTP, works through all proxies, and the browser handles reconnection with Last-Event-ID for missed event recovery automatically. With HTTP/2, the connection limit is eliminated via multiplexing. For the server, it's a standard HTTP response with `Content-Type: text/event-stream` that never closes. We use named events to route different message types, and periodic comment pings to keep connections alive through load balancers."

### Quick Architecture Drawing (Draw This!)
```
           CLIENT                              SERVER
    ┌─────────────────┐                ┌─────────────────┐
    │                 │    HTTP GET    │                 │
    │   EventSource   │───────────────▶│   SSE Handler   │
    │                 │                │                 │
    │   onmessage     │◀───event 1─────│                 │
    │   onmessage     │◀───event 2─────│   Event Stream  │
    │   onmessage     │◀───event 3─────│                 │
    │                 │◀───ping (: )───│   Keep-alive    │
    │   onmessage     │◀───event 4─────│                 │
    │                 │                │                 │
    │   [disconnect]  │       ╳        │                 │
    │                 │                │                 │
    │   [auto-retry]  │    GET with    │                 │
    │                 │───Last-Event-ID▶│   Resume from   │
    │   onmessage     │◀───event 5─────│   missed event  │
    │                 │                │                 │
    └─────────────────┘                └─────────────────┘
```

### Interview Rapid Fire (Practice These!)

**Q: "What is Server-Sent Events?"**
> "SSE is a standard for pushing real-time updates from server to client over HTTP. One-way, text-based, with automatic reconnection."

**Q: "When would you use SSE over WebSockets?"**
> "When you only need server-to-client streaming: notifications, live feeds, stock tickers. Simpler than WebSockets, better proxy support."

**Q: "How does SSE handle reconnection?"**
> "Browser automatically reconnects. Server sends event IDs, browser sends Last-Event-ID header on reconnect, server resumes from there."

**Q: "What's the connection limit issue with SSE?"**
> "HTTP/1.1 browsers limit 6 connections per domain. With HTTP/2, multiplexing eliminates this. Solution: use HTTP/2 or domain sharding."

**Q: "How do you send different types of events?"**
> "Use the `event:` field with named events. Client listens with `addEventListener('eventName', handler)` instead of `onmessage`."

**Q: "What's the MIME type for SSE?"**
> "`text/event-stream`. Response must have this Content-Type and no caching headers."

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "What is SSE and when should I use it?"

**Junior Answer:**
> "SSE lets the server push data to the browser in real-time."

**Senior Answer:**
> "Server-Sent Events is a W3C standard for server-to-client streaming over HTTP. Unlike WebSockets which require a protocol upgrade and are bidirectional, SSE is unidirectional and uses plain HTTP.

Use SSE when:
1. **One-way updates** - You only need server→client (client can POST separately)
2. **Text data** - Your payloads are JSON or text, not binary
3. **Simplicity matters** - No special server requirements, just HTTP responses
4. **Proxy compatibility** - SSE works through HTTP proxies without issues
5. **Automatic recovery** - You want built-in reconnection with event resumption

Don't use SSE when:
1. You need bidirectional real-time communication
2. You're sending binary data
3. You need high-frequency messaging (WebSocket is more efficient)
4. You're stuck on HTTP/1.1 with many concurrent streams (6 connection limit)"

### When Asked: "How does SSE handle reconnection?"

**Junior Answer:**
> "The browser reconnects automatically when the connection drops."

**Senior Answer:**
> "SSE has built-in reconnection with state recovery. Here's how it works:

1. **Server sends event IDs**: Each event can have an `id:` field
2. **Browser tracks last ID**: Stores the most recent event ID received
3. **On disconnect**: Browser waits (default 3 seconds, configurable via `retry:`)
4. **On reconnect**: Browser sends `Last-Event-ID` header with the last received ID
5. **Server resumes**: Server can use this ID to send missed events

This is significant because with WebSockets, you implement all of this yourself. SSE gives you durable event streams with minimal code.

The server needs to store enough event history to handle reconnections. Common approach: Redis sorted set with timestamp scores, expire old events after TTL."

### Follow-up Questions to Expect:

| They Ask | You Answer |
|----------|------------|
| "What about the 6 connection limit?" | "HTTP/1.1 browser limitation. HTTP/2 multiplexes over one connection, eliminating it. Use HTTP/2 or as workaround, domain sharding." |
| "Can you send binary data?" | "No, SSE is text-only. Base64 encode if needed, but WebSockets are better for binary." |
| "How do you authenticate SSE?" | "Standard HTTP auth. Cookies work, or pass token in query string. EventSource doesn't support custom headers easily." |
| "What about cross-origin?" | "Standard CORS. Set Access-Control-Allow-Origin on server. EventSource accepts withCredentials option." |

---

## 📚 Table of Contents

1. [EventSource API](#1-eventsource-api)
2. [Server Implementation](#2-server-implementation)
3. [Authentication & Security](#3-authentication--security)
4. [Reconnection & Recovery](#4-reconnection--recovery)
5. [Scaling SSE](#5-scaling-sse)
6. [Performance Optimization](#6-performance-optimization)
7. [Real-World Patterns](#7-real-world-patterns)
8. [SSE vs WebSocket Decision Guide](#8-sse-vs-websocket-decision-guide)
9. [Common Pitfalls](#9-common-pitfalls)
10. [Interview Questions](#10-interview-questions)

---

## 1. EventSource API

### Basic Client Implementation

```typescript
// ════════════════════════════════════════════════════════════════
// NATIVE EVENTSOURCE CLIENT
// ════════════════════════════════════════════════════════════════

// Simple connection
const eventSource = new EventSource('/api/events');

// Connection opened
eventSource.onopen = (event) => {
  console.log('SSE connection opened');
};

// Default message handler (events without 'event:' field)
eventSource.onmessage = (event) => {
  console.log('Message:', event.data);
  // event.data is always a string, parse if JSON
  const data = JSON.parse(event.data);
};

// Error handler (includes reconnection)
eventSource.onerror = (event) => {
  if (eventSource.readyState === EventSource.CONNECTING) {
    console.log('Reconnecting...');
  } else if (eventSource.readyState === EventSource.CLOSED) {
    console.log('Connection closed');
  }
};

// Close connection when done
eventSource.close();

// ════════════════════════════════════════════════════════════════
// NAMED EVENTS
// ════════════════════════════════════════════════════════════════

// Server sends: event: notification\ndata: {...}
eventSource.addEventListener('notification', (event) => {
  const notification = JSON.parse(event.data);
  showNotification(notification);
});

// Server sends: event: user-status\ndata: {...}
eventSource.addEventListener('user-status', (event) => {
  const status = JSON.parse(event.data);
  updateUserStatus(status);
});

// Server sends: event: heartbeat\ndata: ping
eventSource.addEventListener('heartbeat', (event) => {
  console.log('Heartbeat received');
});

// ════════════════════════════════════════════════════════════════
// WITH CREDENTIALS (CORS)
// ════════════════════════════════════════════════════════════════

// Send cookies with cross-origin request
const eventSource = new EventSource('https://api.example.com/events', {
  withCredentials: true
});

// ════════════════════════════════════════════════════════════════
// READYSTATE VALUES
// ════════════════════════════════════════════════════════════════
// EventSource.CONNECTING (0) - Connecting or reconnecting
// EventSource.OPEN (1) - Connected
// EventSource.CLOSED (2) - Connection closed, won't reconnect
```

### Enhanced Client with Custom Features

```typescript
// ════════════════════════════════════════════════════════════════
// ENHANCED SSE CLIENT
// ════════════════════════════════════════════════════════════════

type EventHandler = (data: any) => void;

interface SSEClientOptions {
  url: string;
  withCredentials?: boolean;
  onConnect?: () => void;
  onDisconnect?: () => void;
  onError?: (error: Event) => void;
  maxRetries?: number;
  headers?: Record<string, string>; // For polyfill
}

class SSEClient {
  private eventSource: EventSource | null = null;
  private handlers: Map<string, Set<EventHandler>> = new Map();
  private connected = false;
  private retryCount = 0;
  private options: SSEClientOptions;
  private lastEventId: string | null = null;

  constructor(options: SSEClientOptions) {
    this.options = {
      maxRetries: Infinity,
      withCredentials: false,
      ...options
    };
  }

  connect(): void {
    // Add lastEventId to URL if available (for custom handling)
    let url = this.options.url;
    if (this.lastEventId) {
      const separator = url.includes('?') ? '&' : '?';
      url += `${separator}lastEventId=${encodeURIComponent(this.lastEventId)}`;
    }

    this.eventSource = new EventSource(url, {
      withCredentials: this.options.withCredentials
    });

    this.eventSource.onopen = () => {
      this.connected = true;
      this.retryCount = 0;
      console.log('SSE connected');
      this.options.onConnect?.();
    };

    this.eventSource.onerror = (error) => {
      if (this.eventSource?.readyState === EventSource.CLOSED) {
        this.connected = false;
        this.options.onDisconnect?.();
        
        // Browser won't reconnect, handle manually if needed
        if (this.retryCount < (this.options.maxRetries || Infinity)) {
          this.retryCount++;
          setTimeout(() => this.connect(), 3000 * this.retryCount);
        }
      } else {
        // CONNECTING state - browser is handling reconnection
        console.log('SSE reconnecting...');
      }
      this.options.onError?.(error);
    };

    // Default message handler
    this.eventSource.onmessage = (event) => {
      this.handleEvent('message', event);
    };

    // Re-register named event listeners
    this.handlers.forEach((_, eventName) => {
      if (eventName !== 'message') {
        this.eventSource!.addEventListener(eventName, (event: Event) => {
          this.handleEvent(eventName, event as MessageEvent);
        });
      }
    });
  }

  private handleEvent(eventName: string, event: MessageEvent) {
    // Track last event ID
    if (event.lastEventId) {
      this.lastEventId = event.lastEventId;
    }

    const handlers = this.handlers.get(eventName);
    if (!handlers) return;

    let data = event.data;
    try {
      data = JSON.parse(event.data);
    } catch {
      // Keep as string if not JSON
    }

    handlers.forEach(handler => {
      try {
        handler(data);
      } catch (error) {
        console.error(`Error in SSE handler for ${eventName}:`, error);
      }
    });
  }

  on(eventName: string, handler: EventHandler): () => void {
    if (!this.handlers.has(eventName)) {
      this.handlers.set(eventName, new Set());

      // Add listener to existing connection
      if (this.eventSource && eventName !== 'message') {
        this.eventSource.addEventListener(eventName, (event: Event) => {
          this.handleEvent(eventName, event as MessageEvent);
        });
      }
    }

    this.handlers.get(eventName)!.add(handler);

    // Return unsubscribe function
    return () => this.off(eventName, handler);
  }

  off(eventName: string, handler: EventHandler): void {
    this.handlers.get(eventName)?.delete(handler);
  }

  close(): void {
    this.eventSource?.close();
    this.eventSource = null;
    this.connected = false;
  }

  get isConnected(): boolean {
    return this.connected;
  }

  get readyState(): number {
    return this.eventSource?.readyState ?? EventSource.CLOSED;
  }
}

// ════════════════════════════════════════════════════════════════
// USAGE
// ════════════════════════════════════════════════════════════════

const sse = new SSEClient({
  url: '/api/events',
  withCredentials: true,
  onConnect: () => console.log('Connected!'),
  onDisconnect: () => console.log('Disconnected!')
});

// Subscribe to events
const unsubscribe = sse.on('notification', (data) => {
  console.log('Notification:', data);
});

sse.on('user-status', (data) => {
  console.log('User status:', data);
});

// Connect
sse.connect();

// Later: unsubscribe from specific event
unsubscribe();

// Or close entirely
sse.close();
```

### React Hook for SSE

```typescript
// ════════════════════════════════════════════════════════════════
// REACT HOOK FOR SSE
// ════════════════════════════════════════════════════════════════

import { useEffect, useRef, useState, useCallback } from 'react';

interface UseSSEOptions {
  url: string;
  withCredentials?: boolean;
  onOpen?: () => void;
  onError?: (error: Event) => void;
}

interface UseSSEReturn<T> {
  data: T | null;
  error: Event | null;
  readyState: number;
  close: () => void;
}

function useSSE<T = any>(
  eventName: string,
  options: UseSSEOptions
): UseSSEReturn<T> {
  const [data, setData] = useState<T | null>(null);
  const [error, setError] = useState<Event | null>(null);
  const [readyState, setReadyState] = useState(EventSource.CONNECTING);
  const eventSourceRef = useRef<EventSource | null>(null);

  const close = useCallback(() => {
    eventSourceRef.current?.close();
    eventSourceRef.current = null;
    setReadyState(EventSource.CLOSED);
  }, []);

  useEffect(() => {
    const eventSource = new EventSource(options.url, {
      withCredentials: options.withCredentials
    });

    eventSourceRef.current = eventSource;

    eventSource.onopen = () => {
      setReadyState(EventSource.OPEN);
      setError(null);
      options.onOpen?.();
    };

    eventSource.onerror = (err) => {
      setError(err);
      setReadyState(eventSource.readyState);
      options.onError?.(err);
    };

    const handler = (event: MessageEvent) => {
      try {
        const parsed = JSON.parse(event.data);
        setData(parsed);
      } catch {
        setData(event.data as T);
      }
    };

    if (eventName === 'message') {
      eventSource.onmessage = handler;
    } else {
      eventSource.addEventListener(eventName, handler);
    }

    return () => {
      eventSource.close();
    };
  }, [options.url, eventName, options.withCredentials]);

  return { data, error, readyState, close };
}

// ════════════════════════════════════════════════════════════════
// USAGE IN COMPONENT
// ════════════════════════════════════════════════════════════════

function NotificationFeed() {
  const { data: notification, readyState } = useSSE<Notification>(
    'notification',
    {
      url: '/api/notifications/stream',
      onOpen: () => console.log('Notification stream connected'),
      onError: (err) => console.error('SSE error:', err)
    }
  );

  const [notifications, setNotifications] = useState<Notification[]>([]);

  useEffect(() => {
    if (notification) {
      setNotifications(prev => [notification, ...prev].slice(0, 50));
    }
  }, [notification]);

  if (readyState === EventSource.CONNECTING) {
    return <div>Connecting...</div>;
  }

  return (
    <div>
      <div className="status">
        {readyState === EventSource.OPEN ? '🟢 Connected' : '🔴 Disconnected'}
      </div>
      <ul>
        {notifications.map((n, i) => (
          <li key={i}>{n.message}</li>
        ))}
      </ul>
    </div>
  );
}

// ════════════════════════════════════════════════════════════════
// HOOK FOR MULTIPLE EVENT TYPES
// ════════════════════════════════════════════════════════════════

function useSSEMultiple(url: string, events: string[]) {
  const [messages, setMessages] = useState<Map<string, any>>(new Map());
  const eventSourceRef = useRef<EventSource | null>(null);

  useEffect(() => {
    const eventSource = new EventSource(url);
    eventSourceRef.current = eventSource;

    events.forEach(eventName => {
      const handler = (event: MessageEvent) => {
        try {
          const data = JSON.parse(event.data);
          setMessages(prev => new Map(prev).set(eventName, data));
        } catch {
          setMessages(prev => new Map(prev).set(eventName, event.data));
        }
      };

      if (eventName === 'message') {
        eventSource.onmessage = handler;
      } else {
        eventSource.addEventListener(eventName, handler);
      }
    });

    return () => eventSource.close();
  }, [url, events.join(',')]);

  return messages;
}

// Usage
function Dashboard() {
  const data = useSSEMultiple('/api/stream', [
    'metrics',
    'alerts',
    'user-count'
  ]);

  return (
    <div>
      <MetricsPanel data={data.get('metrics')} />
      <AlertsPanel data={data.get('alerts')} />
      <UserCount count={data.get('user-count')} />
    </div>
  );
}
```

---

## 2. Server Implementation

### Node.js/Express

```typescript
// ════════════════════════════════════════════════════════════════
// EXPRESS SSE SERVER
// ════════════════════════════════════════════════════════════════

import express from 'express';

const app = express();

// Store connected clients
interface SSEClient {
  id: string visio
  userId: string;
  response: express.Response;
  lastEventId: number;
}

const clients: Map<string, SSEClient> = new Map();
let eventIdCounter = 0;

// ════════════════════════════════════════════════════════════════
// SSE ENDPOINT
// ════════════════════════════════════════════════════════════════

app.get('/api/events', (req, res) => {
  // Set SSE headers
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');
  res.setHeader('X-Accel-Buffering', 'no'); // Disable Nginx buffering
  
  // CORS headers if needed
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Credentials', 'true');

  // Flush headers
  res.flushHeaders();

  // Generate client ID
  const clientId = `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  
  // Get user from auth (implement your auth)
  const userId = (req as any).user?.id || 'anonymous';
  
  // Get last event ID for resumption
  const lastEventId = parseInt(req.headers['last-event-id'] as string) || 0;

  // Store client
  const client: SSEClient = {
    id: clientId,
    userId,
    response: res,
    lastEventId
  };
  clients.set(clientId, client);

  console.log(`Client connected: ${clientId} (Last-Event-ID: ${lastEventId})`);

  // Send any missed events
  if (lastEventId > 0) {
    sendMissedEvents(client, lastEventId);
  }

  // Send initial connection event
  sendEvent(res, {
    event: 'connected',
    data: { clientId, timestamp: new Date().toISOString() }
  });

  // Set retry interval (ms) - browser will wait this long before reconnecting
  res.write('retry: 3000\n\n');

  // Keep-alive with periodic comments
  const keepAliveInterval = setInterval(() => {
    res.write(': ping\n\n');
  }, 30000);

  // Clean up on disconnect
  req.on('close', () => {
    clearInterval(keepAliveInterval);
    clients.delete(clientId);
    console.log(`Client disconnected: ${clientId}`);
  });
});

// ════════════════════════════════════════════════════════════════
// SEND EVENT HELPER
// ════════════════════════════════════════════════════════════════

interface SSEEvent {
  id?: number;
  event?: string;
  data: any;
  retry?: number;
}

function sendEvent(res: express.Response, event: SSEEvent) {
  // Assign event ID if not provided
  const eventId = event.id ?? ++eventIdCounter;

  let message = '';
  
  // Event ID (for reconnection)
  message += `id: ${eventId}\n`;
  
  // Event type (optional)
  if (event.event) {
    message += `event: ${event.event}\n`;
  }
  
  // Retry interval (optional)
  if (event.retry) {
    message += `retry: ${event.retry}\n`;
  }
  
  // Data (required) - handle multi-line
  const data = typeof event.data === 'string' 
    ? event.data 
    : JSON.stringify(event.data);
  
  data.split('\n').forEach(line => {
    message += `data: ${line}\n`;
  });
  
  // End of event
  message += '\n';

  res.write(message);
}

// ════════════════════════════════════════════════════════════════
// BROADCAST TO ALL CLIENTS
// ════════════════════════════════════════════════════════════════

function broadcast(event: Omit<SSEEvent, 'id'>) {
  const eventId = ++eventIdCounter;
  
  // Store event for recovery
  storeEvent(eventId, event);
  
  clients.forEach(client => {
    sendEvent(client.response, { ...event, id: eventId });
  });
}

// Send to specific user
function sendToUser(userId: string, event: Omit<SSEEvent, 'id'>) {
  const eventId = ++eventIdCounter;
  
  storeEvent(eventId, event, userId);
  
  clients.forEach(client => {
    if (client.userId === userId) {
      sendEvent(client.response, { ...event, id: eventId });
    }
  });
}

// ════════════════════════════════════════════════════════════════
// EVENT STORAGE FOR RECOVERY
// ════════════════════════════════════════════════════════════════

interface StoredEvent {
  id: number;
  event: Omit<SSEEvent, 'id'>;
  userId?: string;
  timestamp: number;
}

const eventStore: StoredEvent[] = [];
const MAX_STORED_EVENTS = 1000;
const EVENT_TTL = 5 * 60 * 1000; // 5 minutes

function storeEvent(id: number, event: Omit<SSEEvent, 'id'>, userId?: string) {
  eventStore.push({
    id,
    event,
    userId,
    timestamp: Date.now()
  });

  // Cleanup old events
  const cutoff = Date.now() - EVENT_TTL;
  while (eventStore.length > 0 && eventStore[0].timestamp < cutoff) {
    eventStore.shift();
  }
  
  // Also limit by count
  while (eventStore.length > MAX_STORED_EVENTS) {
    eventStore.shift();
  }
}

function sendMissedEvents(client: SSEClient, lastEventId: number) {
  const missedEvents = eventStore.filter(stored => 
    stored.id > lastEventId && 
    (!stored.userId || stored.userId === client.userId)
  );

  console.log(`Sending ${missedEvents.length} missed events to ${client.id}`);

  missedEvents.forEach(stored => {
    sendEvent(client.response, { ...stored.event, id: stored.id });
  });
}

// ════════════════════════════════════════════════════════════════
// EXAMPLE: SEND NOTIFICATIONS
// ════════════════════════════════════════════════════════════════

// From anywhere in your application
function notifyUser(userId: string, notification: any) {
  sendToUser(userId, {
    event: 'notification',
    data: notification
  });
}

// Broadcast system alert
function systemAlert(message: string) {
  broadcast({
    event: 'alert',
    data: { message, timestamp: new Date().toISOString() }
  });
}

// ════════════════════════════════════════════════════════════════
// STATS ENDPOINT
// ════════════════════════════════════════════════════════════════

app.get('/api/events/stats', (req, res) => {
  res.json({
    connectedClients: clients.size,
    storedEvents: eventStore.length,
    currentEventId: eventIdCounter
  });
});

app.listen(3000, () => {
  console.log('SSE server running on port 3000');
});
```

### NestJS Implementation

```typescript
// ════════════════════════════════════════════════════════════════
// NESTJS SSE CONTROLLER
// ════════════════════════════════════════════════════════════════

import { Controller, Get, Sse, Req, MessageEvent } from '@nestjs/common';
import { Observable, interval, Subject, merge, filter } from 'rxjs';
import { map } from 'rxjs/operators';
import { Request } from 'express';

@Controller('events')
export class EventsController {
  // Subject for pushing events
  private events$ = new Subject<{
    userId?: string;
    event: string;
    data: any;
  }>();

  @Sse('stream')
  stream(@Req() req: Request): Observable<MessageEvent> {
    const userId = (req as any).user?.id;

    // Filter events for this user (or global events)
    const userEvents$ = this.events$.pipe(
      filter(event => !event.userId || event.userId === userId),
      map(event => ({
        type: event.event,
        data: JSON.stringify(event.data)
      } as MessageEvent))
    );

    // Keep-alive ping every 30 seconds
    const keepAlive$ = interval(30000).pipe(
      map(() => ({ type: 'ping', data: '' } as MessageEvent))
    );

    // Initial connection event
    const connected$ = new Observable<MessageEvent>(subscriber => {
      subscriber.next({
        type: 'connected',
        data: JSON.stringify({ timestamp: new Date().toISOString() })
      });
      subscriber.complete();
    });

    return merge(connected$, userEvents$, keepAlive$);
  }

  // Method to push events (call from services)
  sendEvent(event: string, data: any, userId?: string) {
    this.events$.next({ event, data, userId });
  }

  // Broadcast to all
  broadcast(event: string, data: any) {
    this.events$.next({ event, data });
  }
}

// ════════════════════════════════════════════════════════════════
// SSE SERVICE
// ════════════════════════════════════════════════════════════════

import { Injectable } from '@nestjs/common';
import { Subject, Observable, BehaviorSubject } from 'rxjs';
import { filter, map } from 'rxjs/operators';

interface SSEMessage {
  id?: string;
  type: string;
  data: any;
  userId?: string;
}

@Injectable()
export class SSEService {
  private messageSubject = new Subject<SSEMessage>();
  private connectionCount = new BehaviorSubject<number>(0);

  // Get observable for specific user
  getUserStream(userId: string): Observable<MessageEvent> {
    return this.messageSubject.pipe(
      filter(msg => !msg.userId || msg.userId === userId),
      map(msg => ({
        id: msg.id,
        type: msg.type,
        data: JSON.stringify(msg.data)
      } as MessageEvent))
    );
  }

  // Push event
  emit(type: string, data: any, userId?: string) {
    this.messageSubject.next({
      id: `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
      type,
      data,
      userId
    });
  }

  // Track connections
  clientConnected() {
    this.connectionCount.next(this.connectionCount.value + 1);
  }

  clientDisconnected() {
    this.connectionCount.next(this.connectionCount.value - 1);
  }

  getConnectionCount(): number {
    return this.connectionCount.value;
  }
}
```

---

## 3. Authentication & Security

```typescript
// ════════════════════════════════════════════════════════════════
// SSE AUTHENTICATION
// ════════════════════════════════════════════════════════════════

// Problem: EventSource API doesn't support custom headers
// Solutions:

// SOLUTION 1: Cookie-based authentication (Recommended)
// ───────────────────────────────────────────────────────────────

// Server: Standard session/JWT cookie
app.get('/api/events', authenticate, (req, res) => {
  const userId = req.user.id;  // From auth middleware
  // ... SSE logic
});

// Client: Cookies sent automatically
const eventSource = new EventSource('/api/events', {
  withCredentials: true  // Include cookies for CORS
});

// SOLUTION 2: Token in URL query string
// ───────────────────────────────────────────────────────────────

// Client
const token = localStorage.getItem('token');
const eventSource = new EventSource(`/api/events?token=${token}`);

// Server
app.get('/api/events', (req, res) => {
  const token = req.query.token as string;
  
  try {
    const user = verifyJWT(token);
    // ... proceed with SSE
  } catch {
    res.status(401).end();
  }
});

// Note: Token in URL is logged in server access logs - use short-lived tokens

// SOLUTION 3: Two-step authentication
// ───────────────────────────────────────────────────────────────

// Step 1: Get one-time ticket via authenticated endpoint
app.post('/api/events/ticket', authenticate, async (req, res) => {
  const ticket = crypto.randomBytes(32).toString('hex');
  
  // Store ticket with user info (Redis, short TTL)
  await redis.set(`sse-ticket:${ticket}`, req.user.id, 'EX', 30);
  
  res.json({ ticket });
});

// Step 2: Use ticket to connect SSE
app.get('/api/events', async (req, res) => {
  const ticket = req.query.ticket as string;
  
  const userId = await redis.get(`sse-ticket:${ticket}`);
  if (!userId) {
    return res.status(401).end();
  }
  
  // Delete ticket (one-time use)
  await redis.del(`sse-ticket:${ticket}`);
  
  // ... proceed with SSE
});

// Client
async function connectSSE() {
  // Get one-time ticket
  const { ticket } = await fetch('/api/events/ticket', {
    method: 'POST',
    headers: { 'Authorization': `Bearer ${token}` }
  }).then(r => r.json());
  
  // Connect with ticket
  return new EventSource(`/api/events?ticket=${ticket}`);
}

// ════════════════════════════════════════════════════════════════
// RATE LIMITING SSE
// ════════════════════════════════════════════════════════════════

import rateLimit from 'express-rate-limit';

// Limit SSE connection attempts
const sseRateLimiter = rateLimit({
  windowMs: 60 * 1000,  // 1 minute
  max: 10,               // 10 connections per minute
  message: 'Too many connection attempts'
});

app.get('/api/events', sseRateLimiter, authenticate, sseHandler);

// ════════════════════════════════════════════════════════════════
// CONNECTION LIMITS
// ════════════════════════════════════════════════════════════════

const MAX_CONNECTIONS_PER_USER = 3;

app.get('/api/events', authenticate, (req, res) => {
  const userId = req.user.id;
  
  // Count existing connections for user
  let userConnections = 0;
  clients.forEach(client => {
    if (client.userId === userId) userConnections++;
  });
  
  if (userConnections >= MAX_CONNECTIONS_PER_USER) {
    return res.status(429).json({ error: 'Too many connections' });
  }
  
  // ... proceed with SSE
});

// ════════════════════════════════════════════════════════════════
// INPUT VALIDATION
// ════════════════════════════════════════════════════════════════

// Validate channel/room subscriptions
app.get('/api/events/:channel', authenticate, async (req, res) => {
  const { channel } = req.params;
  const userId = req.user.id;
  
  // Validate channel access
  const hasAccess = await checkChannelAccess(userId, channel);
  if (!hasAccess) {
    return res.status(403).json({ error: 'Access denied' });
  }
  
  // ... proceed with SSE for this channel
});
```

---

## 4. Reconnection & Recovery

```typescript
// ════════════════════════════════════════════════════════════════
// ROBUST RECONNECTION WITH RECOVERY
// ════════════════════════════════════════════════════════════════

// Server-side: Redis-based event store for recovery
import Redis from 'ioredis';

const redis = new Redis();

class SSEEventStore {
  private readonly keyPrefix = 'sse:events';
  private readonly ttlSeconds = 5 * 60; // 5 minutes
  private eventId = 0;

  constructor(private redis: Redis) {
    this.initEventId();
  }

  private async initEventId() {
    const lastId = await this.redis.get(`${this.keyPrefix}:counter`);
    this.eventId = lastId ? parseInt(lastId) : 0;
  }

  async storeEvent(event: any, channel?: string): Promise<number> {
    const id = ++this.eventId;
    await this.redis.set(`${this.keyPrefix}:counter`, id);

    const key = channel 
      ? `${this.keyPrefix}:${channel}` 
      : `${this.keyPrefix}:global`;

    await this.redis.zadd(key, Date.now(), JSON.stringify({
      id,
      ...event,
      timestamp: Date.now()
    }));

    // Remove old events
    const cutoff = Date.now() - (this.ttlSeconds * 1000);
    await this.redis.zremrangebyscore(key, '-inf', cutoff);

    return id;
  }

  async getEventsSince(lastEventId: number, channel?: string): Promise<any[]> {
    const key = channel 
      ? `${this.keyPrefix}:${channel}` 
      : `${this.keyPrefix}:global`;

    const events = await this.redis.zrange(key, 0, -1);
    
    return events
      .map(e => JSON.parse(e))
      .filter(e => e.id > lastEventId)
      .sort((a, b) => a.id - b.id);
  }

  async getLatestEventId(): Promise<number> {
    return this.eventId;
  }
}

// Usage in SSE handler
const eventStore = new SSEEventStore(redis);

app.get('/api/events', authenticate, async (req, res) => {
  // Set SSE headers...
  
  const lastEventId = parseInt(req.headers['last-event-id'] as string) || 0;
  
  // Send missed events
  if (lastEventId > 0) {
    const missedEvents = await eventStore.getEventsSince(lastEventId);
    
    for (const event of missedEvents) {
      sendEvent(res, {
        id: event.id,
        event: event.event,
        data: event.data
      });
    }
    
    console.log(`Sent ${missedEvents.length} missed events to client`);
  }
  
  // Continue with normal SSE streaming...
});

// Store events when broadcasting
async function broadcast(event: string, data: any) {
  const eventId = await eventStore.storeEvent({ event, data });
  
  clients.forEach(client => {
    sendEvent(client.response, { id: eventId, event, data });
  });
}

// ════════════════════════════════════════════════════════════════
// CLIENT-SIDE: CUSTOM RECONNECTION
// ════════════════════════════════════════════════════════════════

class RobustSSEClient {
  private eventSource: EventSource | null = null;
  private lastEventId: string = '';
  private reconnectAttempts = 0;
  private maxReconnectAttempts = 20;
  private baseDelay = 1000;
  private maxDelay = 30000;
  private reconnectTimer: number | null = null;

  constructor(
    private url: string,
    private onMessage: (event: MessageEvent) => void,
    private onStatusChange: (status: 'connecting' | 'connected' | 'disconnected') => void
  ) {}

  connect() {
    if (this.eventSource) {
      this.eventSource.close();
    }

    this.onStatusChange('connecting');

    // Add last event ID to URL for recovery
    let connectUrl = this.url;
    if (this.lastEventId) {
      const separator = this.url.includes('?') ? '&' : '?';
      connectUrl += `${separator}_lastEventId=${this.lastEventId}`;
    }

    this.eventSource = new EventSource(connectUrl);

    this.eventSource.onopen = () => {
      this.reconnectAttempts = 0;
      this.onStatusChange('connected');
    };

    this.eventSource.onmessage = (event) => {
      // Track last event ID
      if (event.lastEventId) {
        this.lastEventId = event.lastEventId;
      }
      this.onMessage(event);
    };

    this.eventSource.onerror = () => {
      if (this.eventSource?.readyState === EventSource.CLOSED) {
        // Connection closed, won't auto-reconnect
        this.handleDisconnect();
      }
      // If CONNECTING, browser is handling reconnection
    };
  }

  private handleDisconnect() {
    this.onStatusChange('disconnected');

    if (this.reconnectAttempts >= this.maxReconnectAttempts) {
      console.error('Max reconnection attempts reached');
      return;
    }

    // Exponential backoff with jitter
    const delay = Math.min(
      this.baseDelay * Math.pow(2, this.reconnectAttempts) + Math.random() * 1000,
      this.maxDelay
    );

    this.reconnectAttempts++;
    console.log(`Reconnecting in ${delay}ms (attempt ${this.reconnectAttempts})`);

    this.reconnectTimer = window.setTimeout(() => {
      this.connect();
    }, delay);
  }

  close() {
    if (this.reconnectTimer) {
      clearTimeout(this.reconnectTimer);
    }
    this.eventSource?.close();
    this.eventSource = null;
  }
}
```

---

## 5. Scaling SSE

```typescript
// ════════════════════════════════════════════════════════════════
// SCALING SSE WITH REDIS PUB/SUB
// ════════════════════════════════════════════════════════════════

import Redis from 'ioredis';

const publisher = new Redis();
const subscriber = new Redis();

// Subscribe to channels
subscriber.subscribe('sse:broadcast', 'sse:user');

// Handle incoming messages from other servers
subscriber.on('message', (channel, message) => {
  const { event, data, userId } = JSON.parse(message);

  if (channel === 'sse:broadcast') {
    // Send to all local clients
    clients.forEach(client => {
      sendEvent(client.response, { event, data });
    });
  } else if (channel === 'sse:user' && userId) {
    // Send to specific user's local connections
    clients.forEach(client => {
      if (client.userId === userId) {
        sendEvent(client.response, { event, data });
      }
    });
  }
});

// Publish events (called from anywhere)
function broadcastToAllServers(event: string, data: any) {
  publisher.publish('sse:broadcast', JSON.stringify({ event, data }));
}

function sendToUserAllServers(userId: string, event: string, data: any) {
  publisher.publish('sse:user', JSON.stringify({ event, data, userId }));
}

// ════════════════════════════════════════════════════════════════
// LOAD BALANCER CONFIGURATION (Nginx)
// ════════════════════════════════════════════════════════════════

/*
# Nginx config for SSE

upstream sse_servers {
    # Use IP hash for basic stickiness
    # (not strictly required for SSE, but helps with event ordering)
    ip_hash;
    
    server sse1.internal:3000;
    server sse2.internal:3000;
    server sse3.internal:3000;
}

server {
    listen 443 ssl http2;  # HTTP/2 important for SSE!
    server_name api.example.com;

    location /api/events {
        proxy_pass http://sse_servers;
        proxy_http_version 1.1;
        
        # SSE-specific settings
        proxy_set_header Connection '';
        proxy_buffering off;
        proxy_cache off;
        
        # No timeout (or very long timeout)
        proxy_read_timeout 86400s;
        proxy_send_timeout 86400s;
        
        # Chunked transfer
        chunked_transfer_encoding on;
        
        # Disable request body buffering
        proxy_request_buffering off;
        
        # Forward client IP
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
        # Disable gzip for event-stream
        gzip off;
    }
}
*/

// ════════════════════════════════════════════════════════════════
// KUBERNETES DEPLOYMENT
// ════════════════════════════════════════════════════════════════

/*
# Kubernetes annotations for SSE
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: sse-ingress
  annotations:
    # Increase timeouts
    nginx.ingress.kubernetes.io/proxy-read-timeout: "86400"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "86400"
    # Disable buffering
    nginx.ingress.kubernetes.io/proxy-buffering: "off"
    # Connection header
    nginx.ingress.kubernetes.io/configuration-snippet: |
      proxy_set_header Connection '';
*/

// ════════════════════════════════════════════════════════════════
// MONITORING SSE CONNECTIONS
// ════════════════════════════════════════════════════════════════

import { collectDefaultMetrics, Gauge, Counter, Registry } from 'prom-client';

const register = new Registry();
collectDefaultMetrics({ register });

const sseConnections = new Gauge({
  name: 'sse_connections_total',
  help: 'Total SSE connections',
  registers: [register]
});

const sseEventsTotal = new Counter({
  name: 'sse_events_total',
  help: 'Total SSE events sent',
  labelNames: ['event_type'],
  registers: [register]
});

// Update metrics
function trackConnection() {
  sseConnections.inc();
}

function trackDisconnection() {
  sseConnections.dec();
}

function trackEvent(eventType: string) {
  sseEventsTotal.labels(eventType).inc();
}

// Metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.send(await register.metrics());
});
```

---

## 6. Performance Optimization

```typescript
// ════════════════════════════════════════════════════════════════
// SSE PERFORMANCE OPTIMIZATION
// ════════════════════════════════════════════════════════════════

// 1. BATCH EVENTS
// ───────────────────────────────────────────────────────────────

class EventBatcher {
  private queue: Map<string, any[]> = new Map(); // clientId -> events
  private flushInterval: NodeJS.Timeout;

  constructor(
    private clients: Map<string, SSEClient>,
    private flushMs = 100
  ) {
    this.flushInterval = setInterval(() => this.flush(), this.flushMs);
  }

  add(clientId: string, event: any) {
    if (!this.queue.has(clientId)) {
      this.queue.set(clientId, []);
    }
    this.queue.get(clientId)!.push(event);
  }

  private flush() {
    this.queue.forEach((events, clientId) => {
      const client = this.clients.get(clientId);
      if (!client || events.length === 0) return;

      // Send batch as single event
      sendEvent(client.response, {
        event: 'batch',
        data: events
      });
    });
    this.queue.clear();
  }

  stop() {
    clearInterval(this.flushInterval);
  }
}

// 2. COMPRESSION (for large events)
// ───────────────────────────────────────────────────────────────

import zlib from 'zlib';

// Compress large payloads
function compressIfNeeded(data: any): { compressed: boolean; data: string } {
  const json = JSON.stringify(data);
  
  if (json.length < 1024) {
    return { compressed: false, data: json };
  }

  const compressed = zlib.gzipSync(json).toString('base64');
  
  // Only use compression if it actually reduces size
  if (compressed.length < json.length) {
    return { compressed: true, data: compressed };
  }

  return { compressed: false, data: json };
}

// Send with compression info
function sendCompressedEvent(res: express.Response, event: string, payload: any) {
  const { compressed, data } = compressIfNeeded(payload);
  
  sendEvent(res, {
    event,
    data: { compressed, payload: data }
  });
}

// Client-side decompression
function decompressEvent(data: { compressed: boolean; payload: string }): any {
  if (!data.compressed) {
    return JSON.parse(data.payload);
  }

  // Browser decompression
  const binaryString = atob(data.payload);
  const bytes = new Uint8Array(binaryString.length);
  for (let i = 0; i < binaryString.length; i++) {
    bytes[i] = binaryString.charCodeAt(i);
  }
  
  const decompressed = pako.ungzip(bytes, { to: 'string' });
  return JSON.parse(decompressed);
}

// 3. ADAPTIVE KEEP-ALIVE
// ───────────────────────────────────────────────────────────────

class AdaptiveKeepAlive {
  private intervalId: NodeJS.Timeout | null = null;
  private lastActivityTime = Date.now();
  private interval = 15000; // Start at 15s
  private maxInterval = 45000;
  private minInterval = 15000;

  constructor(private res: express.Response) {}

  start() {
    this.intervalId = setInterval(() => {
      this.sendPing();
      this.adjustInterval();
    }, this.interval);
  }

  activity() {
    this.lastActivityTime = Date.now();
  }

  private sendPing() {
    this.res.write(': ping\n\n');
  }

  private adjustInterval() {
    const idleTime = Date.now() - this.lastActivityTime;
    
    if (idleTime > 60000) {
      // Long idle - can slow down pings
      this.interval = Math.min(this.interval + 5000, this.maxInterval);
    } else if (idleTime < 5000) {
      // Active connection - more frequent pings
      this.interval = this.minInterval;
    }
  }

  stop() {
    if (this.intervalId) {
      clearInterval(this.intervalId);
    }
  }
}

// 4. EVENT FILTERING
// ───────────────────────────────────────────────────────────────

// Allow clients to subscribe to specific event types
interface SSESubscription {
  clientId: string;
  userId: string;
  response: express.Response;
  subscribedEvents: Set<string>;
}

const subscriptions: Map<string, SSESubscription> = new Map();

// Client subscribes to specific events
app.post('/api/events/subscribe', authenticate, (req, res) => {
  const { clientId, events } = req.body;
  
  const subscription = subscriptions.get(clientId);
  if (subscription) {
    subscription.subscribedEvents = new Set(events);
  }
  
  res.json({ success: true });
});

// Only send events client cares about
function sendFilteredEvent(event: string, data: any) {
  subscriptions.forEach(subscription => {
    if (subscription.subscribedEvents.size === 0 || 
        subscription.subscribedEvents.has(event)) {
      sendEvent(subscription.response, { event, data });
    }
  });
}

// 5. HTTP/2 UTILIZATION
// ───────────────────────────────────────────────────────────────

import http2 from 'http2';
import fs from 'fs';

const server = http2.createSecureServer({
  key: fs.readFileSync('server.key'),
  cert: fs.readFileSync('server.crt')
});

server.on('stream', (stream, headers) => {
  if (headers[':path'] === '/api/events') {
    // SSE over HTTP/2
    stream.respond({
      ':status': 200,
      'content-type': 'text/event-stream',
      'cache-control': 'no-cache'
    });

    // HTTP/2 handles multiplexing - no 6 connection limit!
    
    const keepAlive = setInterval(() => {
      stream.write(': ping\n\n');
    }, 30000);

    stream.on('close', () => {
      clearInterval(keepAlive);
    });
  }
});

server.listen(443);
```

---

## 7. Real-World Patterns

### Live Notifications System

```typescript
// ════════════════════════════════════════════════════════════════
// LIVE NOTIFICATIONS WITH SSE
// ════════════════════════════════════════════════════════════════

interface Notification {
  id: string;
  type: 'info' | 'warning' | 'error' | 'success';
  title: string;
  message: string;
  link?: string;
  read: boolean;
  createdAt: Date;
}

class NotificationService {
  private eventStore: SSEEventStore;
  
  constructor(
    private redis: Redis,
    private clients: Map<string, SSEClient>
  ) {
    this.eventStore = new SSEEventStore(redis);
  }

  async send(userId: string, notification: Omit<Notification, 'id' | 'createdAt' | 'read'>) {
    const fullNotification: Notification = {
      ...notification,
      id: `notif-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
      read: false,
      createdAt: new Date()
    };

    // Save to database
    await this.saveNotification(userId, fullNotification);

    // Store for SSE recovery
    const eventId = await this.eventStore.storeEvent(
      { event: 'notification', data: fullNotification },
      `user:${userId}`
    );

    // Send to all user's connections
    this.clients.forEach(client => {
      if (client.userId === userId) {
        sendEvent(client.response, {
          id: eventId,
          event: 'notification',
          data: fullNotification
        });
      }
    });

    // Also send unread count update
    const unreadCount = await this.getUnreadCount(userId);
    this.sendUnreadCount(userId, unreadCount);

    return fullNotification;
  }

  async markAsRead(userId: string, notificationId: string) {
    await this.updateNotification(userId, notificationId, { read: true });
    
    const unreadCount = await this.getUnreadCount(userId);
    this.sendUnreadCount(userId, unreadCount);
  }

  async markAllAsRead(userId: string) {
    await this.markAllRead(userId);
    this.sendUnreadCount(userId, 0);
  }

  private sendUnreadCount(userId: string, count: number) {
    this.clients.forEach(client => {
      if (client.userId === userId) {
        sendEvent(client.response, {
          event: 'notification-count',
          data: { count }
        });
      }
    });
  }

  // Database operations (implement these)
  private async saveNotification(userId: string, notification: Notification): Promise<void> {}
  private async updateNotification(userId: string, id: string, update: Partial<Notification>): Promise<void> {}
  private async getUnreadCount(userId: string): Promise<number> { return 0; }
  private async markAllRead(userId: string): Promise<void> {}
}

// ════════════════════════════════════════════════════════════════
// CLIENT COMPONENT (React)
// ════════════════════════════════════════════════════════════════

function useNotifications() {
  const [notifications, setNotifications] = useState<Notification[]>([]);
  const [unreadCount, setUnreadCount] = useState(0);

  useEffect(() => {
    const sse = new EventSource('/api/notifications/stream', {
      withCredentials: true
    });

    sse.addEventListener('notification', (event) => {
      const notification = JSON.parse(event.data);
      setNotifications(prev => [notification, ...prev]);
    });

    sse.addEventListener('notification-count', (event) => {
      const { count } = JSON.parse(event.data);
      setUnreadCount(count);
    });

    return () => sse.close();
  }, []);

  const markAsRead = async (id: string) => {
    await fetch(`/api/notifications/${id}/read`, { method: 'POST' });
    setNotifications(prev =>
      prev.map(n => n.id === id ? { ...n, read: true } : n)
    );
  };

  return { notifications, unreadCount, markAsRead };
}
```

### Real-Time Dashboard

```typescript
// ════════════════════════════════════════════════════════════════
// REAL-TIME DASHBOARD WITH SSE
// ════════════════════════════════════════════════════════════════

interface DashboardMetrics {
  activeUsers: number;
  requestsPerSecond: number;
  errorRate: number;
  avgResponseTime: number;
  cpuUsage: number;
  memoryUsage: number;
}

// Server: Metrics aggregator
class MetricsAggregator {
  private metrics: DashboardMetrics = {
    activeUsers: 0,
    requestsPerSecond: 0,
    errorRate: 0,
    avgResponseTime: 0,
    cpuUsage: 0,
    memoryUsage: 0
  };

  constructor(private redis: Redis) {
    this.startCollection();
  }

  private startCollection() {
    // Collect metrics every second
    setInterval(async () => {
      this.metrics = {
        activeUsers: await this.getActiveUsers(),
        requestsPerSecond: await this.getRPS(),
        errorRate: await this.getErrorRate(),
        avgResponseTime: await this.getAvgResponseTime(),
        cpuUsage: await this.getCPUUsage(),
        memoryUsage: await this.getMemoryUsage()
      };
    }, 1000);
  }

  getMetrics(): DashboardMetrics {
    return { ...this.metrics };
  }

  // Implement these based on your monitoring setup
  private async getActiveUsers(): Promise<number> { return 0; }
  private async getRPS(): Promise<number> { return 0; }
  private async getErrorRate(): Promise<number> { return 0; }
  private async getAvgResponseTime(): Promise<number> { return 0; }
  private async getCPUUsage(): Promise<number> { return 0; }
  private async getMemoryUsage(): Promise<number> { return 0; }
}

// SSE endpoint for dashboard
app.get('/api/dashboard/stream', authenticate, requireRole('admin'), (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');
  res.flushHeaders();

  const metricsAggregator = new MetricsAggregator(redis);

  // Send metrics every second
  const metricsInterval = setInterval(() => {
    sendEvent(res, {
      event: 'metrics',
      data: metricsAggregator.getMetrics()
    });
  }, 1000);

  // Also send alerts
  const alertHandler = (alert: any) => {
    sendEvent(res, { event: 'alert', data: alert });
  };
  alertEmitter.on('alert', alertHandler);

  req.on('close', () => {
    clearInterval(metricsInterval);
    alertEmitter.off('alert', alertHandler);
  });
});

// React Dashboard Component
function Dashboard() {
  const [metrics, setMetrics] = useState<DashboardMetrics | null>(null);
  const [alerts, setAlerts] = useState<Alert[]>([]);

  useEffect(() => {
    const sse = new EventSource('/api/dashboard/stream', {
      withCredentials: true
    });

    sse.addEventListener('metrics', (event) => {
      setMetrics(JSON.parse(event.data));
    });

    sse.addEventListener('alert', (event) => {
      const alert = JSON.parse(event.data);
      setAlerts(prev => [alert, ...prev].slice(0, 10));
    });

    return () => sse.close();
  }, []);

  if (!metrics) return <Loading />;

  return (
    <div className="dashboard">
      <MetricCard title="Active Users" value={metrics.activeUsers} />
      <MetricCard title="RPS" value={metrics.requestsPerSecond} />
      <MetricCard title="Error Rate" value={`${metrics.errorRate}%`} />
      <MetricCard title="Avg Response" value={`${metrics.avgResponseTime}ms`} />
      <MetricCard title="CPU" value={`${metrics.cpuUsage}%`} />
      <MetricCard title="Memory" value={`${metrics.memoryUsage}%`} />
      <AlertsList alerts={alerts} />
    </div>
  );
}
```

---

## 8. SSE vs WebSocket Decision Guide

```
┌─────────────────────────────────────────────────────────────────┐
│              WHEN TO USE SSE vs WEBSOCKET                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  USE SSE WHEN:                                                  │
│  ────────────                                                   │
│  ✅ Server-to-client streaming only                            │
│     • Live feeds, news, stock tickers                          │
│     • Notifications                                             │
│     • Dashboard updates                                         │
│     • Progress indicators                                       │
│                                                                 │
│  ✅ Simplicity is priority                                     │
│     • No complex scaling infrastructure                        │
│     • Team familiar with HTTP                                  │
│     • Quick implementation needed                               │
│                                                                 │
│  ✅ Text-based data                                            │
│     • JSON payloads                                            │
│     • Log streaming                                             │
│     • Event notifications                                       │
│                                                                 │
│  ✅ Reconnection/recovery important                            │
│     • Built-in Last-Event-ID                                   │
│     • Browser handles reconnection                              │
│                                                                 │
│  ✅ Proxy compatibility required                               │
│     • Corporate firewalls                                       │
│     • Standard HTTP infrastructure                              │
│                                                                 │
│  ────────────────────────────────────────────────────────────   │
│                                                                 │
│  USE WEBSOCKET WHEN:                                            │
│  ──────────────────                                             │
│  ✅ Bidirectional real-time                                    │
│     • Chat applications                                         │
│     • Collaborative editing                                     │
│     • Gaming                                                    │
│     • Live auctions                                             │
│                                                                 │
│  ✅ High-frequency messaging                                   │
│     • Many messages per second                                  │
│     • Low latency critical                                      │
│                                                                 │
│  ✅ Binary data                                                 │
│     • File transfers                                            │
│     • Audio/video streaming                                     │
│     • Efficient binary protocols                                │
│                                                                 │
│  ✅ Complex interaction patterns                               │
│     • Request-response over socket                              │
│     • Multiple message types                                    │
│     • Acknowledgments needed                                    │
│                                                                 │
│  ────────────────────────────────────────────────────────────   │
│                                                                 │
│  HYBRID APPROACH:                                               │
│  ───────────────                                                │
│  • Use SSE for server→client updates                           │
│  • Use HTTP POST for client→server messages                    │
│  • Simple, scalable, works everywhere                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 9. Common Pitfalls

```typescript
// ════════════════════════════════════════════════════════════════
// SSE COMMON PITFALLS AND SOLUTIONS
// ════════════════════════════════════════════════════════════════

// ❌ PITFALL 1: Forgetting to set correct headers
// Problem: Client doesn't receive events or connection closes

// Bad
app.get('/events', (req, res) => {
  res.send('data: hello\n\n');  // Wrong!
});

// Good
app.get('/events', (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');
  res.flushHeaders();  // Important!
  
  res.write('data: hello\n\n');
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 2: Not handling HTTP/1.1 connection limit
// Problem: Only 6 connections per domain in HTTP/1.1

// Bad: Opening many SSE connections
const feed1 = new EventSource('/api/feed1');
const feed2 = new EventSource('/api/feed2');
const feed3 = new EventSource('/api/feed3');
// ... blocks other HTTP requests!

// Good: Single multiplexed connection
const eventSource = new EventSource('/api/events');

// Server sends different event types
eventSource.addEventListener('feed1', handleFeed1);
eventSource.addEventListener('feed2', handleFeed2);
eventSource.addEventListener('feed3', handleFeed3);

// Or better: Use HTTP/2!

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 3: Not implementing keep-alive
// Problem: Proxies/load balancers close idle connections

// Bad
app.get('/events', (req, res) => {
  // ... setup
  // Only send events when they happen - connection may timeout
});

// Good
app.get('/events', (req, res) => {
  // ... setup
  
  // Send comment (ignored by client) every 30 seconds
  const keepAlive = setInterval(() => {
    res.write(': keep-alive\n\n');
  }, 30000);
  
  req.on('close', () => clearInterval(keepAlive));
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 4: Not using event IDs for recovery
// Problem: Events lost during reconnection

// Bad
function broadcast(data) {
  res.write(`data: ${JSON.stringify(data)}\n\n`);
}

// Good
let eventId = 0;
function broadcast(data) {
  eventId++;
  res.write(`id: ${eventId}\n`);
  res.write(`data: ${JSON.stringify(data)}\n\n`);
  
  // Store for recovery
  storeEvent(eventId, data);
}

// On reconnection
const lastEventId = req.headers['last-event-id'];
if (lastEventId) {
  sendMissedEvents(res, parseInt(lastEventId));
}

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 5: Memory leaks from unclosed connections
// Problem: Server runs out of memory

// Bad
const clients = [];

app.get('/events', (req, res) => {
  clients.push(res);
  // Never removed!
});

// Good
const clients = new Set();

app.get('/events', (req, res) => {
  clients.add(res);
  
  req.on('close', () => {
    clients.delete(res);  // Clean up!
  });
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 6: Not handling errors on write
// Problem: Writing to closed connections crashes server

// Bad
function broadcast(data) {
  clients.forEach(res => {
    res.write(`data: ${data}\n\n`);  // May throw!
  });
}

// Good
function broadcast(data) {
  clients.forEach(res => {
    try {
      if (!res.writableEnded) {
        res.write(`data: ${data}\n\n`);
      }
    } catch (error) {
      // Connection already closed
      clients.delete(res);
    }
  });
}

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 7: Not disabling Nginx buffering
// Problem: Events are buffered and sent in batches

// Nginx config - Add this:
// proxy_buffering off;
// X-Accel-Buffering: no  (header from app)

// In Express:
res.setHeader('X-Accel-Buffering', 'no');

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 8: Not handling authentication properly
// Problem: EventSource doesn't support custom headers

// Bad: Trying to set Authorization header
// NOT POSSIBLE with EventSource!

// Good: Use cookies or URL tokens
const eventSource = new EventSource('/api/events', {
  withCredentials: true  // Send cookies
});
// Or: /api/events?token=xxx

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 9: Incorrect event format
// Problem: Events not received by client

// Bad - Missing newlines
res.write('data: hello');        // Wrong!
res.write('data: hello\n');      // Wrong!

// Good - Double newline ends event
res.write('data: hello\n\n');    // Correct!

// Multi-line data
res.write('data: line1\n');
res.write('data: line2\n');
res.write('\n');                  // End of event

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 10: Not testing reconnection
// Problem: Reconnection doesn't work as expected

// Test by:
// 1. Killing server while client connected
// 2. Checking Last-Event-ID is sent on reconnect
// 3. Verifying missed events are replayed
// 4. Testing with various network conditions
```

---

## 10. Interview Questions

### Basic Questions

**Q: "What is Server-Sent Events?"**
> "SSE is a W3C standard for server-to-client streaming over HTTP. The server sends events to the browser through a persistent HTTP connection using the `text/event-stream` content type. The browser's EventSource API handles connection management, automatic reconnection, and event parsing. Unlike WebSockets, SSE is one-way (server to client) and uses plain HTTP."

**Q: "What's the difference between SSE and WebSocket?"**
> "SSE is one-way (server→client), uses HTTP, has automatic reconnection with Last-Event-ID, text-only, and 6 connection limit on HTTP/1.1. WebSocket is bidirectional, uses its own protocol (upgrade from HTTP), requires manual reconnection logic, supports binary, and doesn't have the same connection limit. SSE is simpler for server push; WebSocket for bidirectional real-time."

**Q: "What is the Last-Event-ID header?"**
> "When SSE reconnects after disconnection, the browser automatically sends the last received event ID in the `Last-Event-ID` header. The server can use this to resume sending from where the client left off, ensuring no events are missed. This is built into the EventSource API - no custom code needed on client side."

### Intermediate Questions

**Q: "How do you handle the 6 connection limit with SSE?"**
> "The 6 connections per domain limit is HTTP/1.1 browser restriction. Solutions:
1. **Use HTTP/2** - multiplexes all connections over one TCP connection, eliminating the limit
2. **Domain sharding** - Use subdomains like events1.example.com, events2.example.com
3. **Single multiplexed stream** - Use one SSE connection with named events for different data types
4. **Combine with polling** - Only use SSE for critical real-time data

HTTP/2 is the best solution - modern servers and browsers support it, and it solves many SSE scaling issues."

**Q: "How do you implement authentication with SSE?"**
> "EventSource API doesn't support custom headers, so you can't use Authorization headers directly. Options:
1. **Cookies** - Most common. Use HttpOnly session cookies, set `withCredentials: true` on EventSource
2. **Token in URL** - Pass JWT as query parameter, but it appears in logs
3. **Two-step auth** - POST to get one-time ticket, use ticket in SSE URL
4. **Service Worker** - Intercept requests and add headers (complex)

Cookies with HttpOnly is the safest and simplest approach for same-origin requests."

**Q: "How do you scale SSE across multiple servers?"**
> "Use a pub/sub system like Redis. When an event needs to be broadcast:
1. Publish to Redis channel
2. All servers subscribe to this channel
3. Each server sends to its connected clients

For state recovery, store events in Redis sorted set with event ID. On reconnection, query events since Last-Event-ID. Configure load balancer to not buffer SSE responses and set long timeouts."

### Advanced Questions

**Q: "How would you implement a real-time notification system with SSE?"**
> "Architecture:
1. **Single SSE endpoint per user** - `/api/notifications/stream`
2. **Event types** - notification (new), count-update (unread count)
3. **Redis pub/sub** - For multi-server broadcasting
4. **Event store** - Redis sorted set for reconnection recovery
5. **API endpoints** - POST for mark-read, GET for history

On notification, publish to user's Redis channel, all servers receive and send to that user's connections. Track event IDs for seamless reconnection. Use named events to separate notification types."

**Q: "What are the performance considerations for SSE?"**
> "Key considerations:
1. **Keep connections alive** - Send comments (`: ping`) every 30s to prevent proxy timeouts
2. **Event batching** - Batch high-frequency events to reduce write calls
3. **Memory management** - Clean up closed connections immediately
4. **HTTP/2** - Use for multiplexing and better performance
5. **Event storage limits** - Cap stored events for recovery to prevent memory issues
6. **Connection limits** - Track per-user connections to prevent abuse
7. **Compression** - For large payloads, consider gzip or application-level compression

Monitor connection counts, message rates, and server memory usage."

**Q: "How would you debug SSE issues in production?"**
> "Debugging approach:
1. **Client-side** - Chrome DevTools Network tab shows EventSource connections and received events
2. **Server-side** - Log connection/disconnection with client IDs, log event sends
3. **Proxies** - Ensure `X-Accel-Buffering: no` header, check proxy timeout settings
4. **Reconnection** - Log Last-Event-ID values, verify event recovery
5. **Metrics** - Track connection count, reconnection rate, event delivery latency

Common issues: proxy buffering (add headers), timeout disconnects (keep-alive), missing events (check event IDs), connection limit (use HTTP/2)."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                  SSE IMPLEMENTATION CHECKLIST                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SERVER SETUP:                                                  │
│  □ Set Content-Type: text/event-stream                         │
│  □ Set Cache-Control: no-cache                                 │
│  □ Set Connection: keep-alive                                  │
│  □ Set X-Accel-Buffering: no (for Nginx)                       │
│  □ Call res.flushHeaders()                                     │
│                                                                 │
│  EVENT FORMAT:                                                  │
│  □ id: <event-id>\n                                            │
│  □ event: <event-name>\n (optional)                            │
│  □ data: <payload>\n                                           │
│  □ \n (empty line = end of event)                              │
│                                                                 │
│  KEEP-ALIVE:                                                    │
│  □ Send `: ping\n\n` every 30 seconds                          │
│  □ Clean up interval on disconnect                             │
│                                                                 │
│  RECONNECTION:                                                  │
│  □ Include event IDs with every event                          │
│  □ Store recent events for recovery                            │
│  □ Check Last-Event-ID header on connect                       │
│  □ Replay missed events                                        │
│                                                                 │
│  AUTHENTICATION:                                                │
│  □ Use cookies (withCredentials: true)                         │
│  □ Or token in query string (one-time tickets)                 │
│  □ Verify before streaming                                     │
│                                                                 │
│  SCALING:                                                       │
│  □ Use Redis pub/sub for multi-server                          │
│  □ Use HTTP/2 to avoid connection limit                        │
│  □ Configure load balancer for long connections                │
│                                                                 │
│  CLEANUP:                                                       │
│  □ Track all connections                                       │
│  □ Remove on req.close event                                   │
│  □ Handle write errors gracefully                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

EVENT FORMAT QUICK REFERENCE:
┌─────────────────────────────────────────────┐
│ Basic:     data: Hello World\n\n           │
│ With ID:   id: 123\ndata: Hello\n\n        │
│ Named:     event: notify\ndata: {...}\n\n  │
│ Retry:     retry: 5000\n\n                 │
│ Comment:   : keep-alive\n\n                │
└─────────────────────────────────────────────┘

EVENTSOURCE READY STATES:
┌────────────────────────────────────┐
│ CONNECTING = 0  (trying to connect) │
│ OPEN = 1        (connected)         │
│ CLOSED = 2      (connection closed) │
└────────────────────────────────────┘
```

---

*Last updated: February 2026*

