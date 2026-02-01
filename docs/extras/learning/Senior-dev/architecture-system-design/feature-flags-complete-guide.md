# 🚩 Feature Flags Complete Guide

> A comprehensive guide to Feature Flags - LaunchDarkly, gradual rollouts, A/B testing, kill switches, and safely deploying features to production.

---

## 🧠 MUST REMEMBER TO IMPRESS (Memorize This!)

### 1-Liner Definition
> "Feature flags are conditional statements that enable or disable functionality at runtime without deploying new code - allowing gradual rollouts, A/B testing, and instant rollbacks via a kill switch."

### The 4 Types of Feature Flags (Memorize!)
```
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. RELEASE FLAGS (Short-lived)                                 │
│     └── Deploy code, enable feature gradually                   │
│     └── Remove after 100% rollout                              │
│     └── Example: "new-checkout-flow"                           │
│                                                                  │
│  2. EXPERIMENT FLAGS (Short-lived)                              │
│     └── A/B testing, measure impact                            │
│     └── Remove after experiment concludes                      │
│     └── Example: "pricing-page-variant-b"                      │
│                                                                  │
│  3. OPERATIONAL FLAGS (Long-lived)                              │
│     └── Kill switches, circuit breakers                        │
│     └── Keep indefinitely for safety                           │
│     └── Example: "disable-external-api"                        │
│                                                                  │
│  4. PERMISSION FLAGS (Long-lived)                               │
│     └── Premium features, entitlements                         │
│     └── Based on user tier/plan                                │
│     └── Example: "enable-advanced-analytics"                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Terms to Drop (Sound Smart!)
| Term | Use It Like This |
|------|------------------|
| **"Trunk-based development"** | "Feature flags enable trunk-based development - merge to main daily" |
| **"Progressive delivery"** | "We do progressive delivery - 5% → 25% → 100% rollout" |
| **"Kill switch"** | "If metrics drop, we hit the kill switch - instant rollback" |
| **"Targeting rules"** | "Flag targets internal users first, then beta, then everyone" |
| **"Stale flags"** | "We audit and remove stale flags monthly to reduce tech debt" |
| **"Flag debt"** | "Too many flags create flag debt - harder to reason about code" |
| **"Canary release"** | "Canary release with 1% traffic to detect issues early" |

### The Flag Lifecycle
```
FLAG LIFECYCLE:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. CREATE                                                      │
│     └── Define flag with clear naming convention               │
│     └── Document purpose, owner, expected removal date         │
│                                                                  │
│  2. DEVELOP                                                     │
│     └── Wrap new code in flag check                           │
│     └── Both paths (on/off) should work                       │
│                                                                  │
│  3. TEST                                                        │
│     └── Test with flag ON and OFF                             │
│     └── Verify metrics, monitoring in place                   │
│                                                                  │
│  4. ROLLOUT                                                     │
│     └── Internal → Beta → Percentage → 100%                   │
│     └── Monitor metrics at each stage                         │
│                                                                  │
│  5. CLEANUP                                                     │
│     └── Remove flag code after stable rollout                 │
│     └── Delete flag from system                               │
│     └── THIS STEP IS CRITICAL - most teams skip it!          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Numbers to Remember
| Metric | Value | Why |
|--------|-------|-----|
| Max active flags | **~50-100** | More = cognitive overload |
| Flag evaluation | **<10ms** | Must be fast (cached locally) |
| Stale flag threshold | **30 days** | Review flags older than this |
| Initial rollout | **1-5%** | Start small, catch issues |
| Rollout stages | **1% → 10% → 50% → 100%** | Gradual increase |

### The "Wow" Statement (Memorize This!)
> "Feature flags transformed how we deploy. We merge to main multiple times daily - code ships but features are dark until ready. For our last major launch, we rolled out to 1% of users, monitored error rates and latency, then gradually increased to 100% over a week. When we saw a 15% increase in checkout abandonment, we killed the feature in 30 seconds without a deploy. The flag also powered our A/B test - variant B improved conversion by 8%. After full rollout, we cleaned up the flag code to avoid flag debt. Our rule: every flag has an owner and expiration date."

### Quick Architecture Drawing (Draw This!)
```
┌─────────────────────────────────────────────────────────────────┐
│                   FEATURE FLAG ARCHITECTURE                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   FLAG MANAGEMENT (Control Plane)                               │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │  LaunchDarkly / Unleash / Custom Service               │  │
│   │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐   │  │
│   │  │  Flag   │  │Targeting│  │  Audit  │  │ Metrics │   │  │
│   │  │  Store  │  │  Rules  │  │   Log   │  │Dashboard│   │  │
│   │  └─────────┘  └─────────┘  └─────────┘  └─────────┘   │  │
│   └─────────────────────────────────────────────────────────┘  │
│          │ SDK pulls config (streaming/polling)                 │
│          ▼                                                       │
│   APPLICATION (Data Plane)                                      │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │                                                          │  │
│   │  ┌──────────────────────────────────────────────────┐   │  │
│   │  │             Flag SDK (Local Cache)                │   │  │
│   │  │  ┌────────────────────────────────────────────┐  │   │  │
│   │  │  │ new-checkout: true for user-123            │  │   │  │
│   │  │  │ dark-mode: 50% rollout                     │  │   │  │
│   │  │  │ premium-feature: plan === 'pro'            │  │   │  │
│   │  │  └────────────────────────────────────────────┘  │   │  │
│   │  └──────────────────────────────────────────────────┘   │  │
│   │                          │                               │  │
│   │                          ▼                               │  │
│   │  ┌──────────────────────────────────────────────────┐   │  │
│   │  │ if (flags.isEnabled('new-checkout', user)) {     │   │  │
│   │  │   return <NewCheckout />;                        │   │  │
│   │  │ }                                                │   │  │
│   │  │ return <OldCheckout />;                          │   │  │
│   │  └──────────────────────────────────────────────────┘   │  │
│   │                                                          │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Interview Rapid Fire (Practice These!)

**Q: "What are feature flags?"**
> "Runtime conditionals that enable/disable features without deploying. Powers gradual rollouts, A/B tests, and kill switches. Decouples deployment from release."

**Q: "Why not just use if statements?"**
> "Feature flags are externally controlled, can change without deploy, support targeting rules (user segments), provide audit logs, and integrate with analytics."

**Q: "What's progressive delivery?"**
> "Rolling out features gradually - 1% → 10% → 50% → 100%. Monitor metrics at each stage. If issues arise, stop or rollback instantly."

**Q: "What's flag debt?"**
> "Accumulation of stale flags that should have been removed. Makes code harder to understand, increases complexity. Clean up flags after full rollout."

**Q: "Kill switch vs circuit breaker?"**
> "Kill switch: manually disable a feature (human decision). Circuit breaker: automatically disable based on error rates (automated). Both are operational flags."

---

## 🎯 How to Explain Like a Senior Developer

### When Asked: "How do you implement feature flags?"

**Junior Answer:**
> "Use if statements with a config file."

**Senior Answer:**
> "Feature flags involve several components:

**1. Flag Storage & Management**
- Dedicated service or SaaS (LaunchDarkly, Unleash)
- Defines flag state, targeting rules, rollout percentage
- UI for non-engineers to toggle flags

**2. SDK Integration**
- Client library caches flags locally (performance)
- Evaluates flags with user context (targeting)
- Streams updates for real-time changes

**3. Targeting Rules**
- Target by user attributes (plan, country, beta tester)
- Percentage rollouts (sticky - same user, same result)
- Override for specific users (internal testing)

**4. Flag Types & Lifecycle**
- Release flags: short-lived, remove after rollout
- Operational flags: long-lived kill switches
- Clean up flags to avoid technical debt

**5. Observability**
- Track flag evaluations in analytics
- Alert on flag changes
- Audit log for compliance

Trade-offs:
- Added complexity to codebase
- Risk of stale flags accumulating
- Dependency on flag service availability
- Testing matrix increases (flag on/off combinations)"

### Follow-up Questions to Expect:

| They Ask | You Answer |
|----------|------------|
| "What if flag service is down?" | "SDK caches flags locally. Falls back to cached values or default. Design for flag service unavailability." |
| "How do you test?" | "Test both flag states. Integration tests with flags on/off. Feature tests simulate different user segments." |
| "What about consistency?" | "Use sticky bucketing - same user always gets same variant. Hash user ID + flag name for deterministic assignment." |
| "Build vs buy?" | "Build for simple on/off. Buy (LaunchDarkly) for targeting, A/B testing, audit, multi-environment. Team size and needs matter." |

---

## 📚 Table of Contents

1. [Core Concepts](#1-core-concepts)
2. [Implementation Patterns](#2-implementation-patterns)
3. [Gradual Rollouts](#3-gradual-rollouts)
4. [A/B Testing](#4-ab-testing)
5. [Kill Switches](#5-kill-switches)
6. [Tools & Platforms](#6-tools--platforms)
7. [When to Use / Not Use](#7-when-to-use--not-use)
8. [Interview Questions](#8-interview-questions)

---

## 1. Core Concepts

### Flag Evaluation Flow

```
FLAG EVALUATION:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  INPUT: Flag Key + User Context                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ flagKey: "new-dashboard"                                │   │
│  │ user: { id: "123", email: "john@acme.com",              │   │
│  │         plan: "pro", country: "US", beta: true }        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                          │                                       │
│                          ▼                                       │
│  EVALUATION RULES (checked in order)                            │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ 1. User Override: user.id === "123" → true              │   │
│  │ 2. Kill Switch: flag.killed → false                     │   │
│  │ 3. Targeting: user.beta === true → true                 │   │
│  │ 4. Percentage: hash(user.id + flag) % 100 < 25 → true  │   │
│  │ 5. Default: → false                                     │   │
│  └─────────────────────────────────────────────────────────┘   │
│                          │                                       │
│                          ▼                                       │
│  OUTPUT: Boolean (or Variant)                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ result: true                                            │   │
│  │ reason: "targeting_match"                               │   │
│  │ variant: "treatment"                                    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Sticky Bucketing

