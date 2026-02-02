# TCP vs UDP - Complete Guide

> **MUST REMEMBER**: TCP is connection-oriented, reliable, ordered - use for HTTP, database connections, file transfers. UDP is connectionless, fast, no guarantees - use for video streaming, gaming, DNS queries, VoIP. TCP has three-way handshake, flow control, congestion control. UDP has lower latency but you handle packet loss yourself. Modern protocols like QUIC build reliability on UDP.

---

## How to Explain Like a Senior Developer

"TCP and UDP are both transport layer protocols but with opposite tradeoffs. TCP establishes a connection (SYN, SYN-ACK, ACK), guarantees delivery and ordering, handles congestion - but has overhead and latency. UDP just sends packets with no connection, no guarantees, no ordering - but it's fast. Use TCP when you can't lose data: HTTP, databases, file transfers. Use UDP when speed matters more than perfection: video calls (a dropped frame is fine), gaming (old position data is useless), DNS (simple request-response). QUIC is interesting - it's UDP underneath but builds TCP-like reliability on top, getting the best of both."

---

## Core Implementation

### TCP Server and Client

```typescript
// tcp/server.ts
import * as net from 'net';

interface Client {
  socket: net.Socket;
  id: string;
  address: string;
}

class TcpServer {
  private server: net.Server;
  private clients: Map<string, Client> = new Map();
  
  constructor(private port: number) {
    this.server = net.createServer((socket) => this.handleConnection(socket));
  }
  
  start(): void {
    this.server.listen(this.port, () => {
      console.log(`TCP server listening on port ${this.port}`);
    });
    
    this.server.on('error', (err) => {
      console.error('Server error:', err);
    });
  }
  
  private handleConnection(socket: net.Socket): void {
    const clientId = `${socket.remoteAddress}:${socket.remotePort}`;
    
    const client: Client = {
      socket,
      id: clientId,
      address: socket.remoteAddress || 'unknown',
    };
    
    this.clients.set(clientId, client);
    console.log(`Client connected: ${clientId}`);
    
    // Set socket options
    socket.setKeepAlive(true, 60000); // Keep-alive every 60s
    socket.setNoDelay(true); // Disable Nagle's algorithm for low latency
    
    // Handle incoming data
    socket.on('data', (data) => {
      console.log(`Received from ${clientId}: ${data.toString()}`);
      
      // Echo back
      socket.write(`Echo: ${data.toString()}`);
    });
    
    // Handle client disconnect
    socket.on('close', (hadError) => {
      console.log(`Client disconnected: ${clientId}, error: ${hadError}`);
      this.clients.delete(clientId);
    });
    
    socket.on('error', (err) => {
      console.error(`Socket error for ${clientId}:`, err);
    });
    
    // Handle timeout
    socket.setTimeout(30000); // 30 second timeout
    socket.on('timeout', () => {
      console.log(`Client timeout: ${clientId}`);
      socket.end();
    });
  }
  
  broadcast(message: string): void {
    for (const [id, client] of this.clients) {
      client.socket.write(message);
    }
  }
  
  stop(): void {
    // Gracefully close all connections
    for (const [id, client] of this.clients) {
      client.socket.end();
    }
    
    this.server.close(() => {
      console.log('TCP server stopped');
    });
  }
}

// TCP Client
class TcpClient {
  private socket: net.Socket;
  private connected = false;
  
  constructor(
    private host: string,
    private port: number
  ) {
    this.socket = new net.Socket();
    this.setupHandlers();
  }
  
  private setupHandlers(): void {
    this.socket.on('connect', () => {
      this.connected = true;
      console.log('Connected to server');
    });
    
    this.socket.on('data', (data) => {
      console.log('Received:', data.toString());
    });
    
    this.socket.on('close', () => {
      this.connected = false;
      console.log('Connection closed');
    });
    
    this.socket.on('error', (err) => {
      console.error('Connection error:', err);
    });
  }
  
  connect(): Promise<void> {
    return new Promise((resolve, reject) => {
      this.socket.connect(this.port, this.host, () => {
        resolve();
      });
      
      this.socket.once('error', reject);
    });
  }
  
  send(data: string): void {
    if (!this.connected) {
      throw new Error('Not connected');
    }
    this.socket.write(data);
  }
  
  disconnect(): void {
    this.socket.end();
  }
}

// Usage
const server = new TcpServer(8080);
server.start();

// Client usage
async function runClient() {
  const client = new TcpClient('localhost', 8080);
  await client.connect();
  client.send('Hello, TCP server!');
}
```

