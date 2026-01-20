# Backend API Routes Map

**Generated:** 2026-01-20
**Total Controller Files:** 13
**Total Controller Classes:** 15
**Total Routes:** 96

---

## Summary

| Controller | Base Path | Routes | Auth |
|------------|-----------|--------|------|
| AuthController | /auth | 14 | Mixed |
| FamilyController | /families | 14 | JwtAuthGuard (with Public exceptions) |
| CareRecipientController | / (root) | 12 | ApiBearerAuth |
| MedicationsController | /care-recipients/:careRecipientId/medications | 8 | ApiBearerAuth |
| MedicationLogsController | /medications | 1 | ApiBearerAuth |
| AppointmentsController | /care-recipients/:careRecipientId/appointments | 10 | ApiBearerAuth |
| CaregiverShiftsController | /care-recipients/:careRecipientId/shifts | 8 | ApiBearerAuth |
| MyShiftsController | /my-shifts | 1 | ApiBearerAuth |
| TimelineController | /care-recipients/:careRecipientId/timeline | 6 | ApiBearerAuth |
| DocumentsController | /families/:familyId/documents | 6 | ApiBearerAuth |
| EmergencyController | /care-recipients/:careRecipientId/emergency | 6 | ApiBearerAuth |
| NotificationsController | /notifications | 6 | ApiBearerAuth |
| ChatController | /chat | 4 | ApiBearerAuth |
| HealthController | /health | 3 | Public |
| MetricsController | /metrics | 1 | Public (excluded from Swagger) |

---

## Auth Controller

**File:** `apps/api/src/auth/controller/auth.controller.ts`
**Base Path:** `@Controller('auth')`
**API Tags:** `auth`

| Method | Decorator | Path | Full Route | Params | Query | Body DTO | Auth |
|--------|-----------|------|------------|--------|-------|----------|------|
| register | @Post() | register | POST /auth/register | - | - | RegisterDto | @Public(), @Throttle(5/60s) |
| verifyEmail | @Post() | verify-email | POST /auth/verify-email | - | - | VerifyEmailDto | @Public(), @Throttle(5/60s) |
| resendVerification | @Post() | resend-verification | POST /auth/resend-verification | - | - | ResendVerificationDto | @Public(), @Throttle(3/60s) |
| login | @Post() | login | POST /auth/login | - | - | LoginDto | @Public(), @Throttle(10/60s) |
| refresh | @Post() | refresh | POST /auth/refresh | - | - | RefreshTokenDto | @Public() |
| logout | @Post() | logout | POST /auth/logout | - | - | RefreshTokenDto | @UseGuards(JwtAuthGuard) |
| logoutAll | @Post() | logout-all | POST /auth/logout-all | - | - | - | @UseGuards(JwtAuthGuard) |
| forgotPassword | @Post() | forgot-password | POST /auth/forgot-password | - | - | ForgotPasswordDto | @Public(), @Throttle(3/60s) |
| verifyResetToken | @Get() | verify-reset-token/:token | GET /auth/verify-reset-token/:token | token: string | - | - | @Public(), @Throttle(10/60s) |
| resetPassword | @Post() | reset-password | POST /auth/reset-password | - | - | ResetPasswordDto | @Public(), @Throttle(5/60s) |
| changePassword | @Post() | change-password | POST /auth/change-password | - | - | ChangePasswordDto | @UseGuards(JwtAuthGuard) |
| getSessions | @Get() | sessions | GET /auth/sessions | - | - | - | @UseGuards(JwtAuthGuard) |
| getProfile | @Get() | me | GET /auth/me | - | - | - | @UseGuards(JwtAuthGuard) |
| completeOnboarding | @Post() | complete-onboarding | POST /auth/complete-onboarding | - | - | - | @UseGuards(JwtAuthGuard) |

---

## Family Controller

**File:** `apps/api/src/family/family.controller.ts`
**Base Path:** `@Controller('families')`
**Global Guard:** `@UseGuards(JwtAuthGuard)`
**API Tags:** `Families`

