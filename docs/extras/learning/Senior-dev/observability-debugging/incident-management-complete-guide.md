# Incident Management - Complete Guide

> **MUST REMEMBER**: Incidents require clear roles (commander, communicator, responders), structured processes (detect → triage → mitigate → resolve), and blameless postmortems. Define SLOs/SLAs/SLIs upfront. Runbooks provide step-by-step guidance. The goal during an incident is to restore service, not find root cause - investigation comes after.

---

## How to Explain Like a Senior Developer

"Incident management is how you handle production emergencies systematically. First, you need to DETECT issues before users do - that's where monitoring, alerting, and SLIs come in. Then you TRIAGE: is this P1 (customer-facing, immediate) or P4 (minor, can wait)? During incidents, assign clear roles: one person leads (incident commander), one communicates status, others work on fixes. Focus on MITIGATION first - get the system working even if degraded. Root cause analysis happens in postmortems AFTER the incident. Runbooks are your playbooks - pre-written steps for common incidents. Track SLOs (99.9% uptime means ~44 min downtime/month) and use error budgets to balance reliability with velocity."

---

## Core Implementation

### Service Level Definitions

```typescript
// slo/definitions.ts

/**
 * SLI (Service Level Indicator): The metric you measure
 * SLO (Service Level Objective): Your target for that metric
 * SLA (Service Level Agreement): The contractual promise with consequences
 */

interface ServiceLevel {
  name: string;
  sli: {
    metric: string;
    goodThreshold: string;
  };
  slo: {
    target: number; // e.g., 0.999 = 99.9%
    window: string; // e.g., "30d"
  };
  sla?: {
    target: number;
    consequence: string;
  };
}

export const serviceLevels: ServiceLevel[] = [
  {
    name: 'API Availability',
    sli: {
      metric: 'successful_requests / total_requests',
      goodThreshold: 'status_code < 500',
    },
    slo: {
      target: 0.999, // 99.9%
      window: '30d',
    },
    sla: {
      target: 0.995, // 99.5%
      consequence: '10% credit for each 0.1% below target',
    },
  },
  {
    name: 'API Latency',
    sli: {
      metric: 'p95_latency',
      goodThreshold: 'latency < 200ms',
    },
    slo: {
      target: 0.99, // 99% of requests under 200ms
      window: '30d',
    },
  },
  {
    name: 'Data Freshness',
    sli: {
      metric: 'max(data_age)',
      goodThreshold: 'age < 60 seconds',
    },
    slo: {
      target: 0.999,
      window: '30d',
    },
  },
];

// Error budget calculation
export function calculateErrorBudget(
  slo: number,
  windowDays: number,
  currentSuccessRate: number
): {
  totalBudgetMinutes: number;
  usedBudgetMinutes: number;
  remainingBudgetMinutes: number;
  budgetUsedPercent: number;
} {
  const totalMinutes = windowDays * 24 * 60;
  const allowedFailureRate = 1 - slo;
  const totalBudgetMinutes = totalMinutes * allowedFailureRate;
  
  const actualFailureRate = 1 - currentSuccessRate;
  const usedBudgetMinutes = totalMinutes * actualFailureRate;
  const remainingBudgetMinutes = totalBudgetMinutes - usedBudgetMinutes;
  
  return {
    totalBudgetMinutes,
    usedBudgetMinutes,
    remainingBudgetMinutes,
    budgetUsedPercent: (usedBudgetMinutes / totalBudgetMinutes) * 100,
  };
}

// Example: 99.9% SLO over 30 days
// Total budget: 30 * 24 * 60 * 0.001 = 43.2 minutes of allowed downtime
```

### Incident Detection and Alerting

