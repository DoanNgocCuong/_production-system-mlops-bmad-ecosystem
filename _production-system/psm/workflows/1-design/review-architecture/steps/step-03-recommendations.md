---
step_id: RA-S03
step_name: Recommendations
workflow_id: RA001
sequence: 3
type: "recommendation"
agent: "Khang (Architect)"
description: "Generate ADRs, trade-off analysis, action plan"
estimated_time: "60-75 minutes"
is_final_step: true
outputFile: "{project-root}/psm/outputs/review-architecture/architecture-review-report.md"
---

# Step 03: Architecture Recommendations & Action Plan

## Objective

Synthesize analysis into actionable recommendations:
1. Generate Architecture Decision Records (ADRs)
2. Analyze trade-offs for key improvements
3. Create prioritized action plan
4. Align team on next quarter's work

## BMAD v6.2 Checkpoint Protocol

**Before proceeding, ensure output document exists with YAML frontmatter:**
```yaml
---
stepsCompleted: ["step-01-context.md", "step-02-analysis.md"]
lastStep: "step-03-recommendations.md"
lastSaved: "{current-timestamp}"
workflowType: "review-architecture"
---
```

**SAVING RULE:** After completing all execution sections, BEFORE loading the next step:
1. Update output document: append current step's output to the document
2. Update frontmatter: add this step to `stepsCompleted`, update `lastStep` and `lastSaved`
3. Save the document

---

## Execution Flow

### Substep 3A: Generate Architecture Decision Records (ADRs)

For each major improvement, create an ADR in standard format:

**ADR Template**:
```markdown
# ADR-XXX: [Decision Title]

**Date**: 2026-03-17
**Status**: Proposed (not yet decided)
**Context**: [Why is this decision needed?]
**Problem**: [What's the issue we're solving?]

## Proposed Solution

[Describe the solution in detail]

## Trade-offs

| Aspect | Pro | Con |
|--------|-----|-----|
| Complexity | ... | ... |
| Cost | ... | ... |
| Performance | ... | ... |
| Maintenance | ... | ... |

## Alternatives Considered

1. [Option A] — Pro/con summary
2. [Option B] — Pro/con summary
3. [Option C] — Pro/con summary

## Recommendation

**Choose**: [Best option] because [rationale]

## Implementation Plan

- Effort: [X days/weeks]
- Risk: [Low/Medium/High] and why
- Rollback Strategy: [How to undo if needed]
- Dependencies: [Other work that must complete first]

## Success Criteria

- [ ] Acceptance test 1
- [ ] Acceptance test 2
- [ ] Performance benchmark
- [ ] No production incidents during rollout
```

**Example ADRs**:

### ADR-001: Implement Database Read Replicas

```markdown
# ADR-001: Implement Database Read Replicas

**Status**: Proposed
**Context**: Database CPU at 70% at current 500 RPS, approaching limit for 10x growth
**Problem**: Single PostgreSQL instance becomes bottleneck; cannot scale read-heavy queries

## Proposed Solution

Deploy 2 read replicas of PostgreSQL:
- Master handles writes
- Replicas handle reads (non-critical data)
- Connection pooling (PgBouncer) distributes traffic
- Replica lag monitoring (must be < 1s)

## Trade-offs

| Aspect | Pro | Con |
|--------|-----|-----|
| Scalability | Read throughput 3x | Replication lag introduces staleness |
| Cost | Replicas cost $700/month | Total DB cost increases 35% |
| Complexity | Read scaling simpler than sharding | Replica lag management needed |
| Maintenance | Standard PostgreSQL setup | More instances to monitor |

## Alternatives

1. **Vertical scaling** (bigger instance) — Easier now, limits future growth
2. **Database sharding** — Better long-term, requires app changes
3. **CQRS pattern** (separate read DB) — Overkill for current scale

## Recommendation

**Choose**: Read replicas because:
- Solves immediate scaling problem (allows 3x growth)
- Lower risk than sharding (no app changes)
- Standard PostgreSQL approach (team familiar)
- Cost reasonable ($700/month)

## Implementation Plan

**Effort**: 1 week
- 2 days: Setup replicas, test failover
- 2 days: Implement read routing in app
- 1 day: Monitor and validate

**Risk**: Medium
- Replication lag could expose stale data
- Mitigation: Read-after-write consistency for critical data

**Rollback**: Can disable read routing in 1 hour if issues

**Dependencies**: None (can start immediately)

## Success Criteria

- [ ] Replicas replicating with < 1s lag
- [ ] Read queries distributed across replicas
- [ ] Non-critical data consistent (user profiles, inventory)
- [ ] Stale data acceptable for critical reads (order history)
- [ ] Alerts for replica lag > 5s
```

