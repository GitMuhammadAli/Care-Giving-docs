# Cross-Team Communication - Complete Guide

> **MUST REMEMBER**: Cross-team work fails due to communication, not technology. Key practices: document APIs and contracts, communicate changes early, define clear ownership, establish SLAs. Use RFC process for breaking changes. Async communication for most things, sync for urgent/complex. Over-communicate rather than assume. Dependencies are risks - manage them explicitly.

---

## How to Explain Like a Senior Developer

"Most cross-team failures aren't technical - they're communication failures. Team A changes an API, Team B's service breaks because no one told them. The fix: make dependencies explicit and communication systematic. Document contracts (API specs, schemas). Use RFCs for breaking changes - give teams time to react. Establish SLAs for response times. Default to async communication (Slack, docs) but use sync (meetings) for complex discussions or urgent issues. Over-communicate changes: if you think 'they probably know,' they probably don't. And treat every dependency as a risk to be managed, not just a technical integration."

---

## Communication Frameworks

### The RACI for Cross-Team Work

```typescript
// cross-team/raci.ts

/**
 * RACI Matrix for cross-team responsibilities
 * R = Responsible (does the work)
 * A = Accountable (final decision maker)
 * C = Consulted (gives input)
 * I = Informed (kept updated)
 */

interface RACIMatrix {
  activity: string;
  teams: Record<string, 'R' | 'A' | 'C' | 'I' | '-'>;
}

const exampleRaci: RACIMatrix[] = [
  {
    activity: 'Define API contract',
    teams: {
      'API Team': 'R',
      'Consumer Teams': 'C',
      'Platform Lead': 'A',
      'Security': 'C',
    },
  },
  {
    activity: 'Implement API changes',
    teams: {
      'API Team': 'R',
      'Consumer Teams': 'I',
      'Platform Lead': 'A',
      'QA': 'C',
    },
  },
  {
    activity: 'Migrate to new API version',
    teams: {
      'API Team': 'C',
      'Consumer Teams': 'R',
      'Platform Lead': 'I',
      'QA': 'R',
    },
  },
];

/**
 * When to create RACI
 */
const whenToRaci = [
  'Multi-team projects',
  'Ownership is unclear',
  'Previous handoff failures',
  'New team members joining',
  'Complex integrations',
];
```

### Communication Channels

```typescript
// cross-team/channels.ts

/**
 * Choosing the right communication channel
 */
const communicationChannels = {
  
  async: {
    slack: {
      when: [
        'Quick questions',
        'FYI updates',
        'Non-urgent discussions',
        'Sharing links/resources',
      ],
      expectations: 'Response within business day',
      tips: [
        'Use threads to keep organized',
        'Tag specific people, not @channel for everything',
        'Include context (links, screenshots)',
      ],
    },
    
    email: {
      when: [
        'External stakeholders',
        'Formal announcements',
        'Decision records',
        'Cross-org communication',
      ],
      expectations: 'Response within 1-2 business days',
    },
    
    documentation: {
      when: [
        'Permanent reference',
        'Decisions and rationale',
        'Technical specs',
        'Onboarding information',
      ],
      expectations: 'Read before asking questions',
    },
    
    tickets: {
      when: [
        'Tracking work',
        'Bug reports',
        'Feature requests',
        'Handoffs between teams',
      ],
      expectations: 'Per team SLA',
    },
  },
  
  sync: {
    meeting: {
      when: [
        'Complex discussions with back-and-forth',
        'Relationship building',
        'Conflict resolution',
        'Brainstorming',
      ],
      tips: [
        'Send agenda beforehand',
        'Keep under 30 minutes if possible',
        'Document decisions',
        'Only invite necessary people',
      ],
    },
    
    call: {
      when: [
        'Urgent issues',
        'Quick clarification (< 5 min)',
        'Sensitive topics',
      ],
      tips: 'Always follow up with written summary',
    },
  },
  
  escalation: {
    when: [
      'No response to async after SLA',
      'Blocking work',
      'Disagreement that can\'t be resolved',
      'Urgent production issue',
    ],
    how: [
      'Direct message first',
      'Then their team lead',
      'Then shared manager',
    ],
  },
};

/**
 * SLA expectations between teams
 */
const teamSLAs = {
  slackMessage: '24 hours (business days)',
  ticketTriage: '48 hours',
  codeReview: '24 hours',
  apiChangeNotice: '2 weeks before deployment',
  breakingChangeRfc: '4 weeks review period',
  incidentResponse: 'Per severity (P1: 15min, P2: 1hr)',
};
```

---

## API Contracts and Dependencies

### Documenting API Contracts

