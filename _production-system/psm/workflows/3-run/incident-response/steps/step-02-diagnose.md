---
step_number: 2
step_name: "Systematic Root Cause Diagnosis"
next_step_file: "steps/step-03-fix.md"
estimated_time: "10-30 minutes"
lead_agent: "SRE Minh"
supporting_agents: ["Architect Khang"]
skill_loads:
  - "references/5.08 - RUN - Observability & Monitoring/SKILL.md"
  - "references/5.07 - RUN - Reliability & Resilience/SKILL.md"
  - "references/5.09 - RUN - Error Handling/SKILL.md"
is_final_step: false
outputFile: "{project-root}/psm/outputs/incident-response/{incident_id}/incident-report.md"
workflowType: "incident-response"
---

# STEP 2: Systematic Root Cause Diagnosis

## STEP GOAL

Using observability data (metrics, logs, distributed traces), identify the root cause of the incident. This is the **investigative phase** — methodical analysis to distinguish symptom from cause.

**Outputs**:
- Root cause hypothesis (supported by data)
- Evidence collected from metrics, logs, traces
- Affected system layer (application, database, network, infrastructure)
- Timeline of events leading to incident
- Next steps for fix

## Skill Files to Load

```
Load: references/5.08 - RUN - Observability & Monitoring/SKILL.md
      → Structured logging queries, metrics dashboards, common alert patterns

Load: references/5.07 - RUN - Reliability & Resilience/SKILL.md
      → Circuit breaker states, timeout cascades, fallback failures

Load: references/5.09 - RUN - Error Handling/SKILL.md
      → Error code classification, recovery mechanism failures
```

## MANDATORY EXECUTION RULES

**Investigation Protocol**:
1. **Collect first** — Gather all observability data before hypothesizing
2. **Correlate** — Find patterns across metrics + logs + traces
3. **Eliminate** — Rule out innocent services systematically
4. **Evidence-based** — Every hypothesis must be supported by data
5. **Timeline reconstruction** — Build exact sequence of events

**Data Collection Priority**:
1. Metrics dashboards (latency p50/p95/p99, error rate, throughput)
2. Error logs (ERROR, PANIC, FATAL entries)
3. Distributed traces (identify request bottleneck)
4. System logs (OOM, file descriptors, network errors)
5. Dependency changes (recent code/config/infra changes)

**Time-Box**: Diagnose within 20 minutes. If root cause not clear → hypothesize best guess and move to fix attempt.

## BMAD v6.2 Checkpoint Protocol

**Before proceeding, ensure output document exists with YAML frontmatter:**
```yaml
---
stepsCompleted: ["step-01-triage.md"]
lastStep: "step-02-diagnose.md"
lastSaved: "{current-timestamp}"
workflowType: "incident-response"
---
```

**SAVING RULE:** After completing all execution sections, BEFORE loading the next step:
1. Update output document: append current step's output to the document
2. Update frontmatter: add this step to `stepsCompleted`, update `lastStep` and `lastSaved`
3. Save the document

---

## EXECUTION PROTOCOL

### Phase 1: Metrics Dashboard Analysis (8 min)

**Key Metrics to Check**:

| Metric | What It Tells | Where to Find |
|--------|---------------|---------------|
| **Latency (p50/p95/p99)** | Response time distribution | Prometheus, Datadog, CloudWatch |
| **Error Rate** | % of failed requests (4xx/5xx) | Application metrics, HTTP status codes |
| **Throughput (RPS)** | Requests per second | API gateway, load balancer metrics |
| **CPU/Memory** | Resource utilization on servers | Host metrics, container monitoring |
| **Database Latency** | Query execution time | Database-specific monitoring |
| **Queue Depth** | Message backlog in async systems | Message broker, job queue monitoring |

**Analysis Template**:

