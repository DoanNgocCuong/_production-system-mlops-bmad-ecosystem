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

## Entry Point Logic

### Continuation Detection (BMAD v6.2)

**MUST check in this order:**

1. **Check for existing session**: Look for output document at `{outputFile}`
   - IF exists AND has YAML frontmatter with `stepsCompleted` AND workflow NOT complete:
     → **AUTO-PROCEED**: Load `./steps/step-01b-continue.md` (no user choice needed)
   - IF exists AND is complete (all steps done):
     → Display completion message, offer [R] Review / [N] New
   - IF no document or no stepsCompleted:
     → Fresh workflow: start from step-01

2. **For each step execution:**
   - Load step file completely
   - Execute all numbered sections
   - Present checkpoint menu at end
   - Update output document frontmatter (stepsCompleted, lastStep, lastSaved)
   - Wait for user selection before proceeding

---

**Navigation**: [← Back to quick-flow](../), [Next: Step 01 →](steps/step-01-context.md)