### UDP Server and Client

```typescript
// udp/server.ts
import * as dgram from 'dgram';

interface UdpMessage {
  data: Buffer;
  rinfo: dgram.RemoteInfo;
}

class UdpServer {
  private socket: dgram.Socket;
  private clients: Map<string, dgram.RemoteInfo> = new Map();
  
  constructor(private port: number) {
    // Create IPv4 UDP socket
    this.socket = dgram.createSocket('udp4');
    this.setupHandlers();
  }
  
  private setupHandlers(): void {
    this.socket.on('message', (msg, rinfo) => {
      this.handleMessage(msg, rinfo);
    });
    
    this.socket.on('error', (err) => {
      console.error('UDP server error:', err);
      this.socket.close();
    });
    
    this.socket.on('listening', () => {
      const address = this.socket.address();
      console.log(`UDP server listening on ${address.address}:${address.port}`);
    });
  }
  
  private handleMessage(msg: Buffer, rinfo: dgram.RemoteInfo): void {
    const clientKey = `${rinfo.address}:${rinfo.port}`;
    const message = msg.toString();
    
    console.log(`Received from ${clientKey}: ${message}`);
    
    // Track client
    this.clients.set(clientKey, rinfo);
    
    // Send response
    const response = Buffer.from(`Echo: ${message}`);
    this.socket.send(response, rinfo.port, rinfo.address, (err) => {
      if (err) console.error('Send error:', err);
    });
  }
  
  start(): void {
    this.socket.bind(this.port);
  }
  
  broadcast(message: string): void {
    const buffer = Buffer.from(message);
    
    for (const [key, rinfo] of this.clients) {
      this.socket.send(buffer, rinfo.port, rinfo.address);
    }
  }
  
  stop(): void {
    this.socket.close(() => {
      console.log('UDP server stopped');
    });
  }
}

// UDP Client
class UdpClient {
  private socket: dgram.Socket;
  
  constructor(
    private serverHost: string,
    private serverPort: number
  ) {
    this.socket = dgram.createSocket('udp4');
    
    this.socket.on('message', (msg, rinfo) => {
      console.log(`Received from server: ${msg.toString()}`);
    });
    
    this.socket.on('error', (err) => {
      console.error('UDP client error:', err);
    });
  }
  
  send(message: string): void {
    const buffer = Buffer.from(message);
    
    this.socket.send(
      buffer,
      this.serverPort,
      this.serverHost,
      (err) => {
        if (err) console.error('Send error:', err);
      }
    );
  }
  
  close(): void {
    this.socket.close();
  }
}

// Usage
const udpServer = new UdpServer(8081);
udpServer.start();

const udpClient = new UdpClient('localhost', 8081);
udpClient.send('Hello, UDP server!');
```

### Reliable UDP Implementation

