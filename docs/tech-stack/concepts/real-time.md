# Real-Time Communication

> Understanding how CareCircle delivers instant updates.

---

## The Mental Model

Think of real-time communication like **different ways to get news**:

- **Polling** = Checking your mailbox every hour
- **Long Polling** = Telling the mailman to wait until there's mail
- **Server-Sent Events** = A one-way radio broadcast
- **WebSocket** = A phone call (two-way, open connection)

Each has trade-offs in complexity, resource usage, and capabilities.

---

## Why Real-Time Matters for CareCircle

### Critical Real-Time Scenarios

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    WHY REAL-TIME FOR CAREGIVING?                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  EMERGENCY ALERTS                                                            │
│  ───────────────                                                            │
│  Grandma pressed emergency button                                            │
│  ALL family members need to know IMMEDIATELY                                │
│  Delay = potential harm                                                      │
│                                                                              │
│  MEDICATION LOGGING                                                          │
│  ─────────────────                                                          │
│  Caregiver logs that Mom took her pills                                     │
│  Other caregivers should see this NOW                                       │
│  Prevents double-dosing or missed doses                                     │
│                                                                              │
│  SHIFT HANDOFFS                                                              │
│  ─────────────                                                              │
│  Night caregiver checking in                                                │
│  Day caregiver should be notified                                           │
│  Ensures coverage continuity                                                │
│                                                                              │
│  CHAT/COORDINATION                                                           │
│  ────────────────                                                            │
│  Family discussing care plan                                                 │
│  Messages must be instant                                                    │
│  Standard chat expectation                                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Communication Patterns Compared

### 1. Traditional Polling

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          POLLING                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  CLIENT              SERVER              DATABASE                            │
│     │                  │                    │                               │
│     │─── GET /updates ─►│                    │                               │
│     │◄── [] (empty) ────│                    │                               │
│     │                  │                    │                               │
│     │  (wait 5 sec)    │                    │                               │
│     │                  │                    │                               │
│     │─── GET /updates ─►│                    │                               │
│     │◄── [] (empty) ────│                    │                               │
│     │                  │                    │                               │
│     │  (wait 5 sec)    │                    │                               │
│     │                  │                    │                               │
│     │─── GET /updates ─►│───── query ───────►│                               │
│     │◄── [{new data}] ──│◄──── result ───────│                               │
│                                                                              │
│  PROS:                              CONS:                                   │
│  • Simple to implement             • Wastes bandwidth (empty responses)     │
│  • Works everywhere                • Delay between data and delivery        │
│  • Stateless server                • Database hammered with queries         │
│                                                                              │
│  USE WHEN:                                                                  │
│  • Updates are infrequent                                                   │
│  • 5-30 second delay is acceptable                                         │
│  • Simplicity is priority                                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2. Long Polling

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        LONG POLLING                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  CLIENT              SERVER              DATABASE                            │
│     │                  │                    │                               │
│     │─── GET /updates ─►│ (holds connection) │                               │
│     │                  │                    │                               │
│     │                  │  (waits for data)  │                               │
│     │                  │                    │                               │
│     │                  │◄── new data event ─│                               │
│     │◄── [{new data}] ──│                    │                               │
│     │                  │                    │                               │
│     │─── GET /updates ─►│ (new connection)  │                               │
│     │                  │                    │                               │
│                                                                              │
│  PROS:                              CONS:                                   │
│  • Nearly real-time               • Keeps connections open (resource heavy) │
│  • Less traffic than polling      • Complex timeout handling               │
│  • Works through firewalls        • One-way only                           │
│                                                                              │
│  USE WHEN:                                                                  │
│  • Need near-real-time but WebSocket not available                          │
│  • One-way updates are sufficient                                           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3. Server-Sent Events (SSE)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    SERVER-SENT EVENTS (SSE)                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  CLIENT              SERVER                                                  │
│     │                  │                                                     │
│     │─── EventSource ──►│  (opens persistent connection)                     │
│     │                  │                                                     │
│     │◄── event: update │                                                     │
│     │    data: {...}   │                                                     │
│     │                  │                                                     │
│     │◄── event: update │  (server pushes whenever ready)                    │
│     │    data: {...}   │                                                     │
│     │                  │                                                     │
│     │◄── event: update │                                                     │
│     │    data: {...}   │                                                     │
│                                                                              │
│  PROS:                              CONS:                                   │
│  • Built into browsers (EventSource API)                                    │
│  • Auto-reconnection              • One-way only (server → client)          │
│  • Simpler than WebSocket         • Limited browser connections (~6)        │
│  • Text-only (easy to debug)                                                │
│                                                                              │
│  USE WHEN:                                                                  │
│  • Updates flow one direction (server → client)                            │
│  • Live feeds, notifications, activity streams                             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 4. WebSocket (What CareCircle Uses)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          WEBSOCKET                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  CLIENT              SERVER                                                  │
│     │                  │                                                     │
│     │═══ WS handshake ══►│  (upgrades HTTP to WS)                            │
│     │◄══════════════════│                                                    │
│     │                  │                                                     │
│     │══► send message ══►│  (client can send anytime)                       │
│     │                  │                                                     │
│     │◄══ push event ════│  (server can send anytime)                        │
│     │                  │                                                     │
│     │══► send message ══►│                                                   │
│     │◄══ push response ═│                                                    │
│     │                  │                                                     │
│     ║    (bidirectional, full-duplex, persistent)    ║                      │
│                                                                              │
│  PROS:                              CONS:                                   │
│  • True bidirectional              • More complex to implement             │
│  • Low latency                     • Stateful (server tracks connections)  │
│  • Efficient (no HTTP overhead)    • Load balancing complexity             │
│  • Binary support                  • Needs reconnection handling           │
│                                                                              │
│  USE WHEN:                                                                  │
│  • Bidirectional communication                                              │
│  • Lowest possible latency required                                         │
│  • High-frequency updates                                                   │
│  • Chat, gaming, collaboration                                              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## CareCircle's WebSocket Architecture

