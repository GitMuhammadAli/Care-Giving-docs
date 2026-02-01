# Frontend API Clients Map

**Generated:** 2026-01-20
**Total API Files:** 12
**API Base URL:** `process.env.NEXT_PUBLIC_API_URL || 'http://localhost:4000/api/v1'`

---

## Summary

| API File | API Object | Endpoints |
|----------|------------|-----------|
| client.ts | api (ApiClient) | Base client with fetch, get, post, put, patch, delete, upload |
| auth.ts | authApi | 12 |
| family.ts | familyApi | 13 |
| care-recipients.ts | careRecipientsApi | 9 |
| medications.ts | medicationsApi | 10 |
| appointments.ts | appointmentsApi | 9 |
| documents.ts | documentsApi | 8 |
| emergency.ts | emergencyApi | 6 |
| shifts.ts | shiftsApi | 12 |
| timeline.ts | timelineApi | 6 |
| notifications.ts | notificationsApi | 7 |

---

## Auth API (`authApi`)

**File:** `apps/web/src/lib/api/auth.ts`

| Method | HTTP | Frontend Endpoint | Backend Route | Match |
|--------|------|-------------------|---------------|-------|
| register | POST | /auth/register | POST /auth/register | ✅ |
| verifyEmail | POST | /auth/verify-email | POST /auth/verify-email | ✅ |
| resendVerification | POST | /auth/resend-verification | POST /auth/resend-verification | ✅ |
| login | POST | /auth/login | POST /auth/login | ✅ |
| logout | POST | /auth/logout | POST /auth/logout | ✅ |
| refresh | POST | /auth/refresh | POST /auth/refresh | ✅ |
| getProfile | GET | /auth/me | GET /auth/me | ✅ |
| updateProfile | PATCH | /auth/me | ❌ NO BACKEND ROUTE | ❌ |
| forgotPassword | POST | /auth/forgot-password | POST /auth/forgot-password | ✅ |
| verifyResetToken | GET | /auth/verify-reset-token/:token | GET /auth/verify-reset-token/:token | ✅ |
| resetPassword | POST | /auth/reset-password | POST /auth/reset-password | ✅ |
| completeOnboarding | POST | /auth/complete-onboarding | POST /auth/complete-onboarding | ✅ |

**Issues Found:** 1
- `updateProfile` calls `PATCH /auth/me` but backend doesn't have this endpoint (only `GET /auth/me`)

---

## Family API (`familyApi`)

**File:** `apps/web/src/lib/api/family.ts`

| Method | HTTP | Frontend Endpoint | Backend Route | Match |
|--------|------|-------------------|---------------|-------|
| list | GET | /families | GET /families | ✅ |
| get | GET | /families/:id | GET /families/:familyId | ✅ |
| create | POST | /families | POST /families | ✅ |
| update | PATCH | /families/:id | PATCH /families/:familyId | ✅ |
| delete | DELETE | /families/:id | DELETE /families/:familyId | ✅ |
| getMembers | GET | /families/:familyId/members | ❌ NO BACKEND ROUTE | ❌ |
| updateMemberRole | PATCH | /families/:familyId/members/:memberId/role | PATCH /families/:familyId/members/:memberId/role | ✅ |
| removeMember | DELETE | /families/:familyId/members/:memberId | DELETE /families/:familyId/members/:memberId | ✅ |
| invite | POST | /families/:familyId/invite | POST /families/:familyId/invite | ✅ |
| getPendingInvitations | GET | /families/:familyId/invitations | ❌ NO BACKEND ROUTE | ❌ |
| cancelInvitation | DELETE | /families/:familyId/invitations/:invitationId | DELETE /families/invitations/:invitationId | ⚠️ PATH MISMATCH |
| resendInvitation | POST | /families/:familyId/invitations/:invitationId/resend | POST /families/invitations/:invitationId/resend | ⚠️ PATH MISMATCH |
| acceptInvitation | POST | /families/accept-invite/:token | POST /families/invitations/:token/accept | ⚠️ PATH MISMATCH |
| resetMemberPassword | POST | /families/:familyId/members/:userId/reset-password | POST /families/:familyId/members/:userId/reset-password | ✅ |

