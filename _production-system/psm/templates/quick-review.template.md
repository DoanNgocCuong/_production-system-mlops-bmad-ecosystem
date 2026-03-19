---
template_name: quick-review
template_version: "1.0.0"
created_date: 2026-03-19
workflow_id: quick-review
description: Quick review findings report for rapid assessment of system health, code quality, or configuration
---

# Quick Review Report

**Review Subject**: {{SERVICE_NAME | CODE_REPOSITORY | CONFIGURATION | ARCHITECTURE}}
**Reviewer**: {{REVIEWER_NAME}}
**Review Date**: {{DATE}}
**Review Type**: {{CODE_REVIEW | CONFIG_REVIEW | HEALTH_CHECK | SECURITY_SCAN | DEPENDENCY_AUDIT | OTHER}}
**Duration**: {{DURATION_MINUTES}} minutes

---

## Executive Summary

{{1-2 paragraph summary of what was reviewed and key findings}}

**Overall Assessment**: {{✅ STRONG | ⚠️ ADEQUATE | ❌ NEEDS_IMPROVEMENT}}

**Recommendation**: {{APPROVE | REQUEST_CHANGES | REJECT | DEFER}}

---

## Review Scope

### What Was Reviewed

- {{Scope item 1}}
- {{Scope item 2}}
- {{Scope item 3}}

### What Was Excluded

- {{Excluded item 1}} ({{Reason}})
- {{Excluded item 2}} ({{Reason}})

### Review Criteria

{{Based on {{COMPANY_STANDARDS|OPEN_SOURCE_BEST_PRACTICES|INTERNAL_POLICIES|OTHER}}:}}

- {{Criterion 1}}
- {{Criterion 2}}
- {{Criterion 3}}

---

## Strengths

### What Is Working Well

**1. {{Strength Title}}**

{{Description of the strength and why it is effective}}

**Evidence**: {{Specific example, file, or metric}}

---

**2. {{Strength Title}}**

{{Description of the strength and why it is effective}}

**Evidence**: {{Specific example, file, or metric}}

---

**3. {{Strength Title}}**

{{Description of the strength and why it is effective}}

**Evidence**: {{Specific example, file, or metric}}

---

### Positive Patterns

- {{Pattern 1}} — Well implemented, consider applying elsewhere
- {{Pattern 2}} — Well implemented, consider applying elsewhere
- {{Pattern 3}} — Well implemented, consider applying elsewhere

---

## Gaps

### Issues Identified

**Issue #1: {{ISSUE_TITLE}}**

- **Severity**: {{HIGH | MEDIUM | LOW | INFO}}
- **Category**: {{CODE_QUALITY | SECURITY | PERFORMANCE | CONFIGURATION | DOCUMENTATION | TESTING}}
- **Location**: {{File path, line number, or component}}
- **Description**: {{Detailed description of the issue}}
- **Impact**: {{Why this matters, what could go wrong}}
- **Recommendation**: {{How to fix}}
- **Effort**: {{LOW | MEDIUM | HIGH}}

---

**Issue #2: {{ISSUE_TITLE}}**

- **Severity**: {{HIGH | MEDIUM | LOW | INFO}}
- **Category**: {{CODE_QUALITY | SECURITY | PERFORMANCE | CONFIGURATION | DOCUMENTATION | TESTING}}
- **Location**: {{File path, line number, or component}}
- **Description**: {{Detailed description of the issue}}
- **Impact**: {{Why this matters, what could go wrong}}
- **Recommendation**: {{How to fix}}
- **Effort**: {{LOW | MEDIUM | HIGH}}

---

**Issue #3: {{ISSUE_TITLE}}**

- **Severity**: {{HIGH | MEDIUM | LOW | INFO}}
- **Category**: {{CODE_QUALITY | SECURITY | PERFORMANCE | CONFIGURATION | DOCUMENTATION | TESTING}}
- **Location**: {{File path, line number, or component}}
- **Description**: {{Detailed description of the issue}}
- **Impact**: {{Why this matters, what could go wrong}}
- **Recommendation**: {{How to fix}}
- **Effort**: {{LOW | MEDIUM | HIGH}}

---

### Summary by Severity

