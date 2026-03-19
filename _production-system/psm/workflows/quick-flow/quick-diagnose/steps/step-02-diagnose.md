---
step_id: QD-S02
step_name: Diagnose & Fix
workflow_id: QD001
sequence: 2
type: "diagnosis"
agent: "Minh (SRE)"
description: "Analyze metrics, identify root cause, suggest fix, verify"
estimated_time: "10-15 minutes"
is_final_step: true
outputFile: "{project-root}/psm/outputs/quick-diagnose/{incident_id}/diagnosis-report.md"
workflowType: "quick-diagnose"
---

# Step 02: Root Cause Analysis & Fix Suggestion

## Objective

Diagnose root cause and provide actionable fix:
1. Analyze metrics from Step 01
2. Formulate root cause hypothesis
3. Suggest immediate fix
4. Recommend verification steps

## BMAD v6.2 Checkpoint Protocol

**Before proceeding, ensure output document exists with YAML frontmatter:**
```yaml
---
stepsCompleted: ["step-01-gather.md"]
lastStep: "step-02-diagnose.md"
lastSaved: "{current-timestamp}"
workflowType: "quick-diagnose"
---
```

**SAVING RULE:** After completing all execution sections, BEFORE loading the next step:
1. Update output document: append current step's output to the document
2. Update frontmatter: add this step to `stepsCompleted`, update `lastStep` and `lastSaved`
3. Save the document

---

## Execution Flow

### Diagnosis Framework

Using symptom + metrics, work through common causes:

```
Is it a TRAFFIC ISSUE?
├─ YES → Check: Rate limiting? Auto-scaling? Capacity?
└─ NO → Next check

Is it a SLOW QUERY?
├─ YES → Check: N+1 queries? Missing index? Connection pool exhausted?
└─ NO → Next check

Is it a DEPENDENCY TIMEOUT?
├─ YES → Check: Downstream service down/slow? Network issue? Circuit breaker?
└─ NO → Next check

Is it a RESOURCE EXHAUSTION?
├─ YES → Check: Memory leak? Disk full? CPU overload? Handles/connections?
└─ NO → Next check

Is it a DEPLOYMENT ISSUE?
├─ YES → Check: Bad code deployed? Config changed? Database migration?
└─ NO → Next check (environmental)
```

### Common Production Issues & Fixes

#### Issue 1: High API Latency (p99 > target)

**Diagnosis Checklist**:
```
□ Is error rate also high? (if yes, likely error handling issue)
□ Is request rate normal? (if no, likely traffic surge)
□ Check slow query logs
□ Check CPU/memory utilization
□ Check database connection pool
□ Check external dependency latency
```

**Common Root Causes** (in order of likelihood):
1. Database query performance (missing index, N+1 query)
2. Traffic surge (auto-scale not responding fast enough)
3. Slow downstream dependency
4. Memory leak or resource exhaustion
5. Bad code deployment

**Quick Fixes**:
```
IF it's a missing database index:
  IMMEDIATE: Kill slow queries, restart service if needed
  FIX: Add index (5-10 min if small table)
  VERIFY: Query latency drops immediately

IF it's a slow downstream service:
  IMMEDIATE: Reduce timeout, implement fallback
  FIX: Contact dependent team, scale their service
  VERIFY: Monitor their status page

IF it's a traffic surge:
  IMMEDIATE: Enable rate limiting, reduce canary traffic
  FIX: Scale up more instances (5-10 min)
  VERIFY: Latency recovers as traffic redistributes
```

---

#### Issue 2: High Error Rate (500 errors)

**Diagnosis Checklist**:
```
□ Check error logs (what's the error?)
□ Check if it's new code (recent deployment?)
□ Check if it's dependent service failure
□ Check database connectivity
□ Check configuration
```

**Common Root Causes**:
1. Recent deployment with bug
2. Database connectivity loss
3. Dependent service down/returning errors
4. Configuration/secrets issue
5. Resource exhaustion (OOM, disk full)

**Quick Fixes**:
```
IF it's a recent deployment:
  IMMEDIATE: Rollback to previous version (1-2 min)
  FIX: Debug issue in development
  VERIFY: Error rate drops to baseline

IF it's database connectivity:
  IMMEDIATE: Check database status, connections
  FIX: Restart connection pool, scale database
  VERIFY: Connection count normalizes

IF it's a dependent service:
  IMMEDIATE: Enable circuit breaker / fallback
  FIX: Contact dependent team
  VERIFY: Service recovers or fallback activates
```

---

#### Issue 3: Intermittent Timeouts

