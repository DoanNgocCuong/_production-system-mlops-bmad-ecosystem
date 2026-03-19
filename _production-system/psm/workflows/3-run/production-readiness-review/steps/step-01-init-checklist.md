---
step_id: PRR-S01
step_name: Init Checklist
workflow_id: PRR001
sequence: 1
type: "initialization"
agent: "Minh (SRE)"
description: "Load framework, present PRR dimensions, gather service context"
estimated_time: "20-30 minutes"
is_final_step: false
outputFile: "{project-root}/psm/outputs/production-readiness-review/{service_name}/prr-report.md"
workflowType: "production-readiness-review"
---

# Step 01: Initialize Production Readiness Review

## Objective

Initialize the PRR process by:
1. Loading the Production Readiness framework (9 dimensions)
2. Presenting dimensional overview to participant
3. Gathering service context and basic information
4. Establishing baseline understanding for deep review in Step 02

## BMAD v6.2 Checkpoint Protocol

**Before proceeding, ensure output document exists with YAML frontmatter:**
```yaml
---
stepsCompleted: []
lastStep: "step-01-init-checklist.md"
lastSaved: "{current-timestamp}"
workflowType: "production-readiness-review"
---
```

**SAVING RULE:** After completing all execution sections, BEFORE loading the next step:
1. Update output document: append current step's output to the document
2. Update frontmatter: add this step to `stepsCompleted`, update `lastStep` and `lastSaved`
3. Save the document

---

## Execution Flow

### Substep 1A: Load Framework & Introduce Minh

As SRE Minh, greet the participant and introduce the PRR process:

```
"Hi! I'm Minh, your SRE for this production readiness review.

Before we can confidently run your service in production, we need to validate
readiness across 9 critical dimensions. This isn't about perfection—it's about
understanding risks and ensuring we have mitigation plans in place.

Let me walk you through the dimensions we'll evaluate today..."
```

Present the 9 dimensions with brief explanations:

1. **Reliability** — Can we meet our SLO? Do we understand failure modes?
2. **Observability** — Can we see what's happening in production?
3. **Performance** — Will performance degrade under expected load?
4. **Security** — Can we defend against threats and comply with policy?
5. **Capacity** — Do we have resources for peak demand and growth?
6. **Data** — Can we recover data? Do we govern it properly?
7. **Runbooks** — Can operators respond to incidents without escalation?
8. **Dependencies** — Are external/internal dependencies stable and contractual?
9. **Rollback** — Can we safely undo a bad deployment?

### Substep 1B: Gather Service Context

Ask the participant to provide (at least briefly):

```
1. Service Name & Owner
   - What is the service called?
   - Who is the primary owner/maintainer?

2. Purpose & Criticality
   - What does the service do?
   - How critical is it? (P0=revenue-blocking, P1=degraded, P2=nice-to-have)

3. Current Architecture
   - Tech stack? (language, framework, database, message queue, etc.)
   - High-level topology? (monolith vs microservices, stateless vs stateful)
   - Key external dependencies? (payment gateway, auth provider, data warehouse)

4. Existing Metrics/Monitoring
   - What dashboards/alerts exist today?
   - Any defined SLOs? Targets?
   - Known issues or risks?
```

Document responses in a scratch pad for reference during Step 02.

### Substep 1C: Present PRR Scoring System

Explain the scoring rubric participants will see in Step 02:

```
RED (❌)     — Blocker. Service CANNOT go to production. Needs fixes.
YELLOW (⚠️)  — Risk. Service can go, but with documented caveats and monitoring.
GREEN (✅)   — Ready. No blockers identified. Proceed with confidence.

Blockers are categorized:
- P0 (Must Fix)  — Hold release until resolved
- P1 (Should Fix) — Can proceed if documented and monitored
```

### Substep 1D: Alignment Check & Transition

Confirm participant is ready to proceed:

```
"Great! I have a good understanding of your service now.

In Step 02, I'll do a deep technical review of each dimension:
- Checking your service against architectural patterns and best practices
- Scoring RED/YELLOW/GREEN for each dimension
- Identifying concrete blockers and recommendations
- Building a scorecard we'll use in Step 03 for the final decision

Let's move to the deep review. Ready?"
```

**On confirmation → Proceed to [Step 02](step-02-deep-review.md)**

## Deliverables from This Step

1. **Service Context Document** (markdown or structured form)
   - Service name, owner, purpose, criticality
   - Architecture overview (tech stack, topology, dependencies)
   - Links to existing dashboards/runbooks if available

2. **Dimensional Framework Understanding**
   - Participant understands 9 dimensions
   - Participant understands RED/YELLOW/GREEN scoring
   - Minh has context for deep review in Step 02

3. **Readiness Assessment Starter**
   - Initial sense of where risks likely are
   - Any known pain points documented

## Key Questions to Ask

- "What's kept you up at night about deploying this service?"
- "If this service went down right now, how would you respond? Do you have a runbook?"
- "What's your current SLO target? How confident are you in hitting it?"
- "Do you have a rollback plan if a deployment goes wrong?"

## Troubleshooting

| Issue | Resolution |
|-------|-----------|
| Participant unfamiliar with SLOs | Explain: "An SLO is a promise to users. e.g., 'We'll be available 99.9% of the time.'" |
| Limited existing metrics | Note as YELLOW/RED in Observability dimension; plan to add in Step 02 |
| No documented architecture | Schedule brief architecture discussion; may need async follow-up |

## Rollback

If participant is not ready to proceed to Step 02:
- Save context document for later
- Offer to reconvene after they've gathered more information
- Note any prerequisite information needed

---

## Checkpoint

**Select an Option:**
**[C] Continue to step-02-deep-review.md**
**[A] Advanced Elicitation**
**[P] Party Mode**
**[Y] YOLO**

Menu Handling:
- IF C: Update frontmatter (add this step to stepsCompleted, update lastStep, lastSaved), save document, then read fully and follow `./step-02-deep-review.md`
- IF A: Perform advanced elicitation on current step output
- IF P: Invite other agents to party mode
- IF Y: Proceed without stopping
- IF Any other: respond and redisplay menu

**EXECUTION RULES:**
- 🛑 ALWAYS halt and wait for user input after presenting checkpoint menu
- 🛑 NEVER load the next step file until user selects 'C'
- 🛑 ALWAYS update frontmatter BEFORE loading next step
- 📖 Read the entire next step file before execution
