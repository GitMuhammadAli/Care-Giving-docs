# WebSocket Protocol - Complete Guide

> **MUST REMEMBER**: WebSocket provides full-duplex communication over a single TCP connection. Starts with HTTP upgrade handshake, then switches to binary framing protocol. Frame types: text, binary, ping, pong, close. Use ping/pong for keep-alive (detect dead connections). Close handshake is two-way. Opcodes: 0x1=text, 0x2=binary, 0x8=close, 0x9=ping, 0xA=pong.

---

## How to Explain Like a Senior Developer

"WebSocket is a protocol for real-time bidirectional communication. Unlike HTTP's request-response model, WebSocket opens a persistent connection where either side can send messages anytime. It starts as an HTTP request with 'Upgrade: websocket' header, server responds with 101 Switching Protocols, then the connection becomes WebSocket. Data is sent in frames - small packets with opcode (type), length, and payload. Ping/pong frames keep the connection alive and detect dead clients. The close handshake is two-way: one side sends close frame, other acknowledges, then TCP closes. Understanding the framing format helps debug issues and implement efficient messaging."

---

## Core Implementation

### WebSocket Framing Explained

```typescript
// websocket/framing.ts

/**
 * WebSocket Frame Format:
 * 
 *  0                   1                   2                   3
 *  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 * +-+-+-+-+-------+-+-------------+-------------------------------+
 * |F|R|R|R| opcode|M| Payload len |    Extended payload length    |
 * |I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
 * |N|V|V|V|       |S|             |   (if payload len==126/127)   |
 * | |1|2|3|       |K|             |                               |
 * +-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
 * |     Extended payload length continued, if payload len == 127  |
 * + - - - - - - - - - - - - - - - +-------------------------------+
 * |                               |Masking-key, if MASK set to 1  |
 * +-------------------------------+-------------------------------+
 * | Masking-key (continued)       |          Payload Data         |
 * +-------------------------------- - - - - - - - - - - - - - - - +
 * :                     Payload Data continued ...                :
 * +---------------------------------------------------------------+
 */

enum Opcode {
  CONTINUATION = 0x0,
  TEXT = 0x1,
  BINARY = 0x2,
  CLOSE = 0x8,
  PING = 0x9,
  PONG = 0xA,
}

interface WebSocketFrame {
  fin: boolean;        // Final fragment
  rsv1: boolean;       // Reserved (for extensions)
  rsv2: boolean;
  rsv3: boolean;
  opcode: Opcode;
  masked: boolean;     // Client frames must be masked
  payloadLength: number;
  maskingKey?: Buffer;
  payload: Buffer;
}

// Parse a WebSocket frame
function parseFrame(buffer: Buffer): WebSocketFrame | null {
  if (buffer.length < 2) return null;
  
  const firstByte = buffer[0];
  const secondByte = buffer[1];
  
  const fin = (firstByte & 0x80) !== 0;
  const rsv1 = (firstByte & 0x40) !== 0;
  const rsv2 = (firstByte & 0x20) !== 0;
  const rsv3 = (firstByte & 0x10) !== 0;
  const opcode = firstByte & 0x0F;
  
  const masked = (secondByte & 0x80) !== 0;
  let payloadLength = secondByte & 0x7F;
  
  let offset = 2;
  
  // Extended payload length
  if (payloadLength === 126) {
    if (buffer.length < 4) return null;
    payloadLength = buffer.readUInt16BE(2);
    offset = 4;
  } else if (payloadLength === 127) {
    if (buffer.length < 10) return null;
    // Node can't handle 64-bit, use lower 32 bits
    payloadLength = buffer.readUInt32BE(6);
    offset = 10;
  }
  
  // Masking key
  let maskingKey: Buffer | undefined;
  if (masked) {
    if (buffer.length < offset + 4) return null;
    maskingKey = buffer.subarray(offset, offset + 4);
    offset += 4;
  }
  
  // Payload
  if (buffer.length < offset + payloadLength) return null;
  let payload = buffer.subarray(offset, offset + payloadLength);
  
  // Unmask if needed
  if (masked && maskingKey) {
    payload = Buffer.alloc(payloadLength);
    for (let i = 0; i < payloadLength; i++) {
      payload[i] = buffer[offset + i] ^ maskingKey[i % 4];
    }
  }
  
  return {
    fin,
    rsv1,
    rsv2,
    rsv3,
    opcode: opcode as Opcode,
    masked,
    payloadLength,
    maskingKey,
    payload,
  };
}

// Create a WebSocket frame
function createFrame(opcode: Opcode, payload: Buffer, mask: boolean = false): Buffer {
  const fin = true;
  const payloadLength = payload.length;
  
  // Calculate frame size
  let headerSize = 2;
  if (payloadLength > 65535) {
    headerSize += 8;
  } else if (payloadLength > 125) {
    headerSize += 2;
  }
  if (mask) headerSize += 4;
  
  const frame = Buffer.alloc(headerSize + payloadLength);
  let offset = 0;
  
  // First byte: FIN + opcode
  frame[offset++] = (fin ? 0x80 : 0) | opcode;
  
  // Second byte: MASK + length
  let lengthByte = mask ? 0x80 : 0;
  if (payloadLength > 65535) {
    lengthByte |= 127;
    frame[offset++] = lengthByte;
    // 64-bit length (we only use 32-bit)
    frame.writeUInt32BE(0, offset);
    offset += 4;
    frame.writeUInt32BE(payloadLength, offset);
    offset += 4;
  } else if (payloadLength > 125) {
    lengthByte |= 126;
    frame[offset++] = lengthByte;
    frame.writeUInt16BE(payloadLength, offset);
    offset += 2;
  } else {
    lengthByte |= payloadLength;
    frame[offset++] = lengthByte;
  }
  
  // Masking key and payload
  if (mask) {
    const maskingKey = Buffer.alloc(4);
    for (let i = 0; i < 4; i++) maskingKey[i] = Math.floor(Math.random() * 256);
    maskingKey.copy(frame, offset);
    offset += 4;
    
    for (let i = 0; i < payloadLength; i++) {
      frame[offset + i] = payload[i] ^ maskingKey[i % 4];
    }
  } else {
    payload.copy(frame, offset);
  }
  
  return frame;
}

export { Opcode, WebSocketFrame, parseFrame, createFrame };
```

