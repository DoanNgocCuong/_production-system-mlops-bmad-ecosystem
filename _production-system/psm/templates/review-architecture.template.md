---
template_name: review-architecture
template_version: "1.0.0"
created_date: 2026-03-19
workflow_id: review-architecture
description: Architecture review report for evaluating system design across scalability, reliability, security, maintainability, and cost
---

# Architecture Review Report

**Service**: {{SERVICE_NAME}}
**Owner**: {{SERVICE_OWNER}}
**Reviewer**: {{ARCHITECTURE_LEAD}} (Minh)
**Review Date**: {{DATE}}
**Architecture Version**: {{VERSION}}
**Review Type**: {{INITIAL | QUARTERLY | AD-HOC}}

---

## Executive Summary

{{1-2 paragraphs summarizing the architecture review, overall assessment, and key recommendations}}

**Overall Assessment**: {{STRONG | ADEQUATE | NEEDS_IMPROVEMENT}}

**Timeline**: Architecture {{is | is conditionally | is not}} ready for production use {{on {{DATE}}}}

---

## Architecture Overview

### System Description

{{High-level description of the system architecture, its purpose, and key components}}

### Architecture Pattern

**Pattern Used**: {{MICROSERVICES | MONOLITH | EVENT-DRIVEN | SERVERLESS | HYBRID | OTHER}}

{{Brief explanation of why this pattern was chosen and how it fits the use case}}

### Component Inventory

| Component | Technology | Purpose | Criticality |
|-----------|------------|---------|-------------|
| {{Component 1}} | {{Tech stack}} | {{Purpose}} | {{HIGH|MED|MEDIUM|LOW}} |
| {{Component 2}} | {{Tech stack}} | {{Purpose}} | {{HIGH|MED|MEDIUM|LOW}} |
| {{Component 3}} | {{Tech stack}} | {{Purpose}} | {{HIGH|MED|MEDIUM|LOW}} |
| {{Component 4}} | {{Tech stack}} | {{Purpose}} | {{HIGH|MED|MEDIUM|LOW}} |

### Architecture Diagram

{{Include or link to system architecture, data flow, and deployment topology}}

---

## 5-Dimension Evaluation

### Scorecard Summary

| Dimension | Score | Status | Key Finding |
|-----------|-------|--------|-------------|
| Scalability | {{GREEN\|YELLOW\|RED}} | ✅/⚠️/❌ | {{Brief finding}} |
| Reliability | {{GREEN\|YELLOW\|RED}} | ✅/⚠️/❌ | {{Brief finding}} |
| Security | {{GREEN\|YELLOW\|RED}} | ✅/⚠️/❌ | {{Brief finding}} |
| Maintainability | {{GREEN\|YELLOW\|RED}} | ✅/⚠️/❌ | {{Brief finding}} |
| Cost | {{GREEN\|YELLOW\|RED}} | ✅/⚠️/❌ | {{Brief finding}} |

**Summary**: {{X}} GREEN, {{Y}} YELLOW, {{Z}} RED

---

### 1. Scalability

**Goal**: Architecture supports horizontal scaling and handles expected growth in traffic, data, and users.

**Findings**:

- [ ] {{Finding 1}} ({{Status}})
- [ ] {{Finding 2}} ({{Status}})
- [ ] {{Finding 3}} ({{Status}})

**Assessment**: {{Detailed narrative on scalability approach, bottlenecks identified, and growth projections}}

**Score**: {{GREEN|YELLOW|RED}}

**Recommendations**:
1. {{Recommendation 1}}
2. {{Recommendation 2}}

---

### 2. Reliability

**Goal**: Architecture has fault tolerance, redundancy, and graceful degradation patterns in place.

**Findings**:

- [ ] {{Finding 1}} ({{Status}})
- [ ] {{Finding 2}} ({{Status}})
- [ ] {{Finding 3}} ({{Status}})

**Assessment**: {{Detailed narrative on reliability patterns, single points of failure, and recovery mechanisms}}

**Score**: {{GREEN|YELLOW|RED}}

**Recommendations**:
1. {{Recommendation 1}}
2. {{Recommendation 2}}

---

### 3. Security

**Goal**: Architecture implements defense-in-depth with secure-by-default patterns.

**Findings**:

- [ ] {{Finding 1}} ({{Status}})
- [ ] {{Finding 2}} ({{Status}})
- [ ] {{Finding 3}} ({{Status}})

**Assessment**: {{Detailed narrative on security architecture, threat model, and compliance considerations}}

**Score**: {{GREEN|YELLOW|RED}}

**Recommendations**:
1. {{Recommendation 1}}
2. {{Recommendation 2}}

---

### 4. Maintainability

**Goal**: Architecture enables clean code, easy deployments, and smooth team collaboration.

**Findings**:

- [ ] {{Finding 1}} ({{Status}})
- [ ] {{Finding 2}} ({{Status}})
- [ ] {{Finding 3}} ({{Status}})

**Assessment**: {{Detailed narrative on code organization, deployment complexity, and team ownership model}}

**Score**: {{GREEN|YELLOW|RED}}

**Recommendations**:
1. {{Recommendation 1}}
2. {{Recommendation 2}}

---

### 5. Cost

**Goal**: Architecture is cost-efficient with clear resource allocation and optimization opportunities.

**Findings**:

- [ ] {{Finding 1}} ({{Status}})
- [ ] {{Finding 2}} ({{Status}})
- [ ] {{Finding 3}} ({{Status}})

