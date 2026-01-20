# Frontend Types Map

**Generated:** January 20, 2026
**Total Type Files:** 15
**Total Interfaces:** 45
**Total Type Aliases:** 4
**Total Enums (Union Types):** 3

---

## Summary Statistics

| Category               | Count  |
| ---------------------- | ------ |
| API Type Files         | 12     |
| Hook Type Files        | 13     |
| Context Files          | 1      |
| Utility Files          | 2      |
| **Total Interfaces**   | **45** |
| **Total Type Aliases** | **4**  |

---

## File: apps/web/src/lib/api/auth.ts

### Interface: UserFamily

| Property       | Type                                 | Optional | Notes |
| -------------- | ------------------------------------ | -------- | ----- |
| id             | `string`                             | No       |       |
| name           | `string`                             | No       |       |
| role           | `'ADMIN' \| 'CAREGIVER' \| 'VIEWER'` | No       |       |
| careRecipients | `CareRecipient[]`                    | Yes      |       |

### Interface: User

| Property              | Type           | Optional | Notes                                             |
| --------------------- | -------------- | -------- | ------------------------------------------------- |
| id                    | `string`       | No       |                                                   |
| email                 | `string`       | No       |                                                   |
| fullName              | `string`       | No       |                                                   |
| name                  | `string`       | Yes      | ⚠️ ALIAS for fullName - only in frontend          |
| phone                 | `string`       | Yes      |                                                   |
| timezone              | `string`       | Yes      | ⚠️ Required in Prisma (default: America/New_York) |
| createdAt             | `string`       | No       | ⚠️ Prisma: DateTime, Frontend: string             |
| avatarUrl             | `string`       | Yes      | ✅ Matches Prisma                                 |
| families              | `UserFamily[]` | Yes      | Derived from FamilyMember relation                |
| onboardingCompleted   | `boolean`      | Yes      |                                                   |
| onboardingCompletedAt | `string`       | Yes      | ⚠️ Prisma: DateTime, Frontend: string             |

**⚠️ Missing from Frontend (in Prisma User):**

- `passwordHash` (intentionally excluded - security)
- `emailVerified`, `emailVerifiedAt`
- `phoneVerified`, `phoneVerifiedAt`
- `lastLoginAt`
- `failedLoginAttempts`, `lockedUntil`
- `passwordResetToken`, `passwordResetExpiresAt`
- `passwordChangedAt`
- `emailVerificationCode`, `emailVerificationExpiresAt`
- `preferences`
- `status`
- `updatedAt`

### Interface: AuthResponse

| Property    | Type     | Optional | Notes |
| ----------- | -------- | -------- | ----- |
| user        | `User`   | No       |       |
| accessToken | `string` | No       |       |

### Interface: RegisterInput

| Property | Type     | Optional | Notes |
| -------- | -------- | -------- | ----- |
| fullName | `string` | No       |       |
| email    | `string` | No       |       |
| password | `string` | No       |       |
| phone    | `string` | Yes      |       |

### Interface: LoginInput

| Property | Type     | Optional | Notes |
| -------- | -------- | -------- | ----- |
| email    | `string` | No       |       |
| password | `string` | No       |       |

### Interface: VerifyEmailInput

| Property | Type     | Optional | Notes |
| -------- | -------- | -------- | ----- |
| email    | `string` | No       |       |
| otp      | `string` | No       |       |

### Interface: ResendVerificationInput

| Property | Type     | Optional | Notes |
| -------- | -------- | -------- | ----- |
| email    | `string` | No       |       |

---

## File: apps/web/src/lib/api/care-recipients.ts

### Interface: CareRecipient

| Property          | Type       | Optional | Notes                                            |
| ----------------- | ---------- | -------- | ------------------------------------------------ |
| id                | `string`   | No       |                                                  |
| familyId          | `string`   | No       |                                                  |
| fullName          | `string`   | No       |                                                  |
| preferredName     | `string`   | Yes      |                                                  |
| dateOfBirth       | `string`   | Yes      | ⚠️ Prisma: DateTime?, Frontend: string           |
| bloodType         | `string`   | Yes      |                                                  |
| allergies         | `string[]` | No       |                                                  |
| conditions        | `string[]` | No       |                                                  |
| notes             | `string`   | Yes      |                                                  |
| photoUrl          | `string`   | Yes      | ✅ Matches Prisma (uses photoUrl, NOT avatarUrl) |
| primaryHospital   | `string`   | Yes      | ✅ Matches Prisma                                |
| hospitalAddress   | `string`   | Yes      |                                                  |
| insuranceProvider | `string`   | Yes      |                                                  |
| insurancePolicyNo | `string`   | Yes      | ✅ Matches Prisma (not insurancePolicyNumber)    |
| createdAt         | `string`   | No       | ⚠️ Prisma: DateTime, Frontend: string            |
| updatedAt         | `string`   | No       | ⚠️ Prisma: DateTime, Frontend: string            |

### Interface: Doctor

| Property  | Type     | Optional | Notes |
| --------- | -------- | -------- | ----- |
| id        | `string` | No       |       |
| name      | `string` | No       |       |
| specialty | `string` | No       |       |
| phone     | `string` | No       |       |
| fax       | `string` | Yes      |       |
| email     | `string` | Yes      |       |
| address   | `string` | Yes      |       |
| notes     | `string` | Yes      |       |

**⚠️ Missing from Frontend (in Prisma Doctor):**

- `careRecipientId` (intentionally excluded - contextual)
- `createdAt`, `updatedAt`

### Interface: EmergencyContact