### Raw WebSocket Server Implementation

```typescript
// websocket/raw-server.ts
import * as http from 'http';
import * as crypto from 'crypto';
import { Opcode, parseFrame, createFrame } from './framing';

const WEBSOCKET_GUID = '258EAFA5-E914-47DA-95CA-C5AB0DC85B11';

interface WebSocketConnection {
  socket: any;
  id: string;
  isAlive: boolean;
}

class RawWebSocketServer {
  private httpServer: http.Server;
  private connections: Map<string, WebSocketConnection> = new Map();
  private pingInterval?: NodeJS.Timeout;
  
  constructor(port: number) {
    this.httpServer = http.createServer((req, res) => {
      res.writeHead(426); // Upgrade Required
      res.end('WebSocket connections only');
    });
    
    this.httpServer.on('upgrade', (request, socket, head) => {
      this.handleUpgrade(request, socket, head);
    });
    
    this.httpServer.listen(port, () => {
      console.log(`WebSocket server listening on port ${port}`);
    });
    
    // Ping interval to detect dead connections
    this.pingInterval = setInterval(() => this.pingAll(), 30000);
  }
  
  private handleUpgrade(request: http.IncomingMessage, socket: any, head: Buffer): void {
    // Verify WebSocket request
    const key = request.headers['sec-websocket-key'];
    const version = request.headers['sec-websocket-version'];
    
    if (!key || version !== '13') {
      socket.end('HTTP/1.1 400 Bad Request\r\n\r\n');
      return;
    }
    
    // Calculate accept key
    const acceptKey = crypto
      .createHash('sha1')
      .update(key + WEBSOCKET_GUID)
      .digest('base64');
    
    // Send upgrade response
    const response = [
      'HTTP/1.1 101 Switching Protocols',
      'Upgrade: websocket',
      'Connection: Upgrade',
      `Sec-WebSocket-Accept: ${acceptKey}`,
      '', ''
    ].join('\r\n');
    
    socket.write(response);
    
    // Create connection
    const connectionId = crypto.randomUUID();
    const connection: WebSocketConnection = {
      socket,
      id: connectionId,
      isAlive: true,
    };
    
    this.connections.set(connectionId, connection);
    console.log(`Client connected: ${connectionId}`);
    
    // Handle data
    let buffer = Buffer.alloc(0);
    
    socket.on('data', (data: Buffer) => {
      buffer = Buffer.concat([buffer, data]);
      
      while (buffer.length > 0) {
        const frame = parseFrame(buffer);
        if (!frame) break;
        
        // Calculate frame size for buffer advance
        let frameSize = 2 + frame.payloadLength;
        if (frame.payloadLength > 65535) frameSize += 8;
        else if (frame.payloadLength > 125) frameSize += 2;
        if (frame.masked) frameSize += 4;
        
        buffer = buffer.subarray(frameSize);
        
        this.handleFrame(connection, frame);
      }
    });
    
    socket.on('close', () => {
      this.connections.delete(connectionId);
      console.log(`Client disconnected: ${connectionId}`);
    });
    
    socket.on('error', (err: Error) => {
      console.error(`Socket error: ${err.message}`);
      this.connections.delete(connectionId);
    });
  }
  
  private handleFrame(connection: WebSocketConnection, frame: any): void {
    switch (frame.opcode) {
      case Opcode.TEXT:
        const message = frame.payload.toString('utf8');
        console.log(`Received text: ${message}`);
        this.onMessage(connection, message);
        break;
        
      case Opcode.BINARY:
        console.log(`Received binary: ${frame.payload.length} bytes`);
        break;
        
      case Opcode.PING:
        // Respond with pong
        const pongFrame = createFrame(Opcode.PONG, frame.payload);
        connection.socket.write(pongFrame);
        break;
        
      case Opcode.PONG:
        connection.isAlive = true;
        break;
        
      case Opcode.CLOSE:
        // Close handshake
        const closeFrame = createFrame(Opcode.CLOSE, Buffer.alloc(0));
        connection.socket.write(closeFrame);
        connection.socket.end();
        break;
    }
  }
  
  private onMessage(connection: WebSocketConnection, message: string): void {
    // Echo back
    this.send(connection, `Echo: ${message}`);
  }
  
  send(connection: WebSocketConnection, message: string): void {
    const frame = createFrame(Opcode.TEXT, Buffer.from(message));
    connection.socket.write(frame);
  }
  
  broadcast(message: string): void {
    const frame = createFrame(Opcode.TEXT, Buffer.from(message));
    for (const connection of this.connections.values()) {
      connection.socket.write(frame);
    }
  }
  
  private pingAll(): void {
    for (const [id, connection] of this.connections) {
      if (!connection.isAlive) {
        // Connection dead, terminate
        console.log(`Terminating dead connection: ${id}`);
        connection.socket.destroy();
        this.connections.delete(id);
        continue;
      }
      
      connection.isAlive = false;
      const pingFrame = createFrame(Opcode.PING, Buffer.alloc(0));
      connection.socket.write(pingFrame);
    }
  }
  
  close(): void {
    if (this.pingInterval) {
      clearInterval(this.pingInterval);
    }
    
    // Close all connections gracefully
    for (const connection of this.connections.values()) {
      const closeFrame = createFrame(Opcode.CLOSE, Buffer.alloc(0));
      connection.socket.write(closeFrame);
    }
    
    this.httpServer.close();
  }
}

export { RawWebSocketServer };
```