| Method | Decorator | Path | Full Route | Params | Query | Body DTO | Auth |
|--------|-----------|------|------------|--------|-------|----------|------|
| createFamily | @Post() | - | POST /families | - | - | CreateFamilyDto | JwtAuthGuard |
| getMyFamilies | @Get() | - | GET /families | - | - | - | JwtAuthGuard |
| getFamily | @Get() | :familyId | GET /families/:familyId | familyId: string | - | - | JwtAuthGuard |
| updateFamily | @Patch() | :familyId | PATCH /families/:familyId | familyId: string | - | UpdateFamilyDto | JwtAuthGuard |
| deleteFamily | @Delete() | :familyId | DELETE /families/:familyId | familyId: string | - | - | JwtAuthGuard |
| inviteMember | @Post() | :familyId/invite | POST /families/:familyId/invite | familyId: string | - | InviteMemberDto | JwtAuthGuard |
| getInvitationDetails | @Get() | invitations/:token/details | GET /families/invitations/:token/details | token: string | - | - | @Public() |
| acceptInvitation | @Post() | invitations/:token/accept | POST /families/invitations/:token/accept | token: string | - | - | JwtAuthGuard |
| declineInvitation | @Post() | invitations/:token/decline | POST /families/invitations/:token/decline | token: string | - | - | @Public() |
| updateMemberRole | @Patch() | :familyId/members/:memberId/role | PATCH /families/:familyId/members/:memberId/role | familyId: string, memberId: string | - | UpdateMemberRoleDto | JwtAuthGuard |
| removeMember | @Delete() | :familyId/members/:memberId | DELETE /families/:familyId/members/:memberId | familyId: string, memberId: string | - | - | JwtAuthGuard |
| resetMemberPassword | @Post() | :familyId/members/:userId/reset-password | POST /families/:familyId/members/:userId/reset-password | familyId: string, userId: string | - | - | JwtAuthGuard |
| cancelInvitation | @Delete() | invitations/:invitationId | DELETE /families/invitations/:invitationId | invitationId: string | - | - | JwtAuthGuard |
| resendInvitation | @Post() | invitations/:invitationId/resend | POST /families/invitations/:invitationId/resend | invitationId: string | - | - | JwtAuthGuard |

---

## Care Recipient Controller

**File:** `apps/api/src/care-recipient/care-recipient.controller.ts`
**Base Path:** `@Controller()` (root - no prefix)
**API Tags:** `Care Recipients`

| Method | Decorator | Path | Full Route | Params | Query | Body DTO | Auth |
|--------|-----------|------|------------|--------|-------|----------|------|
| create | @Post() | families/:familyId/care-recipients | POST /families/:familyId/care-recipients | familyId: string (UUID) | - | CreateCareRecipientDto | ApiBearerAuth |
| findAll | @Get() | families/:familyId/care-recipients | GET /families/:familyId/care-recipients | familyId: string (UUID) | - | - | ApiBearerAuth |
| findOne | @Get() | care-recipients/:id | GET /care-recipients/:id | id: string (UUID) | - | - | ApiBearerAuth |
| update | @Patch() | care-recipients/:id | PATCH /care-recipients/:id | id: string (UUID) | - | Partial\<CreateCareRecipientDto\> | ApiBearerAuth |
| remove | @Delete() | care-recipients/:id | DELETE /care-recipients/:id | id: string (UUID) | - | - | ApiBearerAuth |
| addDoctor | @Post() | care-recipients/:careRecipientId/doctors | POST /care-recipients/:careRecipientId/doctors | careRecipientId: string (UUID) | - | CreateDoctorDto | ApiBearerAuth |
| getDoctors | @Get() | care-recipients/:careRecipientId/doctors | GET /care-recipients/:careRecipientId/doctors | careRecipientId: string (UUID) | - | - | ApiBearerAuth |
| updateDoctor | @Patch() | doctors/:doctorId | PATCH /doctors/:doctorId | doctorId: string (UUID) | - | Partial\<CreateDoctorDto\> | ApiBearerAuth |
| deleteDoctor | @Delete() | doctors/:doctorId | DELETE /doctors/:doctorId | doctorId: string (UUID) | - | - | ApiBearerAuth |
| addEmergencyContact | @Post() | care-recipients/:careRecipientId/emergency-contacts | POST /care-recipients/:careRecipientId/emergency-contacts | careRecipientId: string (UUID) | - | CreateEmergencyContactDto | ApiBearerAuth |
| getEmergencyContacts | @Get() | care-recipients/:careRecipientId/emergency-contacts | GET /care-recipients/:careRecipientId/emergency-contacts | careRecipientId: string (UUID) | - | - | ApiBearerAuth |
| deleteEmergencyContact | @Delete() | emergency-contacts/:contactId | DELETE /emergency-contacts/:contactId | contactId: string (UUID) | - | - | ApiBearerAuth |

---

## Medications Controller

**File:** `apps/api/src/medications/medications.controller.ts`
**Base Path:** `@Controller('care-recipients/:careRecipientId/medications')`
**API Tags:** `Medications`