### Socket.io Choice

```
WHY SOCKET.IO (not raw WebSocket)?
──────────────────────────────────

1. AUTOMATIC FALLBACK
   WebSocket → Long Polling → Polling
   Works even in restrictive networks

2. RECONNECTION
   Auto-reconnect with exponential backoff
   No manual implementation needed

3. ROOMS & NAMESPACES
   Easy to group connections
   "All family members" = one room

4. ACKNOWLEDGMENTS
   Confirm message delivery
   Retry if not acknowledged

5. BINARY SUPPORT
   Send files, images directly

6. WIDESPREAD SUPPORT
   Works on web, React Native, Node.js
```

### Connection Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      SOCKET.IO ARCHITECTURE                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                         ┌─────────────────┐                                 │
│                         │   NestJS API    │                                 │
│                         │   (Socket.io    │                                 │
│                         │    Gateway)     │                                 │
│                         └────────┬────────┘                                 │
│                                  │                                          │
│                    ┌─────────────┼─────────────┐                           │
│                    │             │             │                           │
│                    ▼             ▼             ▼                           │
│              ┌──────────┐ ┌──────────┐ ┌──────────┐                       │
│              │  Room:   │ │  Room:   │ │  Room:   │                       │
│              │ family:1 │ │ family:2 │ │ family:3 │                       │
│              └────┬─────┘ └────┬─────┘ └────┬─────┘                       │
│                   │            │            │                              │
│          ┌────────┼────────┐   │     ┌──────┼──────┐                      │
│          │        │        │   │     │      │      │                      │
│          ▼        ▼        ▼   ▼     ▼      ▼      ▼                      │
│       User A   User B   User C    User D  User E  User F                  │
│       (web)    (web)    (mobile)  (web)   (mobile)(web)                   │
│                                                                              │
│  ROOMS:                                                                     │
│  • Each family has a room                                                   │
│  • Users join their family's room on connect                                │
│  • Events broadcast to room reach all family members                        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Event Types

```typescript
// SERVER → CLIENT Events
socket.to(familyRoom).emit('medication:logged', {
  medicationId: '123',
  careRecipientId: '456',
  loggedBy: 'User A',
  timestamp: new Date(),
});

socket.to(familyRoom).emit('emergency:alert', {
  alertId: '789',
  careRecipientId: '456',
  type: 'fall_detected',
  severity: 'high',
});

socket.to(familyRoom).emit('shift:started', {
  shiftId: '101',
  caregiverId: 'user-123',
  startTime: new Date(),
});

// CLIENT → SERVER Events
socket.emit('join:family', { familyId: '123' });
socket.emit('typing:start', { channelId: 'channel-456' });
socket.emit('presence:update', { status: 'online' });
```

---

## Handling Real-Time State

### Optimistic Updates

```
PROBLEM:
────────
User clicks "Log Medication"
Wait for server → Update UI
User sees delay

SOLUTION: OPTIMISTIC UPDATE
───────────────────────────
User clicks "Log Medication"
Immediately show as logged (optimistic)
Send to server in background
If server confirms → great!
If server fails → rollback UI, show error
```

```typescript
// React implementation with TanStack Query
const logMedication = useMutation({
  mutationFn: (data) => api.logMedication(data),
  
  // Optimistically update cache
  onMutate: async (newLog) => {
    await queryClient.cancelQueries(['medications']);
    
    const previousMeds = queryClient.getQueryData(['medications']);
    
    // Immediately update UI
    queryClient.setQueryData(['medications'], (old) => 
      old.map(med => 
        med.id === newLog.medicationId 
          ? { ...med, lastLogged: new Date() }
          : med
      )
    );
    
    return { previousMeds };
  },
  
  // Rollback on error
  onError: (err, newLog, context) => {
    queryClient.setQueryData(['medications'], context.previousMeds);
    toast.error('Failed to log medication');
  },
  
  // Sync with server
  onSettled: () => {
    queryClient.invalidateQueries(['medications']);
  },
});
```

### Syncing WebSocket with React Query