**Issues Found:** 5
- `getMembers` calls endpoint that doesn't exist (members come with `GET /families/:familyId`)
- `getPendingInvitations` calls endpoint that doesn't exist
- `cancelInvitation` uses `/families/:familyId/invitations/:id` but backend is `/families/invitations/:id` (no familyId)
- `resendInvitation` uses `/families/:familyId/invitations/:id/resend` but backend is `/families/invitations/:id/resend` (no familyId)
- `acceptInvitation` uses `/families/accept-invite/:token` but backend is `/families/invitations/:token/accept`

---

## Care Recipients API (`careRecipientsApi`)

**File:** `apps/web/src/lib/api/care-recipients.ts`

| Method | HTTP | Frontend Endpoint | Backend Route | Match |
|--------|------|-------------------|---------------|-------|
| list | GET | /families/:familyId/care-recipients | GET /families/:familyId/care-recipients | ✅ |
| get | GET | /care-recipients/:id | GET /care-recipients/:id | ✅ |
| create | POST | /families/:familyId/care-recipients | POST /families/:familyId/care-recipients | ✅ |
| update | PATCH | /care-recipients/:id | PATCH /care-recipients/:id | ✅ |
| delete | DELETE | /care-recipients/:id | DELETE /care-recipients/:id | ✅ |
| getDoctors | GET | /care-recipients/:careRecipientId/doctors | GET /care-recipients/:careRecipientId/doctors | ✅ |
| addDoctor | POST | /care-recipients/:careRecipientId/doctors | POST /care-recipients/:careRecipientId/doctors | ✅ |
| getEmergencyContacts | GET | /care-recipients/:careRecipientId/emergency-contacts | GET /care-recipients/:careRecipientId/emergency-contacts | ✅ |
| addEmergencyContact | POST | /care-recipients/:careRecipientId/emergency-contacts | POST /care-recipients/:careRecipientId/emergency-contacts | ✅ |

**Issues Found:** 0 ✅

---

## Medications API (`medicationsApi`)

**File:** `apps/web/src/lib/api/medications.ts`

| Method | HTTP | Frontend Endpoint | Backend Route | Match |
|--------|------|-------------------|---------------|-------|
| list | GET | /care-recipients/:careRecipientId/medications | GET /care-recipients/:careRecipientId/medications | ✅ |
| get | GET | /medications/:id | GET /care-recipients/:careRecipientId/medications/:id | ⚠️ PATH MISMATCH |
| create | POST | /care-recipients/:careRecipientId/medications | POST /care-recipients/:careRecipientId/medications | ✅ |
| update | PATCH | /medications/:id | PATCH /care-recipients/:careRecipientId/medications/:id | ⚠️ PATH MISMATCH |
| delete | DELETE | /medications/:id | DELETE /care-recipients/:careRecipientId/medications/:id | ⚠️ PATH MISMATCH |
| getTodaySchedule | GET | /care-recipients/:careRecipientId/medications/schedule/today | GET /care-recipients/:careRecipientId/medications/schedule/today | ✅ |
| log | POST | /medications/:medicationId/log | POST /medications/:medicationId/log | ✅ |
| getLowSupply | GET | /care-recipients/:careRecipientId/medications/low-supply | ❌ NO BACKEND ROUTE | ❌ |
| updateSupply | PATCH | /medications/:id/supply | ❌ NO BACKEND ROUTE | ❌ |