```typescript
// ════════════════════════════════════════════════════════════════
// STICKY BUCKETING: Consistent user experience
// ════════════════════════════════════════════════════════════════

// Problem: Random 50% rollout could give different result each request
// Solution: Hash user ID + flag name for deterministic assignment

import { createHash } from 'crypto';

function isUserInRollout(
  userId: string,
  flagName: string,
  percentage: number
): boolean {
  // Create deterministic hash
  const hash = createHash('md5')
    .update(`${userId}:${flagName}`)
    .digest('hex');
  
  // Convert first 8 chars to number (0-4294967295)
  const hashValue = parseInt(hash.substring(0, 8), 16);
  
  // Normalize to 0-100
  const bucket = (hashValue / 0xFFFFFFFF) * 100;
  
  // User is in rollout if their bucket is below percentage
  return bucket < percentage;
}

// Same user + flag always returns same result
isUserInRollout('user-123', 'new-feature', 50);  // Always true or always false
isUserInRollout('user-123', 'new-feature', 50);  // Same result!

// Different flag = different assignment
isUserInRollout('user-123', 'other-feature', 50);  // Could be different
```

### Flag Naming Conventions

```
FLAG NAMING CONVENTIONS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  FORMAT: {type}-{feature}-{description}                         │
│                                                                  │
│  RELEASE FLAGS:                                                 │
│  └── release-checkout-redesign                                 │
│  └── release-dashboard-v2                                      │
│  └── release-search-autocomplete                               │
│                                                                  │
│  EXPERIMENT FLAGS:                                              │
│  └── exp-pricing-annual-discount                               │
│  └── exp-onboarding-video-tutorial                             │
│  └── exp-cta-button-color                                      │
│                                                                  │
│  OPERATIONAL FLAGS:                                             │
│  └── ops-kill-external-api                                     │
│  └── ops-maintenance-mode                                      │
│  └── ops-disable-notifications                                 │
│                                                                  │
│  PERMISSION FLAGS:                                              │
│  └── perm-advanced-analytics                                   │
│  └── perm-api-access                                           │
│  └── perm-white-label                                          │
│                                                                  │
│  ANTI-PATTERNS:                                                 │
│  ✗ flag1, test, new_feature (unclear)                         │
│  ✗ johns-experiment (personal names)                           │
│  ✗ TEMP_FLAG_DELETE_LATER (never gets deleted)                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Implementation Patterns

### Basic Feature Flag Service

```typescript
// ════════════════════════════════════════════════════════════════
// SIMPLE FEATURE FLAG SERVICE (Build Your Own)
// ════════════════════════════════════════════════════════════════

interface Flag {
  key: string;
  enabled: boolean;
  description: string;
  percentage?: number;      // Rollout percentage (0-100)
  targetUsers?: string[];   // Specific user IDs
  targetRules?: TargetRule[];
  createdAt: Date;
  updatedAt: Date;
  owner: string;
  expiresAt?: Date;
}

interface TargetRule {
  attribute: string;        // e.g., "plan", "country"
  operator: 'equals' | 'contains' | 'in' | 'gt' | 'lt';
  value: any;
  enabled: boolean;
}

interface UserContext {
  id: string;
  email?: string;
  plan?: string;
  country?: string;
  beta?: boolean;
  [key: string]: any;
}

class FeatureFlagService {
  private flags: Map<string, Flag> = new Map();
  private cache: Map<string, boolean> = new Map();
  
  async loadFlags(): Promise<void> {
    // Load from database/config
    const flagsFromDB = await db.flags.findMany();
    flagsFromDB.forEach(flag => this.flags.set(flag.key, flag));
  }
  
  isEnabled(flagKey: string, user?: UserContext): boolean {
    const flag = this.flags.get(flagKey);
    
    // Flag doesn't exist = disabled
    if (!flag) return false;
    
    // Flag globally disabled
    if (!flag.enabled) return false;
    
    // Check user-specific overrides
    if (user && flag.targetUsers?.includes(user.id)) {
      return true;
    }
    
    // Check targeting rules
    if (user && flag.targetRules) {
      for (const rule of flag.targetRules) {
        if (this.evaluateRule(rule, user)) {
          return rule.enabled;
        }
      }
    }
    
    // Check percentage rollout
    if (flag.percentage !== undefined && user) {
      return this.isInPercentage(user.id, flagKey, flag.percentage);
    }
    
    // Default to flag's enabled state
    return flag.enabled;
  }
  
  private evaluateRule(rule: TargetRule, user: UserContext): boolean {
    const userValue = user[rule.attribute];
    
    switch (rule.operator) {
      case 'equals':
        return userValue === rule.value;
      case 'contains':
        return String(userValue).includes(rule.value);
      case 'in':
        return Array.isArray(rule.value) && rule.value.includes(userValue);
      case 'gt':
        return userValue > rule.value;
      case 'lt':
        return userValue < rule.value;
      default:
        return false;
    }
  }
  
  private isInPercentage(userId: string, flagKey: string, percentage: number): boolean {
    const hash = this.hashString(`${userId}:${flagKey}`);
    const bucket = (hash % 100);
    return bucket < percentage;
  }
  
  private hashString(str: string): number {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      const char = str.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash;
    }
    return Math.abs(hash);
  }
}

// Usage
const flags = new FeatureFlagService();
await flags.loadFlags();

if (flags.isEnabled('new-checkout', { id: 'user-123', plan: 'pro' })) {
  renderNewCheckout();
} else {
  renderOldCheckout();
}
```

### React Integration

```tsx
// ════════════════════════════════════════════════════════════════
// REACT: Feature Flag Provider & Hooks
// ════════════════════════════════════════════════════════════════

import React, { createContext, useContext, useState, useEffect } from 'react';