---

### ADR-002: Implement Circuit Breaker for Payment Service

```markdown
# ADR-002: Implement Circuit Breaker for Payment Service

**Status**: Proposed
**Context**: Payment service failure causes order service timeout (cascading failure)
**Problem**: No protection against slow/failing payment service; affects user experience

## Proposed Solution

Implement circuit breaker pattern:
- Success threshold: 95% success rate
- Failure threshold: 3 consecutive failures or error rate > 5%
- Open state (fail-fast): Return error immediately, inform user
- Half-open state: Test with 10% traffic
- Closed state (normal): Pass through all traffic

## Implementation

Use proven library (e.g., Resilience4j, go-resilience):
```
Order Service
  └─ Circuit Breaker (around payment call)
      └─ Payment Service
```

## Trade-offs

| Aspect | Pro | Con |
|--------|-----|-----|
| Reliability | Fail-fast instead of cascade | Users see payment failures |
| Latency | Faster failure response | Adds 1-2ms latency in happy path |
| Complexity | Simple library to integrate | Adds error handling logic |
| User Experience | Immediate feedback | Need retry/queue mechanism |

## Alternatives

1. **Timeout only** — Simple but doesn't prevent cascading
2. **Bulkheads** (thread pool limit) — Complements circuit breaker
3. **Async payment + queue** — Better long-term, bigger change

## Recommendation

**Choose**: Circuit breaker (first), queue payment async (next quarter)

## Implementation Plan

**Effort**: 2-3 days
- 1 day: Add circuit breaker library
- 1 day: Implement and test
- 0.5 day: Deploy and monitor

**Risk**: Low (library well-tested, doesn't change payment logic)

**Rollback**: Remove circuit breaker annotation, redeploy (1 minute)

**Dependencies**: None

## Success Criteria

- [ ] Circuit breaker fails fast (< 10ms) when payment unavailable
- [ ] No cascading timeouts in order service
- [ ] Alerts trigger when circuit opens
- [ ] Circuit recovers automatically when payment healthy
```

---

### Substep 3B: Prioritized Action Plan

Organize recommendations by impact/effort:

```markdown
## Quarterly Action Plan (Next 3 Months)

### Week 1-2: Quick Wins (Cost Savings)

**[R1] Consolidate Kubernetes Nodes**
- Impact: -$1,500/month (18% savings)
- Effort: 3 days
- Owner: DevOps Lead
- Risk: Low (Kubernetes handles migration)
- Status: Proposed → In Progress

**[R2] Reduce RDS Backup Retention**
- Impact: -$200/month (2% savings)
- Effort: 1 hour
- Owner: DBA
- Risk: Low (only changes retention policy)
- Status: Proposed → In Progress

---

### Week 3-4: Foundation Work (Reliability)

**[R3] Implement Circuit Breaker (Payment)**
- Impact: Prevents cascading failures
- Effort: 2-3 days
- Owner: Backend Lead
- Risk: Low
- Status: Proposed

**[R4] Add Missing Runbooks**
- Payment service failure scenario
- Cache failure scenario
- Database replica lag scenario
- Impact: Faster incident resolution (MTTR: 45min → 15min)
- Effort: 2 days
- Owner: SRE Lead
- Risk: None (documentation only)
- Status: Proposed

---

### Week 5-6: Scaling Foundation (Scalability)

**[R5] Implement Database Read Replicas**
- Impact: Allows 3x read scaling
- Effort: 1 week
- Owner: DBA + Backend Lead
- Risk: Medium (replication lag management)
- Status: Proposed
- Dependencies: [R4] (need monitoring/runbooks first)

**[R6] Implement Redis Cluster**
- Impact: 3x cache capacity, high availability
- Effort: 1 week
- Owner: DevOps Lead
- Risk: Medium (cluster failover testing needed)
- Status: Proposed
- Dependencies: None

---

### Week 7-8: Developer Experience

**[R7] Document API Contracts (OpenAPI)**
- Impact: Faster onboarding, easier integration testing
- Effort: 3-4 days
- Owner: Backend Lead + Tech Writer
- Risk: None (documentation)
- Status: Proposed

**[R8] Improve Test Coverage**
- Current: ~55% (Payment 60%, Order 45%, Auth 70%)
- Target: 70%
- Effort: 1.5 weeks
- Owner: Full backend team
- Risk: None (testing adds safety)
- Status: Proposed

---

### Week 9-12: Long-term Scaling (Deferred to Q2)

**[R9] Consider Async Payment Processing**
- Impact: Decouples order from payment, better scaling
- Effort: 2-3 weeks
- Owner: Backend + Platform team
- Risk: Medium (adds complexity, eventual consistency)
- Status: Deferred to Q2 (depends on traffic growth)

**[R10] Database Sharding Strategy**
- Impact: Enables 10x+ growth
- Effort: 4-6 weeks
- Owner: DBA + Backend team
- Risk: High (significant app changes)
- Status: Deferred to Q3 (only if needed)

---

## Effort vs Impact Summary

```
        HIGH IMPACT
             ▲
             |
             |  [R3] Circuit Breaker
             |  [R4] Runbooks        [R5] Read Replicas
             |                      [R6] Redis Cluster
             |
             |  [R7] API Docs
   [R8] Tests|          [R2] Backup Retention
             |  [R1] K8s Consolidation
             |
             |_________________________________________________________► EFFORT
        LOW    LOW                                                HIGH
```

**Quick Wins** (low effort, good impact):
- [R1] K8s consolidation (3 days → -$1,500/month)
- [R2] Backup retention (1 hour → -$200/month)
- [R4] Runbooks (2 days → MTTR improvement)

**Medium-term** (1-2 weeks, significant impact):
- [R3] Circuit breaker (3 days → reliability improvement)
- [R5] Read replicas (1 week → 3x read scaling)
- [R6] Redis cluster (1 week → cache scaling)
- [R7] API docs (4 days → developer experience)

**Long-term** (4+ weeks, high complexity):
- [R8] Test coverage (ongoing)
- [R9] Async payment (Q2)
- [R10] Database sharding (Q3)
```

---

### Substep 3C: Roadmap & Timeline

Create visual roadmap:

```markdown
## 12-Week Architecture Roadmap

### Q1 2026 (Next 12 Weeks)

**Month 1: Quick Wins + Foundation**
- Week 1-2: Cost optimizations (K8s, backups)
- Week 3: Circuit breaker implementation
- Week 4: Runbooks documentation

**Month 2: Scaling Foundation**
- Week 5-6: Read replicas (DB scaling)
- Week 6-7: Redis cluster (cache scaling)
- Week 8: Integration testing

**Month 3: Developer Experience**
- Week 9: API contract documentation
- Week 10-12: Test coverage improvements

### Q2 2026 (If Needed)

- Async payment processing (if growth continues)
- Further observability improvements
- Cost optimization round 2

### Q3 2026+ (Long-term)

- Database sharding (if 10x growth needed)
- CQRS pattern (if read/write ratio becomes extreme)
- Microservice mesh (if inter-service complexity high)
```

---

### Substep 3D: Risk Mitigation

Address potential risks:

```markdown
## Implementation Risks & Mitigations

### Risk 1: Read Replica Lag Causes Data Consistency Issues

**Mitigation**:
- Implement read-after-write consistency
- Use master for critical paths (order confirmation)
- Use replicas for non-critical (user profiles, analytics)
- Monitor lag continuously (alert if > 5s)
- Have runbook for replication lag scenario

---

### Risk 2: Circuit Breaker Causes User Experience Degradation

**Mitigation**:
- Implement graceful degradation (queue payment for later)
- Show user: "Payment processing, will confirm shortly"
- Queue has SLA (process within 1 hour)
- Automatic retry with exponential backoff
- Monitor queue depth

---

### Risk 3: Redis Cluster Failover Issues

**Mitigation**:
- Test failover scenario monthly
- Implement fallback to single-node mode
- Have runbook for cluster recovery
- Monitor replica sync lag
- Alert on cluster health status changes

---

### Risk 4: Database Load Tests Cause Production Impact

**Mitigation**:
- Run tests in staging environment first
- Use production data sample (sanitized)
- Schedule tests during low-traffic hours
- Have rollback plan (revert to single instance)
```

