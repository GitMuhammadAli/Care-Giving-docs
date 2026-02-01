# Incident Response Runbook

Guide for responding to production incidents in CareCircle.

## Severity Levels

| Level | Description | Response Time | Examples |
|-------|-------------|---------------|----------|
| **SEV-1** | Critical - Service down | < 15 min | Complete outage, data loss, security breach |
| **SEV-2** | Major - Significant degradation | < 30 min | Core feature broken, significant performance issues |
| **SEV-3** | Minor - Limited impact | < 2 hours | Non-critical feature broken, minor bugs |
| **SEV-4** | Low - Minimal impact | < 24 hours | Cosmetic issues, minor inconveniences |

## Escalation Path

```
SEV-1/SEV-2:
  1. On-call engineer (immediate)
  2. Team lead (within 15 min if unresolved)
  3. Engineering manager (within 30 min if unresolved)
  4. CTO (for extended SEV-1 incidents)

SEV-3/SEV-4:
  1. On-call engineer
  2. Team lead (if needed)
```

## First Response Checklist

### 1. Acknowledge & Assess (First 5 minutes)

- [ ] Acknowledge the incident in monitoring channel
- [ ] Determine severity level
- [ ] Create incident channel: `#incident-YYYY-MM-DD-brief-description`
- [ ] Post initial status update

**Initial Status Template:**
```
🔴 INCIDENT DECLARED
Severity: SEV-X
Summary: [Brief description]
Impact: [Who/what is affected]
Status: Investigating
Lead: @your-name
```

### 2. Gather Information (Next 5-10 minutes)

- [ ] Check monitoring dashboards
- [ ] Review recent deployments
- [ ] Check error logs
- [ ] Identify affected services