**Issues Found:** 5
- `get` uses `/medications/:id` but backend requires `/care-recipients/:careRecipientId/medications/:id`
- `update` uses `/medications/:id` but backend requires `/care-recipients/:careRecipientId/medications/:id`
- `delete` uses `/medications/:id` but backend requires `/care-recipients/:careRecipientId/medications/:id`
- `getLowSupply` endpoint doesn't exist in backend
- `updateSupply` endpoint doesn't exist in backend

---

## Appointments API (`appointmentsApi`)

**File:** `apps/web/src/lib/api/appointments.ts`

| Method | HTTP | Frontend Endpoint | Backend Route | Match |
|--------|------|-------------------|---------------|-------|
| list | GET | /care-recipients/:careRecipientId/appointments | GET /care-recipients/:careRecipientId/appointments | ✅ |
| getUpcoming | GET | /care-recipients/:careRecipientId/appointments/upcoming | GET /care-recipients/:careRecipientId/appointments/upcoming | ✅ |
| getForDay | GET | /care-recipients/:careRecipientId/appointments/day | GET /care-recipients/:careRecipientId/appointments/day | ✅ |
| get | GET | /appointments/:id | GET /care-recipients/:careRecipientId/appointments/:id | ⚠️ PATH MISMATCH |
| create | POST | /care-recipients/:careRecipientId/appointments | POST /care-recipients/:careRecipientId/appointments | ✅ |
| update | PATCH | /appointments/:id | PATCH /care-recipients/:careRecipientId/appointments/:id | ⚠️ PATH MISMATCH |
| cancel | PATCH | /appointments/:id/cancel | PATCH /care-recipients/:careRecipientId/appointments/:id/cancel | ⚠️ PATH MISMATCH |
| delete | DELETE | /appointments/:id | DELETE /care-recipients/:careRecipientId/appointments/:id | ⚠️ PATH MISMATCH |
| assignTransport | POST | /appointments/:id/transport | POST /care-recipients/:careRecipientId/appointments/:id/transport | ⚠️ PATH MISMATCH |

**Issues Found:** 5
- `get` uses `/appointments/:id` but backend requires `/care-recipients/:careRecipientId/appointments/:id`
- `update` uses `/appointments/:id` but backend requires `/care-recipients/:careRecipientId/appointments/:id`
- `cancel` uses `/appointments/:id/cancel` but backend requires `/care-recipients/:careRecipientId/appointments/:id/cancel`
- `delete` uses `/appointments/:id` but backend requires `/care-recipients/:careRecipientId/appointments/:id`
- `assignTransport` uses `/appointments/:id/transport` but backend requires `/care-recipients/:careRecipientId/appointments/:id/transport`

---

## Documents API (`documentsApi`)

**File:** `apps/web/src/lib/api/documents.ts`

| Method | HTTP | Frontend Endpoint | Backend Route | Match |
|--------|------|-------------------|---------------|-------|
| upload | POST | /families/:familyId/documents | POST /families/:familyId/documents | ✅ |
| list | GET | /families/:familyId/documents | GET /families/:familyId/documents | ✅ |
| get | GET | /families/:familyId/documents/:documentId | GET /families/:familyId/documents/:id | ✅ |
| getSignedUrl | GET | /families/:familyId/documents/:documentId/url | ❌ NO BACKEND ROUTE | ❌ |
| getByCategory | GET | /families/:familyId/documents/by-category | ❌ NO BACKEND ROUTE | ❌ |
| getExpiring | GET | /families/:familyId/documents/expiring | GET /families/:familyId/documents/expiring | ✅ |
| update | PATCH | /families/:familyId/documents/:documentId | PATCH /families/:familyId/documents/:id | ✅ |
| delete | DELETE | /families/:familyId/documents/:documentId | DELETE /families/:familyId/documents/:id | ✅ |

**Issues Found:** 2
- `getSignedUrl` endpoint doesn't exist in backend
- `getByCategory` endpoint doesn't exist in backend

---

## Emergency API (`emergencyApi`)

**File:** `apps/web/src/lib/api/emergency.ts`