```typescript
// incidents/alerting.ts
import { logger } from '../logger';

type Severity = 'P1' | 'P2' | 'P3' | 'P4';

interface AlertRule {
  name: string;
  condition: string;
  severity: Severity;
  runbookUrl: string;
  notifyChannels: string[];
}

interface Alert {
  id: string;
  rule: AlertRule;
  triggeredAt: Date;
  value: number;
  status: 'firing' | 'acknowledged' | 'resolved';
  acknowledgedBy?: string;
  resolvedAt?: Date;
}

const alertRules: AlertRule[] = [
  {
    name: 'High Error Rate',
    condition: 'error_rate > 5%',
    severity: 'P1',
    runbookUrl: 'https://runbooks.internal/high-error-rate',
    notifyChannels: ['pagerduty', 'slack-incidents'],
  },
  {
    name: 'High Latency',
    condition: 'p99_latency > 2s for 5m',
    severity: 'P2',
    runbookUrl: 'https://runbooks.internal/high-latency',
    notifyChannels: ['slack-incidents'],
  },
  {
    name: 'Database Connection Pool Exhausted',
    condition: 'db_pool_available == 0 for 1m',
    severity: 'P1',
    runbookUrl: 'https://runbooks.internal/db-pool',
    notifyChannels: ['pagerduty', 'slack-incidents'],
  },
];

// Severity definitions
const severityConfig: Record<Severity, {
  responseTime: string;
  escalationTime: string;
  description: string;
}> = {
  P1: {
    responseTime: '5 minutes',
    escalationTime: '15 minutes',
    description: 'Customer-facing outage, data loss risk',
  },
  P2: {
    responseTime: '30 minutes',
    escalationTime: '2 hours',
    description: 'Degraded service, significant impact',
  },
  P3: {
    responseTime: '4 hours',
    escalationTime: '1 business day',
    description: 'Minor impact, workaround available',
  },
  P4: {
    responseTime: '1 business day',
    escalationTime: '1 week',
    description: 'Minimal impact, cosmetic issues',
  },
};

// Alert handling
export async function handleAlert(alert: Alert): Promise<void> {
  logger.warn({
    msg: 'Alert triggered',
    alertName: alert.rule.name,
    severity: alert.rule.severity,
    value: alert.value,
  });
  
  // Notify appropriate channels
  for (const channel of alert.rule.notifyChannels) {
    await notify(channel, alert);
  }
  
  // Create incident for P1/P2
  if (alert.rule.severity === 'P1' || alert.rule.severity === 'P2') {
    await createIncident(alert);
  }
}

async function notify(channel: string, alert: Alert): Promise<void> {
  // Implementation depends on channel
}

async function createIncident(alert: Alert): Promise<void> {
  // Create incident in incident management system
}
```

### Incident Lifecycle Management