| Property     | Type      | Optional | Notes |
| ------------ | --------- | -------- | ----- |
| id           | `string`  | No       |       |
| name         | `string`  | No       |       |
| relationship | `string`  | No       |       |
| phone        | `string`  | No       |       |
| email        | `string`  | Yes      |       |
| isPrimary    | `boolean` | No       |       |

**⚠️ Missing from Frontend (in Prisma EmergencyContact):**

- `careRecipientId` (intentionally excluded - contextual)
- `notes`
- `createdAt`

### Interface: CreateCareRecipientInput

| Property          | Type       | Optional | Notes |
| ----------------- | ---------- | -------- | ----- |
| fullName          | `string`   | No       |       |
| preferredName     | `string`   | Yes      |       |
| dateOfBirth       | `string`   | Yes      |       |
| bloodType         | `string`   | Yes      |       |
| allergies         | `string[]` | Yes      |       |
| conditions        | `string[]` | Yes      |       |
| notes             | `string`   | Yes      |       |
| photoUrl          | `string`   | Yes      |       |
| primaryHospital   | `string`   | Yes      |       |
| hospitalAddress   | `string`   | Yes      |       |
| insuranceProvider | `string`   | Yes      |       |
| insurancePolicyNo | `string`   | Yes      |       |

---

## File: apps/web/src/lib/api/medications.ts

### Interface: Medication

| Property        | Type       | Optional | Notes                                                  |
| --------------- | ---------- | -------- | ------------------------------------------------------ |
| id              | `string`   | No       |                                                        |
| careRecipientId | `string`   | No       |                                                        |
| name            | `string`   | No       |                                                        |
| dosage          | `string`   | No       |                                                        |
| form            | `string`   | No       | ⚠️ Prisma: MedicationForm enum, Frontend: string       |
| frequency       | `string`   | No       | ⚠️ Prisma: MedicationFrequency enum, Frontend: string  |
| scheduledTimes  | `string[]` | No       |                                                        |
| instructions    | `string`   | Yes      |                                                        |
| prescribedBy    | `string`   | Yes      |                                                        |
| pharmacy        | `string`   | Yes      |                                                        |
| pharmacyPhone   | `string`   | Yes      |                                                        |
| currentSupply   | `number`   | Yes      |                                                        |
| refillAt        | `number`   | Yes      | ✅ Matches Prisma field name                           |
| isActive        | `boolean`  | No       |                                                        |
| startDate       | `string`   | Yes      | ⚠️ Prisma: DateTime (default now()), Frontend: string? |
| endDate         | `string`   | Yes      | ⚠️ Prisma: DateTime?, Frontend: string                 |
| notes           | `string`   | Yes      |                                                        |
| createdAt       | `string`   | No       |                                                        |
| updatedAt       | `string`   | Yes      | ⚠️ Prisma: DateTime, Frontend: optional string         |

**⚠️ Missing from Frontend (in Prisma Medication):**

- `genericName`
- `timesPerDay`
- `lastRefillDate`

### Interface: MedicationScheduleItem

| Property      | Type                                            | Optional | Notes                                 |
| ------------- | ----------------------------------------------- | -------- | ------------------------------------- |
| medication    | `Medication`                                    | No       |                                       |
| scheduledTime | `string`                                        | No       |                                       |
| time          | `string`                                        | No       |                                       |
| status        | `'PENDING' \| 'GIVEN' \| 'SKIPPED' \| 'MISSED'` | No       | ✅ Matches Prisma MedicationLogStatus |
| logId         | `string`                                        | Yes      |                                       |
| givenTime     | `string`                                        | Yes      |                                       |
| givenBy       | `{ id: string; fullName: string; }`             | Yes      |                                       |
| skipReason    | `string`                                        | Yes      |                                       |

### Interface: CreateMedicationInput

| Property       | Type       | Optional | Notes        |
| -------------- | ---------- | -------- | ------------ |
| name           | `string`   | No       |              |
| genericName    | `string`   | Yes      | ✅ In Prisma |
| dosage         | `string`   | No       |              |
| form           | `string`   | No       |              |
| frequency      | `string`   | No       |              |
| timesPerDay    | `number`   | Yes      | ✅ In Prisma |
| scheduledTimes | `string[]` | Yes      |              |
| instructions   | `string`   | Yes      |              |
| prescribedBy   | `string`   | Yes      |              |
| pharmacy       | `string`   | Yes      |              |
| pharmacyPhone  | `string`   | Yes      |              |
| currentSupply  | `number`   | Yes      |              |
| refillAt       | `number`   | Yes      |              |
| startDate      | `string`   | Yes      |              |
| endDate        | `string`   | Yes      |              |
| notes          | `string`   | Yes      |              |

### Interface: LogMedicationInput

| Property      | Type                   | Optional | Notes                                                       |
| ------------- | ---------------------- | -------- | ----------------------------------------------------------- |
| status        | `'GIVEN' \| 'SKIPPED'` | No       | ⚠️ Subset of MedicationLogStatus (excludes MISSED, PENDING) |
| scheduledTime | `string`               | No       |                                                             |
| skipReason    | `string`               | Yes      |                                                             |
| notes         | `string`               | Yes      |                                                             |

---

## File: apps/web/src/lib/api/family.ts

### Interface: Family

| Property       | Type                                                                            | Optional | Notes           |
| -------------- | ------------------------------------------------------------------------------- | -------- | --------------- |
| id             | `string`                                                                        | No       |                 |
| name           | `string`                                                                        | No       |                 |
| createdAt      | `string`                                                                        | No       |                 |
| members        | `FamilyMember[]`                                                                | Yes      |                 |
| careRecipients | `{ id: string; fullName: string; preferredName?: string; photoUrl?: string }[]` | Yes      | Simplified type |
| invitations    | `FamilyInvitation[]`                                                            | Yes      |                 |