---

### Substep 3E: Success Metrics

Define how to measure success:

```markdown
## Post-Implementation Success Metrics

### Scalability Metrics
- [ ] Database query latency stable at < 50ms p99 (current: 100ms)
- [ ] Cache hit rate > 85% (current: 80%)
- [ ] Can handle 2x traffic without changes (current limit: 1.5x)
- [ ] Load testing confirms 3x throughput capacity

### Reliability Metrics
- [ ] Incidents per month: 3 → 1 (by end of Q1)
- [ ] MTTR: 45 min → 15 min (by week 4)
- [ ] On-call pager frequency: 2-3x/week → < 1x/week

### Cost Metrics
- [ ] Monthly spend: $8,500 → $6,000 (quick wins)
- [ ] Cost per user: $1.70 → $1.20 per DAU
- [ ] Cloud bill trend: steady (not growing with traffic)

### Developer Experience
- [ ] Onboarding time: 2 weeks → 1 week
- [ ] Test coverage: 55% → 75%
- [ ] API documentation: Available (OpenAPI spec)

### Business Metrics
- [ ] Feature development velocity: unblocked by infrastructure
- [ ] User growth accommodated without new infrastructure
- [ ] Team satisfaction: Reduced on-call burden
```

---

## Deliverables from This Step

**Output**: Architecture Review Report (use `architecture-review.template.md`)

1. **Executive Summary** (1 page)
   - Dimensional scorecard (5 dimensions)
   - Key findings (strengths and gaps)
   - Recommended action plan

2. **Dimensional Analysis** (per-dimension details)
   - Scalability assessment
   - Reliability assessment
   - Maintainability assessment
   - Operational complexity
   - Cost efficiency

3. **Architecture Decision Records** (3-5 ADRs)
   - Problem statement
   - Proposed solutions with trade-offs
   - Recommendation and rationale
   - Implementation plan

4. **Action Plan** (12-week roadmap)
   - Prioritized recommendations
   - Effort estimates
   - Risk assessments
   - Success metrics

5. **Sign-offs**
   - Architecture Lead approval
   - Team lead acknowledgments
   - Timeline agreement

---

## Workflow Complete

This is the **FINAL STEP** of the Review Architecture workflow.

```
"Architecture review complete!

Summary:
- System is well-designed at current scale
- Can support 2-3x growth with planned improvements
- 10x growth requires sharding (deferred to Q3)
- Quick wins available: $1,700/month savings + operational improvements

Recommended Focus (Next 3 Months):
1. Cost optimizations (quick wins)
2. Reliability hardening (circuit breaker, runbooks)
3. Scalability foundation (read replicas, redis cluster)
4. Developer experience (docs, tests)

Timeline: 12 weeks, achievable with current team.

Questions on recommendations or timeline?"
```

**Next Steps**:
- Refine ADRs with team feedback
- Prioritize backlog items for next sprint
- Assign work to team members
- Track progress in next quarterly review

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

This is the **FINAL STEP** of the Review Architecture workflow.

```
"Architecture review complete!

Summary:
- System is well-designed at current scale
- Can support 2-3x growth with planned improvements
- 10x growth requires sharding (deferred to Q3)
- Quick wins available: $1,700/month savings + operational improvements

Recommended Focus (Next 3 Months):
1. Cost optimizations (quick wins)
2. Reliability hardening (circuit breaker, runbooks)
3. Scalability foundation (read replicas, redis cluster)
4. Developer experience (docs, tests)

Timeline: 12 weeks, achievable with current team.

Questions on recommendations or timeline?"
```

**Next Steps**:
- Refine ADRs with team feedback
- Prioritize backlog items for next sprint
- Assign work to team members
- Track progress in next quarterly review
