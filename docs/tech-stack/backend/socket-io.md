# Socket.io Server

> Real-time WebSocket communication for live updates.

## Overview

| Aspect | Details |
|--------|---------|
| **What** | WebSocket server with NestJS |
| **Why** | Real-time updates, bidirectional communication |
| **Version** | 4.x |
| **Location** | `apps/api/src/gateway/` |

## Gateway Setup

```typescript
// gateway/app.gateway.ts
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  OnGatewayConnection,
  OnGatewayDisconnect,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';

@WebSocketGateway({
  cors: {
    origin: process.env.FRONTEND_URL,
    credentials: true,
  },
  namespace: '/',
})
export class AppGateway implements OnGatewayConnection, OnGatewayDisconnect {
  @WebSocketServer()
  server: Server;

  private userSockets = new Map<string, Set<string>>();

  async handleConnection(client: Socket) {
    try {
      // Authenticate via token
      const token = client.handshake.auth.token;
      const user = await this.authService.validateToken(token);
      
      // Store user's socket
      client.data.userId = user.id;
      this.addUserSocket(user.id, client.id);

      // Auto-join family rooms
      const families = await this.familyService.getUserFamilyIds(user.id);
      families.forEach(familyId => {
        client.join(`family:${familyId}`);
      });

      this.logger.log(`Client connected: ${client.id} (User: ${user.id})`);
    } catch (error) {
      client.disconnect();
    }
  }

  handleDisconnect(client: Socket) {
    const userId = client.data.userId;
    if (userId) {
      this.removeUserSocket(userId, client.id);
    }
    this.logger.log(`Client disconnected: ${client.id}`);
  }
}
```

## Room Management

```typescript
// Join/Leave Family Rooms
@SubscribeMessage('family:join')
async handleJoinFamily(client: Socket, data: { familyId: string }) {
  // Verify membership
  const isMember = await this.familyService.isMember(
    client.data.userId,
    data.familyId
  );
  
  if (isMember) {
    client.join(`family:${data.familyId}`);
    return { success: true };
  }
  
  return { success: false, error: 'Not a family member' };
}

@SubscribeMessage('family:leave')
handleLeaveFamily(client: Socket, data: { familyId: string }) {
  client.leave(`family:${data.familyId}`);
  return { success: true };
}
```

## Broadcasting Events

```typescript
// Broadcast to family room
emitToFamily(familyId: string, event: string, data: any) {
  this.server.to(`family:${familyId}`).emit(event, data);
}

// Broadcast to specific user (all their devices)
emitToUser(userId: string, event: string, data: any) {
  const socketIds = this.userSockets.get(userId);
  if (socketIds) {
    socketIds.forEach(socketId => {
      this.server.to(socketId).emit(event, data);
    });
  }
}

// Broadcast to multiple users
emitToUsers(userIds: string[], event: string, data: any) {
  userIds.forEach(userId => this.emitToUser(userId, event, data));
}
```

## Event Types

### Medication Events
```typescript
// Emit when medication is logged
async onMedicationLogged(log: MedicationLog) {
  const medication = await this.getMedicationWithFamily(log.medicationId);
  
  this.emitToFamily(medication.careRecipient.familyId, 'medication.logged', {
    medicationId: medication.id,
    medicationName: medication.name,
    status: log.status,
    loggedBy: log.loggedById,
    timestamp: log.createdAt.toISOString(),
  });
}
```

### Emergency Events
```typescript
// Emit emergency alert
async onEmergencyAlertCreated(alert: EmergencyAlert) {
  const familyId = await this.getFamilyIdForAlert(alert.id);
  
  // Broadcast to all family members
  this.emitToFamily(familyId, 'emergency.alert.created', {
    id: alert.id,
    type: alert.type,
    severity: alert.severity,
    careRecipientId: alert.careRecipientId,
    createdAt: alert.createdAt.toISOString(),
  });
}

// Emit acknowledgment
async onEmergencyAcknowledged(alertId: string, userId: string) {
  const familyId = await this.getFamilyIdForAlert(alertId);
  
  this.emitToFamily(familyId, 'emergency.alert.acknowledged', {
    alertId,
    acknowledgedBy: userId,
    timestamp: new Date().toISOString(),
  });
}
```

