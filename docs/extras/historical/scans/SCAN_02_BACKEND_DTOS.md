# Backend DTOs Map

**Generated:** 2026-01-20
**Total DTO Files:** 31
**Total DTO Classes:** 45+

---

## Summary by Module

| Module | Files | Classes |
|--------|-------|---------|
| Auth | 6 | 12 |
| User | 2 | 5 |
| Family | 5 | 6 |
| Care Recipient | 3 | 4 |
| Medications | 3 | 3 |
| Appointments | 3 | 4 |
| Caregiver Shifts | 2 | 2 |
| Documents | 2 | 3 |
| Emergency | 2 | 2 |
| Timeline | 1 | 1 |
| Events | 1 | 25+ interfaces |

---

## Auth Module

### File: `apps/api/src/auth/dto/login.dto.ts`

#### Class: LoginDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| email | string | No | @IsEmail() | @Transform (toLowerCase, trim) |
| password | string | No | @IsString() | - |
| rememberMe | boolean | Yes | @IsOptional(), @IsBoolean() | - |

---

### File: `apps/api/src/auth/dto/register.dto.ts`

#### Class: RegisterDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| fullName | string | No | @IsString(), @MinLength(2), @MaxLength(100) | - |
| email | string | No | @IsEmail() | @Transform (toLowerCase, trim) |
| password | string | No | @IsString(), @MinLength(12), @MaxLength(128), @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/) | - |
| phone | string | Yes | @IsOptional(), @IsPhoneNumber() | - |
| timezone | string | Yes | @IsOptional(), @IsString() | - |

---

### File: `apps/api/src/auth/dto/refresh.dto.ts`

#### Class: RefreshDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| refreshToken | string | No | @IsString() | - |

---

### File: `apps/api/src/auth/dto/refresh-token.dto.ts`

#### Class: RefreshTokenDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| refreshToken | string | Yes | @IsOptional(), @IsString() | - |

---

### File: `apps/api/src/auth/dto/verify-email.dto.ts`

#### Class: VerifyEmailDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| email | string | No | @IsEmail() | @Transform (toLowerCase, trim) |
| otp | string | No | @IsString(), @Length(6, 6) | - |

#### Class: ResendVerificationDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| email | string | No | @IsEmail() | @Transform (toLowerCase, trim) |

---

### File: `apps/api/src/auth/dto/password-reset.dto.ts`

#### Class: ForgotPasswordDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| email | string | No | @IsEmail() | @Transform (toLowerCase, trim) |

#### Class: ResetPasswordDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| token | string | No | @IsString() | - |
| newPassword | string | No | @IsString(), @MinLength(12), @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/) | - |

#### Class: ChangePasswordDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| currentPassword | string | No | @IsString() | - |
| newPassword | string | No | @IsString(), @MinLength(12), @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/) | - |

---

### File: `apps/api/src/auth/dto/auth-response.dto.ts`

**Note:** These are response DTOs (no validators - for Swagger docs only)

#### Class: TokensDto
| Property | Type | Optional |
|----------|------|----------|
| accessToken | string | No |
| refreshToken | string | No |

#### Class: AuthUserDto
| Property | Type | Optional |
|----------|------|----------|
| id | string | No |
| email | string | No |
| fullName | string | No |
| phone | string | Yes |
| avatarUrl | string | Yes |
| status | string | No |
| emailVerified | boolean | No |
| onboardingCompleted | boolean | No |
| families | any[] | No |

#### Class: LoginResponseDto
| Property | Type | Optional |
|----------|------|----------|
| accessToken | string | No |
| user | AuthUserDto | No |

#### Class: RegisterResponseDto
| Property | Type | Optional |
|----------|------|----------|
| message | string | No |
| user | { email: string; fullName: string } | No |

#### Class: VerifyEmailResponseDto
| Property | Type | Optional |
|----------|------|----------|
| message | string | No |
| accessToken | string | No |
| user | AuthUserDto | No |

#### Class: MessageResponseDto
| Property | Type | Optional |
|----------|------|----------|
| message | string | No |

#### Class: ErrorResponseDto
| Property | Type | Optional |
|----------|------|----------|
| statusCode | number | No |
| error | string | No |
| message | string | No |
| timestamp | string | No |
| path | string | No |

