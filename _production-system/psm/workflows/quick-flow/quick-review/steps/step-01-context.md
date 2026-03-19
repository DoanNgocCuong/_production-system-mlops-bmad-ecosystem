---
step_id: QR-S01
step_name: Context
workflow_id: QR001
sequence: 1
type: "scoping"
agent: "Khang (Architect)"
description: "Gather what to review and load relevant checklist"
estimated_time: "5-10 minutes"
is_final_step: false
outputFile: "{project-root}/psm/outputs/quick-review/{review_id}/review-report.md"
workflowType: "quick-review"
---

# Step 01: Review Context & Scope

## Objective

Understand what needs review and load appropriate checklist:
1. What type of review? (architecture, API, database schema, deployment strategy)
2. What's the context? (problem statement, constraints, goals)
3. Load relevant BMAD skill framework
4. Establish success criteria

## BMAD v6.2 Checkpoint Protocol

**Before proceeding, ensure output document exists with YAML frontmatter:**
```yaml
---
stepsCompleted: []
lastStep: "step-01-context.md"
lastSaved: "{current-timestamp}"
workflowType: "quick-review"
---
```

**SAVING RULE:** After completing all execution sections, BEFORE loading the next step:
1. Update output document: append current step's output to the document
2. Update frontmatter: add this step to `stepsCompleted`, update `lastStep` and `lastSaved`
3. Save the document

---

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

## Checkpoint

**Select an Option:**
**[C] Continue to step-02-review.md**
**[A] Advanced Elicitation**
**[P] Party Mode**
**[Y] YOLO**

Menu Handling:
- IF C: Update frontmatter (add this step to stepsCompleted, update lastStep, lastSaved), save document, then read fully and follow `./step-02-review.md`
- IF A: Perform advanced elicitation on current step output
- IF P: Invite other agents to party mode
- IF Y: Proceed without stopping
- IF Any other: respond and redisplay menu

**EXECUTION RULES:**
- 🛑 ALWAYS halt and wait for user input after presenting checkpoint menu
- 🛑 NEVER load the next step file until user selects 'C'
- 🛑 ALWAYS update frontmatter BEFORE loading next step
- 📖 Read the entire next step file before execution
