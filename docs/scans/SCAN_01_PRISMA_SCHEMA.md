# Prisma Schema Map

**Generated:** 2026-01-20
**Source:** `packages/database/prisma/schema.prisma`

---

## Summary

- **Total Models:** 19
- **Total Enums:** 15
- **Database:** PostgreSQL

---

## Models

### Model: User
**Table:** `User` (no @@map)

| Field | Type | Optional | Default | DB Column (@map) |
|-------|------|----------|---------|------------------|
| id | String | No | uuid() | - |
| email | String | No | - | - |
| passwordHash | String | No | - | - |
| fullName | String | No | - | - |
| phone | String | Yes | - | - |
| timezone | String | No | "America/New_York" | - |
| avatarUrl | String | Yes | - | - |
| status | String | No | "PENDING" | - |
| emailVerified | Boolean | No | false | - |
| emailVerifiedAt | DateTime | Yes | - | - |
| phoneVerified | Boolean | No | false | - |
| phoneVerifiedAt | DateTime | Yes | - | - |
| lastLoginAt | DateTime | Yes | - | - |
| failedLoginAttempts | Int | No | 0 | - |
| lockedUntil | DateTime | Yes | - | - |
| passwordResetToken | String | Yes | - | - |
| passwordResetExpiresAt | DateTime | Yes | - | - |
| passwordChangedAt | DateTime | Yes | - | - |
| emailVerificationCode | String | Yes | - | - |
| emailVerificationExpiresAt | DateTime | Yes | - | - |
| preferences | Json | Yes | - | - |
| onboardingCompleted | Boolean | No | false | - |
| onboardingCompletedAt | DateTime | Yes | - | - |
| createdAt | DateTime | No | now() | - |
| updatedAt | DateTime | No | @updatedAt | - |
| sessions | Session[] | - | - | Relation |
| familyMemberships | FamilyMember[] | - | - | Relation |
| caregiverShifts | CaregiverShift[] | - | - | Relation |
| medicationLogs | MedicationLog[] | - | - | Relation |
| timelineEntries | TimelineEntry[] | - | - | Relation |
| notifications | Notification[] | - | - | Relation |
| emergencyAlerts | EmergencyAlert[] | - | - | Relation |
| pushTokens | PushToken[] | - | - | Relation |

**Indexes:**
- Primary: `id`
- Unique: `email`

---

### Model: Session
**Table:** `Session` (no @@map)

| Field | Type | Optional | Default | DB Column (@map) |
|-------|------|----------|---------|------------------|
| id | String | No | uuid() | - |
| userId | String | No | - | - |
| refreshToken | String | No | - | - |
| expiresAt | DateTime | No | - | - |
| ipAddress | String | Yes | - | - |
| userAgent | String | Yes | - | - |
| deviceInfo | String | Yes | - | - |
| isActive | Boolean | No | true | - |
| lastUsedAt | DateTime | Yes | - | - |
| createdAt | DateTime | No | now() | - |
| updatedAt | DateTime | No | @updatedAt | - |
| user | User | No | - | Relation |

**Indexes:**
- Primary: `id`
- Unique: `refreshToken`
- Index: `userId`
- Index: `refreshToken`
- Index: `expiresAt`

---

### Model: PushToken
**Table:** `PushToken` (no @@map)

| Field | Type | Optional | Default | DB Column (@map) |
|-------|------|----------|---------|------------------|
| id | String | No | uuid() | - |
| userId | String | No | - | - |
| token | String | No | - | - |
| platform | Platform (enum) | No | - | - |
| createdAt | DateTime | No | now() | - |
| user | User | No | - | Relation |

**Indexes:**
- Primary: `id`
- Unique: `token`
- Index: `userId`

---

### Model: Family
**Table:** `Family` (no @@map)

| Field | Type | Optional | Default | DB Column (@map) |
|-------|------|----------|---------|------------------|
| id | String | No | uuid() | - |
| name | String | No | - | - |
| createdAt | DateTime | No | now() | - |
| updatedAt | DateTime | No | @updatedAt | - |
| members | FamilyMember[] | - | - | Relation |
| careRecipients | CareRecipient[] | - | - | Relation |
| documents | Document[] | - | - | Relation |
| invitations | FamilyInvitation[] | - | - | Relation |