```
METRICS ANALYSIS:
1. Latency Impact?
   - Baseline p99: [X]ms
   - Current p99: [Y]ms
   - Delta: [Y-X]ms → Root cause: [Application | DB | Network | External]?

2. Error Rate Impact?
   - Baseline: [X]%
   - Current: [Y]%
   - Spike at: [Timestamp] coinciding with what else?

3. Throughput Change?
   - Baseline RPS: [X]
   - Current RPS: [Y]
   - Drop/Spike suggests: [Rate limiting? Downstream failure? Client timeout?]

4. Resource Saturation?
   - CPU: [X]% → Normal [Y]% baseline
   - Memory: [X]% → Normal [Y]% baseline
   - Connections: [X] open → Normal [Y] baseline
   - Suggests: [Scale issue? Memory leak? Connection pool exhaustion?]
```

**Questions to Ask**:

1. "When did the spike start? Exactly timestamp from metrics."
2. "Which service shows the spike first? Trace the cascade backward."
3. "Is latency degradation uniform or in tail? p99 spike but p50 normal?"
4. "Error rate came from where? 5xx (server) or 4xx (client)?"

### Phase 2: Error Log Analysis (8 min)

**Log Query Strategy**:

```
Step 1: Find errors in time window
  Query: ERROR logs between [incident_start - 2min] and [now]
  Filters: service="[service_name]"

Step 2: Find error patterns
  What's the most common error message?
  Which code module generated it?
  How many? (100s? 1000s? growing?)

Step 3: Extract error details
  error_code: [specific code]
  error_message: [exact message]
  affected_user_count: [derived from logs]
  retry_count: [how many automatic retries fired?]
```

**Common Error Patterns**:

```
❌ "Connection timeout to database"
  → Diagnosis: DB connection pool exhausted or DB unresponsive
  → Evidence: database_connections_total increasing, DB latency spike

❌ "Circuit breaker open for service-X"
  → Diagnosis: Dependent service is down/slow, circuit breaker protecting us
  → Evidence: Calls to service-X failing, circuit breaker threshold hit

❌ "Out of memory (OOM kill)"
  → Diagnosis: Memory leak or insufficient heap
  → Evidence: Memory usage increasing linearly, no gc.pause spikes

❌ "Rate limit exceeded"
  → Diagnosis: Traffic spike or thundering herd from client
  → Evidence: Request rate suddenly increased, client IP concentrated

❌ "Deployment rollout in progress"
  → Diagnosis: New version has bug, killing old instances too fast
  → Evidence: Timeline: deployment started → errors started, pod restarts increasing
```

**Error Analysis Template**:

```
LOG ANALYSIS:
Error Message: [Most common error]
Frequency: [X errors/second at peak]
First Occurrence: [Timestamp] → suggests upstream cause at [T-lag]
Affected Component: [Service + function/method]
Affected Users: [X% of traffic or Y concurrent users]

Error Code Breakdown:
  - 500 Internal Server Error: X%
  - 503 Service Unavailable: Y%
  - 504 Gateway Timeout: Z%
  - [Other]: A%

Root Cause Hypothesis (from logs):
  [What the logs are telling us]
```

### Phase 3: Distributed Tracing (6 min)

**Trace Analysis Questions**:

1. **Request Flow**: "Where is the request getting stuck? At which service/database?"
   - Trace request from entry point → find where latency is
   - Is it: API gateway? Service A? Database call? External API?

2. **Span Durations**: "Which operation takes longest?"
   - If database call spans show: 5s at baseline, now 30s → DB problem
   - If API gateway → Service A spans normal but API gateway → client is 20s → network issue

3. **Error Context**: "What request attributes correlate with errors?"
   - All errors are user_id=X? → User-specific issue (their data, permissions)
   - All errors are operation=Y? → Specific operation broken
   - All errors are region=Z? → Infrastructure in that region

**Trace Template**:

```
TRACE ANALYSIS:
Sample Trace ID: [trace_id]
Request Timeline:
  1. API Gateway entry: T+0ms
  2. Service-A processing: T+50ms (expected: 30ms) → +20ms overhead
  3. Database query: T+100ms (expected: 80ms) → +20ms added
  4. Cache lookup: T+150ms (TIMEOUT after 50ms) → BOTTLENECK
  5. Response back: T+200ms+ (timeout or very slow)

Bottleneck Identified: [Service/Resource/Operation]
Critical Span: [operation_name taking expected_time → actual_time]
Suggests: [Root cause hypothesis based on trace data]
```

### Phase 4: Identify Root Cause Candidate (5 min)

**Cause Categories** (pick which applies):

**A. Dependency Failure**
- Dependent service down or timing out
- Evidence: Calls to service-X failing, circuit breaker triggered
- Fix direction: Restore dependent service, or enable fallback

**B. Resource Exhaustion**
- CPU, memory, connections, file descriptors maxed out
- Evidence: CPU/memory at 100%, connection pool exhausted
- Fix direction: Scale up, kill runaway processes, optimize usage

**C. Database Issue**
- Query timeout, connection pool exhaustion, slow queries
- Evidence: DB latency spike, connection count spike, query logs showing slow queries
- Fix direction: Kill slow query, optimize index, scale DB reads, restart DB connections

**D. Code Regression**
- New deployment introduced bug
- Evidence: Errors started exactly at deployment time, error traces point to new code
- Fix direction: Rollback deployment, hotfix if rollback not available

**E. Configuration Change**
- Setting changed that broke behavior
- Evidence: Errors started after config update, specific to changed setting
- Fix direction: Revert config change, apply fix to code

**F. External Issue**
- External service/API down, network issue, DNS issue
- Evidence: Calls to external service timeout, DNS lookups fail
- Fix direction: Wait for external service recovery, switch provider, enable fallback

**Root Cause Verdict**:

```
ROOT CAUSE IDENTIFIED:
Category: [A | B | C | D | E | F]
Specific Cause: [Detailed description]

Evidence:
  1. [Data point 1]
  2. [Data point 2]
  3. [Data point 3]

Confidence: [High | Medium | Low]

If Low Confidence:
  - Best guess: [Most likely cause]
  - Backup hypothesis: [Second most likely]
  - Plan: Attempt fix for best guess, monitor for confirmation
```

## DECISION POINTS

### [A] Advanced Investigation (Deeper Trace Analysis)

Use if:
- Root cause still unclear after initial analysis
- Multiple hypotheses equally likely
- Need to analyze more traces, logs, or metrics

Actions:
- Collect 5-10 trace samples, analyze patterns
- Query logs with additional filters (user_id, operation, region)
- Check for correlated events (deployments, config changes, traffic spikes)

### [C] Continue to Fix

- Accept root cause hypothesis
- Move to step-03-fix.md
- Prepare fix strategy based on cause category

---

## SUCCESS / FAILURE METRICS

**Diagnosis Success**:
- ✓ Root cause identified with high confidence (>80%)
- ✓ Evidence collected from 2+ observability sources
- ✓ Timeline shows clear sequence of events
- ✓ Diagnosis completed within 20 minutes

**Diagnosis Failures** (refinement for postmortem):
- ✗ Root cause was wrong, fix didn't work
- ✗ Missed obvious evidence (error logs, metric spike)
- ✗ Hypothesized too early, spent time on wrong direction

---

## Checkpoint

**Select an Option:**
**[C] Continue to step-03-fix.md**
**[A] Advanced Elicitation**
**[P] Party Mode**
**[Y] YOLO**

Menu Handling:
- IF C: Update frontmatter (add this step to stepsCompleted, update lastStep, lastSaved), save document, then read fully and follow `./step-03-fix.md`
- IF A: Perform advanced elicitation on current step output
- IF P: Invite other agents to party mode
- IF Y: Proceed without stopping
- IF Any other: respond and redisplay menu

**EXECUTION RULES:**
- 🛑 ALWAYS halt and wait for user input after presenting checkpoint menu
- 🛑 NEVER load the next step file until user selects 'C'
- 🛑 ALWAYS update frontmatter BEFORE loading next step
- 📖 Read the entire next step file before execution