interface FeatureFlagsContextType {
  flags: Record<string, boolean>;
  isEnabled: (flagKey: string) => boolean;
  loading: boolean;
}

const FeatureFlagsContext = createContext<FeatureFlagsContextType | null>(null);

// Provider
export function FeatureFlagsProvider({ 
  children,
  user,
}: { 
  children: React.ReactNode;
  user: UserContext;
}) {
  const [flags, setFlags] = useState<Record<string, boolean>>({});
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    async function loadFlags() {
      try {
        // Fetch evaluated flags for this user
        const response = await fetch('/api/feature-flags', {
          method: 'POST',
          body: JSON.stringify({ userId: user.id, context: user }),
        });
        const evaluatedFlags = await response.json();
        setFlags(evaluatedFlags);
      } catch (error) {
        console.error('Failed to load feature flags', error);
        // Use defaults on error
        setFlags({});
      } finally {
        setLoading(false);
      }
    }
    
    loadFlags();
  }, [user.id]);
  
  const isEnabled = (flagKey: string): boolean => {
    return flags[flagKey] ?? false;
  };
  
  return (
    <FeatureFlagsContext.Provider value={{ flags, isEnabled, loading }}>
      {children}
    </FeatureFlagsContext.Provider>
  );
}

// Hook
export function useFeatureFlag(flagKey: string): boolean {
  const context = useContext(FeatureFlagsContext);
  if (!context) throw new Error('useFeatureFlag must be used within FeatureFlagsProvider');
  return context.isEnabled(flagKey);
}

// Hook with loading state
export function useFeatureFlagWithLoading(flagKey: string): {
  enabled: boolean;
  loading: boolean;
} {
  const context = useContext(FeatureFlagsContext);
  if (!context) throw new Error('Must be used within FeatureFlagsProvider');
  return {
    enabled: context.isEnabled(flagKey),
    loading: context.loading,
  };
}

// ════════════════════════════════════════════════════════════════
// COMPONENT USAGE
// ════════════════════════════════════════════════════════════════

function CheckoutPage() {
  const showNewCheckout = useFeatureFlag('release-checkout-redesign');
  
  if (showNewCheckout) {
    return <NewCheckoutFlow />;
  }
  
  return <LegacyCheckoutFlow />;
}

// Conditional rendering component
function FeatureFlag({ 
  flag,
  children,
  fallback = null,
}: {
  flag: string;
  children: React.ReactNode;
  fallback?: React.ReactNode;
}) {
  const isEnabled = useFeatureFlag(flag);
  return isEnabled ? <>{children}</> : <>{fallback}</>;
}

// Usage
function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      
      <FeatureFlag flag="release-new-analytics" fallback={<OldAnalytics />}>
        <NewAnalytics />
      </FeatureFlag>
      
      <FeatureFlag flag="perm-advanced-reports">
        <AdvancedReportsSection />
      </FeatureFlag>
    </div>
  );
}
```

### Server-Side Implementation

```typescript
// ════════════════════════════════════════════════════════════════
// NEXT.JS: Server-Side Feature Flags
// ════════════════════════════════════════════════════════════════

// lib/feature-flags.ts
import { cookies } from 'next/headers';
import { cache } from 'react';

// Cache flag evaluation per request
export const getFlags = cache(async (userId: string): Promise<Record<string, boolean>> => {
  const response = await fetch(`${process.env.FLAG_SERVICE_URL}/evaluate`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ userId }),
    next: { revalidate: 60 }, // Cache for 60 seconds
  });
  
  return response.json();
});

export async function isEnabled(flagKey: string, userId: string): Promise<boolean> {
  const flags = await getFlags(userId);
  return flags[flagKey] ?? false;
}

// ════════════════════════════════════════════════════════════════
// SERVER COMPONENT USAGE
// ════════════════════════════════════════════════════════════════

// app/dashboard/page.tsx
import { isEnabled } from '@/lib/feature-flags';
import { getCurrentUser } from '@/lib/auth';

export default async function DashboardPage() {
  const user = await getCurrentUser();
  const showNewDashboard = await isEnabled('release-dashboard-v2', user.id);
  
  if (showNewDashboard) {
    return <NewDashboard />;
  }
  
  return <LegacyDashboard />;
}

// ════════════════════════════════════════════════════════════════
// API ROUTE WITH FLAGS
// ════════════════════════════════════════════════════════════════

// app/api/checkout/route.ts
import { NextResponse } from 'next/server';
import { isEnabled } from '@/lib/feature-flags';

export async function POST(request: Request) {
  const { userId, cart } = await request.json();
  
  // Different logic based on flag
  if (await isEnabled('release-new-payment-flow', userId)) {
    return handleNewPaymentFlow(cart);
  }
  
  return handleLegacyPaymentFlow(cart);
}
```

### Backend Service Integration

```typescript
// ════════════════════════════════════════════════════════════════
// EXPRESS MIDDLEWARE
// ════════════════════════════════════════════════════════════════

import express from 'express';

// Middleware to evaluate flags for request
const featureFlagsMiddleware = async (
  req: express.Request,
  res: express.Response,
  next: express.NextFunction
) => {
  const userId = req.user?.id || req.ip;  // Fallback to IP for anonymous
  
  // Evaluate all flags for this user
  const flags = await flagService.evaluateAll(userId, {
    plan: req.user?.plan,
    country: req.headers['cf-ipcountry'],
    userAgent: req.headers['user-agent'],
  });
  
  // Attach to request
  req.flags = flags;
  
  // Add helper function
  req.isFeatureEnabled = (flagKey: string) => flags[flagKey] ?? false;
  
  next();
};

app.use(featureFlagsMiddleware);

// Usage in route
app.get('/api/products', async (req, res) => {
  if (req.isFeatureEnabled('release-new-search')) {
    return newSearchProducts(req, res);
  }
  return legacySearchProducts(req, res);
});

// ════════════════════════════════════════════════════════════════
// DECORATOR PATTERN (NestJS style)
// ════════════════════════════════════════════════════════════════

function FeatureGuard(flagKey: string) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value;
    
    descriptor.value = async function (...args: any[]) {
      const req = args[0];
      
      if (!req.isFeatureEnabled(flagKey)) {
        throw new NotFoundError(`Feature ${flagKey} is not available`);
      }
      
      return originalMethod.apply(this, args);
    };
    
    return descriptor;
  };
}

// Usage
class ProductsController {
  @FeatureGuard('release-bulk-operations')
  async bulkUpdate(req: Request, res: Response) {
    // Only accessible if flag is enabled
  }
}
```

---

## 3. Gradual Rollouts

### Progressive Delivery Strategy

```
GRADUAL ROLLOUT STAGES:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  STAGE 1: INTERNAL (Day 1)                                      │
│  └── Target: employees only (email domain match)               │
│  └── Goal: Find obvious bugs                                   │
│  └── Duration: 1-2 days                                        │
│  └── Success: No critical errors                               │
│                                                                  │
│  STAGE 2: BETA USERS (Day 3)                                   │
│  └── Target: users.beta === true                               │
│  └── Goal: Get feedback from engaged users                     │
│  └── Duration: 3-5 days                                        │
│  └── Success: Positive feedback, no regressions               │
│                                                                  │
│  STAGE 3: CANARY (Day 7)                                       │
│  └── Target: 1-5% of all users (random)                       │
│  └── Goal: Validate at scale                                   │
│  └── Duration: 2-3 days                                        │
│  └── Success: Error rate stable, metrics normal               │
│                                                                  │
│  STAGE 4: GRADUAL ROLLOUT (Day 10+)                           │
│  └── Target: 10% → 25% → 50% → 100%                           │
│  └── Goal: Full rollout with monitoring                       │
│  └── Duration: 3-7 days                                        │
│  └── Success: Full rollout, no rollbacks needed               │
│                                                                  │
│  STAGE 5: CLEANUP (Day 20+)                                    │
│  └── Remove flag code                                          │
│  └── Delete flag from system                                   │
│  └── Update documentation                                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Rollout Configuration