**⚠️ Missing from Frontend (in Prisma Family):**

- `updatedAt`
- `documents` relation

### Interface: FamilyMember

| Property | Type                                               | Optional | Notes                      |
| -------- | -------------------------------------------------- | -------- | -------------------------- |
| id       | `string`                                           | No       |                            |
| userId   | `string`                                           | No       |                            |
| familyId | `string`                                           | No       |                            |
| role     | `'ADMIN' \| 'CAREGIVER' \| 'VIEWER'`               | No       | ✅ Matches FamilyRole enum |
| joinedAt | `string`                                           | No       |                            |
| user     | `{ id: string; fullName: string; email: string; }` | No       | Expanded relation          |

**⚠️ Missing from Frontend (in Prisma FamilyMember):**

- `nickname`
- `canEdit`
- `isActive`
- `notifications` (Json)

### Interface: FamilyInvitation

| Property    | Type                                                 | Optional | Notes                                          |
| ----------- | ---------------------------------------------------- | -------- | ---------------------------------------------- |
| id          | `string`                                             | No       |                                                |
| familyId    | `string`                                             | No       |                                                |
| email       | `string`                                             | No       |                                                |
| role        | `'ADMIN' \| 'CAREGIVER' \| 'VIEWER'`                 | No       |                                                |
| status      | `'PENDING' \| 'ACCEPTED' \| 'DECLINED' \| 'EXPIRED'` | No       | ⚠️ Has DECLINED not in Prisma InvitationStatus |
| invitedById | `string`                                             | No       |                                                |
| createdAt   | `string`                                             | No       |                                                |
| expiresAt   | `string`                                             | No       |                                                |

**⚠️ Prisma InvitationStatus:** PENDING, ACCEPTED, EXPIRED, CANCELLED
**⚠️ Frontend has DECLINED but not CANCELLED - MISMATCH!**

### Interface: CreateFamilyInput

| Property | Type     | Optional | Notes |
| -------- | -------- | -------- | ----- |
| name     | `string` | No       |       |

### Interface: InviteMemberInput

| Property | Type                                 | Optional | Notes |
| -------- | ------------------------------------ | -------- | ----- |
| email    | `string`                             | No       |       |
| role     | `'ADMIN' \| 'CAREGIVER' \| 'VIEWER'` | No       |       |

---

## File: apps/web/src/lib/api/appointments.ts

### Interface: Appointment

| Property            | Type                                                                             | Optional | Notes                                              |
| ------------------- | -------------------------------------------------------------------------------- | -------- | -------------------------------------------------- |
| id                  | `string`                                                                         | No       |                                                    |
| careRecipientId     | `string`                                                                         | No       |                                                    |
| title               | `string`                                                                         | No       |                                                    |
| type                | `string`                                                                         | No       | ⚠️ Prisma: AppointmentType enum, Frontend: string  |
| startTime           | `string`                                                                         | No       |                                                    |
| endTime             | `string`                                                                         | Yes      | ⚠️ Prisma: DateTime (required), Frontend: optional |
| location            | `string`                                                                         | Yes      |                                                    |
| address             | `string`                                                                         | Yes      |                                                    |
| notes               | `string`                                                                         | Yes      |                                                    |
| status              | `'SCHEDULED' \| 'CONFIRMED' \| 'CANCELLED' \| 'COMPLETED'`                       | No       | ⚠️ Missing NO_SHOW from Prisma                     |
| recurrence          | `string`                                                                         | Yes      | ⚠️ Prisma has isRecurring + recurrenceRule         |
| reminderMinutes     | `number[]`                                                                       | No       |                                                    |
| createdAt           | `string`                                                                         | No       |                                                    |
| transportAssignment | `{ id: string; assignedTo: { id: string; fullName: string; }; notes?: string; }` | Yes      |                                                    |

**⚠️ Missing from Frontend (in Prisma Appointment):**

- `doctorId`
- `isRecurring` (boolean)
- `recurrenceRule` (string)
- `updatedAt`

**⚠️ Prisma AppointmentStatus:** SCHEDULED, CONFIRMED, COMPLETED, CANCELLED, NO_SHOW
**⚠️ Frontend missing NO_SHOW - MISMATCH!**

### Interface: CreateAppointmentInput

| Property        | Type       | Optional | Notes |
| --------------- | ---------- | -------- | ----- |
| title           | `string`   | No       |       |
| type            | `string`   | No       |       |
| startTime       | `string`   | No       |       |
| endTime         | `string`   | Yes      |       |
| location        | `string`   | Yes      |       |
| address         | `string`   | Yes      |       |
| notes           | `string`   | Yes      |       |
| recurrence      | `string`   | Yes      |       |
| reminderMinutes | `number[]` | Yes      |       |

### Interface: AssignTransportInput

| Property     | Type     | Optional | Notes |
| ------------ | -------- | -------- | ----- |
| assignedToId | `string` | No       |       |
| notes        | `string` | Yes      |       |

---

## File: apps/web/src/lib/api/timeline.ts

### Interface: TimelineEntry