```typescript
// incidents/lifecycle.ts
import { logger } from '../logger';

type IncidentStatus = 
  | 'detected' 
  | 'acknowledged' 
  | 'investigating' 
  | 'mitigating' 
  | 'monitoring' 
  | 'resolved';

interface Incident {
  id: string;
  title: string;
  severity: 'P1' | 'P2' | 'P3' | 'P4';
  status: IncidentStatus;
  commander?: string;
  communicator?: string;
  responders: string[];
  timeline: TimelineEvent[];
  createdAt: Date;
  acknowledgedAt?: Date;
  resolvedAt?: Date;
  impactSummary?: string;
  customerCommunication?: string[];
}

interface TimelineEvent {
  timestamp: Date;
  actor: string;
  action: string;
  details?: string;
}

// Incident management
const incidents: Map<string, Incident> = new Map();

export function createIncident(
  title: string,
  severity: Incident['severity']
): Incident {
  const incident: Incident = {
    id: `INC-${Date.now()}`,
    title,
    severity,
    status: 'detected',
    responders: [],
    timeline: [{
      timestamp: new Date(),
      actor: 'system',
      action: 'Incident created',
    }],
    createdAt: new Date(),
  };
  
  incidents.set(incident.id, incident);
  logger.info({ msg: 'Incident created', incident });
  
  return incident;
}

export function acknowledgeIncident(
  incidentId: string,
  responder: string
): void {
  const incident = incidents.get(incidentId);
  if (!incident) throw new Error('Incident not found');
  
  incident.status = 'acknowledged';
  incident.acknowledgedAt = new Date();
  incident.responders.push(responder);
  incident.timeline.push({
    timestamp: new Date(),
    actor: responder,
    action: 'Acknowledged incident',
  });
  
  logger.info({ msg: 'Incident acknowledged', incidentId, responder });
}

export function assignCommander(
  incidentId: string,
  commander: string
): void {
  const incident = incidents.get(incidentId);
  if (!incident) throw new Error('Incident not found');
  
  incident.commander = commander;
  incident.timeline.push({
    timestamp: new Date(),
    actor: commander,
    action: 'Assigned as incident commander',
  });
}

export function updateStatus(
  incidentId: string,
  status: IncidentStatus,
  actor: string,
  details?: string
): void {
  const incident = incidents.get(incidentId);
  if (!incident) throw new Error('Incident not found');
  
  const previousStatus = incident.status;
  incident.status = status;
  incident.timeline.push({
    timestamp: new Date(),
    actor,
    action: `Status changed: ${previousStatus} → ${status}`,
    details,
  });
  
  if (status === 'resolved') {
    incident.resolvedAt = new Date();
  }
  
  logger.info({ msg: 'Incident status updated', incidentId, status });
}

export function addTimelineNote(
  incidentId: string,
  actor: string,
  note: string
): void {
  const incident = incidents.get(incidentId);
  if (!incident) throw new Error('Incident not found');
  
  incident.timeline.push({
    timestamp: new Date(),
    actor,
    action: 'Added note',
    details: note,
  });
}

// Get incident duration
export function getIncidentDuration(incidentId: string): {
  total: number;
  timeToAcknowledge: number;
  timeToResolve?: number;
} {
  const incident = incidents.get(incidentId);
  if (!incident) throw new Error('Incident not found');
  
  const now = new Date();
  
  return {
    total: (incident.resolvedAt || now).getTime() - incident.createdAt.getTime(),
    timeToAcknowledge: incident.acknowledgedAt 
      ? incident.acknowledgedAt.getTime() - incident.createdAt.getTime()
      : (now.getTime() - incident.createdAt.getTime()),
    timeToResolve: incident.resolvedAt 
      ? incident.resolvedAt.getTime() - incident.createdAt.getTime()
      : undefined,
  };
}
```

### Runbook System

```typescript
// runbooks/runbook-system.ts

interface RunbookStep {
  order: number;
  title: string;
  description: string;
  command?: string;
  expectedOutput?: string;
  warningIfFailed?: string;
}

interface Runbook {
  id: string;
  title: string;
  description: string;
  severity: string[];
  symptoms: string[];
  steps: RunbookStep[];
  escalation: {
    when: string;
    to: string;
  };
  lastUpdated: Date;
  owner: string;
}

export const runbooks: Runbook[] = [
  {
    id: 'rb-high-error-rate',
    title: 'High Error Rate',
    description: 'API error rate exceeds 5%',
    severity: ['P1', 'P2'],
    symptoms: [
      'Error rate alert firing',
      'Customer complaints about failures',
      'Elevated 5xx responses in dashboards',
    ],
    steps: [
      {
        order: 1,
        title: 'Check recent deployments',
        description: 'Identify if a recent deployment caused the issue',
        command: 'kubectl rollout history deployment/api -n production',
        expectedOutput: 'List of recent deployments with timestamps',
      },
      {
        order: 2,
        title: 'Check error logs',
        description: 'Look for error patterns in recent logs',
        command: 'kubectl logs -l app=api -n production --since=10m | grep ERROR',
        expectedOutput: 'Error messages indicating the failure type',
      },
      {
        order: 3,
        title: 'Check downstream dependencies',
        description: 'Verify database and external services are healthy',
        command: 'curl -s http://api/health/deep | jq',
        expectedOutput: 'JSON with all dependencies status',
      },
      {
        order: 4,
        title: 'Rollback if recent deployment',
        description: 'If deployment caused issue, rollback immediately',
        command: 'kubectl rollout undo deployment/api -n production',
        warningIfFailed: 'Contact platform team if rollback fails',
      },
      {
        order: 5,
        title: 'Scale up if load-related',
        description: 'Increase replicas if error rate is due to load',
        command: 'kubectl scale deployment/api -n production --replicas=10',
      },
    ],
    escalation: {
      when: 'After 15 minutes without improvement or if steps fail',
      to: 'Platform team lead and engineering manager',
    },
    lastUpdated: new Date('2026-01-15'),
    owner: 'platform-team',
  },
  {
    id: 'rb-database-connection-pool',
    title: 'Database Connection Pool Exhausted',
    description: 'No available database connections',
    severity: ['P1'],
    symptoms: [
      'Connection pool alerts firing',
      'Timeout errors in logs',
      'Slow or failed API requests',
    ],
    steps: [
      {
        order: 1,
        title: 'Check active connections',
        description: 'See current connection count and what\'s using them',
        command: `psql -c "SELECT state, count(*) FROM pg_stat_activity GROUP BY state"`,
        expectedOutput: 'Connection counts by state',
      },
      {
        order: 2,
        title: 'Kill idle connections',
        description: 'Terminate long-idle connections if safe',
        command: `psql -c "SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE state = 'idle' AND query_start < now() - interval '10 minutes'"`,
      },
      {
        order: 3,
        title: 'Check for long-running queries',
        description: 'Find queries that might be holding connections',
        command: `psql -c "SELECT pid, now() - query_start as duration, query FROM pg_stat_activity WHERE state = 'active' ORDER BY duration DESC LIMIT 10"`,
      },
      {
        order: 4,
        title: 'Restart application pods (last resort)',
        description: 'Force release connections by restarting',
        command: 'kubectl rollout restart deployment/api -n production',
        warningIfFailed: 'This will cause brief service disruption',
      },
    ],
    escalation: {
      when: 'If issue persists after connection cleanup',
      to: 'Database team and platform lead',
    },
    lastUpdated: new Date('2026-01-10'),
    owner: 'database-team',
  },
];

// Find relevant runbook for symptoms
export function findRunbook(symptoms: string[]): Runbook[] {
  return runbooks.filter(rb => 
    rb.symptoms.some(s => 
      symptoms.some(symptom => 
        s.toLowerCase().includes(symptom.toLowerCase())
      )
    )
  );
}
```

