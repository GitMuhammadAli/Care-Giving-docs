# On-Call & Incident Response - Complete Guide

> **MUST REMEMBER**: On-call = being available to respond to production issues. Incident response: Detect → Triage → Mitigate → Communicate → Resolve → Post-mortem. Severity levels define response time (P1: minutes, P4: next business day). Always mitigate first, fix root cause later. Blameless post-mortems improve systems, not blame people. Runbooks reduce MTTR (Mean Time To Recovery).

---

## How to Explain Like a Senior Developer

"On-call means you're the first responder when production breaks. The goal isn't to prevent all incidents - it's to detect them fast, mitigate quickly, and learn from them. When an alert fires, first assess severity: is the site down? Can users work around it? Then communicate: stakeholders need to know what's happening. Focus on mitigation (stop the bleeding) before root cause (why it's bleeding). After the incident, do a blameless post-mortem - not to find who screwed up, but to find what systems failed and how to prevent recurrence. Good on-call has clear runbooks, reasonable alert thresholds, and proper rotation so no one burns out."

---

## Incident Severity Levels

### Defining Severity

```typescript
// incident-response/severity.ts

interface SeverityLevel {
  level: 'P1' | 'P2' | 'P3' | 'P4';
  name: string;
  description: string;
  responseTime: string;
  examples: string[];
  communication: string;
  escalation: string;
}

const severityLevels: SeverityLevel[] = [
  {
    level: 'P1',
    name: 'Critical',
    description: 'Complete outage or major functionality unavailable for all users',
    responseTime: '< 15 minutes',
    examples: [
      'Site completely down',
      'Payment processing broken',
      'Data breach / security incident',
      'Data loss occurring',
    ],
    communication: 'Immediate stakeholder notification, public status page',
    escalation: 'Page on-call, notify engineering manager, prepare exec summary',
  },
  {
    level: 'P2',
    name: 'High',
    description: 'Major feature broken for many users, or degraded for all',
    responseTime: '< 30 minutes',
    examples: [
      'Login broken for subset of users',
      'API response times > 10s',
      'Key integration down',
      'Mobile app crashing frequently',
    ],
    communication: 'Notify stakeholders, update status page',
    escalation: 'Page on-call, involve team leads if needed',
  },
  {
    level: 'P3',
    name: 'Medium',
    description: 'Minor feature broken, workaround available',
    responseTime: '< 4 hours (business hours)',
    examples: [
      'Non-critical feature unavailable',
      'Performance degradation for some users',
      'Error rate elevated but under threshold',
    ],
    communication: 'Team notification, status page if user-facing',
    escalation: 'On-call responds, no page if after hours',
  },
  {
    level: 'P4',
    name: 'Low',
    description: 'Minor issue, no immediate user impact',
    responseTime: 'Next business day',
    examples: [
      'Internal tool issue',
      'Warning in logs',
      'Minor UI bug',
      'Documentation error',
    ],
    communication: 'Team notification',
    escalation: 'Normal ticket queue',
  },
];
```

### Incident Lifecycle

