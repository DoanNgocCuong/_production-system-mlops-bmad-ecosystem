---
step_id: RA-S02
step_name: Analysis
workflow_id: RA001
sequence: 2
type: "analysis"
agent: "Khang (Architect)"
description: "Analyze architecture against 5 dimensions, score, identify gaps"
estimated_time: "90-120 minutes"
is_final_step: false
outputFile: "{project-root}/psm/outputs/review-architecture/architecture-review-report.md"
---

# Step 02: Dimensional Architecture Analysis

## Objective

Systematically evaluate architecture across 5 key dimensions:
- Scalability
- Reliability
- Maintainability
- Operational Complexity
- Cost Efficiency

## BMAD v6.2 Checkpoint Protocol

**Before proceeding, ensure output document exists with YAML frontmatter:**
```yaml
---
stepsCompleted: ["step-01-context.md"]
lastStep: "step-02-analysis.md"
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

### Dimension 1: Scalability

**Can the system grow 10x?**

Evaluate:
- [ ] Can traffic increase without architectural changes?
- [ ] Stateless services (can add more instances)?
- [ ] Database scaling strategy (sharding, read replicas)?
- [ ] Caching strategy efficient?
- [ ] Message queues for async work?
- [ ] CDN or edge caching?

**Scoring Criteria**:
- **GREEN**: Horizontal scaling automatic; load-testing proven; no bottlenecks identified
- **YELLOW**: Scaling possible but requires manual intervention; some bottlenecks; database scaling planned
- **RED**: Cannot scale 10x without major redesign; single points of failure; bottleneck approaching

**Analysis Example**:
```
Current: 500 RPS
Target (10x): 5,000 RPS

Current Bottleneck Analysis:
├─ API Servers: 3 pods, ~166 RPS each → Can add more pods ✅
├─ Database: Single PostgreSQL, handles 200 RPS comfortably
│  Problem: At 500 RPS, CPU at 70%. At 5K RPS, 700% CPU (not viable)
│  Solution Needed: Read replicas + connection pooling
├─ Redis Cache: Single instance at 90% memory utilization
│  Problem: Cache hits at 85%, misses at 15% cause DB hits
│  Solution Needed: Cache clustering or Memcached farm
└─ Message Queue: Not used; all processing synchronous
   Problem: Slow tasks block user request
   Solution Needed: Async job processing (Kafka, RabbitMQ)

SCORE: YELLOW (Scalable to 2x with connection pooling, needs redesign for 10x)
```

---

### Dimension 2: Reliability

**Can the system survive failures?**

Evaluate:
- [ ] Redundancy (no single points of failure)?
- [ ] Circuit breakers for external calls?
- [ ] Graceful degradation (degrade service vs fail completely)?
- [ ] Automatic failover (database, cache, services)?
- [ ] Incident response runbooks?
- [ ] Error recovery (retry logic, exponential backoff)?

**Scoring Criteria**:
- **GREEN**: Redundancy across all layers; circuit breakers active; graceful degradation; auto-failover tested
- **YELLOW**: Some redundancy; some failure modes mitigated; manual recovery needed for some failures
- **RED**: Single points of failure; no circuit breakers; cascading failures likely

**Analysis Example**:
```
Failure Mode Analysis:

SCENARIO 1: Payment service goes down
├─ Current: Order service blocks on payment → User sees timeout
├─ Problem: Cascading failure; orders stall
├─ Mitigation: Add circuit breaker (fail after 3 timeouts)
├─ Better: Async payment + graceful degradation (try again later)
└─ Score: RED → YELLOW (with circuit breaker) → GREEN (with async)

SCENARIO 2: Redis cache goes down
├─ Current: Cache miss → DB query → response slow but works
├─ Problem: Database load spikes (expected 10x load)
├─ Mitigation: Cache warming, fallback to local cache
└─ Score: YELLOW (degrades gracefully but database at risk)

SCENARIO 3: Database replica lag
├─ Current: Eventual consistency acceptable?
├─ Problem: Stale data shown to users if read from replica
├─ Mitigation: Read-after-write consistency for critical paths
└─ Score: YELLOW (acceptable for non-critical reads)