| Property        | Type                                                                                                                                | Optional | Notes                                          |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------- | -------- | ---------------------------------------------- |
| id              | `string`                                                                                                                            | No       |                                                |
| careRecipientId | `string`                                                                                                                            | No       |                                                |
| createdById     | `string`                                                                                                                            | No       |                                                |
| type            | `string`                                                                                                                            | No       | ⚠️ Prisma: TimelineType enum, Frontend: string |
| title           | `string`                                                                                                                            | No       |                                                |
| description     | `string`                                                                                                                            | Yes      |                                                |
| severity        | `'LOW' \| 'MEDIUM' \| 'HIGH' \| 'CRITICAL'`                                                                                         | Yes      | ✅ Matches Prisma Severity enum                |
| vitals          | `{ bloodPressure?: string; heartRate?: number; temperature?: number; oxygenLevel?: number; bloodSugar?: number; weight?: number; }` | Yes      | ⚠️ Prisma: Json                                |
| occurredAt      | `string`                                                                                                                            | No       |                                                |
| createdAt       | `string`                                                                                                                            | No       |                                                |
| createdBy       | `{ id: string; fullName: string; }`                                                                                                 | No       | Expanded relation                              |

**⚠️ Missing from Frontend (in Prisma TimelineEntry):**

- `attachments` (string[])

### Interface: CreateTimelineEntryInput

| Property    | Type                                                                                                                                | Optional | Notes |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------- | -------- | ----- |
| type        | `string`                                                                                                                            | No       |       |
| title       | `string`                                                                                                                            | No       |       |
| description | `string`                                                                                                                            | Yes      |       |
| severity    | `'LOW' \| 'MEDIUM' \| 'HIGH' \| 'CRITICAL'`                                                                                         | Yes      |       |
| vitals      | `{ bloodPressure?: string; heartRate?: number; temperature?: number; oxygenLevel?: number; bloodSugar?: number; weight?: number; }` | Yes      |       |
| occurredAt  | `string`                                                                                                                            | Yes      |       |

---

## File: apps/web/src/lib/api/emergency.ts

### Interface: EmergencyInfo

| Property          | Type                                                                            | Optional | Notes             |
| ----------------- | ------------------------------------------------------------------------------- | -------- | ----------------- |
| name              | `string`                                                                        | No       |                   |
| preferredName     | `string`                                                                        | Yes      |                   |
| dateOfBirth       | `string`                                                                        | Yes      |                   |
| bloodType         | `string`                                                                        | Yes      |                   |
| allergies         | `string[]`                                                                      | No       |                   |
| conditions        | `string[]`                                                                      | No       |                   |
| medications       | `{ name: string; dosage: string; frequency: string; instructions?: string; }[]` | No       | Simplified for ER |
| emergencyContacts | `EmergencyContact[]`                                                            | No       |                   |
| familyMembers     | `{ name: string; phone?: string; role: string; }[]`                             | No       |                   |
| doctors           | `{ name: string; specialty: string; phone: string; isPrimary?: boolean; }[]`    | No       |                   |
| primaryHospital   | `string`                                                                        | Yes      |                   |
| hospitalAddress   | `string`                                                                        | Yes      |                   |
| insuranceProvider | `string`                                                                        | Yes      |                   |
| insurancePolicyNo | `string`                                                                        | Yes      |                   |
| criticalDocuments | `{ id: string; name: string; type: string; url?: string; }[]`                   | No       |                   |

### Interface: EmergencyAlert

| Property        | Type                                                               | Optional | Notes                                                  |
| --------------- | ------------------------------------------------------------------ | -------- | ------------------------------------------------------ |
| id              | `string`                                                           | No       |                                                        |
| careRecipientId | `string`                                                           | No       |                                                        |
| createdById     | `string`                                                           | No       |                                                        |
| type            | `'FALL' \| 'MEDICAL' \| 'HOSPITALIZATION' \| 'MISSING' \| 'OTHER'` | No       | ✅ Matches Prisma EmergencyType                        |
| status          | `'ACTIVE' \| 'ACKNOWLEDGED' \| 'RESOLVED'`                         | No       | ✅ Matches Prisma AlertStatus                          |
| location        | `string`                                                           | Yes      |                                                        |
| description     | `string`                                                           | Yes      | ⚠️ Prisma has `title` AND `description`, both required |
| createdAt       | `string`                                                           | No       |                                                        |
| acknowledgedAt  | `string`                                                           | Yes      | ⚠️ Not in Prisma model                                 |
| resolvedAt      | `string`                                                           | Yes      |                                                        |
| resolutionNotes | `string`                                                           | Yes      |                                                        |
| createdBy       | `{ id: string; fullName: string; }`                                | No       | Expanded relation                                      |
| acknowledgedBy  | `{ id: string; fullName: string; }`                                | Yes      | ⚠️ Not in Prisma model                                 |

**⚠️ Missing from Frontend (in Prisma EmergencyAlert):**

- `title` (required in Prisma)
- `resolvedById`

**⚠️ Extra in Frontend (not in Prisma):**

- `acknowledgedAt`
- `acknowledgedBy`

### Interface: CreateEmergencyAlertInput

| Property    | Type                                                               | Optional | Notes |
| ----------- | ------------------------------------------------------------------ | -------- | ----- |
| type        | `'FALL' \| 'MEDICAL' \| 'HOSPITALIZATION' \| 'MISSING' \| 'OTHER'` | No       |       |
| location    | `string`                                                           | Yes      |       |
| description | `string`                                                           | Yes      |       |

**⚠️ Missing `title` - required in Prisma!**

---

## File: apps/web/src/lib/api/documents.ts

### Type: DocumentType

```typescript
type DocumentType =
  | "INSURANCE_CARD"
  | "PHOTO_ID"
  | "MEDICAL_RECORD"
  | "LAB_RESULT"
  | "PRESCRIPTION"
  | "POWER_OF_ATTORNEY"
  | "LIVING_WILL"
  | "DNR"
  | "OTHER";
```