```typescript
// ════════════════════════════════════════════════════════════════
// ROLLOUT STAGES CONFIGURATION
// ════════════════════════════════════════════════════════════════

interface RolloutStage {
  name: string;
  percentage?: number;
  rules?: TargetRule[];
  duration: number;  // Hours before auto-advance
  successCriteria: SuccessCriteria;
}

interface SuccessCriteria {
  maxErrorRate: number;       // e.g., 0.01 (1%)
  maxLatencyP99: number;      // e.g., 500ms
  minSampleSize: number;      // e.g., 1000 requests
}

const rolloutConfig: RolloutStage[] = [
  {
    name: 'internal',
    rules: [{ attribute: 'email', operator: 'contains', value: '@mycompany.com' }],
    duration: 48,
    successCriteria: { maxErrorRate: 0.05, maxLatencyP99: 1000, minSampleSize: 100 },
  },
  {
    name: 'beta',
    rules: [{ attribute: 'beta', operator: 'equals', value: true }],
    duration: 72,
    successCriteria: { maxErrorRate: 0.02, maxLatencyP99: 800, minSampleSize: 500 },
  },
  {
    name: 'canary',
    percentage: 5,
    duration: 48,
    successCriteria: { maxErrorRate: 0.01, maxLatencyP99: 500, minSampleSize: 1000 },
  },
  {
    name: 'rollout-25',
    percentage: 25,
    duration: 24,
    successCriteria: { maxErrorRate: 0.01, maxLatencyP99: 500, minSampleSize: 5000 },
  },
  {
    name: 'rollout-50',
    percentage: 50,
    duration: 24,
    successCriteria: { maxErrorRate: 0.01, maxLatencyP99: 500, minSampleSize: 10000 },
  },
  {
    name: 'full-rollout',
    percentage: 100,
    duration: 0,  // Stay here
    successCriteria: { maxErrorRate: 0.01, maxLatencyP99: 500, minSampleSize: 50000 },
  },
];

// ════════════════════════════════════════════════════════════════
// AUTOMATED ROLLOUT MANAGER
// ════════════════════════════════════════════════════════════════

class AutomatedRolloutManager {
  private currentStage: number = 0;
  
  async checkAndAdvance(flagKey: string): Promise<void> {
    const stage = rolloutConfig[this.currentStage];
    const metrics = await this.getMetrics(flagKey, stage.duration);
    
    // Check success criteria
    if (this.meetsCriteria(metrics, stage.successCriteria)) {
      // Advance to next stage
      if (this.currentStage < rolloutConfig.length - 1) {
        this.currentStage++;
        await this.updateFlag(flagKey, rolloutConfig[this.currentStage]);
        await this.notifyTeam(`${flagKey} advanced to ${rolloutConfig[this.currentStage].name}`);
      }
    } else {
      // Rollback on failure
      await this.rollback(flagKey, metrics);
    }
  }
  
  private meetsCriteria(metrics: Metrics, criteria: SuccessCriteria): boolean {
    return (
      metrics.sampleSize >= criteria.minSampleSize &&
      metrics.errorRate <= criteria.maxErrorRate &&
      metrics.latencyP99 <= criteria.maxLatencyP99
    );
  }
  
  private async rollback(flagKey: string, metrics: Metrics): Promise<void> {
    // Disable flag entirely
    await flagService.update(flagKey, { enabled: false });
    
    // Alert team
    await this.alertTeam({
      severity: 'high',
      message: `Rollout ${flagKey} failed criteria`,
      metrics,
    });
  }
}
```

---

## 4. A/B Testing

### Experiment Configuration

```typescript
// ════════════════════════════════════════════════════════════════
// A/B TEST EXPERIMENT
// ════════════════════════════════════════════════════════════════

interface Experiment {
  key: string;
  name: string;
  hypothesis: string;
  primaryMetric: string;       // What we're measuring
  secondaryMetrics: string[];
  variants: Variant[];
  trafficPercentage: number;   // % of users in experiment
  startDate: Date;
  endDate: Date;
  minimumSampleSize: number;
  statisticalSignificance: number;  // e.g., 0.95 (95%)
}

interface Variant {
  key: string;
  name: string;
  weight: number;  // Distribution weight (e.g., 50/50)
  config?: Record<string, any>;  // Variant-specific config
}

const pricingExperiment: Experiment = {
  key: 'exp-pricing-annual-discount',
  name: 'Annual Pricing Discount Test',
  hypothesis: 'Showing 20% annual discount will increase conversion by 10%',
  primaryMetric: 'conversion_rate',
  secondaryMetrics: ['revenue_per_user', 'plan_upgrades'],
  variants: [
    { key: 'control', name: 'Current Pricing (10% discount)', weight: 50 },
    { key: 'treatment', name: 'New Pricing (20% discount)', weight: 50 },
  ],
  trafficPercentage: 100,  // 100% of eligible users
  startDate: new Date('2024-01-15'),
  endDate: new Date('2024-02-15'),
  minimumSampleSize: 10000,
  statisticalSignificance: 0.95,
};

// ════════════════════════════════════════════════════════════════
// VARIANT ASSIGNMENT
// ════════════════════════════════════════════════════════════════

function assignVariant(experiment: Experiment, userId: string): Variant | null {
  // Check if user is in experiment traffic
  const inExperiment = isUserInRollout(
    userId,
    `${experiment.key}:traffic`,
    experiment.trafficPercentage
  );
  
  if (!inExperiment) return null;
  
  // Assign variant based on weights
  const hash = hashString(`${userId}:${experiment.key}:variant`);
  const bucket = hash % 100;
  
  let cumulativeWeight = 0;
  for (const variant of experiment.variants) {
    cumulativeWeight += variant.weight;
    if (bucket < cumulativeWeight) {
      return variant;
    }
  }
  
  return experiment.variants[0];  // Fallback to first variant
}

// ════════════════════════════════════════════════════════════════
// TRACKING & ANALYTICS
// ════════════════════════════════════════════════════════════════

class ExperimentTracker {
  trackExposure(experiment: Experiment, variant: Variant, userId: string): void {
    // Track that user saw this variant
    analytics.track('experiment_exposure', {
      experimentKey: experiment.key,
      variantKey: variant.key,
      userId,
      timestamp: new Date(),
    });
  }
  
  trackConversion(experimentKey: string, metric: string, value: number, userId: string): void {
    // Track conversion event
    analytics.track('experiment_conversion', {
      experimentKey,
      metric,
      value,
      userId,
      timestamp: new Date(),
    });
  }
}
```

### React A/B Testing Component