---

## User Module

### File: `apps/api/src/user/dto/create-user.dto.ts`

#### Class: CreateUserDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| fullName | string | No | @IsString(), @MinLength(2), @MaxLength(100) | - |
| email | string | No | @IsEmail() | @Transform (toLowerCase, trim) |
| password | string | No | @IsString(), @MinLength(12), @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/) | - |
| phone | string | Yes | @IsOptional(), @IsPhoneNumber() | - |
| timezone | string | Yes | @IsOptional(), @IsString() | - |

---

### File: `apps/api/src/user/dto/update-user.dto.ts`

#### Class: NotificationPreferences (Nested)
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| email | boolean | No | @IsBoolean() | - |
| push | boolean | No | @IsBoolean() | - |
| sms | boolean | No | @IsBoolean() | - |
| emergencyOnly | boolean | No | @IsBoolean() | - |

#### Class: DisplayPreferences (Nested)
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| theme | 'light' \| 'dark' \| 'system' | No | @IsIn(['light', 'dark', 'system']) | - |
| language | string | No | @IsString() | - |

#### Class: UserPreferences (Nested)
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| notifications | NotificationPreferences | No | @ValidateNested() | @Type(() => NotificationPreferences) |
| display | DisplayPreferences | No | @ValidateNested() | @Type(() => DisplayPreferences) |

#### Class: UpdateUserDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| fullName | string | Yes | @IsOptional(), @IsString(), @MinLength(2), @MaxLength(100) | - |
| phone | string | Yes | @IsOptional(), @IsPhoneNumber() | - |
| avatarUrl | string | Yes | @IsOptional(), @IsUrl() | - |
| timezone | string | Yes | @IsOptional(), @IsString() | - |
| preferences | UserPreferences | Yes | @IsOptional(), @IsObject(), @ValidateNested() | @Type(() => UserPreferences) |

---

## Family Module

### File: `apps/api/src/family/dto/create-family.dto.ts`

#### Class: CreateFamilyDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| name | string | No | @IsString(), @MaxLength(100) | - |
| description | string | Yes | @IsOptional(), @IsString(), @MaxLength(500) | - |
| avatarUrl | string | Yes | @IsOptional(), @IsUrl() | - |
| timezone | string | Yes | @IsOptional(), @IsString() | - |
| nickname | string | Yes | @IsOptional(), @IsString(), @MaxLength(50) | - |

---

### File: `apps/api/src/family/dto/update-family.dto.ts`

#### Class: UpdateFamilyDto
**Extends:** `PartialType(CreateFamilyDto)`

All properties from CreateFamilyDto become optional.

---

### File: `apps/api/src/family/dto/invite-member.dto.ts`

#### Class: InviteMemberDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| email | string | No | @IsEmail() | @Transform (toLowerCase, trim) |
| role | FamilyRole (enum) | No | @IsEnum(FamilyRole) | - |
| message | string | Yes | @IsOptional(), @IsString(), @MaxLength(500) | - |

**FamilyRole Enum Values:** ADMIN, CAREGIVER, VIEWER

---

### File: `apps/api/src/family/dto/update-member.dto.ts`

#### Class: MemberNotifications (Nested)
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| emergencies | boolean | No | @IsBoolean() | - |
| medications | boolean | No | @IsBoolean() | - |
| appointments | boolean | No | @IsBoolean() | - |
| shifts | boolean | No | @IsBoolean() | - |

#### Class: UpdateMemberDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| role | FamilyRole (enum) | Yes | @IsOptional(), @IsEnum(FamilyRole) | - |
| nickname | string | Yes | @IsOptional(), @IsString() | - |
| notifications | MemberNotifications | Yes | @IsOptional(), @IsObject(), @ValidateNested() | @Type(() => MemberNotifications) |

---

### File: `apps/api/src/family/dto/update-member-role.dto.ts`

#### Class: UpdateMemberRoleDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| role | 'ADMIN' \| 'CAREGIVER' \| 'VIEWER' | No | @IsEnum(['ADMIN', 'CAREGIVER', 'VIEWER']) | - |

---

## Care Recipient Module

### File: `apps/api/src/care-recipient/dto/create-care-recipient.dto.ts`

