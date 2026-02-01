# RabbitMQ

> Message broker for event-driven architecture.

## Overview

| Aspect | Details |
|--------|---------|
| **What** | AMQP message broker |
| **Why** | Async processing, event-driven, decoupling |
| **Package** | `@golevelup/nestjs-rabbitmq` |
| **Location** | `apps/api/src/rabbitmq/` |

## Setup

```typescript
// app.module.ts
import { RabbitMQModule } from '@golevelup/nestjs-rabbitmq';

@Module({
  imports: [
    RabbitMQModule.forRootAsync(RabbitMQModule, {
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        uri: config.get('AMQP_URL'),
        exchanges: [
          { name: 'domain.events', type: 'topic' },
          { name: 'notifications', type: 'topic' },
          { name: 'dead-letter', type: 'direct' },
          { name: 'audit', type: 'topic' },
        ],
        connectionInitOptions: { wait: false },
        enableControllerDiscovery: true,
      }),
    }),
  ],
})
export class AppModule {}
```

## Exchange Topology

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        RabbitMQ Exchanges                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  domain.events (topic)                                                   │
│  ├── medication.logged.*     ──► WebSocket consumer                     │
│  ├── emergency.alert.*       ──► WebSocket + Push consumer              │
│  ├── appointment.*           ──► WebSocket consumer                     │
│  └── shift.*                 ──► WebSocket consumer                     │
│                                                                          │
│  notifications (topic)                                                   │
│  ├── push.*                  ──► Push notification worker               │
│  ├── email.*                 ──► Email worker                           │
│  └── sms.*                   ──► SMS worker (optional)                  │
│                                                                          │
│  audit (topic)                                                           │
│  └── *.audit                 ──► Audit log consumer                     │
│                                                                          │
│  dead-letter (direct)                                                    │
│  └── dlq                     ──► DLQ processor                          │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

## Publishing Events

```typescript
// rabbitmq/publisher.service.ts
import { AmqpConnection } from '@golevelup/nestjs-rabbitmq';

@Injectable()
export class EventPublisher {
  constructor(private readonly amqpConnection: AmqpConnection) {}

  // Publish medication event
  async publishMedicationLogged(data: MedicationLoggedEvent) {
    await this.amqpConnection.publish(
      'domain.events',
      `medication.logged.${data.status.toLowerCase()}`,
      data,
      { persistent: true }
    );
  }

  // Publish emergency alert
  async publishEmergencyAlert(data: EmergencyAlertEvent) {
    await this.amqpConnection.publish(
      'domain.events',
      `emergency.alert.${data.type.toLowerCase()}`,
      data,
      { 
        persistent: true,
        priority: 10, // High priority
      }
    );
  }

  // Publish push notification
  async publishPushNotification(data: PushNotificationPayload) {
    await this.amqpConnection.publish(
      'notifications',
      'push.send',
      data,
      { persistent: true }
    );
  }

  // Publish to audit log
  async publishAuditEvent(data: AuditLogEvent) {
    await this.amqpConnection.publish(
      'audit',
      `${data.resourceType.toLowerCase()}.audit`,
      data,
      { persistent: true }
    );
  }
}
```

## Consuming Events

```typescript
// consumers/websocket.consumer.ts
import { RabbitSubscribe } from '@golevelup/nestjs-rabbitmq';

@Injectable()
export class WebSocketConsumer {
  constructor(private readonly gateway: AppGateway) {}

  @RabbitSubscribe({
    exchange: 'domain.events',
    routingKey: 'medication.logged.*',
    queue: 'websocket-medication-updates',
    queueOptions: {
      durable: true,
      deadLetterExchange: 'dead-letter',
    },
  })
  async handleMedicationLogged(msg: MedicationLoggedEvent) {
    this.gateway.emitToFamily(msg.familyId, 'medication.logged', msg);
  }

  @RabbitSubscribe({
    exchange: 'domain.events',
    routingKey: 'emergency.alert.*',
    queue: 'websocket-emergency-alerts',
    queueOptions: {
      durable: true,
      deadLetterExchange: 'dead-letter',
    },
  })
  async handleEmergencyAlert(msg: EmergencyAlertEvent) {
    this.gateway.emitToFamily(msg.familyId, 'emergency.alert.created', msg);
  }
}
```

## Audit Consumer

```typescript
// consumers/audit.consumer.ts
@Injectable()
export class AuditConsumer {
  constructor(private readonly prisma: PrismaService) {}

  @RabbitSubscribe({
    exchange: 'audit',
    routingKey: '*.audit',
    queue: 'audit-log-queue',
    queueOptions: { durable: true },
  })
  async handleAuditEvent(msg: AuditLogEvent) {
    await this.prisma.auditLog.create({
      data: {
        action: msg.action,
        userId: msg.userId,
        resourceType: msg.resourceType,
        resourceId: msg.resourceId,
        oldValue: msg.oldValue,
        newValue: msg.newValue,
        ipAddress: msg.ipAddress,
        userAgent: msg.userAgent,
      },
    });
  }
}
```

