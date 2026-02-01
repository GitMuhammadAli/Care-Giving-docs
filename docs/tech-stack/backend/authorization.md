# Authorization Concepts

> Understanding how CareCircle controls what users can do.

---

## 1. What Is Authorization?

### Plain English Explanation

Authorization answers: **"Are you allowed to do this?"**

Think of it like a **concert venue**:
- Authentication = Ticket check at the door (Are you who you say you are?)
- Authorization = VIP section bouncer (Do you have access to this area?)

### Authentication vs Authorization

```
AUTHENTICATION: "Who are you?"
  → Verify identity (login)
  → Result: User object with ID

AUTHORIZATION: "What can you do?"
  → Check permissions
  → Result: Allow or deny action
```

---

## 2. CareCircle's Authorization Model

### Two-Level Access Control

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    CARECIRCLE AUTHORIZATION MODEL                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  LEVEL 1: FAMILY MEMBERSHIP                                                  │
│  ──────────────────────────                                                  │
│  "Is this user a member of Family X?"                                        │
│                                                                              │
│  User ──► FamilyMember ──► Family ──► Resources                             │
│                                                                              │
│  If not a member → 403 Forbidden (no access to any family resources)        │
│                                                                              │
│  LEVEL 2: ROLE-BASED ACCESS (RBAC)                                          │
│  ─────────────────────────────────                                          │
│  "What role does this member have?"                                         │
│                                                                              │
│  ┌───────────┬────────────────────────────────────────────────────────────┐ │
│  │   ADMIN   │ Full access: invite members, delete care recipients,      │ │
│  │           │ manage settings, reset passwords, all CAREGIVER actions   │ │
│  ├───────────┼────────────────────────────────────────────────────────────┤ │
│  │ CAREGIVER │ Care actions: log medications, create shifts, update      │ │
│  │           │ appointments, add timeline entries, all VIEWER actions    │ │
│  ├───────────┼────────────────────────────────────────────────────────────┤ │
│  │  VIEWER   │ Read-only: view medications, appointments, documents,     │ │
│  │           │ timeline entries (cannot modify anything)                 │ │
│  └───────────┴────────────────────────────────────────────────────────────┘ │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Why This Model

| Requirement | Solution |
|-------------|----------|
| Data isolation between families | Family-scoped queries |
| Different permission levels | RBAC with 3 roles |
| Elderly family member access | VIEWER role (read-only) |
| Caregiver accountability | Actions tracked by user |

---

## 3. How It Works in Code

### The Guard Chain

```
Request arrives
     │
     ▼
┌─────────────────┐
│  JwtAuthGuard   │  Extract user from JWT
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│FamilyAccessGuard│  Check user is family member
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   RolesGuard    │  Check user has required role
└────────┬────────┘
         │
         ▼
    Controller
```

### Guard Implementation Pattern

```typescript
// 1. JwtAuthGuard - Already authenticated, attach user to request
request.user = { id: 'user-123', email: 'user@example.com' };

// 2. FamilyAccessGuard - Check family membership
const membership = await familyMemberRepo.findOne({
  where: { userId: request.user.id, familyId: request.params.familyId }
});
if (!membership) throw new ForbiddenException('Not a family member');
request.familyMembership = membership;

// 3. RolesGuard - Check role permission
const requiredRoles = ['ADMIN', 'CAREGIVER'];  // From @Roles() decorator
if (!requiredRoles.includes(request.familyMembership.role)) {
  throw new ForbiddenException('Insufficient permissions');
}
```

---

## 4. When to Use Each Pattern ✅

### Use Family Scoping When:
- Accessing any care recipient data
- Accessing family-level resources
- Any endpoint with `:familyId` parameter

### Use Role Checks When:
- Modifying data (POST, PUT, DELETE)
- Performing administrative actions
- Actions that affect other members

### Use No Authorization When:
- Public endpoints (health checks)
- User's own profile data
- Cross-family operations (rare)

---

## 5. When to AVOID Patterns ❌

### DON'T Skip Family Scoping

```typescript
// ❌ BAD: Direct query without family check
async getMedication(id: string) {
  return this.medicationRepo.findOne({ where: { id } });
  // Any authenticated user can access any medication!
}

// ✅ GOOD: Family-scoped query
async getMedication(id: string, userId: string) {
  return this.medicationRepo.findOne({
    where: {
      id,
      careRecipient: {
        family: {
          members: { userId }
        }
      }
    }
  });
}
```

### DON'T Trust Client Data

```typescript
// ❌ BAD: Use familyId from request body
async create(@Body() dto: { familyId: string; name: string }) {
  // User could send any familyId!
}

// ✅ GOOD: Validate familyId through guard
@UseGuards(FamilyAccessGuard)
async create(
  @Param('familyId') familyId: string,  // Validated by guard
  @Body() dto: { name: string }
) { }
```

---

## 6. Best Practices

### Always Apply Guards in Order

```typescript
@UseGuards(JwtAuthGuard, FamilyAccessGuard, RolesGuard)
@Roles('ADMIN', 'CAREGIVER')
@Post()
async create() { }
```

### Use Decorators for Clean Code

```typescript
// Custom decorator combining common guards
@ApplyGuards()  // Custom decorator that applies all three guards
@Roles('ADMIN')
@Delete(':id')
async delete() { }
```

### Query Pattern for Family Scoping

```typescript
// Always join through family membership
this.repository
  .createQueryBuilder('medication')
  .innerJoin('medication.careRecipient', 'cr')
  .innerJoin('cr.family', 'f')
  .innerJoin('f.members', 'fm')
  .where('fm.userId = :userId', { userId })
  .andWhere('medication.id = :id', { id })
  .getOne();
```

---

## 7. Common Mistakes & How to Avoid Them

### Mistake 1: Forgetting Guards on New Endpoints

Every new endpoint needs authorization guards. Add them immediately when creating endpoints.

### Mistake 2: Role Confusion

```
ADMIN can do everything CAREGIVER can do
CAREGIVER can do everything VIEWER can do

Roles are HIERARCHICAL, not separate
```

### Mistake 3: Not Testing Authorization

Always write tests that verify:
- Unauthenticated request → 401
- Non-member request → 403
- Wrong role request → 403
- Correct role request → 200

---

## 8. Role Permission Matrix

| Action | ADMIN | CAREGIVER | VIEWER |
|--------|-------|-----------|--------|
| View medications | ✅ | ✅ | ✅ |
| Log medication | ✅ | ✅ | ❌ |
| Create medication | ✅ | ✅ | ❌ |
| Delete medication | ✅ | ❌ | ❌ |
| View family members | ✅ | ✅ | ✅ |
| Invite members | ✅ | ❌ | ❌ |
| Remove members | ✅ | ❌ | ❌ |
| Reset member password | ✅ | ❌ | ❌ |
| Create shifts | ✅ | ✅ | ❌ |
| Delete family | ✅ | ❌ | ❌ |

---

## 9. Quick Reference

### Guard Decorators

```typescript
@UseGuards(JwtAuthGuard)           // Must be authenticated
@UseGuards(FamilyAccessGuard)      // Must be family member
@UseGuards(RolesGuard)             // Must have required role
@Roles('ADMIN')                    // Specify required roles
@Roles('ADMIN', 'CAREGIVER')       // Multiple roles allowed
```

### Common Response Codes

| Code | Meaning |
|------|---------|
| 401 | Not authenticated (no/invalid token) |
| 403 | Not authorized (wrong role/not member) |
| 404 | Resource not found (after auth passes) |

---

*Next: [Authentication](authentication.md) | [Security Principles](../security/principles.md)*