**Indexes:**
- Primary: `id`

---

### Model: FamilyMember
**Table:** `FamilyMember` (no @@map)

| Field | Type | Optional | Default | DB Column (@map) |
|-------|------|----------|---------|------------------|
| id | String | No | uuid() | - |
| familyId | String | No | - | - |
| userId | String | No | - | - |
| role | FamilyRole (enum) | No | - | - |
| nickname | String | Yes | - | - |
| canEdit | Boolean | No | true | - |
| isActive | Boolean | No | true | - |
| joinedAt | DateTime | No | now() | - |
| notifications | Json | Yes | - | - |
| family | Family | No | - | Relation |
| user | User | No | - | Relation |

**Indexes:**
- Primary: `id`
- Unique: `[familyId, userId]` (composite)
- Index: `userId`

---

### Model: FamilyInvitation
**Table:** `FamilyInvitation` (no @@map)

| Field | Type | Optional | Default | DB Column (@map) |
|-------|------|----------|---------|------------------|
| id | String | No | uuid() | - |
| familyId | String | No | - | - |
| email | String | No | - | - |
| role | FamilyRole (enum) | No | - | - |
| token | String | No | - | - |
| status | InvitationStatus (enum) | No | PENDING | - |
| invitedById | String | No | - | - |
| expiresAt | DateTime | No | - | - |
| createdAt | DateTime | No | now() | - |
| family | Family | No | - | Relation |

**Indexes:**
- Primary: `id`
- Unique: `token`
- Index: `familyId`
- Index: `email`

---

### Model: CareRecipient
**Table:** `CareRecipient` (no @@map)

| Field | Type | Optional | Default | DB Column (@map) |
|-------|------|----------|---------|------------------|
| id | String | No | uuid() | - |
| familyId | String | No | - | - |
| fullName | String | No | - | - |
| preferredName | String | Yes | - | - |
| dateOfBirth | DateTime | Yes | - | - |
| photoUrl | String | Yes | - | - |
| bloodType | String | Yes | - | - |
| allergies | String[] | No | [] | - |
| conditions | String[] | No | [] | - |
| notes | String | Yes | - | - |
| primaryHospital | String | Yes | - | - |
| hospitalAddress | String | Yes | - | - |
| insuranceProvider | String | Yes | - | - |
| insurancePolicyNo | String | Yes | - | - |
| createdAt | DateTime | No | now() | - |
| updatedAt | DateTime | No | @updatedAt | - |
| family | Family | No | - | Relation |
| doctors | Doctor[] | - | - | Relation |
| medications | Medication[] | - | - | Relation |
| appointments | Appointment[] | - | - | Relation |
| caregiverShifts | CaregiverShift[] | - | - | Relation |
| timelineEntries | TimelineEntry[] | - | - | Relation |
| emergencyContacts | EmergencyContact[] | - | - | Relation |
| emergencyAlerts | EmergencyAlert[] | - | - | Relation |

**Indexes:**
- Primary: `id`
- Index: `familyId`

---

### Model: Doctor
**Table:** `Doctor` (no @@map)

| Field | Type | Optional | Default | DB Column (@map) |
|-------|------|----------|---------|------------------|
| id | String | No | uuid() | - |
| careRecipientId | String | No | - | - |
| name | String | No | - | - |
| specialty | String | No | - | - |
| phone | String | No | - | - |
| fax | String | Yes | - | - |
| email | String | Yes | - | - |
| address | String | Yes | - | - |
| notes | String | Yes | - | - |
| createdAt | DateTime | No | now() | - |
| updatedAt | DateTime | No | @updatedAt | - |
| careRecipient | CareRecipient | No | - | Relation |
| appointments | Appointment[] | - | - | Relation |

**Indexes:**
- Primary: `id`
- Index: `careRecipientId`

---

### Model: EmergencyContact
**Table:** `EmergencyContact` (no @@map)

| Field | Type | Optional | Default | DB Column (@map) |
|-------|------|----------|---------|------------------|
| id | String | No | uuid() | - |
| careRecipientId | String | No | - | - |
| name | String | No | - | - |
| relationship | String | No | - | - |
| phone | String | No | - | - |
| email | String | Yes | - | - |
| isPrimary | Boolean | No | false | - |
| notes | String | Yes | - | - |
| createdAt | DateTime | No | now() | - |
| careRecipient | CareRecipient | No | - | Relation |

