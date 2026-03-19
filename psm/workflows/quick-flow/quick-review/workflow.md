---
workflow_id: QR001
workflow_name: Quick Review
description: Quick architecture or code review leveraging BMAD frameworks
entry_point: steps/step-01-context.md
phase: quick-flow
lead_agent: "Khang (Architect)"
status: "active"
created_date: 2026-03-17
version: "1.0.0"
estimated_duration: "20-30 minutes"
outputFile: '{output_folder}/psm-artifacts/quick-review-{{date}}.md'
---

# Workflow: Quick Review

## Goal
Rapidly review architecture or code against BMAD patterns and best practices.

## Overview

Quick Review provides lightweight architectural and code guidance:

1. **Gathers** context on what needs review
2. **Applies** relevant BMAD review checklist
3. **Reports** findings and recommendations

## Execution Path

```
START
  ↓
[Step 01] Context (What to review? Load checklist)
  ↓
[Step 02] Review (Apply checklist → findings → recommendations)
  ↓
END
```

## Key Roles

| Role | Agent |
|------|-------|
| Lead | Khang (Architect) |

## Input Requirements

- **What to review** — Architecture diagram, code, design document, API spec
- **Context** — What problem are you solving? What constraints?
- **Concerns** — Any specific areas of concern?

## Output Deliverable

- **Quick Review Report** (markdown, 1-2 pages)
  - Findings (strengths and gaps)
  - Recommendations (prioritized)
  - Trade-off analysis (if applicable)

## Success Criteria

1. Clear understanding of design
2. Key findings identified (3-5 items)
3. Actionable recommendations provided
4. Trade-offs documented

---

**Navigation**: [← Back to quick-flow](../), [Next: Step 01 →](steps/step-01-context.md)
