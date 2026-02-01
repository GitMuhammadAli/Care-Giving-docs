# 🔑 Authorization Patterns - Complete Guide

> A comprehensive guide to authorization patterns - RBAC, ABAC, permissions, policies, ACLs, and implementing fine-grained access control.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Authorization determines WHAT an authenticated user can do - from simple role checks (admin/user) to complex attribute-based policies that consider context like time, location, and resource ownership."

### Authorization Mental Model
```
AUTHORIZATION MODELS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  RBAC (Role-Based Access Control)                               │
│  ─────────────────────────────────                              │
│  User → Role → Permissions                                      │
│                                                                  │
│  user.role = 'admin'                                           │
│  admin → [create, read, update, delete]                        │
│  user  → [read]                                                │
│                                                                  │
│  Simple, widely used, easy to understand                       │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  ABAC (Attribute-Based Access Control)                         │
│  ──────────────────────────────────────                         │
│  Policy based on attributes of:                                │
│  • Subject (user): role, department, clearance                 │
│  • Resource: owner, type, sensitivity                          │
│  • Action: read, write, delete                                 │
│  • Environment: time, location, device                         │
│                                                                  │
│  Example: "Allow if user.department == resource.department     │
│            AND time is between 9am-5pm"                         │
│                                                                  │
│  Flexible, complex, harder to audit                            │
│                                                                  │
│  ════════════════════════════════════════════════════════════   │
│                                                                  │
│  ACL (Access Control List)                                     │
│  ──────────────────────────                                     │
│  Resource → List of (User/Group, Permissions)                  │
│                                                                  │
│  document_123:                                                 │
│    - alice: [read, write]                                      │
│    - bob: [read]                                                │
│    - editors_group: [read, write]                              │
│                                                                  │
│  Per-resource permissions, like file systems                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### The "Wow" Statement
> "We evolved from simple RBAC to a hybrid model. Started with roles (admin, editor, viewer), but needed fine-grained control - editors should only edit their own team's content. We implemented RBAC + ownership checks: roles grant capabilities, but resource access also requires ownership or explicit sharing. For cross-team access, we use ACLs on specific resources. The key insight: start with RBAC for simplicity, add ABAC attributes as needed, use ACLs for exceptions. All authorization logic is centralized in a policy service, so changes are consistent across the app."

---

## 📚 Core Patterns

### RBAC (Role-Based Access Control)

```typescript
// ════════════════════════════════════════════════════════════════
// BASIC RBAC IMPLEMENTATION
// ════════════════════════════════════════════════════════════════

// Define roles and permissions
const ROLES = {
    admin: ['users:read', 'users:write', 'users:delete', 'posts:read', 'posts:write', 'posts:delete'],
    editor: ['posts:read', 'posts:write', 'posts:delete'],
    viewer: ['posts:read']
} as const;

type Role = keyof typeof ROLES;
type Permission = typeof ROLES[Role][number];

// Check permission
function hasPermission(userRole: Role, permission: Permission): boolean {
    return ROLES[userRole]?.includes(permission) ?? false;
}

// Middleware
function requirePermission(permission: Permission) {
    return (req, res, next) => {
        if (!hasPermission(req.user.role, permission)) {
            return res.status(403).json({ error: 'Forbidden' });
        }
        next();
    };
}

// Usage
app.delete('/api/posts/:id', 
    authenticate,
    requirePermission('posts:delete'),
    async (req, res) => {
        await db.posts.delete(req.params.id);
        res.json({ success: true });
    }
);

// ════════════════════════════════════════════════════════════════
// HIERARCHICAL RBAC
// ════════════════════════════════════════════════════════════════

const ROLE_HIERARCHY = {
    superadmin: ['admin'],
    admin: ['editor'],
    editor: ['viewer'],
    viewer: []
};

function getAllPermissions(role: string): Set<string> {
    const permissions = new Set(ROLES[role] || []);
    
    // Add inherited permissions
    for (const parentRole of ROLE_HIERARCHY[role] || []) {
        for (const perm of getAllPermissions(parentRole)) {
            permissions.add(perm);
        }
    }
    
    return permissions;
}

// superadmin gets admin + editor + viewer permissions
```

### ABAC (Attribute-Based Access Control)

```typescript
// ════════════════════════════════════════════════════════════════
// ABAC IMPLEMENTATION
// ════════════════════════════════════════════════════════════════

interface Subject {
    id: string;
    role: string;
    department: string;
    clearanceLevel: number;
}