**Indexes:**
- Primary: `id`
- Index: `careRecipientId`

---

### Model: Appointment
**Table:** `Appointment` (no @@map)

| Field | Type | Optional | Default | DB Column (@map) |
|-------|------|----------|---------|------------------|
| id | String | No | uuid() | - |
| careRecipientId | String | No | - | - |
| doctorId | String | Yes | - | - |
| title | String | No | - | - |
| type | AppointmentType (enum) | No | - | - |
| startTime | DateTime | No | - | - |
| endTime | DateTime | No | - | - |
| location | String | Yes | - | - |
| address | String | Yes | - | - |
| notes | String | Yes | - | - |
| isRecurring | Boolean | No | false | - |
| recurrenceRule | String | Yes | - | - |
| reminderMinutes | Int[] | No | [60, 1440] | - |
| status | AppointmentStatus (enum) | No | SCHEDULED | - |
| createdAt | DateTime | No | now() | - |
| updatedAt | DateTime | No | @updatedAt | - |
| careRecipient | CareRecipient | No | - | Relation |
| doctor | Doctor | Yes | - | Relation |
| transportAssignment | TransportAssignment | Yes | - | Relation |

**Indexes:**
- Primary: `id`
- Index: `careRecipientId`
- Index: `startTime`

---

### Model: TransportAssignment
**Table:** `TransportAssignment` (no @@map)

| Field | Type | Optional | Default | DB Column (@map) |
|-------|------|----------|---------|------------------|
| id | String | No | uuid() | - |
| appointmentId | String | No | - | - |
| assignedToId | String | No | - | - |
| notes | String | Yes | - | - |
| confirmed | Boolean | No | false | - |
| createdAt | DateTime | No | now() | - |
| appointment | Appointment | No | - | Relation |

**Indexes:**
- Primary: `id`
- Unique: `appointmentId`

---

### Model: Medication
**Table:** `Medication` (no @@map)

| Field | Type | Optional | Default | DB Column (@map) |
|-------|------|----------|---------|------------------|
| id | String | No | uuid() | - |
| careRecipientId | String | No | - | - |
| name | String | No | - | - |
| genericName | String | Yes | - | - |
| dosage | String | No | - | - |
| form | MedicationForm (enum) | No | - | - |
| instructions | String | Yes | - | - |
| prescribedBy | String | Yes | - | - |
| pharmacy | String | Yes | - | - |
| pharmacyPhone | String | Yes | - | - |
| frequency | MedicationFrequency (enum) | No | - | - |
| timesPerDay | Int | No | 1 | - |
| scheduledTimes | String[] | No | [] | - |
| currentSupply | Int | Yes | - | - |
| refillAt | Int | Yes | - | - |
| lastRefillDate | DateTime | Yes | - | - |
| isActive | Boolean | No | true | - |
| startDate | DateTime | No | now() | - |
| endDate | DateTime | Yes | - | - |
| notes | String | Yes | - | - |
| createdAt | DateTime | No | now() | - |
| updatedAt | DateTime | No | @updatedAt | - |
| careRecipient | CareRecipient | No | - | Relation |
| logs | MedicationLog[] | - | - | Relation |

**Indexes:**
- Primary: `id`
- Index: `careRecipientId`
- Index: `isActive`

---

### Model: MedicationLog
**Table:** `MedicationLog` (no @@map)

| Field | Type | Optional | Default | DB Column (@map) |
|-------|------|----------|---------|------------------|
| id | String | No | uuid() | - |
| medicationId | String | No | - | - |
| givenById | String | No | - | - |
| scheduledTime | DateTime | No | - | - |
| givenTime | DateTime | Yes | - | - |
| status | MedicationLogStatus (enum) | No | - | - |
| skipReason | String | Yes | - | - |
| notes | String | Yes | - | - |
| createdAt | DateTime | No | now() | - |
| medication | Medication | No | - | Relation |
| givenBy | User | No | - | Relation |