```yaml
# cross-team/api-contract.yaml

# OpenAPI spec that serves as contract between teams
openapi: 3.0.3
info:
  title: User Service API
  description: |
    API for user management. 
    
    ## Ownership
    - **Team:** Identity Team
    - **Slack:** #identity-team
    - **On-call:** identity-oncall@pagerduty
    
    ## SLA
    - Availability: 99.9%
    - Latency: p99 < 200ms
    
    ## Change Policy
    - Breaking changes: RFC required, 4-week notice
    - Non-breaking: Announced in #api-changes, 1-week notice
    
    ## Versioning
    - URL versioned: /v1/, /v2/
    - Current supported: v1, v2
    - Deprecated: none
    
  version: 2.0.0
  
servers:
  - url: https://api.internal/users/v2
    description: Production
  - url: https://api.staging/users/v2
    description: Staging

paths:
  /users/{id}:
    get:
      summary: Get user by ID
      description: |
        Returns user details. 
        
        **Rate limit:** 1000 req/min per client
        **Cache:** 5 minute TTL recommended
        
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
            format: uuid
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          description: User not found

components:
  schemas:
    User:
      type: object
      required:
        - id
        - email
        - created_at
      properties:
        id:
          type: string
          format: uuid
          description: Unique user identifier
        email:
          type: string
          format: email
        name:
          type: string
          description: "Display name (optional, may be null)"
        created_at:
          type: string
          format: date-time
        # New fields are added as optional to maintain compatibility
        metadata:
          type: object
          additionalProperties: true
          description: "Added in v2.1 - custom metadata"
```

### Managing Breaking Changes

```markdown
# RFC: User Service API v3

## Status
**Draft** → Under Review → Approved → Implementing → Complete

## Summary
Proposal to release User Service API v3 with breaking changes to 
support multi-tenant architecture.

## Motivation
Current API assumes single-tenant. Enterprise customers require 
tenant isolation. Current schema doesn't support tenant_id on users.

## Breaking Changes

### 1. User ID format change
- **Current:** UUID (e.g., `550e8400-e29b-41d4-a716-446655440000`)
- **New:** Prefixed UUID (e.g., `usr_550e8400-e29b-41d4-a716-446655440000`)
- **Impact:** All consumers storing/parsing user IDs

### 2. Required `tenant_id` header
- **Current:** Not required
- **New:** `X-Tenant-ID` header required on all requests
- **Impact:** All consumers must pass tenant context

### 3. Response schema change
- **Current:** `{ id, email, name }`
- **New:** `{ id, email, name, tenant_id }`
- **Impact:** Consumers parsing responses

## Migration Plan

### Timeline
- **Week 1-2:** RFC review period
- **Week 3:** v3 deployed (v2 still available)
- **Week 4-8:** Consumer migration
- **Week 9:** v2 deprecated
- **Week 17:** v2 sunset

### Support During Migration
- Identity team available for pairing
- Migration guide: [link]
- Office hours: Thursdays 2-3pm

## Affected Teams
- [ ] **Payments Team** - Uses user lookup (contacted: 2024-01-15)
- [ ] **Notifications Team** - Uses user preferences (contacted: 2024-01-15)
- [ ] **Analytics Team** - Stores user events (contacted: 2024-01-16)

## Feedback
Please comment by **2024-02-01**.

Questions: #identity-team or @alice
```

### Dependency Tracking

```typescript
// cross-team/dependencies.ts

interface ServiceDependency {
  consumer: string;
  provider: string;
  type: 'api' | 'event' | 'database' | 'library';
  criticality: 'critical' | 'high' | 'medium' | 'low';
  contract: string;  // Link to contract/spec
  owner: string;
  slackChannel: string;
  fallbackBehavior: string;
}

const serviceDependencies: ServiceDependency[] = [
  {
    consumer: 'Order Service',
    provider: 'User Service',
    type: 'api',
    criticality: 'critical',
    contract: 'https://docs/user-api-spec',
    owner: 'Identity Team',
    slackChannel: '#identity-team',
    fallbackBehavior: 'Return cached user data, queue for retry',
  },
  {
    consumer: 'Email Service',
    provider: 'Order Service',
    type: 'event',
    criticality: 'high',
    contract: 'https://docs/order-events-spec',
    owner: 'Commerce Team',
    slackChannel: '#commerce-team',
    fallbackBehavior: 'Store event in dead letter queue, alert',
  },
];

/**
 * Dependency risk assessment
 */
const assessDependencyRisk = (dep: ServiceDependency): string[] => {
  const risks: string[] = [];
  
  if (dep.criticality === 'critical' && dep.fallbackBehavior === 'None') {
    risks.push('Critical dependency with no fallback - single point of failure');
  }
  
  if (!dep.contract) {
    risks.push('No documented contract - high risk of breaking changes');
  }
  
  return risks;
};
```

---

## Real-World Scenarios

### Scenario 1: Coordinating a Cross-Team Feature