| Method | Decorator | Path | Full Route | Params | Query | Body DTO | Auth |
|--------|-----------|------|------------|--------|-------|----------|------|
| create | @Post() | - | POST /care-recipients/:careRecipientId/medications | careRecipientId: string (UUID) | - | CreateMedicationDto | ApiBearerAuth |
| findAll | @Get() | - | GET /care-recipients/:careRecipientId/medications | careRecipientId: string (UUID) | activeOnly?: string | - | ApiBearerAuth |
| getTodaySchedule | @Get() | schedule/today | GET /care-recipients/:careRecipientId/medications/schedule/today | careRecipientId: string (UUID) | - | - | ApiBearerAuth |
| findOne | @Get() | :id | GET /care-recipients/:careRecipientId/medications/:id | careRecipientId: string (UUID), id: string (UUID) | - | - | ApiBearerAuth |
| getLogs | @Get() | :id/logs | GET /care-recipients/:careRecipientId/medications/:id/logs | careRecipientId: string (UUID), id: string (UUID) | startDate?, endDate?, limit? | - | ApiBearerAuth |
| update | @Patch() | :id | PATCH /care-recipients/:careRecipientId/medications/:id | careRecipientId: string (UUID), id: string (UUID) | - | UpdateMedicationDto | ApiBearerAuth |
| deactivate | @Patch() | :id/deactivate | PATCH /care-recipients/:careRecipientId/medications/:id/deactivate | careRecipientId: string (UUID), id: string (UUID) | - | - | ApiBearerAuth |
| delete | @Delete() | :id | DELETE /care-recipients/:careRecipientId/medications/:id | careRecipientId: string (UUID), id: string (UUID) | - | - | ApiBearerAuth |

---

## Medication Logs Controller

**File:** `apps/api/src/medications/medications.controller.ts`
**Base Path:** `@Controller('medications')`
**API Tags:** `Medication Logs`

| Method | Decorator | Path | Full Route | Params | Query | Body DTO | Auth |
|--------|-----------|------|------------|--------|-------|----------|------|
| logMedication | @Post() | :medicationId/log | POST /medications/:medicationId/log | medicationId: string (UUID) | - | LogMedicationDto | ApiBearerAuth |

---

## Appointments Controller

**File:** `apps/api/src/appointments/appointments.controller.ts`
**Base Path:** `@Controller('care-recipients/:careRecipientId/appointments')`
**API Tags:** `Appointments`

| Method | Decorator | Path | Full Route | Params | Query | Body DTO | Auth |
|--------|-----------|------|------------|--------|-------|----------|------|
| create | @Post() | - | POST /care-recipients/:careRecipientId/appointments | careRecipientId: string (UUID) | - | CreateAppointmentDto | ApiBearerAuth |
| findAll | @Get() | - | GET /care-recipients/:careRecipientId/appointments | careRecipientId: string (UUID) | startDate?, endDate?, status? | - | ApiBearerAuth |
| getUpcoming | @Get() | upcoming | GET /care-recipients/:careRecipientId/appointments/upcoming | careRecipientId: string (UUID) | days?: string | - | ApiBearerAuth |
| getForDay | @Get() | day | GET /care-recipients/:careRecipientId/appointments/day | careRecipientId: string (UUID) | date: string | - | ApiBearerAuth |
| findOne | @Get() | :id | GET /care-recipients/:careRecipientId/appointments/:id | careRecipientId: string (UUID), id: string (UUID) | - | - | ApiBearerAuth |
| update | @Patch() | :id | PATCH /care-recipients/:careRecipientId/appointments/:id | careRecipientId: string (UUID), id: string (UUID) | - | UpdateAppointmentDto | ApiBearerAuth |
| cancel | @Patch() | :id/cancel | PATCH /care-recipients/:careRecipientId/appointments/:id/cancel | careRecipientId: string (UUID), id: string (UUID) | - | - | ApiBearerAuth |
| assignTransport | @Post() | :id/transport | POST /care-recipients/:careRecipientId/appointments/:id/transport | careRecipientId: string (UUID), id: string (UUID) | - | AssignTransportDto | ApiBearerAuth |
| confirmTransport | @Post() | :id/transport/confirm | POST /care-recipients/:careRecipientId/appointments/:id/transport/confirm | careRecipientId: string (UUID), id: string (UUID) | - | - | ApiBearerAuth |
| delete | @Delete() | :id | DELETE /care-recipients/:careRecipientId/appointments/:id | careRecipientId: string (UUID), id: string (UUID) | - | - | ApiBearerAuth |

---

## Caregiver Shifts Controller

**File:** `apps/api/src/caregiver-shifts/caregiver-shifts.controller.ts`
**Base Path:** `@Controller('care-recipients/:careRecipientId/shifts')`
**API Tags:** `Caregiver Shifts`

