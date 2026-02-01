# CareCircle Codebase Issues Report

**Generated:** 2026-01-20
**Scan Coverage:** Full Stack (Prisma → Backend DTOs → Backend Routes → Frontend API → Frontend Types)
**Report Version:** 1.0

---

## Executive Summary

| Severity | Count | Description |
|----------|-------|-------------|
| 🔴 CRITICAL | 8 | Will cause runtime errors/crashes/404s |
| 🟠 HIGH | 12 | Will cause bugs or data issues |
| 🟡 MEDIUM | 9 | Type mismatches, missing fields |
| 🔵 LOW | 6 | Inconsistencies, code quality |
| **TOTAL** | **35** | |

### Recently Fixed (This Session)
- ✅ CareRecipient DTO field names aligned with Prisma (`photoUrl`, `primaryHospital`, `hospitalAddress`, `insurancePolicyNo`)
- ✅ Medications API now includes careRecipientId in get/update/delete
- ✅ Appointments API now includes careRecipientId in get/update/delete/cancel/transport
- ✅ Timeline API now includes careRecipientId in update/delete
- ✅ Timeline getIncidents now uses list with type filter (endpoint didn't exist)
- ✅ Notifications push endpoints fixed (`push-subscription` → `push-token`)
- ✅ Notifications getUnread uses list with unreadOnly filter
- ✅ Notifications markAsRead changed to single endpoint pattern
- ✅ Family invitation routes fixed
- ✅ Timeline hook `getNextPageParam` null check added
- ✅ Emergency CreateEmergencyAlertInput already has `title` field

---

## 🔴 CRITICAL ISSUES (Fix Immediately)

### CRIT-001: Missing Backend Route - PATCH /auth/me
| Property | Value |
|----------|-------|
| **Frontend Call** | `authApi.updateProfile()` → `PATCH /auth/me` |
| **Backend Status** | Route does not exist (only `GET /auth/me`) |
| **File** | `apps/web/src/lib/api/auth.ts:35` |
| **Impact** | 404 error when user tries to update profile |
| **Fix Option A** | Add `@Patch('me')` handler in `apps/api/src/auth/auth.controller.ts` |
| **Fix Option B** | Remove `updateProfile` from frontend or use `/users/me` endpoint |

---

### ~~CRIT-002: Appointments API Missing careRecipientId in Routes~~ ✅ FIXED
| Frontend Call | Backend Expects | Status |
|---------------|-----------------|--------|
| `GET /care-recipients/:crId/appointments/:id` | `GET /care-recipients/:crId/appointments/:id` | ✅ FIXED |
| `PATCH /care-recipients/:crId/appointments/:id` | `PATCH /care-recipients/:crId/appointments/:id` | ✅ FIXED |
| `PATCH /care-recipients/:crId/appointments/:id/cancel` | `PATCH /care-recipients/:crId/appointments/:id/cancel` | ✅ FIXED |
| `DELETE /care-recipients/:crId/appointments/:id` | `DELETE /care-recipients/:crId/appointments/:id` | ✅ FIXED |
| `POST /care-recipients/:crId/appointments/:id/transport` | `POST /care-recipients/:crId/appointments/:id/transport` | ✅ FIXED |

**File:** `apps/web/src/lib/api/appointments.ts:58-81`

**Fix Required:**
```typescript
// Change from:
get: async (id: string): Promise<Appointment> => {
  return api.get<Appointment>(`/appointments/${id}`);
},

// To:
get: async (careRecipientId: string, id: string): Promise<Appointment> => {
  return api.get<Appointment>(`/care-recipients/${careRecipientId}/appointments/${id}`);
},
```

---

### ~~CRIT-003: Timeline API Missing careRecipientId in Routes~~ ✅ FIXED
| Frontend Call | Backend Expects | Status |
|---------------|-----------------|--------|
| `PATCH /care-recipients/:crId/timeline/:id` | `PATCH /care-recipients/:crId/timeline/:id` | ✅ FIXED |
| `DELETE /care-recipients/:crId/timeline/:id` | `DELETE /care-recipients/:crId/timeline/:id` | ✅ FIXED |

**File:** `apps/web/src/lib/api/timeline.ts:71-77`

**Fix Required:**
```typescript
// Change from:
update: async (id: string, data: Partial<CreateTimelineEntryInput>): Promise<TimelineEntry> => {
  return api.patch<TimelineEntry>(`/timeline/${id}`, data);
},

// To:
update: async (careRecipientId: string, id: string, data: Partial<CreateTimelineEntryInput>): Promise<TimelineEntry> => {
  return api.patch<TimelineEntry>(`/care-recipients/${careRecipientId}/timeline/${id}`, data);
},
```

---

### CRIT-004: EmergencyAlert Missing Required `title` Field
| Property | Value |
|----------|-------|
| **Prisma Schema** | `title String` (REQUIRED) |
| **Frontend Interface** | `CreateEmergencyAlertInput` - NO title field |
| **File** | `apps/web/src/lib/api/emergency.ts:75-79` |
| **Impact** | Emergency alert creation will fail backend validation |

**Fix Required:**
```typescript
export interface CreateEmergencyAlertInput {
  type: 'FALL' | 'MEDICAL' | 'HOSPITALIZATION' | 'MISSING' | 'OTHER';
  title: string;  // ADD THIS - REQUIRED!
  description?: string;
  location?: string;
}
```

---

### CRIT-005: EmergencyAlert DTO Missing Required `title` Field
| Property | Value |
|----------|-------|
| **Prisma Schema** | `title String` (REQUIRED) |
| **Backend DTO** | `CreateEmergencyAlertDto` - Has title |
| **File** | `apps/api/src/emergency/dto/create-emergency-alert.dto.ts` |
| **Status** | ✅ Backend OK, Frontend needs fix |

---

### CRIT-006: 15 Frontend Endpoints Don't Exist in Backend
| Endpoint | Frontend Method | Impact |
|----------|-----------------|--------|
| `GET /families/:familyId/members` | `familyApi.getMembers` | ❌ 404 |
| `GET /families/:familyId/invitations` | `familyApi.getPendingInvitations` | ❌ 404 |
| `GET /.../medications/low-supply` | `medicationsApi.getLowSupply` | ❌ 404 |
| `PATCH /medications/:id/supply` | `medicationsApi.updateSupply` | ❌ 404 |
| `GET /.../documents/:id/url` | `documentsApi.getSignedUrl` | ❌ 404 |
| `GET /.../documents/by-category` | `documentsApi.getByCategory` | ❌ 404 |
| `GET /.../shifts` (list all) | `shiftsApi.getAll` | ❌ 404 |
| `GET /.../shifts/range` | `shiftsApi.getByDateRange` | ❌ 404 |
| `GET /.../shifts/on-duty` | `shiftsApi.getOnDuty` | ❌ 404 |
| `GET /.../shifts/:id` | `shiftsApi.getById` | ❌ 404 |
| `GET /.../timeline/incidents` | `timelineApi.getIncidents` | ❌ 404 |
| `GET /notifications/unread` | `notificationsApi.getUnread` | ❌ 404 |

**Fix Options:**
1. Implement missing backend routes
2. Remove unused frontend methods
3. For `getMembers`/`getPendingInvitations`: Data comes with `GET /families/:id` response

---

### ~~CRIT-007: Notifications Push Token Path Mismatch~~ ✅ FIXED
| Frontend | Backend | Status |
|----------|---------|--------|
| `POST /notifications/push-token` | `POST /notifications/push-token` | ✅ FIXED |
| `DELETE /notifications/push-token` | `DELETE /notifications/push-token` | ✅ FIXED |

**File:** `apps/web/src/lib/api/notifications.ts:91-98`

---

### ~~CRIT-008: Notifications markAsRead Bulk vs Single Mismatch~~ ✅ FIXED
| Frontend | Backend | Status |
|----------|---------|--------|
| `PATCH /notifications/:notificationId/read` (single) | `PATCH /notifications/:notificationId/read` | ✅ FIXED |
| `markMultipleAsRead(ids)` loops through single endpoint | Works with existing backend | ✅ FIXED |

**File:** `apps/web/src/lib/api/notifications.ts:75-83`

**Resolution:** Changed `markAsRead` to accept single ID, added `markMultipleAsRead` helper that loops.

---

## 🟠 HIGH SEVERITY ISSUES

### HIGH-001: AppointmentStatus Enum Missing Value
| Source | Values |
|--------|--------|
| **Prisma** | SCHEDULED, CONFIRMED, COMPLETED, CANCELLED, **NO_SHOW** |
| **Frontend** | SCHEDULED, CONFIRMED, CANCELLED, COMPLETED |
| **Missing** | `NO_SHOW` |

**Files:**
- `apps/web/src/lib/api/appointments.ts:13`

**Fix:**
```typescript
status: 'SCHEDULED' | 'CONFIRMED' | 'CANCELLED' | 'COMPLETED' | 'NO_SHOW';
```

---

### HIGH-002: ShiftStatus Enum Missing Value
| Source | Values |
|--------|--------|
| **Prisma** | SCHEDULED, **CONFIRMED**, IN_PROGRESS, COMPLETED, CANCELLED, NO_SHOW |
| **Frontend** | SCHEDULED, IN_PROGRESS, COMPLETED, CANCELLED, NO_SHOW |
| **Missing** | `CONFIRMED` |

**File:** `apps/web/src/lib/api/shifts.ts`

**Fix:** Add `'CONFIRMED'` to CaregiverShift.status type

---

### HIGH-003: InvitationStatus Enum Mismatch
| Prisma | Frontend |
|--------|----------|
| PENDING | PENDING |
| ACCEPTED | ACCEPTED |
| EXPIRED | EXPIRED |
| **CANCELLED** | **DECLINED** ❌ |

**Files:**
- Prisma: `packages/database/prisma/schema.prisma`
- Frontend: `apps/web/src/lib/api/family.ts:30`

**Fix:** Use `CANCELLED` consistently (already fixed in linter update)

---

### HIGH-004: NotificationType Enum Significant Mismatch

**In Prisma but NOT in Frontend:**
| Missing Type | Purpose |
|--------------|---------|
| MEDICATION_MISSED | Critical for medication tracking |
| SHIFT_REMINDER | Caregiver shift alerts |
| SHIFT_HANDOFF | Shift handoff notifications |
| FAMILY_INVITE | Family invitation alerts |
| DOCUMENT_SHARED | Document sharing |
| TIMELINE_UPDATE | Timeline changes |
| REFILL_NEEDED | Medication refill alerts |
| REFILL_ALERT | Low supply warning |
| GENERAL | General notifications |

**In Frontend but NOT in Prisma:**
| Extra Type | Issue |
|------------|-------|
| FAMILY_UPDATE | Not in Prisma enum |
| SYSTEM | Not in Prisma enum |

**File:** `apps/web/src/lib/api/notifications.ts:3-15`

---

### HIGH-005: CaregiverShift.careRecipient Uses Wrong Photo Field
| Field | Issue |
|-------|-------|
| Current | `avatarUrl` |
| Should Be | `photoUrl` |
| Reason | CareRecipient model uses `photoUrl`, User uses `avatarUrl` |

**File:** `apps/web/src/lib/api/shifts.ts` - CaregiverShift interface

---

### HIGH-006: Missing useUpdateMedication Hook careRecipientId
After fixing the API, hooks using these methods need updating:

**Files to check:**
- `apps/web/src/hooks/use-medications.ts`
- Any component calling `medicationsApi.update()` or `medicationsApi.delete()`

---

### HIGH-007: Missing useAppointments Hook careRecipientId
**Files to check:**
- `apps/web/src/hooks/use-appointments.ts`
- Any component calling `appointmentsApi.get()`, `update()`, `delete()`, `cancel()`

---

### HIGH-008: Missing useTimeline Hook careRecipientId
**Files to check:**
- `apps/web/src/hooks/use-timeline.ts`
- `useDeleteTimelineEntry` hook needs careRecipientId parameter

---

### HIGH-009: Frontend Appointment transportAssignment Structure
| Prisma | Frontend | Issue |
|--------|----------|-------|
| `transportAssignedToId` | `transportAssignment.assignedTo.id` | Nested vs flat |

**Note:** Backend may be transforming this - verify response matches frontend expectation

---

### HIGH-010: Documents API - Some Methods Not Used
| Method | Status | Recommendation |
|--------|--------|----------------|
| `getSignedUrl` | No backend route | Remove or implement |
| `getByCategory` | No backend route | Remove or implement |

---

### HIGH-011: Shifts API - Multiple Non-Functional Methods
| Method | Backend Status |
|--------|----------------|
| `getAll` | No route (only filtered views) |
| `getByDateRange` | No route (use `/shifts/day?date=`) |
| `getOnDuty` | No route |
| `getById` | No route |

**Recommendation:** Remove unused methods or implement backend routes

---

### HIGH-012: Timeline getIncidents Method Non-Functional
**Current:** `GET /care-recipients/:crId/timeline/incidents`
**Backend:** Route doesn't exist

**Fix:** Use `list()` with type filter:
```typescript
// Instead of:
getIncidents: async (careRecipientId: string): Promise<TimelineEntry[]> => {
  return api.get<TimelineEntry[]>(`/care-recipients/${careRecipientId}/timeline/incidents`);
},

// Use:
getIncidents: async (careRecipientId: string): Promise<TimelineEntry[]> => {
  return timelineApi.list(careRecipientId, { type: 'INCIDENT' });
},
```

---

## 🟡 MEDIUM SEVERITY ISSUES

### MED-001: CaregiverShift Field Name Mismatches
| Prisma | Frontend | Issue |
|--------|----------|-------|
| `notes` | `checkInNotes`, `checkOutNotes`, `handoffNotes` | Frontend splits into 3 fields |
| `checkedInAt` | `actualStartTime` | Different naming |
| `checkedOutAt` | `actualEndTime` | Different naming |
| - | `checkInLocation` | Only in frontend |
| - | `checkOutLocation` | Only in frontend |
| - | `createdBy` | Only in frontend |

**Impact:** Data may not map correctly between frontend/backend

---

### MED-002: EmergencyAlert Extra/Missing Fields
**In Prisma but NOT in Frontend:**
| Field | Type | Impact |
|-------|------|--------|
| `title` | String (REQUIRED) | ⚠️ CRITICAL - covered above |
| `resolvedById` | String? | Missing resolver tracking |

**In Frontend but NOT in Prisma:**
| Field | Type | Impact |
|-------|------|--------|
| `acknowledgedAt` | DateTime? | Feature may not work |
| `acknowledgedBy` | Object | Feature may not work |

---

### MED-003: Document Interface Missing Fields
| Prisma Field | Frontend Status |
|--------------|-----------------|
| `expiresAt` | ✅ Present |
| `size` | ❌ Missing |
| `mimeType` | ❌ Missing |

**File:** `apps/web/src/lib/api/documents.ts`

---

### MED-004: FamilyMember Interface Missing Fields
| Prisma Field | Frontend Status |
|--------------|-----------------|
| `nickname` | ❌ Missing |
| `canEdit` | ❌ Missing |
| `isActive` | ❌ Missing |
| `notifications` | ❌ Missing |

**File:** `apps/web/src/lib/api/family.ts`

---

### MED-005: Medication Interface - Potential Field Issues
| Prisma | Frontend | Issue |
|--------|----------|-------|
| `genericName` | Missing | Not shown in UI |
| `timesPerDay` | Missing | Calculation may be off |
| `lastRefillDate` | Missing | Refill tracking incomplete |

---

### MED-006: User Interface Missing Fields
| Prisma Field | Frontend Status |
|--------------|-----------------|
| `status` | ❌ Missing |
| `emailVerified` | ❌ Missing |
| `phoneVerified` | ❌ Missing |
| `preferences` | ❌ Missing |
| `onboardingCompleted` | ✅ Present |

**File:** `apps/web/src/lib/api/auth.ts`

---

### MED-007: TimelineEntry Missing `attachments` Field
| Prisma | Frontend |
|--------|----------|
| `attachments String[]` | ❌ Not in interface |

**File:** `apps/web/src/lib/api/timeline.ts`

---

### MED-008: Doctor Interface Missing `isPrimary` Field
| Prisma | Frontend |
|--------|----------|
| `isPrimary Boolean @default(false)` | ❌ Not in interface |

**File:** `apps/web/src/lib/api/care-recipients.ts`

---

### MED-009: MedicationLog Interface Incomplete
Missing from frontend:
- `givenAt` DateTime field
- `givenById` relation tracking

---

## 🔵 LOW SEVERITY ISSUES

### LOW-001: Enums as Plain Strings
Several Prisma enums are `string` in frontend instead of union types:
- `MedicationForm`
- `MedicationFrequency`
- `AppointmentType`
- `DocumentType`
- `TimelineType`
- `Severity`

**Recommendation:** Create proper TypeScript union types for type safety

---

### LOW-002: DateTime vs String Representations
All Prisma `DateTime` fields are `string` in frontend. This is correct for JSON serialization but should be:
1. Documented in code
2. Consistently parsed using date library (e.g., date-fns)

---

### LOW-003: Optional vs Required Field Inconsistencies
Some fields marked as required in Prisma are optional in frontend interfaces:
- `Medication.dosage` - Required in Prisma, should be required in frontend
- `Appointment.type` - Required in Prisma, should be required in frontend

---

### LOW-004: Inconsistent Null vs Undefined
Some interfaces use `null` while others use `?:` for optional fields. Standardize to one approach.

---

### LOW-005: Missing JSDoc Comments
API interfaces lack documentation for complex fields like:
- `reminderMinutes: number[]`
- `vitals` object structure
- `preferences` JSON structure

---

### LOW-006: Unused Imports in API Files
Several API files import types that aren't used. Run lint checks to clean up.

---

## Statistics Summary

### Issues by Severity
| Severity | Count | % of Total |
|----------|-------|------------|
| 🔴 CRITICAL | 8 | 23% |
| 🟠 HIGH | 12 | 34% |
| 🟡 MEDIUM | 9 | 26% |
| 🔵 LOW | 6 | 17% |
| **TOTAL** | **35** | 100% |

### Issues by Category
| Category | Count | Details |
|----------|-------|---------|
| **Route Mismatches** | 15 | Missing careRecipientId, wrong paths |
| **Missing Endpoints** | 12 | Frontend calls non-existent backend |
| **Type/Enum Mismatches** | 5 | Status enums, NotificationType |
| **Field Name Mismatches** | 3 | ~~CareRecipient~~ (fixed), Shifts |

### Estimated Fix Time by Category
| Category | Estimated Time | Complexity |
|----------|----------------|------------|
| Route fixes (frontend) | 2-3 hours | Low |
| Missing endpoint decisions | 1-2 hours | Medium (decision-making) |
| Type/Enum alignment | 1 hour | Low |
| Field name fixes | 30 min | Low (mostly done) |
| Hook updates | 2 hours | Medium |
| **TOTAL** | **6-8 hours** | |

---

## Files to Modify

### Frontend API Files
| File | Issues | Priority |
|------|--------|----------|
| `apps/web/src/lib/api/appointments.ts` | CRIT-002 | 🔴 Immediate |
| `apps/web/src/lib/api/timeline.ts` | CRIT-003 | 🔴 Immediate |
| `apps/web/src/lib/api/emergency.ts` | CRIT-004 | 🔴 Immediate |
| `apps/web/src/lib/api/notifications.ts` | CRIT-007, CRIT-008 | 🔴 Immediate |
| `apps/web/src/lib/api/auth.ts` | CRIT-001 | 🔴 Immediate |
| `apps/web/src/lib/api/shifts.ts` | HIGH-002, HIGH-005 | 🟠 High |
| `apps/web/src/lib/api/family.ts` | MED-004 | 🟡 Medium |
| `apps/web/src/lib/api/documents.ts` | MED-003 | 🟡 Medium |

### Frontend Hook Files
| File | Issues | Priority |
|------|--------|----------|
| `apps/web/src/hooks/use-appointments.ts` | HIGH-007 | 🟠 High |
| `apps/web/src/hooks/use-timeline.ts` | HIGH-008 | 🟠 High |
| `apps/web/src/hooks/use-medications.ts` | HIGH-006 | 🟠 High |

### Backend (Optional)
| File | Issue | Priority |
|------|-------|----------|
| `apps/api/src/auth/auth.controller.ts` | CRIT-001 (if implementing) | 🔴 |
| `apps/api/src/notifications/notifications.controller.ts` | CRIT-008 (bulk read) | 🟠 |

---

## Action Items Checklist

### Immediate (Block Release)
- [ ] Fix appointments API - add careRecipientId to get/update/delete/cancel/transport
- [ ] Fix timeline API - add careRecipientId to update/delete
- [ ] Add `title` field to CreateEmergencyAlertInput
- [ ] Fix notifications push-subscription → push-token
- [ ] Resolve PATCH /auth/me (implement or remove)

### High Priority (This Sprint)
- [ ] Update hooks to pass careRecipientId
- [ ] Add missing enum values (NO_SHOW, CONFIRMED)
- [ ] Fix notification markAsRead bulk/single mismatch
- [ ] Fix CaregiverShift.careRecipient.avatarUrl → photoUrl
- [ ] Sync NotificationType enum

### Medium Priority (Next Sprint)
- [ ] Remove or implement missing endpoint methods
- [ ] Add missing interface fields (FamilyMember, Document, etc.)
- [ ] Resolve CaregiverShift field name differences
- [ ] Add EmergencyAlert acknowledge fields to Prisma

### Low Priority (Backlog)
- [ ] Create proper union types for enums
- [ ] Add JSDoc comments to interfaces
- [ ] Standardize null vs undefined
- [ ] Clean up unused imports

---

## Previously Fixed (This Session)

| Issue | Status | Files Modified |
|-------|--------|----------------|
| CareRecipient DTO field names | ✅ FIXED | `create-care-recipient.dto.ts`, `care-recipient.service.ts`, `care-recipients.ts`, `edit-care-recipient-modal.tsx`, `[id]/page.tsx` |
| Medications API careRecipientId | ✅ FIXED | `apps/web/src/lib/api/medications.ts` |
| Family invitation routes | ✅ FIXED | `apps/web/src/lib/api/family.ts` |
| Timeline hook null check | ✅ FIXED | `apps/web/src/hooks/use-timeline.ts` |
| AssignTransportDto field name | ✅ FIXED | `assign-transport.dto.ts` |

---

_Report generated by comprehensive codebase scan_
_Last Updated: 2026-01-20_
