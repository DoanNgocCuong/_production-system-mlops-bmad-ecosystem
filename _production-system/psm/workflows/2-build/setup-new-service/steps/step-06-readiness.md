---
step_number: 6
step_name: "Production Readiness Review & Go/No-Go Decision"
next_step_file: null
estimated_time: "1.5-2 hours"
lead_agent: "SRE Minh"
supporting_agents: ["Architect Khang", "Mary Analyst"]
skill_loads:
  - "references/5.10 - RUN - Production Readiness/SKILL.md"
  - "references/5.14 - CROSS - Documentation & Runbooks/SKILL.md"
is_final_step: true
outputFile: "{project-root}/psm/outputs/setup-new-service/{service_name}/production-readiness-report.md"
workflowType: "setup-new-service"
---

# STEP 6: Production Readiness Review & Go/No-Go Decision

## STEP GOAL

Comprehensive production readiness assessment and final go/no-go decision:

- **PRR Checklist**: 50+ criteria covering all dimensions
- **Runbook creation**: Operational guides for common scenarios
- **Incident response plan**: Escalation, mitigation, communication
- **Team readiness**: On-call rotation, training, documentation
- **Final decision**: Go to production or identify blockers

**Output**: `production-readiness-report.md` + `runbook.md` + `incident-response-guide.md`

**FINAL STEP** — Ready for production deployment

## Skill Files to Load

```
Load: references/5.10 - RUN - Production Readiness/SKILL.md
      → Production readiness checklist, launch criteria

Load: references/5.14 - CROSS - Documentation & Runbooks/SKILL.md
      → Documentation standards, runbook templates, knowledge transfer
```

## MANDATORY RULES

1. **Checklist driven** — All 50+ items must be evaluated
2. **Evidence required** — Each item needs documented evidence
3. **No assumptions** — Everything must be tested and verified
4. **Team aligned** — Go/no-go decision is team consensus
5. **Fallback ready** — Rollback plan must exist before launch

## BMAD v6.2 Checkpoint Protocol

**Before proceeding, ensure output document exists with YAML frontmatter:**
```yaml
---
stepsCompleted: ["step-01-architecture.md", "step-02-api-database.md", "step-03-build-deploy.md", "step-04-reliability.md", "step-05-security-infra.md"]
lastStep: "step-06-readiness.md"
lastSaved: "{current-timestamp}"
workflowType: "setup-new-service"
---
```

**SAVING RULE:** After completing all execution sections, BEFORE loading the next step:
1. Update output document: append current step's output to the document
2. Update frontmatter: add this step to `stepsCompleted`, update `lastStep` and `lastSaved`
3. Save the document

---

## EXECUTION PROTOCOL

### Phase 1: Production Readiness Checklist (40 min)

**Architecture & Design (10 items)**

- [ ] Architecture design document reviewed and approved
- [ ] Service boundaries clearly defined and documented
- [ ] Technology choices justified and aligned with team skills
- [ ] Scalability analysis completed (10x growth plan)
- [ ] Database schema reviewed and normalized
- [ ] API contracts defined with OpenAPI/GraphQL schema
- [ ] Data flow diagram created and reviewed
- [ ] Single points of failure identified and mitigated
- [ ] Failover strategy documented
- [ ] High availability architecture confirmed

**Development & Code Quality (8 items)**

- [ ] Code review process defined and in place
- [ ] All code is type-checked (TypeScript, mypy, etc.)
- [ ] Linting passes with zero errors
- [ ] Test coverage >= 75% for critical paths
- [ ] No hardcoded secrets in codebase
- [ ] Security dependencies up to date
- [ ] Code follows team style guide
- [ ] Documentation for complex logic exists

**Testing & QA (10 items)**

- [ ] Unit test suite passes 100%
- [ ] Integration test suite passes 100%
- [ ] E2E tests for critical user flows pass
- [ ] Load testing completed (peak load + 30%)
- [ ] Chaos engineering tests executed
- [ ] Failure scenario testing completed
- [ ] Database migration tested with rollback
- [ ] Performance benchmarks established
- [ ] Security testing completed (OWASP top 10)
- [ ] Accessibility testing passed (if applicable)