**Indexes:**
- Primary: `id`
- Index: `[medicationId, scheduledTime]` (composite)
- Index: `givenById`

---

### Model: CaregiverShift
**Table:** `CaregiverShift` (no @@map)

| Field | Type | Optional | Default | DB Column (@map) |
|-------|------|----------|---------|------------------|
| id | String | No | uuid() | - |
| careRecipientId | String | No | - | - |
| caregiverId | String | No | - | - |
| startTime | DateTime | No | - | - |
| endTime | DateTime | No | - | - |
| notes | String | Yes | - | - |
| status | ShiftStatus (enum) | No | SCHEDULED | - |
| checkedInAt | DateTime | Yes | - | - |
| checkedOutAt | DateTime | Yes | - | - |
| createdAt | DateTime | No | now() | - |
| updatedAt | DateTime | No | @updatedAt | - |
| careRecipient | CareRecipient | No | - | Relation |
| caregiver | User | No | - | Relation |

**Indexes:**
- Primary: `id`
- Index: `[careRecipientId, startTime]` (composite)
- Index: `caregiverId`

---

### Model: TimelineEntry
**Table:** `TimelineEntry` (no @@map)

| Field | Type | Optional | Default | DB Column (@map) |
|-------|------|----------|---------|------------------|
| id | String | No | uuid() | - |
| careRecipientId | String | No | - | - |
| createdById | String | No | - | - |
| type | TimelineType (enum) | No | - | - |
| title | String | No | - | - |
| description | String | Yes | - | - |
| severity | Severity (enum) | Yes | - | - |
| vitals | Json | Yes | - | - |
| attachments | String[] | No | [] | - |
| occurredAt | DateTime | No | now() | - |
| createdAt | DateTime | No | now() | - |
| careRecipient | CareRecipient | No | - | Relation |
| createdBy | User | No | - | Relation |

**Indexes:**
- Primary: `id`
- Index: `[careRecipientId, occurredAt]` (composite)
- Index: `createdById`

---

### Model: Document
**Table:** `Document` (no @@map)

| Field | Type | Optional | Default | DB Column (@map) |
|-------|------|----------|---------|------------------|
| id | String | No | uuid() | - |
| familyId | String | No | - | - |
| uploadedById | String | No | - | - |
| name | String | No | - | - |
| type | DocumentType (enum) | No | - | - |
| s3Key | String | No | - | - |
| mimeType | String | No | - | - |
| sizeBytes | Int | No | - | - |
| expiresAt | DateTime | Yes | - | - |
| notes | String | Yes | - | - |
| createdAt | DateTime | No | now() | - |
| updatedAt | DateTime | No | @updatedAt | - |
| family | Family | No | - | Relation |

**Indexes:**
- Primary: `id`
- Index: `[familyId, type]` (composite)

---

### Model: EmergencyAlert
**Table:** `EmergencyAlert` (no @@map)

| Field | Type | Optional | Default | DB Column (@map) |
|-------|------|----------|---------|------------------|
| id | String | No | uuid() | - |
| careRecipientId | String | No | - | - |
| createdById | String | No | - | - |
| type | EmergencyType (enum) | No | - | - |
| title | String | No | - | - |
| description | String | No | - | - |
| location | String | Yes | - | - |
| status | AlertStatus (enum) | No | ACTIVE | - |
| resolvedAt | DateTime | Yes | - | - |
| resolvedById | String | Yes | - | - |
| resolutionNotes | String | Yes | - | - |
| createdAt | DateTime | No | now() | - |
| careRecipient | CareRecipient | No | - | Relation |
| createdBy | User | No | - | Relation |

**Indexes:**
- Primary: `id`
- Index: `careRecipientId`
- Index: `status`

---

### Model: Notification
**Table:** `Notification` (no @@map)

| Field | Type | Optional | Default | DB Column (@map) |
|-------|------|----------|---------|------------------|
| id | String | No | uuid() | - |
| userId | String | No | - | - |
| type | NotificationType (enum) | No | - | - |
| title | String | No | - | - |
| body | String | No | - | - |
| data | Json | Yes | - | - |
| read | Boolean | No | false | - |
| readAt | DateTime | Yes | - | - |
| createdAt | DateTime | No | now() | - |
| user | User | No | - | Relation |