| Method | HTTP | Frontend Endpoint | Backend Route | Match |
|--------|------|-------------------|---------------|-------|
| getEmergencyInfo | GET | /care-recipients/:careRecipientId/emergency/info | GET /care-recipients/:careRecipientId/emergency/info | ✅ |
| triggerAlert | POST | /care-recipients/:careRecipientId/emergency/alerts | POST /care-recipients/:careRecipientId/emergency/alerts | ✅ |
| getActiveAlerts | GET | /care-recipients/:careRecipientId/emergency/alerts | GET /care-recipients/:careRecipientId/emergency/alerts | ✅ |
| getAlertHistory | GET | /care-recipients/:careRecipientId/emergency/alerts/history | GET /care-recipients/:careRecipientId/emergency/alerts/history | ✅ |
| acknowledgeAlert | POST | /care-recipients/:careRecipientId/emergency/alerts/:alertId/acknowledge | POST /care-recipients/:careRecipientId/emergency/alerts/:alertId/acknowledge | ✅ |
| resolveAlert | POST | /care-recipients/:careRecipientId/emergency/alerts/:alertId/resolve | POST /care-recipients/:careRecipientId/emergency/alerts/:alertId/resolve | ✅ |

**Issues Found:** 0 ✅

---

## Shifts API (`shiftsApi`)

**File:** `apps/web/src/lib/api/shifts.ts`

| Method | HTTP | Frontend Endpoint | Backend Route | Match |
|--------|------|-------------------|---------------|-------|
| create | POST | /care-recipients/:careRecipientId/shifts | POST /care-recipients/:careRecipientId/shifts | ✅ |
| getAll | GET | /care-recipients/:careRecipientId/shifts | ❌ NO BACKEND ROUTE | ❌ |
| getByDateRange | GET | /care-recipients/:careRecipientId/shifts/range | ❌ NO BACKEND ROUTE | ❌ |
| getCurrent | GET | /care-recipients/:careRecipientId/shifts/current | GET /care-recipients/:careRecipientId/shifts/current | ✅ |
| getUpcoming | GET | /care-recipients/:careRecipientId/shifts/upcoming | GET /care-recipients/:careRecipientId/shifts/upcoming | ✅ |
| getOnDuty | GET | /care-recipients/:careRecipientId/shifts/on-duty | ❌ NO BACKEND ROUTE | ❌ |
| getById | GET | /care-recipients/:careRecipientId/shifts/:shiftId | ❌ NO BACKEND ROUTE | ❌ |
| checkIn | POST | /care-recipients/:careRecipientId/shifts/:shiftId/checkin | POST /care-recipients/:careRecipientId/shifts/:id/checkin | ✅ |
| checkOut | POST | /care-recipients/:careRecipientId/shifts/:shiftId/checkout | POST /care-recipients/:careRecipientId/shifts/:id/checkout | ✅ |
| cancel | PATCH | /care-recipients/:careRecipientId/shifts/:shiftId/cancel | PATCH /care-recipients/:careRecipientId/shifts/:id/cancel | ✅ |
| getMyShifts | GET | /my-shifts | GET /my-shifts | ✅ |

**Issues Found:** 4
- `getAll` endpoint doesn't exist (backend only has filtered views)
- `getByDateRange` endpoint doesn't exist (backend has `/shifts/day` with date query)
- `getOnDuty` endpoint doesn't exist
- `getById` endpoint doesn't exist

---

## Timeline API (`timelineApi`)

**File:** `apps/web/src/lib/api/timeline.ts`