**Diagnosis Checklist**:
```
□ Are timeouts on specific operations? (if yes, check that operation)
□ Are they spiking at specific times? (if yes, check for scheduled jobs)
□ Is one pod consistently timing out? (if yes, check that pod)
□ Is it load-dependent? (if yes, check overload handling)
```

**Common Root Causes**:
1. Specific operation with performance issue
2. Scheduled background job running (daily task, nightly backup)
3. Pod with memory leak or high CPU
4. Overload handling / queue backlog
5. Flaky external dependency

**Quick Fixes**:
```
IF it's a specific operation:
  IMMEDIATE: Move operation to background job if possible
  FIX: Optimize the query/code
  VERIFY: Operation latency improves

IF it's a scheduled job:
  IMMEDIATE: Temporarily disable job if not critical
  FIX: Schedule during off-peak or optimize job
  VERIFY: Timeouts drop during job window

IF it's a bad pod:
  IMMEDIATE: Kill the pod (Kubernetes auto-restarts)
  FIX: Monitor pod for memory leak or high CPU
  VERIFY: New pod performs normally
```

---

### Generate Diagnosis Report

Create quick diagnosis summary:

```markdown
## Quick Diagnosis Report

**Issue**: High API latency (p99 = 450ms, target = 100ms)
**Timeline**: Started 20 minutes ago
**Impact**: 10% of requests affected, some users experiencing slow response
**Duration**: Ongoing

### Root Cause Hypothesis

**PRIMARY**: Database query performance (missing index on users.email)

Evidence:
- API latency correlated with database query latency (both spiked at T-0:15)
- Slow query log shows 50+ queries on users.email taking 200-500ms each
- Index on users.email missing (checked schema)
- Request rate normal (no traffic surge)
- CPU/memory normal

### Immediate Mitigation

1. **Option A: Kill slow queries** (30 seconds)
   ```sql
   SELECT query FROM pg_stat_statements
   WHERE query LIKE '%users.email%'
   ORDER BY mean_time DESC;
   -- Kill: SELECT pg_terminate_backend(pid) WHERE query = '...'
   ```
   Impact: Releases database connections, service recovers temporarily
   Duration: Works until requests restart

2. **Option B: Reduce query load** (2-3 minutes)
   - Disable specific feature if possible
   - Reduce caching TTL (lower caching overhead, faster recovery)
   - Implement fallback response (degrade feature gracefully)

### Root Fix

**Add database index** (effort: 5-10 minutes)
```sql
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);
-- CONCURRENTLY avoids locking table
-- Takes ~10 min on 100M row table
```

**Effort**: 5-10 minutes
**Risk**: Low (index is purely additive)
**Verification**:
- Query latency drops to < 10ms after index creation
- API p99 latency recovers to < 100ms
- No errors during index creation

### Verification Steps

```
1. Monitor database query latency (should drop from 500ms to < 10ms)
2. Monitor API latency (should drop from 450ms to < 100ms)
3. Verify no errors during index creation
4. Confirm user complaints stop
```

### Follow-up Actions

- [ ] Post-incident: Run ANALYZE to update query planner
- [ ] Add monitoring for slow queries (alert if > 100ms)
- [ ] Review schema for other missing indexes
- [ ] Document: Which features trigger this query? How is email searched?
```

---

### Suggest Verification

Once fix applied, verify:

```
VERIFICATION CHECKLIST:

□ API latency (p99) back to baseline?
□ Error rate back to normal?
□ No new errors after fix?
□ Users reporting normal experience?
□ Monitoring shows stability?
```

---

## Workflow Complete

```
"Diagnosis complete. The issue is [root cause].

Immediate action: [fix option A/B if needed for immediate relief]
Root fix: [permanent fix option]

Effort: [X minutes]
Urgency: [HIGH/MEDIUM/LOW based on impact]

Should I proceed with [fix option]?"
```

---

## Deliverables

1. **Diagnosis Summary**
   - Root cause identified
   - Evidence provided
   - Confidence level stated

2. **Fix Recommendation**
   - Immediate mitigation (if needed)
   - Root fix option
   - Effort estimate

3. **Verification Plan**
   - What metrics to watch
   - How to verify fix worked

---

## Workflow Complete

This is the **FINAL STEP** of the Quick Diagnose workflow.

```
"Quick diagnosis complete!

Steps Completed:
  1. Step 01: Gather Context (symptoms, metrics, timeline)
  2. Step 02: Diagnose & Fix (root cause, recommendations)

Root Cause: [Identified]
Fix Recommendation: [Provided]
Verification: [Plan documented]

Quick Diagnose Workflow: COMPLETE"
```

**Next Steps**:
1. Apply the recommended fix
2. Verify recovery per the verification plan
3. Document lessons learned for future incidents

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