**Deployment & Infrastructure (10 items)**

- [ ] Docker image builds and runs locally
- [ ] Docker image scanned for vulnerabilities
- [ ] Kubernetes manifests validated with kubeval
- [ ] Helm charts tested in staging environment
- [ ] Blue-green deployment strategy documented
- [ ] Canary deployment process defined
- [ ] Rollback process tested
- [ ] DNS records prepared (but not activated)
- [ ] SSL/TLS certificates configured
- [ ] Load balancer configuration verified

**Reliability & Observability (10 items)**

- [ ] All services emit structured logs
- [ ] Correlation IDs implemented across services
- [ ] Metrics collection verified (Prometheus scraping)
- [ ] Alert rules configured and tested
- [ ] Dashboard created in Grafana
- [ ] Distributed tracing verified (OpenTelemetry)
- [ ] Circuit breakers implemented and tested
- [ ] Retry logic with backoff verified
- [ ] Timeout policies configured
- [ ] Fallback mechanisms tested

**Security & Compliance (8 items)**

- [ ] Authentication implemented and tested
- [ ] Authorization (RBAC) configured
- [ ] Secrets management in place (Vault/AWS Secrets)
- [ ] Secret rotation automated
- [ ] TLS 1.2+ enforced for all connections
- [ ] Network policies configured
- [ ] Pod security policies applied
- [ ] Compliance requirements verified (SOC2/HIPAA/GDPR if needed)

**Operations & Support (6 items)**

- [ ] Runbook created for common scenarios
- [ ] Escalation procedures documented
- [ ] On-call rotation established
- [ ] Team trained on troubleshooting
- [ ] Incident response playbook created
- [ ] Communication plan for incidents established

**Documentation (8 items)**

- [ ] Architecture documentation complete
- [ ] API documentation generated and tested
- [ ] Database schema documentation complete
- [ ] Deployment guide written
- [ ] Troubleshooting guide created
- [ ] FAQ created from team questions
- [ ] Training materials prepared for team
- [ ] Links to all documentation centralized

**Pre-Launch Testing (10 items)**

- [ ] Staging environment mirrors production
- [ ] Staging deployment successfully completed
- [ ] Smoke tests pass in staging
- [ ] Customer acceptance testing completed
- [ ] Performance test in staging passes
- [ ] Security scan of staging passed
- [ ] Data migration tested (if applicable)
- [ ] Backup/restore procedures tested
- [ ] Monitoring dashboard functional
- [ ] Incident response drills completed

### Phase 2: Runbook Creation (30 min)

**Runbook Template**:

```markdown
# Runbook: [Service Name]

## Overview
- **Service**: Order Processing API
- **Owner**: SRE Minh
- **On-Call**: See rotation at [wiki/oncall]
- **Dependencies**: Payment Gateway, Inventory Service, PostgreSQL

## Common Scenarios

### Scenario 1: High Error Rate Alert

**Symptoms**: Alert: HighErrorRate fired
- Error rate > 1% for 5 minutes
- Customer complaints about failures

**Investigation**:
1. Check CloudWatch logs for error patterns
   ```bash
   kubectl logs -f deployment/api-service -n production
   ```
2. Check recent deployments
   ```bash
   kubectl rollout history deployment/api-service -n production
   ```
3. Check metrics in Grafana
   - Dashboard: [link]
   - Look for: Error rate spike, specific endpoint failures

**Causes & Fixes**:

| Cause | Check Command | Fix |
|-------|---------------|-----|
| Dependent service down | `curl https://payment-api/health` | Page on-call engineer for that service |
| Database connection pool exhausted | `SELECT count(*) FROM pg_stat_activity;` | Increase pool size or rollback recent changes |
| Memory leak / high memory usage | `kubectl top pod` | Rolling restart pods |
| Recent bad deployment | `kubectl rollout history` | `kubectl rollout undo` |

**Escalation**:
- If issue persists > 10 min: Page service owner
- If issue persists > 30 min: Page engineering lead
- If affecting production data: Page CTO