## Dead Letter Queue Handler

```typescript
// consumers/dlq.consumer.ts
@Injectable()
export class DeadLetterConsumer {
  private readonly logger = new Logger(DeadLetterConsumer.name);

  @RabbitSubscribe({
    exchange: 'dead-letter',
    routingKey: 'dlq',
    queue: 'dead-letter-queue',
  })
  async handleDeadLetter(msg: any, amqpMsg: ConsumeMessage) {
    const originalQueue = amqpMsg.properties.headers?.['x-death']?.[0]?.queue;
    
    this.logger.error({
      message: 'Dead letter received',
      originalQueue,
      payload: msg,
      error: amqpMsg.properties.headers?.['x-death']?.[0]?.reason,
    });

    // Store for investigation
    await this.prisma.deadLetter.create({
      data: {
        originalQueue,
        payload: JSON.stringify(msg),
        error: amqpMsg.properties.headers?.['x-death']?.[0]?.reason,
      },
    });

    // Optional: Send Slack alert
    if (process.env.SLACK_DLQ_WEBHOOK) {
      await this.sendSlackAlert(msg, originalQueue);
    }
  }
}
```

## Integration with Services

```typescript
// medications.service.ts
@Injectable()
export class MedicationsService {
  constructor(
    private prisma: PrismaService,
    private eventPublisher: EventPublisher,
  ) {}

  async logMedication(medicationId: string, userId: string, dto: LogMedicationDto) {
    const log = await this.prisma.medicationLog.create({
      data: { ... },
      include: {
        medication: {
          include: { careRecipient: true },
        },
      },
    });

    // Publish event for async processing
    await this.eventPublisher.publishMedicationLogged({
      medicationId: log.medicationId,
      medicationName: log.medication.name,
      status: log.status,
      familyId: log.medication.careRecipient.familyId,
      loggedBy: userId,
      timestamp: log.createdAt.toISOString(),
    });

    // Publish audit event
    await this.eventPublisher.publishAuditEvent({
      action: 'MEDICATION_LOGGED',
      userId,
      resourceType: 'MedicationLog',
      resourceId: log.id,
    });

    return log;
  }
}
```

## Event Types

```typescript
// events/types.ts
interface MedicationLoggedEvent {
  medicationId: string;
  medicationName: string;
  status: 'GIVEN' | 'SKIPPED' | 'MISSED';
  familyId: string;
  loggedBy: string;
  timestamp: string;
}

interface EmergencyAlertEvent {
  alertId: string;
  type: 'FALL' | 'MEDICAL' | 'MISSING' | 'HOSPITALIZATION' | 'OTHER';
  severity: 'LOW' | 'MEDIUM' | 'HIGH' | 'CRITICAL';
  careRecipientId: string;
  familyId: string;
  createdBy: string;
  timestamp: string;
}

interface PushNotificationPayload {
  userIds: string[];
  title: string;
  body: string;
  url?: string;
  data?: Record<string, any>;
}

interface AuditLogEvent {
  action: string;
  userId: string;
  resourceType: string;
  resourceId: string;
  oldValue?: any;
  newValue?: any;
  ipAddress?: string;
  userAgent?: string;
}
```

## Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `AMQP_URL` | RabbitMQ connection URL | `amqp://guest:guest@localhost:5672` |
| `AMQP_HOST` | RabbitMQ host | `localhost` |
| `AMQP_USER` | Username | `guest` |
| `AMQP_PASSWORD` | Password | `guest` |
| `AMQP_TLS` | Enable TLS | `true` (cloud) |

## Local Development

```bash
# RabbitMQ is included in docker-compose.yml
docker-compose up rabbitmq -d

# Access management UI
# URL: http://localhost:15672
# User: guest
# Pass: guest
```

## Cloud Setup (CloudAMQP)

1. Create account at [cloudamqp.com](https://cloudamqp.com)
2. Create instance (Little Lemur = free tier)
3. Copy AMQP URL
4. Set `AMQP_URL` environment variable

## Troubleshooting

### Connection Issues
- Check `AMQP_URL` format
- Verify network connectivity
- Check TLS settings for cloud

### Messages Not Consumed
- Verify exchange/queue bindings
- Check routing key pattern
- Ensure consumer is running

### Messages Going to DLQ
- Check consumer error handling
- Verify message format matches consumer expectation

---

*See also: [BullMQ](../workers/bullmq.md), [Socket.io](socket-io.md)*


