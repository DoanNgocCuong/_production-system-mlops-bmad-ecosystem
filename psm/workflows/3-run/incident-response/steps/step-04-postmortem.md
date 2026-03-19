---
step_number: 4
step_name: "Postmortem Documentation & Action Items"
next_step_file: null
is_final_step: true
estimated_time: "15-30 minutes"
lead_agent: "SRE Minh"
supporting_agents: ["Architect Khang", "Mary Analyst"]
skill_loads:
  - "references/5.14 - DOCUMENTATION - Runbooks & Incident Reports/SKILL.md"
  - "references/5.10 - PRODUCTION READINESS - Go-Live & Operations/SKILL.md"
---

# STEP 4: Postmortem Documentation & Action Items

## STEP GOAL

Document the incident thoroughly for organizational learning, identify root causes at system level (not just surface fix), and create action items to prevent recurrence. This is the **reflection phase** — turning incident into knowledge.

**Outputs**:
- Comprehensive incident postmortem document
- Root cause analysis (5-why analysis)
- Prevention action items (monitored, assigned, deadlined)
- Lessons learned documented
- Incident timeline with exact timestamps

## Skill Files to Load

```
Load: references/5.14 - DOCUMENTATION - Runbooks & Incident Reports/SKILL.md
      → Postmortem templates, incident report structure, action item tracking

Load: references/5.10 - PRODUCTION READINESS - Go-Live & Operations/SKILL.md
      → Prevention checklist, SLO/SLA implications, alerting improvements
```

## MANDATORY EXECUTION RULES

**Postmortem Protocol**:
1. **Blameless** — Focus on systems/processes, not individuals
2. **Complete** — Timeline must include all significant events
3. **Evidence-based** — Every claim supported by logs/metrics/traces
4. **Actionable** — Each action item has owner, deadline, and acceptance criteria
5. **Timely** — Complete within 24 hours of incident resolution

**Action Item Standards**:
1. **SMART format**: Specific, Measurable, Assignable, Realistic, Time-bound
2. **Prevention-focused**: Prevents same incident, not just adds alerting
3. **Owner-assigned**: Engineer + manager, not "team"
4. **Deadline clear**: Days, not "ASAP" or "next sprint"
5. **Acceptance criteria**: How do we know action is done and effective?

**5-Why Analysis Rule**:
- Go 5 levels deep: "Why?" x 5 to find root system cause, not proximate cause

## EXECUTION PROTOCOL

### Phase 1: Incident Timeline Reconstruction (5 min)

**Timeline Template**:

```
INCIDENT TIMELINE (in UTC):

2026-03-17T14:30:00Z — [DETECTION]
  Alert fired: "Error rate > 25%"
  Service: payment-service
  Detection lag: ~0 seconds
  Source: Prometheus alert

2026-03-17T14:31:00Z — [USER REPORT]
  Slack: users reported "checkout fails with 500"
  Estimate: ~100 users affected at detection time

2026-03-17T14:32:00Z — [ON-CALL ENGAGED]
  SRE Minh paged (response time: ~1 minute)
  Escalation: Architect Khang also engaged
  Status: Incident #1 created

2026-03-17T14:35:00Z — [TRIAGE COMPLETE]
  Classification: SEV1 (revenue-blocking)
  Affected services: payment-service, order-service
  Root cause hypothesis: Database connection timeout

2026-03-17T14:50:00Z — [ROOT CAUSE IDENTIFIED]
  Diagnosis: Database query timeout in order-service
  Culprit: New deployment (v2.5.1) introduced N+1 query
  Evidence: Query execution logs show 50s per order lookup

2026-03-17T15:10:00Z — [FIX APPLIED]
  Action: Rollback to v2.5.0
  Method: Blue-green deployment, instant traffic flip
  Time to execute: 30 seconds
  Metrics response: Immediate improvement

2026-03-17T15:15:00Z — [RECOVERY VERIFIED]
  Error rate: 95% → 2% (within baseline)
  Latency p99: 15s → 200ms (baseline restoration)
  User reports: No new complaints in last 5 minutes
  Status: RESOLVED

2026-03-17T15:30:00Z — [POSTMORTEM START]
  SRE begins documentation
```

**Key Metrics to Include**:
- Detection to resolution time: X minutes (MTTR)
- Users affected: Y users or Z% of traffic
- Revenue impact: $X estimated loss (if applicable)
- Severity: SEV1/2/3/4
- Root cause: [Specific]

