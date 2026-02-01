# 🎥 WebRTC - Complete Guide

> A comprehensive guide to WebRTC - peer-to-peer connections, STUN/TURN servers, media streams, data channels, and signaling server implementation.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "WebRTC (Web Real-Time Communication) enables direct peer-to-peer audio, video, and data transfer between browsers without plugins, using ICE for NAT traversal, STUN/TURN for connectivity, and a signaling server for connection establishment."

### The 7 Key Concepts (Remember These!)
```
1. PEER-TO-PEER   → Direct connection between browsers (no server relay*)
2. SIGNALING      → Exchange connection info via your server (SDP, ICE)
3. ICE            → Interactive Connectivity Establishment (finds best path)
4. STUN           → Discovers public IP/port (NAT traversal)
5. TURN           → Relay server when P2P fails (~10-20% of connections)
6. SDP            → Session Description Protocol (media capabilities)
7. DATA CHANNEL   → Low-latency arbitrary data (not just media)
```

### WebRTC Connection Flow
```
┌─────────────────────────────────────────────────────────────────┐
│                   WEBRTC CONNECTION FLOW                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  PEER A                Signaling Server              PEER B     │
│  ──────                ────────────────              ──────     │
│                                                                 │
│  1. createOffer()                                               │
│         │                                                       │
│         ▼                                                       │
│  2. setLocalDescription(offer)                                  │
│         │                                                       │
│         └──────────▶ offer ─────────────────────────▶          │
│                                            3. setRemoteDescription│
│                                                      │          │
│                                            4. createAnswer()    │
│                                                      │          │
│                                            5. setLocalDescription│
│         ◀────────── answer ◀────────────────────────┘          │
│  6. setRemoteDescription                                        │
│         │                                                       │
│         ▼                                                       │
│  7. ICE candidates ◀════════════════════▶ ICE candidates       │
│         │                                            │          │
│         └────────────── P2P Connection ──────────────┘          │
│                                                                 │
│  * If P2P fails, TURN relay kicks in                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### STUN vs TURN
```
┌─────────────────────────────────────────────────────────────────┐
│                    STUN vs TURN                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  STUN (Session Traversal Utilities for NAT)                    │
│  ─────────────────────────────────────────                      │
│                                                                 │
│  [Browser A]──▶ STUN Server ──▶ "Your public IP: 1.2.3.4:5678" │
│       │                                                        │
│       └────────── Direct P2P connection ──────────▶[Browser B] │
│                                                                 │
│  • Free, lightweight                                           │
│  • Just discovers public IP/port                               │
│  • Works ~80-90% of the time                                   │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  TURN (Traversal Using Relays around NAT)                      │
│  ────────────────────────────────────────                       │
│                                                                 │
│  [Browser A]──▶ TURN Server ◀──[Browser B]                     │
│                    │                                           │
│                All traffic relayed                              │
│                                                                 │
│  • More expensive (bandwidth costs)                            │
│  • Relays all traffic                                          │
│  • Fallback when P2P fails (~10-20%)                           │
│  • Required for symmetric NATs, strict firewalls               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"ICE candidates"** | "We exchange ICE candidates via signaling for NAT traversal" |
| **"SDP offer/answer"** | "The caller creates an SDP offer, callee responds with answer" |
| **"Trickle ICE"** | "We use trickle ICE to send candidates as they're discovered" |
| **"DTLS-SRTP"** | "Media is encrypted using DTLS-SRTP, end-to-end by default" |
| **"Perfect negotiation"** | "We implement perfect negotiation pattern for robust connection handling" |
| **"Data channel"** | "We use WebRTC data channels for low-latency game state sync" |

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| STUN success rate | **~80-90%** | P2P works most of the time |
| TURN fallback rate | **~10-20%** | For symmetric NAT, firewalls |
| TURN bandwidth cost | **High** | All traffic relayed |
| ICE gathering timeout | **~30 seconds** | Finding best path |
| RTCDataChannel latency | **~50ms** | Lower than WebSocket |
| Max data channel message | **16KB-64KB** | Browser dependent |

### The "Wow" Statement (Memorize This!)
> "We built a video conferencing app using WebRTC with a custom signaling server over WebSocket. The flow is: client creates RTCPeerConnection with STUN/TURN servers, gets local media stream, creates offer, exchanges SDP and ICE candidates via signaling. We implement trickle ICE for faster connection establishment - candidates are sent as discovered rather than waiting for all. For reliability, we include TURN servers that handle ~15% of connections where P2P fails due to symmetric NAT. We also use data channels for real-time features like chat and screen annotations, leveraging WebRTC's low-latency transport. The media is end-to-end encrypted by default via DTLS-SRTP."

### Quick Architecture Drawing (Draw This!)
```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│      ┌───────────┐                      ┌───────────┐          │
│      │  PEER A   │                      │  PEER B   │          │
│      │           │                      │           │          │
│      │ Camera    │                      │ Camera    │          │
│      │ Mic       │                      │ Mic       │          │
│      └─────┬─────┘                      └─────┬─────┘          │
│            │                                  │                │
│            │    ┌───────────────────┐        │                │
│            └───▶│ Signaling Server  │◀───────┘                │
│                 │ (WebSocket/HTTP)  │                         │
│                 │ • Exchange SDP    │                         │
│                 │ • Exchange ICE    │                         │
│                 └───────────────────┘                         │
│                                                                │
│            │                                  │                │
│            │ ┌─────────────────────────────┐ │                │
│            └▶│        ICE Layer            │◀┘                │
│              │ 1. Try P2P (via STUN)       │                  │
│              │ 2. Fallback to TURN relay   │                  │
│              └─────────────────────────────┘                  │
│                           │                                   │
│                           ▼                                   │
│              ┌─────────────────────────────┐                  │
│              │     P2P Media Stream        │                  │
│              │   (or TURN relay ~15%)      │                  │
│              └─────────────────────────────┘                  │
│                                                                │
└─────────────────────────────────────────────────────────────────┘
```

### Interview Rapid Fire (Practice These!)

**Q: "What is WebRTC?"**
> "API for real-time peer-to-peer audio, video, and data in browsers. Uses ICE for connectivity, STUN/TURN for NAT traversal, signaling server for setup."

**Q: "What is a signaling server?"**
> "Your server that exchanges connection metadata (SDP offers/answers, ICE candidates) between peers. WebRTC doesn't specify protocol - use WebSocket, HTTP, or anything."

**Q: "STUN vs TURN?"**
> "STUN discovers your public IP for direct P2P - free, lightweight, works 80-90%. TURN relays traffic when P2P fails - more expensive, needed for symmetric NAT."

**Q: "What is SDP?"**
> "Session Description Protocol. Describes media capabilities - codecs, encryption, resolution. Exchanged as offer/answer to negotiate connection parameters."

**Q: "What is ICE?"**
> "Interactive Connectivity Establishment. Finds the best path between peers. Gathers candidates (host, srflx, relay), tests connectivity, selects optimal route."

**Q: "When would you use data channels instead of WebSocket?"**
> "Lower latency (no TCP head-of-line blocking), P2P (no server relay), unreliable/ordered options. Good for gaming, collaborative apps, file transfer."

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "How does WebRTC work?"

**Junior Answer:**
> "It lets browsers connect directly to each other for video calls."

**Senior Answer:**
> "WebRTC enables peer-to-peer communication but requires careful orchestration:

1. **Signaling** (your server): Exchange connection metadata. Not part of WebRTC spec - use WebSocket, HTTP, carrier pigeon. Peers exchange SDP offers/answers describing media capabilities.

2. **ICE (NAT Traversal)**: Most devices are behind NAT. ICE finds a path:
   - Host candidates: local IP (works on same network)
   - Server reflexive (srflx): public IP via STUN
   - Relay: TURN server as fallback

3. **Connection**: After exchanging SDP and ICE candidates, peers establish DTLS connection for encryption, then SRTP for media.

4. **Media/Data**: Streams flow directly P2P, or through TURN if P2P fails. Data channels use SCTP over DTLS for arbitrary data with configurable reliability."

### When Asked: "How do you handle connection failures?"