### Postmortem Template

```typescript
// incidents/postmortem.ts

interface Postmortem {
  incidentId: string;
  title: string;
  date: Date;
  authors: string[];
  status: 'draft' | 'in_review' | 'published';
  
  summary: string;
  impact: {
    duration: string;
    usersAffected: string;
    revenueImpact?: string;
    slaImpact?: string;
  };
  
  timeline: {
    time: Date;
    event: string;
  }[];
  
  rootCause: string;
  
  contributing_factors: string[];
  
  actionItems: {
    id: string;
    description: string;
    owner: string;
    dueDate: Date;
    priority: 'high' | 'medium' | 'low';
    status: 'open' | 'in_progress' | 'done';
  }[];
  
  lessonsLearned: {
    whatWentWell: string[];
    whatWentPoorly: string[];
    whereWeGotLucky: string[];
  };
}

// Example postmortem
export const examplePostmortem: Postmortem = {
  incidentId: 'INC-20260115-001',
  title: 'API Outage Due to Database Connection Pool Exhaustion',
  date: new Date('2026-01-15'),
  authors: ['alice@company.com', 'bob@company.com'],
  status: 'published',
  
  summary: `On January 15, 2026, the API experienced a 47-minute outage due to 
database connection pool exhaustion. The root cause was a missing connection 
timeout in a new feature deployed earlier that day, which caused connections 
to be held indefinitely during slow database queries.`,
  
  impact: {
    duration: '47 minutes',
    usersAffected: '~12,000 users',
    revenueImpact: '$15,000 in lost transactions',
    slaImpact: 'Used 109% of monthly error budget',
  },
  
  timeline: [
    { time: new Date('2026-01-15T14:00:00'), event: 'New feature deployed to production' },
    { time: new Date('2026-01-15T15:23:00'), event: 'First connection pool warning alert' },
    { time: new Date('2026-01-15T15:28:00'), event: 'P1 alert: API error rate > 50%' },
    { time: new Date('2026-01-15T15:30:00'), event: 'Incident commander assigned' },
    { time: new Date('2026-01-15T15:35:00'), event: 'Correlated with recent deployment' },
    { time: new Date('2026-01-15T15:40:00'), event: 'Rollback initiated' },
    { time: new Date('2026-01-15T15:45:00'), event: 'Rollback complete, connections draining' },
    { time: new Date('2026-01-15T16:15:00'), event: 'Service fully recovered' },
  ],
  
  rootCause: `The new feature made database queries that could take up to 30 seconds 