### Phase 2: Root Cause Analysis via 5-Why (8 min)

**Why We Use 5-Why**:
- Symptom: "Database queries timing out"
- Proximate cause: "N+1 query pattern in new code"
- System cause: "Code review didn't catch N+1 pattern"
- Process cause: "No automated test for query performance"
- Organizational cause: "Performance testing not in deployment checklist"

**5-Why Template**:

```
SYMPTOM: Database queries taking 50 seconds (vs. 100ms baseline)

WHY #1: Why are queries slow?
ANSWER: New code in v2.5.1 uses N+1 query pattern
EVIDENCE: Code review shows loop with query inside

WHY #2: Why did N+1 pattern get deployed?
ANSWER: Code review didn't catch the performance issue
EVIDENCE: Review process focused on functionality, not performance

WHY #3: Why didn't code review catch it?
ANSWER: No automated test for query performance regression
EVIDENCE: Test suite runs integration tests, but not performance benchmarks

WHY #4: Why is there no performance regression test?
ANSWER: Performance testing not part of standard test suite
EVIDENCE: CONTRIBUTING.md mentions unit/integration tests, not performance tests

WHY #5: Why isn't performance testing required?
ANSWER: System design assumption that "engineers know to avoid N+1"
EVIDENCE: No explicit training, no automated guard rails

ROOT SYSTEM CAUSE:
Missing automated guard rails for query performance.
Assumption that developers won't write N+1 queries proven wrong.
```

**Root Cause Classification**:

```
Direct Root Cause: Code regression in v2.5.1 (N+1 query pattern)
Enabling Factor: No code review for performance implications
Systemic Issue: Lack of automated query performance testing

Which one should we fix to prevent this?
→ Enabling factor (code review improvement)
→ Systemic issue (add automated tests) — MOST IMPORTANT
```

### Phase 3: Identify Prevention Actions (10 min)

**Prevention Actions by Category**:

#### **Category 1: Immediate (Deploy This Week)**

```
Action: Add query performance check to CI/CD pipeline
Type: Automated guard rail
Implementation:
  - Tool: QueryTracer or similar (analyze query execution)
  - Trigger: On every PR, run performance test suite
  - Criteria: Reject if single query > 100ms (configurable)
  - Owner: Architect Khang
  - Deadline: 2026-03-20 EOD
  - Acceptance: Green CI check passes on all PRs, catches slow queries

Owner: Architect Khang
Deadline: Friday 2026-03-20 EOD
Testing: Add test case that intentionally adds N+1, verify CI rejects
```

#### **Category 2: Short-term (Deploy Next Sprint)**

```
Action: Implement query performance monitoring dashboard
Type: Observability improvement
Implementation:
  - Metrics: query_execution_time_p99 per endpoint
  - Alert: If p99 > 200ms, alert #incidents
  - Dashboard: Grafana showing top slow queries
  - Owner: SRE Minh
  - Deadline: 2026-03-31 EOD
  - Acceptance: Dashboard shows all queries, alerts firing on p99 > 200ms

Owner: SRE Minh
Deadline: Sprint 13 end (2026-03-31)
Testing: Verify alerts fire when slow query introduced
```

#### **Category 3: Long-term (Architectural Improvement)**

```
Action: Implement ORM query depth limit / N+1 detection
Type: Framework-level protection
Implementation:
  - Tool: Add query analysis to ORM layer
  - Feature: Fail fast if N+1 pattern detected at runtime
  - Config: max_query_depth = 3 (configurable per endpoint)
  - Owner: Architect Khang
  - Deadline: 2026-04-30 EOD (next quarter)
  - Acceptance: Framework rejects N+1 at runtime with clear error

Owner: Architect Khang
Deadline: Q2 2026 (2026-04-30)
Testing: Verify both development and production environments enforce
```

**Action Items Template**:

```
ACTION ITEM #1: [Title]
Type: [Automated guard rail | Monitoring | Process | Training]
Description: [What should be done and why]
Owner: [Engineer name + manager]
Deadline: [Specific date]
Acceptance Criteria: [How do we verify it's done and effective?]
Priority: [High | Medium | Low]
Prevention Impact: [Percentage reduction in similar incidents]

ACTION ITEM #2: ...
```

### Phase 4: Lessons Learned & Recommendations (5 min)

**Lessons Learned Section**:

```
WHAT WENT WELL:
1. Detection was immediate (Prometheus alert < 1 second)
   → Alert sensitivity correctly tuned

2. Root cause identified quickly (15 minutes to diagnosis)
   → Strong observability tools (logs, traces, metrics)

3. Deployment rollback executed cleanly (30 seconds)
   → Blue-green deployment strategy effective

WHAT COULD BE BETTER:
1. Code review missed performance implications
   → Need performance review checklist in CR process

2. No pre-production performance test
   → Should add performance regression tests to test suite

3. Deployment process allowed slow query without alert
   → Should have production monitoring in place before go-live

RECOMMENDATIONS:
1. Add query performance regression tests (high priority)
2. Update code review checklist to include "Performance implications?"
3. Implement query latency monitoring dashboard
4. Add performance acceptance criteria to definition of done
5. Train team on N+1 query patterns and how to detect
```

### Phase 5: Generate Postmortem Document (5 min)

**Final Postmortem Format**:

```markdown
# INCIDENT POSTMORTEM

## Executive Summary
- Incident ID: INC-2026-03-17-001
- Severity: SEV1 (Revenue-blocking)
- Duration: 45 minutes (2026-03-17 14:30:00 - 15:15:00 UTC)
- Users Affected: ~5,000 users (2% of DAU)
- Revenue Impact: ~$10k estimated loss
- Root Cause: N+1 query pattern in v2.5.1 deployment

## Timeline
[Detailed timeline as reconstructed]

## Impact Assessment
- Services Down: payment-service, order-service
- Operations Blocked: Checkout, payment processing
- User-Facing: "500 Internal Server Error" on checkout
- Dependent Services: Reports service couldn't generate reports (downstream)

## Root Cause Analysis (5-Why)
[Complete 5-Why analysis as above]

## What Happened (Narrative)
[2-3 paragraph explanation of incident flow]

## Resolution
- Action: Rollback v2.5.1 → v2.5.0
- Result: Immediate recovery
- Verification: Error rate returned to baseline within 2 minutes

## Prevention Actions
[List of action items with owners and deadlines]

## Lessons Learned
[What went well, what to improve]

## Action Items
- [ ] #1: Add query performance check to CI (Khang, 2026-03-20)
- [ ] #2: Implement query performance dashboard (Minh, 2026-03-31)
- [ ] #3: ORM query depth limiting (Khang, 2026-04-30)
```

## DECISION POINTS

### [A] Advanced Review (Multi-Team Discussion)

Use if:
- Incident affects multiple teams
- Root cause spans multiple systems
- Want to discuss prevention strategy with stakeholders

Actions:
- Schedule 30-minute postmortem meeting with lead engineers
- Present timeline, root cause, and proposed action items
- Get alignment on owners, deadlines, and priorities

### [C] Complete Postmortem

- Accept postmortem as documented
- Assign action items to owners
- Publish postmortem to team wiki/Slack
- Schedule follow-up to verify action item completion

---

## SUCCESS / FAILURE METRICS

**Postmortem Success**:
- ✓ Timeline accurate (verified by participants)
- ✓ Root cause understood by team
- ✓ 3+ prevention action items identified
- ✓ All action items assigned with clear deadlines
- ✓ Postmortem shared with team < 24 hours post-incident

**Postmortem Failures** (Refinement):
- ✗ Timeline incomplete or inaccurate
- ✗ Root cause misunderstood (proven by similar incident recurring)
- ✗ Action items vague or unowned
- ✗ Prevention action items never executed

**Incident Prevention Success (Measured Later)**:
- ✓ Similar incident does NOT recur within 90 days
- ✓ Action items completed on time
- ✓ Observability improvements catch similar patterns proactively
- ✓ Team learns from incident (evidenced in code reviews)

---

## INCIDENT RESOLUTION COMPLETE

✓ **Status**: RESOLVED
✓ **MTTR**: [X minutes] — from detection to recovery
✓ **Root Cause**: [Identified]
✓ **User Impact**: [Resolved]
✓ **Postmortem**: [Documented]
✓ **Prevention**: [Action items assigned]

**Next Steps**:
1. Share postmortem with team (Slack + Wiki)
2. Schedule 1:1 follow-up with action item owners
3. Review postmortem in next team meeting
4. Verify action items completed by deadline

Cảm ơn toàn bộ team đã hỗ trợ để resolve incident này.
Chúng ta sẽ học hỏi và cải tiến để tránh incident tương tự.

**Incident Response Workflow: COMPLETE**

---

**[C] Hoàn tất postmortem**
