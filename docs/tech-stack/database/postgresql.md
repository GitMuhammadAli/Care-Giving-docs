# PostgreSQL

> Primary relational database for CareCircle.

## Overview

| Aspect | Details |
|--------|---------|
| **What** | Open-source relational database |
| **Why** | ACID compliance, JSON support, reliability |
| **Version** | 16 |
| **Provider** | Neon (cloud) / Docker (local) |

## Local Setup

```yaml
# docker-compose.yml
services:
  postgres:
    image: postgres:16-alpine
    container_name: carecircle-postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: 1234
      POSTGRES_DB: carecircle
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
```

## Connection

```env
# Local
DATABASE_URL=postgresql://postgres:1234@localhost:5432/carecircle

# Neon (Cloud)
DATABASE_URL=postgresql://user:pass@ep-xxx.us-east-1.aws.neon.tech/neondb?sslmode=require
```

## Schema Overview

### Core Tables

| Table | Description | Rows (typical) |
|-------|-------------|----------------|
| `User` | User accounts | 100s |
| `Family` | Family units | 10s |
| `FamilyMember` | User-family relationships | 100s |
| `CareRecipient` | Care recipients | 10s |
| `Medication` | Medications | 100s |
| `MedicationLog` | Medication logs | 1000s |
| `Appointment` | Appointments | 100s |
| `CaregiverShift` | Shifts | 100s |
| `Document` | Documents | 100s |
| `TimelineEntry` | Timeline entries | 1000s |
| `EmergencyAlert` | Alerts | 10s |
| `Notification` | Notifications | 1000s |

### Relationships

```
User
  └── FamilyMember (many) ──► Family
  └── Session (many)
  └── PushToken (many)
  └── Notification (many)

Family
  └── FamilyMember (many)
  └── FamilyInvitation (many)
  └── CareRecipient (many)
  └── Document (many)

CareRecipient
  └── Doctor (many)
  └── EmergencyContact (many)
  └── Medication (many)
  └── Appointment (many)
  └── CaregiverShift (many)
  └── TimelineEntry (many)
  └── EmergencyAlert (many)

Medication
  └── MedicationLog (many)
```

## Key Indexes

```sql
-- User lookups
CREATE UNIQUE INDEX "User_email_key" ON "User"("email");

-- Family member lookups
CREATE INDEX "FamilyMember_userId_idx" ON "FamilyMember"("userId");
CREATE INDEX "FamilyMember_familyId_idx" ON "FamilyMember"("familyId");
CREATE UNIQUE INDEX "FamilyMember_userId_familyId_key" ON "FamilyMember"("userId", "familyId");

-- Medication queries
CREATE INDEX "Medication_careRecipientId_idx" ON "Medication"("careRecipientId");
CREATE INDEX "Medication_isActive_idx" ON "Medication"("isActive");

-- Log queries (high volume)
CREATE INDEX "MedicationLog_medicationId_idx" ON "MedicationLog"("medicationId");
CREATE INDEX "MedicationLog_createdAt_idx" ON "MedicationLog"("createdAt");

-- Appointment queries
CREATE INDEX "Appointment_careRecipientId_idx" ON "Appointment"("careRecipientId");
CREATE INDEX "Appointment_dateTime_idx" ON "Appointment"("dateTime");
CREATE INDEX "Appointment_status_idx" ON "Appointment"("status");

-- Notification queries
CREATE INDEX "Notification_userId_idx" ON "Notification"("userId");
CREATE INDEX "Notification_isRead_idx" ON "Notification"("isRead");
```

## Query Examples

### Find User with Families
```sql
SELECT 
  u.id, u.email, u.fullName,
  json_agg(json_build_object(
    'familyId', f.id,
    'familyName', f.name,
    'role', fm.role
  )) as families
FROM "User" u
LEFT JOIN "FamilyMember" fm ON u.id = fm."userId"
LEFT JOIN "Family" f ON fm."familyId" = f.id
WHERE u.id = $1
GROUP BY u.id;
```

### Get Today's Medications
```sql
SELECT m.*, 
  COALESCE(
    (SELECT json_agg(ml.*) 
     FROM "MedicationLog" ml 
     WHERE ml."medicationId" = m.id 
     AND ml."scheduledTime"::date = CURRENT_DATE),
    '[]'
  ) as logs
FROM "Medication" m
WHERE m."careRecipientId" = $1
  AND m."isActive" = true
ORDER BY m.name;
```

### Get Unread Notification Count
```sql
SELECT COUNT(*) 
FROM "Notification"
WHERE "userId" = $1 AND "isRead" = false;
```

## Neon Setup

1. **Create Account**: [neon.tech](https://neon.tech)
2. **Create Project**: Choose region (us-east-1 recommended)
3. **Get Connection String**: Dashboard → Connection Details
4. **Configure SSL**: Add `?sslmode=require` to URL

### Neon Features

- **Serverless**: Scales to zero when idle
- **Branching**: Create database branches for testing
- **PITR**: Point-in-time recovery
- **Free Tier**: 0.5 GB storage, 3GB transfer

### Connection Pooling

Neon uses PgBouncer for connection pooling:

```env
# Pooled connection (recommended for serverless)
DATABASE_URL=postgresql://user:pass@ep-xxx-pooler.us-east-1.aws.neon.tech/neondb?sslmode=require

# Direct connection (for migrations)
DIRECT_URL=postgresql://user:pass@ep-xxx.us-east-1.aws.neon.tech/neondb?sslmode=require
```

## Backup & Recovery

### Local Backup
```bash
# Backup
pg_dump -U postgres carecircle > backup.sql

# Restore
psql -U postgres carecircle < backup.sql
```

### Neon Backup
- Automatic daily backups
- Point-in-time recovery (PITR)
- Branch for testing restores

## Performance Monitoring

### Slow Query Logging
```sql
-- Enable slow query logging (>100ms)
ALTER SYSTEM SET log_min_duration_statement = 100;
SELECT pg_reload_conf();
```

### Connection Monitoring
```sql
SELECT 
  count(*) as connections,
  state,
  usename
FROM pg_stat_activity
GROUP BY state, usename;
```

## Common Commands

```bash
# Connect via psql
psql -h localhost -U postgres -d carecircle

# List tables
\dt

# Describe table
\d "User"

# Show running queries
SELECT * FROM pg_stat_activity WHERE state = 'active';

# Kill query
SELECT pg_cancel_backend(pid);
```

## Troubleshooting

### Connection Issues
- Check `DATABASE_URL` format
- Verify SSL mode for cloud
- Check firewall/network

### Slow Queries
- Add indexes for WHERE clauses
- Use EXPLAIN ANALYZE
- Consider query optimization

### Storage Issues
- Vacuum tables regularly
- Archive old data
- Monitor with `pg_stat_user_tables`

---

*See also: [Prisma](../backend/prisma.md), [Migrations](migrations.md)*