```typescript
// cross-team/feature-coordination.ts

/**
 * Example: Launch new "Premium Subscription" feature
 * Requires: Payments, Users, Notifications, Frontend teams
 */

const featureCoordination = {
  
  kickoff: {
    meeting: 'All-hands kickoff with all team leads',
    outcomes: [
      'Shared understanding of requirements',
      'Rough timeline',
      'RACI assignment',
      'Communication plan',
    ],
  },
  
  weeklySync: {
    participants: 'One rep from each team',
    duration: '30 minutes',
    agenda: [
      'Progress updates',
      'Blockers',
      'Integration points discussion',
      'Upcoming milestones',
    ],
  },
  
  sharedChannel: {
    name: '#project-premium-subs',
    purpose: 'Async communication for the project',
    contents: [
      'Decision log',
      'Blockers as they arise',
      'Links to specs and docs',
    ],
  },
  
  integrationPlan: {
    approach: 'Build in parallel, integrate incrementally',
    milestones: [
      { week: 2, milestone: 'API contracts agreed' },
      { week: 4, milestone: 'Individual services complete' },
      { week: 5, milestone: 'Integration testing' },
      { week: 6, milestone: 'End-to-end testing' },
      { week: 7, milestone: 'Staged rollout' },
    ],
  },
  
  riskMitigation: {
    'API changes': 'Lock contracts early, use contract tests',
    'Timeline slip': 'Buffer in schedule, scope cut options',
    'Integration issues': 'Early integration environment, daily integration builds',
  },
};
```

### Scenario 2: Handling Urgent Cross-Team Issue

```typescript
// cross-team/urgent-issue.ts

/**
 * Scenario: Production bug affecting another team's service
 */

const urgentIssueResponse = {
  
  step1_detect: {
    action: 'Your monitoring shows errors from dependency',
    response: 'Check if it\'s your issue or theirs',
  },
  
  step2_communicate: {
    // If it's their issue affecting you
    immediate: [
      'Post in their team channel with details',
      'Tag their on-call if P1/P2',
      'Include: symptoms, impact, when started, what you see',
    ],
    
    template: `
      🚨 **Urgent** - Seeing issues with [Service Name]
      
      **Impact:** [Your service] unable to [function], affecting [X users]
      **Started:** [time]
      **Symptoms:** [What you're seeing - error rates, timeouts, etc.]
      **Your dashboards:** [link]
      
      Is this a known issue? Do you need help from our side?
      
      cc: @oncall-identity
    `,
  },
  
  step3_collaborate: {
    offer: [
      'Share relevant logs/metrics',
      'Set up shared call if needed',
      'Help with testing once fixed',
    ],
    avoid: [
      'Blame',
      'Demanding immediate fix',
      'Bypassing their process',
    ],
  },
  
  step4_followUp: {
    afterResolution: [
      'Thank them for quick response',
      'Discuss if joint post-mortem needed',
      'Update your runbooks if needed',
    ],
  },
};
```

---

## Common Pitfalls

### 1. Assuming Shared Context

```typescript
// ❌ BAD: "We're changing the user endpoint"
// (Which endpoint? What change? When? Who's affected?)

// ✅ GOOD: "We're deprecating GET /users/v1/{id} on March 1st.
// Please migrate to /users/v2/{id}. Changes: [doc link]
// Affected teams: Payments, Notifications. Questions: #identity-team"
```

### 2. Last-Minute Surprises

```typescript
// ❌ BAD: Breaking change deployed with no notice
// "We fixed a bug in the API response format"
// (Other teams' services break)

// ✅ GOOD: RFC for breaking changes, 4+ weeks notice
// Non-breaking changes announced 1 week ahead
// Migration support offered
```

### 3. No Single Source of Truth

```typescript
// ❌ BAD: API spec in wiki, some in Slack, some in code comments
// No one knows what's current

// ✅ GOOD: OpenAPI spec in repo, auto-generated docs
// One canonical source, versioned with code
// Clear ownership labeled
```

---

## Interview Questions

### Q1: How do you manage dependencies between teams?

**A:** I make dependencies explicit and documented. API contracts (OpenAPI specs) serve as source of truth. I use contract testing to catch breaking changes early. I establish SLAs for response times. For cross-team features, I create RACI matrices and have regular syncs. I treat every dependency as a risk and plan fallbacks.

### Q2: How do you communicate breaking changes?

**A:** I follow an RFC process: document the change, impact, migration path, and timeline. I give at least 4 weeks notice for breaking changes. I proactively reach out to affected teams, don't just broadcast. I offer migration support (docs, office hours, pairing). I maintain old versions during migration period.

### Q3: How do you resolve conflicts between teams?

**A:** First, I try to understand both perspectives - usually conflicts come from different priorities or missing context. I focus on shared goals ("we both want X to work for users"). I suggest data-driven decisions when possible. If we can't resolve it, I escalate to shared leadership for prioritization. I document decisions so they don't resurface.

### Q4: How do you handle urgent issues affecting another team?

**A:** I communicate immediately in their team channel with specifics: what I'm seeing, impact, when it started. I tag their on-call for P1/P2. I offer to help - share logs, join a call, test fixes. I avoid blame. After resolution, I thank them and suggest a joint post-mortem if warranted.

---

## Quick Reference Checklist

### Starting Cross-Team Work
- [ ] Identify all stakeholders
- [ ] Create RACI matrix
- [ ] Set up shared channel
- [ ] Agree on SLAs
- [ ] Document contracts

### API Changes
- [ ] RFC for breaking changes
- [ ] 4-week review period
- [ ] Notify affected teams directly
- [ ] Provide migration guide
- [ ] Support during migration

### Ongoing Communication
- [ ] Weekly syncs for active projects
- [ ] Async updates in shared channel
- [ ] Decisions documented
- [ ] Escalation path clear

---

*Last updated: February 2026*