| Method | Decorator | Path | Full Route | Params | Query | Body DTO | Auth |
|--------|-----------|------|------------|--------|-------|----------|------|
| create | @Post() | - | POST /care-recipients/:careRecipientId/shifts | careRecipientId: string (UUID) | - | CreateShiftDto | ApiBearerAuth |
| getCurrentShift | @Get() | current | GET /care-recipients/:careRecipientId/shifts/current | careRecipientId: string (UUID) | - | - | ApiBearerAuth |
| getUpcoming | @Get() | upcoming | GET /care-recipients/:careRecipientId/shifts/upcoming | careRecipientId: string (UUID) | days?: string | - | ApiBearerAuth |
| getForDay | @Get() | day | GET /care-recipients/:careRecipientId/shifts/day | careRecipientId: string (UUID) | date: string | - | ApiBearerAuth |
| checkIn | @Post() | :id/checkin | POST /care-recipients/:careRecipientId/shifts/:id/checkin | careRecipientId: string (UUID), id: string (UUID) | - | - | ApiBearerAuth |
| checkOut | @Post() | :id/checkout | POST /care-recipients/:careRecipientId/shifts/:id/checkout | careRecipientId: string (UUID), id: string (UUID) | - | CheckOutDto | ApiBearerAuth |
| confirmShift | @Post() | :id/confirm | POST /care-recipients/:careRecipientId/shifts/:id/confirm | careRecipientId: string (UUID), id: string (UUID) | - | - | ApiBearerAuth |
| cancelShift | @Patch() | :id/cancel | PATCH /care-recipients/:careRecipientId/shifts/:id/cancel | careRecipientId: string (UUID), id: string (UUID) | - | - | ApiBearerAuth |

---

## My Shifts Controller

**File:** `apps/api/src/caregiver-shifts/caregiver-shifts.controller.ts`
**Base Path:** `@Controller('my-shifts')`
**API Tags:** `My Shifts`

| Method | Decorator | Path | Full Route | Params | Query | Body DTO | Auth |
|--------|-----------|------|------------|--------|-------|----------|------|
| getMyShifts | @Get() | - | GET /my-shifts | - | upcomingOnly?: string | - | ApiBearerAuth |

---

## Timeline Controller

**File:** `apps/api/src/timeline/timeline.controller.ts`
**Base Path:** `@Controller('care-recipients/:careRecipientId/timeline')`
**API Tags:** `Timeline`

| Method | Decorator | Path | Full Route | Params | Query | Body DTO | Auth |
|--------|-----------|------|------------|--------|-------|----------|------|
| create | @Post() | - | POST /care-recipients/:careRecipientId/timeline | careRecipientId: string (UUID) | - | CreateTimelineEntryDto | ApiBearerAuth |
| findAll | @Get() | - | GET /care-recipients/:careRecipientId/timeline | careRecipientId: string (UUID) | type?, startDate?, endDate?, limit?, offset? | - | ApiBearerAuth |
| getRecentVitals | @Get() | vitals | GET /care-recipients/:careRecipientId/timeline/vitals | careRecipientId: string (UUID) | days?: string | - | ApiBearerAuth |
| findOne | @Get() | :id | GET /care-recipients/:careRecipientId/timeline/:id | careRecipientId: string (UUID), id: string (UUID) | - | - | ApiBearerAuth |
| update | @Patch() | :id | PATCH /care-recipients/:careRecipientId/timeline/:id | careRecipientId: string (UUID), id: string (UUID) | - | Partial\<CreateTimelineEntryDto\> | ApiBearerAuth |
| remove | @Delete() | :id | DELETE /care-recipients/:careRecipientId/timeline/:id | careRecipientId: string (UUID), id: string (UUID) | - | - | ApiBearerAuth |

---

## Documents Controller

**File:** `apps/api/src/documents/documents.controller.ts`
**Base Path:** `@Controller('families/:familyId/documents')`
**API Tags:** `Documents`

| Method | Decorator | Path | Full Route | Params | Query | Body DTO | Auth |
|--------|-----------|------|------------|--------|-------|----------|------|
| create | @Post() | - | POST /families/:familyId/documents | familyId: string (UUID) | - | UploadDocumentDto + { s3Key, mimeType, sizeBytes } | ApiBearerAuth |
| findAll | @Get() | - | GET /families/:familyId/documents | familyId: string (UUID) | type?: string | - | ApiBearerAuth |
| getExpiring | @Get() | expiring | GET /families/:familyId/documents/expiring | familyId: string (UUID) | days?: string | - | ApiBearerAuth |
| findOne | @Get() | :id | GET /families/:familyId/documents/:id | familyId: string (UUID), id: string (UUID) | - | - | ApiBearerAuth |
| update | @Patch() | :id | PATCH /families/:familyId/documents/:id | familyId: string (UUID), id: string (UUID) | - | UpdateDocumentDto | ApiBearerAuth |
| remove | @Delete() | :id | DELETE /families/:familyId/documents/:id | familyId: string (UUID), id: string (UUID) | - | - | ApiBearerAuth |