```tsx
// ════════════════════════════════════════════════════════════════
// REACT A/B TEST COMPONENT
// ════════════════════════════════════════════════════════════════

import { useEffect } from 'react';

interface ExperimentResult {
  variant: string;
  config: Record<string, any>;
}

function useExperiment(experimentKey: string): ExperimentResult | null {
  const { user } = useUser();
  const [result, setResult] = useState<ExperimentResult | null>(null);
  
  useEffect(() => {
    async function getVariant() {
      const response = await fetch(`/api/experiments/${experimentKey}/assign`, {
        method: 'POST',
        body: JSON.stringify({ userId: user.id }),
      });
      
      if (response.ok) {
        const variant = await response.json();
        setResult(variant);
        
        // Track exposure
        analytics.track('experiment_exposure', {
          experimentKey,
          variant: variant.key,
        });
      }
    }
    
    getVariant();
  }, [experimentKey, user.id]);
  
  return result;
}

// Usage
function PricingPage() {
  const experiment = useExperiment('exp-pricing-annual-discount');
  
  if (!experiment) {
    // Not in experiment, show default
    return <DefaultPricing />;
  }
  
  // Track conversion when user subscribes
  const handleSubscribe = (plan: string) => {
    analytics.track('experiment_conversion', {
      experimentKey: 'exp-pricing-annual-discount',
      metric: 'subscription',
      value: 1,
    });
    // ... proceed with subscription
  };
  
  if (experiment.variant === 'treatment') {
    return <NewPricing discount={20} onSubscribe={handleSubscribe} />;
  }
  
  return <DefaultPricing discount={10} onSubscribe={handleSubscribe} />;
}

// ════════════════════════════════════════════════════════════════
// EXPERIMENT WRAPPER COMPONENT
// ════════════════════════════════════════════════════════════════

function Experiment({
  experimentKey,
  variants,
  fallback,
}: {
  experimentKey: string;
  variants: Record<string, React.ReactNode>;
  fallback: React.ReactNode;
}) {
  const experiment = useExperiment(experimentKey);
  
  if (!experiment) {
    return <>{fallback}</>;
  }
  
  return <>{variants[experiment.variant] || fallback}</>;
}

// Usage
function OnboardingPage() {
  return (
    <Experiment
      experimentKey="exp-onboarding-flow"
      variants={{
        control: <ClassicOnboarding />,
        'video-tutorial': <VideoOnboarding />,
        'interactive': <InteractiveOnboarding />,
      }}
      fallback={<ClassicOnboarding />}
    />
  );
}
```

### Statistical Significance

```typescript
// ════════════════════════════════════════════════════════════════
// EXPERIMENT ANALYSIS
// ════════════════════════════════════════════════════════════════

interface ExperimentResults {
  control: { conversions: number; total: number };
  treatment: { conversions: number; total: number };
}

function analyzeExperiment(results: ExperimentResults): {
  controlRate: number;
  treatmentRate: number;
  lift: number;
  significant: boolean;
  pValue: number;
} {
  const controlRate = results.control.conversions / results.control.total;
  const treatmentRate = results.treatment.conversions / results.treatment.total;
  const lift = ((treatmentRate - controlRate) / controlRate) * 100;
  
  // Calculate statistical significance (z-test)
  const pooledRate = (results.control.conversions + results.treatment.conversions) /
                     (results.control.total + results.treatment.total);
  
  const standardError = Math.sqrt(
    pooledRate * (1 - pooledRate) * 
    (1/results.control.total + 1/results.treatment.total)
  );
  
  const zScore = (treatmentRate - controlRate) / standardError;
  const pValue = 2 * (1 - normalCDF(Math.abs(zScore)));
  
  return {
    controlRate,
    treatmentRate,
    lift,
    significant: pValue < 0.05,
    pValue,
  };
}

// Example results
const results = analyzeExperiment({
  control: { conversions: 450, total: 5000 },    // 9% conversion
  treatment: { conversions: 520, total: 5000 },  // 10.4% conversion
});

// { controlRate: 0.09, treatmentRate: 0.104, lift: 15.5%, significant: true, pValue: 0.023 }
```

---

## 5. Kill Switches

### Kill Switch Patterns

```
KILL SWITCH TYPES:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. FEATURE KILL SWITCH                                         │
│     └── Disable a specific feature                             │
│     └── "ops-kill-new-checkout"                                │
│     └── Use: Feature causing issues, needs instant disable     │
│                                                                  │
│  2. INTEGRATION KILL SWITCH                                     │
│     └── Disable external service integration                   │
│     └── "ops-kill-stripe-payments"                             │
│     └── Use: Third-party service is down/degraded              │
│                                                                  │
│  3. MAINTENANCE MODE                                            │
│     └── Put entire app in read-only mode                       │
│     └── "ops-maintenance-mode"                                 │
│     └── Use: Database migration, major update                  │
│                                                                  │
│  4. CIRCUIT BREAKER (Automatic)                                │
│     └── Auto-disable based on error rates                      │
│     └── "ops-circuit-payment-service"                          │
│     └── Use: Automated protection against cascading failures   │
│                                                                  │
│  5. LOAD SHEDDING                                               │
│     └── Disable non-critical features under load               │
│     └── "ops-shed-recommendations"                             │
│     └── Use: System under heavy load, prioritize core features │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Kill Switch Implementation

```typescript
// ════════════════════════════════════════════════════════════════
// OPERATIONAL FLAGS: Kill Switches
// ════════════════════════════════════════════════════════════════

// Define operational flags that should never be removed
const OPERATIONAL_FLAGS = {
  'ops-kill-checkout': {
    description: 'Disable checkout flow, show maintenance message',
    fallbackBehavior: 'show-maintenance',
  },
  'ops-kill-payments': {
    description: 'Disable payment processing',
    fallbackBehavior: 'queue-for-later',
  },
  'ops-kill-external-api': {
    description: 'Disable calls to external API',
    fallbackBehavior: 'use-cache',
  },
  'ops-maintenance-mode': {
    description: 'Put app in read-only mode',
    fallbackBehavior: 'read-only',
  },
  'ops-shed-recommendations': {
    description: 'Disable recommendation engine under load',
    fallbackBehavior: 'static-recommendations',
  },
};

// ════════════════════════════════════════════════════════════════
// KILL SWITCH WITH GRACEFUL DEGRADATION
// ════════════════════════════════════════════════════════════════

class KillSwitchService {
  async handlePayment(paymentData: PaymentData): Promise<PaymentResult> {
    // Check kill switch
    if (flags.isEnabled('ops-kill-payments')) {
      // Graceful degradation: queue for later processing
      await this.queuePaymentForLater(paymentData);
      
      return {
        status: 'queued',
        message: 'Payment will be processed shortly',
        retryAt: new Date(Date.now() + 30 * 60 * 1000), // 30 minutes
      };
    }
    
    // Normal flow
    return this.processPayment(paymentData);
  }
  
  async getRecommendations(userId: string): Promise<Product[]> {
    // Load shedding: return static recommendations under load
    if (flags.isEnabled('ops-shed-recommendations')) {
      return this.getStaticRecommendations();
    }
    
    // Check external API kill switch
    if (flags.isEnabled('ops-kill-external-api')) {
      return this.getCachedRecommendations(userId);
    }
    
    return this.fetchRecommendations(userId);
  }
}

// ════════════════════════════════════════════════════════════════
// AUTOMATIC CIRCUIT BREAKER
// ════════════════════════════════════════════════════════════════

class CircuitBreakerFlag {
  private errorCount = 0;
  private lastErrorTime = 0;
  private isOpen = false;
  
  constructor(
    private flagKey: string,
    private threshold: number = 5,        // Errors before opening
    private windowMs: number = 60000,     // 1 minute window
    private cooldownMs: number = 30000,   // 30 second cooldown
  ) {}
  
  async execute<T>(fn: () => Promise<T>): Promise<T> {
    // Check if circuit is open
    if (this.isOpen) {
      // Check if cooldown has passed
      if (Date.now() - this.lastErrorTime > this.cooldownMs) {
        this.isOpen = false;
        this.errorCount = 0;
      } else {
        throw new CircuitOpenError('Circuit breaker is open');
      }
    }
    
    try {
      const result = await fn();
      this.errorCount = 0;  // Reset on success
      return result;
    } catch (error) {
      this.recordError();
      throw error;
    }
  }
  
  private recordError(): void {
    const now = Date.now();
    
    // Reset if outside window
    if (now - this.lastErrorTime > this.windowMs) {
      this.errorCount = 0;
    }
    
    this.errorCount++;
    this.lastErrorTime = now;
    
    if (this.errorCount >= this.threshold) {
      this.isOpen = true;
      this.notifyCircuitOpen();
    }
  }
  
  private notifyCircuitOpen(): void {
    // Alert team
    alerting.send({
      severity: 'critical',
      message: `Circuit breaker opened for ${this.flagKey}`,
      errorCount: this.errorCount,
    });
    
    // Update flag (for visibility in dashboard)
    flagService.update(this.flagKey, { 
      enabled: false,
      metadata: { reason: 'circuit_breaker', openedAt: new Date() },
    });
  }
}