✅ **Matches Prisma DocumentType enum exactly**

### Interface: Document

| Property     | Type           | Optional | Notes |
| ------------ | -------------- | -------- | ----- |
| id           | `string`       | No       |       |
| familyId     | `string`       | No       |       |
| uploadedById | `string`       | No       |       |
| name         | `string`       | No       |       |
| type         | `DocumentType` | No       |       |
| s3Key        | `string`       | No       |       |
| mimeType     | `string`       | No       |       |
| sizeBytes    | `number`       | No       |       |
| expiresAt    | `string`       | Yes      |       |
| notes        | `string`       | Yes      |       |
| createdAt    | `string`       | No       |       |
| updatedAt    | `string`       | No       |       |

✅ **Matches Prisma Document model**

### Interface: UploadDocumentInput

| Property  | Type           | Optional | Notes               |
| --------- | -------------- | -------- | ------------------- |
| name      | `string`       | No       |                     |
| type      | `DocumentType` | No       |                     |
| file      | `File`         | No       | Browser File object |
| expiresAt | `string`       | Yes      |                     |
| notes     | `string`       | Yes      |                     |

### Interface: UpdateDocumentInput

| Property  | Type           | Optional | Notes |
| --------- | -------------- | -------- | ----- |
| name      | `string`       | Yes      |       |
| type      | `DocumentType` | Yes      |       |
| expiresAt | `string`       | Yes      |       |
| notes     | `string`       | Yes      |       |

### Interface: DocumentsByCategory

| Property           | Type         | Optional | Notes           |
| ------------------ | ------------ | -------- | --------------- |
| [category: string] | `Document[]` | No       | Index signature |

---

## File: apps/web/src/lib/api/notifications.ts

### Type: NotificationType

```typescript
type NotificationType =
  | "MEDICATION_REMINDER"
  | "APPOINTMENT_REMINDER"
  | "EMERGENCY_ALERT"
  | "FAMILY_UPDATE"
  | "SYSTEM"
  | "CARE_RECIPIENT_DELETED"
  | "CARE_RECIPIENT_UPDATED"
  | "MEDICATION_DELETED"
  | "APPOINTMENT_DELETED"
  | "FAMILY_MEMBER_REMOVED"
  | "FAMILY_MEMBER_ROLE_CHANGED"
  | "FAMILY_DELETED";
```

**⚠️ Prisma NotificationType has MORE values:**

- `MEDICATION_MISSED`
- `SHIFT_REMINDER`
- `SHIFT_HANDOFF`
- `FAMILY_INVITE`
- `DOCUMENT_SHARED`
- `TIMELINE_UPDATE`
- `REFILL_NEEDED`
- `REFILL_ALERT`
- `GENERAL`

**⚠️ Frontend has types NOT in Prisma:**

- `FAMILY_UPDATE`
- `SYSTEM`

### Interface: Notification

| Property  | Type                  | Optional | Notes            |
| --------- | --------------------- | -------- | ---------------- |
| id        | `string`              | No       |                  |
| userId    | `string`              | No       |                  |
| title     | `string`              | No       |                  |
| body      | `string`              | No       |                  |
| type      | `NotificationType`    | No       |                  |
| read      | `boolean`             | No       |                  |
| data      | `Record<string, any>` | Yes      | ⚠️ Prisma: Json? |
| createdAt | `string`              | No       |                  |

**⚠️ Missing from Frontend (in Prisma Notification):**

- `readAt`

### Interface: PushSubscription

| Property       | Type                                | Optional | Notes |
| -------------- | ----------------------------------- | -------- | ----- |
| endpoint       | `string`                            | No       |       |
| expirationTime | `number \| null`                    | Yes      |       |
| keys           | `{ p256dh: string; auth: string; }` | No       |       |

### Interface: PushSubscriptionRequest

| Property   | Type                                | Optional | Notes |
| ---------- | ----------------------------------- | -------- | ----- |
| endpoint   | `string`                            | No       |       |
| keys       | `{ p256dh: string; auth: string; }` | No       |       |
| platform   | `'web' \| 'ios' \| 'android'`       | Yes      |       |
| deviceName | `string`                            | Yes      |       |

---

## File: apps/web/src/lib/api/shifts.ts

### Interface: CaregiverShift

| Property         | Type                                                                            | Optional | Notes                                            |
| ---------------- | ------------------------------------------------------------------------------- | -------- | ------------------------------------------------ |
| id               | `string`                                                                        | No       |                                                  |
| careRecipientId  | `string`                                                                        | No       |                                                  |
| caregiverId      | `string`                                                                        | No       |                                                  |
| startTime        | `string`                                                                        | No       |                                                  |
| endTime          | `string`                                                                        | No       |                                                  |
| status           | `'SCHEDULED' \| 'IN_PROGRESS' \| 'COMPLETED' \| 'CANCELLED' \| 'NO_SHOW'`       | No       | ⚠️ Missing CONFIRMED from Prisma                 |
| actualStartTime  | `string`                                                                        | Yes      | ⚠️ Not in Prisma (has checkedInAt)               |
| actualEndTime    | `string`                                                                        | Yes      | ⚠️ Not in Prisma (has checkedOutAt)              |
| checkInNotes     | `string`                                                                        | Yes      | ⚠️ Not in Prisma                                 |
| checkOutNotes    | `string`                                                                        | Yes      | ⚠️ Not in Prisma                                 |
| handoffNotes     | `string`                                                                        | Yes      | ⚠️ Not in Prisma                                 |
| checkInLocation  | `string`                                                                        | Yes      | ⚠️ Not in Prisma                                 |
| checkOutLocation | `string`                                                                        | Yes      | ⚠️ Not in Prisma                                 |
| createdAt        | `string`                                                                        | No       |                                                  |
| updatedAt        | `string`                                                                        | No       |                                                  |
| caregiver        | `{ id: string; fullName: string; email: string; avatarUrl?: string; }`          | Yes      |                                                  |
| careRecipient    | `{ id: string; fullName: string; preferredName?: string; avatarUrl?: string; }` | Yes      | ⚠️ Uses avatarUrl but CareRecipient has photoUrl |
| createdBy        | `{ id: string; fullName: string; }`                                             | Yes      | ⚠️ Not in Prisma                                 |