---

## Emergency Controller

**File:** `apps/api/src/emergency/emergency.controller.ts`
**Base Path:** `@Controller('care-recipients/:careRecipientId/emergency')`
**API Tags:** `Emergency`

| Method | Decorator | Path | Full Route | Params | Query | Body DTO | Auth |
|--------|-----------|------|------------|--------|-------|----------|------|
| getEmergencyInfo | @Get() | info | GET /care-recipients/:careRecipientId/emergency/info | careRecipientId: string (UUID) | - | - | ApiBearerAuth |
| createAlert | @Post() | alerts | POST /care-recipients/:careRecipientId/emergency/alerts | careRecipientId: string (UUID) | - | CreateEmergencyAlertDto | ApiBearerAuth |
| getActiveAlerts | @Get() | alerts | GET /care-recipients/:careRecipientId/emergency/alerts | careRecipientId: string (UUID) | - | - | ApiBearerAuth |
| getAlertHistory | @Get() | alerts/history | GET /care-recipients/:careRecipientId/emergency/alerts/history | careRecipientId: string (UUID) | limit?: string | - | ApiBearerAuth |
| acknowledgeAlert | @Post() | alerts/:alertId/acknowledge | POST /care-recipients/:careRecipientId/emergency/alerts/:alertId/acknowledge | careRecipientId: string (UUID), alertId: string (UUID) | - | - | ApiBearerAuth |
| resolveAlert | @Post() | alerts/:alertId/resolve | POST /care-recipients/:careRecipientId/emergency/alerts/:alertId/resolve | careRecipientId: string (UUID), alertId: string (UUID) | - | ResolveAlertDto | ApiBearerAuth |

---

## Notifications Controller

**File:** `apps/api/src/notifications/notifications.controller.ts`
**Base Path:** `@Controller('notifications')`
**API Tags:** `Notifications`

| Method | Decorator | Path | Full Route | Params | Query | Body DTO | Auth |
|--------|-----------|------|------------|--------|-------|----------|------|
| findAll | @Get() | - | GET /notifications | - | unreadOnly?: string, limit?: string | - | ApiBearerAuth |
| getUnreadCount | @Get() | unread/count | GET /notifications/unread/count | - | - | - | ApiBearerAuth |
| markAsRead | @Patch() | :notificationId/read | PATCH /notifications/:notificationId/read | notificationId: string | - | - | ApiBearerAuth |
| markAllAsRead | @Patch() | read/all | PATCH /notifications/read/all | - | - | - | ApiBearerAuth |
| registerPushToken | @Post() | push-token | POST /notifications/push-token | - | - | { token: string; platform: Platform } | ApiBearerAuth |
| removePushToken | @Delete() | push-token | DELETE /notifications/push-token | - | - | { token: string } | ApiBearerAuth |

---

## Chat Controller

**File:** `apps/api/src/chat/controller/chat.controller.ts`
**Base Path:** `@Controller('chat')`
**API Tags:** `Chat`

| Method | Decorator | Path | Full Route | Params | Query | Body DTO | Auth |
|--------|-----------|------|------------|--------|-------|----------|------|
| getUserToken | @Get() | token | GET /chat/token | - | - | - | ApiBearerAuth |
| createFamilyChannel | @Post() | family/:familyId/channel | POST /chat/family/:familyId/channel | familyId: string | - | { familyName: string; memberIds: string[] } | ApiBearerAuth |
| createTopicChannel | @Post() | family/:familyId/topic | POST /chat/family/:familyId/topic | familyId: string | - | { topic: string; topicName: string; memberIds: string[] } | ApiBearerAuth |
| getUserChannels | @Get() | channels | GET /chat/channels | - | - | - | ApiBearerAuth |

---

## Health Controller

**File:** `apps/api/src/health/health.controller.ts`
**Base Path:** `@Controller('health')`
**API Tags:** `Health`

| Method | Decorator | Path | Full Route | Params | Query | Body DTO | Auth |
|--------|-----------|------|------------|--------|-------|----------|------|
| check | @Get() | - | GET /health | - | - | - | Public |
| ready | @Get() | ready | GET /health/ready | - | - | - | Public |
| live | @Get() | live | GET /health/live | - | - | - | Public |

