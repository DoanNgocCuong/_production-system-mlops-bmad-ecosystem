---
workflow_id: RA001
workflow_name: Review Architecture
description: In-depth review of existing architecture decisions and patterns
entry_point: steps/step-01-context.md
phase: 1-design
lead_agent: "Khang (Architect)"
status: "active"
created_date: 2026-03-17
version: "1.0.0"
estimated_duration: "3-4 hours"
outputFile: '{output_folder}/psm-artifacts/architecture-review-{{project_name}}-{{date}}.md'
---

# Workflow: Review Architecture

## Goal
Conduct comprehensive architecture review using BMAD patterns, evaluating scalability, reliability, maintainability, and operational complexity.

## Overview

Architecture Review is an in-depth analysis workflow that evaluates system design against architectural best practices:

1. **Gathers** architecture context (diagrams, decisions, constraints)
2. **Analyzes** design against BMAD patterns (scalability, reliability, etc.)
3. **Recommends** improvements with Architecture Decision Records (ADRs)

## Execution Path

```
START
  ↓
[Step 01] Context (Gather diagrams, decisions, constraints)
  ↓
[Step 02] Analysis (Evaluate against patterns, score dimensions)
  ↓
[Step 03] Recommendations (ADRs, trade-offs, action plan)
  ↓
END
```

## Key Roles

| Role | Agent | Responsibility |
|------|-------|-----------------|
| Lead | Khang (Architect) | Coordinate review, lead analysis |
| Subject Matter | Service Architects | Explain design decisions |
| Stakeholders | Tech Leads, DevOps | Provide operational perspective |

## Evaluation Dimensions (5)

1. **Scalability** — Can system handle 10x traffic? Growth strategy clear?
2. **Reliability** — Can system handle failures? Recovery automatic?
3. **Maintainability** — Is code organized? Are abstractions clear?
4. **Operational Complexity** — Can a team operate this? Runbooks exist?
5. **Cost Efficiency** — Are resources used efficiently? Optimization opportunities?

## Input Requirements

- **System architecture diagram** — Components, relationships, data flows
- **Decision log** — Why are components designed this way?
- **Deployment topology** — How is system deployed? Scaling strategy?
- **Known constraints** — What are limitations? Budget constraints?
- **Operational context** — Team size, SLOs, incident history?

## Output Deliverable

- **Architecture Review Report** (template: `architecture-review.template.md`)
  - Dimensional assessment (5 dimensions, color-coded)
  - Strengths and areas for improvement
  - Architecture Decision Records (ADRs) for recommendations
  - Priority action plan with effort/impact trade-offs

## Success Criteria

1. All 5 dimensions evaluated with clear rationale
2. Strengths of current design identified
3. Improvement opportunities prioritized (effort vs impact)
4. ADRs document key decisions and trade-offs
5. Team alignment on direction for next 6-12 months

## Next Steps After Workflow

- Share ADRs with broader team for feedback
- Prioritize recommendations into roadmap
- Plan architecture refactoring work
- Track improvements in follow-up reviews

---

**Navigation**: [← Back to 1-design](../), [Next: Step 01 →](steps/step-01-context.md)