**Junior Answer:**
> "Add a TURN server."

**Senior Answer:**
> "Multi-layered approach:

1. **ICE Configuration**: Include multiple STUN servers and TURN servers (TCP and UDP). Test TURN credentials before calls.

2. **ICE Restart**: When connection drops, trigger ICE restart (`restartIce()`) rather than creating new peer connection. Preserves state.

3. **Connection State Monitoring**: Listen to `connectionstatechange`, `iceconnectionstatechange`. Implement reconnection logic on `disconnected` or `failed`.

4. **Fallback Strategy**: If WebRTC completely fails (restrictive firewall), fall back to server-mediated solution.

5. **Quality Adaptation**: Monitor `getStats()` for packet loss, jitter. Adjust bitrate, resolution dynamically."

### Follow-up Questions to Expect:

| They Ask | You Answer |
|----------|------------|
| "What percentage needs TURN?" | "Typically 10-20%. Higher in corporate environments with strict firewalls. Always provision TURN - users notice when it fails." |
| "How do you scale TURN?" | "TURN is stateful and bandwidth-intensive. Use multiple servers with geographic distribution. Consider coturn, Twilio TURN, or cloud provider solutions." |
| "Is WebRTC encrypted?" | "Yes, mandatory. DTLS for key exchange, SRTP for media encryption. End-to-end encrypted unless you're doing server-side processing (SFU/MCU)." |
| "What about multi-party calls?" | "Mesh for small groups (<4-5), SFU (Selective Forwarding Unit) for medium, MCU (Multipoint Control Unit) for large. Each has tradeoffs." |

---

## 📚 Table of Contents

