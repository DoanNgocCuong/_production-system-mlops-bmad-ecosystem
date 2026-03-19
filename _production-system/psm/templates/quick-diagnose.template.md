---
template_name: quick-diagnose
template_version: "1.0.0"
created_date: 2026-03-19
workflow_id: quick-diagnose
description: Quick diagnosis report for rapid issue identification and resolution
---

# Quick Diagnosis Report

**Service**: {{SERVICE_NAME}}
**Engineer**: {{ENGINEER_NAME}}
**Date**: {{DATE}}
**Duration**: {{DURATION_MINUTES}} minutes
**Status**: {{RESOLVED | INVESTIGATING | ESCALATED}}

---

## Issue Summary

**Title**: {{ISSUE_TITLE}}

**Classification**: {{HIGH_LATENCY | HIGH_ERROR_RATE | TIMEOUT | RESOURCE_EXHAUSTION | DEPENDENCY_FAILURE | OTHER}}

**Severity**: {{SEV1 | SEV2 | SEV3 | SEV4}}

**Impact**:
- {{X}}% of {{USERS|REQUESTS}} affected
- User-facing symptom: {{Description}}
- Business impact: {{Description}}

**Start Time**: {{TIMESTAMP}}
**Detection Time**: {{TIMESTAMP}} (MTTD: {{X}} minutes)

---

## Step 1: Gather

### Initial Observations

{{What was the first indicator that something was wrong?}}

### Metrics Check

| Metric | Normal | Current | Delta |
|--------|--------|---------|-------|
| Error Rate | {{X}}% | {{X}}% | {{+/- X%}} |
| Latency P99 | {{X}}ms | {{X}}ms | {{+/- X%}} |
| Throughput | {{X}} req/s | {{X}} req/s | {{+/- X%}} |
| CPU | {{X}}% | {{X}}% | {{+/- X%}} |
| Memory | {{X}}% | {{X}}% | {{X}}% |
| {{Custom}} | {{X}} | {{X}} | {{+/- X%}} |

### Log Analysis

**Error Pattern Detected**:
```
{{Relevant log entries showing the error}}

{{TIMESTAMP}} ERROR: {{MESSAGE}}
{{TIMESTAMP}} ERROR: {{MESSAGE}}
```

**Error Frequency**: {{X errors per minute}}

### Recent Changes

| Change | Time | Who | Type |
|--------|------|-----|------|
| {{Change 1}} | {{TIME}} | {{Name}} | {{DEPLOY|CONFIG|INFRA|DEPENDENCY}} |
| {{Change 2}} | {{TIME}} | {{Name}} | {{DEPLOY|CONFIG|INFRA|DEPENDENCY}} |

---

## Step 2: Diagnose

### Issue Classification

**Primary Issue Type**: {{HIGH_LATENCY | HIGH_ERROR_RATE | TIMEOUT | RESOURCE_EXHAUSTION | DEPENDENCY_FAILURE}}

{{Brief explanation of how the issue was classified based on evidence}}

### Root Cause Analysis

**Root Cause**: {{ROOT_CAUSE_TITLE}}

{{Detailed explanation of what caused the issue}}

**Evidence Chain**:
1. {{Evidence 1}} — {{What this tells us}}
2. {{Evidence 2}} — {{What this tells us}}
3. {{Evidence 3}} — {{What this tells us}}

**Contributing Factors**:
- {{Factor 1}} — {{Brief explanation}}
- {{Factor 2}} — {{Brief explanation}}

### Why Didn't Monitoring Catch This Earlier?

- {{Gap 1}} — {{What metric would have caught it}}
- {{Gap 2}} — {{What threshold would have alerted}}

### Cascading Effects

- {{Effected Service 1}}: {{Impact description}}
- {{Effected Service 2}}: {{Impact description}}

---

## Quick Fix Recommendation

### Recommended Action

**Fix**: {{DESCRIPTION OF FIX}}

### Implementation Steps

1. **{{Step 1}}** — {{What to do}}
   - Command/Action: {{Command or action}}
   - Verification: {{How to verify it worked}}

2. **{{Step 2}}** — {{What to do}}
   - Command/Action: {{Command or action}}
   - Verification: {{How to verify it worked}}

3. **{{Step 3}}**** — {{What to do}}
   - Command/Action: {{Command or action}}
   - Verification: {{How to verify it worked}}

### Rollback Plan (If Fix Involves Changes)

- **Rollback Command**: {{Command}}
- **Rollback Verification**: {{How to verify rollback}}
- **Escalation If Rollback Fails**: {{Contact}}

---

## Verification Steps

### Pre-Fix Baseline

| Metric | Value |
|--------|-------|
| Error Rate | {{X}}% |
| Latency P99 | {{X}}ms |
| Throughput | {{X}} req/s |

### Post-Fix Verification

**Time Verified**: {{TIMESTAMP}} ({{X}} minutes after fix)

| Metric | Expected | Actual | Status |
|--------|----------|--------|--------|
| Error Rate | < {{X}}% | {{X}}% | {{✅/❌}} |
| Latency P99 | < {{X}}ms | {{X}}ms | {{✅/❌}} |
| Throughput | > {{X}} req/s | {{X}} req/s | {{✅/❌}} |

### Verification Actions

- [ ] Smoke test executed: {{PASS|FAIL}}
- [ ] User-facing test: {{PASS|FAIL}}
- [ ] Dependent services confirmed healthy: {{YES|NO}}
- [ ] Alert acknowledged and closed: {{YES|NO}}

**Resolution Status**: {{✅ RESOLVED | ⚠️ PARTIALLY RESOLVED | ❌ NOT RESOLVED}}

---

## Resolution Summary

**Resolution Time (MTTR)**: {{X}} minutes

**Resolution Method**: {{AUTO-REMEDIATED | MANUAL_FIX | ROLLBACK | ESCALATED | CONFIG_CHANGE}}

**Final State**: {{FULLY_RESTORED | PARTIALLY_RESTORED | STILL_IMPACTED}}

{{1-2 sentence summary of how the issue was resolved}}

---

## Follow-up Actions

### Immediate (Prevent Recurrence)

- [ ] **{{Action 1}}** — {{Description}}
  - Owner: {{Name}} | Deadline: {{DATE}}
  - Type: {{PREVENTION}}

- [ ] **{{Action 2}}** — {{Description}}
  - Owner: {{Name}} | Deadline: {{DATE}}
  - Type: {{PREVENTION}}

### Monitoring Improvements

- [ ] Add alert for {{metric}} when {{condition}}
  - Owner: {{Name}} | Deadline: {{DATE}}

- [ ] Adjust threshold for {{existing alert}}
  - Current: {{X}} | Recommended: {{X}}
  - Owner: {{Name}} | Deadline: {{DATE}}

### Runbook Updates

- [ ] Update {{runbook name}} with this incident scenario
  - Owner: {{Name}} | Deadline: {{DATE}}

### Postmortem

- [ ] Conduct postmortem for this incident
  - Owner: {{Name}} | Deadline: {{DATE}}
  - Severity: {{SEV1|SEV2|SEV3}} — Required: {{YES|NO}}

---

## Distribution & Communication

- [x] Stakeholders notified: {{YES ({{Names}}) | NO}}
- [x] Status page updated: {{YES|NO|N/A}}
- [x] Customer communication: {{SENT|NOT NEEDED|PENDING}}
- [x] Action items tracked: {{YES (in {{JIRA/BACKLOG}})|NO}}

---

**Report prepared by**: {{ENGINEER_NAME}}
**Report date**: {{DATE}}
**Follow-up review**: {{DATE}}
