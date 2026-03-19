---
step_id: PRR-S02
step_name: Deep Review
workflow_id: PRR001
sequence: 2
type: "analysis"
agent: "Minh (SRE)"
description: "Perform technical deep review of each dimension, score, identify blockers"
estimated_time: "90-120 minutes"
---

# Step 02: Deep Technical Review

## Objective

Conduct systematic technical evaluation across all 9 dimensions using Production Systems BMAD framework:
- Score each dimension (RED/YELLOW/GREEN)
- Identify concrete blockers and risks
- Generate recommendations and action items
- Build comprehensive scorecard

## Execution Flow

### Substep 2A: Reliability Review

**Skill Reference**: 5.10 SLO & Error Budget

Evaluate:
- [ ] SLO defined? (e.g., 99.9% availability, P99 latency < 500ms)
- [ ] Error budget calculated and communicated?
- [ ] Failure modes documented? (network partition, database unavailable, dependency timeout)
- [ ] Incident response plan exists? (escalation, pages, communication)
- [ ] Recovery point objective (RPO) and recovery time objective (RTO) defined?

**Scoring Criteria**:
- **GREEN**: SLO defined, error budget tracked, major failure modes known, incident response playbook ready
- **YELLOW**: SLO loosely defined or untested, some failure modes unknown, partial incident response plan
- **RED**: No SLO, no error budget, failure modes unknown, no incident response process

**Prompt for Participant**:
> "Let's talk about reliability. What's your SLO target? How often does the service currently fail? Do you have a documented list of ways this service could fail, and what you'd do if it happens?"

---

### Substep 2B: Observability Review

**Skill Reference**: 5.07 Observability & Alerting

Evaluate:
- [ ] Structured logging implemented? (JSON, fields, correlation IDs)
- [ ] Key metrics instrumented? (request rate, error rate, latency percentiles)
- [ ] Distributed tracing in place? (e.g., Jaeger, Datadog APM)
- [ ] Dashboards created for operational visibility?
- [ ] Alerts configured for key thresholds?
- [ ] Log retention and searchability adequate?

**Scoring Criteria**:
- **GREEN**: Comprehensive logging, metrics, tracing; dashboards and alerts operational
- **YELLOW**: Basic logging/metrics present; some blind spots in visibility
- **RED**: Minimal/no logging; blind spots everywhere; alert fatigue or silence

**Prompt for Participant**:
> "If something goes wrong at 3 AM, how would you find out? What would you look at? Do you have dashboards? Can you search logs?"

---

### Substep 2C: Performance Review

**Skill Reference**: 5.08 Performance & Optimization

Evaluate:
- [ ] Performance requirements documented? (P50, P95, P99 latency targets)
- [ ] Load testing performed? Against what SLA targets?
- [ ] Database query performance optimized? (n+1 queries, missing indexes)
- [ ] Caching strategy in place? (Redis, CDN, browser cache)
- [ ] Database connection pooling configured?
- [ ] Cold start latency acceptable?
- [ ] Scalability tested under expected peak load?

**Scoring Criteria**:
- **GREEN**: Load tested; latency targets understood and met; optimization done; scalability proven
- **YELLOW**: Some load testing done; latency acceptable but not proven at peak; optimization opportunities exist
- **RED**: No load testing; performance unknown; P99 behavior unpredictable; will degrade under expected load

**Prompt for Participant**:
> "What's the slowest request your service can take? Have you tested how it behaves with 10x more traffic? What happens if a downstream service is slow?"

---

### Substep 2D: Security Review

**Skill Reference**: 5.11 Security & 5.12 Config Management

Evaluate:
- [ ] Authentication & authorization configured? (JWT, OAuth2, RBAC)
- [ ] Secrets management in place? (no hardcoded credentials, encrypted in transit)
- [ ] Input validation and sanitization? (SQL injection, XSS protection)
- [ ] API rate limiting and abuse detection?
- [ ] Encryption in transit (TLS) and at rest?
- [ ] Audit logging for security-relevant events?
- [ ] Security headers and CSP configured?
- [ ] Dependencies scanned for vulnerabilities?

**Scoring Criteria**:
- **GREEN**: Auth/authz implemented, secrets managed, inputs validated, encryption enforced, audit logs present
- **YELLOW**: Most security controls present but some gaps; dependencies not regularly scanned
- **RED**: Missing auth, hardcoded secrets, unvalidated inputs, or no encryption in transit

**Prompt for Participant**:
> "How do you authenticate users? Where do you store secrets? Have you ever checked if your dependencies have known vulnerabilities? What would happen if someone tried to use your API as an attacker?"

---

### Substep 2E: Capacity Review

**Skill Reference**: 5.05 Infrastructure & Deployment

Evaluate:
- [ ] Resource requirements defined? (CPU, memory, disk per instance)
- [ ] Scaling policies in place? (CPU threshold, queue depth)
- [ ] Peak capacity calculated and provisioned?
- [ ] Growth headroom planned? (6-month forecast)
- [ ] Cost projections reasonable and approved?
- [ ] Reserved capacity or spot instances optimized?
- [ ] Database scaling strategy (vertical vs read replicas)?

**Scoring Criteria**:
- **GREEN**: Resource limits known, scaling policies active, peak capacity verified, cost acceptable
- **YELLOW**: Some resource planning done; scaling partially automated; growth forecast vague
- **RED**: No capacity planning; will fail under expected load; cost unknown and potentially excessive