under certain conditions. The code did not set a query timeout, and the connection 
pool was not configured with a maximum wait time. As slow queries accumulated, 
all connections were consumed and new requests could not be served.`,
  
  contributing_factors: [
    'Missing query timeout in new code',
    'Connection pool configuration lacked timeout settings',
    'Load testing did not simulate slow database scenarios',
    'Monitoring did not alert on connection pool saturation early enough',
  ],
  
  actionItems: [
    {
      id: 'AI-001',
      description: 'Add default query timeout to all database clients',
      owner: 'alice@company.com',
      dueDate: new Date('2026-01-22'),
      priority: 'high',
      status: 'in_progress',
    },
    {
      id: 'AI-002',
      description: 'Configure connection pool timeouts (acquire: 5s, idle: 30s)',
      owner: 'bob@company.com',
      dueDate: new Date('2026-01-19'),
      priority: 'high',
      status: 'done',
    },
    {
      id: 'AI-003',
      description: 'Add alert for connection pool > 80% utilization',
      owner: 'carol@company.com',
      dueDate: new Date('2026-01-20'),
      priority: 'high',
      status: 'done',
    },
    {
      id: 'AI-004',
      description: 'Add slow query simulation to load tests',
      owner: 'dave@company.com',
      dueDate: new Date('2026-02-01'),
      priority: 'medium',
      status: 'open',
    },
  ],
  
  lessonsLearned: {
    whatWentWell: [
      'Incident was detected within 5 minutes',
      'Rollback was smooth and quick',
      'Communication to stakeholders was timely',
    ],
    whatWentPoorly: [
      'Root cause took 10 minutes to identify',
      'No pre-deployment check for database timeouts',
      'Connection pool metrics were not on main dashboard',
    ],
    whereWeGotLucky: [
      'Incident happened during low-traffic hours',
      'The engineer who wrote the code was available',
    ],
  },
};
```

---

## Real-World Scenarios

### Scenario 1: On-Call Response Flow

```typescript
// oncall/response-flow.ts

enum ResponseStep {
  ACKNOWLEDGE = 'acknowledge',
  TRIAGE = 'triage',
  ASSEMBLE = 'assemble',
  INVESTIGATE = 'investigate',
  MITIGATE = 'mitigate',
  COMMUNICATE = 'communicate',
  RESOLVE = 'resolve',
  DOCUMENT = 'document',
}

const responsePlaybook = {
  [ResponseStep.ACKNOWLEDGE]: {
    timeLimit: '5 minutes',
    actions: [
      'Acknowledge alert in PagerDuty',
      'Join incident Slack channel',
      'Check if incident already has commander',
    ],
  },
  
  [ResponseStep.TRIAGE]: {
    timeLimit: '10 minutes',
    actions: [
      'Verify the alert is valid (not false positive)',
      'Determine severity based on impact',
      'Check for related alerts or incidents',
      'Find relevant runbook',
    ],
  },
  
  [ResponseStep.ASSEMBLE]: {
    timeLimit: '15 minutes',
    actions: [
      'Assign incident commander (if P1/P2)',
      'Page additional responders if needed',
      'Start incident bridge call for P1',
    ],
  },
  
  [ResponseStep.INVESTIGATE]: {
    timeLimit: 'Ongoing',
    actions: [
      'Follow runbook steps',
      'Check dashboards and logs',
      'Form hypothesis about cause',
      'Test hypothesis safely',
    ],
  },
  
  [ResponseStep.MITIGATE]: {
    timeLimit: 'ASAP',
    actions: [
      'Focus on restoring service, not finding root cause',
      'Consider: rollback, scale up, feature flag, failover',
      'Implement the fastest safe fix',
    ],
  },
  
  [ResponseStep.COMMUNICATE]: {
    timeLimit: 'Every 30 minutes for P1',
    actions: [
      'Update status page',
      'Post to incident channel',
      'Notify customer success for major incidents',
    ],
  },
  
  [ResponseStep.RESOLVE]: {
    timeLimit: 'When service is stable',
    actions: [
      'Verify metrics are back to normal',
      'Monitor for 15+ minutes',
      'Resolve incident and alerts',
    ],
  },
  
  [ResponseStep.DOCUMENT]: {
    timeLimit: '24-48 hours',
    actions: [
      'Create postmortem document',
      'Schedule postmortem meeting',
      'Create action items',
    ],
  },
};
```