```typescript
// udp/reliable-udp.ts
import * as dgram from 'dgram';

/**
 * Building reliability on top of UDP
 * (Similar to what QUIC does)
 */

interface ReliablePacket {
  sequenceNumber: number;
  ack: number;
  timestamp: number;
  data: Buffer;
}

class ReliableUdpSender {
  private socket: dgram.Socket;
  private sequenceNumber = 0;
  private pendingAcks: Map<number, {
    packet: ReliablePacket;
    sentAt: number;
    retries: number;
  }> = new Map();
  private maxRetries = 5;
  private retryTimeout = 1000; // ms
  
  constructor(
    private host: string,
    private port: number
  ) {
    this.socket = dgram.createSocket('udp4');
    
    // Handle ACKs
    this.socket.on('message', (msg) => {
      this.handleAck(msg);
    });
    
    // Start retry timer
    setInterval(() => this.checkRetries(), 100);
  }
  
  send(data: Buffer): number {
    const packet: ReliablePacket = {
      sequenceNumber: this.sequenceNumber++,
      ack: -1,
      timestamp: Date.now(),
      data,
    };
    
    this.sendPacket(packet);
    
    // Track for acknowledgment
    this.pendingAcks.set(packet.sequenceNumber, {
      packet,
      sentAt: Date.now(),
      retries: 0,
    });
    
    return packet.sequenceNumber;
  }
  
  private sendPacket(packet: ReliablePacket): void {
    const serialized = this.serialize(packet);
    this.socket.send(serialized, this.port, this.host);
  }
  
  private handleAck(msg: Buffer): void {
    const packet = this.deserialize(msg);
    
    if (packet.ack >= 0) {
      // Remove from pending
      this.pendingAcks.delete(packet.ack);
      console.log(`ACK received for sequence ${packet.ack}`);
    }
  }
  
  private checkRetries(): void {
    const now = Date.now();
    
    for (const [seq, pending] of this.pendingAcks) {
      if (now - pending.sentAt > this.retryTimeout) {
        if (pending.retries >= this.maxRetries) {
          console.error(`Packet ${seq} failed after ${this.maxRetries} retries`);
          this.pendingAcks.delete(seq);
          continue;
        }
        
        // Retry
        console.log(`Retrying packet ${seq}, attempt ${pending.retries + 1}`);
        pending.retries++;
        pending.sentAt = now;
        this.sendPacket(pending.packet);
      }
    }
  }
  
  private serialize(packet: ReliablePacket): Buffer {
    const header = Buffer.alloc(16);
    header.writeUInt32BE(packet.sequenceNumber, 0);
    header.writeInt32BE(packet.ack, 4);
    header.writeBigInt64BE(BigInt(packet.timestamp), 8);
    return Buffer.concat([header, packet.data]);
  }
  
  private deserialize(buffer: Buffer): ReliablePacket {
    return {
      sequenceNumber: buffer.readUInt32BE(0),
      ack: buffer.readInt32BE(4),
      timestamp: Number(buffer.readBigInt64BE(8)),
      data: buffer.subarray(16),
    };
  }
}

class ReliableUdpReceiver {
  private socket: dgram.Socket;
  private expectedSequence = 0;
  private buffer: Map<number, ReliablePacket> = new Map();
  
  constructor(
    private port: number,
    private onMessage: (data: Buffer) => void
  ) {
    this.socket = dgram.createSocket('udp4');
    
    this.socket.on('message', (msg, rinfo) => {
      this.handlePacket(msg, rinfo);
    });
    
    this.socket.bind(port);
  }
  
  private handlePacket(msg: Buffer, rinfo: dgram.RemoteInfo): void {
    const packet = this.deserialize(msg);
    
    // Send ACK
    this.sendAck(packet.sequenceNumber, rinfo);
    
    if (packet.sequenceNumber === this.expectedSequence) {
      // In-order packet
      this.deliverPacket(packet);
      this.expectedSequence++;
      
      // Check buffer for next packets
      while (this.buffer.has(this.expectedSequence)) {
        const buffered = this.buffer.get(this.expectedSequence)!;
        this.buffer.delete(this.expectedSequence);
        this.deliverPacket(buffered);
        this.expectedSequence++;
      }
    } else if (packet.sequenceNumber > this.expectedSequence) {
      // Out-of-order, buffer it
      this.buffer.set(packet.sequenceNumber, packet);
    }
    // Else: duplicate, ignore
  }
  
  private deliverPacket(packet: ReliablePacket): void {
    this.onMessage(packet.data);
  }
  
  private sendAck(sequenceNumber: number, rinfo: dgram.RemoteInfo): void {
    const ackPacket: ReliablePacket = {
      sequenceNumber: 0,
      ack: sequenceNumber,
      timestamp: Date.now(),
      data: Buffer.alloc(0),
    };
    
    const buffer = this.serialize(ackPacket);
    this.socket.send(buffer, rinfo.port, rinfo.address);
  }
  
  private serialize(packet: ReliablePacket): Buffer {
    const header = Buffer.alloc(16);
    header.writeUInt32BE(packet.sequenceNumber, 0);
    header.writeInt32BE(packet.ack, 4);
    header.writeBigInt64BE(BigInt(packet.timestamp), 8);
    return Buffer.concat([header, packet.data]);
  }
  
  private deserialize(buffer: Buffer): ReliablePacket {
    return {
      sequenceNumber: buffer.readUInt32BE(0),
      ack: buffer.readInt32BE(4),
      timestamp: Number(buffer.readBigInt64BE(8)),
      data: buffer.subarray(16),
    };
  }
}
```