#### Enum: BloodType (Local)
| Value |
|-------|
| A_POSITIVE = 'A+' |
| A_NEGATIVE = 'A-' |
| B_POSITIVE = 'B+' |
| B_NEGATIVE = 'B-' |
| AB_POSITIVE = 'AB+' |
| AB_NEGATIVE = 'AB-' |
| O_POSITIVE = 'O+' |
| O_NEGATIVE = 'O-' |
| UNKNOWN = 'Unknown' |

#### Class: CreateCareRecipientDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| fullName | string | No | @IsString(), @MaxLength(100) | - |
| preferredName | string | Yes | @IsOptional(), @IsString(), @MaxLength(50) | - |
| dateOfBirth | string | Yes | @IsOptional(), @IsDateString() | - |
| bloodType | BloodType (enum) | Yes | @IsOptional(), @IsEnum(BloodType) | - |
| avatarUrl | string | Yes | @IsOptional(), @IsUrl() | - |
| allergies | string[] | Yes | @IsOptional(), @IsArray(), @IsString({ each: true }) | - |
| conditions | string[] | Yes | @IsOptional(), @IsArray(), @IsString({ each: true }) | - |
| notes | string | Yes | @IsOptional(), @IsString(), @MaxLength(1000) | - |
| preferredHospital | string | Yes | @IsOptional(), @IsString() | - |
| preferredHospitalAddress | string | Yes | @IsOptional(), @IsString() | - |
| insuranceProvider | string | Yes | @IsOptional(), @IsString() | - |
| insurancePolicyNumber | string | Yes | @IsOptional(), @IsString() | - |
| insuranceGroupNumber | string | Yes | @IsOptional(), @IsString() | - |

---

### File: `apps/api/src/care-recipient/dto/create-doctor.dto.ts`

#### Class: CreateDoctorDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| name | string | No | @IsString(), @MaxLength(100) | - |
| specialty | string | No | @IsString() | - |
| phone | string | No | @IsString() | - |
| fax | string | Yes | @IsOptional(), @IsString() | - |
| email | string | Yes | @IsOptional(), @IsEmail() | - |
| address | string | Yes | @IsOptional(), @IsString() | - |
| notes | string | Yes | @IsOptional(), @IsString(), @MaxLength(500) | - |

---

### File: `apps/api/src/care-recipient/dto/create-emergency-contact.dto.ts`

#### Class: CreateEmergencyContactDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| name | string | No | @IsString(), @MaxLength(100) | - |
| relationship | string | No | @IsString() | - |
| phone | string | No | @IsString() | - |
| email | string | Yes | @IsOptional(), @IsEmail() | - |
| isPrimary | boolean | Yes | @IsOptional(), @IsBoolean() | - |
| notes | string | Yes | @IsOptional(), @IsString(), @MaxLength(500) | - |

---

## Medications Module

### File: `apps/api/src/medications/dto/create-medication.dto.ts`

#### Class: CreateMedicationDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| name | string | No | @IsString(), @IsNotEmpty() | - |
| genericName | string | Yes | @IsOptional(), @IsString() | - |
| dosage | string | No | @IsString(), @IsNotEmpty() | - |
| form | MedicationForm (enum) | No | @IsEnum(MedicationForm) | - |
| frequency | MedicationFrequency (enum) | No | @IsEnum(MedicationFrequency) | - |
| timesPerDay | number | Yes | @IsOptional(), @IsInt(), @Min(1) | - |
| scheduledTimes | string[] | Yes | @IsOptional(), @IsArray(), @IsString({ each: true }) | - |
| instructions | string | Yes | @IsOptional(), @IsString() | - |
| prescribedBy | string | Yes | @IsOptional(), @IsString() | - |
| pharmacy | string | Yes | @IsOptional(), @IsString() | - |
| pharmacyPhone | string | Yes | @IsOptional(), @IsString() | - |
| currentSupply | number | Yes | @IsOptional(), @IsInt(), @Min(0) | - |
| refillAt | number | Yes | @IsOptional(), @IsInt(), @Min(0) | - |
| startDate | string | Yes | @IsOptional(), @IsDateString() | - |
| endDate | string | Yes | @IsOptional(), @IsDateString() | - |
| notes | string | Yes | @IsOptional(), @IsString() | - |