| Severity | Count | Blocking? |
|----------|-------|-----------|
| HIGH | {{X}} | {{YES|NO}} |
| MEDIUM | {{X}} | {{YES|NO}} |
| LOW | {{X}} | {{NO}} |
| INFO | {{X}} | {{NO}} |

---

## Trade-off Analysis

### Decisions Made (With Rationale)

| Decision | Trade-off | Rationale | Acceptable? |
|----------|-----------|-----------|-------------|
| {{Decision 1}} | {{Benefit vs. Cost or Risk}} | {{Why this was chosen}} | {{YES|PARTIALLY|NO}} |
| {{Decision 2}} | {{Benefit vs. Cost or Risk}} | {{Why this was chosen}} | {{YES|PARTIALLY|NO}} |
| {{Decision 3}} | {{Benefit vs. Cost or Risk}} | {{Why this was chosen}} | {{YES|PARTIALLY|NO}} |

### Acceptable Trade-offs

These trade-offs are reasonable given the current constraints:

1. **{{Trade-off}}** — {{Why acceptable}}
   - Benefit: {{What we gain}}
   - Cost: {{What we give up}}
   - Mitigation: {{Any monitoring or safeguards}}

### Trade-offs to Revisit

These should be reconsidered in the future:

1. **{{Trade-off}}** — {{Why should revisit}}
   - Current: {{Current approach}}
   - Future: {{Better approach when {{condition}}}}
   - Trigger: {{When to revisit}}

---

## Recommendations

### Critical (Must Address)

These issues block approval or must be addressed immediately:

- [ ] **{{Recommendation 1}}**
  - Owner: {{Name}}
  - Deadline: {{DATE}}
  - Type: {{BLOCKING}}

- [ ] **{{Recommendation 2}}**
  - Owner: {{Name}}
  - Deadline: {{DATE}}
  - Type: {{BLOCKING}}

### High Priority (Before Next Release)

- [ ] **{{Recommendation 1}}**
  - Owner: {{Name}}
  - Deadline: {{DATE}}

### Medium Priority (Within 1 Sprint)

- [ ] {{Recommendation 1}} — Owner: {{Name}}
- [ ] {{Recommendation 2}} — Owner: {{Name}}

### Low Priority / Nice to Have

- [ ] {{Recommendation 1}} — Owner: {{Name}}
- [ ] {{Recommendation 2}} — Owner: {{Name}}

---

## Action Items

| ID | Action | Severity | Owner | Deadline | Status |
|----|--------|----------|-------|----------|--------|
| A1 | {{Action}} | {{HIGH|MED|LOW|INFO}} | {{Name}} | {{Date}} | ☐ |
| A2 | {{Action}} | {{HIGH|MED|LOW|INFO}} | {{Name}} | {{Date}} | ☐ |
| A3 | {{Action}} | {{HIGH|MED|LOW|INFO}} | {{Name}} | {{Date}} | ☐ |

---

## Sign-off

### Reviewer

**Reviewer**: {{REVIEWER_NAME}}
**Assessment**: {{APPROVED | APPROVED WITH CONDITIONS | REQUEST_CHANGES | REJECTED}}
**Date**: {{DATE}}

### Conditions (If Approved with Conditions)

1. **{{Condition 1}}** — {{Description}}
   - Owner: {{Name}} | Deadline: {{DATE}}

2. **{{Condition 2}}** — {{Description}}
   - Owner: {{Name}} | Deadline: {{DATE}}

### Acknowledgment

- [ ] **Author/Owner** ({{NAME}}) — Reviewed findings and committed to action plan
  - Signature: ________________________ Date: __________

- [ ] **{{Approver 2}}** ({{NAME}}) — Second-level approval
  - Signature: ________________________ Date: __________

---

## Appendix

### Files Reviewed

| File | Path | Findings |
|------|------|----------|
| {{File 1}} | {{Path}} | {{X HIGH, Y MED, Z LOW}} |
| {{File 2}} | {{Path}} | {{X HIGH, Y MED, Z LOW}} |
| {{File 3}} | {{Path}} | {{X HIGH, Y MED, Z LOW}} |

### Tool Output / Scan Results

{{If applicable:}}

```
{{Tool scan results, security findings, lint output, etc.}}
```

### Related Documents

- [Link to related design document]
- [Link to related architecture ADR]
- [Link to related runbook]

---

**Report prepared by**: {{REVIEWER_NAME}}
**Report date**: {{DATE}}