---

## Metrics Controller

**File:** `apps/api/src/metrics/metrics.controller.ts`
**Base Path:** `@Controller('metrics')`
**API Tags:** Excluded from Swagger (@ApiExcludeController)

| Method | Decorator | Path | Full Route | Params | Query | Body DTO | Auth |
|--------|-----------|------|------------|--------|-------|----------|------|
| getMetrics | @Get() | - | GET /metrics | - | - | - | Public |

---

## Complete Route List (Sorted Alphabetically)

| Full Route | Method | Controller | Handler | Auth Required |
|------------|--------|------------|---------|---------------|
| DELETE /care-recipients/:careRecipientId/appointments/:id | DELETE | AppointmentsController | delete | Yes |
| DELETE /care-recipients/:careRecipientId/medications/:id | DELETE | MedicationsController | delete | Yes |
| DELETE /care-recipients/:careRecipientId/timeline/:id | DELETE | TimelineController | remove | Yes |
| DELETE /care-recipients/:id | DELETE | CareRecipientController | remove | Yes |
| DELETE /doctors/:doctorId | DELETE | CareRecipientController | deleteDoctor | Yes |
| DELETE /emergency-contacts/:contactId | DELETE | CareRecipientController | deleteEmergencyContact | Yes |
| DELETE /families/:familyId | DELETE | FamilyController | deleteFamily | Yes |
| DELETE /families/:familyId/documents/:id | DELETE | DocumentsController | remove | Yes |
| DELETE /families/:familyId/members/:memberId | DELETE | FamilyController | removeMember | Yes |
| DELETE /families/invitations/:invitationId | DELETE | FamilyController | cancelInvitation | Yes |
| DELETE /notifications/push-token | DELETE | NotificationsController | removePushToken | Yes |
| GET /auth/me | GET | AuthController | getProfile | Yes |
| GET /auth/sessions | GET | AuthController | getSessions | Yes |
| GET /auth/verify-reset-token/:token | GET | AuthController | verifyResetToken | No |
| GET /care-recipients/:careRecipientId/appointments | GET | AppointmentsController | findAll | Yes |
| GET /care-recipients/:careRecipientId/appointments/:id | GET | AppointmentsController | findOne | Yes |
| GET /care-recipients/:careRecipientId/appointments/day | GET | AppointmentsController | getForDay | Yes |
| GET /care-recipients/:careRecipientId/appointments/upcoming | GET | AppointmentsController | getUpcoming | Yes |
| GET /care-recipients/:careRecipientId/doctors | GET | CareRecipientController | getDoctors | Yes |
| GET /care-recipients/:careRecipientId/emergency-contacts | GET | CareRecipientController | getEmergencyContacts | Yes |
| GET /care-recipients/:careRecipientId/emergency/alerts | GET | EmergencyController | getActiveAlerts | Yes |
| GET /care-recipients/:careRecipientId/emergency/alerts/history | GET | EmergencyController | getAlertHistory | Yes |
| GET /care-recipients/:careRecipientId/emergency/info | GET | EmergencyController | getEmergencyInfo | Yes |
| GET /care-recipients/:careRecipientId/medications | GET | MedicationsController | findAll | Yes |
| GET /care-recipients/:careRecipientId/medications/:id | GET | MedicationsController | findOne | Yes |
| GET /care-recipients/:careRecipientId/medications/:id/logs | GET | MedicationsController | getLogs | Yes |
| GET /care-recipients/:careRecipientId/medications/schedule/today | GET | MedicationsController | getTodaySchedule | Yes |
| GET /care-recipients/:careRecipientId/shifts/current | GET | CaregiverShiftsController | getCurrentShift | Yes |
| GET /care-recipients/:careRecipientId/shifts/day | GET | CaregiverShiftsController | getForDay | Yes |
| GET /care-recipients/:careRecipientId/shifts/upcoming | GET | CaregiverShiftsController | getUpcoming | Yes |
| GET /care-recipients/:careRecipientId/timeline | GET | TimelineController | findAll | Yes |
| GET /care-recipients/:careRecipientId/timeline/:id | GET | TimelineController | findOne | Yes |
| GET /care-recipients/:careRecipientId/timeline/vitals | GET | TimelineController | getRecentVitals | Yes |
| GET /care-recipients/:id | GET | CareRecipientController | findOne | Yes |
| GET /chat/channels | GET | ChatController | getUserChannels | Yes |
| GET /chat/token | GET | ChatController | getUserToken | Yes |
| GET /families | GET | FamilyController | getMyFamilies | Yes |
| GET /families/:familyId | GET | FamilyController | getFamily | Yes |
| GET /families/:familyId/care-recipients | GET | CareRecipientController | findAll | Yes |
| GET /families/:familyId/documents | GET | DocumentsController | findAll | Yes |
| GET /families/:familyId/documents/:id | GET | DocumentsController | findOne | Yes |
| GET /families/:familyId/documents/expiring | GET | DocumentsController | getExpiring | Yes |
| GET /families/invitations/:token/details | GET | FamilyController | getInvitationDetails | No |
| GET /health | GET | HealthController | check | No |
| GET /health/live | GET | HealthController | live | No |
| GET /health/ready | GET | HealthController | ready | No |
| GET /metrics | GET | MetricsController | getMetrics | No |
| GET /my-shifts | GET | MyShiftsController | getMyShifts | Yes |
| GET /notifications | GET | NotificationsController | findAll | Yes |
| GET /notifications/unread/count | GET | NotificationsController | getUnreadCount | Yes |
| PATCH /care-recipients/:careRecipientId/appointments/:id | PATCH | AppointmentsController | update | Yes |
| PATCH /care-recipients/:careRecipientId/appointments/:id/cancel | PATCH | AppointmentsController | cancel | Yes |
| PATCH /care-recipients/:careRecipientId/medications/:id | PATCH | MedicationsController | update | Yes |
| PATCH /care-recipients/:careRecipientId/medications/:id/deactivate | PATCH | MedicationsController | deactivate | Yes |
| PATCH /care-recipients/:careRecipientId/shifts/:id/cancel | PATCH | CaregiverShiftsController | cancelShift | Yes |
| PATCH /care-recipients/:careRecipientId/timeline/:id | PATCH | TimelineController | update | Yes |
| PATCH /care-recipients/:id | PATCH | CareRecipientController | update | Yes |
| PATCH /doctors/:doctorId | PATCH | CareRecipientController | updateDoctor | Yes |
| PATCH /families/:familyId | PATCH | FamilyController | updateFamily | Yes |
| PATCH /families/:familyId/documents/:id | PATCH | DocumentsController | update | Yes |
| PATCH /families/:familyId/members/:memberId/role | PATCH | FamilyController | updateMemberRole | Yes |
| PATCH /notifications/:notificationId/read | PATCH | NotificationsController | markAsRead | Yes |
| PATCH /notifications/read/all | PATCH | NotificationsController | markAllAsRead | Yes |
| POST /auth/change-password | POST | AuthController | changePassword | Yes |
| POST /auth/complete-onboarding | POST | AuthController | completeOnboarding | Yes |
| POST /auth/forgot-password | POST | AuthController | forgotPassword | No |
| POST /auth/login | POST | AuthController | login | No |
| POST /auth/logout | POST | AuthController | logout | Yes |
| POST /auth/logout-all | POST | AuthController | logoutAll | Yes |
| POST /auth/refresh | POST | AuthController | refresh | No |
| POST /auth/register | POST | AuthController | register | No |
| POST /auth/resend-verification | POST | AuthController | resendVerification | No |
| POST /auth/reset-password | POST | AuthController | resetPassword | No |
| POST /auth/verify-email | POST | AuthController | verifyEmail | No |
| POST /care-recipients/:careRecipientId/appointments | POST | AppointmentsController | create | Yes |
| POST /care-recipients/:careRecipientId/appointments/:id/transport | POST | AppointmentsController | assignTransport | Yes |
| POST /care-recipients/:careRecipientId/appointments/:id/transport/confirm | POST | AppointmentsController | confirmTransport | Yes |
| POST /care-recipients/:careRecipientId/doctors | POST | CareRecipientController | addDoctor | Yes |
| POST /care-recipients/:careRecipientId/emergency-contacts | POST | CareRecipientController | addEmergencyContact | Yes |
| POST /care-recipients/:careRecipientId/emergency/alerts | POST | EmergencyController | createAlert | Yes |
| POST /care-recipients/:careRecipientId/emergency/alerts/:alertId/acknowledge | POST | EmergencyController | acknowledgeAlert | Yes |
| POST /care-recipients/:careRecipientId/emergency/alerts/:alertId/resolve | POST | EmergencyController | resolveAlert | Yes |
| POST /care-recipients/:careRecipientId/medications | POST | MedicationsController | create | Yes |
| POST /care-recipients/:careRecipientId/shifts | POST | CaregiverShiftsController | create | Yes |
| POST /care-recipients/:careRecipientId/shifts/:id/checkin | POST | CaregiverShiftsController | checkIn | Yes |
| POST /care-recipients/:careRecipientId/shifts/:id/checkout | POST | CaregiverShiftsController | checkOut | Yes |
| POST /care-recipients/:careRecipientId/shifts/:id/confirm | POST | CaregiverShiftsController | confirmShift | Yes |
| POST /care-recipients/:careRecipientId/timeline | POST | TimelineController | create | Yes |
| POST /chat/family/:familyId/channel | POST | ChatController | createFamilyChannel | Yes |
| POST /chat/family/:familyId/topic | POST | ChatController | createTopicChannel | Yes |
| POST /families | POST | FamilyController | createFamily | Yes |
| POST /families/:familyId/care-recipients | POST | CareRecipientController | create | Yes |
| POST /families/:familyId/documents | POST | DocumentsController | create | Yes |
| POST /families/:familyId/invite | POST | FamilyController | inviteMember | Yes |
| POST /families/:familyId/members/:userId/reset-password | POST | FamilyController | resetMemberPassword | Yes |
| POST /families/invitations/:invitationId/resend | POST | FamilyController | resendInvitation | Yes |
| POST /families/invitations/:token/accept | POST | FamilyController | acceptInvitation | Yes |
| POST /families/invitations/:token/decline | POST | FamilyController | declineInvitation | No |
| POST /medications/:medicationId/log | POST | MedicationLogsController | logMedication | Yes |
| POST /notifications/push-token | POST | NotificationsController | registerPushToken | Yes |