**MedicationForm Enum:** TABLET, CAPSULE, LIQUID, INJECTION, PATCH, CREAM, INHALER, DROPS, OTHER

**MedicationFrequency Enum:** DAILY, TWICE_DAILY, THREE_TIMES_DAILY, FOUR_TIMES_DAILY, WEEKLY, AS_NEEDED, OTHER

---

### File: `apps/api/src/medications/dto/update-medication.dto.ts`

#### Class: UpdateMedicationDto
**Extends:** `PartialType(CreateMedicationDto)`

| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| isActive | boolean | Yes | @IsOptional(), @IsBoolean() | - |
| *(all CreateMedicationDto properties)* | - | Yes | - | - |

---

### File: `apps/api/src/medications/dto/log-medication.dto.ts`

#### Class: LogMedicationDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| status | MedicationLogStatus (enum) | No | @IsEnum(MedicationLogStatus), @IsNotEmpty() | - |
| scheduledTime | string | No | @IsDateString(), @IsNotEmpty() | - |
| skipReason | string | Yes | @IsOptional(), @IsString() | - |
| notes | string | Yes | @IsOptional(), @IsString() | - |

**MedicationLogStatus Enum:** GIVEN, SKIPPED, MISSED, PENDING

---

## Appointments Module

### File: `apps/api/src/appointments/dto/create-appointment.dto.ts`

#### Enum: RecurrencePattern (Local)
| Value |
|-------|
| NONE = 'none' |
| DAILY = 'daily' |
| WEEKLY = 'weekly' |
| BIWEEKLY = 'biweekly' |
| MONTHLY = 'monthly' |

#### Class: CreateAppointmentDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| title | string | No | @IsString(), @IsNotEmpty() | - |
| type | AppointmentType (enum) | No | @IsEnum(AppointmentType) | - |
| doctorId | string | Yes | @IsOptional(), @IsUUID() | - |
| doctorName | string | Yes | @IsOptional(), @IsString() | - |
| location | string | Yes | @IsOptional(), @IsString() | - |
| address | string | Yes | @IsOptional(), @IsString() | - |
| startTime | string | Yes | @IsOptional(), @IsDateString() | - |
| endTime | string | Yes | @IsOptional(), @IsDateString() | - |
| dateTime | string | Yes | @IsOptional(), @IsDateString() | - |
| duration | number | Yes | @IsOptional(), @IsInt(), @Min(15) | - |
| notes | string | Yes | @IsOptional(), @IsString() | - |
| isRecurring | boolean | Yes | @IsOptional(), @IsBoolean() | - |
| recurrenceRule | string | Yes | @IsOptional(), @IsString() | - |
| recurrence | RecurrencePattern (enum) | Yes | @IsOptional(), @IsEnum(RecurrencePattern) | - |
| recurrenceEndDate | string | Yes | @IsOptional(), @IsDateString() | - |
| reminderMinutes | number[] | Yes | @IsOptional(), @IsArray(), @IsInt({ each: true }) | - |
| reminderBefore | string[] | Yes | @IsOptional(), @IsArray(), @IsString({ each: true }) | - |
| transportAssignedToId | string | Yes | @IsOptional(), @IsUUID() | - |

**AppointmentType Enum:** DOCTOR_VISIT, PHYSICAL_THERAPY, LAB_WORK, IMAGING, SPECIALIST, HOME_HEALTH, OTHER

---

### File: `apps/api/src/appointments/dto/update-appointment.dto.ts`

#### Class: UpdateAppointmentDto
**Extends:** `PartialType(CreateAppointmentDto)`

| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| status | AppointmentStatus (enum) | Yes | @IsOptional(), @IsEnum(AppointmentStatus) | - |
| *(all CreateAppointmentDto properties)* | - | Yes | - | - |

**AppointmentStatus Enum:** SCHEDULED, CONFIRMED, COMPLETED, CANCELLED, NO_SHOW

---

### File: `apps/api/src/appointments/dto/assign-transport.dto.ts`

#### Class: AssignTransportDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| assignedToId | string | No | @IsString() | - |
| notes | string | Yes | @IsOptional(), @IsString() | - |