### WebSocket Close Codes

```typescript
// websocket/close-codes.ts

/**
 * WebSocket Close Status Codes (RFC 6455)
 */

const CloseCodes = {
  // Normal closure
  NORMAL: 1000,
  
  // Endpoint going away (browser tab close, server shutdown)
  GOING_AWAY: 1001,
  
  // Protocol error
  PROTOCOL_ERROR: 1002,
  
  // Unsupported data (e.g., text when only binary supported)
  UNSUPPORTED_DATA: 1003,
  
  // Reserved (should not be used)
  RESERVED: 1004,
  
  // No status code present (internal use only)
  NO_STATUS: 1005,
  
  // Abnormal closure (connection lost without close frame)
  ABNORMAL: 1006,
  
  // Invalid payload data (e.g., non-UTF8 text)
  INVALID_PAYLOAD: 1007,
  
  // Policy violation
  POLICY_VIOLATION: 1008,
  
  // Message too big
  TOO_BIG: 1009,
  
  // Extension required but not negotiated
  EXTENSION_REQUIRED: 1010,
  
  // Unexpected server error
  INTERNAL_ERROR: 1011,
  
  // Service restart
  SERVICE_RESTART: 1012,
  
  // Try again later
  TRY_AGAIN_LATER: 1013,
  
  // Bad gateway
  BAD_GATEWAY: 1014,
  
  // TLS handshake failed (internal use only)
  TLS_FAILED: 1015,
};

// Create close frame with code and reason
function createCloseFrame(code: number, reason: string = ''): Buffer {
  const reasonBuffer = Buffer.from(reason, 'utf8');
  const payload = Buffer.alloc(2 + reasonBuffer.length);
  
  payload.writeUInt16BE(code, 0);
  reasonBuffer.copy(payload, 2);
  
  return createFrame(Opcode.CLOSE, payload);
}

// Parse close frame
function parseCloseFrame(payload: Buffer): { code: number; reason: string } {
  if (payload.length < 2) {
    return { code: CloseCodes.NO_STATUS, reason: '' };
  }
  
  const code = payload.readUInt16BE(0);
  const reason = payload.subarray(2).toString('utf8');
  
  return { code, reason };
}

import { Opcode, createFrame } from './framing';

export { CloseCodes, createCloseFrame, parseCloseFrame };
```