// Usage
const paymentCircuit = new CircuitBreakerFlag('ops-circuit-payments');

async function processPayment(data: PaymentData) {
  return paymentCircuit.execute(async () => {
    return stripeClient.createCharge(data);
  });
}
```

### Emergency Procedures

```typescript
// ════════════════════════════════════════════════════════════════
// EMERGENCY KILL SWITCH API
// ════════════════════════════════════════════════════════════════

// POST /api/admin/kill-switch
// Requires: admin role + 2FA confirmation

import { requireAdmin, require2FA } from '@/lib/auth';

app.post('/api/admin/kill-switch/:flagKey', 
  requireAdmin,
  require2FA,
  async (req, res) => {
    const { flagKey } = req.params;
    const { action, reason } = req.body;
    
    // Validate it's an operational flag
    if (!flagKey.startsWith('ops-')) {
      return res.status(400).json({ error: 'Can only kill operational flags' });
    }
    
    // Record who did it and why
    await auditLog.record({
      action: `kill_switch_${action}`,
      flagKey,
      reason,
      user: req.user.email,
      timestamp: new Date(),
    });
    
    // Update flag
    await flagService.update(flagKey, {
      enabled: action === 'enable',
    });
    
    // Notify team
    await slack.send({
      channel: '#incidents',
      message: `🚨 Kill switch ${action}d: ${flagKey}\nBy: ${req.user.email}\nReason: ${reason}`,
    });
    
    res.json({ success: true });
  }
);

// ════════════════════════════════════════════════════════════════
// RUNBOOK: Kill Switch Checklist
// ════════════════════════════════════════════════════════════════

/*
## Emergency Kill Switch Procedure

### Before Killing
1. [ ] Confirm the issue requires a kill switch
2. [ ] Identify which flag to toggle
3. [ ] Understand the fallback behavior
4. [ ] Notify on-call engineer

### Executing
1. [ ] Navigate to flag dashboard or use CLI
2. [ ] Toggle the kill switch
3. [ ] Verify change propagated (check SDK logs)
4. [ ] Monitor error rates drop

### After Killing
1. [ ] Post in #incidents channel
2. [ ] Create incident ticket
3. [ ] Investigate root cause
4. [ ] Plan fix and re-enable
5. [ ] Post-mortem if significant impact

### Re-enabling
1. [ ] Fix verified and deployed
2. [ ] Test in staging with flag enabled
3. [ ] Enable flag (gradual rollout if unsure)
4. [ ] Monitor metrics for 30 minutes
5. [ ] Close incident ticket
*/
```

### Flag Cleanup & Technical Debt

```typescript
// ════════════════════════════════════════════════════════════════
// STALE FLAG DETECTION
// ════════════════════════════════════════════════════════════════

interface FlagMetadata {
  key: string;
  createdAt: Date;
  lastEvaluatedAt: Date;
  owner: string;
  expiresAt?: Date;
  type: 'release' | 'experiment' | 'operational' | 'permission';
}

class FlagCleanupService {
  async findStaleFlags(thresholdDays: number = 30): Promise<FlagMetadata[]> {
    const allFlags = await flagService.getAllFlags();
    const staleDate = new Date(Date.now() - thresholdDays * 24 * 60 * 60 * 1000);
    
    return allFlags.filter(flag => {
      // Operational flags are never stale
      if (flag.type === 'operational') return false;
      
      // Permission flags are never stale
      if (flag.type === 'permission') return false;
      
      // Check if flag is fully rolled out (100%) and old
      if (flag.percentage === 100 && flag.createdAt < staleDate) {
        return true;
      }
      
      // Check if flag has expired
      if (flag.expiresAt && flag.expiresAt < new Date()) {
        return true;
      }
      
      // Check if flag hasn't been evaluated recently
      if (flag.lastEvaluatedAt < staleDate) {
        return true;
      }
      
      return false;
    });
  }
  
  async sendStaleReports(): Promise<void> {
    const staleFlags = await this.findStaleFlags();
    
    // Group by owner
    const byOwner = groupBy(staleFlags, 'owner');
    
    for (const [owner, flags] of Object.entries(byOwner)) {
      await email.send({
        to: owner,
        subject: `Action required: ${flags.length} stale feature flags`,
        body: `
          The following flags need to be cleaned up:
          
          ${flags.map(f => `- ${f.key} (created ${f.createdAt})`).join('\n')}
          
          Please remove the flag code and delete the flags, or extend their expiration.
        `,
      });
    }
  }
}

// Run weekly
cron.schedule('0 9 * * MON', async () => {
  await new FlagCleanupService().sendStaleReports();
});

// ════════════════════════════════════════════════════════════════
// ESLINT RULE: Detect Flag Code to Remove
// ════════════════════════════════════════════════════════════════

// .eslintrc.js (custom rule concept)
// When a flag is at 100% for 30 days, ESLint warns to remove the code

/*
// This would trigger a warning:
if (flags.isEnabled('release-old-feature-100-percent')) {
  // Remove this flag check - fully rolled out
}
*/
```

---

## 6. Tools & Platforms

### Platform Comparison

```
FEATURE FLAG PLATFORMS:
┌──────────────────────────────────────────────────────────────────────────┐
│                                                                           │
│  LAUNCHDARKLY (Market Leader)                                            │
│  ├── Pricing: $$$ (expensive, per seat + MAU)                           │
│  ├── Features: Best-in-class, A/B testing, targeting, audit             │
│  ├── SDKs: All languages, client + server                               │
│  ├── Best for: Enterprise, teams that need everything                   │
│  └── Learning: Easy, great docs                                         │
│                                                                           │
│  UNLEASH (Open Source)                                                   │
│  ├── Pricing: Free (self-hosted) or $ (cloud)                          │
│  ├── Features: Core features, strategies, constraints                   │
│  ├── SDKs: Most languages                                               │
│  ├── Best for: Cost-conscious, want to self-host                       │
│  └── Learning: Easy                                                     │
│                                                                           │
│  FLAGSMITH (Open Source)                                                 │
│  ├── Pricing: Free (self-hosted) or $ (cloud)                          │
│  ├── Features: Flags + remote config, segments                          │
│  ├── SDKs: Most languages                                               │
│  ├── Best for: Startups, simple needs                                  │
│  └── Learning: Easy                                                     │
│                                                                           │
│  POSTHOG (Product Analytics + Flags)                                    │
│  ├── Pricing: $ (generous free tier)                                   │
│  ├── Features: Feature flags + analytics + experiments                  │
│  ├── SDKs: Most languages                                               │
│  ├── Best for: Want analytics + flags together                         │
│  └── Learning: Easy                                                     │
│                                                                           │
│  SPLIT.IO                                                                │
│  ├── Pricing: $$ (per seat)                                            │
│  ├── Features: Feature delivery platform, experimentation              │
│  ├── SDKs: All languages                                                │
│  ├── Best for: Enterprise experimentation                              │
│  └── Learning: Medium                                                   │
│                                                                           │
│  BUILD YOUR OWN                                                          │
│  ├── Pricing: Dev time                                                  │
│  ├── Features: What you build                                           │
│  ├── Best for: Simple on/off flags, learning                           │
│  └── Warning: Harder than it looks                                     │
│                                                                           │
│  RECOMMENDATION:                                                         │
│  ├── Startup (budget): Unleash or Flagsmith (self-hosted)             │
│  ├── Startup (convenience): PostHog (flags + analytics)               │
│  ├── Growth: LaunchDarkly or Split.io                                  │
│  ├── Enterprise: LaunchDarkly                                          │
│  └── Learning: Build your own first, then migrate                      │
│                                                                           │
└──────────────────────────────────────────────────────────────────────────┘
```

### LaunchDarkly Integration

```typescript
// ════════════════════════════════════════════════════════════════
// LAUNCHDARKLY: Full Integration Example
// ════════════════════════════════════════════════════════════════