---

## Query Parameters Summary

| Route | Query Params |
|-------|--------------|
| GET /care-recipients/:careRecipientId/appointments | startDate?, endDate?, status? |
| GET /care-recipients/:careRecipientId/appointments/day | date (required) |
| GET /care-recipients/:careRecipientId/appointments/upcoming | days? (default: 30) |
| GET /care-recipients/:careRecipientId/emergency/alerts/history | limit? (default: 20) |
| GET /care-recipients/:careRecipientId/medications | activeOnly? (default: true) |
| GET /care-recipients/:careRecipientId/medications/:id/logs | startDate?, endDate?, limit? (default: 100) |
| GET /care-recipients/:careRecipientId/shifts/day | date (required) |
| GET /care-recipients/:careRecipientId/shifts/upcoming | days? (default: 7) |
| GET /care-recipients/:careRecipientId/timeline | type?, startDate?, endDate?, limit? (default: 50), offset? (default: 0) |
| GET /care-recipients/:careRecipientId/timeline/vitals | days? (default: 7) |
| GET /families/:familyId/documents | type? |
| GET /families/:familyId/documents/expiring | days? (default: 30) |
| GET /my-shifts | upcomingOnly? |
| GET /notifications | unreadOnly?, limit? (default: 50) |

---

## Route Parameter Names