**Prompt for Participant**:
> "How much memory and CPU does each instance need? What happens if you get 10x traffic? Do you auto-scale? What will it cost?"

---

### Substep 2F: Data Review

**Skill Reference**: 5.14 Data Governance & Storage

Evaluate:
- [ ] Schema versioning strategy? (migrations tested)
- [ ] Backup strategy documented? (frequency, retention, tested restores)
- [ ] Data retention policy defined? (what data, how long, compliance)
- [ ] PII handling compliant? (encryption, access control, audit logging)
- [ ] Data consistency strategy? (ACID vs eventual consistency documented)
- [ ] Disaster recovery tested? (backup restore time acceptable)

**Scoring Criteria**:
- **GREEN**: Schema versioning automated, backups tested, retention policy clear, PII secured, DR plan tested
- **YELLOW**: Backups exist but not regularly tested; retention policy partial; PII mostly secured
- **RED**: No backup/restore plan; data loss possible; compliance concerns; DR untested

**Prompt for Participant**:
> "If your database is corrupted right now, how long would it take to recover? Have you tested that? How do you handle PII? What data needs to be retained and for how long?"

---

### Substep 2G: Runbooks Review

**Skill Reference**: 5.06 Operations & Runbooks

Evaluate:
- [ ] Incident runbook exists? (symptoms → triage → resolution steps)
- [ ] Deployment runbook exists and tested?
- [ ] Rollback runbook documented and practiced?
- [ ] Troubleshooting guide for common issues?
- [ ] On-call handoff and context transfer documented?
- [ ] Runbooks regularly reviewed and updated?

**Scoring Criteria**:
- **GREEN**: Comprehensive runbooks for incidents/deployment/rollback; regularly reviewed; practiced
- **YELLOW**: Some runbooks exist; gaps in coverage; not recently tested
- **RED**: No runbooks; operational knowledge tribal; unclear escalation path

**Prompt for Participant**:
> "If this service woke up on-call at 3 AM with alerts, what would they do? Is that documented? Have they practiced it?"

---

### Substep 2H: Dependencies Review

**Skill Reference**: 5.09 External Systems & Resilience

Evaluate:
- [ ] External dependencies listed and versioned?
- [ ] SLA expectations from each dependency understood?
- [ ] Fallback/degradation strategy for each critical dependency?
- [ ] Circuit breaker patterns implemented?
- [ ] Timeout and retry logic configured?
- [ ] Version pinning and compatibility tested?
- [ ] Dependency updates tested before production?

**Scoring Criteria**:
- **GREEN**: Dependencies mapped, SLAs understood, fallbacks designed, circuit breakers active
- **YELLOW**: Dependencies mostly identified; fallbacks missing for some; retry logic basic
- **RED**: Dependencies unclear; no fallback strategy; breaks on dependency upgrade; no timeouts

**Prompt for Participant**:
> "What external services does this depend on? What happens if they go down? Do you have a fallback? What if they're slow—do you timeout?"

---

### Substep 2I: Rollback Review

**Skill Reference**: 5.03 Deployment Patterns & Canary

Evaluate:
- [ ] Rollback strategy documented? (full vs partial, time to rollback)
- [ ] Feature flags or blue-green deployment in use?
- [ ] Canary deployment with monitoring?
- [ ] Smoke tests defined and run pre-deployment?
- [ ] Database migration reversibility tested?
- [ ] Rollback time acceptable? (RTO < acceptable window)

**Scoring Criteria**:
- **GREEN**: Multiple rollback mechanisms (feature flags, blue-green, canary); smoke tests; tested regularly
- **YELLOW**: Rollback possible but manual; database migrations may be hard to reverse
- **RED**: No rollback strategy; breaking changes possible; database migrations non-reversible; high RTO

**Prompt for Participant**:
> "If we deploy something that breaks, how long would it take to rollback? Can we do it instantly or is it 30 minutes? Is the database change reversible?"

---

## Scoring Summary

Document all findings in the scorecard format:

| Dimension | Score | Summary | P0 Blockers | P1 Risks | Recommendations |
|-----------|-------|---------|------------|---------|-----------------|
| Reliability | GREEN/YELLOW/RED | ... | ... | ... | ... |
| Observability | GREEN/YELLOW/RED | ... | ... | ... | ... |
| Performance | GREEN/YELLOW/RED | ... | ... | ... | ... |
| Security | GREEN/YELLOW/RED | ... | ... | ... | ... |
| Capacity | GREEN/YELLOW/RED | ... | ... | ... | ... |
| Data | GREEN/YELLOW/RED | ... | ... | ... | ... |
| Runbooks | GREEN/YELLOW/RED | ... | ... | ... | ... |
| Dependencies | GREEN/YELLOW/RED | ... | ... | ... | ... |
| Rollback | GREEN/YELLOW/RED | ... | ... | ... | ... |

## Transition to Step 03

Once all 9 dimensions scored:

```
"Excellent! I've completed the technical deep dive.

Here's what I found:
- [Number] GREENs (ready to go)
- [Number] YELLOWs (manageable risks)
- [Number] REDs (blockers to address)

Let's move to Step 03 where I'll make the final GO/NO-GO/CONDITIONAL-GO recommendation
and we'll prioritize action items for any gaps."
```

**Proceed to [Step 03](step-03-final-decision.md)**

---

**Navigation**: [← Back to Step 01](step-01-init-checklist.md), [Next: Step 03 →](step-03-final-decision.md)