### Scenario 2: Service Pod Keeps Crashing (CrashLoopBackOff)

**Investigation**:
```bash
# Check pod logs
kubectl logs <pod-name> -n production --previous

# Check events
kubectl describe pod <pod-name> -n production

# Check resource usage
kubectl top pod <pod-name> -n production
```

**Common Causes**:

| Cause | Evidence | Fix |
|-------|----------|-----|
| Out of Memory | `OOMKilled` in events | Increase memory limit |
| Missing secrets | `KeyError` in logs | Verify secrets exist in Vault |
| Bad config | `ValueError` in logs | Verify ConfigMap values |
| Health check failing | `liveness probe failed` | Check service is actually ready |

**Fix Process**:
1. Check Vault/Secrets are accessible
2. Verify ConfigMaps are correct
3. Increase resource limits if needed
4. Restart pod: `kubectl rollout restart deployment/api-service`

### Scenario 3: High Latency (p99 > 500ms)

**Investigation**:
```bash
# Check database query performance
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 'xyz';

# Check service metrics
- Grafana dashboard: Look at http_request_duration histogram
- Check if specific endpoints are slow
- Check for heavy background jobs running
```

**Causes & Fixes**:

| Cause | Check | Fix |
|-------|-------|-----|
| Missing database index | EXPLAIN output shows Seq Scan | Add index: `CREATE INDEX idx_user_id ON orders(user_id);` |
| External API timeout | Trace shows long calls to dependency | Page service owner, check their status page |
| High CPU usage | `kubectl top node` | Scale to more nodes or optimize queries |
| GC pauses | Check JVM logs (if applicable) | Tune GC settings or increase heap |

### Scenario 4: Data Integrity Issue Discovered

**CRITICAL - Follow Incident Response Plan**

**Steps**:
1. STOP: Don't make changes without approval
2. Page service owner immediately
3. Page DBA for database investigation
4. Document the issue: Who found it? When? What data?
5. Assess blast radius: How many customers affected?
6. Plan fix: Rollback, hotfix, or data correction?

**Communication**:
- Update status page immediately (even if "investigating")
- Notify customer success team
- Prepare customer communication

---

## Alert Response Matrix

| Alert | Severity | Immediate Action | Owner |
|-------|----------|------------------|-------|
| HighErrorRate | Critical | Check logs, consider rollback | SRE |
| HighLatency | Warning | Check databases, scale if needed | SRE |
| CircuitBreakerOpen | Warning | Check dependent service | SRE |
| PodCrashing | Critical | Check logs, rollback recent change | SRE |
| DiskFull | Critical | Scale storage or cleanup | DevOps |

## Escalation Contacts

- **SRE On-Call**: [Phone] [Slack] (pages every 1 week)
- **Service Owner**: [Name] (main decision maker)
- **Engineering Lead**: [Name] (technical escalation)
- **VP Engineering**: [Name] (business escalation)
- **CTO**: [Name] (critical incidents)

## Reference Documentation

- Architecture: [Link]
- API Docs: [Link]
- Database Schema: [Link]
- Monitoring: [Grafana URL]
```

### Phase 3: Incident Response Plan (25 min)

**Incident Severity Classification**:

```
SEV 0 - Critical
- Data loss or corruption
- > 50% of users affected
- Revenue-impacting outage
- Requires CTO notification

SEV 1 - High
- > 10% of users affected
- Service completely unavailable
- Requires service owner paging

SEV 2 - Medium
- < 10% of users affected
- Degraded performance
- Requires SRE acknowledgment

SEV 3 - Low
- < 1% affected or cosmetic issue
- Ticket creation sufficient
```

**Incident Response Flow**:

```
1. Detection (Automated alert or customer report)
   ↓
2. Acknowledgment (On-call engineer confirms incident)
   ↓
3. Assessment (Severity classification, scope)
   ↓
4. Notification (Appropriate team pages based on severity)
   ↓
5. Investigation (Root cause analysis, using runbook)
   ↓
