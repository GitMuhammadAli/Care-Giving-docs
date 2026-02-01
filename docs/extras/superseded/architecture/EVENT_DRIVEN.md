# Event-Driven Architecture

CareCircle uses RabbitMQ for reliable, scalable event-driven communication. This document explains the architecture, patterns, and how to use the event system.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              CareCircle Event Flow                          │
│                                                                             │
│   ┌─────────────┐                                                           │
│   │   Service   │                                                           │
│   │ (e.g. Meds) │                                                           │
│   └──────┬──────┘                                                           │
│          │                                                                  │
│          │ 1. Create event in outbox                                        │
│          │    (same DB transaction)                                         │
│          ▼                                                                  │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                    PostgreSQL (Outbox Table)                        │   │
│   │  ┌────────────────────────────────────────────────────────────────┐ │   │
│   │  │ id │ event_type │ payload │ status │ created_at │ processed_at │ │   │
│   │  └────────────────────────────────────────────────────────────────┘ │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│          │                                                                  │
│          │ 2. Outbox Processor polls every 5s                               │
│          ▼                                                                  │
│   ┌──────────────────┐                                                      │
│   │ Outbox Processor │                                                      │
│   │ (Cron Job)       │                                                      │
│   └────────┬─────────┘                                                      │
│            │                                                                │
│            │ 3. Publish to RabbitMQ                                         │
│            ▼                                                                │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                         RabbitMQ Broker                             │   │
│   │                                                                     │   │
│   │  ┌─────────────────────────────────────────────────────────────┐    │   │
│   │  │              Exchanges                                      │    │   │
│   │  │  • domain.events (topic) - Domain events                    │    │   │
│   │  │  • notifications (direct) - Push, email, SMS                │    │   │
│   │  │  • audit (fanout) - Audit logging                          │    │   │
│   │  │  • dlx (direct) - Dead letter exchange                     │    │   │
│   │  └─────────────────────────────────────────────────────────────┘    │   │
│   │                            │                                        │   │
│   │  ┌─────────────────────────┼─────────────────────────────────┐      │   │
│   │  │                    Queues                                  │      │   │
│   │  │  • websocket.updates - Real-time broadcasts               │      │   │
│   │  │  • notifications.push - Push notifications                │      │   │
│   │  │  • notifications.email - Email sending                    │      │   │
│   │  │  • audit.log - Audit logging                              │      │   │
│   │  │  • dlq.* - Dead letter queues                             │      │   │
│   │  └───────────────────────────────────────────────────────────┘      │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                            │                                                │
│            ┌───────────────┼───────────────┬───────────────┐                │
│            │               │               │               │                │
│            ▼               ▼               ▼               ▼                │
│   ┌────────────┐   ┌────────────┐   ┌────────────┐   ┌────────────┐         │
│   │ WebSocket  │   │   Push     │   │   Email    │   │   Audit    │         │
│   │ Consumer   │   │ Consumer   │   │ Consumer   │   │ Consumer   │         │
│   └─────┬──────┘   └────────────┘   └────────────┘   └────────────┘         │
│         │                                                                   │
│         │ 4. Emit to EventEmitter                                           │
│         ▼                                                                   │
│   ┌─────────────┐                                                           │
│   │  WebSocket  │───────► Browser clients                                   │
│   │   Gateway   │                                                           │
│   └─────────────┘                                                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Outbox Pattern

The **Outbox Pattern** ensures reliable event delivery by storing events in the same database transaction as domain changes.

### Why Outbox Pattern?

**Problem without outbox:**
```
1. Save medication log to DB ✅
2. Publish event to RabbitMQ ❌ (broker down)
3. Event is lost forever!
```

**With outbox pattern:**
```
1. Start transaction
2. Save medication log to DB ✅
3. Save event to outbox table ✅
4. Commit transaction ✅
5. (Later) Outbox processor publishes to RabbitMQ ✅
6. Mark event as processed ✅
```

### Outbox Table Structure

```sql
CREATE TABLE event_outbox (
  id            UUID PRIMARY KEY,
  event_type    VARCHAR NOT NULL,     -- e.g., 'medication.logged'
  exchange      VARCHAR NOT NULL,     -- Target RabbitMQ exchange
  routing_key   VARCHAR NOT NULL,     -- Routing key
  payload       JSONB NOT NULL,       -- Event data (CloudEvents format)
  aggregate_type VARCHAR NOT NULL,    -- e.g., 'Medication'
  aggregate_id  VARCHAR NOT NULL,     -- e.g., medication UUID
  status        ENUM('PENDING', 'PROCESSING', 'PROCESSED', 'FAILED'),
  retry_count   INT DEFAULT 0,
  last_error    TEXT,
  processed_at  TIMESTAMPTZ,
  correlation_id VARCHAR,             -- For distributed tracing
  caused_by     VARCHAR,              -- User who triggered the event
  created_at    TIMESTAMPTZ DEFAULT NOW()
);
```

## Event Types

### Domain Events (Topic Exchange)

| Event | Routing Key | Description |
|-------|-------------|-------------|
| Medication Logged | `medication.logged` | When a dose is marked as given/skipped |
| Medication Due | `medication.due` | Reminder that medication is due |
| Medication Refill | `medication.refill_needed` | Supply is running low |
| Appointment Created | `appointment.created` | New appointment added |
| Appointment Reminder | `appointment.reminder` | Upcoming appointment reminder |
| Emergency Alert | `emergency.alert.created` | Emergency button pressed |
| Emergency Resolved | `emergency.alert.resolved` | Emergency was resolved |
| Shift Started | `shift.started` | Caregiver checked in |
| Shift Ended | `shift.ended` | Caregiver checked out |
| Timeline Entry | `timeline.entry.created` | New health log entry |

