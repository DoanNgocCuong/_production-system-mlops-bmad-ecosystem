# Incident Patterns for PSM

Learned incident patterns, recurring failure modes, and recovery playbooks accumulated from past production events.

## Incident Pattern Log

<!-- Record incident patterns with timestamps for future reference -->

| Date | Pattern | Root Cause | Fix Applied | Recurrence Risk |
|------|---------|-----------|-------------|----------------|
| | | | | |

## Common Failure Modes

### Database Layer

| Pattern | Symptoms | Quick Fix | Prevention |
|---------|----------|----------|------------|
| Connection pool exhaustion | 500 errors, connection timeouts | Increase pool size, kill long queries | Monitor pool utilization trend |
| Long-running queries | Latency spike, slow queries in logs | Add indexes, optimize query | Query review gate in CI |
| Replication lag | Stale reads, inconsistent data | Reduce write load on primary | Monitor lag metric, alerts |
| Lock contention | Requests blocked, thread stacks | Kill blocking transaction | Shorter transactions |

### API/Service Layer

| Pattern | Symptoms | Quick Fix | Prevention |
|---------|----------|----------|------------|
| N+1 queries | Latency spike, many DB calls | JOIN optimization | ORM query review |
| Memory leak | OOM kills, gradual degradation | Restart service | Monitor RSS, memory trends |
| Thread pool exhaustion | Requests queued, timeouts | Increase threads, optimize | Monitor thread pool util |
| Circuit breaker open | 503 errors, downstream calls failing | Check downstream health | Timeout + retry with backoff |

### Caching Layer

| Pattern | Symptoms | Quick Fix | Prevention |
|---------|----------|----------|------------|
| Cache miss storm | DB load spike, latency | Warm cache, rate limit | Cache-aside pattern |
| Cache key collision | Data corruption, wrong results | Flush cache, regenerate | Consistent key hashing |
| Redis connection limit | Eviction failures, errors | Increase max clients | Monitor connected clients |

### Deployment

| Pattern | Symptoms | Quick Fix | Prevention |
|---------|----------|----------|------------|
| Migration lock | Deploy hangs, service down | Cancel migration, rollback | Zero-downtime migration pattern |
| Config drift | Unexpected behavior | Sync config, redeploy | Config as code, GitOps |
| Dependency version | Import errors, breaking changes | Pin versions, rollback | Dependency audit in CI |

## Recovery Playbooks

### Playbook: Database Connection Pool Exhaustion

```
SYMPTOMS:
- HTTP 500 on API requests
- "connection refused" errors
- Database connection count at max

IMMEDIATE (0-5 min):
1. Check current connection count: SELECT count(*) FROM pg_stat_activity;
2. Identify blocking queries: SELECT * FROM pg_stat_activity WHERE state = 'active' AND query_start < now() - interval '5 minutes';
3. Kill long-running queries if safe
4. Increase connection pool limit temporarily

SHORT-TERM (5-30 min):
1. Identify root cause (long query, pool too small, connection leak)
2. Apply fix (add index, increase pool, fix leak)
3. Monitor recovery

PREVENTION:
- Alert on connection pool > 70% sustained
- Query timeout at application layer
- Connection pool monitoring trend
```

### Playbook: High Latency Spike

```
SYMPTOMS:
- p99 latency > 2x baseline
- Specific endpoint affected

IMMEDIATE (0-5 min):
1. Check if recent deployment (git log, K8s rollout history)
2. Check database slow query log
3. Check upstream dependencies (external APIs, DB)

DIAGNOSE:
- N+1 query? → Check ORM queries
- Index missing? → Check EXPLAIN ANALYZE
- External dependency slow? → Check their status page
- GC pause? → Check JVM metrics if applicable

FIX:
- Rollback if deployment-related
- Add index if query-related
- Circuit break if external

PREVENTION:
- Load test in staging
- Query performance regression in CI
- SLO alerting on latency
```

## Lessons Learned Template

```
## Incident: [Brief Title]

**Date**: YYYY-MM-DD
**SEV**: SEV1/2/3/4
**Duration**: X hours Y minutes
**Root Cause**: [1-sentence description]

### What Happened
[2-3 paragraphs on timeline and impact]

### Root Cause
[Technical explanation of why]

### What Went Well
- [Positive observation 1]
- [Positive observation 2]

### What Could Be Better
- [Improvement area 1]
- [Improvement area 2]

### Action Items
| Action | Owner | Due |
|--------|-------|-----|
| | | |
```
