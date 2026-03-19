---
name: 'step-01b-continue'
description: 'Resume interrupted workflow from last completed step'
outputFile: '{output_folder}/psm-artifacts/service-setup-{{project_name}}-{{date}}.md'
---

# Step 1B: Resume Workflow

## STEP GOAL

Resume an interrupted workflow by loading the existing output document, displaying progress, and routing to the next incomplete step.

## MANDATORY EXECUTION RULES

- Read the entire step file before acting
- Speak in `Vietnamese-English`
- NEVER re-execute completed steps

---

## EXECUTION PROTOCOLS

- Follow the MANDATORY SEQUENCE exactly
- Load the next step only when instructed

## CONTEXT BOUNDARIES

- Available context: Output document with progress frontmatter
- Focus: Load progress and route to next step
- Limits: Do not re-execute completed steps
- Dependencies: Output document must exist from a previous run

## MANDATORY SEQUENCE

**CRITICAL:** Follow this sequence exactly. Do not skip, reorder, or improvise.

### 1. Load Output Document

Read `{outputFile}` and parse YAML frontmatter for:

- `stepsCompleted` — array of completed step names
- `lastStep` — last completed step name
- `lastSaved` — timestamp of last save

**If `{outputFile}` does not exist**, display:

"⚠️ **No previous progress found.** There is no output document to resume from. Please use **[C] Create** to start a fresh workflow run."

**THEN:** Halt. Do not proceed.

---

### 2. Display Progress Dashboard

Display:

"📋 **Workflow Resume — PSM Service Setup (SS001)**

**Last saved:** {lastSaved}
**Steps completed:** {stepsCompleted.length} of 6

1. step-01-architecture (Architecture Design) — {✅ if in stepsCompleted, ⬜ otherwise}
2. step-02-api-database (API & Database) — {✅ if in stepsCompleted, ⬜ otherwise}
3. step-03-build-deploy (Build & Deploy) — {✅ if in stepsCompleted, ⬜ otherwise}
4. step-04-reliability (Reliability) — {✅ if in stepsCompleted, ⬜ otherwise}
5. step-05-security-infra (Security & Infra) — {✅ if in stepsCompleted, ⬜ otherwise}
6. step-06-readiness (Production Readiness) — {✅ if in stepsCompleted, ⬜ otherwise}"

---

### 3. Route to Next Step

Based on `lastStep`, load the next incomplete step:

- `'step-01-architecture'` → Load `./step-02-api-database.md`
- `'step-02-api-database'` → Load `./step-03-build-deploy.md`
- `'step-03-build-deploy'` → Load `./step-04-reliability.md`
- `'step-04-reliability'` → Load `./step-05-security-infra.md`
- `'step-05-security-infra'` → Load `./step-06-readiness.md`
- `'step-06-readiness'` → **Workflow already complete.** Display: "✅ **All steps completed.** Use **[V] Validate** to review outputs or **[E] Edit** to make revisions." Then halt.

**If `lastStep` does not match any value above**, display: "⚠️ **Unknown progress state** (`lastStep`: {lastStep}). Please use **[C] Create** to start fresh." Then halt.

**Otherwise**, load the identified step file, read completely, and execute.

The existing content in `{outputFile}` provides context from previously completed steps. Use it as reference for remaining steps.

---

## 🚨 SYSTEM SUCCESS/FAILURE METRICS

### ✅ SUCCESS:

- Output document loaded and parsed correctly
- Progress dashboard displayed accurately
- Routed to correct next step

### ❌ SYSTEM FAILURE:

- Not loading output document
- Incorrect progress display
- Routing to wrong step
- Re-executing completed steps

**Master Rule:** Resume MUST route to the exact next incomplete step. Never re-execute completed steps.