6. Mitigation (Rollback, hotfix, workaround)
   ↓
7. Communication (Status page, customer emails, Slack updates)
   ↓
8. Resolution (Issue fully resolved, in production)
   ↓
9. Postmortem (Root cause analysis, action items)
   ↓
10. Prevention (Monitoring alerts, tests added)
```

**Communication Checklist**:

- [ ] Status page updated within 5 minutes
- [ ] Slack notification sent (#incidents channel)
- [ ] Customer emails queued (if > SEV 1)
- [ ] Executive team notified (if > SEV 0)
- [ ] Regular updates every 15 minutes (for SEV 0-1)

### Phase 4: Go/No-Go Decision (15 min)

**Scoring System**:

```
RED: Critical blocker exists (must fix before launch)
- Failing security test
- Data loss risk identified
- Cannot meet compliance requirement

YELLOW: Issue exists but has workaround (can launch with caution)
- Non-critical feature incomplete
- Performance not optimal but acceptable
- Monitoring gap exists but documented

GREEN: Clear to launch
- All critical items checked
- Team confidence high
- Rollback plan ready
```

**Go/No-Go Criteria**:

✓ ALL RED items must be GREEN
✓ Majority of YELLOW items have documented mitigations
✓ Runbook completed and reviewed
✓ Team has conducted incident response drill
✓ Communication plan approved
✓ Rollback plan tested

**Final Decision Form**:

```
================================================================================
PRODUCTION READINESS REVIEW — FINAL DECISION
================================================================================

Service: [Name]
Review Date: [Date]
Reviewed By: [Team members]

PRR Checklist Completion: ___/50 items (must be >= 48)

Critical Blockers: ___  (must be 0)

Go/No-Go Vote:
- SRE Lead: [GO / NO-GO] — [Signature / Reason]
- Architecture: [GO / NO-GO] — [Signature / Reason]
- Product Lead: [GO / NO-GO] — [Signature / Reason]

FINAL DECISION: [GO TO PRODUCTION / NO-GO - BLOCKERS: list]

Deployment Window Scheduled: [Date/Time]

Rollback Trigger: [What condition would trigger rollback?]

Post-Launch Monitoring Plan: [24hr no-issue period? Key metrics to watch?]

Sign-off: ________________________  Date: __________

================================================================================
```

---

## Checkpoint

**Select an Option:**
**[C] Complete Workflow**
**[A] Advanced Elicitation**
**[P] Party Mode**
**[Y] YOLO**

Menu Handling:
- IF C: Update frontmatter (add this step to stepsCompleted, update lastStep, lastSaved), save document, then present Workflow Complete summary
- IF A: Perform advanced elicitation on current step output
- IF P: Invite other agents to party mode
- IF Y: Proceed without stopping
- IF Any other: respond and redisplay menu

**EXECUTION RULES:**
- 🛑 ALWAYS halt and wait for user input after presenting checkpoint menu
- 🛑 NEVER present "Workflow Complete" until user selects 'C'
- 🛑 ALWAYS update frontmatter BEFORE presenting completion
- 📖 Read the entire output document before completion summary

---

## Workflow Complete

This is the **FINAL STEP** of the Setup New Service workflow.

```
"Congratulations! Service is production-ready.

All output documents are available in: ./outputs/setup-new-service-{service_name}/

Workflow Summary:
- Step 01: Architecture & Requirements — COMPLETE
- Step 02: API & Database Design — COMPLETE
- Step 03: Build & Deployment Pipeline — COMPLETE
- Step 04: Reliability & Observability — COMPLETE
- Step 05: Security & Infrastructure — COMPLETE
- Step 06: Production Readiness Review — COMPLETE

Go/No-Go Decision: [APPROVED FOR PRODUCTION]

Next steps:
1. Archive this workflow
2. Schedule service reviews (quarterly)
3. Begin collecting production metrics
4. Plan ongoing improvements based on learnings"
```

**Next Steps**:
1. Archive this workflow
2. Schedule service reviews (quarterly)
3. Begin collecting production metrics
4. Plan ongoing improvements based on learnings
