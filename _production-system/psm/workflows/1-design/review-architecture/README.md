# Review Architecture Workflow

## Overview

**Workflow**: Review Architecture
**ID**: RA001
**Lead Agent**: Khang (Architect)
**Phase**: 1-design
**Status**: Active
**Version**: 1.0.0
**Estimated Duration**: 3-4 hours

## What This Workflow Does

Conducts a comprehensive review of existing architecture decisions and patterns against BMAD best practices. Led by **Khang** (Architect), the workflow evaluates system design across 5 dimensions and produces actionable recommendations.

## 3-Step Execution Path

```
START
  │
  ├── [Step 01] Context ── Gather architecture artifacts, decisions, constraints
  │                    (45-60 min)
  │
  ├── [Step 02] Analysis ── Evaluate against 5 dimensions, score GREEN/YELLOW/RED
  │                    (60-90 min)
  │
  └── [Step 03] Recommendations ── ADRs, trade-offs, priority action plan
                       (45-60 min)
```

## 5 Evaluation Dimensions

1. **Scalability** — Can the system handle 10x traffic? Is growth strategy clear?
2. **Reliability** — Can the system handle failures? Is recovery automatic?
3. **Maintainability** — Is the code organized? Are abstractions clear?
4. **Operational Complexity** — Can a team operate this sustainably? Runbooks exist?
5. **Cost Efficiency** — Are resources used efficiently? Optimization opportunities?

## File Inventory

```
review-architecture/
├── workflow.md                   # Entry point, 3-step overview
├── bmad-skill-manifest.yaml      # BMAD v6.2 manifest
├── checklist.md                  # 23 validation items, 5 categories
├── README.md                     # This file
└── steps/
    ├── step-01-context.md       # Gather architecture artifacts & decisions
    ├── step-02-analysis.md       # 5-dimension scoring & deep analysis
    └── step-03-recommendations.md # ADRs, trade-offs, action plan
```

## Lead Agent: Khang (Architect)

- **Skills**: 5.01 Architecture Principles, 5.02 System Design, 5.03 Database
- **Style**: Calm, pragmatic, systematic
- **Communication**: Vietnamese-English bilingual
- **Role**: Coordinates review, leads dimensional analysis, produces ADRs

## Skill References

| Skill | Domain | Relevance |
|-------|--------|-----------|
| 5.01 | Architecture Principles | System design evaluation |
| 5.02 | System Design Patterns | Scalability & reliability patterns |
| 5.03 | Database | Data architecture, schema, indexing |
| 5.04 | Observability | Metrics, traces, health checks |
| 5.05 | Reliability Engineering | SLOs, SLAs, failure modes |

## Input Requirements

- System architecture diagram (components, relationships, data flows)
- Architecture decision log (why components are designed this way)
- Deployment topology
- Known constraints (budget, team, time)
- Operational context (SLOs, incident history, team size)

## Output Deliverable

**Architecture Review Report** saved to `{project-root}/_bmad-output/psm-artifacts/architecture-review-{{project_name}}-{{date}}.md`:
- 5-dimension assessment with color-coded scores
- Architecture diagram with annotations
- Strengths and areas for improvement
- Architecture Decision Records (ADRs)
- Priority action plan with effort/impact trade-offs

## Success Criteria

1. All 5 dimensions evaluated with clear rationale
2. Strengths of current design identified
3. Improvement opportunities prioritized (effort vs impact)
4. ADRs document key decisions and trade-offs
5. Team alignment on direction for next 6-12 months