**⚠️ Prisma ShiftStatus:** SCHEDULED, CONFIRMED, IN_PROGRESS, COMPLETED, CANCELLED, NO_SHOW
**⚠️ Frontend missing CONFIRMED - MISMATCH!**

**⚠️ Prisma CaregiverShift fields:**

- `notes` (not checkInNotes/checkOutNotes/handoffNotes)
- `checkedInAt` (not actualStartTime)
- `checkedOutAt` (not actualEndTime)

### Interface: CreateShiftDto

| Property    | Type     | Optional | Notes |
| ----------- | -------- | -------- | ----- |
| caregiverId | `string` | No       |       |
| startTime   | `string` | No       |       |
| endTime     | `string` | No       |       |
| notes       | `string` | Yes      |       |

### Interface: CheckInDto

| Property | Type     | Optional | Notes |
| -------- | -------- | -------- | ----- |
| notes    | `string` | Yes      |       |
| location | `string` | Yes      |       |

### Interface: CheckOutDto

| Property     | Type     | Optional | Notes |
| ------------ | -------- | -------- | ----- |
| notes        | `string` | Yes      |       |
| location     | `string` | Yes      |       |
| handoffNotes | `string` | Yes      |       |

### Interface: OnDutyResponse

| Property  | Type                                                                   | Optional | Notes |
| --------- | ---------------------------------------------------------------------- | -------- | ----- |
| caregiver | `{ id: string; fullName: string; email: string; avatarUrl?: string; }` | No       |       |
| shift     | `CaregiverShift`                                                       | No       |       |

---

## File: apps/web/src/lib/api/client.ts

### Class: ApiError (extends Error)

| Property | Type                                  | Optional | Notes            |
| -------- | ------------------------------------- | -------- | ---------------- |
| status   | `number`                              | No       | HTTP status code |
| data     | `{ message: string; error?: string }` | No       | Error details    |

### Interface: RequestOptions (extends RequestInit)

| Property | Type      | Optional | Notes                      |
| -------- | --------- | -------- | -------------------------- |
| skipAuth | `boolean` | Yes      | Skip authentication header |

---

## File: apps/web/src/hooks/use-auth.ts

### Interface: AuthState

| Property           | Type                                                                                               | Optional | Notes |
| ------------------ | -------------------------------------------------------------------------------------------------- | -------- | ----- |
| user               | `User \| null`                                                                                     | No       |       |
| token              | `string \| null`                                                                                   | No       |       |
| isLoading          | `boolean`                                                                                          | No       |       |
| isAuthenticated    | `boolean`                                                                                          | No       |       |
| sessionChecked     | `boolean`                                                                                          | No       |       |
| login              | `(data: LoginInput) => Promise<void>`                                                              | No       |       |
| register           | `(data: RegisterInput) => Promise<{ message: string; user: { email: string; fullName: string } }>` | No       |       |
| verifyEmail        | `(data: VerifyEmailInput) => Promise<void>`                                                        | No       |       |
| resendVerification | `(data: ResendVerificationInput) => Promise<void>`                                                 | No       |       |
| logout             | `() => Promise<void>`                                                                              | No       |       |
| fetchUser          | `() => Promise<void>`                                                                              | No       |       |
| refetchUser        | `() => Promise<void>`                                                                              | No       |       |
| updateUser         | `(data: Partial<User>) => Promise<void>`                                                           | No       |       |
| setUser            | `(user: User \| null) => void`                                                                     | No       |       |
| setToken           | `(token: string \| null) => void`                                                                  | No       |       |
| clearAuth          | `() => void`                                                                                       | No       |       |

---

## File: apps/web/src/hooks/use-chat.ts

### Interface: UseChatOptions

| Property    | Type      | Optional | Notes |
| ----------- | --------- | -------- | ----- |
| familyId    | `string`  | Yes      |       |
| autoConnect | `boolean` | Yes      |       |

---

## File: apps/web/src/lib/offline-storage.ts

### Interface: PendingAction

| Property   | Type                                                                          | Optional | Notes          |
| ---------- | ----------------------------------------------------------------------------- | -------- | -------------- |
| id         | `string`                                                                      | No       |                |
| type       | `'medication_log' \| 'timeline_entry' \| 'shift_checkin' \| 'shift_checkout'` | No       |                |
| payload    | `any`                                                                         | No       |                |
| createdAt  | `number`                                                                      | No       | Unix timestamp |
| retryCount | `number`                                                                      | No       |                |

---

## File: apps/web/src/contexts/family-space-context.tsx

### Interface: FamilySpaceSelection

| Property        | Type             | Optional | Notes |
| --------------- | ---------------- | -------- | ----- |
| familyId        | `string \| null` | No       |       |
| careRecipientId | `string \| null` | No       |       |