```typescript
// incident-response/lifecycle.ts

interface Incident {
  id: string;
  title: string;
  severity: 'P1' | 'P2' | 'P3' | 'P4';
  status: IncidentStatus;
  timeline: TimelineEntry[];
  commander: string;
  participants: string[];
  customerImpact: string;
  rootCause?: string;
  resolution?: string;
}

type IncidentStatus = 
  | 'detected'      // Alert fired
  | 'acknowledged'  // Someone is looking
  | 'investigating' // Actively debugging
  | 'mitigating'    // Applying fix
  | 'resolved'      // Fix confirmed working
  | 'postmortem'    // Learning from it
  ;

interface TimelineEntry {
  timestamp: Date;
  action: string;
  author: string;
}

/**
 * Incident response workflow
 */
const incidentWorkflow = {
  
  step1_detect: {
    description: 'Alert fires, incident detected',
    actions: [
      'Alert routes to on-call',
      'On-call acknowledges (stops escalation)',
      'Create incident channel (#inc-YYYYMMDD-title)',
    ],
  },
  
  step2_triage: {
    description: 'Assess severity and impact',
    actions: [
      'Determine severity level (P1-P4)',
      'Identify affected systems/users',
      'Assign incident commander (usually on-call)',
      'Notify stakeholders per severity',
    ],
    triageQuestions: [
      'Is the site up?',
      'How many users affected?',
      'Is there data loss/security risk?',
      'Is there a workaround?',
      'Is it getting worse?',
    ],
  },
  
  step3_mitigate: {
    description: 'Stop the bleeding first',
    actions: [
      'Apply quickest fix to restore service',
      'Rollback if recent deployment caused it',
      'Scale up if capacity issue',
      'Enable circuit breaker if dependency issue',
      'Redirect traffic if regional issue',
    ],
    principle: 'Mitigate first, root cause later',
  },
  
  step4_communicate: {
    description: 'Keep stakeholders informed',
    actions: [
      'Update status page',
      'Post in incident channel every 15-30 min',
      'Notify customer support',
      'Prepare customer communication if needed',
    ],
    template: `
      **Status Update - [TIME]**
      
      Current status: [Investigating/Mitigating/Resolved]
      Impact: [What users are experiencing]
      Next steps: [What we're doing now]
      ETA: [When we expect resolution, or "investigating"]
    `,
  },
  
  step5_resolve: {
    description: 'Confirm issue is fixed',
    actions: [
      'Verify metrics returned to normal',
      'Confirm user reports resolved',
      'Document what was done',
      'Update status page to resolved',
      'Schedule post-mortem',
    ],
  },
  
  step6_postmortem: {
    description: 'Learn and improve',
    timeframe: 'Within 48-72 hours of resolution',
    // Detailed below
  },
};
```

---

## On-Call Best Practices

### Setting Up On-Call

```typescript
// incident-response/oncall-setup.ts

interface OnCallRotation {
  teamSize: number;
  rotationLength: string;
  primaryBackup: boolean;
  followTheSun: boolean;
  compensation: string;
}

const onCallBestPractices = {
  
  rotation: {
    idealLength: '1 week',
    why: 'Long enough for continuity, short enough to not burn out',
    
    structure: {
      primary: 'First responder, carries pager',
      secondary: 'Backup if primary unavailable, shadows for learning',
      escalation: 'Manager or senior engineer for P1',
    },
    
    handoff: {
      when: 'Start of business day (not midnight)',
      process: [
        'Review active incidents',
        'Discuss recent issues',
        'Update contact info in PagerDuty',
        'Test pager is working',
      ],
    },
  },
  
  alerting: {
    principles: [
      'Only page for actionable issues',
      'Tune thresholds to reduce noise',
      'Every page should require human action',
      'Aim for < 2 pages per on-call shift',
    ],
    
    badAlerts: [
      'Warning-level alerts that page',
      'Alerts that auto-resolve quickly',
      'Alerts with no runbook',
      'Duplicate alerts for same issue',
    ],
    
    goodAlerts: [
      'Clear title indicating the problem',
      'Link to runbook',
      'Link to dashboard',
      'Severity level',
    ],
  },
  
  tooling: {
    required: [
      'PagerDuty/OpsGenie (paging)',
      'Slack/Teams (communication)',
      'Datadog/Grafana (monitoring)',
      'Status page (communication)',
      'Runbook repository',
    ],
    
    recommended: [
      'Incident management tool (incident.io)',
      'On-call schedule tool',
      'Post-mortem templates',
    ],
  },
  
  selfCare: {
    guidelines: [
      'Swap shifts if you need a break',
      'Take comp time after busy shifts',
      'Speak up if alert volume is too high',
      'Don\'t sacrifice sleep for non-critical alerts',
    ],
  },
};
```

### Runbook Template

```markdown
# Runbook: [Alert Name]

## Overview
- **Service:** [service name]
- **Alert:** [alert description]
- **Severity:** [typical severity]
- **Last updated:** [date]

## What This Alert Means
[Brief explanation of what triggers this alert and why it matters]

## Impact
- **User impact:** [What users experience when this fires]
- **Business impact:** [Revenue/reputation impact]

## Quick Links
- [Dashboard](link)
- [Logs](link)
- [Service repo](link)
- [Service owner](link)

## Investigation Steps

### 1. Check if it's real
- [ ] Verify alert in monitoring dashboard
- [ ] Check if multiple alerts (possible upstream issue)
- [ ] Check if it auto-resolved

### 2. Identify scope
```bash
# Check error rate
curl https://api.example.com/health

# Check recent logs
kubectl logs -f deployment/service-name --tail=100 | grep ERROR
```

### 3. Common causes and fixes

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| High error rate after deploy | Bad deployment | Rollback |
| Gradual increase in latency | Memory leak | Restart pods |
| Sudden spike in errors | Upstream dependency | Check dependency status |

### 4. Mitigation Steps

**Rollback:**
```bash
kubectl rollout undo deployment/service-name
```

**Scale up:**
```bash
kubectl scale deployment/service-name --replicas=5
```

**Enable circuit breaker:**
```bash
kubectl set env deployment/service-name CIRCUIT_BREAKER_ENABLED=true
```

## Escalation
- **Primary on-call:** [rotation link]
- **Service owner:** @team-name
- **Escalation (if unresolved after 30min):** @engineering-manager

## Historical Context
- [Link to past incident]
- [Link to related post-mortem]
```

---

## Post-Mortem Process

### Blameless Post-Mortem Template

```markdown
# Post-Mortem: [Incident Title]

**Date of Incident:** 2024-01-15
**Duration:** 2 hours 15 minutes
**Severity:** P1
**Author:** [name]
**Status:** Complete

---

## Summary
[2-3 sentence summary of what happened and impact]

On January 15, 2024, our payment service was unavailable for 2 hours and 15 minutes, 
affecting approximately 50,000 users who could not complete purchases. The root cause 
was a database connection pool exhaustion triggered by a query that was not using indexes.

---

## Timeline (All times UTC)

| Time | Event |
|------|-------|
| 14:00 | Deployment of payment-service v2.3.1 |
| 14:15 | Error rate alert fires |
| 14:17 | On-call acknowledges alert |
| 14:20 | Incident declared, #inc-20240115-payment created |
| 14:25 | Identified connection pool exhaustion |
| 14:35 | Rolled back to v2.3.0 |
| 14:40 | Error rate returning to normal |
| 14:45 | Confirmed service recovered |
| 16:15 | Post-mortem meeting scheduled |

---

## Impact

**Customer Impact:**
- ~50,000 users unable to complete purchases
- Estimated $150,000 in lost revenue
- Support received 200+ tickets

**Technical Impact:**
- Payment service: 2h 15m downtime
- Checkout service: Degraded (dependent on payment)
- No data loss

---

## Root Cause

The deployment included a new feature that queried the orders table with a filter 
on `status` column, which was not indexed. At scale, this query took 30+ seconds, 
holding database connections. Connection pool (max 20) was exhausted, causing all 
subsequent requests to fail.

```sql
-- The problematic query
SELECT * FROM orders WHERE status = 'pending' AND created_at > '2024-01-01'
-- Missing index on (status, created_at)
```

---

## Contributing Factors

1. **No index on status column** - Query worked fine in staging (small dataset)
2. **No query performance tests** - CI didn't catch slow queries
3. **Connection pool too small** - 20 connections insufficient for load
4. **No circuit breaker** - Failures cascaded to all requests

---

## What Went Well

- Alert fired within 15 minutes of issue starting
- On-call responded quickly
- Rollback was smooth
- Communication was clear

---

## What Went Poorly

- Took 20 minutes to identify root cause
- No runbook for connection pool issues
- Staging didn't catch the issue
- Customer communication was delayed

---

## Action Items

| Action | Owner | Due Date | Status |
|--------|-------|----------|--------|
| Add index on orders(status, created_at) | @alice | 2024-01-17 | ✅ Done |
| Increase connection pool to 50 | @bob | 2024-01-17 | ✅ Done |
| Add query performance test to CI | @alice | 2024-01-22 | 🔄 In Progress |
| Create runbook for connection pool alerts | @carol | 2024-01-22 | ⏳ Pending |
| Add circuit breaker to payment service | @bob | 2024-01-29 | ⏳ Pending |
| Staging dataset to match prod scale | @dave | 2024-02-15 | ⏳ Pending |

---

## Lessons Learned

1. **Query performance testing is critical** - We need automated tests that catch slow queries before production.

2. **Connection pools need headroom** - Our pool was sized for average load, not spikes. We should size for peak + 20%.

3. **Circuit breakers prevent cascading failures** - A single slow query shouldn't take down the entire service.

---

## References

- [Incident Slack channel](#inc-20240115-payment)
- [Alert that fired](link)
- [Dashboard during incident](link)
- [Related PR that fixed the issue](link)
```

---

## Common Pitfalls

### 1. Blaming People in Post-Mortems

```typescript
// ❌ BAD: "Bob deployed bad code and caused the outage"
// Creates fear, people hide problems

// ✅ GOOD: "The deployment process didn't catch the issue"
// Focus on systems, not people
// Ask: "What made it easy to make this mistake?"
```

### 2. Alert Fatigue

```typescript
// ❌ BAD: 50 pages per week, most are noise
// People start ignoring alerts

// ✅ GOOD: Every page requires human action
// Target: < 2 pages per on-call shift
// Regularly review and tune alerts
```

### 3. Fixing Root Cause During Incident

```typescript
// ❌ BAD: Spending hours debugging while site is down
// Users are suffering

// ✅ GOOD: Mitigate first (rollback, scale, restart)
// Then investigate root cause calmly
// Proper fix in follow-up PR
```

---

## Interview Questions

### Q1: How do you handle a P1 incident?

**A:** First, acknowledge the alert and assess severity. Create an incident channel for communication. Focus on mitigation - rollback, scale up, or enable fallbacks. Communicate status every 15-30 minutes. Once mitigated, document what happened and schedule a post-mortem. The key is: mitigate first, root cause later.

### Q2: What makes a good on-call rotation?

**A:** Clear escalation paths, reasonable alert volume (< 2 pages/shift target), good runbooks, proper handoffs, and compensation for off-hours work. Alerts should be actionable - every page should require human intervention. The team should regularly review alert quality and tune thresholds.

### Q3: How do you run a blameless post-mortem?

**A:** Focus on systems, not people. Ask "what made this possible?" not "who did this?". Create a timeline of events. Identify root cause and contributing factors. Celebrate what went well. Create specific, assigned action items. The goal is learning and prevention, not blame.

### Q4: How do you reduce alert fatigue?

**A:** Audit alert quality regularly - remove noisy alerts, tune thresholds, combine duplicate alerts. Every alert should have a runbook. Track alert volume and page frequency. If an alert fires frequently without action needed, it shouldn't page. Use different severity for informational vs actionable alerts.

---

## Quick Reference Checklist

### When Paged
- [ ] Acknowledge alert
- [ ] Assess severity
- [ ] Create incident channel (P1/P2)
- [ ] Communicate status
- [ ] Mitigate (don't debug endlessly)
- [ ] Document timeline
- [ ] Schedule post-mortem

### Post-Mortem
- [ ] Blameless tone
- [ ] Clear timeline
- [ ] Root cause identified
- [ ] Contributing factors listed
- [ ] Action items assigned
- [ ] Follow up on action items

### On-Call Hygiene
- [ ] Test pager works
- [ ] Review recent incidents at handoff
- [ ] Keep runbooks updated
- [ ] Track alert volume
- [ ] Tune noisy alerts

---

*Last updated: February 2026*