### Ping/Pong Heartbeat Implementation

```typescript
// websocket/heartbeat.ts
import WebSocket from 'ws';

/**
 * Ping/Pong is essential for:
 * 1. Detecting dead connections (client crashed, network issue)
 * 2. Keeping NAT/firewall connections alive
 * 3. Measuring latency
 */

class HeartbeatWebSocketServer {
  private wss: WebSocket.Server;
  private pingInterval: NodeJS.Timeout;
  private clients: Map<WebSocket, { isAlive: boolean; latency: number }> = new Map();
  
  constructor(port: number) {
    this.wss = new WebSocket.Server({ port });
    
    this.wss.on('connection', (ws) => {
      this.clients.set(ws, { isAlive: true, latency: 0 });
      
      ws.on('pong', () => {
        const client = this.clients.get(ws);
        if (client) {
          client.isAlive = true;
          // Calculate latency from ping timestamp
          const pingTime = (ws as any)._lastPingTime;
          if (pingTime) {
            client.latency = Date.now() - pingTime;
          }
        }
      });
      
      ws.on('close', () => {
        this.clients.delete(ws);
      });
    });
    
    // Ping every 30 seconds
    this.pingInterval = setInterval(() => this.checkConnections(), 30000);
  }
  
  private checkConnections(): void {
    for (const [ws, client] of this.clients) {
      if (!client.isAlive) {
        // No pong received since last ping = dead connection
        console.log('Terminating dead connection');
        ws.terminate();
        this.clients.delete(ws);
        continue;
      }
      
      // Mark as not alive until pong received
      client.isAlive = false;
      
      // Store ping time for latency calculation
      (ws as any)._lastPingTime = Date.now();
      
      // Send ping
      ws.ping();
    }
  }
  
  getClientStats(): Array<{ latency: number }> {
    return Array.from(this.clients.values()).map(c => ({
      latency: c.latency,
    }));
  }
  
  close(): void {
    clearInterval(this.pingInterval);
    this.wss.close();
  }
}

// Client-side heartbeat
class HeartbeatWebSocketClient {
  private ws: WebSocket | null = null;
  private pingTimeout?: NodeJS.Timeout;
  private url: string;
  
  constructor(url: string) {
    this.url = url;
  }
  
  connect(): void {
    this.ws = new WebSocket(this.url);
    
    this.ws.on('open', () => {
      console.log('Connected');
      this.resetPingTimeout();
    });
    
    this.ws.on('ping', () => {
      // Server sent ping, pong is sent automatically by ws library
      this.resetPingTimeout();
    });
    
    this.ws.on('close', () => {
      this.clearPingTimeout();
      // Reconnect after delay
      setTimeout(() => this.connect(), 5000);
    });
    
    this.ws.on('error', (err) => {
      console.error('WebSocket error:', err);
    });
  }
  
  private resetPingTimeout(): void {
    this.clearPingTimeout();
    
    // Expect ping within 35 seconds (30s interval + 5s grace)
    this.pingTimeout = setTimeout(() => {
      console.log('Ping timeout, terminating connection');
      this.ws?.terminate();
    }, 35000);
  }
  
  private clearPingTimeout(): void {
    if (this.pingTimeout) {
      clearTimeout(this.pingTimeout);
    }
  }
}

export { HeartbeatWebSocketServer, HeartbeatWebSocketClient };
```

---

## Real-World Scenarios

### Scenario 1: WebSocket Subprotocols

