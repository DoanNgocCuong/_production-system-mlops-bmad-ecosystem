---
step_number: 1
step_name: "Incident Triage & Severity Classification"
next_step_file: "steps/step-02-diagnose.md"
estimated_time: "5-10 minutes"
lead_agent: "SRE Minh"
supporting_agents: ["Mary Analyst"]
skill_loads:
  - "references/5.08 - OBSERVABILITY - Monitoring & Logging/SKILL.md"
  - "references/5.10 - PRODUCTION READINESS - Go-Live & Operations/SKILL.md"
---

# STEP 1: Incident Triage & Severity Classification

## STEP GOAL

Rapidly gather initial incident information, assess impact scope, and classify severity level. This is the **speed phase** — we need accurate triage in < 5 minutes so we can move to diagnosis immediately.

**Outputs**:
- Incident ID (INC-YYYY-MM-DD-NNN)
- Severity classification (SEV1-4)
- Affected services list
- Initial impact hypothesis
- Triage report for team communication

## Skill Files to Load

```
Load: references/5.08 - OBSERVABILITY - Monitoring & Logging/SKILL.md
      → Alert patterns, symptom interpretation, service dependencies

Load: references/5.10 - PRODUCTION READINESS - Go-Live & Operations/SKILL.md
      → SLO/SLA definitions, impact assessment criteria
```

## MANDATORY EXECUTION RULES

**Speed Protocol**:
1. Ask only essential questions — max 5 questions total
2. Set hard 5-minute timer for triage phase
3. If severity unclear → escalate to SEV2, refine during diagnosis
4. Document answers as they come — don't wait for perfection

**Communication Protocol**:
1. Immediately notify #incidents Slack channel with initial report
2. Update every step with: "Triage: User impact: X%, Services affected: Y"
3. Page on-call architect if SEV1 detected

**Accuracy Rules**:
1. Get facts from users/monitoring FIRST before hypothesizing
2. Do not assume root cause during triage
3. Distinguish between "what we see" vs. "why it happens"

## EXECUTION PROTOCOL

### Phase 1: Initial Symptoms Gathering (2 min)

**Greeting**:

Xin chào, tôi là SRE Minh. Chúng ta sẽ xác định tình trạng incident.

Tôi cần hiểu nhanh tình hình hiện tại để vào phase diagnose.

**Critical Questions** (ask in order):

1. **Symptoms**: "What's happening right now? Are users getting errors, timeouts, or empty responses?"
   - Expected: Error messages, behavior description, affected operations

2. **Time Window**: "When did this start? Has it been happening continuously or intermittently?"
   - Expected: Timestamp (relative or absolute), duration estimate

3. **Scope**: "Which services/features are affected? Is the entire system down or partial?"
   - Expected: Service names, affected operations, user count estimate

4. **Recent Changes**: "Did anything change in last 15 minutes? Deployment, config, traffic spike?"
   - Expected: Deployments, config changes, scheduled maintenance, events

5. **User Impact**: "How many users are affected? Are they completely blocked or experiencing degradation?"
   - Expected: Percentage, absolute count, or "all users"

**Document Template**:

```
INCIDENT SYMPTOMS:
- What: [User report]
- When: [Timestamp - detection → report lag]
- Where: [Affected services/regions]
- Duration: [How long has this been happening]
- Scope: [% of users, absolute count if available]
```

### Phase 2: Severity Classification (2 min)

**Decision Matrix**:

| SEV | Definition | Response | Examples |
|-----|-----------|----------|----------|
| **SEV1** | Complete service down, revenue-blocking | Page on-call immediately | Authentication down, payment processing 100% failure, all API endpoints 5xx |
| **SEV2** | Major degradation, significant users affected | Escalate to lead on-call within 2 min | 50% error rate, API latency > 10s, 10k+ users blocked |
| **SEV3** | Moderate impact, some users affected | Assign to senior engineer | 10% error rate, latency spike to 2-5s, < 1k users affected, workaround available |
| **SEV4** | Minor issue, limited users, deferrable | Can be addressed during business hours | < 1% error rate, single feature degraded, specific user type affected |

**Classification Algorithm**:

```
IF (service == "authentication" OR service == "payments") AND impact > 10%
  → SEV1: Immediate escalation

ELSE IF (error_rate > 25% OR affected_users > 5000)
  → SEV2: Escalate quickly

ELSE IF (error_rate > 5% OR affected_users > 500)
  → SEV3: Assign to senior engineer

ELSE
  → SEV4: Schedule for business hours or next sprint
```

**Classification Format**:

```
SEVERITY CLASSIFICATION:
- Severity: [SEV1 | SEV2 | SEV3 | SEV4]
- Rationale: [1-2 sentences explaining why]
- Affected Users: [X users or Y%]
- Revenue Impact: [Direct loss estimate if applicable]
- SLO Breach: [Yes/No - which SLO violated?]
```

### Phase 3: Impact & Scope Summary (1 min)

**Gather Information**:

1. **Affected Services**: List all services showing errors/degradation
   ```
   Service A: 95% error rate (4xx/5xx)
   Service B: Latency increased from 100ms to 5s
   Service C: Database connection timeout
   ```

2. **User-Facing Impact**: How are end users experiencing this?
   ```
   Users unable to [specific operation]
   Users seeing [specific error message]
   Users experiencing [specific behavior]
   ```

3. **System Dependencies**: What's downstream from the problem?
   ```
   Service A failure → Service B cannot process → Users blocked on checkout
   ```

**Impact Summary**:

```
INITIAL IMPACT ASSESSMENT:
- Direct impact: [What's immediately broken]
- Cascade risk: [What might break next]
- Estimated reach: [X users, Y% of traffic]
- Business impact: [Revenue, SLA, reputation]
```

### Phase 4: Generate Triage Report (1 min)

**Report Format** (send to #incidents channel immediately):

```
🚨 INCIDENT TRIAGE REPORT — INC-[DATE]-[NNN]

Severity: [SEV1 | SEV2 | SEV3 | SEV4]
Status: Triaged — Moving to diagnosis

Symptoms:
  • [Symptom 1]
  • [Symptom 2]

Affected Services:
  • Service-1: [Impact description]
  • Service-2: [Impact description]

Timeline:
  • Started: [Time]
  • Detected: [Time]
  • Triaged: [Now]

User Impact:
  • ~X users affected
  • Y% of traffic blocked

Next Step: Moving to diagnosis phase
Assigned: SRE Minh + team
```

## DECISION POINTS

### [A] Advanced Elicitation (Ask More Questions)

Use if:
- Symptoms unclear or contradictory
- Scope seems larger than reported
- Multiple services implicated

Questions to refine:
- "Can you reproduce this? What exact steps?"
- "Are there any recent alerts or warnings?"
- "Is traffic volume unusual right now?"
- "Are there any database errors in logs?"

### [C] Continue to Diagnosis

- Accept triage classification
- Move to step-02-diagnose.md
- Load observability tools (metrics, logs, traces)

---

## SUCCESS / FAILURE METRICS

**Triage Success**:
- ✓ Severity correctly classified (verify in postmortem)
- ✓ All affected services identified
- ✓ Report delivered to team < 5 minutes from incident start
- ✓ On-call engineer escalated (if SEV1)

**Triage Failures** (learn for next incident):
- ✗ Severity mismatch discovered during diagnosis
- ✗ Additional services discovered that weren't mentioned
- ✗ Root cause was obvious, should have been caught during triage

---

**Bây giờ hãy báo cáo: Symptoms? Scope? User impact?**

**[A] Ask more | [C] Continue to diagnose**