---

## Caregiver Shifts Module

### File: `apps/api/src/caregiver-shifts/dto/create-shift.dto.ts`

#### Class: CreateShiftDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| startTime | string | No | @IsDateString(), @IsNotEmpty() | - |
| endTime | string | No | @IsDateString(), @IsNotEmpty() | - |
| caregiverId | string | No | @IsUUID(), @IsNotEmpty() | - |
| notes | string | Yes | @IsOptional(), @IsString() | - |

---

### File: `apps/api/src/caregiver-shifts/dto/check-out.dto.ts`

#### Class: CheckOutDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| notes | string | Yes | @IsOptional(), @IsString() | - |
| location | string | Yes | @IsOptional(), @IsString() | - |
| handoffNotes | string | Yes | @IsOptional(), @IsString() | - |

---

## Documents Module

### File: `apps/api/src/documents/dto/upload-document.dto.ts`

#### Enum: DocumentCategory (Local)
| Value |
|-------|
| MEDICAL = 'medical' |
| INSURANCE = 'insurance' |
| LEGAL = 'legal' |
| IDENTIFICATION = 'identification' |
| FINANCIAL = 'financial' |
| OTHER = 'other' |

#### Class: UploadDocumentDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| name | string | Yes | @IsOptional(), @IsString() | - |
| title | string | No | @IsString(), @IsNotEmpty() | - |
| type | DocumentType (enum) | Yes | @IsOptional(), @IsEnum(DocumentType) | - |
| category | DocumentCategory (enum) | Yes | @IsOptional(), @IsEnum(DocumentCategory) | - |
| description | string | Yes | @IsOptional(), @IsString() | - |
| expiresAt | string | Yes | @IsOptional(), @IsDateString() | - |
| expirationDate | string | Yes | @IsOptional(), @IsDateString() | - |
| notes | string | Yes | @IsOptional(), @IsString() | - |

**DocumentType Enum (Prisma):** INSURANCE_CARD, PHOTO_ID, MEDICAL_RECORD, LAB_RESULT, PRESCRIPTION, POWER_OF_ATTORNEY, LIVING_WILL, DNR, OTHER

---

### File: `apps/api/src/documents/dto/update-document.dto.ts`

#### Class: UpdateDocumentDto
**Extends:** `PartialType(UploadDocumentDto)`

All properties from UploadDocumentDto become optional.

---

## Emergency Module

### File: `apps/api/src/emergency/dto/create-emergency-alert.dto.ts`

#### Class: CreateEmergencyAlertDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| type | EmergencyType (enum) | No | @IsEnum(EmergencyType), @IsNotEmpty() | - |
| title | string | No | @IsString(), @IsNotEmpty() | - |
| description | string | No | @IsString() | - |
| location | string | Yes | @IsOptional(), @IsString() | - |

**EmergencyType Enum:** FALL, MEDICAL, MISSING, HOSPITALIZATION, OTHER

---

### File: `apps/api/src/emergency/dto/resolve-alert.dto.ts`

#### Class: ResolveAlertDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| resolutionNotes | string | Yes | @IsOptional(), @IsString() | - |

---

## Timeline Module

### File: `apps/api/src/timeline/dto/create-timeline-entry.dto.ts`

#### Class: CreateTimelineEntryDto
| Property | Type | Optional | Validators | Transforms |
|----------|------|----------|------------|------------|
| type | TimelineType (enum) | No | @IsEnum(TimelineType), @IsNotEmpty() | - |
| title | string | No | @IsString(), @IsNotEmpty() | - |
| description | string | Yes | @IsOptional(), @IsString() | - |
| severity | Severity (enum) | Yes | @IsOptional(), @IsEnum(Severity) | - |
| vitals | object | Yes | @IsOptional(), @IsObject() | - |
| occurredAt | string | Yes | @IsOptional(), @IsDateString() | - |
| attachments | string[] | Yes | @IsOptional(), @IsArray(), @IsString({ each: true }) | - |

**Vitals Object Structure:**
```typescript
{
  bloodPressure?: string;
  heartRate?: number;
  temperature?: number;
  bloodSugar?: number;
  oxygenLevel?: number;
  weight?: number;
}
```