---

## Common Pitfalls

### 1. Blame Culture in Postmortems

```markdown
❌ BAD: "The incident was caused by John's code not handling errors properly."

✅ GOOD: "The incident was caused by missing error handling in the payment 
processing code. Our code review checklist did not include error handling 
verification, and our tests did not cover this failure mode."
```

### 2. No SLOs Defined

```typescript
// ❌ BAD: Reacting to every alert without context
if (errorRate > 1%) {
  triggerPagerDuty(); // Alert fatigue!
}

// ✅ GOOD: Alert based on error budget burn rate
if (errorBudgetBurnRate > 10x) { // Burning 10x faster than sustainable
  triggerPagerDuty();
} else if (errorBudgetBurnRate > 2x) {
  sendSlackWarning();
}
```

### 3. Not Following Up on Action Items

```typescript
// Track postmortem action items with deadlines and reviews
const actionItemTracker = {
  items: [],
  
  addItem(item: { description: string; owner: string; dueDate: Date }): void {
    this.items.push({ ...item, status: 'open', createdAt: new Date() });
  },
  
  // Weekly review of overdue items
  getOverdueItems(): object[] {
    return this.items.filter(
      item => item.status === 'open' && new Date() > item.dueDate
    );
  },
  
  // Report metrics
  getMetrics(): object {
    const total = this.items.length;
    const completed = this.items.filter(i => i.status === 'done').length;
    const overdue = this.getOverdueItems().length;
    
    return {
      total,
      completed,
      completionRate: `${((completed / total) * 100).toFixed(1)}%`,
      overdue,
    };
  },
};
```

---

## Interview Questions

### Q1: What's the difference between SLI, SLO, and SLA?

**A:** SLI (Service Level Indicator) is the metric you measure - like request success rate or latency. SLO (Service Level Objective) is your internal target for that metric - like 99.9% success rate. SLA (Service Level Agreement) is the contractual promise to customers with consequences if breached - like 99.5% uptime or credits are issued.

### Q2: How do you run a blameless postmortem?

**A:** Focus on systems, not people. Ask "what conditions allowed this to happen?" not "who caused this?". Assume everyone acted with good intentions given the information they had. Look for systemic improvements: better testing, clearer runbooks, improved monitoring. The goal is learning, not punishment.

### Q3: What's an error budget and how do you use it?

**A:** Error budget is the inverse of your SLO. If you have a 99.9% SLO, you have 0.1% (43 minutes/month) of allowed downtime. When you've used your budget, freeze deployments and focus on reliability. When you have budget remaining, you can take risks with new features. It balances reliability with development velocity.

### Q4: What roles are needed during a major incident?

**A:** Incident Commander (leads response, makes decisions), Communications Lead (updates stakeholders, status page), Technical Responders (investigate and fix), Scribe (documents timeline). For P1s, one person should NOT try to do everything - clear role separation prevents chaos.

---

## Quick Reference Checklist

### Before Incidents
- [ ] Define SLOs for critical services
- [ ] Set up error budget tracking
- [ ] Create runbooks for common failures
- [ ] Establish on-call rotation
- [ ] Define severity levels and response times

### During Incidents
- [ ] Acknowledge within response time
- [ ] Assign incident commander
- [ ] Follow runbook if available
- [ ] Communicate status regularly
- [ ] Focus on mitigation, not root cause
- [ ] Document timeline as you go

### After Incidents
- [ ] Write postmortem within 48 hours
- [ ] Hold blameless review meeting
- [ ] Create and assign action items
- [ ] Track action item completion
- [ ] Update runbooks with learnings

---

*Last updated: February 2026*

