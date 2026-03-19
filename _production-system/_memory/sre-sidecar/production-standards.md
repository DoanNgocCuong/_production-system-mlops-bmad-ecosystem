# Production Standards for PSM

SRE operational standards, incident response protocols, and production quality benchmarks for the Production Systems & MLOps module.

## User Specified CRITICAL Rules - Supersedes General Rules

None

## General CRITICAL RULES

### Rule 1: SLO-First Approach

ALL production decisions MUST reference defined SLOs. No optimization without measurement baseline.

### Rule 2: Blameless Postmortems

NEVER assign individual blame in incident analysis. Focus on systemic improvements and process fixes.

### Rule 3: Change Management

ALL production changes MUST have rollback plan, monitoring review, and stakeholder communication before deployment.

### Rule 4: Severity Classification

| SEV | Definition | Response Time | Examples |
|-----|-----------|---------------|----------|
| SEV1 | Complete outage, >50% users affected | Immediate (on-call paged) | Auth down, payment 100% failure |
| SEV2 | Major degradation, >20% users affected | 15 min | 50% error rate, p99 > 10s |
| SEV3 | Minor impact, <20% users affected | 1 hour | 10% error rate, single feature down |
| SEV4 | Cosmetic, few users, deferrable | Next business day | UI glitch, non-blocking bug |

### Rule 5: On-Call Principles

- Acknowledge alerts within 5 minutes
- Escalate when MTTR exceeds 30 minutes
- Update stakeholders every 15 minutes for SEV1/SEV2
- Document all actions taken during incident

### Rule 6: Observability Before Action

NEVER make changes to production without first checking metrics, logs, and traces. Data before decision.

## Operational Standards

### Incident Response

1. **Detect**: Alert fires → on-call notified within 5 min
2. **Triage**: Classify SEV, assess scope, declare incident
3. **Command**: Assign incident commander, open incident channel
4. **Diagnose**: Gather data from observability stack
5. **Fix**: Apply minimal change to restore service
6. **Verify**: Confirm metrics return to baseline
7. **Postmortem**: Document within 48 hours

### Reliability Patterns

- Circuit breaker: Failure threshold 50%, reset timeout 30s
- Retry: Exponential backoff, max 3 attempts, jitter
- Timeout: Per-dependency timeouts, never infinite waits
- Fallback: Always have degraded-mode capability
- Bulkhead: Isolate resource pools per dependency

### Monitoring Standards

- Golden signals: Latency, Traffic, Errors, Saturation
- SLO targets must be defined before launch
- Alert on symptoms, not causes
- Alert noise ratio target: < 0.1 (10% actionable)
- Runbooks for every alert

### Deployment Safety

- Blue-green or canary deployment mandatory
- Feature flags for risky changes
- Database migrations: backward-compatible first
- Rollback tested in staging before production
- Post-deploy smoke test within 5 minutes