### Notification Events (Direct Exchange)

| Event | Routing Key | Description |
|-------|-------------|-------------|
| Push | `notify.push` | Send push notification |
| Email | `notify.email` | Send email |
| SMS | `notify.sms` | Send SMS (emergency only) |

## Usage

### Publishing Domain Events (Recommended)

```typescript
import { EventPublisherService } from '../events';
import { ROUTING_KEYS } from '../events/events.constants';

@Injectable()
export class MedicationsService {
  constructor(private eventPublisher: EventPublisherService) {}

  async logMedication(medicationId: string, dto: LogDto, user: User) {
    // Use transaction for atomicity
    return this.dataSource.transaction(async (manager) => {
      // 1. Perform domain operation
      const log = await manager.save(MedicationLog, { ... });

      // 2. Publish event (stored in outbox)
      await this.eventPublisher.publish(
        ROUTING_KEYS.MEDICATION_LOGGED,
        {
          medicationId,
          status: dto.status,
          loggedById: user.id,
          // ... other data
        },
        {
          aggregateType: 'Medication',
          aggregateId: medicationId,
        },
        {
          useOutbox: true,  // Default, recommended
          causedBy: user.id,
          familyId: family.id,
          careRecipientId: recipient.id,
        },
      );

      return log;
    });
  }
}
```

### Publishing Non-Critical Events (Direct)

```typescript
// For non-critical events that can be lost if broker is unavailable
await this.eventPublisher.publishDirect(
  ROUTING_KEYS.MEDICATION_DUE,
  { ... },
  {
    familyId: family.id,
    careRecipientId: recipient.id,
  },
);
```

### Publishing Notifications

```typescript
// Push notification
await this.eventPublisher.publishNotification('push', {
  userId: 'user-123',
  title: 'Medication Due',
  body: 'Metformin 500mg is due now',
  url: '/medications',
  priority: 'high',
});

// Email notification
await this.eventPublisher.publishNotification('email', {
  to: 'user@example.com',
  subject: 'Appointment Reminder',
  template: 'appointment-reminder',
  context: { ... },
  priority: 'normal',
});
```

## Consuming Events

### Creating a Consumer

```typescript
import { RabbitSubscribe, Nack } from '@golevelup/nestjs-rabbitmq';
import { EXCHANGES, QUEUES } from '../events/events.constants';

@Injectable()
export class MyConsumer {
  @RabbitSubscribe({
    exchange: EXCHANGES.DOMAIN_EVENTS,
    routingKey: 'medication.*',  // Listen to all medication events
    queue: QUEUES.MY_QUEUE,
    queueOptions: { durable: true },
  })
  async handleMedicationEvent(event: BaseEvent): Promise<void | Nack> {
    try {
      // Process the event
      console.log('Received:', event.type, event.data);
    } catch (error) {
      // Return Nack to reject and requeue
      return new Nack(true);
    }
  }
}
```

## CloudEvents Format

All events follow the [CloudEvents](https://cloudevents.io/) specification:

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "type": "medication.logged",
  "source": "carecircle-api",
  "timestamp": "2024-01-15T10:30:00Z",
  "specVersion": "1.0",
  "correlationId": "req-123-456",
  "causedBy": "user-789",
  "familyId": "family-abc",
  "careRecipientId": "recipient-xyz",
  "data": {
    "medicationId": "med-123",
    "status": "GIVEN",
    "loggedBy": "Sarah Thompson"
  }
}
```

## Dead Letter Handling

Failed messages are routed to Dead Letter Queues (DLQ) for manual inspection:

- `carecircle.dlq.notifications` - Failed notifications
- `carecircle.dlq.processing` - Failed processing events

### Viewing Dead Letters

Access the RabbitMQ Management UI at `http://localhost:15672` (dev) with:
- Username: `carecircle`
- Password: `carecircle_dev`

## Monitoring

### Outbox Statistics

The outbox processor logs statistics every hour:
```
Outbox stats - Pending: 0, Processing: 0, Failed: 2
```

### Health Checks

Monitor these metrics:
- Outbox pending count (should be low)
- Outbox failed count (should be zero)
- RabbitMQ queue depths
- Consumer lag

## Best Practices

### DO ✅

1. **Use outbox for domain events** - Critical events that must not be lost
2. **Use transactions** - Wrap domain changes + outbox write in a transaction
3. **Include correlation IDs** - For distributed tracing
4. **Handle idempotency** - Consumers should handle duplicate messages
5. **Log event processing** - For debugging and audit

### DON'T ❌

1. **Don't skip outbox for critical events** - Emergency alerts MUST use outbox
2. **Don't block on event publishing** - Events are async
3. **Don't put large payloads in events** - Keep events small, fetch data if needed
4. **Don't rely on event ordering** - Design consumers to handle out-of-order

## Local Development

### Starting RabbitMQ

```bash
docker-compose up -d rabbitmq
```

### Management UI

- URL: http://localhost:15672
- Username: carecircle
- Password: carecircle_dev

### CloudAMQP (Free Tier)

For cloud-hosted RabbitMQ:
1. Sign up at https://www.cloudamqp.com
2. Create a free "Little Lemur" instance
3. Copy the AMQP URL to `RABBITMQ_URL` in `.env`

## Production Considerations

### Scaling

- Run multiple API instances (consumers auto-balance)
- Use consumer prefetch count to limit concurrent processing
- Consider dedicated worker nodes for heavy consumers

### High Availability

- Use RabbitMQ cluster (3+ nodes)
- Enable quorum queues for critical queues
- Configure mirroring policies

### Security

- Use TLS connections (amqps://)
- Rotate credentials regularly
- Limit queue/exchange permissions per service