### Comparison: When to Use Each

```typescript
// comparison.ts

interface ProtocolChoice {
  protocol: 'TCP' | 'UDP';
  reasons: string[];
}

const useCases: Record<string, ProtocolChoice> = {
  // Use TCP
  'HTTP/HTTPS': {
    protocol: 'TCP',
    reasons: [
      'Web pages must load completely',
      'Data integrity is critical',
      'Order matters for HTML parsing',
    ],
  },
  
  'Database connections': {
    protocol: 'TCP',
    reasons: [
      'Queries and results must not be lost',
      'Transaction integrity',
      'Connection state management',
    ],
  },
  
  'File transfer (FTP, SFTP)': {
    protocol: 'TCP',
    reasons: [
      'Files must arrive complete',
      'Corruption is unacceptable',
    ],
  },
  
  'Email (SMTP, IMAP)': {
    protocol: 'TCP',
    reasons: [
      'Messages must be delivered reliably',
      'No partial emails',
    ],
  },
  
  'SSH/Remote terminals': {
    protocol: 'TCP',
    reasons: [
      'Commands must execute in order',
      'Every keystroke matters',
    ],
  },
  
  // Use UDP
  'Video streaming': {
    protocol: 'UDP',
    reasons: [
      'Dropping a frame is better than buffering',
      'Real-time is more important than perfect',
      'Retransmission would be too late',
    ],
  },
  
  'Online gaming': {
    protocol: 'UDP',
    reasons: [
      'Low latency is critical',
      'Old position data is worthless',
      'Can tolerate some packet loss',
    ],
  },
  
  'VoIP (Voice calls)': {
    protocol: 'UDP',
    reasons: [
      'Real-time audio needs low latency',
      'Small gaps are acceptable',
      'Retransmission would cause echo',
    ],
  },
  
  'DNS queries': {
    protocol: 'UDP',
    reasons: [
      'Simple request-response',
      'Small packets fit in single datagram',
      'Can retry on timeout',
    ],
  },
  
  'IoT sensors': {
    protocol: 'UDP',
    reasons: [
      'High frequency updates',
      'Missing one reading is OK',
      'Resource-constrained devices',
    ],
  },
  
  // Hybrid: QUIC
  'HTTP/3 (QUIC)': {
    protocol: 'UDP',
    reasons: [
      'UDP base for flexibility',
      'Built-in encryption',
      'Stream multiplexing without head-of-line blocking',
      'Reliability built in application layer',
    ],
  },
};
```

---

## Real-World Scenarios

### Scenario 1: Game Server Architecture

```typescript
// game/server.ts
import * as dgram from 'dgram';

interface GameState {
  players: Map<string, {
    x: number;
    y: number;
    health: number;
    lastUpdate: number;
  }>;
}

interface PlayerInput {
  playerId: string;
  sequence: number;
  dx: number;
  dy: number;
  action: 'move' | 'shoot' | 'jump';
}

class GameServer {
  private socket: dgram.Socket;
  private state: GameState = { players: new Map() };
  private tickRate = 60; // Updates per second
  
  constructor(private port: number) {
    this.socket = dgram.createSocket('udp4');
    
    this.socket.on('message', (msg, rinfo) => {
      this.handleInput(msg, rinfo);
    });
  }
  
  start(): void {
    this.socket.bind(this.port);
    
    // Game loop
    setInterval(() => {
      this.tick();
      this.broadcastState();
    }, 1000 / this.tickRate);
  }
  
  private handleInput(msg: Buffer, rinfo: dgram.RemoteInfo): void {
    try {
      const input: PlayerInput = JSON.parse(msg.toString());
      const playerId = `${rinfo.address}:${rinfo.port}`;
      
      // Apply input to game state
      let player = this.state.players.get(playerId);
      if (!player) {
        player = { x: 0, y: 0, health: 100, lastUpdate: Date.now() };
        this.state.players.set(playerId, player);
      }
      
      // Server-side validation and movement
      player.x += input.dx;
      player.y += input.dy;
      player.lastUpdate = Date.now();
      
    } catch (error) {
      // Ignore malformed packets
    }
  }
  
  private tick(): void {
    // Game logic, physics, collision detection
    // Remove disconnected players (no input for 5 seconds)
    const now = Date.now();
    for (const [id, player] of this.state.players) {
      if (now - player.lastUpdate > 5000) {
        this.state.players.delete(id);
      }
    }
  }
  
  private broadcastState(): void {
    // Send game state to all players
    const stateUpdate = {
      tick: Date.now(),
      players: Object.fromEntries(this.state.players),
    };
    
    const buffer = Buffer.from(JSON.stringify(stateUpdate));
    
    // UDP broadcast - some packets may be lost, that's OK
    for (const [id, player] of this.state.players) {
      const [address, port] = id.split(':');
      this.socket.send(buffer, parseInt(port), address);
    }
  }
}
```