**Indexes:**
- Primary: `id`
- Index: `[userId, read, createdAt]` (composite)

---

### Model: AuditLog
**Table:** `AuditLog` (no @@map)

| Field | Type | Optional | Default | DB Column (@map) |
|-------|------|----------|---------|------------------|
| id | String | No | uuid() | - |
| userId | String | Yes | - | - |
| action | String | No | - | - |
| resource | String | Yes | - | - |
| resourceId | String | Yes | - | - |
| ipAddress | String | Yes | - | - |
| userAgent | String | Yes | - | - |
| metadata | Json | Yes | - | - |
| timestamp | DateTime | No | now() | - |

**Indexes:**
- Primary: `id`
- Index: `userId`
- Index: `timestamp`
- Index: `[resource, resourceId]` (composite)

---

## Enums

### Enum: Platform
| Value |
|-------|
| WEB |
| IOS |
| ANDROID |

---

### Enum: FamilyRole
| Value |
|-------|
| ADMIN |
| CAREGIVER |
| VIEWER |

---

### Enum: InvitationStatus
| Value |
|-------|
| PENDING |
| ACCEPTED |
| EXPIRED |
| CANCELLED |

---

### Enum: AppointmentType
| Value |
|-------|
| DOCTOR_VISIT |
| PHYSICAL_THERAPY |
| LAB_WORK |
| IMAGING |
| SPECIALIST |
| HOME_HEALTH |
| OTHER |

---

### Enum: AppointmentStatus
| Value |
|-------|
| SCHEDULED |
| CONFIRMED |
| COMPLETED |
| CANCELLED |
| NO_SHOW |

---

### Enum: MedicationForm
| Value |
|-------|
| TABLET |
| CAPSULE |
| LIQUID |
| INJECTION |
| PATCH |
| CREAM |
| INHALER |
| DROPS |
| OTHER |

---

### Enum: MedicationFrequency
| Value |
|-------|
| DAILY |
| TWICE_DAILY |
| THREE_TIMES_DAILY |
| FOUR_TIMES_DAILY |
| WEEKLY |
| AS_NEEDED |
| OTHER |

---

### Enum: MedicationLogStatus
| Value |
|-------|
| GIVEN |
| SKIPPED |
| MISSED |
| PENDING |

---

### Enum: ShiftStatus
| Value |
|-------|
| SCHEDULED |
| CONFIRMED |
| IN_PROGRESS |
| COMPLETED |
| CANCELLED |
| NO_SHOW |

---

### Enum: TimelineType
| Value |
|-------|
| NOTE |
| VITALS |
| SYMPTOM |
| INCIDENT |
| MOOD |
| MEAL |
| ACTIVITY |
| SLEEP |
| BATHROOM |
| MEDICATION_CHANGE |
| APPOINTMENT_SUMMARY |
| OTHER |

---

### Enum: Severity
| Value |
|-------|
| LOW |
| MEDIUM |
| HIGH |
| CRITICAL |

---

### Enum: DocumentType
| Value |
|-------|
| INSURANCE_CARD |
| PHOTO_ID |
| MEDICAL_RECORD |
| LAB_RESULT |
| PRESCRIPTION |
| POWER_OF_ATTORNEY |
| LIVING_WILL |
| DNR |
| OTHER |

---

### Enum: EmergencyType
| Value |
|-------|
| FALL |
| MEDICAL |
| MISSING |
| HOSPITALIZATION |
| OTHER |

---

### Enum: AlertStatus
| Value |
|-------|
| ACTIVE |
| ACKNOWLEDGED |
| RESOLVED |

---

### Enum: NotificationType
| Value |
|-------|
| MEDICATION_REMINDER |
| MEDICATION_MISSED |
| APPOINTMENT_REMINDER |
| SHIFT_REMINDER |
| SHIFT_HANDOFF |
| EMERGENCY_ALERT |
| FAMILY_INVITE |
| DOCUMENT_SHARED |
| TIMELINE_UPDATE |
| REFILL_NEEDED |
| REFILL_ALERT |
| GENERAL |
| CARE_RECIPIENT_DELETED |
| CARE_RECIPIENT_UPDATED |
| MEDICATION_DELETED |
| APPOINTMENT_DELETED |
| FAMILY_MEMBER_REMOVED |
| FAMILY_MEMBER_ROLE_CHANGED |
| FAMILY_DELETED |

