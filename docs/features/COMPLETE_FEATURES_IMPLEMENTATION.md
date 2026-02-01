# CareCircle Complete Features Implementation Guide

**Version:** 2.0  
**Last Updated:** January 2026  
**Status:** Production-Ready

---

## Table of Contents

- [Feature Summary](#feature-summary)
- [Authentication & Authorization](#authentication--authorization)
- [Family Management](#family-management)
- [Care Recipients](#care-recipients)
- [Medications](#medications)
- [Appointments & Calendar](#appointments--calendar)
- [Caregiver Shifts](#caregiver-shifts)
- [Documents](#documents)
- [Emergency Alerts](#emergency-alerts)
- [Health Timeline](#health-timeline)
- [Notifications](#notifications)
- [Real-Time Features](#real-time-features)
- [Chat Integration](#chat-integration)
- [Background Workers](#background-workers)
- [Frontend Pages](#frontend-pages)
- [API Endpoints Reference](#api-endpoints-reference)

---

## Feature Summary

| Feature Category | Status | Backend | Frontend | Real-Time |
|------------------|--------|---------|----------|-----------|
| Authentication | ✅ Complete | ✅ | ✅ | N/A |
| Family Management | ✅ Complete | ✅ | ✅ | ✅ |
| Care Recipients | ✅ Complete | ✅ | ✅ | ✅ |
| Medications | ✅ Complete | ✅ | ✅ | ✅ |
| Appointments | ✅ Complete | ✅ | ✅ | ✅ |
| Caregiver Shifts | ✅ Complete | ✅ | ✅ | ✅ |
| Documents | ✅ Complete | ✅ | ✅ | N/A |
| Emergency Alerts | ✅ Complete | ✅ | ✅ | ✅ |
| Health Timeline | ✅ Complete | ✅ | ✅ | ✅ |
| Notifications | ✅ Complete | ✅ | ✅ | ✅ |
| Web Push | ✅ Complete | ✅ | ✅ | ✅ |
| Chat (Stream) | ✅ Complete | ✅ | ✅ | ✅ |
| Background Workers | ✅ Complete | ✅ | N/A | N/A |
| Offline Support | ✅ Complete | N/A | ✅ | N/A |
| i18n (EN/FR) | ✅ Complete | ✅ | ✅ | N/A |

---

## Authentication & Authorization

### Implementation Status: ✅ Complete

**Backend Location:** `apps/api/src/auth/`

### Features

| Feature | Status | API Endpoint |
|---------|--------|--------------|
| User Registration | ✅ | `POST /api/v1/auth/register` |
| Email Verification (OTP) | ✅ | `POST /api/v1/auth/verify-email` |
| Resend Verification | ✅ | `POST /api/v1/auth/resend-verification` |
| Login | ✅ | `POST /api/v1/auth/login` |
| Token Refresh | ✅ | `POST /api/v1/auth/refresh` |
| Logout | ✅ | `POST /api/v1/auth/logout` |
| Logout All Devices | ✅ | `POST /api/v1/auth/logout-all` |
| Forgot Password | ✅ | `POST /api/v1/auth/forgot-password` |
| Verify Reset Token | ✅ | `GET /api/v1/auth/verify-reset-token/:token` |
| Reset Password | ✅ | `POST /api/v1/auth/reset-password` |
| Change Password | ✅ | `POST /api/v1/auth/change-password` |
| Get Active Sessions | ✅ | `GET /api/v1/auth/sessions` |
| Get Profile | ✅ | `GET /api/v1/auth/me` |
| Update Profile | ✅ | `PATCH /api/v1/auth/me` |
| Complete Onboarding | ✅ | `POST /api/v1/auth/complete-onboarding` |
| Invalidate Cache | ✅ | `POST /api/v1/auth/invalidate-cache` |

### Security Features

- **JWT Authentication:** Access tokens (15 min) + Refresh tokens (7-30 days)
- **HTTP-only Cookies:** Secure token storage
- **Password Hashing:** bcrypt with cost factor 10
- **Rate Limiting:** Per-endpoint throttling (5-10 requests/minute)
- **Session Management:** Track IP, user agent, device info
- **Token Rotation:** Refresh tokens are rotated on use

### Frontend Pages

| Page | Path | Description |
|------|------|-------------|
| Login | `/login` | User authentication |
| Register | `/register` | New account creation |
| Verify Email | `/verify-email` | OTP verification |
| Forgot Password | `/forgot-password` | Password reset request |
| Reset Password | `/reset-password` | Set new password |

---

## Family Management

### Implementation Status: ✅ Complete

**Backend Location:** `apps/api/src/family/`

### Features

| Feature | Status | API Endpoint |
|---------|--------|--------------|
| Create Family | ✅ | `POST /api/v1/families` |
| Get My Families | ✅ | `GET /api/v1/families` |
| Get Family Details | ✅ | `GET /api/v1/families/:familyId` |
| Update Family | ✅ | `PATCH /api/v1/families/:familyId` |
| Delete Family | ✅ | `DELETE /api/v1/families/:familyId` |
| Get Members | ✅ | `GET /api/v1/families/:familyId/members` |
| Invite Member | ✅ | `POST /api/v1/families/:familyId/invite` |
| Get Pending Invitations | ✅ | `GET /api/v1/families/:familyId/invitations` |
| Get Invitation Details | ✅ | `GET /api/v1/families/invitations/:token/details` |
| Accept Invitation | ✅ | `POST /api/v1/families/invitations/:token/accept` |
| Decline Invitation | ✅ | `POST /api/v1/families/invitations/:token/decline` |
| Update Member Role | ✅ | `PATCH /api/v1/families/:familyId/members/:memberId/role` |
| Remove Member | ✅ | `DELETE /api/v1/families/:familyId/members/:memberId` |
| Reset Member Password | ✅ | `POST /api/v1/families/:familyId/members/:userId/reset-password` |
| Cancel Invitation | ✅ | `DELETE /api/v1/families/invitations/:invitationId` |
| Resend Invitation | ✅ | `POST /api/v1/families/invitations/:invitationId/resend` |

### Role-Based Access Control

| Role | Permissions |
|------|-------------|
| **ADMIN** | Full access - manage family, members, care recipients, all features |
| **CAREGIVER** | Log medications, manage shifts, create timeline entries, view all |
| **VIEWER** | Read-only access to all family data |

### Frontend Pages

| Page | Path | Description |
|------|------|-------------|
| Family Management | `/family` | View and manage family members |
| Accept Invite | `/accept-invite/[token]` | Accept family invitation |
| Onboarding | `/onboarding` | New user onboarding flow |

---

## Care Recipients

### Implementation Status: ✅ Complete

**Backend Location:** `apps/api/src/care-recipient/`

### Features

| Feature | Status | API Endpoint |
|---------|--------|--------------|
| Create Care Recipient | ✅ | `POST /api/v1/families/:familyId/care-recipients` |
| List Care Recipients | ✅ | `GET /api/v1/families/:familyId/care-recipients` |
| Get Care Recipient | ✅ | `GET /api/v1/care-recipients/:id` |
| Update Care Recipient | ✅ | `PATCH /api/v1/care-recipients/:id` |
| Delete Care Recipient | ✅ | `DELETE /api/v1/care-recipients/:id` |
| Add Doctor | ✅ | `POST /api/v1/care-recipients/:id/doctors` |
| List Doctors | ✅ | `GET /api/v1/care-recipients/:id/doctors` |
| Update Doctor | ✅ | `PATCH /api/v1/doctors/:doctorId` |
| Delete Doctor | ✅ | `DELETE /api/v1/doctors/:doctorId` |
| Add Emergency Contact | ✅ | `POST /api/v1/care-recipients/:id/emergency-contacts` |
| List Emergency Contacts | ✅ | `GET /api/v1/care-recipients/:id/emergency-contacts` |
| Delete Emergency Contact | ✅ | `DELETE /api/v1/emergency-contacts/:contactId` |

### Data Model

- **Personal Info:** Name, preferred name, date of birth, photo
- **Medical Info:** Blood type, allergies (array), conditions (array), notes
- **Emergency Info:** Primary hospital, hospital address, insurance provider, policy number
- **Related Entities:** Doctors, Emergency Contacts, Medications, Appointments, Shifts

### Frontend Pages

| Page | Path | Description |
|------|------|-------------|
| Care Recipients List | `/care-recipients` | View all care recipients |
| Care Recipient Detail | `/care-recipients/[id]` | Detailed view with all info |

---

## Medications

### Implementation Status: ✅ Complete

**Backend Location:** `apps/api/src/medications/`

### Features

| Feature | Status | API Endpoint |
|---------|--------|--------------|
| Create Medication | ✅ | `POST /api/v1/care-recipients/:id/medications` |
| List Medications | ✅ | `GET /api/v1/care-recipients/:id/medications` |
| Get Today's Schedule | ✅ | `GET /api/v1/care-recipients/:id/medications/schedule/today` |
| Get Medication | ✅ | `GET /api/v1/care-recipients/:id/medications/:medId` |
| Get Medication Logs | ✅ | `GET /api/v1/care-recipients/:id/medications/:medId/logs` |
| Update Medication | ✅ | `PATCH /api/v1/care-recipients/:id/medications/:medId` |
| Deactivate Medication | ✅ | `PATCH /api/v1/care-recipients/:id/medications/:medId/deactivate` |
| Delete Medication | ✅ | `DELETE /api/v1/care-recipients/:id/medications/:medId` |
| Log Medication | ✅ | `POST /api/v1/medications/:medicationId/log` |

### Drug Interaction Checking

| Feature | Status | API Endpoint |
|---------|--------|--------------|
| Check Current Interactions | ✅ | `GET /api/v1/care-recipients/:id/medications/interactions` |
| Check New Medication | ✅ | `POST /api/v1/care-recipients/:id/medications/interactions/check-new` |
| Check Interactions (Global) | ✅ | `POST /api/v1/medications/interactions/check` |
| Get Interaction Details | ✅ | `GET /api/v1/medications/interactions/details` |
| Get Known Interactions | ✅ | `GET /api/v1/medications/interactions/known` |

### Medication Properties

- **Basic:** Name, generic name, dosage, form (tablet, capsule, liquid, etc.)
- **Instructions:** Instructions, prescribed by, pharmacy, pharmacy phone
- **Schedule:** Frequency, times per day, scheduled times array
- **Supply Tracking:** Current supply, refill threshold, last refill date
- **Status:** Active/inactive, start date, end date

### Log Statuses

- `GIVEN` - Medication was administered
- `SKIPPED` - Medication was intentionally skipped
- `MISSED` - Medication was not given at scheduled time
- `PENDING` - Scheduled but not yet due

### Frontend Pages

| Page | Path | Description |
|------|------|-------------|
| Medications | `/medications` | View and manage medications |

---

## Appointments & Calendar

### Implementation Status: ✅ Complete

**Backend Location:** `apps/api/src/appointments/`

### Features

| Feature | Status | API Endpoint |
|---------|--------|--------------|
| Create Appointment | ✅ | `POST /api/v1/care-recipients/:id/appointments` |
| Create Recurring Series | ✅ | `POST /api/v1/care-recipients/:id/appointments/recurring` |
| Get Recurring Series | ✅ | `GET /api/v1/care-recipients/:id/appointments/recurring/series` |
| List Appointments | ✅ | `GET /api/v1/care-recipients/:id/appointments` |
| Get Upcoming | ✅ | `GET /api/v1/care-recipients/:id/appointments/upcoming` |
| Get For Day | ✅ | `GET /api/v1/care-recipients/:id/appointments/day` |
| Get Appointment | ✅ | `GET /api/v1/appointments/:id` |
| Get Recurring Occurrences | ✅ | `GET /api/v1/appointments/:id/occurrences` |
| Update Appointment | ✅ | `PATCH /api/v1/appointments/:id` |
| Cancel Appointment | ✅ | `PATCH /api/v1/appointments/:id/cancel` |
| Delete Appointment | ✅ | `DELETE /api/v1/appointments/:id` |
| Assign Transport | ✅ | `POST /api/v1/appointments/:id/transport` |
| Confirm Transport | ✅ | `POST /api/v1/appointments/:id/transport/confirm` |
| Update Recurring Series | ✅ | `PATCH /api/v1/appointments/series/:seriesId` |
| Cancel Recurring Series | ✅ | `PATCH /api/v1/appointments/series/:seriesId/cancel` |

### Appointment Types

- `DOCTOR_VISIT`
- `PHYSICAL_THERAPY`
- `LAB_WORK`
- `IMAGING`
- `SPECIALIST`
- `HOME_HEALTH`
- `OTHER`

### Recurrence Support

- Uses **RRULE** format for complex recurrence patterns
- Supports: daily, weekly, biweekly, monthly, custom
- Configurable max occurrences (default: 52)

### Frontend Pages

| Page | Path | Description |
|------|------|-------------|
| Calendar | `/calendar` | View appointments in calendar view |

---

## Caregiver Shifts

### Implementation Status: ✅ Complete

**Backend Location:** `apps/api/src/caregiver-shifts/`

### Features

| Feature | Status | API Endpoint |
|---------|--------|--------------|
| Create Shift | ✅ | `POST /api/v1/care-recipients/:id/shifts` |
| List Shifts | ✅ | `GET /api/v1/care-recipients/:id/shifts` |
| Get Shifts by Date Range | ✅ | `GET /api/v1/care-recipients/:id/shifts/range` |
| Get Current Shift | ✅ | `GET /api/v1/care-recipients/:id/shifts/current` |
| Get On Duty | ✅ | `GET /api/v1/care-recipients/:id/shifts/on-duty` |
| Get Upcoming Shifts | ✅ | `GET /api/v1/care-recipients/:id/shifts/upcoming` |
| Get Shifts for Day | ✅ | `GET /api/v1/care-recipients/:id/shifts/day` |
| Get Shift by ID | ✅ | `GET /api/v1/care-recipients/:id/shifts/:shiftId` |
| Check In | ✅ | `POST /api/v1/care-recipients/:id/shifts/:shiftId/checkin` |
| Check Out | ✅ | `POST /api/v1/care-recipients/:id/shifts/:shiftId/checkout` |
| Confirm Shift | ✅ | `POST /api/v1/care-recipients/:id/shifts/:shiftId/confirm` |
| Cancel Shift | ✅ | `PATCH /api/v1/care-recipients/:id/shifts/:shiftId/cancel` |
| Get My Shifts | ✅ | `GET /api/v1/my-shifts` |

### Shift Statuses

- `SCHEDULED` - Shift is planned
- `CONFIRMED` - Caregiver has confirmed
- `IN_PROGRESS` - Currently active (checked in)
- `COMPLETED` - Finished (checked out)
- `CANCELLED` - Cancelled
- `NO_SHOW` - Caregiver didn't show up

### Handoff Notes

Caregivers can leave handoff notes when checking out to communicate important information to the next caregiver.

### Frontend Pages

| Page | Path | Description |
|------|------|-------------|
| Caregivers | `/caregivers` | Manage shifts and view schedule |

---

## Documents

### Implementation Status: ✅ Complete

**Backend Location:** `apps/api/src/documents/`

### Features

| Feature | Status | API Endpoint |
|---------|--------|--------------|
| Upload Document | ✅ | `POST /api/v1/families/:familyId/documents` |
| List Documents | ✅ | `GET /api/v1/families/:familyId/documents` |
| Get Expiring Documents | ✅ | `GET /api/v1/families/:familyId/documents/expiring` |
| Get Documents by Category | ✅ | `GET /api/v1/families/:familyId/documents/by-category` |
| Get Document | ✅ | `GET /api/v1/families/:familyId/documents/:id` |
| Get Signed URL | ✅ | `GET /api/v1/families/:familyId/documents/:id/url` |
| Update Document | ✅ | `PATCH /api/v1/families/:familyId/documents/:id` |
| Delete Document | ✅ | `DELETE /api/v1/families/:familyId/documents/:id` |

### Document Types

- `INSURANCE_CARD`
- `PHOTO_ID`
- `MEDICAL_RECORD`
- `LAB_RESULT`
- `PRESCRIPTION`
- `POWER_OF_ATTORNEY`
- `LIVING_WILL`
- `DNR`
- `OTHER`

### Storage

- **Provider:** Cloudinary (primary) / AWS S3 (alternative)
- **Processing:** Async upload via Bull queue
- **Access:** Signed URLs for secure viewing/downloading

### Frontend Pages

| Page | Path | Description |
|------|------|-------------|
| Documents | `/documents` | Document vault |

---

## Emergency Alerts

### Implementation Status: ✅ Complete

**Backend Location:** `apps/api/src/emergency/`

### Features

| Feature | Status | API Endpoint |
|---------|--------|--------------|
| Get Emergency Info | ✅ | `GET /api/v1/care-recipients/:id/emergency/info` |
| Create Emergency Alert | ✅ | `POST /api/v1/care-recipients/:id/emergency/alerts` |
| Get Active Alerts | ✅ | `GET /api/v1/care-recipients/:id/emergency/alerts` |
| Get Alert History | ✅ | `GET /api/v1/care-recipients/:id/emergency/alerts/history` |
| Acknowledge Alert | ✅ | `POST /api/v1/care-recipients/:id/emergency/alerts/:alertId/acknowledge` |
| Resolve Alert | ✅ | `POST /api/v1/care-recipients/:id/emergency/alerts/:alertId/resolve` |

### Emergency Types

- `FALL` - Person has fallen
- `MEDICAL` - Medical emergency
- `MISSING` - Person is missing
- `HOSPITALIZATION` - Hospitalized
- `OTHER` - Other emergency

### Alert Statuses

- `ACTIVE` - Alert is active, needs attention
- `ACKNOWLEDGED` - Family member has acknowledged
- `RESOLVED` - Emergency has been resolved

### Real-Time Notifications

When an emergency alert is created:
1. WebSocket event broadcast to all family members
2. Push notifications sent to all subscribed devices
3. In-app notifications created for all family members

### Frontend Pages

| Page | Path | Description |
|------|------|-------------|
| Emergency | `/emergency` | Emergency alerts and info |

---

## Health Timeline

### Implementation Status: ✅ Complete

**Backend Location:** `apps/api/src/timeline/`

### Features

| Feature | Status | API Endpoint |
|---------|--------|--------------|
| Create Entry | ✅ | `POST /api/v1/care-recipients/:id/timeline` |
| List Entries | ✅ | `GET /api/v1/care-recipients/:id/timeline` |
| Get Recent Vitals | ✅ | `GET /api/v1/care-recipients/:id/timeline/vitals` |
| Get Incidents | ✅ | `GET /api/v1/care-recipients/:id/timeline/incidents` |
| Get Entry | ✅ | `GET /api/v1/care-recipients/:id/timeline/:entryId` |
| Update Entry | ✅ | `PATCH /api/v1/timeline/:id` |
| Delete Entry | ✅ | `DELETE /api/v1/timeline/:id` |
| Get Family Activity Feed | ✅ | `GET /api/v1/families/:familyId/activity` |

### Timeline Entry Types

- `NOTE` - General notes
- `VITALS` - Vital signs recording
- `SYMPTOM` - Symptom tracking
- `INCIDENT` - Incidents (falls, etc.)
- `MOOD` - Mood tracking
- `MEAL` - Meal logging
- `ACTIVITY` - Physical activity
- `SLEEP` - Sleep tracking
- `BATHROOM` - Bathroom visits
- `MEDICATION_CHANGE` - Medication changes
- `APPOINTMENT_SUMMARY` - Appointment notes
- `OTHER` - Other entries

### Severity Levels

- `LOW`
- `MEDIUM`
- `HIGH`
- `CRITICAL`

### Frontend Pages

| Page | Path | Description |
|------|------|-------------|
| Timeline | `/timeline` | Health timeline view |

---

## Notifications

### Implementation Status: ✅ Complete

**Backend Location:** `apps/api/src/notifications/`

### Features

| Feature | Status | API Endpoint |
|---------|--------|--------------|
| List Notifications | ✅ | `GET /api/v1/notifications` |
| Get Unread | ✅ | `GET /api/v1/notifications/unread` |
| Get Unread Count | ✅ | `GET /api/v1/notifications/unread/count` |
| Mark as Read (Batch) | ✅ | `PATCH /api/v1/notifications/read` |
| Mark as Read (Single) | ✅ | `PATCH /api/v1/notifications/:id/read` |
| Mark All as Read | ✅ | `PATCH /api/v1/notifications/read/all` |
| Subscribe to Push | ✅ | `POST /api/v1/notifications/push-subscription` |
| Unsubscribe from Push | ✅ | `DELETE /api/v1/notifications/push-subscription` |
| Register Push Token | ✅ | `POST /api/v1/notifications/push-token` |
| Remove Push Token | ✅ | `DELETE /api/v1/notifications/push-token` |
| Send Test Push | ✅ | `POST /api/v1/notifications/test-push` |

### Notification Types

- `MEDICATION_REMINDER` - Time to take medication
- `MEDICATION_MISSED` - Medication was missed
- `APPOINTMENT_REMINDER` - Upcoming appointment
- `SHIFT_REMINDER` - Upcoming shift
- `SHIFT_HANDOFF` - Shift handoff notes
- `EMERGENCY_ALERT` - Emergency alert
- `FAMILY_INVITE` - Family invitation
- `DOCUMENT_SHARED` - Document shared
- `TIMELINE_UPDATE` - Timeline entry added
- `REFILL_NEEDED` - Medication refill needed
- `REFILL_ALERT` - Low medication supply
- `GENERAL` - General notification
- `CARE_RECIPIENT_DELETED/UPDATED` - Admin actions
- `MEDICATION_DELETED` - Medication removed
- `APPOINTMENT_DELETED` - Appointment cancelled
- `FAMILY_MEMBER_REMOVED/ROLE_CHANGED` - Membership changes
- `FAMILY_DELETED` - Family removed

### Web Push (VAPID)

- Uses native Web Push API (no Firebase)
- VAPID keys for secure push
- Works offline with service workers

---

## Real-Time Features

### Implementation Status: ✅ Complete

**Backend Location:** `apps/api/src/gateway/`

### WebSocket Events

The application uses Socket.io for real-time communication:

- **Connection:** Authenticated via JWT token
- **Rooms:** Family-scoped rooms for targeted broadcasts
- **Events:** Medication logs, emergency alerts, shift updates, etc.

### Event Types

- `medication.logged` - Medication was administered
- `emergency.alert.created` - New emergency alert
- `emergency.alert.acknowledged` - Alert acknowledged
- `emergency.alert.resolved` - Alert resolved
- `shift.checkin` - Caregiver checked in
- `shift.checkout` - Caregiver checked out
- `appointment.created` - New appointment
- `appointment.updated` - Appointment modified
- `timeline.entry.created` - New timeline entry

---

## Chat Integration

### Implementation Status: ✅ Complete

**Backend Location:** `apps/api/src/chat/`

### Features

| Feature | Status | API Endpoint |
|---------|--------|--------------|
| Get User Token | ✅ | `GET /api/v1/chat/token` |
| Initialize Family Chat | ✅ | `GET /api/v1/chat/family/:familyId/init` |
| Create Family Channel | ✅ | `POST /api/v1/chat/family/:familyId/channel` |
| Create Topic Channel | ✅ | `POST /api/v1/chat/family/:familyId/topic` |
| Add Member to Channel | ✅ | `POST /api/v1/chat/family/:familyId/member/:memberId` |
| Get User Channels | ✅ | `GET /api/v1/chat/channels` |
| Get Chat Status | ✅ | `GET /api/v1/chat/status` |

### Provider: Stream Chat

- **Free Tier:** 5M API calls/month
- **Features:** Typing indicators, read receipts, file sharing, reactions, threads
- **UI Components:** `stream-chat-react` for pre-built React components

### Frontend Pages

| Page | Path | Description |
|------|------|-------------|
| Chat | `/chat` | Family messaging |

---

## Background Workers

### Implementation Status: ✅ Complete

**Location:** `apps/workers/src/`

### Worker Types

| Worker | Queue | Purpose |
|--------|-------|---------|
| Medication Reminder | `medication-reminders` | Send reminders before scheduled medication times |
| Appointment Reminder | `appointment-reminders` | Send reminders before appointments |
| Shift Reminder | `shift-reminders` | Remind caregivers of upcoming shifts |
| Refill Alert | `refill-alerts` | Alert when medication supply is low |
| Notification | `notifications` | Process and deliver notifications |
| Dead Letter | `dlq` | Handle failed jobs |

### Scheduler

The scheduler (`apps/workers/src/scheduler.ts`) runs on an interval and:
1. Checks for upcoming medications to remind
2. Checks for upcoming appointments
3. Checks for upcoming shifts
4. Monitors medication supply levels

### Technologies

- **BullMQ:** Redis-based job queue
- **web-push:** Push notification delivery
- **nodemailer:** Email sending
- **Twilio:** SMS notifications

### Health Check

Workers expose health endpoints on port 3002:
- `GET /health` - Overall health status
- `GET /ready` - Readiness check

---

## Frontend Pages

### Marketing Pages

| Page | Path |
|------|------|
| Landing | `/` |
| About | `/about` |
| How It Works | `/how-it-works` |
| Pricing | `/pricing` |
| Stories | `/stories` |
| Journal | `/journal` |
| Contact | `/contact` |

### Auth Pages

| Page | Path |
|------|------|
| Login | `/login` |
| Register | `/register` |
| Verify Email | `/verify-email` |
| Forgot Password | `/forgot-password` |
| Reset Password | `/reset-password` |
| Accept Invite | `/accept-invite/[token]` |

### App Pages (Protected)

| Page | Path |
|------|------|
| Dashboard | `/dashboard` |
| Care Recipients | `/care-recipients` |
| Care Recipient Detail | `/care-recipients/[id]` |
| Medications | `/medications` |
| Calendar | `/calendar` |
| Caregivers | `/caregivers` |
| Documents | `/documents` |
| Emergency | `/emergency` |
| Timeline | `/timeline` |
| Family | `/family` |
| Chat | `/chat` |
| Settings | `/settings` |
| Onboarding | `/onboarding` |
| Offline | `/offline` |

---

## API Endpoints Reference

### Health & Metrics

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/health` | GET | Full health check |
| `/api/v1/health/ready` | GET | Readiness check |
| `/api/v1/health/live` | GET | Liveness check |
| `/api/v1/metrics` | GET | Prometheus metrics |

### Total API Endpoint Count: 90+

For complete API documentation, access Swagger UI at:
- **Local:** `http://localhost:4000/api`
- **Production:** `https://your-domain.com/api`

---

## Database Schema

### Core Entities (19 models)

1. **User** - User accounts
2. **Session** - Refresh token sessions
3. **PushToken** - Push notification subscriptions
4. **Family** - Family units
5. **FamilyMember** - User-family relationships
6. **FamilyInvitation** - Pending invitations
7. **CareRecipient** - Care recipients
8. **Doctor** - Healthcare providers
9. **EmergencyContact** - Emergency contacts
10. **Medication** - Medications
11. **MedicationLog** - Medication administration logs
12. **Appointment** - Calendar appointments
13. **TransportAssignment** - Transport assignments
14. **CaregiverShift** - Shift schedules
15. **TimelineEntry** - Health timeline
16. **Document** - Document storage
17. **EmergencyAlert** - Emergency alerts
18. **Notification** - In-app notifications
19. **AuditLog** - HIPAA compliance audit trail

**Schema Location:** `packages/database/prisma/schema.prisma`

---

## Environment Variables

See `env/base.env` for complete list of required environment variables.

### Key Variables

| Variable | Description |
|----------|-------------|
| `DATABASE_URL` | PostgreSQL connection string |
| `REDIS_URL` | Redis connection string |
| `JWT_SECRET` | JWT signing secret |
| `VAPID_PUBLIC_KEY` | Web push public key |
| `VAPID_PRIVATE_KEY` | Web push private key |
| `CLOUDINARY_*` | Cloudinary configuration |
| `MAILTRAP_*` | Email service configuration |
| `NEXT_PUBLIC_STREAM_API_KEY` | Stream Chat API key |

---

*Last Updated: January 2026*