### Scenario 2: Hybrid TCP/UDP Application

```typescript
// hybrid/server.ts

/**
 * Many real applications use both:
 * - TCP for critical data (login, purchases, chat messages)
 * - UDP for real-time data (position updates, voice)
 */

class HybridGameServer {
  private tcpServer: net.Server;
  private udpSocket: dgram.Socket;
  private players: Map<string, {
    tcpSocket: net.Socket;
    udpAddress?: { address: string; port: number };
    authenticated: boolean;
  }> = new Map();
  
  constructor(
    private tcpPort: number,
    private udpPort: number
  ) {
    // TCP for reliable operations
    this.tcpServer = net.createServer((socket) => {
      this.handleTcpConnection(socket);
    });
    
    // UDP for real-time updates
    this.udpSocket = dgram.createSocket('udp4');
    this.udpSocket.on('message', (msg, rinfo) => {
      this.handleUdpMessage(msg, rinfo);
    });
  }
  
  start(): void {
    this.tcpServer.listen(this.tcpPort);
    this.udpSocket.bind(this.udpPort);
    console.log(`TCP: ${this.tcpPort}, UDP: ${this.udpPort}`);
  }
  
  private handleTcpConnection(socket: net.Socket): void {
    socket.on('data', (data) => {
      const message = JSON.parse(data.toString());
      
      switch (message.type) {
        case 'login':
          this.handleLogin(socket, message);
          break;
        case 'chat':
          this.handleChat(socket, message);
          break;
        case 'purchase':
          this.handlePurchase(socket, message);
          break;
      }
    });
  }
  
  private handleLogin(socket: net.Socket, message: any): void {
    // Authenticate via TCP (reliable)
    const playerId = message.playerId;
    
    this.players.set(playerId, {
      tcpSocket: socket,
      authenticated: true,
    });
    
    // Send success response via TCP
    socket.write(JSON.stringify({
      type: 'loginSuccess',
      udpPort: this.udpPort,
    }));
  }
  
  private handleUdpMessage(msg: Buffer, rinfo: dgram.RemoteInfo): void {
    const message = JSON.parse(msg.toString());
    
    // Find player and verify authenticated
    const player = this.players.get(message.playerId);
    if (!player?.authenticated) {
      return; // Ignore unauthenticated UDP
    }
    
    // Store UDP address for responses
    player.udpAddress = { address: rinfo.address, port: rinfo.port };
    
    // Handle real-time updates
    if (message.type === 'position') {
      this.broadcastPosition(message);
    }
  }
  
  private broadcastPosition(position: any): void {
    const buffer = Buffer.from(JSON.stringify(position));
    
    for (const [id, player] of this.players) {
      if (player.udpAddress && id !== position.playerId) {
        this.udpSocket.send(
          buffer,
          player.udpAddress.port,
          player.udpAddress.address
        );
      }
    }
  }
  
  private handleChat(socket: net.Socket, message: any): void {
    // Chat via TCP - must be delivered
  }
  
  private handlePurchase(socket: net.Socket, message: any): void {
    // Purchases via TCP - critical, must succeed
  }
}

import * as net from 'net';
import * as dgram from 'dgram';
```

---

## Common Pitfalls