**Key URLs:**
- Monitoring: [Your monitoring URL]
- Logs: [Your logging URL]  
- Deployments: [GitHub Actions](https://github.com/your-org/carecircle/actions)

### 3. Mitigate (Ongoing)

- [ ] Implement temporary fix or rollback if needed
- [ ] Communicate with stakeholders
- [ ] Post regular updates (every 15-30 min)

## Service-Specific Procedures

### API Service Down

**Symptoms:** 5xx errors, health check failures, no response

**Immediate Actions:**
```bash
# Check pod status
kubectl get pods -n carecircle -l app=carecircle-api

# Check logs
kubectl logs -n carecircle -l app=carecircle-api --tail=100

# Check recent events
kubectl describe deployment carecircle-api -n carecircle

# Restart if needed
kubectl rollout restart deployment/carecircle-api -n carecircle
```

**Rollback if recent deployment:**
```bash
kubectl rollout undo deployment/carecircle-api -n carecircle
```

### Database Issues

**Symptoms:** Connection errors, slow queries, timeouts

**Immediate Actions:**
```bash
# Check Neon console for database status
# https://console.neon.tech

# Check connection count (via Prisma/API)
curl -s http://api-internal/health/details | jq '.checks.database'

# For connection pool exhaustion, restart API pods
kubectl rollout restart deployment/carecircle-api -n carecircle
```

**Recovery from corruption:**
1. Check Neon console for Point-in-Time Recovery (PITR)
2. Create branch from specific timestamp
3. Verify data integrity
4. Switch connection string if needed

See: [BACKUP_PROCEDURES.md](../operations/BACKUP_PROCEDURES.md)

### Redis Issues

**Symptoms:** Cache misses, queue failures, session issues

**Immediate Actions:**
```bash
# Check Upstash console
# https://console.upstash.com

# Test connection
redis-cli -h your-redis-host ping

# Check memory usage
redis-cli info memory

# Flush cache if corrupted (CAUTION: clears all sessions)
redis-cli FLUSHDB
```

### Worker Failures

**Symptoms:** Jobs not processing, queue buildup, DLQ messages

**Immediate Actions:**
```bash
# Check worker pods
kubectl get pods -n carecircle -l app=carecircle-workers

# Check worker health
kubectl exec -n carecircle -l app=carecircle-workers -- curl localhost:3002/health

# Check logs for errors
kubectl logs -n carecircle -l app=carecircle-workers --tail=200

# Restart workers
kubectl rollout restart deployment/carecircle-workers -n carecircle
```

**Queue backup:**
```bash
# Check queue lengths via API or Redis directly
redis-cli LLEN bull:medication-reminder:wait
redis-cli LLEN bull:notification:wait
```

### Web App Down

**Symptoms:** 5xx from web, blank pages, build errors

**Immediate Actions:**
```bash
# Check pod status
kubectl get pods -n carecircle -l app=carecircle-web

# Check logs
kubectl logs -n carecircle -l app=carecircle-web --tail=100

# Rollback if recent deployment
kubectl rollout undo deployment/carecircle-web -n carecircle
```

### High CPU/Memory

**Symptoms:** Slow responses, OOM kills, pod restarts

**Immediate Actions:**
```bash
# Check resource usage
kubectl top pods -n carecircle

# Check for memory leaks in logs
kubectl logs -n carecircle -l app=carecircle-api --tail=500 | grep -i "memory\|heap\|oom"

# Scale up if needed
kubectl scale deployment/carecircle-api --replicas=4 -n carecircle
```

## Rollback Procedures

### Application Rollback

```bash
# List rollout history
kubectl rollout history deployment/carecircle-api -n carecircle

# Rollback to previous version
kubectl rollout undo deployment/carecircle-api -n carecircle

# Rollback to specific revision
kubectl rollout undo deployment/carecircle-api -n carecircle --to-revision=3

# Monitor rollback
kubectl rollout status deployment/carecircle-api -n carecircle
```

### Database Rollback

See: [BACKUP_PROCEDURES.md](../operations/BACKUP_PROCEDURES.md)

1. Use Neon's PITR to create branch at safe point
2. Verify data integrity
3. Update DATABASE_URL in secrets
4. Restart API and workers

## Communication Templates

### Initial Notification
```
🔴 We're investigating an issue affecting [service/feature].
Impact: [Brief description of impact]
Status: Our team is actively working on this.
Next update: In 15 minutes.
```

### Ongoing Update
```
🟡 UPDATE on [service/feature] issue:
- We've identified [root cause/area]
- Currently [action being taken]
- ETA for resolution: [time or "investigating"]
Next update: In [time].
```

### Resolution
```
🟢 RESOLVED: [service/feature] is now operating normally.
- Duration: [start time] to [end time] ([total duration])
- Root cause: [brief description]
- Fix: [what was done]
- Follow-up: [any pending actions]

We apologize for any inconvenience.
```

## Post-Incident

### Immediate (Within 24 hours)
- [ ] Close incident channel
- [ ] Update status page
- [ ] Notify stakeholders of resolution

### Post-Mortem (Within 48 hours)
- [ ] Schedule post-mortem meeting
- [ ] Gather timeline and logs
- [ ] Identify root cause
- [ ] Document action items
- [ ] Share learnings with team

**Post-Mortem Template:**
```markdown
# Incident Post-Mortem: [Title]
Date: [Date]
Duration: [Start] - [End]
Severity: SEV-X

## Summary
[1-2 sentence summary]

## Timeline
- HH:MM - [Event]
- HH:MM - [Event]

## Root Cause
[Description]

## Resolution
[What fixed it]

## Action Items
- [ ] [Action] - Owner: @name - Due: [date]

## Lessons Learned
- [Lesson]
```

## Useful Commands Reference

```bash
# Kubernetes Status
kubectl get all -n carecircle
kubectl top pods -n carecircle
kubectl get events -n carecircle --sort-by='.lastTimestamp'

# Logs
kubectl logs -n carecircle -l app=carecircle-api --tail=100 -f
kubectl logs -n carecircle -l app=carecircle-api --previous  # crashed pod

# Restart Services
kubectl rollout restart deployment/carecircle-api -n carecircle
kubectl rollout restart deployment/carecircle-web -n carecircle
kubectl rollout restart deployment/carecircle-workers -n carecircle

# Scale
kubectl scale deployment/carecircle-api --replicas=4 -n carecircle

# Exec into pod
kubectl exec -it -n carecircle $(kubectl get pod -n carecircle -l app=carecircle-api -o jsonpath='{.items[0].metadata.name}') -- sh
```

## Contact List

| Role | Contact | When to Reach |
|------|---------|---------------|
| On-call Engineer | @oncall | First responder |
| Team Lead | @teamlead | SEV-1/2, escalation |
| Engineering Manager | @manager | Extended SEV-1 |
| Infrastructure | @infra | K8s/cloud issues |
| Database | @dba | DB-specific issues |

---

**Remember:** Stay calm, communicate frequently, and document everything.

