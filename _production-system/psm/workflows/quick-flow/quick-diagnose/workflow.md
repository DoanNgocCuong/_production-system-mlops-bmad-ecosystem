---
workflow_id: QD001
workflow_name: Quick Diagnose
description: Fast diagnosis of production issue with root cause and fix suggestion
entry_point: steps/step-01-gather.md
phase: quick-flow
lead_agent: "Minh (SRE)"
status: "active"
created_date: 2026-03-17
version: "1.0.0"
estimated_duration: "15-25 minutes"
outputFile: '{output_folder}/psm-artifacts/quick-diagnose-{{date}}.md'
---

# Workflow: Quick Diagnose Production Issue

## Goal
Rapidly diagnose production issues by gathering symptom data, checking metrics, and suggesting fixes.

## Overview

Quick Diagnose is a lightweight workflow for time-sensitive production troubleshooting:

1. **Gathers** symptom description and quick metrics check
2. **Diagnoses** root cause using observability data
3. **Suggests** fix or mitigation immediately

## Execution Path

```
START
  ↓
[Step 01] Gather Context (What's broken? Check metrics)
  ↓
[Step 02] Diagnose & Fix (Root cause analysis → fix suggestion → verify)
  ↓
END
```

## Key Roles

| Role | Agent |
|------|-------|
| Lead | Minh (SRE) |

## Input Requirements

- **Symptom description** — What is failing? (error message, behavior, timeline)
- **Affected service/component** — What system is broken?
- **Timeline** — When did it start? Is it ongoing?
- **Impact** — How many users affected? Is revenue impacted?

## Output Deliverable

- **Quick Diagnosis Report** (markdown, 1-2 pages)
  - Symptom analysis
  - Root cause hypothesis
  - Immediate mitigation (if needed)
  - Fix suggestion with effort
  - Follow-up actions

## Success Criteria

1. Root cause identified within 15-20 minutes
2. Immediate mitigation available (if needed)
3. Fix suggestion documented with clear steps
4. Team knows what to do next

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

## Quick Diagnose vs Full Production Readiness Review

| Aspect | Quick Diagnose | Full PRR |
|--------|---|---|
| Trigger | Active incident | Pre-deployment |
| Duration | 15-25 min | 2-3 hours |
| Scope | Single issue | All 9 dimensions |
| Goal | Fix now | Prevent issues |

---

**Navigation**: [← Back to quick-flow](../), [Next: Step 01 →](steps/step-01-gather.md)
