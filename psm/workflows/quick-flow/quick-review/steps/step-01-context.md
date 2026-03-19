---
step_id: QR-S01
step_name: Context
workflow_id: QR001
sequence: 1
type: "scoping"
agent: "Khang (Architect)"
description: "Gather what to review and load relevant checklist"
estimated_time: "5-10 minutes"
---

# Step 01: Review Context & Scope

## Objective

Understand what needs review and load appropriate checklist:
1. What type of review? (architecture, API, database schema, deployment strategy)
2. What's the context? (problem statement, constraints, goals)
3. Load relevant BMAD skill framework
4. Establish success criteria

## Execution Flow

### Quick Scoping

As Khang (Architect), ask:

```
1. WHAT TYPE OF REVIEW?
   "Are you reviewing architecture, API design, database schema,
    code structure, or deployment strategy?"

2. WHAT'S THE CONTEXT?
   "What problem are you solving? What are the constraints/requirements?"

3. CONCERNS?
   "Any specific areas you're worried about?"

4. TIMELINE?
   "Is this blocking work or planning-phase feedback?"
```

### Load Relevant Checklist

Based on review type, load framework:

**For Architecture Review**:
- Load 5.01 Architectural Patterns
- Check: Scalability, reliability, maintainability, operational complexity

**For API Design Review**:
- Load 5.11 Security
- Check: Input validation, versioning, rate limiting, error handling

**For Database Schema Review**:
- Load 5.14 Data Governance & Storage
- Check: Normalization, indexing strategy, migration plan, backup approach

**For Deployment Strategy Review**:
- Load 5.03 Deployment Patterns
- Check: Blue-green, canary, rollback, smoke tests, SLA targets

### Transition to Step 02

```
"Got it. I'm reviewing [type] against [framework].
Let me apply the checklist and identify key findings."
```

**Proceed to [Step 02](step-02-review.md)**

---

**Navigation**: [← Back to Workflow Overview](../workflow.md), [Next: Step 02 →](step-02-review.md)