### Interface: FamilySpaceContextValue

| Property                 | Type                                         | Optional | Notes |
| ------------------------ | -------------------------------------------- | -------- | ----- |
| selectedFamilyId         | `string \| null`                             | No       |       |
| selectedCareRecipientId  | `string \| null`                             | No       |       |
| selectedFamily           | `UserFamily \| null`                         | No       |       |
| selectedCareRecipient    | `CareRecipient \| null`                      | No       |       |
| families                 | `UserFamily[]`                               | No       |       |
| careRecipients           | `CareRecipient[]`                            | No       |       |
| currentRole              | `'ADMIN' \| 'CAREGIVER' \| 'VIEWER' \| null` | No       |       |
| setSelectedFamily        | `(familyId: string \| null) => void`         | No       |       |
| setSelectedCareRecipient | `(careRecipientId: string \| null) => void`  | No       |       |
| isLoading                | `boolean`                                    | No       |       |

---

## File: apps/web/src/lib/websocket.ts

### Type: EventCallback

```typescript
type EventCallback = (data: any) => void;
```

### Const: WS_EVENTS (as const)

```typescript
const WS_EVENTS = {
  // Incoming events from backend
  EMERGENCY_ALERT: "emergency_alert",
  EMERGENCY_RESOLVED: "emergency_resolved",
  MEDICATION_LOGGED: "medication_logged",
  MEDICATION_REMINDER: "medication_reminder",
  MEDICATION_DELETED: "medication_deleted",
  APPOINTMENT_CREATED: "appointment_created",
  APPOINTMENT_UPDATED: "appointment_updated",
  APPOINTMENT_REMINDER: "appointment_reminder",
  APPOINTMENT_DELETED: "appointment_deleted",
  TIMELINE_ENTRY: "timeline_entry",
  SHIFT_UPDATE: "shift_update",
  SHIFT_CHECKED_IN: "shift_checked_in",
  SHIFT_CHECKED_OUT: "shift_checked_out",
  FAMILY_MEMBER_JOINED: "family_member_joined",
  NOTIFICATION: "notification",

  // Admin action events
  CARE_RECIPIENT_DELETED: "care_recipient_deleted",
  CARE_RECIPIENT_UPDATED: "care_recipient_updated",
  FAMILY_MEMBER_REMOVED: "family_member_removed",
  FAMILY_MEMBER_ROLE_UPDATED: "family_member_role_updated",
  FAMILY_DELETED: "family_deleted",

  // User-specific events
  YOU_WERE_REMOVED: "you_were_removed",
  YOUR_ROLE_CHANGED: "your_role_changed",

  // Broadcast events
  WS_BROADCAST: "ws_broadcast",
  EMERGENCY_NOTIFICATION: "emergency_notification",

  // Outgoing events
  JOIN_FAMILY: "join_family",
  LEAVE_FAMILY: "leave_family",
} as const;
```

---

## ⚠️ Field Name Mismatches Summary

### User/CareRecipient Photo Fields

| Context                                 | Field Name  | Notes                           |
| --------------------------------------- | ----------- | ------------------------------- |
| User (Prisma)                           | `avatarUrl` | ✅ Matches frontend             |
| User (Frontend)                         | `avatarUrl` | ✅                              |
| CareRecipient (Prisma)                  | `photoUrl`  | ✅ Matches frontend             |
| CareRecipient (Frontend)                | `photoUrl`  | ✅                              |
| CaregiverShift.careRecipient (Frontend) | `avatarUrl` | ⚠️ WRONG - should be `photoUrl` |

### Medication Refill Fields

| Context               | Field Name | Notes      |
| --------------------- | ---------- | ---------- |
| Medication (Prisma)   | `refillAt` | ✅ Int?    |
| Medication (Frontend) | `refillAt` | ✅ Matches |

### Insurance Policy Fields

| Context                  | Field Name          | Notes      |
| ------------------------ | ------------------- | ---------- |
| CareRecipient (Prisma)   | `insurancePolicyNo` | ✅         |
| CareRecipient (Frontend) | `insurancePolicyNo` | ✅ Matches |

### Hospital Fields

| Context                  | Field Name        | Notes      |
| ------------------------ | ----------------- | ---------- |
| CareRecipient (Prisma)   | `primaryHospital` | ✅         |
| CareRecipient (Frontend) | `primaryHospital` | ✅ Matches |

---

## ⚠️ Type Mismatches Summary

### DateTime vs String

All DateTime fields in Prisma are represented as `string` in frontend types. This is correct for JSON serialization but should be documented:

- `createdAt`, `updatedAt`, `occurredAt`, `startTime`, `endTime`, `expiresAt`, etc.

### Enums vs Strings

Several Prisma enums are represented as plain `string` in frontend:
| Prisma Enum | Frontend Type | Files Affected |
|-------------|---------------|----------------|
| `MedicationForm` | `string` | medications.ts |
| `MedicationFrequency` | `string` | medications.ts |
| `AppointmentType` | `string` | appointments.ts |
| `TimelineType` | `string` | timeline.ts |

### Status Enum Mismatches