interface Resource {
    id: string;
    type: string;
    ownerId: string;
    department: string;
    sensitivityLevel: number;
}

interface Environment {
    time: Date;
    ipAddress: string;
    deviceType: string;
}

interface PolicyContext {
    subject: Subject;
    resource: Resource;
    action: string;
    environment: Environment;
}

// Policy definition
type Policy = (context: PolicyContext) => boolean;

const policies: Policy[] = [
    // Admins can do anything
    ({ subject }) => subject.role === 'admin',
    
    // Users can read own resources
    ({ subject, resource, action }) => 
        action === 'read' && resource.ownerId === subject.id,
    
    // Same department can read
    ({ subject, resource, action }) =>
        action === 'read' && subject.department === resource.department,
    
    // Clearance must be >= sensitivity
    ({ subject, resource }) =>
        subject.clearanceLevel >= resource.sensitivityLevel,
    
    // Business hours only for sensitive data
    ({ resource, environment }) => {
        if (resource.sensitivityLevel < 3) return true;
        const hour = environment.time.getHours();
        return hour >= 9 && hour < 17;
    }
];

// Evaluate all policies (all must pass)
function authorize(context: PolicyContext): boolean {
    return policies.every(policy => policy(context));
}

// Usage
app.get('/api/documents/:id', authenticate, async (req, res) => {
    const document = await db.documents.findById(req.params.id);
    
    const allowed = authorize({
        subject: req.user,
        resource: document,
        action: 'read',
        environment: {
            time: new Date(),
            ipAddress: req.ip,
            deviceType: req.headers['user-agent']
        }
    });
    
    if (!allowed) {
        return res.status(403).json({ error: 'Access denied' });
    }
    
    res.json(document);
});
```

### ACL (Access Control List)

```typescript
// ════════════════════════════════════════════════════════════════
// ACL IMPLEMENTATION
// ════════════════════════════════════════════════════════════════

// Database schema for ACL
interface AccessEntry {
    resourceId: string;
    resourceType: string;
    principalId: string;    // User or group ID
    principalType: 'user' | 'group';
    permissions: string[];  // ['read', 'write', 'delete', 'share']
}

class ACLService {
    // Grant permission
    async grant(
        resourceId: string,
        resourceType: string,
        principalId: string,
        principalType: 'user' | 'group',
        permissions: string[]
    ) {
        await db.accessEntries.upsert({
            where: { resourceId, principalId },
            update: { permissions },
            create: { resourceId, resourceType, principalId, principalType, permissions }
        });
    }
    
    // Revoke permission
    async revoke(resourceId: string, principalId: string) {
        await db.accessEntries.delete({
            where: { resourceId, principalId }
        });
    }
    
    // Check permission
    async hasPermission(
        userId: string,
        resourceId: string,
        permission: string
    ): Promise<boolean> {
        // Get user's groups
        const userGroups = await db.groupMembers
            .findMany({ where: { userId } })
            .then(members => members.map(m => m.groupId));
        
        // Check direct user access
        const userAccess = await db.accessEntries.findUnique({
            where: { resourceId, principalId: userId }
        });
        
        if (userAccess?.permissions.includes(permission)) {
            return true;
        }
        
        // Check group access
        const groupAccess = await db.accessEntries.findMany({
            where: {
                resourceId,
                principalId: { in: userGroups },
                principalType: 'group'
            }
        });
        
        return groupAccess.some(entry => 
            entry.permissions.includes(permission)
        );
    }
}

// Usage - Sharing a document
app.post('/api/documents/:id/share', authenticate, async (req, res) => {
    const { userId, permissions } = req.body;
    const documentId = req.params.id;
    
    // Check if current user can share
    const canShare = await acl.hasPermission(req.user.id, documentId, 'share');
    if (!canShare) {
        return res.status(403).json({ error: 'Cannot share this document' });
    }
    
    await acl.grant(documentId, 'document', userId, 'user', permissions);
    res.json({ success: true });
});
```

### Policy-Based Authorization (Centralized)

```typescript
// ════════════════════════════════════════════════════════════════
// CENTRALIZED POLICY SERVICE
// ════════════════════════════════════════════════════════════════

// Define policies declaratively
const policies = {
    'posts:read': [
        { effect: 'allow' }  // Everyone can read posts
    ],
    
    'posts:write': [
        { effect: 'allow', condition: 'isAuthenticated' }
    ],
    
    'posts:delete': [
        { effect: 'allow', condition: 'isOwner' },
        { effect: 'allow', condition: 'isAdmin' }
    ],
    
    'admin:access': [
        { effect: 'allow', condition: 'hasRole:admin' }
    ]
};