| Method | HTTP | Frontend Endpoint | Backend Route | Match |
|--------|------|-------------------|---------------|-------|
| list | GET | /care-recipients/:careRecipientId/timeline | GET /care-recipients/:careRecipientId/timeline | ✅ |
| getVitalsHistory | GET | /care-recipients/:careRecipientId/timeline/vitals | GET /care-recipients/:careRecipientId/timeline/vitals | ✅ |
| getIncidents | GET | /care-recipients/:careRecipientId/timeline/incidents | ❌ NO BACKEND ROUTE | ❌ |
| create | POST | /care-recipients/:careRecipientId/timeline | POST /care-recipients/:careRecipientId/timeline | ✅ |
| update | PATCH | /timeline/:id | PATCH /care-recipients/:careRecipientId/timeline/:id | ⚠️ PATH MISMATCH |
| delete | DELETE | /timeline/:id | DELETE /care-recipients/:careRecipientId/timeline/:id | ⚠️ PATH MISMATCH |

**Issues Found:** 3
- `getIncidents` endpoint doesn't exist (use `type` query param with `list` instead)
- `update` uses `/timeline/:id` but backend requires `/care-recipients/:careRecipientId/timeline/:id`
- `delete` uses `/timeline/:id` but backend requires `/care-recipients/:careRecipientId/timeline/:id`

---

## Notifications API (`notificationsApi`)

**File:** `apps/web/src/lib/api/notifications.ts`

| Method | HTTP | Frontend Endpoint | Backend Route | Match |
|--------|------|-------------------|---------------|-------|
| list | GET | /notifications | GET /notifications | ✅ |
| getUnread | GET | /notifications/unread | ❌ NO BACKEND ROUTE | ❌ |
| getUnreadCount | GET | /notifications/unread/count | GET /notifications/unread/count | ✅ |
| markAsRead | PATCH | /notifications/read | PATCH /notifications/:notificationId/read | ⚠️ PATH MISMATCH |
| markAllAsRead | PATCH | /notifications/read/all | PATCH /notifications/read/all | ✅ |
| subscribeToPush | POST | /notifications/push-subscription | POST /notifications/push-token | ⚠️ PATH MISMATCH |
| unsubscribeFromPush | DELETE | /notifications/push-subscription | DELETE /notifications/push-token | ⚠️ PATH MISMATCH |

**Issues Found:** 4
- `getUnread` endpoint doesn't exist (use `unreadOnly=true` query param with `list`)
- `markAsRead` uses bulk `/notifications/read` with `{ ids }` but backend uses `/notifications/:notificationId/read`
- `subscribeToPush` uses `/notifications/push-subscription` but backend is `/notifications/push-token`
- `unsubscribeFromPush` uses `/notifications/push-subscription` but backend is `/notifications/push-token`

---

## Issues Summary

### Critical Path Mismatches (Will 404)