import * as LaunchDarkly from 'launchdarkly-node-server-sdk';

// Initialize client (do once at startup)
const ldClient = LaunchDarkly.init(process.env.LAUNCHDARKLY_SDK_KEY!);

// Wait for initialization
await ldClient.waitForInitialization();

// User context
const user: LaunchDarkly.LDUser = {
  key: 'user-123',
  email: 'john@example.com',
  custom: {
    plan: 'pro',
    company: 'Acme Inc',
    signupDate: '2023-01-15',
  },
};

// Boolean flag
const showNewFeature = await ldClient.variation('release-new-dashboard', user, false);

// Multivariate flag (string)
const checkoutVariant = await ldClient.variation('exp-checkout-flow', user, 'control');

// JSON flag (config)
const pricingConfig = await ldClient.variation('config-pricing-tiers', user, {
  basic: 9,
  pro: 29,
  enterprise: 99,
});

// ════════════════════════════════════════════════════════════════
// LAUNCHDARKLY: React SDK
// ════════════════════════════════════════════════════════════════

// Client-side (React)
import { withLDProvider, useFlags, useLDClient } from 'launchdarkly-react-client-sdk';

// Wrap app with provider
function App() {
  return (
    <LDProvider
      clientSideID={process.env.NEXT_PUBLIC_LD_CLIENT_ID!}
      user={{
        key: user.id,
        email: user.email,
        custom: { plan: user.plan },
      }}
    >
      <MyApp />
    </LDProvider>
  );
}

// Use flags in components
function Dashboard() {
  const flags = useFlags();
  const ldClient = useLDClient();
  
  // Track custom event
  const handleUpgrade = () => {
    ldClient?.track('upgrade-clicked', user);
    // ... handle upgrade
  };
  
  if (flags.releaseNewDashboard) {
    return <NewDashboard onUpgrade={handleUpgrade} />;
  }
  
  return <LegacyDashboard />;
}
```

### Unleash (Self-Hosted)

```typescript
// ════════════════════════════════════════════════════════════════
// UNLEASH: Open Source Feature Flags
// ════════════════════════════════════════════════════════════════

import { initialize, isEnabled } from 'unleash-client';

// Initialize
const unleash = initialize({
  url: 'http://unleash.mycompany.com/api/',
  appName: 'my-app',
  customHeaders: {
    Authorization: process.env.UNLEASH_API_TOKEN,
  },
});

// Wait for ready
unleash.on('ready', () => {
  console.log('Unleash is ready');
});

// Check flag
const enabled = isEnabled('new-feature', {
  userId: 'user-123',
  properties: {
    plan: 'pro',
    country: 'US',
  },
});

// ════════════════════════════════════════════════════════════════
// UNLEASH STRATEGIES (Targeting Rules)
// ════════════════════════════════════════════════════════════════

/*
Unleash built-in strategies:

1. Standard - Simple on/off
2. GradualRolloutUserId - Percentage rollout by user ID
3. GradualRolloutSessionId - Percentage by session
4. GradualRolloutRandom - Random percentage
5. UserWithId - Enable for specific user IDs
6. RemoteAddress - Enable for specific IPs
7. ApplicationHostname - Enable for specific hosts
8. FlexibleRollout - Percentage + constraints

Custom strategy example:
- PlanBasedStrategy: Enable for specific plans
*/

// Define custom strategy
class PlanBasedStrategy {
  name = 'planBased';
  
  isEnabled(parameters: any, context: any): boolean {
    const allowedPlans = parameters.plans?.split(',') || [];
    return allowedPlans.includes(context.properties?.plan);
  }
}

// Register custom strategy
unleash.registerStrategy(new PlanBasedStrategy());
```

### PostHog (Analytics + Flags)

```typescript
// ════════════════════════════════════════════════════════════════
// POSTHOG: Feature Flags + Analytics
// ════════════════════════════════════════════════════════════════

import PostHog from 'posthog-node';

const posthog = new PostHog(process.env.POSTHOG_API_KEY!, {
  host: 'https://app.posthog.com',
});

// Server-side flag check
const showNewFeature = await posthog.isFeatureEnabled(
  'new-checkout',
  'user-123',
  {
    personProperties: {
      plan: 'pro',
      country: 'US',
    },
  }
);

// Get all flags for user (efficient for multiple checks)
const flags = await posthog.getAllFlags('user-123');

// Track event (for analytics)
posthog.capture({
  distinctId: 'user-123',
  event: 'checkout_completed',
  properties: {
    total: 99.99,
    items: 3,
  },
});

// ════════════════════════════════════════════════════════════════
// POSTHOG: React Integration
// ════════════════════════════════════════════════════════════════

import { PostHogProvider, useFeatureFlagEnabled, usePostHog } from 'posthog-js/react';

function App() {
  return (
    <PostHogProvider
      apiKey={process.env.NEXT_PUBLIC_POSTHOG_KEY!}
      options={{ api_host: 'https://app.posthog.com' }}
    >
      <MyApp />
    </PostHogProvider>
  );
}

function PricingPage() {
  const showNewPricing = useFeatureFlagEnabled('new-pricing');
  const posthog = usePostHog();
  
  const handleSubscribe = (plan: string) => {
    // Track conversion
    posthog?.capture('subscribed', { plan });
  };
  
  return showNewPricing 
    ? <NewPricing onSubscribe={handleSubscribe} />
    : <OldPricing onSubscribe={handleSubscribe} />;
}
```

---

## 7. When to Use / Not Use

### When TO Use Feature Flags

```
✅ USE FEATURE FLAGS WHEN:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. PROGRESSIVE DELIVERY                                        │
│     └── Gradual rollout to catch issues early                  │
│     └── Canary releases (1% → 10% → 100%)                      │
│     └── Internal testing before public release                 │
│                                                                  │
│  2. A/B TESTING & EXPERIMENTATION                              │
│     └── Test different variants with users                     │
│     └── Measure impact on metrics                              │
│     └── Data-driven product decisions                          │
│                                                                  │
│  3. TRUNK-BASED DEVELOPMENT                                     │
│     └── Merge incomplete features to main                      │
│     └── Feature is dark until ready                            │
│     └── Avoid long-lived branches                              │
│                                                                  │
│  4. KILL SWITCHES & SAFETY                                     │
│     └── Instant rollback without deploy                        │
│     └── Disable integrations during outages                    │
│     └── Maintenance mode                                       │
│                                                                  │
│  5. ENTITLEMENTS & PERMISSIONS                                 │
│     └── Premium features for paid users                        │
│     └── Beta access for specific users                         │
│     └── Feature gating by plan                                 │
│                                                                  │
│  6. OPERATIONAL CONTROL                                        │
│     └── Load shedding under high traffic                       │
│     └── Disable expensive features during incidents            │
│     └── Regional feature availability                          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### When NOT to Use Feature Flags