```typescript
// When WebSocket receives update, sync with cache
useEffect(() => {
  socket.on('medication:logged', (data) => {
    // Update cache with server data
    queryClient.setQueryData(['medications'], (old) =>
      old?.map(med =>
        med.id === data.medicationId
          ? { ...med, lastLogged: data.timestamp }
          : med
      )
    );
  });

  return () => socket.off('medication:logged');
}, [socket, queryClient]);
```

---

## Connection Management

### Reconnection Strategy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      RECONNECTION STRATEGY                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  DISCONNECT DETECTED                                                         │
│       │                                                                      │
│       ▼                                                                      │
│  Attempt 1: Wait 1 second, reconnect                                        │
│       │                                                                      │
│       │──── Failed ────►                                                    │
│       │                                                                      │
│       ▼                                                                      │
│  Attempt 2: Wait 2 seconds, reconnect                                       │
│       │                                                                      │
│       │──── Failed ────►                                                    │
│       │                                                                      │
│       ▼                                                                      │
│  Attempt 3: Wait 4 seconds, reconnect                                       │
│       │                                                                      │
│       │──── Failed ────►                                                    │
│       │                                                                      │
│       ▼                                                                      │
│  (Continue with exponential backoff, max 30 seconds)                        │
│       │                                                                      │
│       │──── Max attempts (10) reached ────►  Show "Connection lost" UI     │
│       │                                                                      │
│       ▼                                                                      │
│  CONNECTED!                                                                  │
│       │                                                                      │
│       ▼                                                                      │
│  Re-authenticate (send JWT)                                                  │
│       │                                                                      │
│       ▼                                                                      │
│  Re-join rooms (family rooms)                                               │
│       │                                                                      │
│       ▼                                                                      │
│  Fetch missed updates (since lastEventId)                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Connection State UI

```typescript
// Connection status hook
function useConnectionStatus() {
  const [status, setStatus] = useState<'connected' | 'connecting' | 'disconnected'>('connecting');

  useEffect(() => {
    socket.on('connect', () => setStatus('connected'));
    socket.on('disconnect', () => setStatus('disconnected'));
    socket.on('reconnecting', () => setStatus('connecting'));

    return () => {
      socket.off('connect');
      socket.off('disconnect');
      socket.off('reconnecting');
    };
  }, []);

  return status;
}

// Usage in UI
function ConnectionBanner() {
  const status = useConnectionStatus();

  if (status === 'connected') return null;

  return (
    <div className={status === 'connecting' ? 'bg-yellow-500' : 'bg-red-500'}>
      {status === 'connecting' ? 'Reconnecting...' : 'Connection lost'}
    </div>
  );
}
```

---

## Security Considerations

### Authentication over WebSocket

```typescript
// Server: Verify JWT on connection
@WebSocketGateway()
export class EventsGateway implements OnGatewayConnection {
  async handleConnection(client: Socket) {
    try {
      const token = client.handshake.auth.token;
      const user = await this.authService.verifyToken(token);
      
      // Attach user to socket
      client.data.user = user;
      
      // Join user's family rooms
      const families = await this.getFamilies(user.id);
      families.forEach(f => client.join(`family:${f.id}`));
      
    } catch (error) {
      // Invalid token - disconnect
      client.disconnect();
    }
  }
}

// Client: Send token on connect
const socket = io(SOCKET_URL, {
  auth: {
    token: accessToken,
  },
});
```

### Authorization for Events

```typescript
// Don't trust the client - verify server-side
@SubscribeMessage('family:send-message')
async handleMessage(
  @ConnectedSocket() client: Socket,
  @MessageBody() data: { familyId: string; message: string }
) {
  const user = client.data.user;
  
  // Verify user is actually in this family
  const isMember = await this.familyService.isMember(user.id, data.familyId);
  if (!isMember) {
    throw new WsException('Not a family member');
  }
  
  // Now safe to broadcast
  this.server.to(`family:${data.familyId}`).emit('message', {
    from: user.id,
    message: data.message,
  });
}
```

---

## Quick Reference

### When to Use What

| Scenario | Solution | Why |
|----------|----------|-----|
| Emergency alerts | WebSocket | Critical, instant delivery |
| Medication logs | WebSocket | Family coordination |
| Chat messages | WebSocket (Stream Chat) | Standard chat behavior |
| Dashboard stats | Polling (1 min) | Acceptable delay |
| Notification badge | WebSocket | Should be instant |
| Activity feed | Polling (30 sec) or SSE | One-way, not critical |

### Socket.io Cheatsheet

```typescript
// Server-side
io.to('room').emit('event', data);          // Send to room
socket.join('room');                         // Add to room
socket.leave('room');                        // Remove from room
socket.broadcast.emit('event', data);        // All except sender
socket.emit('event', data, callback);        // With acknowledgment

// Client-side
socket.emit('event', data);                  // Send to server
socket.on('event', (data) => {});            // Listen for event
socket.off('event');                         // Remove listener
socket.connect();                            // Connect
socket.disconnect();                         // Disconnect
```

---

*Next: [Socket.io Deep Dive](../backend/socket-io.md) | [Stream Chat Integration](../frontend/stream-chat.md)*