**Assessment**: {{Detailed narrative on current costs, projected growth, and cost optimization strategies}}

**Score**: {{GREEN|YELLOW|RED}}

**Recommendations**:
1. {{Recommendation 1}}
2. {{Recommendation 2}}

---

## Overall Trade-off Analysis

### Key Trade-offs Identified

| Decision | Trade-off | Why Acceptable | Risk |
|----------|-----------|---------------|------|
| {{Trade-off 1}} | {{Benefit vs. Cost}} | {{Rationale}} | {{Risk level}} |
| {{Trade-off 2}} | {{Benefit vs. Cost}} | {{Rationale}} | {{Risk level}} |
| {{Trade-off 3}} | {{Benefit vs. Cost}} | {{Rationale}} | {{Risk level}} |

### Architectural Debt

| Item | Severity | Impact | Fix Effort | Owner |
|------|----------|--------|------------|-------|
| {{Debt item 1}} | {{HIGH|MED|LOW}} | {{Description}} | {{Est. time}} | {{Name}} |
| {{Debt item 2}} | {{HIGH|MED|LOW}} | {{Description}} | {{Est. time}} | {{Name}} |

### Strengths

- {{Strength 1}} — This is working well and should be maintained
- {{Strength 2}} — This is working well and should be maintained
- {{Strength 3}} — This is working well and should be maintained

### Gaps

- {{Gap 1}} — {{Impact and recommended approach}}
- {{Gap 2}} — {{Impact and recommended approach}}
- {{Gap 3}} — {{Impact and recommended approach}}

---

## Quarterly Action Plan

### Effort/Impact Matrix

| Action | Effort | Impact | Priority | Quarter |
|--------|--------|--------|----------|---------|
| {{Action 1}} | {{HIGH|MED|LOW}} | {{HIGH|MED|LOW}} | {{P0|P1|P2}} | {{Q1|Q2|Q3|Q4}} |
| {{Action 2}} | {{HIGH|MED|LOW}} | {{HIGH|MED|LOW}} | {{P0|P1|P2}} | {{Q1|Q2|Q3|Q4}} |
| {{Action 3}} | {{HIGH|MED|LOW}} | {{HIGH|MED|LOW}} | {{P0|P1|P2}} | {{Q1|Q2|Q3|Q4}} |
| {{Action 4}} | {{HIGH|MED|LOW}} | {{HIGH|MED|LOW}} | {{P0|P1|P2}} | {{Q1|Q2|Q3|Q4}} |

### Q1 Actions

- [ ] **{{Action 1}}** — {{Description}}
  - Owner: {{Name}}
  - Deadline: {{DATE}}
  - Expected outcome: {{Result}}

### Q2 Actions

- [ ] **{{Action 2}}** — {{Description}}
  - Owner: {{Name}}
  - Deadline: {{DATE}}
  - Expected outcome: {{Result}}

### Q3 Actions

- [ ] **{{Action 3}}** — {{Description}}
  - Owner: {{Name}}
  - Deadline: {{DATE}}
  - Expected outcome: {{Result}}

### Q4 Actions

- [ ] **{{Action 4}}** — {{Description}}
  - Owner: {{Name}}
  - Deadline: {{DATE}}
  - Expected outcome: {{Result}}

---

## Critical Issues (Must Address)

{{If any critical architectural issues exist:}}

### Critical Issue #1: {{ISSUE_TITLE}}

- **Dimension**: {{Which dimension}}
- **Description**: {{What's the problem}}
- **Impact**: {{Why it's critical to the system}}
- **Resolution**: {{How to fix}}
- **Owner**: {{Who must address}}
- **Deadline**: {{When it must be done}}
- **Acceptance Criteria**: {{How we verify the fix}}

### Critical Issue #2: {{ISSUE_TITLE}}

{{Repeat format}}

---

## Recommendations

### Immediate (Next Sprint)

- {{Recommendation 1}}
- {{Recommendation 2}}

### Short-term (This Quarter)

- {{Recommendation 1}}
- {{Recommendation 2}}

### Medium-term (Next 2 Quarters)

- {{Recommendation 1}}
- {{Recommendation 2}}

### Long-term (Next Year)

- {{Recommendation 1}}
- {{Recommendation 2}}

---

## Sign-offs & Approvals

### Approval Chain

- [ ] **Architecture Lead** ({{NAME}}) — Architecture review completed
  - Signature: ________________________ Date: __________

- [ ] **Service Owner** ({{NAME}}) — Acknowledged findings and action plan
  - Signature: ________________________ Date: __________

- [ ] **VP Engineering** ({{NAME}}) — Approved architecture direction
  - Signature: ________________________ Date: __________

---

## Next Review

**Scheduled Review Date**: {{DATE}} ({{QUARTERLY | 6-MONTH | ANNUAL}})

**Review Focus Areas**:
- {{Focus area 1}}
- {{Focus area 2}}
- {{Focus area 3}}

---

## Appendix

### A. Relevant Architecture Decision Records (ADRs)

- {{ADR-001}}: {{Title}} — {{Status}}
- {{ADR-002}}: {{Title}} — {{Status}}

### B. Technology Stack Details

[Detailed technology stack information including versions, dependencies, and rationale]

### C. Performance Benchmarks

[Load test results, benchmark data, and capacity planning assumptions]

### D. Related Documentation

- [Architecture diagrams]
- [Design documents]
- [API specifications]

---

**Report prepared by**: {{ARCHITECTURE_LEAD}}
**Report date**: {{DATE}}
**Last updated**: {{DATE}}