| Parameter Name | Type | Used In Controllers |
|----------------|------|---------------------|
| :careRecipientId | UUID | Medications, Appointments, Shifts, Timeline, Emergency, CareRecipient |
| :familyId | UUID | Family, Documents, CareRecipient, Chat |
| :id | UUID | Multiple (appointments, medications, documents, timeline, shifts) |
| :alertId | UUID | Emergency |
| :doctorId | UUID | CareRecipient |
| :contactId | UUID | CareRecipient |
| :memberId | UUID | Family |
| :userId | UUID | Family |
| :medicationId | UUID | MedicationLogs |
| :notificationId | string | Notifications |
| :token | string | Family (invitations), Auth (reset token) |
| :invitationId | string | Family |

---

## Public Routes (No Auth Required)

1. POST /auth/register
2. POST /auth/verify-email
3. POST /auth/resend-verification
4. POST /auth/login
5. POST /auth/refresh
6. POST /auth/forgot-password
7. GET /auth/verify-reset-token/:token
8. POST /auth/reset-password
9. GET /families/invitations/:token/details
10. POST /families/invitations/:token/decline
11. GET /health
12. GET /health/ready
13. GET /health/live
14. GET /metrics

---

## Rate Limited Routes

| Route | Limit |
|-------|-------|
| POST /auth/register | 5 requests per 60 seconds |
| POST /auth/verify-email | 5 requests per 60 seconds |
| POST /auth/resend-verification | 3 requests per 60 seconds |
| POST /auth/login | 10 requests per 60 seconds |
| POST /auth/forgot-password | 3 requests per 60 seconds |
| GET /auth/verify-reset-token/:token | 10 requests per 60 seconds |
| POST /auth/reset-password | 5 requests per 60 seconds |