1. [Basic Peer Connection](#1-basic-peer-connection)
2. [Signaling Server](#2-signaling-server)
3. [Media Streams](#3-media-streams)
4. [Data Channels](#4-data-channels)
5. [ICE and NAT Traversal](#5-ice-and-nat-traversal)
6. [Error Handling & Reconnection](#6-error-handling--reconnection)
7. [Multi-Party Calls](#7-multi-party-calls)
8. [Screen Sharing](#8-screen-sharing)
9. [Common Pitfalls](#9-common-pitfalls)
10. [Interview Questions](#10-interview-questions)

---

## 1. Basic Peer Connection

```typescript
// ════════════════════════════════════════════════════════════════
// BASIC WEBRTC PEER CONNECTION
// ════════════════════════════════════════════════════════════════

// ICE Server Configuration
const iceServers: RTCIceServer[] = [
  // Public STUN servers
  { urls: 'stun:stun.l.google.com:19302' },
  { urls: 'stun:stun1.l.google.com:19302' },
  
  // Your TURN server
  {
    urls: [
      'turn:turn.example.com:3478?transport=udp',
      'turn:turn.example.com:3478?transport=tcp',
      'turns:turn.example.com:5349?transport=tcp'  // TLS
    ],
    username: 'turnuser',
    credential: 'turnpassword'
  }
];

// Create peer connection
const peerConnection = new RTCPeerConnection({
  iceServers,
  iceCandidatePoolSize: 10,  // Pre-gather candidates
  bundlePolicy: 'max-bundle'  // Bundle media for efficiency
});

// ════════════════════════════════════════════════════════════════
// EVENT HANDLERS
// ════════════════════════════════════════════════════════════════

// ICE candidate discovered
peerConnection.onicecandidate = (event) => {
  if (event.candidate) {
    // Send candidate to peer via signaling
    signalingChannel.send({
      type: 'ice-candidate',
      candidate: event.candidate
    });
  }
};

// ICE gathering complete
peerConnection.onicegatheringstatechange = () => {
  console.log('ICE gathering state:', peerConnection.iceGatheringState);
  // 'new' | 'gathering' | 'complete'
};

// ICE connection state
peerConnection.oniceconnectionstatechange = () => {
  console.log('ICE connection state:', peerConnection.iceConnectionState);
  // 'new' | 'checking' | 'connected' | 'completed' | 'failed' | 'disconnected' | 'closed'
  
  if (peerConnection.iceConnectionState === 'failed') {
    // Try ICE restart
    peerConnection.restartIce();
  }
};

// Overall connection state
peerConnection.onconnectionstatechange = () => {
  console.log('Connection state:', peerConnection.connectionState);
  // 'new' | 'connecting' | 'connected' | 'disconnected' | 'failed' | 'closed'
};

// Remote track received
peerConnection.ontrack = (event) => {
  console.log('Remote track received:', event.track.kind);
  
  // Add to video element
  const remoteVideo = document.getElementById('remoteVideo') as HTMLVideoElement;
  if (remoteVideo.srcObject !== event.streams[0]) {
    remoteVideo.srcObject = event.streams[0];
  }
};

// ════════════════════════════════════════════════════════════════
// OFFER/ANSWER EXCHANGE (Caller)
// ════════════════════════════════════════════════════════════════

async function makeCall() {
  // Add local tracks (see Media Streams section)
  const localStream = await navigator.mediaDevices.getUserMedia({
    video: true,
    audio: true
  });
  
  localStream.getTracks().forEach(track => {
    peerConnection.addTrack(track, localStream);
  });

  // Create offer
  const offer = await peerConnection.createOffer({
    offerToReceiveAudio: true,
    offerToReceiveVideo: true
  });

  // Set local description
  await peerConnection.setLocalDescription(offer);

  // Send offer via signaling
  signalingChannel.send({
    type: 'offer',
    sdp: peerConnection.localDescription
  });
}

// ════════════════════════════════════════════════════════════════
// HANDLING OFFER (Callee)
// ════════════════════════════════════════════════════════════════

async function handleOffer(offer: RTCSessionDescriptionInit) {
  // Set remote description
  await peerConnection.setRemoteDescription(new RTCSessionDescription(offer));

  // Add local tracks
  const localStream = await navigator.mediaDevices.getUserMedia({
    video: true,
    audio: true
  });
  
  localStream.getTracks().forEach(track => {
    peerConnection.addTrack(track, localStream);
  });

  // Create answer
  const answer = await peerConnection.createAnswer();
  
  // Set local description
  await peerConnection.setLocalDescription(answer);

  // Send answer via signaling
  signalingChannel.send({
    type: 'answer',
    sdp: peerConnection.localDescription
  });
}

// ════════════════════════════════════════════════════════════════
// HANDLING ANSWER (Caller)
// ════════════════════════════════════════════════════════════════

async function handleAnswer(answer: RTCSessionDescriptionInit) {
  await peerConnection.setRemoteDescription(new RTCSessionDescription(answer));
}

// ════════════════════════════════════════════════════════════════
// HANDLING ICE CANDIDATE
// ════════════════════════════════════════════════════════════════

async function handleIceCandidate(candidate: RTCIceCandidateInit) {
  try {
    await peerConnection.addIceCandidate(new RTCIceCandidate(candidate));
  } catch (error) {
    console.error('Error adding ICE candidate:', error);
  }
}
```

---

## 2. Signaling Server

```typescript
// ════════════════════════════════════════════════════════════════
// SIGNALING SERVER (Node.js + WebSocket)
// ════════════════════════════════════════════════════════════════

import { WebSocketServer, WebSocket } from 'ws';
import { createServer } from 'http';

interface SignalingMessage {
  type: 'join' | 'offer' | 'answer' | 'ice-candidate' | 'leave';
  roomId?: string;
  sdp?: RTCSessionDescriptionInit;
  candidate?: RTCIceCandidateInit;
  from?: string;
  to?: string;
}

interface Client {
  id: string;
  ws: WebSocket;
  roomId: string | null;
}

class SignalingServer {
  private wss: WebSocketServer;
  private clients: Map<string, Client> = new Map();
  private rooms: Map<string, Set<string>> = new Map(); // roomId -> clientIds

  constructor(server: ReturnType<typeof createServer>) {
    this.wss = new WebSocketServer({ server });
    this.setupHandlers();
  }

  private setupHandlers() {
    this.wss.on('connection', (ws) => {
      const clientId = this.generateId();
      const client: Client = { id: clientId, ws, roomId: null };
      this.clients.set(clientId, client);

      console.log(`Client connected: ${clientId}`);

      // Send client their ID
      this.send(ws, { type: 'connected', clientId });

      ws.on('message', (data) => {
        try {
          const message: SignalingMessage = JSON.parse(data.toString());
          this.handleMessage(clientId, message);
        } catch (error) {
          console.error('Invalid message:', error);
        }
      });

      ws.on('close', () => {
        this.handleDisconnect(clientId);
      });
    });
  }

  private handleMessage(clientId: string, message: SignalingMessage) {
    const client = this.clients.get(clientId);
    if (!client) return;

    switch (message.type) {
      case 'join':
        this.handleJoin(client, message.roomId!);
        break;

      case 'offer':
      case 'answer':
      case 'ice-candidate':
        this.relayMessage(client, message);
        break;

      case 'leave':
        this.handleLeave(client);
        break;
    }
  }

  private handleJoin(client: Client, roomId: string) {
    // Leave current room if any
    if (client.roomId) {
      this.handleLeave(client);
    }

    // Join new room
    client.roomId = roomId;

    if (!this.rooms.has(roomId)) {
      this.rooms.set(roomId, new Set());
    }
    
    const room = this.rooms.get(roomId)!;
    
    // Notify existing members
    room.forEach(existingClientId => {
      const existingClient = this.clients.get(existingClientId);
      if (existingClient) {
        this.send(existingClient.ws, {
          type: 'peer-joined',
          peerId: client.id
        });
        
        // Tell new client about existing peer
        this.send(client.ws, {
          type: 'peer-joined',
          peerId: existingClientId
        });
      }
    });

    room.add(client.id);
    console.log(`Client ${client.id} joined room ${roomId}`);
  }

  private handleLeave(client: Client) {
    if (!client.roomId) return;

    const room = this.rooms.get(client.roomId);
    if (room) {
      room.delete(client.id);
      
      // Notify other members
      room.forEach(otherId => {
        const other = this.clients.get(otherId);
        if (other) {
          this.send(other.ws, {
            type: 'peer-left',
            peerId: client.id
          });
        }
      });

      if (room.size === 0) {
        this.rooms.delete(client.roomId);
      }
    }

    client.roomId = null;
  }

  private relayMessage(from: Client, message: SignalingMessage) {
    if (!from.roomId) return;

    const room = this.rooms.get(from.roomId);
    if (!room) return;

    // If specific recipient, send only to them
    if (message.to) {
      const target = this.clients.get(message.to);
      if (target && room.has(target.id)) {
        this.send(target.ws, { ...message, from: from.id });
      }
      return;
    }

    // Broadcast to all others in room
    room.forEach(clientId => {
      if (clientId !== from.id) {
        const client = this.clients.get(clientId);
        if (client) {
          this.send(client.ws, { ...message, from: from.id });
        }
      }
    });
  }

  private handleDisconnect(clientId: string) {
    const client = this.clients.get(clientId);
    if (client) {
      this.handleLeave(client);
      this.clients.delete(clientId);
    }
    console.log(`Client disconnected: ${clientId}`);
  }

  private send(ws: WebSocket, data: any) {
    if (ws.readyState === WebSocket.OPEN) {
      ws.send(JSON.stringify(data));
    }
  }

  private generateId(): string {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }
}

// Start server
const httpServer = createServer();
const signaling = new SignalingServer(httpServer);

httpServer.listen(3000, () => {
  console.log('Signaling server running on port 3000');
});

// ════════════════════════════════════════════════════════════════
// CLIENT-SIDE SIGNALING
// ════════════════════════════════════════════════════════════════

class SignalingClient {
  private ws: WebSocket | null = null;
  private clientId: string | null = null;
  public onPeerJoined: ((peerId: string) => void) | null = null;
  public onPeerLeft: ((peerId: string) => void) | null = null;
  public onOffer: ((from: string, sdp: RTCSessionDescriptionInit) => void) | null = null;
  public onAnswer: ((from: string, sdp: RTCSessionDescriptionInit) => void) | null = null;
  public onIceCandidate: ((from: string, candidate: RTCIceCandidateInit) => void) | null = null;

  connect(url: string): Promise<void> {
    return new Promise((resolve, reject) => {
      this.ws = new WebSocket(url);

      this.ws.onopen = () => {
        console.log('Signaling connected');
      };

      this.ws.onmessage = (event) => {
        const message = JSON.parse(event.data);
        this.handleMessage(message);
        
        if (message.type === 'connected') {
          this.clientId = message.clientId;
          resolve();
        }
      };

      this.ws.onerror = (error) => {
        reject(error);
      };

      this.ws.onclose = () => {
        console.log('Signaling disconnected');
      };
    });
  }

  private handleMessage(message: any) {
    switch (message.type) {
      case 'peer-joined':
        this.onPeerJoined?.(message.peerId);
        break;
      case 'peer-left':
        this.onPeerLeft?.(message.peerId);
        break;
      case 'offer':
        this.onOffer?.(message.from, message.sdp);
        break;
      case 'answer':
        this.onAnswer?.(message.from, message.sdp);
        break;
      case 'ice-candidate':
        this.onIceCandidate?.(message.from, message.candidate);
        break;
    }
  }

  joinRoom(roomId: string) {
    this.send({ type: 'join', roomId });
  }

  sendOffer(to: string, sdp: RTCSessionDescriptionInit) {
    this.send({ type: 'offer', to, sdp });
  }

  sendAnswer(to: string, sdp: RTCSessionDescriptionInit) {
    this.send({ type: 'answer', to, sdp });
  }

  sendIceCandidate(to: string, candidate: RTCIceCandidateInit) {
    this.send({ type: 'ice-candidate', to, candidate });
  }

  private send(message: any) {
    if (this.ws?.readyState === WebSocket.OPEN) {
      this.ws.send(JSON.stringify(message));
    }
  }

  disconnect() {
    this.ws?.close();
  }
}
```

---

## 3. Media Streams

```typescript
// ════════════════════════════════════════════════════════════════
// MEDIA STREAMS - getUserMedia
// ════════════════════════════════════════════════════════════════

// Basic video/audio
async function getMedia(): Promise<MediaStream> {
  return navigator.mediaDevices.getUserMedia({
    video: true,
    audio: true
  });
}

// With constraints
async function getMediaWithConstraints(): Promise<MediaStream> {
  return navigator.mediaDevices.getUserMedia({
    video: {
      width: { ideal: 1280, max: 1920 },
      height: { ideal: 720, max: 1080 },
      frameRate: { ideal: 30, max: 60 },
      facingMode: 'user',  // 'user' = front camera, 'environment' = back
      aspectRatio: { ideal: 16/9 }
    },
    audio: {
      echoCancellation: true,
      noiseSuppression: true,
      autoGainControl: true,
      sampleRate: 48000,
      channelCount: 1
    }
  });
}

// Select specific device
async function getMediaFromDevice(deviceId: string): Promise<MediaStream> {
  return navigator.mediaDevices.getUserMedia({
    video: { deviceId: { exact: deviceId } },
    audio: true
  });
}

// ════════════════════════════════════════════════════════════════
// DEVICE ENUMERATION
// ════════════════════════════════════════════════════════════════

async function getDevices(): Promise<{
  videoInputs: MediaDeviceInfo[];
  audioInputs: MediaDeviceInfo[];
  audioOutputs: MediaDeviceInfo[];
}> {
  const devices = await navigator.mediaDevices.enumerateDevices();
  
  return {
    videoInputs: devices.filter(d => d.kind === 'videoinput'),
    audioInputs: devices.filter(d => d.kind === 'audioinput'),
    audioOutputs: devices.filter(d => d.kind === 'audiooutput')
  };
}

// Switch camera
async function switchCamera(peerConnection: RTCPeerConnection, newDeviceId: string) {
  const newStream = await navigator.mediaDevices.getUserMedia({
    video: { deviceId: { exact: newDeviceId } }
  });

  const [newTrack] = newStream.getVideoTracks();
  
  const sender = peerConnection.getSenders().find(s => 
    s.track?.kind === 'video'
  );
  
  if (sender) {
    await sender.replaceTrack(newTrack);
  }
}

// ════════════════════════════════════════════════════════════════
// TRACK CONTROL
// ════════════════════════════════════════════════════════════════

function muteAudio(stream: MediaStream) {
  stream.getAudioTracks().forEach(track => {
    track.enabled = false;
  });
}

function unmuteAudio(stream: MediaStream) {
  stream.getAudioTracks().forEach(track => {
    track.enabled = true;
  });
}

function toggleVideo(stream: MediaStream): boolean {
  const videoTrack = stream.getVideoTracks()[0];
  if (videoTrack) {
    videoTrack.enabled = !videoTrack.enabled;
    return videoTrack.enabled;
  }
  return false;
}

function stopAllTracks(stream: MediaStream) {
  stream.getTracks().forEach(track => track.stop());
}

// ════════════════════════════════════════════════════════════════
// ADDING/REPLACING TRACKS
// ════════════════════════════════════════════════════════════════

// Add track to peer connection
function addTrackToPeerConnection(
  peerConnection: RTCPeerConnection,
  track: MediaStreamTrack,
  stream: MediaStream
): RTCRtpSender {
  return peerConnection.addTrack(track, stream);
}

// Replace track without renegotiation
async function replaceTrack(
  sender: RTCRtpSender,
  newTrack: MediaStreamTrack
): Promise<void> {
  await sender.replaceTrack(newTrack);
}

// Remove track
function removeTrack(
  peerConnection: RTCPeerConnection,
  sender: RTCRtpSender
): void {
  peerConnection.removeTrack(sender);
}

// ════════════════════════════════════════════════════════════════
// BITRATE CONTROL
// ════════════════════════════════════════════════════════════════

async function setVideoBitrate(
  peerConnection: RTCPeerConnection,
  maxBitrate: number  // in bps
) {
  const sender = peerConnection.getSenders().find(s => 
    s.track?.kind === 'video'
  );
  
  if (!sender) return;

  const params = sender.getParameters();
  
  if (!params.encodings || params.encodings.length === 0) {
    params.encodings = [{}];
  }
  
  params.encodings[0].maxBitrate = maxBitrate;
  
  await sender.setParameters(params);
}

// Adaptive quality based on connection
async function adaptQuality(peerConnection: RTCPeerConnection) {
  const stats = await peerConnection.getStats();
  
  let packetsLost = 0;
  let packetsReceived = 0;
  
  stats.forEach(report => {
    if (report.type === 'inbound-rtp' && report.kind === 'video') {
      packetsLost = report.packetsLost || 0;
      packetsReceived = report.packetsReceived || 0;
    }
  });
  
  const lossRate = packetsLost / (packetsLost + packetsReceived);
  
  if (lossRate > 0.1) {
    // High packet loss, reduce quality
    await setVideoBitrate(peerConnection, 500000);  // 500 kbps
  } else if (lossRate > 0.05) {
    // Moderate loss
    await setVideoBitrate(peerConnection, 1000000);  // 1 Mbps
  } else {
    // Good connection
    await setVideoBitrate(peerConnection, 2500000);  // 2.5 Mbps
  }
}
```

---

## 4. Data Channels

```typescript
// ════════════════════════════════════════════════════════════════
// WEBRTC DATA CHANNELS
// ════════════════════════════════════════════════════════════════

// Create data channel (on initiator side)
function createDataChannel(
  peerConnection: RTCPeerConnection,
  label: string,
  options?: RTCDataChannelInit
): RTCDataChannel {
  const channel = peerConnection.createDataChannel(label, {
    ordered: true,           // Guarantee order (default)
    // OR for unreliable (UDP-like):
    // ordered: false,
    // maxRetransmits: 0,    // No retries
    // maxPacketLifeTime: 100, // ms before giving up
    ...options
  });

  setupDataChannelHandlers(channel);
  return channel;
}

// Handle incoming data channel (on receiver side)
peerConnection.ondatachannel = (event) => {
  const channel = event.channel;
  console.log('Data channel received:', channel.label);
  setupDataChannelHandlers(channel);
};

function setupDataChannelHandlers(channel: RTCDataChannel) {
  channel.onopen = () => {
    console.log(`Data channel ${channel.label} opened`);
  };

  channel.onclose = () => {
    console.log(`Data channel ${channel.label} closed`);
  };

  channel.onerror = (error) => {
    console.error(`Data channel ${channel.label} error:`, error);
  };

  channel.onmessage = (event) => {
    console.log(`Message on ${channel.label}:`, event.data);
    // Handle message
    handleDataChannelMessage(channel.label, event.data);
  };
}

// ════════════════════════════════════════════════════════════════
// SENDING DATA
// ════════════════════════════════════════════════════════════════

function sendMessage(channel: RTCDataChannel, data: any) {
  if (channel.readyState !== 'open') {
    console.warn('Data channel not open');
    return false;
  }

  // Check buffer
  if (channel.bufferedAmount > channel.bufferedAmountLowThreshold) {
    console.warn('Buffer full, waiting...');
    return false;
  }

  if (typeof data === 'string') {
    channel.send(data);
  } else {
    channel.send(JSON.stringify(data));
  }
  
  return true;
}

// Send binary data
function sendBinary(channel: RTCDataChannel, data: ArrayBuffer | Blob) {
  if (channel.readyState !== 'open') return false;
  channel.send(data);
  return true;
}

// ════════════════════════════════════════════════════════════════
// FILE TRANSFER OVER DATA CHANNEL
// ════════════════════════════════════════════════════════════════

const CHUNK_SIZE = 16384;  // 16KB chunks

interface FileTransfer {
  id: string;
  name: string;
  size: number;
  type: string;
  chunks: number;
  received: number;
  data: ArrayBuffer[];
}

const transfers = new Map<string, FileTransfer>();

async function sendFile(channel: RTCDataChannel, file: File) {
  const transferId = `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;

  // Send metadata first
  channel.send(JSON.stringify({
    type: 'file-start',
    id: transferId,
    name: file.name,
    size: file.size,
    fileType: file.type,
    chunks: Math.ceil(file.size / CHUNK_SIZE)
  }));

  // Read and send chunks
  const reader = new FileReader();
  let offset = 0;

  const readNextChunk = () => {
    const slice = file.slice(offset, offset + CHUNK_SIZE);
    reader.readAsArrayBuffer(slice);
  };

  reader.onload = async (e) => {
    const chunk = e.target?.result as ArrayBuffer;
    
    // Wait for buffer to clear if needed
    while (channel.bufferedAmount > CHUNK_SIZE * 10) {
      await new Promise(resolve => setTimeout(resolve, 50));
    }

    // Send chunk with header
    const header = new TextEncoder().encode(JSON.stringify({
      type: 'file-chunk',
      id: transferId,
      offset
    }) + '\n');
    
    const combined = new Uint8Array(header.length + chunk.byteLength);
    combined.set(header, 0);
    combined.set(new Uint8Array(chunk), header.length);
    
    channel.send(combined.buffer);

    offset += chunk.byteLength;

    if (offset < file.size) {
      readNextChunk();
    } else {
      // Send completion
      channel.send(JSON.stringify({
        type: 'file-end',
        id: transferId
      }));
    }
  };

  readNextChunk();
}

// Handle received file data
function handleDataChannelMessage(label: string, data: string | ArrayBuffer) {
  if (typeof data === 'string') {
    const message = JSON.parse(data);
    
    if (message.type === 'file-start') {
      transfers.set(message.id, {
        id: message.id,
        name: message.name,
        size: message.size,
        type: message.fileType,
        chunks: message.chunks,
        received: 0,
        data: []
      });
    } else if (message.type === 'file-end') {
      const transfer = transfers.get(message.id);
      if (transfer) {
        const blob = new Blob(transfer.data, { type: transfer.type });
        downloadFile(blob, transfer.name);
        transfers.delete(message.id);
      }
    }
  } else {
    // Binary chunk
    const arrayBuffer = data as ArrayBuffer;
    const view = new Uint8Array(arrayBuffer);
    
    // Find header end
    let headerEnd = 0;
    for (let i = 0; i < Math.min(view.length, 1000); i++) {
      if (view[i] === 10) {  // newline
        headerEnd = i;
        break;
      }
    }
    
    const headerStr = new TextDecoder().decode(view.slice(0, headerEnd));
    const header = JSON.parse(headerStr);
    const chunkData = arrayBuffer.slice(headerEnd + 1);
    
    const transfer = transfers.get(header.id);
    if (transfer) {
      transfer.data.push(chunkData);
      transfer.received++;
      
      // Progress callback
      const progress = (transfer.received / transfer.chunks) * 100;
      console.log(`File transfer ${progress.toFixed(1)}%`);
    }
  }
}

function downloadFile(blob: Blob, filename: string) {
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = filename;
  a.click();
  URL.revokeObjectURL(url);
}

// ════════════════════════════════════════════════════════════════
// TYPED DATA CHANNEL WRAPPER
// ════════════════════════════════════════════════════════════════

interface DataChannelMessage {
  type: string;
  [key: string]: any;
}

class TypedDataChannel {
  private channel: RTCDataChannel;
  private handlers: Map<string, ((data: any) => void)[]> = new Map();

  constructor(channel: RTCDataChannel) {
    this.channel = channel;
    
    channel.onmessage = (event) => {
      try {
        const message: DataChannelMessage = JSON.parse(event.data);
        const handlers = this.handlers.get(message.type) || [];
        handlers.forEach(h => h(message));
      } catch {
        console.error('Invalid message format');
      }
    };
  }

  on(type: string, handler: (data: any) => void) {
    if (!this.handlers.has(type)) {
      this.handlers.set(type, []);
    }
    this.handlers.get(type)!.push(handler);
  }

  send(type: string, data: any = {}) {
    this.channel.send(JSON.stringify({ type, ...data }));
  }

  get isOpen(): boolean {
    return this.channel.readyState === 'open';
  }
}

// Usage
const typedChannel = new TypedDataChannel(dataChannel);

typedChannel.on('chat', (data) => {
  console.log('Chat message:', data.text);
});

typedChannel.on('cursor', (data) => {
  updateRemoteCursor(data.x, data.y);
});

typedChannel.send('chat', { text: 'Hello!' });
typedChannel.send('cursor', { x: 100, y: 200 });
```

---

## 5. ICE and NAT Traversal

```typescript
// ════════════════════════════════════════════════════════════════
// ICE CONFIGURATION AND TROUBLESHOOTING
// ════════════════════════════════════════════════════════════════

// Comprehensive ICE server configuration
const iceConfig: RTCConfiguration = {
  iceServers: [
    // Multiple STUN servers for reliability
    { urls: 'stun:stun.l.google.com:19302' },
    { urls: 'stun:stun1.l.google.com:19302' },
    { urls: 'stun:stun2.l.google.com:19302' },
    
    // TURN server with multiple protocols
    {
      urls: [
        'turn:turn.example.com:3478?transport=udp',
        'turn:turn.example.com:3478?transport=tcp',
        'turns:turn.example.com:443?transport=tcp'  // TLS on 443 for firewall bypass
      ],
      username: process.env.TURN_USERNAME,
      credential: process.env.TURN_CREDENTIAL
    }
  ],
  
  // ICE transport policy
  iceTransportPolicy: 'all',  // 'all' | 'relay' (force TURN)
  
  // Bundle policy
  bundlePolicy: 'max-bundle',  // Single transport for all media
  
  // Pre-gather ICE candidates
  iceCandidatePoolSize: 10
};

// ════════════════════════════════════════════════════════════════
// ICE CANDIDATE TYPES
// ════════════════════════════════════════════════════════════════

/*
ICE Candidate Types:
1. host     - Local IP address (works on same network)
2. srflx    - Server reflexive (public IP via STUN)
3. prflx    - Peer reflexive (discovered during connectivity checks)
4. relay    - TURN relay (when direct connection fails)
*/

function analyzeIceCandidate(candidate: RTCIceCandidate) {
  const parts = candidate.candidate.split(' ');
  
  return {
    foundation: parts[0],
    component: parts[1],  // 1 = RTP, 2 = RTCP
    protocol: parts[2],   // udp or tcp
    priority: parseInt(parts[3]),
    ip: parts[4],
    port: parseInt(parts[5]),
    type: parts[7],       // host, srflx, prflx, relay
    relatedAddress: parts[9],   // For srflx/relay
    relatedPort: parts[11]
  };
}

// ════════════════════════════════════════════════════════════════
// ICE CONNECTION MONITORING
// ════════════════════════════════════════════════════════════════

class IceMonitor {
  private pc: RTCPeerConnection;
  private candidatesGathered: RTCIceCandidate[] = [];

  constructor(peerConnection: RTCPeerConnection) {
    this.pc = peerConnection;
    this.setupHandlers();
  }

  private setupHandlers() {
    this.pc.onicecandidate = (event) => {
      if (event.candidate) {
        this.candidatesGathered.push(event.candidate);
        const info = analyzeIceCandidate(event.candidate);
        console.log('ICE candidate:', info.type, info.ip, info.port);
      }
    };

    this.pc.onicegatheringstatechange = () => {
      console.log('ICE gathering state:', this.pc.iceGatheringState);
      
      if (this.pc.iceGatheringState === 'complete') {
        this.analyzeGatheredCandidates();
      }
    };

    this.pc.oniceconnectionstatechange = () => {
      console.log('ICE connection state:', this.pc.iceConnectionState);
      
      switch (this.pc.iceConnectionState) {
        case 'checking':
          console.log('Checking connectivity...');
          break;
        case 'connected':
          console.log('Connected! Getting stats...');
          this.logSelectedCandidatePair();
          break;
        case 'failed':
          console.error('ICE failed - check TURN configuration');
          break;
        case 'disconnected':
          console.warn('ICE disconnected - may recover');
          break;
      }
    };
  }

  private analyzeGatheredCandidates() {
    const types = {
      host: 0,
      srflx: 0,
      prflx: 0,
      relay: 0
    };

    this.candidatesGathered.forEach(c => {
      const info = analyzeIceCandidate(c);
      types[info.type as keyof typeof types]++;
    });

    console.log('Gathered candidates:', types);

    if (types.srflx === 0) {
      console.warn('No STUN candidates - may have STUN connectivity issues');
    }
    if (types.relay === 0) {
      console.warn('No TURN candidates - TURN may be misconfigured');
    }
  }

  private async logSelectedCandidatePair() {
    const stats = await this.pc.getStats();
    
    stats.forEach(report => {
      if (report.type === 'candidate-pair' && report.state === 'succeeded') {
        console.log('Selected candidate pair:');
        console.log('  Local:', report.localCandidateId);
        console.log('  Remote:', report.remoteCandidateId);
        console.log('  RTT:', report.currentRoundTripTime);
      }
    });
  }
}

// ════════════════════════════════════════════════════════════════
// TURN SERVER TESTING
// ════════════════════════════════════════════════════════════════

async function testTurnServer(turnConfig: RTCIceServer): Promise<boolean> {
  return new Promise((resolve) => {
    const pc = new RTCPeerConnection({
      iceServers: [turnConfig],
      iceTransportPolicy: 'relay'  // Force TURN
    });

    let hasRelay = false;
    const timeout = setTimeout(() => {
      pc.close();
      resolve(hasRelay);
    }, 10000);

    pc.onicecandidate = (event) => {
      if (event.candidate) {
        const info = analyzeIceCandidate(event.candidate);
        if (info.type === 'relay') {
          hasRelay = true;
          clearTimeout(timeout);
          pc.close();
          resolve(true);
        }
      }
    };

    // Trigger ICE gathering
    pc.createDataChannel('test');
    pc.createOffer()
      .then(offer => pc.setLocalDescription(offer))
      .catch(() => {
        clearTimeout(timeout);
        resolve(false);
      });
  });
}

// Usage
const turnWorks = await testTurnServer({
  urls: 'turn:turn.example.com:3478',
  username: 'user',
  credential: 'pass'
});
console.log('TURN server working:', turnWorks);
```

---

## 6. Error Handling & Reconnection

```typescript
// ════════════════════════════════════════════════════════════════
// ROBUST CONNECTION MANAGEMENT
// ════════════════════════════════════════════════════════════════

class RobustPeerConnection {
  private pc: RTCPeerConnection | null = null;
  private localStream: MediaStream | null = null;
  private signalingClient: SignalingClient;
  private peerId: string;
  private config: RTCConfiguration;
  private reconnectAttempts = 0;
  private maxReconnectAttempts = 5;

  constructor(
    signalingClient: SignalingClient,
    peerId: string,
    config: RTCConfiguration
  ) {
    this.signalingClient = signalingClient;
    this.peerId = peerId;
    this.config = config;
  }

  async connect(localStream: MediaStream): Promise<void> {
    this.localStream = localStream;
    await this.createConnection();
  }

  private async createConnection(): Promise<void> {
    this.pc = new RTCPeerConnection(this.config);
    this.setupEventHandlers();

    // Add local tracks
    if (this.localStream) {
      this.localStream.getTracks().forEach(track => {
        this.pc!.addTrack(track, this.localStream!);
      });
    }
  }

  private setupEventHandlers(): void {
    if (!this.pc) return;

    this.pc.onicecandidate = (event) => {
      if (event.candidate) {
        this.signalingClient.sendIceCandidate(this.peerId, event.candidate.toJSON());
      }
    };

    this.pc.ontrack = (event) => {
      console.log('Remote track received');
      this.onRemoteStream?.(event.streams[0]);
    };

    this.pc.onconnectionstatechange = () => {
      console.log('Connection state:', this.pc?.connectionState);
      
      switch (this.pc?.connectionState) {
        case 'connected':
          this.reconnectAttempts = 0;
          this.onConnected?.();
          break;
        case 'disconnected':
          this.handleDisconnected();
          break;
        case 'failed':
          this.handleFailed();
          break;
        case 'closed':
          this.onClosed?.();
          break;
      }
    };

    this.pc.oniceconnectionstatechange = () => {
      console.log('ICE state:', this.pc?.iceConnectionState);
      
      if (this.pc?.iceConnectionState === 'failed') {
        this.attemptIceRestart();
      }
    };
  }

  private handleDisconnected(): void {
    console.log('Connection disconnected, waiting for recovery...');
    
    // Wait a bit for automatic recovery
    setTimeout(() => {
      if (this.pc?.connectionState === 'disconnected') {
        console.log('Still disconnected, attempting ICE restart');
        this.attemptIceRestart();
      }
    }, 3000);
  }

  private async handleFailed(): Promise<void> {
    console.log('Connection failed');
    
    if (this.reconnectAttempts < this.maxReconnectAttempts) {
      this.reconnectAttempts++;
      console.log(`Reconnect attempt ${this.reconnectAttempts}/${this.maxReconnectAttempts}`);
      await this.reconnect();
    } else {
      console.error('Max reconnection attempts reached');
      this.onFailed?.();
    }
  }

  private async attemptIceRestart(): Promise<void> {
    if (!this.pc) return;

    try {
      console.log('Attempting ICE restart');
      const offer = await this.pc.createOffer({ iceRestart: true });
      await this.pc.setLocalDescription(offer);
      this.signalingClient.sendOffer(this.peerId, offer);
    } catch (error) {
      console.error('ICE restart failed:', error);
      await this.reconnect();
    }
  }

  private async reconnect(): Promise<void> {
    console.log('Reconnecting...');
    
    // Close existing connection
    this.pc?.close();
    
    // Wait before reconnecting
    await new Promise(resolve => 
      setTimeout(resolve, 1000 * Math.pow(2, this.reconnectAttempts - 1))
    );
    
    // Create new connection
    await this.createConnection();
    
    // Initiate new offer
    const offer = await this.pc!.createOffer();
    await this.pc!.setLocalDescription(offer);
    this.signalingClient.sendOffer(this.peerId, offer);
  }

  // Public methods
  async handleOffer(offer: RTCSessionDescriptionInit): Promise<void> {
    if (!this.pc) await this.createConnection();
    
    await this.pc!.setRemoteDescription(new RTCSessionDescription(offer));
    const answer = await this.pc!.createAnswer();
    await this.pc!.setLocalDescription(answer);
    this.signalingClient.sendAnswer(this.peerId, answer);
  }

  async handleAnswer(answer: RTCSessionDescriptionInit): Promise<void> {
    await this.pc?.setRemoteDescription(new RTCSessionDescription(answer));
  }

  async handleIceCandidate(candidate: RTCIceCandidateInit): Promise<void> {
    try {
      await this.pc?.addIceCandidate(new RTCIceCandidate(candidate));
    } catch (error) {
      console.error('Error adding ICE candidate:', error);
    }
  }

  close(): void {
    this.pc?.close();
    this.pc = null;
  }

  // Callbacks
  onRemoteStream: ((stream: MediaStream) => void) | null = null;
  onConnected: (() => void) | null = null;
  onFailed: (() => void) | null = null;
  onClosed: (() => void) | null = null;
}

// ════════════════════════════════════════════════════════════════
// CONNECTION QUALITY MONITORING
// ════════════════════════════════════════════════════════════════

class ConnectionQualityMonitor {
  private pc: RTCPeerConnection;
  private intervalId: NodeJS.Timeout | null = null;

  constructor(peerConnection: RTCPeerConnection) {
    this.pc = peerConnection;
  }

  start(intervalMs = 2000) {
    this.intervalId = setInterval(() => this.checkQuality(), intervalMs);
  }

  stop() {
    if (this.intervalId) {
      clearInterval(this.intervalId);
      this.intervalId = null;
    }
  }

  private async checkQuality() {
    const stats = await this.pc.getStats();
    
    let quality = {
      rtt: 0,
      jitter: 0,
      packetsLost: 0,
      packetsReceived: 0,
      bytesReceived: 0,
      frameRate: 0,
      resolution: { width: 0, height: 0 }
    };

    stats.forEach(report => {
      if (report.type === 'candidate-pair' && report.state === 'succeeded') {
        quality.rtt = report.currentRoundTripTime * 1000; // Convert to ms
      }
      
      if (report.type === 'inbound-rtp' && report.kind === 'video') {
        quality.jitter = report.jitter || 0;
        quality.packetsLost = report.packetsLost || 0;
        quality.packetsReceived = report.packetsReceived || 0;
        quality.bytesReceived = report.bytesReceived || 0;
        quality.frameRate = report.framesPerSecond || 0;
        
        if (report.frameWidth && report.frameHeight) {
          quality.resolution = {
            width: report.frameWidth,
            height: report.frameHeight
          };
        }
      }
    });

    const score = this.calculateQualityScore(quality);
    this.onQualityUpdate?.(quality, score);
  }

  private calculateQualityScore(quality: any): 'excellent' | 'good' | 'fair' | 'poor' {
    const lossRate = quality.packetsLost / (quality.packetsLost + quality.packetsReceived) || 0;

    if (quality.rtt < 100 && lossRate < 0.01) return 'excellent';
    if (quality.rtt < 200 && lossRate < 0.05) return 'good';
    if (quality.rtt < 400 && lossRate < 0.1) return 'fair';
    return 'poor';
  }

  onQualityUpdate: ((quality: any, score: string) => void) | null = null;
}
```

---

## 7. Multi-Party Calls

```typescript
// ════════════════════════════════════════════════════════════════
// MULTI-PARTY CALL ARCHITECTURE
// ════════════════════════════════════════════════════════════════

/*
MESH (Full Mesh):
- Each peer connects to every other peer
- Good for 2-4 participants
- CPU/bandwidth scales as N*(N-1)/2

      P1
     /  \
    /    \
   P2────P3
   
SFU (Selective Forwarding Unit):
- Each peer connects only to server
- Server forwards selected streams
- Better for 4-50 participants

   P1   P2   P3
    \   |   /
     \  |  /
      [SFU]
     /  |  \
    /   |   \
   P4   P5   P6

MCU (Multipoint Control Unit):
- Server decodes and re-encodes
- Single stream to each participant
- Highest server cost
*/

// ════════════════════════════════════════════════════════════════
// MESH IMPLEMENTATION
// ════════════════════════════════════════════════════════════════

class MeshRoom {
  private localStream: MediaStream | null = null;
  private peers: Map<string, RobustPeerConnection> = new Map();
  private signalingClient: SignalingClient;
  private config: RTCConfiguration;

  public onRemoteStream: ((peerId: string, stream: MediaStream) => void) | null = null;
  public onPeerDisconnected: ((peerId: string) => void) | null = null;

  constructor(signalingClient: SignalingClient, config: RTCConfiguration) {
    this.signalingClient = signalingClient;
    this.config = config;
    this.setupSignalingHandlers();
  }

  private setupSignalingHandlers() {
    this.signalingClient.onPeerJoined = async (peerId) => {
      console.log('Peer joined:', peerId);
      // Create connection and send offer (we are the polite peer)
      await this.createPeerConnection(peerId, true);
    };

    this.signalingClient.onPeerLeft = (peerId) => {
      console.log('Peer left:', peerId);
      this.removePeer(peerId);
    };

    this.signalingClient.onOffer = async (from, sdp) => {
      let peer = this.peers.get(from);
      if (!peer) {
        peer = await this.createPeerConnection(from, false);
      }
      await peer.handleOffer(sdp);
    };

    this.signalingClient.onAnswer = async (from, sdp) => {
      const peer = this.peers.get(from);
      if (peer) {
        await peer.handleAnswer(sdp);
      }
    };

    this.signalingClient.onIceCandidate = async (from, candidate) => {
      const peer = this.peers.get(from);
      if (peer) {
        await peer.handleIceCandidate(candidate);
      }
    };
  }

  async join(roomId: string, localStream: MediaStream): Promise<void> {
    this.localStream = localStream;
    this.signalingClient.joinRoom(roomId);
  }

  private async createPeerConnection(
    peerId: string,
    initiator: boolean
  ): Promise<RobustPeerConnection> {
    const peer = new RobustPeerConnection(
      this.signalingClient,
      peerId,
      this.config
    );

    peer.onRemoteStream = (stream) => {
      this.onRemoteStream?.(peerId, stream);
    };

    peer.onFailed = () => {
      this.onPeerDisconnected?.(peerId);
    };

    this.peers.set(peerId, peer);

    await peer.connect(this.localStream!);

    if (initiator) {
      // Send offer
      // (handled inside RobustPeerConnection)
    }

    return peer;
  }

  private removePeer(peerId: string) {
    const peer = this.peers.get(peerId);
    if (peer) {
      peer.close();
      this.peers.delete(peerId);
      this.onPeerDisconnected?.(peerId);
    }
  }

  leave() {
    this.peers.forEach((peer, id) => {
      peer.close();
    });
    this.peers.clear();
    this.signalingClient.disconnect();
  }

  getParticipantCount(): number {
    return this.peers.size + 1; // +1 for local
  }
}

// ════════════════════════════════════════════════════════════════
// USAGE
// ════════════════════════════════════════════════════════════════

async function startVideoCall(roomId: string) {
  const signaling = new SignalingClient();
  await signaling.connect('wss://signaling.example.com');

  const room = new MeshRoom(signaling, iceConfig);

  // Get local media
  const localStream = await navigator.mediaDevices.getUserMedia({
    video: true,
    audio: true
  });

  // Display local video
  const localVideo = document.getElementById('localVideo') as HTMLVideoElement;
  localVideo.srcObject = localStream;

  // Handle remote streams
  room.onRemoteStream = (peerId, stream) => {
    const videoElement = createVideoElement(peerId);
    videoElement.srcObject = stream;
  };

  room.onPeerDisconnected = (peerId) => {
    removeVideoElement(peerId);
  };

  // Join room
  await room.join(roomId, localStream);

  return room;
}
```

---

## 8. Screen Sharing

```typescript
// ════════════════════════════════════════════════════════════════
// SCREEN SHARING
// ════════════════════════════════════════════════════════════════

async function startScreenShare(): Promise<MediaStream> {
  const screenStream = await navigator.mediaDevices.getDisplayMedia({
    video: {
      cursor: 'always',  // 'always' | 'motion' | 'never'
      displaySurface: 'monitor'  // 'monitor' | 'window' | 'browser'
    },
    audio: true  // System audio (not always supported)
  });

  return screenStream;
}

// Replace video track with screen share
async function switchToScreenShare(peerConnection: RTCPeerConnection): Promise<() => void> {
  // Save original video track
  const videoSender = peerConnection.getSenders().find(s => 
    s.track?.kind === 'video'
  );
  const originalTrack = videoSender?.track;

  // Get screen share
  const screenStream = await startScreenShare();
  const screenTrack = screenStream.getVideoTracks()[0];

  // Replace track
  await videoSender?.replaceTrack(screenTrack);

  // Handle when user stops sharing via browser UI
  screenTrack.onended = async () => {
    if (originalTrack) {
      await videoSender?.replaceTrack(originalTrack);
    }
  };

  // Return function to stop screen share
  return async () => {
    screenTrack.stop();
    if (originalTrack) {
      await videoSender?.replaceTrack(originalTrack);
    }
  };
}

// Screen share with audio
async function screenShareWithAudio(
  peerConnection: RTCPeerConnection
): Promise<void> {
  const screenStream = await navigator.mediaDevices.getDisplayMedia({
    video: true,
    audio: true
  });

  // Check if we got audio
  const audioTracks = screenStream.getAudioTracks();
  if (audioTracks.length > 0) {
    // Mix with microphone audio
    const micStream = await navigator.mediaDevices.getUserMedia({ audio: true });
    const audioContext = new AudioContext();
    
    const screenAudio = audioContext.createMediaStreamSource(screenStream);
    const micAudio = audioContext.createMediaStreamSource(micStream);
    
    const destination = audioContext.createMediaStreamDestination();
    
    screenAudio.connect(destination);
    micAudio.connect(destination);
    
    // Replace audio track with mixed audio
    const audioSender = peerConnection.getSenders().find(s => 
      s.track?.kind === 'audio'
    );
    
    const [mixedTrack] = destination.stream.getAudioTracks();
    await audioSender?.replaceTrack(mixedTrack);
  }

  // Replace video track
  const videoSender = peerConnection.getSenders().find(s => 
    s.track?.kind === 'video'
  );
  
  const [screenVideoTrack] = screenStream.getVideoTracks();
  await videoSender?.replaceTrack(screenVideoTrack);
}
```

---

## 9. Common Pitfalls

```typescript
// ════════════════════════════════════════════════════════════════
// WEBRTC PITFALLS
// ════════════════════════════════════════════════════════════════

// ❌ PITFALL 1: Not including TURN servers
// Problem: ~10-20% of connections fail

// Bad
const pc = new RTCPeerConnection({
  iceServers: [{ urls: 'stun:stun.l.google.com:19302' }]
});

// Good
const pc = new RTCPeerConnection({
  iceServers: [
    { urls: 'stun:stun.l.google.com:19302' },
    { 
      urls: 'turn:turn.example.com:3478',
      username: 'user',
      credential: 'pass'
    }
  ]
});

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 2: Adding ICE candidates before remote description
// Problem: Candidates silently fail

// Bad
signalingClient.onIceCandidate = async (candidate) => {
  await pc.addIceCandidate(candidate);  // May fail if no remote description
};

// Good
let pendingCandidates: RTCIceCandidateInit[] = [];
let hasRemoteDescription = false;

signalingClient.onIceCandidate = async (candidate) => {
  if (hasRemoteDescription) {
    await pc.addIceCandidate(candidate);
  } else {
    pendingCandidates.push(candidate);
  }
};

async function handleRemoteDescription(sdp: RTCSessionDescriptionInit) {
  await pc.setRemoteDescription(sdp);
  hasRemoteDescription = true;
  
  // Add pending candidates
  for (const candidate of pendingCandidates) {
    await pc.addIceCandidate(candidate);
  }
  pendingCandidates = [];
}

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 3: Not handling negotiationneeded correctly
// Problem: Race conditions in renegotiation

// Bad
pc.onnegotiationneeded = async () => {
  const offer = await pc.createOffer();
  await pc.setLocalDescription(offer);
  sendOffer(offer);
};

// Good - "Perfect Negotiation" pattern
let makingOffer = false;
let ignoreOffer = false;
const polite = true;  // Set based on who initiated

pc.onnegotiationneeded = async () => {
  try {
    makingOffer = true;
    await pc.setLocalDescription();
    sendOffer(pc.localDescription);
  } finally {
    makingOffer = false;
  }
};

async function handleOffer(offer: RTCSessionDescriptionInit) {
  const offerCollision = makingOffer || pc.signalingState !== 'stable';
  ignoreOffer = !polite && offerCollision;
  
  if (ignoreOffer) return;
  
  await pc.setRemoteDescription(offer);
  await pc.setLocalDescription();
  sendAnswer(pc.localDescription);
}

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 4: Not stopping tracks on cleanup
// Problem: Camera/mic stays active

// Bad
function endCall() {
  pc.close();
}

// Good
function endCall() {
  // Stop all local tracks
  localStream.getTracks().forEach(track => track.stop());
  
  // Close peer connection
  pc.close();
}

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 5: Not handling getUserMedia errors
// Problem: App crashes or shows nothing

// Bad
const stream = await navigator.mediaDevices.getUserMedia({ video: true });

// Good
try {
  const stream = await navigator.mediaDevices.getUserMedia({ video: true });
  // Success
} catch (error) {
  if (error.name === 'NotAllowedError') {
    showMessage('Please allow camera access');
  } else if (error.name === 'NotFoundError') {
    showMessage('No camera found');
  } else if (error.name === 'NotReadableError') {
    showMessage('Camera is in use by another application');
  } else {
    showMessage('Could not access camera');
  }
}

// ════════════════════════════════════════════════════════════════
// ❌ PITFALL 6: CORS issues with TURN server
// Problem: TURN credentials fail

// TURN server must support the credential type
// Most use time-limited credentials:
function getTurnCredentials(): { username: string; credential: string } {
  const timestamp = Math.floor(Date.now() / 1000) + 86400; // 24 hours
  const username = `${timestamp}:${userId}`;
  const credential = generateHMAC(username, turnSecret);
  return { username, credential };
}
```

---

## 10. Interview Questions

### Basic Questions

**Q: "What is WebRTC?"**
> "WebRTC is a browser API for real-time peer-to-peer audio, video, and data communication. It handles media capture, encoding, NAT traversal (via ICE/STUN/TURN), and encrypted transmission (DTLS-SRTP). The API includes RTCPeerConnection for connections, MediaStream for media, and RTCDataChannel for arbitrary data."

**Q: "What is a signaling server?"**
> "A signaling server exchanges connection metadata (SDP offers/answers, ICE candidates) between peers before they're connected. WebRTC doesn't specify the signaling protocol - you can use WebSocket, HTTP, or any transport. Once the WebRTC connection is established, signaling can stop."

**Q: "What's the difference between STUN and TURN?"**
> "STUN discovers your public IP address for NAT traversal - it's lightweight and free. TURN relays all traffic through a server when direct P2P fails (symmetric NAT, firewalls). TURN is expensive (bandwidth) but necessary for ~10-20% of connections."

### Intermediate Questions

**Q: "Explain the WebRTC connection flow."**
> "1. Both peers create RTCPeerConnection with ICE servers. 2. Caller creates offer (SDP), sets as local description. 3. Offer sent via signaling to callee. 4. Callee sets offer as remote description, creates answer. 5. Answer sent back, caller sets as remote description. 6. Throughout, both exchange ICE candidates as discovered. 7. ICE selects best path (or TURN fallback). 8. DTLS handshake for encryption, then media flows."

**Q: "What is SDP?"**
> "Session Description Protocol describes media session parameters - codecs supported, encryption keys, media types, network information. In WebRTC, SDP is exchanged as offer/answer during signaling to negotiate compatible parameters. The format is text-based with lines like `m=video`, `a=rtpmap`."

**Q: "What are ICE candidates and types?"**
> "ICE candidates are potential connection paths. Types: **host** (local IP - works on same network), **srflx** (server-reflexive - public IP via STUN), **relay** (TURN server relay). ICE tests these paths to find one that works, preferring direct connections over relay."

### Advanced Questions

**Q: "How would you architect a video conferencing app for 10+ participants?"**
> "Mesh topology doesn't scale past 4-5 peers. Use an SFU (Selective Forwarding Unit): each peer sends one stream to the SFU, which forwards selectively to others. This means each client has 1 upload + N-1 downloads instead of N-1 of each. SFUs can implement simulcast (multiple quality levels) for bandwidth adaptation. For 50+ participants, consider MCU or hybrid approaches."

**Q: "How do you handle connection failures?"**
> "Multiple layers: 1) Include TURN servers for NAT traversal failures. 2) Monitor `connectionstatechange` and `iceconnectionstatechange`. 3) On `disconnected`, wait briefly for recovery. 4) On `failed`, attempt ICE restart with `restartIce()`. 5) If restart fails, create new connection. 6) Ultimate fallback to server-mediated solution if WebRTC completely fails."

**Q: "Explain the Perfect Negotiation pattern."**
> "Perfect Negotiation handles race conditions when both peers try to negotiate simultaneously. It designates one peer as 'polite' (will yield) and one as 'impolite'. When offer collision occurs (both creating offers), the polite peer rolls back and accepts the impolite peer's offer. This ensures exactly one offer/answer exchange completes without deadlock."

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────┐
│                  WEBRTC IMPLEMENTATION CHECKLIST                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ICE CONFIGURATION:                                             │
│  □ Multiple STUN servers                                       │
│  □ TURN server (UDP, TCP, TLS on 443)                          │
│  □ Test TURN credentials before calls                          │
│                                                                 │
│  SIGNALING:                                                     │
│  □ Exchange SDP offers/answers                                 │
│  □ Exchange ICE candidates (trickle)                           │
│  □ Handle reconnection scenarios                               │
│                                                                 │
│  CONNECTION:                                                    │
│  □ Monitor connectionstatechange                               │
│  □ Monitor iceconnectionstatechange                            │
│  □ Implement ICE restart on failure                            │
│  □ Queue candidates until remote description set               │
│                                                                 │
│  MEDIA:                                                         │
│  □ Handle getUserMedia errors gracefully                       │
│  □ Stop tracks on cleanup                                      │
│  □ Support track replacement (camera switch, screen share)     │
│                                                                 │
│  QUALITY:                                                       │
│  □ Monitor getStats() for quality metrics                      │
│  □ Implement adaptive bitrate                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

CONNECTION FLOW:
┌────────────────────────────────────────────────────────────────┐
│ 1. Create RTCPeerConnection with ICE servers                   │
│ 2. Add local media tracks                                      │
│ 3. Create offer, setLocalDescription                           │
│ 4. Send offer via signaling                                    │
│ 5. Receive answer, setRemoteDescription                        │
│ 6. Exchange ICE candidates (both directions)                   │
│ 7. ICE finds path (or falls back to TURN)                      │
│ 8. DTLS handshake, connection established                      │
└────────────────────────────────────────────────────────────────┘

ICE CANDIDATE TYPES:
┌────────────────────────────────────────────────────────────────┐
│ host   - Local IP (same network only)                          │
│ srflx  - Server reflexive (via STUN)                           │
│ prflx  - Peer reflexive (discovered during checks)             │
│ relay  - TURN relay (fallback)                                 │
└────────────────────────────────────────────────────────────────┘

MULTI-PARTY TOPOLOGY:
┌────────────────────────────────────────────────────────────────┐
│ Mesh - P2P all-to-all (2-4 participants)                       │
│ SFU  - Server forwards streams (4-50 participants)             │
│ MCU  - Server mixes streams (any size, high server cost)       │
└────────────────────────────────────────────────────────────────┘
```

---

*Last updated: February 2026*