---

## Relations Map

| Model | Field | Related To | Relation Type | FK Field | On Delete |
|-------|-------|------------|---------------|----------|-----------|
| User | sessions | Session | One-to-Many | - | - |
| User | familyMemberships | FamilyMember | One-to-Many | - | - |
| User | caregiverShifts | CaregiverShift | One-to-Many | - | - |
| User | medicationLogs | MedicationLog | One-to-Many | - | - |
| User | timelineEntries | TimelineEntry | One-to-Many | - | - |
| User | notifications | Notification | One-to-Many | - | - |
| User | emergencyAlerts | EmergencyAlert | One-to-Many | - | - |
| User | pushTokens | PushToken | One-to-Many | - | - |
| Session | user | User | Many-to-One | userId | Cascade |
| PushToken | user | User | Many-to-One | userId | Cascade |
| Family | members | FamilyMember | One-to-Many | - | - |
| Family | careRecipients | CareRecipient | One-to-Many | - | - |
| Family | documents | Document | One-to-Many | - | - |
| Family | invitations | FamilyInvitation | One-to-Many | - | - |
| FamilyMember | family | Family | Many-to-One | familyId | Cascade |
| FamilyMember | user | User | Many-to-One | userId | Cascade |
| FamilyInvitation | family | Family | Many-to-One | familyId | Cascade |
| CareRecipient | family | Family | Many-to-One | familyId | Cascade |
| CareRecipient | doctors | Doctor | One-to-Many | - | - |
| CareRecipient | medications | Medication | One-to-Many | - | - |
| CareRecipient | appointments | Appointment | One-to-Many | - | - |
| CareRecipient | caregiverShifts | CaregiverShift | One-to-Many | - | - |
| CareRecipient | timelineEntries | TimelineEntry | One-to-Many | - | - |
| CareRecipient | emergencyContacts | EmergencyContact | One-to-Many | - | - |
| CareRecipient | emergencyAlerts | EmergencyAlert | One-to-Many | - | - |
| Doctor | careRecipient | CareRecipient | Many-to-One | careRecipientId | Cascade |
| Doctor | appointments | Appointment | One-to-Many | - | - |
| EmergencyContact | careRecipient | CareRecipient | Many-to-One | careRecipientId | Cascade |
| Appointment | careRecipient | CareRecipient | Many-to-One | careRecipientId | Cascade |
| Appointment | doctor | Doctor | Many-to-One | doctorId | (none) |
| Appointment | transportAssignment | TransportAssignment | One-to-One | - | - |
| TransportAssignment | appointment | Appointment | One-to-One | appointmentId | Cascade |
| Medication | careRecipient | CareRecipient | Many-to-One | careRecipientId | Cascade |
| Medication | logs | MedicationLog | One-to-Many | - | - |
| MedicationLog | medication | Medication | Many-to-One | medicationId | Cascade |
| MedicationLog | givenBy | User | Many-to-One | givenById | (none) |
| CaregiverShift | careRecipient | CareRecipient | Many-to-One | careRecipientId | Cascade |
| CaregiverShift | caregiver | User | Many-to-One | caregiverId | (none) |
| TimelineEntry | careRecipient | CareRecipient | Many-to-One | careRecipientId | Cascade |
| TimelineEntry | createdBy | User | Many-to-One | createdById | (none) |
| Document | family | Family | Many-to-One | familyId | Cascade |
| EmergencyAlert | careRecipient | CareRecipient | Many-to-One | careRecipientId | Cascade |
| EmergencyAlert | createdBy | User | Many-to-One | createdById | (none) |
| Notification | user | User | Many-to-One | userId | Cascade |

---

## Notes

1. **No @map directives found** - All field names use camelCase and map directly to database columns
2. **No @@map directives found** - All table names match their model names
3. **All IDs use UUID** - Every model uses `String @id @default(uuid())`
4. **Cascade deletes** - Most child relations use `onDelete: Cascade` for automatic cleanup
5. **HIPAA compliance** - AuditLog model exists for tracking all actions