OVERALL SCORE: YELLOW (Graceful degradation works, but needs circuit breakers)
```

---

### Dimension 3: Maintainability

**Is the code organized and clear?**

Evaluate:
- [ ] Clear service boundaries (responsibility)?
- [ ] Consistent patterns across services?
- [ ] Documentation adequate?
- [ ] Dependency management (no circular deps)?
- [ ] Testing coverage (unit, integration)?
- [ ] Code organization (file structure, naming)?

**Scoring Criteria**:
- **GREEN**: Clear abstractions, consistent patterns, well-documented, > 70% test coverage
- **YELLOW**: Mostly clear, some inconsistencies, partial documentation, 40-70% test coverage
- **RED**: Unclear abstractions, inconsistent patterns, poor documentation, < 40% test coverage

**Analysis Example**:
```
Codebase Review:

STRENGTHS:
├─ Service boundaries clear (auth, order, payment services)
├─ Consistent request/response patterns (all JSON, standard error format)
├─ README files present
└─ Each service has Dockerfile and deployment script

GAPS:
├─ No API contracts documented (OpenAPI/Swagger)
├─ Test coverage: Order service 45%, Auth service 70%, Payment service 60%
├─ Database schema not version controlled (manual migrations)
├─ Inconsistent logging (some services use JSON, others plain text)
├─ No architectural decision records (ADRs) for key choices
└─ Onboarding docs out of date (1+ year old)

DEVELOPER FEEDBACK:
├─ "Takes 2 weeks to understand system"
├─ "Database schema changes are scary"
├─ "Hard to know what each service should do"
├─ "Would like more examples of how to extend services"

SCORE: YELLOW (Works but needs documentation and testing improvements)
```

---

### Dimension 4: Operational Complexity

**Can the team operate this sustainably?**

Evaluate:
- [ ] Runbooks exist (incident response)?
- [ ] Monitoring/alerting covers key metrics?
- [ ] Logging centralized and searchable?
- [ ] Deployment process documented and tested?
- [ ] On-call burden manageable?
- [ ] Incident frequency acceptable?

**Scoring Criteria**:
- **GREEN**: Comprehensive runbooks, automated monitoring, alert fatigue < 10%, < 1 incident/month
- **YELLOW**: Partial runbooks, alerting gaps, moderate alert fatigue, 1-3 incidents/month
- **RED**: Minimal runbooks, blind spots in monitoring, high alert fatigue, > 3 incidents/month

**Analysis Example**:
```
Operational Assessment:

MONITORING:
├─ Request rate: ✅ Tracked
├─ Error rate: ✅ Tracked
├─ Latency: ⚠️ Basic (p50, p95) missing p99
├─ Database connections: ⚠️ Not tracked (critical for scaling)
├─ Cache hit rate: ❌ Not tracked
├─ Deployment success rate: ❌ Not tracked
└─ Incident: Need APM tracing (Datadog, New Relic, Jaeger)

ALERTING:
├─ High error rate (> 1%): ✅ Alerts
├─ High latency (p95 > 500ms): ✅ Alerts
├─ Low cache hit rate: ❌ No alert (alert created this week)
└─ Replication lag > 1s: ❌ No alert (critical!)

RUNBOOKS:
├─ "Database down": ✅ Exists (2 years old, needs update)
├─ "High latency": ⚠️ Exists but incomplete
├─ "Payment service fails": ❌ Doesn't exist
└─ "Cache fills up": ❌ Doesn't exist

ON-CALL BURDEN:
├─ Incidents per month: 3 (goal: < 1)
├─ Average MTTR: 45 min (goal: < 15 min)
├─ False alarms: 30% (goal: < 10%)
├─ On-call pager: Wakes up 2-3x per week

SCORE: YELLOW (Operational but not sustainable; on-call burden high)
```

---

### Dimension 5: Cost Efficiency

**Are resources used optimally?**

Evaluate:
- [ ] Right-sized instances (not under/over-provisioned)?
- [ ] Auto-scaling responsive to actual load?
- [ ] Caching strategy reduces database load?
- [ ] CDN or compression reducing bandwidth?
- [ ] Reserved capacity vs on-demand optimization?
- [ ] Cost tracking and alerts?

**Scoring Criteria**:
- **GREEN**: Optimized sizing, auto-scaling works, caching effective, cost per transaction minimized
- **YELLOW**: Generally efficient, some over-provisioning, caching could be better
- **RED**: Significant over/under-provisioning, wasteful resource usage, cost per transaction high

**Analysis Example**:
```
Cost Analysis:

CURRENT MONTHLY SPEND: $8,500

Breakdown:
├─ Kubernetes cluster: $5,000 (3 nodes, 4 CPUs each)
│  Issue: Nodes at 40% utilization (could consolidate to 2 nodes)
│  Savings: -$1,500/month if consolidated
│
├─ RDS (PostgreSQL): $2,000 (4 CPU, 16 GB RAM, multi-AZ)
│  Issue: 50% CPU utilization at peak, 15% off-peak
│  Savings: Auto-pause standby replica during off-peak: -$400/month
│
├─ ElastiCache (Redis): $800 (cache.r6g.xlarge, single node)
│  Issue: 90% memory utilization, no room for growth
│  Cost: Consider cluster mode (-$200) vs upgrade (+$400)
│
├─ RDS backups: $400
│  Issue: Keeping 30-day backups, only need 7 days
│  Savings: -$200/month
│
├─ Data transfer: $300
│  Issue: Cross-AZ traffic high (no caching at edge)
│  Savings: CDN implementation would reduce 30%: -$90/month
│
└─ Other (monitoring, logging): $2,000

COST PER USER: $8,500 / 5,000 DAU = $1.70 per DAU

Cost Optimization Opportunities:
1. Consolidate Kubernetes nodes: -$1,500/month
2. Auto-pause standby replica: -$400/month
3. Backup retention reduction: -$200/month
4. Consider commit pricing (reserved capacity): -$2,000/month (risk: inflexible)

Total Potential Savings: ~$4,100/month (48% reduction)

SCORE: RED (Significant over-provisioning; quick wins available)
```

---

### Summary Scorecard

Document all findings:

| Dimension | Score | Summary | Top Issue | Effort to Improve |
|-----------|-------|---------|-----------|-------------------|
| Scalability | YELLOW | Scales to 2x, needs DB+cache redesign for 10x | DB connection pooling at limit | 2-3 weeks |
| Reliability | YELLOW | Graceful degradation works, needs circuit breakers | Payment service failure not handled | 1-2 weeks |
| Maintainability | YELLOW | Code organized, docs lacking, test coverage ~55% | No API contracts (OpenAPI) documented | 2-3 weeks |
| Operational | YELLOW | Monitoring works, on-call burden high (3 incidents/mo) | Missing runbook for payment failure | 1 week |
| Cost Efficiency | RED | 48% over-provisioned; quick wins available | K8s nodes at 40% utilization | 1-2 weeks |

---

## Transition to Step 03

Once all dimensions analyzed:

```
"Analysis complete! Here's what I found:

STRENGTHS:
- Service boundaries are clear and well-designed
- Graceful degradation strategy works
- Code organization is good

AREAS TO IMPROVE:
- Scalability limited (DB and cache need work)
- Cost efficiency (significant over-provisioning)
- Operational burden (too many incidents, pager fatigue)

In Step 03, I'll give you specific recommendations with effort/impact tradeoffs,
and we'll build a roadmap for the next quarter."
```

**Proceed to [Step 03](step-03-recommendations.md)**

---

## Checkpoint

**Select an Option:**
**[C] Continue to step-03-recommendations.md**
**[A] Advanced Elicitation**
**[P] Party Mode**
**[Y] YOLO**

Menu Handling:
- IF C: Update frontmatter (add this step to stepsCompleted, update lastStep, lastSaved), save document, then read fully and follow `./step-03-recommendations.md`
- IF A: Perform advanced elicitation on current step output
- IF P: Invite other agents to party mode
- IF Y: Proceed without stopping
- IF Any other: respond and redisplay menu

**EXECUTION RULES:**
- 🛑 ALWAYS halt and wait for user input after presenting checkpoint menu
- 🛑 NEVER load the next step file until user selects 'C'
- 🛑 ALWAYS update frontmatter BEFORE loading next step
- 📖 Read the entire next step file before execution