| Entity            | Prisma Values                                                    | Frontend Values                                       | Issue                                           |
| ----------------- | ---------------------------------------------------------------- | ----------------------------------------------------- | ----------------------------------------------- |
| AppointmentStatus | SCHEDULED, CONFIRMED, COMPLETED, CANCELLED, NO_SHOW              | SCHEDULED, CONFIRMED, CANCELLED, COMPLETED            | Missing NO_SHOW                                 |
| ShiftStatus       | SCHEDULED, CONFIRMED, IN_PROGRESS, COMPLETED, CANCELLED, NO_SHOW | SCHEDULED, IN_PROGRESS, COMPLETED, CANCELLED, NO_SHOW | Missing CONFIRMED                               |
| InvitationStatus  | PENDING, ACCEPTED, EXPIRED, CANCELLED                            | PENDING, ACCEPTED, DECLINED, EXPIRED                  | Has DECLINED (not in Prisma), Missing CANCELLED |
| NotificationType  | Many more values                                                 | Subset + extra FAMILY_UPDATE, SYSTEM                  | Significant mismatch                            |

---

## ⚠️ Schema Differences Summary

### CaregiverShift - Major Structural Differences

| Prisma Field   | Frontend Field                                  | Notes                         |
| -------------- | ----------------------------------------------- | ----------------------------- |
| `notes`        | `checkInNotes`, `checkOutNotes`, `handoffNotes` | Frontend splits into 3 fields |
| `checkedInAt`  | `actualStartTime`                               | Different name                |
| `checkedOutAt` | `actualEndTime`                                 | Different name                |
| -              | `checkInLocation`                               | Only in frontend              |
| -              | `checkOutLocation`                              | Only in frontend              |
| -              | `createdBy`                                     | Only in frontend              |

### EmergencyAlert - Missing/Extra Fields

| Prisma Field       | Frontend Field   | Notes                |
| ------------------ | ---------------- | -------------------- |
| `title` (required) | -                | Missing in frontend! |
| `resolvedById`     | -                | Missing in frontend  |
| -                  | `acknowledgedAt` | Only in frontend     |
| -                  | `acknowledgedBy` | Only in frontend     |

---

## Prisma Enums Reference (for comparison)

### Platform

```prisma
enum Platform { WEB, IOS, ANDROID }
```

### FamilyRole

```prisma
enum FamilyRole { ADMIN, CAREGIVER, VIEWER }
```

### InvitationStatus

```prisma
enum InvitationStatus { PENDING, ACCEPTED, EXPIRED, CANCELLED }
```

### AppointmentType

```prisma
enum AppointmentType {
  DOCTOR_VISIT, PHYSICAL_THERAPY, LAB_WORK, IMAGING,
  SPECIALIST, HOME_HEALTH, OTHER
}
```

### AppointmentStatus

```prisma
enum AppointmentStatus { SCHEDULED, CONFIRMED, COMPLETED, CANCELLED, NO_SHOW }
```

### MedicationForm

```prisma
enum MedicationForm {
  TABLET, CAPSULE, LIQUID, INJECTION, PATCH,
  CREAM, INHALER, DROPS, OTHER
}
```

### MedicationFrequency

```prisma
enum MedicationFrequency {
  DAILY, TWICE_DAILY, THREE_TIMES_DAILY, FOUR_TIMES_DAILY,
  WEEKLY, AS_NEEDED, OTHER
}
```

### MedicationLogStatus

```prisma
enum MedicationLogStatus { GIVEN, SKIPPED, MISSED, PENDING }
```

### ShiftStatus

```prisma
enum ShiftStatus {
  SCHEDULED, CONFIRMED, IN_PROGRESS, COMPLETED, CANCELLED, NO_SHOW
}
```

### TimelineType

```prisma
enum TimelineType {
  NOTE, VITALS, SYMPTOM, INCIDENT, MOOD, MEAL, ACTIVITY,
  SLEEP, BATHROOM, MEDICATION_CHANGE, APPOINTMENT_SUMMARY, OTHER
}
```

### Severity

```prisma
enum Severity { LOW, MEDIUM, HIGH, CRITICAL }
```

### DocumentType

```prisma
enum DocumentType {
  INSURANCE_CARD, PHOTO_ID, MEDICAL_RECORD, LAB_RESULT,
  PRESCRIPTION, POWER_OF_ATTORNEY, LIVING_WILL, DNR, OTHER
}
```

### EmergencyType

```prisma
enum EmergencyType { FALL, MEDICAL, MISSING, HOSPITALIZATION, OTHER }
```

### AlertStatus

```prisma
enum AlertStatus { ACTIVE, ACKNOWLEDGED, RESOLVED }
```

### NotificationType

```prisma
enum NotificationType {
  MEDICATION_REMINDER, MEDICATION_MISSED,
  APPOINTMENT_REMINDER,
  SHIFT_REMINDER, SHIFT_HANDOFF,
  EMERGENCY_ALERT,
  FAMILY_INVITE,
  DOCUMENT_SHARED,
  TIMELINE_UPDATE,
  REFILL_NEEDED, REFILL_ALERT,
  GENERAL,
  // Admin action notifications
  CARE_RECIPIENT_DELETED, CARE_RECIPIENT_UPDATED,
  MEDICATION_DELETED,
  APPOINTMENT_DELETED,
  FAMILY_MEMBER_REMOVED, FAMILY_MEMBER_ROLE_CHANGED,
  FAMILY_DELETED
}
```

---

## Critical Issues to Address

1. **CaregiverShift.careRecipient uses `avatarUrl` instead of `photoUrl`** - Frontend bug
2. **FamilyInvitation status has `DECLINED` but Prisma has `CANCELLED`** - Enum mismatch
3. **AppointmentStatus missing `NO_SHOW`** - Frontend incomplete
4. **ShiftStatus missing `CONFIRMED`** - Frontend incomplete
5. **EmergencyAlert missing required `title` field** - Frontend incomplete
6. **NotificationType significant mismatch** - Many values differ
7. **CaregiverShift structural differences** - Field naming/structure mismatch with Prisma