```
❌ DON'T USE FEATURE FLAGS WHEN:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. SIMPLE CONFIGURATION                                        │
│     └── Static config that rarely changes                      │
│     └── Environment variables are simpler                      │
│     └── Don't need runtime changes                             │
│     → Use: env vars, config files                              │
│                                                                  │
│  2. SECURITY-CRITICAL PATHS                                    │
│     └── Auth checks shouldn't be flag-gated                    │
│     └── Permission logic must be deterministic                 │
│     └── Can't risk flag service being down                     │
│     → Use: code, not flags                                     │
│                                                                  │
│  3. EVERY SMALL CHANGE                                         │
│     └── Flag overhead not worth it for minor changes          │
│     └── Creates flag debt                                      │
│     └── Increases codebase complexity                          │
│     → Use: regular deploys, monitoring                         │
│                                                                  │
│  4. PERMANENT LOGIC BRANCHES                                   │
│     └── If both paths should exist permanently                │
│     └── It's not a flag, it's a feature                       │
│     → Use: regular code, configuration                         │
│                                                                  │
│  5. REPLACING PROPER TESTING                                   │
│     └── Flags don't substitute for QA                         │
│     └── Both paths need testing                                │
│     └── Bugs ship either way                                   │
│     → Use: proper testing + flags                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Common Pitfalls

```
⚠️ FEATURE FLAG PITFALLS:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  1. FLAG DEBT (Most Common!)                                   │
│     └── Never removing flags after rollout                     │
│     └── Code becomes unreadable spaghetti                      │
│     └── Fix: Owner + expiration date for every flag           │
│                                                                  │
│  2. TESTING COMBINATORIAL EXPLOSION                            │
│     └── 10 flags = 1024 combinations                          │
│     └── Can't test everything                                  │
│     └── Fix: Minimize flags, test critical combinations       │
│                                                                  │
│  3. FLAG SERVICE DEPENDENCY                                    │
│     └── App fails if flag service is down                     │
│     └── Fix: Cache locally, sensible defaults                 │
│                                                                  │
│  4. INCONSISTENT FLAG EVALUATION                               │
│     └── User sees different things on refresh                 │
│     └── Fix: Sticky bucketing (hash user ID)                  │
│                                                                  │
│  5. NO AUDIT TRAIL                                             │
│     └── Don't know who changed what when                      │
│     └── Fix: Audit logging, require comments                  │
│                                                                  │
│  6. EVERYONE CAN CHANGE FLAGS                                  │
│     └── Random changes cause incidents                        │
│     └── Fix: RBAC, approval workflow for production           │
│                                                                  │
│  7. DEAD CODE PATHS                                            │
│     └── Old path never gets removed                           │
│     └── Bugs accumulate in unused code                        │
│     └── Fix: Remove code when flag is at 100%                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 8. Interview Questions & Answers

### Basic Questions

**Q1: What are feature flags?**
> **A:** Conditional statements that enable/disable features at runtime without deploying code. Powers gradual rollouts, A/B testing, and kill switches. Decouples deployment from release - code ships, but feature stays dark until ready.

**Q2: What's the difference between feature flags and config?**
> **A:** Feature flags are dynamic (change at runtime), targeted (different values per user), and temporary (release flags should be removed). Config is static, affects all users, and permanent. Flags for releases, config for settings.

**Q3: What are the types of feature flags?**
> **A:** Four main types:
> - **Release flags**: Gradual rollout, short-lived, remove after 100%
> - **Experiment flags**: A/B testing, short-lived, remove after results
> - **Operational flags**: Kill switches, long-lived, kept for safety
> - **Permission flags**: Feature entitlements, long-lived, based on plan

**Q4: What is flag debt?**
> **A:** Accumulation of stale flags that should have been removed. Causes: unclear ownership, no expiration dates, no cleanup process. Makes code hard to understand. Fix: every flag has owner and expiration, monthly cleanup audits.

### Intermediate Questions

**Q5: How do you implement percentage rollouts?**
> **A:** Sticky bucketing - hash user ID + flag name to get deterministic 0-100 value. If hash < percentage, user is in rollout. Same user always gets same result (sticky), but different users get random distribution. Avoids inconsistent experience on refresh.

**Q6: How do you prevent flag service outages from breaking your app?**
> **A:** Multiple safeguards:
> - SDK caches flags locally (survives short outages)
> - Define sensible default values (fallback if no data)
> - Use streaming + polling (redundant update mechanisms)
> - Critical paths should work with defaults
> - Consider: operational flags default to safe mode

**Q7: How do you handle A/B testing with feature flags?**
> **A:**
> - Define variants (control, treatment) with weights (50/50)
> - Assign users deterministically (sticky bucketing)
> - Track exposures (who saw what)
> - Track conversions (did they complete goal?)
> - Calculate statistical significance before declaring winner
> - Clean up losing variant code

**Q8: What's progressive delivery?**
> **A:** Rolling out features gradually with monitoring at each stage:
> 1. Internal users (find obvious bugs)
> 2. Beta users (get feedback)
> 3. Canary (1-5% of all users)
> 4. Gradual increase (10% → 25% → 50% → 100%)
> 
> Monitor error rates, latency, business metrics. Stop if issues arise.

### Advanced Questions

**Q9: How do you test code with feature flags?**
> **A:**
> - Unit tests: test both flag on and off paths
> - Integration tests: test critical combinations
> - Can't test all combinations (2^n), prioritize:
>   - New features (flag on)
>   - Rollback scenarios (flag off)
>   - Flag interactions that might conflict
> - Use feature flag service to set test values

**Q10: Kill switch vs circuit breaker?**
> **A:**
> - **Kill switch**: Manual, human decides to disable. Use for "this feature is broken, turn it off." Reactive.
> - **Circuit breaker**: Automatic, trips on error threshold. Use for external dependencies. Proactive protection.
> 
> Both are operational flags. Kill switches need UI/API, circuit breakers need monitoring integration.

**Q11: How do you migrate from one flag platform to another?**
> **A:**
> 1. Abstract flag access behind interface
> 2. Implement interface for both platforms
> 3. Run both in parallel (shadow mode)
> 4. Compare results, fix discrepancies
> 5. Gradually migrate flags
> 6. Remove old platform
>
> Key: abstraction layer prevents vendor lock-in.

**Q12: Design a feature flag system for 10M users**
> **A:**
> - **Storage**: Redis or fast key-value store for flag config
> - **Evaluation**: Client-side SDK caches rules, evaluates locally
> - **Updates**: Streaming (SSE/WebSocket) for real-time changes
> - **Targeting**: Keep rules simple (complex rules = slow evaluation)
> - **Analytics**: Sample tracking events (can't store 10M * N flags)
> - **Reliability**: Cache locally, fallback defaults, no single point of failure

### Scenario Questions

**Q13: Your 50% rollout is causing 10% higher error rates. What do you do?**
> **A:**
> 1. **Immediate**: Roll back to 0% (kill switch)
> 2. **Verify**: Confirm errors drop after rollback
> 3. **Investigate**: Check logs for 500 errors, find root cause
> 4. **Fix**: Deploy fix
> 5. **Resume**: Start rollout again from 5%, monitor closely
> 6. **Post-mortem**: Why wasn't this caught in testing?

**Q14: A flag has been at 100% for 6 months. What's the risk?**
> **A:**
> - **Risks**: Flag debt, untested old code path, code complexity
> - **What to do**:
>   1. Check if flag code can be removed
>   2. Remove flag checks from code
>   3. Delete flag from platform
>   4. If uncertain, temporarily test with flag off to verify old path works
> - **Prevention**: Set expiration dates, monthly cleanup audits

---

## 🎓 Key Takeaways

1. **Feature flags decouple deployment from release** - ship code, enable feature separately
2. **4 types**: release (short), experiment (short), operational (long), permission (long)
3. **Sticky bucketing** ensures consistent user experience (hash user ID + flag)
4. **Progressive delivery**: internal → beta → canary → gradual → 100%
5. **Kill switches** are operational flags for instant rollback
6. **Flag debt is real** - clean up flags after full rollout
7. **Every flag needs owner and expiration date**
8. **Test both paths** - flag on AND flag off
9. **Cache locally** - survive flag service outages
10. **Build vs buy**: LaunchDarkly for enterprise, Unleash/PostHog for startups

---

## 📚 Resources

### Platforms
- [LaunchDarkly](https://launchdarkly.com/) - Market leader
- [Unleash](https://www.getunleash.io/) - Open source
- [PostHog](https://posthog.com/) - Analytics + flags
- [Flagsmith](https://flagsmith.com/) - Open source

### Documentation
- [Feature Flags Best Practices](https://launchdarkly.com/blog/best-practices-for-feature-flags/)
- [Martin Fowler: Feature Toggles](https://martinfowler.com/articles/feature-toggles.html)

### Books
- "Release It!" by Michael Nygard
- "Accelerate" by Nicole Forsgren (on deployment practices)