| API | Method | Frontend Path | Backend Path |
|-----|--------|---------------|--------------|
| authApi | updateProfile | PATCH /auth/me | ❌ Does not exist |
| familyApi | getMembers | GET /families/:familyId/members | ❌ Does not exist |
| familyApi | getPendingInvitations | GET /families/:familyId/invitations | ❌ Does not exist |
| familyApi | acceptInvitation | POST /families/accept-invite/:token | POST /families/invitations/:token/accept |
| familyApi | cancelInvitation | DELETE /families/:familyId/invitations/:id | DELETE /families/invitations/:id |
| familyApi | resendInvitation | POST /families/:familyId/invitations/:id/resend | POST /families/invitations/:id/resend |
| medicationsApi | get | GET /medications/:id | GET /care-recipients/:careRecipientId/medications/:id |
| medicationsApi | update | PATCH /medications/:id | PATCH /care-recipients/:careRecipientId/medications/:id |
| medicationsApi | delete | DELETE /medications/:id | DELETE /care-recipients/:careRecipientId/medications/:id |
| medicationsApi | getLowSupply | GET /.../medications/low-supply | ❌ Does not exist |
| medicationsApi | updateSupply | PATCH /medications/:id/supply | ❌ Does not exist |
| appointmentsApi | get | GET /appointments/:id | GET /care-recipients/:careRecipientId/appointments/:id |
| appointmentsApi | update | PATCH /appointments/:id | PATCH /care-recipients/:careRecipientId/appointments/:id |
| appointmentsApi | cancel | PATCH /appointments/:id/cancel | PATCH /care-recipients/:careRecipientId/appointments/:id/cancel |
| appointmentsApi | delete | DELETE /appointments/:id | DELETE /care-recipients/:careRecipientId/appointments/:id |
| appointmentsApi | assignTransport | POST /appointments/:id/transport | POST /care-recipients/:careRecipientId/appointments/:id/transport |
| documentsApi | getSignedUrl | GET /.../documents/:id/url | ❌ Does not exist |
| documentsApi | getByCategory | GET /.../documents/by-category | ❌ Does not exist |
| shiftsApi | getAll | GET /.../shifts | ❌ Does not exist (only filtered views) |
| shiftsApi | getByDateRange | GET /.../shifts/range | ❌ Does not exist |
| shiftsApi | getOnDuty | GET /.../shifts/on-duty | ❌ Does not exist |
| shiftsApi | getById | GET /.../shifts/:shiftId | ❌ Does not exist |
| timelineApi | getIncidents | GET /.../timeline/incidents | ❌ Does not exist (use type filter) |
| timelineApi | update | PATCH /timeline/:id | PATCH /care-recipients/:careRecipientId/timeline/:id |
| timelineApi | delete | DELETE /timeline/:id | DELETE /care-recipients/:careRecipientId/timeline/:id |
| notificationsApi | getUnread | GET /notifications/unread | ❌ Does not exist (use unreadOnly param) |
| notificationsApi | markAsRead | PATCH /notifications/read | PATCH /notifications/:notificationId/read |
| notificationsApi | subscribeToPush | POST /notifications/push-subscription | POST /notifications/push-token |
| notificationsApi | unsubscribeFromPush | DELETE /notifications/push-subscription | DELETE /notifications/push-token |

---

## Pattern Issues

### 1. Missing careRecipientId in nested routes

Several frontend methods use direct `/medications/:id`, `/appointments/:id`, `/timeline/:id` routes but backend requires the full nested path with careRecipientId:

```
Frontend: /medications/:id
Backend:  /care-recipients/:careRecipientId/medications/:id

Frontend: /appointments/:id
Backend:  /care-recipients/:careRecipientId/appointments/:id

Frontend: /timeline/:id
Backend:  /care-recipients/:careRecipientId/timeline/:id
```

### 2. Inconsistent invitation paths

```
Frontend: /families/accept-invite/:token
Backend:  /families/invitations/:token/accept

Frontend: /families/:familyId/invitations/:invitationId
Backend:  /families/invitations/:invitationId (no familyId)
```

### 3. Push notification endpoint naming

```
Frontend: /notifications/push-subscription
Backend:  /notifications/push-token
```

---

## Statistics

| Category | Count |
|----------|-------|
| Total Frontend API Methods | 92 |
| Matching Backend Routes | 57 |
| Path Mismatches | 20 |
| Missing Backend Routes | 15 |
| **Total Issues** | **35** |

---

## Recommendations

1. **Add careRecipientId to nested resource methods** in medications, appointments, and timeline APIs
2. **Fix invitation routes** to match backend pattern
3. **Rename push notification endpoints** from `push-subscription` to `push-token`
4. **Remove or implement missing endpoints:**
   - PATCH /auth/me (updateProfile)
   - GET /families/:familyId/members
   - GET /families/:familyId/invitations
   - GET /.../medications/low-supply
   - PATCH /medications/:id/supply
   - GET /.../documents/:id/url
   - GET /.../documents/by-category
   - GET /.../shifts (all)
   - GET /.../shifts/range
   - GET /.../shifts/on-duty
   - GET /.../shifts/:id
   - GET /.../timeline/incidents
   - GET /notifications/unread