### Shift Events
```typescript
// Emit shift check-in
async onShiftCheckIn(shift: CaregiverShift) {
  const familyId = await this.getFamilyIdForShift(shift.id);
  
  this.emitToFamily(familyId, 'shift.checkin', {
    shiftId: shift.id,
    caregiverId: shift.caregiverId,
    careRecipientId: shift.careRecipientId,
    checkedInAt: shift.actualStartTime?.toISOString(),
  });
}
```

## Integration with Services

```typescript
// medications.service.ts
@Injectable()
export class MedicationsService {
  constructor(
    private prisma: PrismaService,
    private gateway: AppGateway,
    private events: EventEmitter2,
  ) {}

  async logMedication(medicationId: string, userId: string, dto: LogMedicationDto) {
    const log = await this.prisma.medicationLog.create({
      data: {
        medicationId,
        loggedById: userId,
        status: dto.status,
        notes: dto.notes,
        scheduledTime: new Date(dto.scheduledTime),
      },
      include: {
        medication: {
          include: {
            careRecipient: true,
          },
        },
      },
    });

    // Emit real-time event
    this.gateway.emitToFamily(
      log.medication.careRecipient.familyId,
      'medication.logged',
      {
        medicationId,
        medicationName: log.medication.name,
        status: dto.status,
        loggedBy: userId,
      }
    );

    return log;
  }
}
```

## Event Emitter Integration

```typescript
// Using NestJS EventEmitter for decoupling
@OnEvent('medication.logged')
async handleMedicationLogged(payload: MedicationLoggedEvent) {
  this.gateway.emitToFamily(payload.familyId, 'medication.logged', payload);
}

@OnEvent('emergency.alert.created')
async handleEmergencyCreated(payload: EmergencyAlertEvent) {
  this.gateway.emitToFamily(payload.familyId, 'emergency.alert.created', payload);
}
```

## Client Message Handlers

```typescript
@SubscribeMessage('medication:log')
async handleMedicationLog(
  client: Socket,
  data: { medicationId: string; status: string }
) {
  try {
    const result = await this.medicationsService.logMedication(
      data.medicationId,
      client.data.userId,
      { status: data.status, scheduledTime: new Date() }
    );
    return { success: true, data: result };
  } catch (error) {
    return { success: false, error: error.message };
  }
}
```

## Connection Authentication

```typescript
// Using middleware for auth
@WebSocketGateway()
export class AppGateway implements OnGatewayInit {
  afterInit(server: Server) {
    server.use(async (socket, next) => {
      try {
        const token = socket.handshake.auth.token;
        if (!token) {
          return next(new Error('Authentication required'));
        }
        
        const payload = await this.jwtService.verifyAsync(token);
        socket.data.userId = payload.sub;
        next();
      } catch (error) {
        next(new Error('Invalid token'));
      }
    });
  }
}
```

## Module Configuration

```typescript
// gateway/gateway.module.ts
@Module({
  imports: [
    JwtModule,
    FamilyModule,
    AuthModule,
  ],
  providers: [AppGateway],
  exports: [AppGateway],
})
export class GatewayModule {}
```

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `NEXT_PUBLIC_WS_URL` | WebSocket URL | `http://localhost:4000` |

## Troubleshooting

### Connection Issues
- Check CORS configuration
- Verify token is passed in `auth`
- Check WebSocket transport is enabled

### Events Not Received
- Verify client joined correct room
- Check event name matches
- Ensure gateway is emitting to correct room

### Memory Leaks
- Clean up user sockets on disconnect
- Remove event listeners properly

---

*See also: [Socket.io Client](../frontend/socket-io-client.md), [NestJS](nestjs.md)*