### 1. Using UDP Without Handling Packet Loss

```typescript
// ❌ BAD: Assuming UDP packets always arrive
udpSocket.send(importantData, port, host);
// Data might be lost!

// ✅ GOOD: Implement acknowledgment or use for non-critical data
// Option 1: Acknowledge important packets
sendWithAck(importantData);

// Option 2: Only use UDP for loss-tolerant data
udpSocket.send(positionUpdate, port, host); // OK to lose occasionally

function sendWithAck(data: Buffer): void {}
const udpSocket = dgram.createSocket('udp4');
const importantData = Buffer.from('');
const positionUpdate = Buffer.from('');
const port = 0;
const host = '';
```

### 2. Not Setting TCP Socket Options

```typescript
// ❌ BAD: Using default TCP settings
const socket = net.connect(port, host);

// ✅ GOOD: Configure for your use case
const socket = net.connect(port, host);
socket.setNoDelay(true);      // Disable Nagle for low latency
socket.setKeepAlive(true, 60000); // Detect dead connections
socket.setTimeout(30000);      // Timeout inactive connections
```

### 3. Blocking on TCP in Games

```typescript
// ❌ BAD: Waiting for TCP response for every frame
const position = await tcpClient.requestPosition(); // Blocks!
renderFrame(position);

// ✅ GOOD: Use UDP for real-time, TCP for critical only
// Last received position (from UDP)
let lastPosition = { x: 0, y: 0 };

udpClient.onPosition((pos) => {
  lastPosition = pos; // Non-blocking update
});

function renderFrame() {
  render(lastPosition); // Use latest available
}

function render(pos: { x: number; y: number }): void {}
const tcpClient = { requestPosition: async () => ({ x: 0, y: 0 }) };
const udpClient = { onPosition: (cb: Function) => {} };
```

---

## Interview Questions

### Q1: Explain the TCP three-way handshake.

**A:** 1) Client sends SYN (synchronize) with initial sequence number. 2) Server responds with SYN-ACK (synchronize-acknowledge) with its sequence number and acknowledging client's. 3) Client sends ACK acknowledging server's sequence. Connection established. This ensures both sides are ready and agree on sequence numbers for ordering.

### Q2: Why would you choose UDP over TCP for a video call?

**A:** 1) Low latency is critical - TCP retransmission would cause noticeable delay. 2) Old video frames are useless - showing a 500ms old frame is worse than dropping it. 3) Some packet loss is acceptable - codecs handle gaps gracefully. 4) No head-of-line blocking - one lost packet doesn't delay others. 5) Simpler congestion handling - can adjust quality in real-time.

### Q3: What is head-of-line blocking in TCP?

**A:** When one packet is lost, TCP holds back all subsequent packets until the lost one is retransmitted and received (to maintain ordering). This blocks the entire stream even if later packets arrived fine. UDP doesn't have this - each packet is independent. HTTP/3/QUIC solves this by implementing reliability per-stream, not per-connection.

### Q4: How does TCP handle congestion?

**A:** TCP uses several mechanisms: 1) Slow start - begins with small window, doubles each RTT. 2) Congestion avoidance - linear increase after threshold. 3) Fast retransmit - retransmits on 3 duplicate ACKs. 4) Fast recovery - doesn't restart slow start on loss. Modern algorithms like BBR measure actual bandwidth rather than just reacting to loss.

---

## Quick Reference Checklist

### Choose TCP When
- [ ] Data must be delivered (files, transactions)
- [ ] Order matters (sequential commands)
- [ ] You need connection state
- [ ] Building on HTTP/HTTPS

### Choose UDP When
- [ ] Real-time is more important than reliability
- [ ] Some data loss is acceptable
- [ ] Low latency is critical
- [ ] Simple request-response (DNS)
- [ ] Broadcasting to many recipients

### TCP Best Practices
- [ ] Set appropriate timeouts
- [ ] Use keep-alive for long connections
- [ ] Handle partial reads/writes
- [ ] Implement graceful shutdown

### UDP Best Practices
- [ ] Implement your own reliability if needed
- [ ] Handle out-of-order packets
- [ ] Keep packets small (< MTU)
- [ ] Rate limit to avoid overwhelming network

---

*Last updated: February 2026*