**TimelineType Enum:** NOTE, VITALS, SYMPTOM, INCIDENT, MOOD, MEAL, ACTIVITY, SLEEP, BATHROOM, MEDICATION_CHANGE, APPOINTMENT_SUMMARY, OTHER

**Severity Enum:** LOW, MEDIUM, HIGH, CRITICAL

---

## Events Module

### File: `apps/api/src/events/dto/events.dto.ts`

**Note:** This file contains TypeScript interfaces for event payloads (CloudEvents specification), not class-validator DTOs.

#### Interface: BaseEvent<T>
| Property | Type | Optional |
|----------|------|----------|
| id | string | No |
| type | string | No |
| source | string | No |
| timestamp | string | No |
| specVersion | '1.0' | No |
| data | T | No |
| correlationId | string | Yes |
| causedBy | string | Yes |
| familyId | string | Yes |
| careRecipientId | string | Yes |

#### Event Payload Interfaces

**Medication Events:**
- MedicationCreatedPayload
- MedicationLoggedPayload
- MedicationDuePayload
- MedicationRefillPayload
- MedicationDeletedPayload

**Appointment Events:**
- AppointmentCreatedPayload
- AppointmentReminderPayload
- AppointmentCancelledPayload
- AppointmentDeletedPayload

**Emergency Events:**
- EmergencyAlertCreatedPayload
- EmergencyAlertResolvedPayload

**Shift Events:**
- ShiftStartedPayload
- ShiftEndedPayload
- ShiftHandoffPayload

**Family Events:**
- FamilyMemberInvitedPayload
- FamilyMemberJoinedPayload
- FamilyMemberRemovedPayload
- FamilyMemberRoleUpdatedPayload
- FamilyInvitationCancelledPayload
- FamilyDeletedPayload

**Care Recipient Events:**
- CareRecipientDeletedPayload
- CareRecipientUpdatedPayload

**Timeline Events:**
- TimelineEntryCreatedPayload

**Notification Events:**
- PushNotificationPayload
- EmailNotificationPayload
- SmsNotificationPayload

---

## Validators Summary

### Common Validators Used

| Validator | Count | Description |
|-----------|-------|-------------|
| @IsString() | 60+ | String validation |
| @IsOptional() | 50+ | Optional field |
| @IsEmail() | 8 | Email format |
| @IsEnum() | 15 | Enum value |
| @IsDateString() | 12 | ISO date string |
| @IsBoolean() | 10 | Boolean value |
| @IsInt() | 6 | Integer value |
| @IsArray() | 8 | Array type |
| @IsNotEmpty() | 12 | Non-empty value |
| @MinLength() | 8 | Minimum string length |
| @MaxLength() | 15 | Maximum string length |
| @Min() | 5 | Minimum number value |
| @IsUUID() | 5 | UUID format |
| @IsUrl() | 3 | URL format |
| @IsPhoneNumber() | 3 | Phone number format |
| @Matches() | 4 | Regex pattern |
| @ValidateNested() | 4 | Nested object validation |
| @Length() | 1 | Exact length |
| @IsIn() | 1 | Value in list |
| @IsObject() | 3 | Object type |

### Transforms Used

| Transform | Count | Description |
|-----------|-------|-------------|
| @Transform (toLowerCase, trim) | 7 | Normalize email input |
| @Type() | 4 | Class transformation for nested objects |

---

## Local Enums Defined in DTOs

| File | Enum | Values |
|------|------|--------|
| create-care-recipient.dto.ts | BloodType | A+, A-, B+, B-, AB+, AB-, O+, O-, Unknown |
| create-appointment.dto.ts | RecurrencePattern | none, daily, weekly, biweekly, monthly |
| upload-document.dto.ts | DocumentCategory | medical, insurance, legal, identification, financial, other |

---

## Extended DTOs (PartialType)

| DTO | Extends | Additional Properties |
|-----|---------|----------------------|
| UpdateMedicationDto | PartialType(CreateMedicationDto) | isActive |
| UpdateAppointmentDto | PartialType(CreateAppointmentDto) | status |
| UpdateDocumentDto | PartialType(UploadDocumentDto) | none |
| UpdateFamilyDto | PartialType(CreateFamilyDto) | none |