class PolicyEngine {
    private conditions = {
        isAuthenticated: (ctx) => !!ctx.user,
        isOwner: (ctx) => ctx.resource?.ownerId === ctx.user?.id,
        isAdmin: (ctx) => ctx.user?.role === 'admin',
        'hasRole:admin': (ctx) => ctx.user?.role === 'admin',
        'hasRole:editor': (ctx) => ctx.user?.role === 'editor',
    };
    
    evaluate(action: string, context: any): boolean {
        const rules = policies[action];
        if (!rules) return false;
        
        for (const rule of rules) {
            if (!rule.condition) {
                return rule.effect === 'allow';
            }
            
            const conditionFn = this.conditions[rule.condition];
            if (conditionFn && conditionFn(context)) {
                return rule.effect === 'allow';
            }
        }
        
        return false;  // Default deny
    }
}

// Middleware
const policyEngine = new PolicyEngine();

function authorize(action: string) {
    return async (req, res, next) => {
        const context = {
            user: req.user,
            resource: req.resource,  // Set by previous middleware
            environment: { ip: req.ip, time: new Date() }
        };
        
        if (!policyEngine.evaluate(action, context)) {
            return res.status(403).json({ error: 'Forbidden' });
        }
        
        next();
    };
}
```

---

## RBAC vs ABAC vs ACL

```
CHOOSING THE RIGHT MODEL:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  RBAC - Role-Based                                              │
│  ✓ Simple to implement and understand                          │
│  ✓ Easy to audit (who has what role)                           │
│  ✓ Works well for most apps                                    │
│  ✗ Role explosion (too many roles)                             │
│  ✗ Coarse-grained (can't express complex rules)               │
│  → Use for: Most web apps, clear role boundaries               │
│                                                                  │
│  ABAC - Attribute-Based                                         │
│  ✓ Very flexible, any condition                                │
│  ✓ Context-aware (time, location)                              │
│  ✓ Fine-grained control                                        │
│  ✗ Complex to implement                                        │
│  ✗ Hard to audit and debug                                     │
│  → Use for: Enterprise, compliance, complex rules              │
│                                                                  │
│  ACL - Access Control List                                      │
│  ✓ Per-resource granularity                                    │
│  ✓ User/group sharing                                          │
│  ✓ Familiar (file system model)                                │
│  ✗ Doesn't scale (many entries)                                │
│  ✗ Hard to answer "what can user X access?"                    │
│  → Use for: Document sharing, file systems                     │
│                                                                  │
│  HYBRID (Common in practice)                                    │
│  • RBAC for base permissions                                   │
│  • Ownership checks for resources                              │
│  • ACL for explicit sharing                                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

**Q: "RBAC vs ABAC?"**
> "RBAC assigns permissions to roles, users get roles. Simple, auditable, but coarse-grained - can't express 'only during business hours' or 'only own department'. ABAC uses attributes of user, resource, and environment to make decisions. Very flexible but complex. I typically use RBAC with ownership checks, adding ABAC only for complex compliance requirements."

**Q: "How do you handle authorization in microservices?"**
> "Two approaches: 1) Centralized - dedicated authorization service, all services call it. Consistent but adds latency. 2) Distributed - embed policies in each service, user claims in JWT. Fast but policies can drift. I prefer hybrid: JWT contains role/basic claims checked locally, complex policies call central service. Use caching to reduce latency."

**Q: "What's the principle of least privilege?"**
> "Users should have only the minimum permissions needed to do their job. In practice: default deny, explicit grants, regular permission audits, time-limited elevated access. For example, developer doesn't need prod database access normally - they request temporary access that auto-expires."

---

## Quick Reference

```
AUTHORIZATION CHEAT SHEET:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  RBAC: User → Role → Permissions                               │
│  ABAC: Policy(subject, resource, action, environment)          │
│  ACL:  Resource → [(principal, permissions)]                   │
│                                                                  │
│  IMPLEMENTATION TIPS:                                           │
│  • Always check on server (never trust client)                 │
│  • Default deny, explicit allow                                │
│  • Centralize authorization logic                              │
│  • Log all access decisions for audit                          │
│  • Regular permission reviews                                  │
│                                                                  │
│  COMMON PATTERNS:                                               │
│  • Ownership: user.id === resource.ownerId                     │
│  • Hierarchy: admin > editor > viewer                          │
│  • Scoping: only see tenant's data                             │
│  • Delegation: share with specific users                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```