```typescript
// websocket/subprotocols.ts
import WebSocket from 'ws';
import http from 'http';

/**
 * Subprotocols define message format/semantics
 * Common subprotocols: graphql-ws, json-rpc, mqtt, stomp
 */

const server = http.createServer();
const wss = new WebSocket.Server({ noServer: true });

server.on('upgrade', (request, socket, head) => {
  // Get requested subprotocols
  const protocols = request.headers['sec-websocket-protocol']?.split(',').map(p => p.trim()) || [];
  
  // Supported protocols
  const supported = ['graphql-ws', 'json-rpc'];
  const selected = protocols.find(p => supported.includes(p));
  
  if (!selected && protocols.length > 0) {
    socket.write('HTTP/1.1 400 Bad Request\r\n\r\n');
    socket.destroy();
    return;
  }
  
  wss.handleUpgrade(request, socket, head, (ws) => {
    // Store selected protocol
    (ws as any).protocol = selected;
    wss.emit('connection', ws, request);
  });
});

wss.on('connection', (ws) => {
  const protocol = (ws as any).protocol;
  console.log(`Client connected with protocol: ${protocol || 'none'}`);
  
  ws.on('message', (data) => {
    if (protocol === 'json-rpc') {
      handleJsonRpc(ws, data.toString());
    } else if (protocol === 'graphql-ws') {
      handleGraphQL(ws, data.toString());
    } else {
      // Plain text
      ws.send(`Echo: ${data}`);
    }
  });
});

function handleJsonRpc(ws: WebSocket, message: string): void {
  try {
    const request = JSON.parse(message);
    
    // JSON-RPC format: { jsonrpc: "2.0", method: "...", params: [...], id: 1 }
    const response = {
      jsonrpc: '2.0',
      result: `Executed: ${request.method}`,
      id: request.id,
    };
    
    ws.send(JSON.stringify(response));
  } catch (err) {
    ws.send(JSON.stringify({
      jsonrpc: '2.0',
      error: { code: -32700, message: 'Parse error' },
      id: null,
    }));
  }
}

function handleGraphQL(ws: WebSocket, message: string): void {
  // graphql-ws protocol handling
  try {
    const msg = JSON.parse(message);
    
    switch (msg.type) {
      case 'connection_init':
        ws.send(JSON.stringify({ type: 'connection_ack' }));
        break;
      case 'subscribe':
        // Handle subscription
        ws.send(JSON.stringify({
          type: 'next',
          id: msg.id,
          payload: { data: { example: 'data' } },
        }));
        break;
    }
  } catch (err) {
    ws.close(1002, 'Protocol error');
  }
}

server.listen(8080);
```

### Scenario 2: Binary Message Handling

```typescript
// websocket/binary.ts
import WebSocket from 'ws';

/**
 * Binary messages for efficient data transfer
 * Common uses: file upload, audio/video, game state
 */

const wss = new WebSocket.Server({ port: 8080 });

// Message types for binary protocol
enum MessageType {
  TEXT = 1,
  FILE_CHUNK = 2,
  AUDIO = 3,
  GAME_STATE = 4,
}

interface BinaryMessage {
  type: MessageType;
  payload: Buffer;
}

function encodeBinaryMessage(type: MessageType, payload: Buffer): Buffer {
  // Format: [type (1 byte)] [length (4 bytes)] [payload]
  const message = Buffer.alloc(5 + payload.length);
  message.writeUInt8(type, 0);
  message.writeUInt32BE(payload.length, 1);
  payload.copy(message, 5);
  return message;
}

function decodeBinaryMessage(data: Buffer): BinaryMessage {
  const type = data.readUInt8(0) as MessageType;
  const length = data.readUInt32BE(1);
  const payload = data.subarray(5, 5 + length);
  return { type, payload };
}

wss.on('connection', (ws) => {
  ws.binaryType = 'nodebuffer';
  
  ws.on('message', (data: Buffer) => {
    if (typeof data === 'string') {
      // Text message
      console.log('Text:', data);
      return;
    }
    
    // Binary message
    const message = decodeBinaryMessage(data);
    
    switch (message.type) {
      case MessageType.FILE_CHUNK:
        handleFileChunk(ws, message.payload);
        break;
      case MessageType.AUDIO:
        handleAudio(ws, message.payload);
        break;
      case MessageType.GAME_STATE:
        handleGameState(ws, message.payload);
        break;
    }
  });
});

function handleFileChunk(ws: WebSocket, chunk: Buffer): void {
  console.log(`Received file chunk: ${chunk.length} bytes`);
  // Acknowledge receipt
  ws.send(encodeBinaryMessage(MessageType.FILE_CHUNK, Buffer.from('ACK')));
}

function handleAudio(ws: WebSocket, data: Buffer): void {
  // Broadcast to other clients
  wss.clients.forEach(client => {
    if (client !== ws && client.readyState === WebSocket.OPEN) {
      client.send(encodeBinaryMessage(MessageType.AUDIO, data));
    }
  });
}

function handleGameState(ws: WebSocket, data: Buffer): void {
  // Parse game state (e.g., player positions)
  // Binary is more efficient than JSON for frequent updates
  const playerId = data.readUInt16BE(0);
  const x = data.readFloatBE(2);
  const y = data.readFloatBE(6);
  console.log(`Player ${playerId} at (${x}, ${y})`);
}
```

