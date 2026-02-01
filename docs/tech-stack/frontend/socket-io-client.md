# Socket.io Client

> Real-time bidirectional event-based communication.

## Overview

| Aspect | Details |
|--------|---------|
| **What** | WebSocket client library |
| **Why** | Real-time updates, event-driven communication |
| **Version** | 4.x |
| **Location** | `apps/web/src/lib/socket.ts` |

## Setup

### Socket Configuration
```typescript
// lib/socket.ts
import { io, Socket } from 'socket.io-client';

let socket: Socket | null = null;

export function getSocket(): Socket {
  if (!socket) {
    socket = io(process.env.NEXT_PUBLIC_WS_URL!, {
      autoConnect: false,
      transports: ['websocket', 'polling'],
      withCredentials: true,
    });
  }
  return socket;
}

export function connectSocket(token: string) {
  const socket = getSocket();
  socket.auth = { token };
  socket.connect();
  return socket;
}

export function disconnectSocket() {
  if (socket) {
    socket.disconnect();
  }
}
```

### Socket Provider
```tsx
// providers/SocketProvider.tsx
'use client';

import { createContext, useContext, useEffect, useState } from 'react';
import { Socket } from 'socket.io-client';
import { getSocket, connectSocket, disconnectSocket } from '@/lib/socket';
import { useAuthStore } from '@/stores/authStore';

const SocketContext = createContext<Socket | null>(null);

export function SocketProvider({ children }: { children: React.ReactNode }) {
  const [socket, setSocket] = useState<Socket | null>(null);
  const { token, isAuthenticated } = useAuthStore();

  useEffect(() => {
    if (isAuthenticated && token) {
      const socketInstance = connectSocket(token);
      setSocket(socketInstance);

      socketInstance.on('connect', () => {
        console.log('Socket connected');
      });

      socketInstance.on('disconnect', () => {
        console.log('Socket disconnected');
      });

      return () => {
        disconnectSocket();
        setSocket(null);
      };
    }
  }, [isAuthenticated, token]);

  return (
    <SocketContext.Provider value={socket}>
      {children}
    </SocketContext.Provider>
  );
}

export const useSocket = () => useContext(SocketContext);
```

## Event Listeners

### Custom Hook
```typescript
// hooks/useSocketEvent.ts
import { useEffect } from 'react';
import { useSocket } from '@/providers/SocketProvider';

export function useSocketEvent<T>(
  event: string,
  handler: (data: T) => void
) {
  const socket = useSocket();

  useEffect(() => {
    if (!socket) return;

    socket.on(event, handler);

    return () => {
      socket.off(event, handler);
    };
  }, [socket, event, handler]);
}
```

### Usage
```tsx
function MedicationDashboard() {
  const queryClient = useQueryClient();

  // Listen for medication logs
  useSocketEvent('medication.logged', (data: MedicationLogEvent) => {
    // Invalidate query to refetch
    queryClient.invalidateQueries({ queryKey: ['medications', data.medicationId] });
    
    // Show notification
    toast.success(`${data.medicationName} logged as ${data.status}`);
  });

  // Listen for emergency alerts
  useSocketEvent('emergency.alert.created', (data: EmergencyAlert) => {
    // Show urgent notification
    toast.error(`EMERGENCY: ${data.type}`, { duration: Infinity });
  });

  return <div>...</div>;
}
```

## Emitting Events

### Basic Emit
```typescript
const socket = useSocket();

// Fire and forget
socket?.emit('medication:log', { medicationId, status: 'GIVEN' });

// With acknowledgment
socket?.emit('shift:checkin', { shiftId }, (response) => {
  if (response.success) {
    toast.success('Checked in!');
  } else {
    toast.error(response.error);
  }
});
```

### With Timeout
```typescript
socket?.timeout(5000).emit('event', data, (err, response) => {
  if (err) {
    // Timeout occurred
    toast.error('Server not responding');
  } else {
    // Handle response
  }
});
```

## Room Management

### Joining Family Room
```typescript
// Join family room on family selection
function FamilyDashboard({ familyId }: { familyId: string }) {
  const socket = useSocket();

  useEffect(() => {
    if (!socket || !familyId) return;

    // Join room
    socket.emit('family:join', { familyId });

    // Cleanup: leave room
    return () => {
      socket.emit('family:leave', { familyId });
    };
  }, [socket, familyId]);

  return <div>...</div>;
}
```

## Event Types

### Server → Client Events
```typescript
interface ServerToClientEvents {
  // Medication events
  'medication.logged': (data: {
    medicationId: string;
    medicationName: string;
    status: 'GIVEN' | 'SKIPPED' | 'MISSED';
    loggedBy: string;
    timestamp: string;
  }) => void;

  // Emergency events
  'emergency.alert.created': (data: EmergencyAlert) => void;
  'emergency.alert.acknowledged': (data: { alertId: string; acknowledgedBy: string }) => void;
  'emergency.alert.resolved': (data: { alertId: string }) => void;

  // Shift events
  'shift.checkin': (data: { shiftId: string; caregiverId: string }) => void;
  'shift.checkout': (data: { shiftId: string; handoffNotes?: string }) => void;

  // Appointment events
  'appointment.created': (data: Appointment) => void;
  'appointment.updated': (data: Appointment) => void;

  // Timeline events
  'timeline.entry.created': (data: TimelineEntry) => void;

  // Notification events
  'notification': (data: Notification) => void;
}
```

### Client → Server Events
```typescript
interface ClientToServerEvents {
  'family:join': (data: { familyId: string }) => void;
  'family:leave': (data: { familyId: string }) => void;
  'medication:log': (data: LogMedicationDto) => void;
  'shift:checkin': (data: { shiftId: string }, callback: (response: any) => void) => void;
}
```

## Connection Status

```tsx
function ConnectionStatus() {
  const socket = useSocket();
  const [isConnected, setIsConnected] = useState(false);

  useEffect(() => {
    if (!socket) return;

    setIsConnected(socket.connected);

    socket.on('connect', () => setIsConnected(true));
    socket.on('disconnect', () => setIsConnected(false));

    return () => {
      socket.off('connect');
      socket.off('disconnect');
    };
  }, [socket]);

  return (
    <div className="flex items-center gap-2">
      <div className={`w-2 h-2 rounded-full ${isConnected ? 'bg-green-500' : 'bg-red-500'}`} />
      <span className="text-sm">{isConnected ? 'Connected' : 'Disconnected'}</span>
    </div>
  );
}
```

## Reconnection Handling

```typescript
const socket = io(SOCKET_URL, {
  reconnection: true,
  reconnectionAttempts: 5,
  reconnectionDelay: 1000,
  reconnectionDelayMax: 5000,
});

socket.on('reconnect', (attemptNumber) => {
  console.log(`Reconnected after ${attemptNumber} attempts`);
  // Re-join rooms, refresh state
});

socket.on('reconnect_failed', () => {
  toast.error('Connection lost. Please refresh the page.');
});
```

## Troubleshooting

### Connection Issues
- Check `NEXT_PUBLIC_WS_URL` is correct
- Verify CORS settings on server
- Check if token is being passed

### Events Not Received
- Verify event name matches server
- Check if joined correct room
- Ensure socket is connected

### Memory Leaks
- Always clean up listeners in useEffect
- Disconnect socket on unmount

---

*See also: [Socket.io Server](../backend/socket-io.md), [TanStack Query](tanstack-query.md)*


