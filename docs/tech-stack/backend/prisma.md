# Prisma ORM

> Next-generation Node.js and TypeScript ORM.

## Overview

| Aspect | Details |
|--------|---------|
| **What** | TypeScript ORM with type-safe queries |
| **Why** | Type safety, migrations, intuitive API |
| **Version** | 5.10+ |
| **Location** | `packages/database/` |

## Schema Location

```
packages/database/
├── prisma/
│   ├── schema.prisma       # Database schema
│   ├── migrations/         # Migration files
│   └── seed.ts            # Seed script
├── src/
│   └── index.ts           # Prisma client export
└── package.json
```

## Schema Overview

```prisma
// packages/database/prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id                String    @id @default(uuid())
  email             String    @unique
  password          String
  fullName          String
  avatarUrl         String?
  phone             String?
  timezone          String    @default("America/New_York")
  isEmailVerified   Boolean   @default(false)
  emailVerificationOtp String?
  otpExpiresAt      DateTime?
  onboardingComplete Boolean  @default(false)
  createdAt         DateTime  @default(now())
  updatedAt         DateTime  @updatedAt

  // Relations
  sessions          Session[]
  pushTokens        PushToken[]
  familyMemberships FamilyMember[]
  sentInvitations   FamilyInvitation[] @relation("InvitedBy")
  notifications     Notification[]
}

model Family {
  id          String   @id @default(uuid())
  name        String
  description String?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  // Relations
  members        FamilyMember[]
  invitations    FamilyInvitation[]
  careRecipients CareRecipient[]
  documents      Document[]
}

model Medication {
  id              String             @id @default(uuid())
  careRecipientId String
  careRecipient   CareRecipient      @relation(fields: [careRecipientId], references: [id], onDelete: Cascade)
  name            String
  genericName     String?
  dosage          String
  form            MedicationForm
  frequency       MedicationFrequency
  scheduledTimes  String[]
  instructions    String?
  prescribedBy    String?
  pharmacy        String?
  pharmacyPhone   String?
  currentSupply   Int?
  refillAt        Int?
  startDate       DateTime           @default(now())
  endDate         DateTime?
  isActive        Boolean            @default(true)
  createdAt       DateTime           @default(now())
  updatedAt       DateTime           @updatedAt

  // Relations
  logs MedicationLog[]
}
```

## Prisma Service (NestJS)

```typescript
// apps/api/src/prisma/prisma.service.ts
import { Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit, OnModuleDestroy {
  async onModuleInit() {
    await this.$connect();
  }

  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```

## Common Queries

### Find Operations
```typescript
// Find one
const user = await prisma.user.findUnique({
  where: { id: userId },
});

// Find one or throw
const user = await prisma.user.findUniqueOrThrow({
  where: { email },
});

// Find first
const medication = await prisma.medication.findFirst({
  where: { careRecipientId, isActive: true },
  orderBy: { createdAt: 'desc' },
});

// Find many
const medications = await prisma.medication.findMany({
  where: {
    careRecipientId,
    isActive: true,
  },
  include: {
    logs: {
      take: 10,
      orderBy: { createdAt: 'desc' },
    },
  },
  orderBy: { name: 'asc' },
});
```

### Create Operations
```typescript
// Create one
const medication = await prisma.medication.create({
  data: {
    name: 'Metformin',
    dosage: '500mg',
    form: 'TABLET',
    frequency: 'TWICE_DAILY',
    scheduledTimes: ['08:00', '20:00'],
    careRecipientId,
  },
});

// Create many
const logs = await prisma.medicationLog.createMany({
  data: medications.map(med => ({
    medicationId: med.id,
    status: 'PENDING',
    scheduledTime: new Date(),
  })),
});
```

### Update Operations
```typescript
// Update one
const updated = await prisma.medication.update({
  where: { id: medicationId },
  data: { currentSupply: { decrement: 1 } },
});

// Update many
await prisma.notification.updateMany({
  where: { userId, isRead: false },
  data: { isRead: true },
});

// Upsert
const token = await prisma.pushToken.upsert({
  where: { token },
  update: { lastUsedAt: new Date() },
  create: { userId, token, platform: 'WEB' },
});
```

### Delete Operations
```typescript
// Delete one
await prisma.medication.delete({
  where: { id: medicationId },
});

// Delete many
await prisma.session.deleteMany({
  where: {
    userId,
    expiresAt: { lt: new Date() },
  },
});
```

## Relations

### Include Relations
```typescript
const careRecipient = await prisma.careRecipient.findUnique({
  where: { id },
  include: {
    doctors: true,
    emergencyContacts: true,
    medications: {
      where: { isActive: true },
      include: {
        logs: {
          take: 5,
          orderBy: { createdAt: 'desc' },
        },
      },
    },
    appointments: {
      where: {
        dateTime: { gte: new Date() },
      },
      orderBy: { dateTime: 'asc' },
      take: 10,
    },
  },
});
```

### Select Specific Fields
```typescript
const users = await prisma.user.findMany({
  select: {
    id: true,
    email: true,
    fullName: true,
    // Don't select password
    familyMemberships: {
      select: {
        role: true,
        family: {
          select: { name: true },
        },
      },
    },
  },
});
```

## Transactions

### Sequential Transactions
```typescript
const [user, family] = await prisma.$transaction([
  prisma.user.create({ data: userData }),
  prisma.family.create({ data: familyData }),
]);
```

### Interactive Transactions
```typescript
await prisma.$transaction(async (tx) => {
  // Create family
  const family = await tx.family.create({
    data: { name: familyName },
  });

  // Add creator as admin
  await tx.familyMember.create({
    data: {
      familyId: family.id,
      userId: creatorId,
      role: 'ADMIN',
    },
  });

  // Create audit log
  await tx.auditLog.create({
    data: {
      action: 'FAMILY_CREATED',
      userId: creatorId,
      resourceType: 'Family',
      resourceId: family.id,
    },
  });

  return family;
});
```

## Common Commands

```bash
# Generate Prisma Client
pnpm --filter @carecircle/database db:generate

# Run migrations (development)
pnpm --filter @carecircle/database db:migrate

# Create migration
pnpm --filter @carecircle/database db:migrate:create

# Reset database
pnpm --filter @carecircle/database db:reset

# Seed database
pnpm --filter @carecircle/database db:seed

# Open Prisma Studio
pnpm --filter @carecircle/database db:studio

# Push schema changes (no migration)
pnpm --filter @carecircle/database db:push
```

## Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `DATABASE_URL` | PostgreSQL connection string | `postgresql://user:pass@host:5432/db` |

### Connection URL Format
```
postgresql://USER:PASSWORD@HOST:PORT/DATABASE?schema=public&sslmode=require
```

**Local:**
```
postgresql://postgres:1234@localhost:5432/carecircle
```

**Neon (Cloud):**
```
postgresql://user:pass@ep-xxx.us-east-1.aws.neon.tech/neondb?sslmode=require
```

## Troubleshooting

### Migration Issues
```bash
# Reset and re-run migrations
pnpm db:reset

# Force push schema (destructive)
pnpm db:push --force-reset
```

### Connection Issues
- Check `DATABASE_URL` format
- Verify SSL mode for cloud databases
- Check network/firewall settings

### Type Errors
```bash
# Regenerate Prisma Client
pnpm db:generate
```

---

*See also: [PostgreSQL](../database/postgresql.md), [Migrations](../database/migrations.md)*