---

## Common Pitfalls

### 1. Not Handling Connection Drops

```typescript
// ❌ BAD: Assuming connection is always alive
ws.send(message); // May fail silently if connection dead

// ✅ GOOD: Check state and handle errors
if (ws.readyState === WebSocket.OPEN) {
  ws.send(message, (err) => {
    if (err) console.error('Send failed:', err);
  });
}

import WebSocket from 'ws';
const message = '';
const ws = new WebSocket('ws://localhost');
```

### 2. Missing Ping/Pong Implementation

```typescript
// ❌ BAD: No heartbeat
const wss = new WebSocket.Server({ port: 8080 });
// Dead connections stay in memory forever!

// ✅ GOOD: Implement ping/pong
const wss = new WebSocket.Server({ port: 8080 });

const interval = setInterval(() => {
  wss.clients.forEach(ws => {
    if ((ws as any).isAlive === false) {
      return ws.terminate();
    }
    (ws as any).isAlive = false;
    ws.ping();
  });
}, 30000);

wss.on('connection', ws => {
  (ws as any).isAlive = true;
  ws.on('pong', () => { (ws as any).isAlive = true; });
});
```

### 3. Not Handling Close Properly

```typescript
// ❌ BAD: Abrupt close
ws.terminate(); // No close handshake!

// ✅ GOOD: Graceful close with code and reason
ws.close(1000, 'Normal closure');

// And handle close event
ws.on('close', (code, reason) => {
  console.log(`Closed: ${code} - ${reason}`);
});
```

---

## Interview Questions

### Q1: How does the WebSocket handshake work?

**A:** Client sends HTTP GET with headers: `Upgrade: websocket`, `Connection: Upgrade`, `Sec-WebSocket-Key: random-base64`, `Sec-WebSocket-Version: 13`. Server responds with 101 Switching Protocols and `Sec-WebSocket-Accept: SHA1(key + magic-guid)`. After this, the connection speaks WebSocket protocol, not HTTP.

### Q2: What are the WebSocket frame opcodes?

**A:** 0x0=continuation (fragmented message), 0x1=text, 0x2=binary, 0x8=close, 0x9=ping, 0xA=pong. 0x3-0x7 and 0xB-0xF are reserved. Ping/pong are for keep-alive. Close initiates the closing handshake.

### Q3: Why is ping/pong important for WebSockets?

**A:** 1) Detect dead connections - if client crashes, server won't know without ping timeout. 2) Keep connections alive through NAT/firewalls that close idle connections. 3) Measure latency by timing ping-to-pong. Without it, you might hold thousands of dead connections.

### Q4: How does WebSocket differ from HTTP/2 Server Push?

**A:** WebSocket is bidirectional - both client and server can initiate messages anytime on a persistent connection. HTTP/2 Server Push is server-initiated only and tied to a client request (pushing related resources). WebSocket is for real-time communication; Server Push is for preloading resources.

---

## Quick Reference Checklist

### Server Setup
- [ ] Handle upgrade request properly
- [ ] Validate Sec-WebSocket-Key
- [ ] Implement ping/pong heartbeat
- [ ] Handle close handshake
- [ ] Track and clean up connections

### Protocol Handling
- [ ] Parse frames correctly
- [ ] Handle fragmented messages
- [ ] Unmask client frames
- [ ] Send proper close codes

### Client Implementation
- [ ] Handle reconnection
- [ ] Respond to pings
- [ ] Detect server ping timeout
- [ ] Close gracefully

### Production
- [ ] Set appropriate timeouts
- [ ] Monitor connection count
- [ ] Handle backpressure
- [ ] Log connection events

---

*Last updated: February 2026*

