---
step_id: PRR-S03
step_name: Final Decision
workflow_id: PRR001
sequence: 3
type: "decision"
agent: "Minh (SRE)"
description: "Render scorecard, make GO/NO-GO/CONDITIONAL-GO decision, action items"
estimated_time: "30-40 minutes"
is_final_step: true
outputFile: "{project-root}/psm/outputs/production-readiness-review/{service_name}/prr-report.md"
workflowType: "production-readiness-review"
---

# Step 03: Final Production Readiness Decision

## Objective

Synthesize findings from Step 02 and render a clear, data-driven decision:
- Scorecard summary (9 dimensions, color coding)
- GO / NO-GO / CONDITIONAL-GO recommendation
- Prioritized action items for any gaps
- Sign-off and next steps

## BMAD v6.2 Checkpoint Protocol

**Before proceeding, ensure output document exists with YAML frontmatter:**
```yaml
---
stepsCompleted: ["step-01-init-checklist.md", "step-02-deep-review.md"]
lastStep: "step-03-final-decision.md"
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

### Substep 3A: Present Scorecard Summary

Display results from Step 02 in clear format:

```markdown
## Production Readiness Scorecard

| # | Dimension | Score | Status |
|---|-----------|-------|--------|
| 1 | Reliability | ✅ GREEN | SLO defined, error budget tracked, incident response plan ready |
| 2 | Observability | ⚠️ YELLOW | Logs and metrics present, dashboards incomplete |
| 3 | Performance | ✅ GREEN | Load tested, P99 acceptable, scaling strategy proven |
| 4 | Security | ⚠️ YELLOW | Auth/authz implemented, dependency scanning needed |
| 5 | Capacity | ✅ GREEN | Resource limits known, auto-scaling active |
| 6 | Data | ✅ GREEN | Backups tested, restore time acceptable, PII secured |
| 7 | Runbooks | ⚠️ YELLOW | Incident runbook partial, rollback procedure untested |
| 8 | Dependencies | ✅ GREEN | Dependencies mapped, fallback strategies in place |
| 9 | Rollback | ✅ GREEN | Feature flags active, blue-green capable, smoke tests ready |

**Summary**: 6 GREEN, 3 YELLOW, 0 RED
**Risk Level**: LOW to MODERATE
**Confidence**: CONDITIONAL
```

### Substep 3B: Categorize Blockers

Separate issues by severity:

#### P0 Blockers (Must Fix Before Production)

```markdown
### P0 Blockers — Must Resolve Before Deployment

P0 blockers are issues that prevent safe production operation. These must be
resolved before GO decision is granted.

**None identified.** All critical gaps are P1 (should fix).
```

OR (if blockers exist):

```markdown
### P0 Blockers — Must Resolve Before Deployment

**[B1] Database Backup Restore Untested**
- Dimension: Data
- Issue: Backups exist but restore process has never been tested
- Risk: Data loss unrecoverable in disaster scenario
- Resolution: Test restore from backup to development environment
- Owner: DBA or Data Engineering Lead
- Deadline: 2026-03-24 (before prod deployment)
- Acceptance Criteria: Restore completes in < 30 minutes, data integrity verified
```

#### P1 Risks (Should Fix, Can Document)

```markdown
### P1 Risks — Should Fix (Can Proceed with Mitigation)

P1 issues should be resolved but can proceed to production with documented
monitoring and contingency plans. Add to backlog, target resolution within
sprint or two.

**[R1] Dashboard Coverage Incomplete**
- Dimension: Observability
- Issue: No dashboard for cache hit/miss rates, only request/error/latency
- Impact: Harder to diagnose performance issues
- Mitigation: Add dashboard before peak traffic; monitor manually if needed
- Owner: DevOps or SRE
- Target Deadline: 2026-03-31 (first sprint after go-live)

**[R2] Dependency Circuit Breaker Testing Limited**
- Dimension: Dependencies
- Issue: Circuit breaker logic designed but not chaos-tested
- Impact: Unknown behavior under actual dependency failure
- Mitigation: Setup monitoring on dependency health; have rollback ready
- Owner: Platform Engineering
- Target Deadline: 2026-04-07 (within 3 weeks)

**[R3] Runbook Rollback Procedure Not Practiced**
- Dimension: Runbooks
- Issue: Rollback runbook written but team hasn't drilled it
- Impact: On-call engineer may be uncertain during actual rollback
- Mitigation: Schedule runbook drill before first major deployment
- Owner: SRE (Minh) & Platform Engineering
- Target Deadline: 2026-03-24 (before first production traffic)
```

### Substep 3C: Render Final Decision

Based on blockers and risks, issue one of three verdicts:

#### Decision Tree

```
Do we have P0 blockers?
  ├─ YES → Render "NO-GO" decision
  └─ NO
      └─ Do we have P1 risks we can actively mitigate?
          ├─ NONE → Render "GO" decision
          └─ YES  → Render "CONDITIONAL-GO" decision with conditions
```

#### Decision 1: GO ✅

```markdown
## FINAL DECISION: GO ✅

**Decision**: Service is APPROVED for production deployment.

**Rationale**:
- 6 dimensions scored GREEN (Reliability, Performance, Capacity, Data, Dependencies, Rollback)
- 3 dimensions scored YELLOW with manageable risks
- 0 P0 blockers identified
- P1 risks documented and actionable
- Team has demonstrated operational readiness
- Confidence level: HIGH

**Approval Chain**:
- [ ] SRE Minh (author of this PRR)
- [ ] Architecture Lead (Khang)
- [ ] Service Owner ({{SERVICE_OWNER}})
- [ ] Product/Leadership ({{APPROVER}})

**Next Steps**:
1. Schedule deployment window (coordinate with Product, Support)
2. Brief on-call team on service architecture and runbooks
3. Setup monitoring dashboards and alerts (ensure ready day-of)
4. Execute deployment per deployment runbook
5. Monitor close for first 24 hours post-deployment
6. Capture lessons learned; update runbooks based on real operations
```

#### Decision 2: NO-GO ❌

```markdown
## FINAL DECISION: NO-GO ❌

**Decision**: Service is NOT APPROVED for production. Must resolve blockers.

**Rationale**:
- P0 blockers identified (see "P0 Blockers" section above)
- Blockers prevent safe production operation
- Risk of data loss, security breach, or unrecoverable outage is unacceptable
- Confidence level: CRITICAL

**Blockers to Resolve**:
1. [B1] Description — Owner, Deadline
2. [B2] Description — Owner, Deadline
3. [B3] Description — Owner, Deadline

**Approval Required to Reschedule PRR**:
- Once all P0 blockers resolved and verified in staging/test
- SRE will re-evaluate and issue new PRR (can be abbreviated)

**Follow-up Timeline**:
- Blockers assigned to owners with deadlines (typically 5-10 days)
- Team synchronizes mid-week to track progress
- Revised PRR scheduled once blockers completed (1 week from now)
- Target go-live: [NEW_DATE]
```

#### Decision 3: CONDITIONAL-GO ⚠️

```markdown
## FINAL DECISION: CONDITIONAL-GO ⚠️

**Decision**: Service is APPROVED for production WITH CONDITIONS.

**Rationale**:
- No P0 blockers identified
- 3 P1 risks present but actively mitigated
- Confidence level: MODERATE (acceptable with risk acknowledgment)
- Benefits of deploying now outweigh risks of delaying

**Conditions for Deployment**:

1. **Observability**: Dashboard coverage must be completed within first sprint
   - Acceptance: Cache metrics visible on dashboard
   - Monitoring: If not ready, manual monitoring of cache behavior
   - Owner: DevOps

2. **Runbooks**: Rollback procedure must be drilled before first major production change
   - Acceptance: Team performs 30-minute drill, validates timing
   - Monitoring: Minh (SRE) reviews drill notes
   - Owner: SRE (Minh)

3. **Operational Oversight**: Close SRE monitoring for first 2 weeks
   - Daily standups with ops team to review metrics
   - Weekly review of any anomalies
   - Escalation path to Minh (on-call SRE)
   - Owner: Minh (SRE)

**Approval Chain**:
- [ ] SRE Minh (author of this PRR)
- [ ] Architecture Lead (Khang)
- [ ] Service Owner ({{SERVICE_OWNER}})
- [ ] VP Engineering (Approves conditional risk) — {{APPROVER}}

**Deployment Requirements**:
- [ ] Conditions explicitly acknowledged by all approvers
- [ ] Risk mitigation checklist signed off
- [ ] Escalation contact clearly defined for operations team
- [ ] Conditions tracked in backlog; progress monitored

**Next Steps**:
1. Obtain explicit sign-off on conditions from all approvers
2. Schedule deployment per deployment runbook
3. Execute deployment with heightened monitoring
4. Daily standups with ops team (first 2 weeks)
5. Close P1 risks per timeline; report progress in standups
6. Transition to normal operational cadence once P1s resolved
```

### Substep 3D: Action Items & Ownership

Create explicit action plan:

```markdown
## Action Items — Ownership & Deadlines

### Critical Path (For CONDITIONAL-GO Conditions)

| ID | Action | Owner | Deadline | Priority | Status |
|----|--------|-------|----------|----------|--------|
| A1 | Drill rollback procedure | Minh (SRE) | 2026-03-24 | P0 | ☐ Pending |
| A2 | Implement dashboard cache metrics | DevOps | 2026-03-31 | P1 | ☐ Pending |
| A3 | Chaos test circuit breaker | Platform Eng | 2026-04-07 | P1 | ☐ Pending |

### Long-term Backlog

| ID | Action | Owner | Suggested Sprint | Notes |
|----|--------|-------|------------------|-------|
| B1 | Enhance dependency monitoring | DevOps | Sprint 2026-04-XX | Track P/F ratio per dependency |
| B2 | Automated smoke test suite | QA | Sprint 2026-05-XX | Pre-deployment validation |
| B3 | Runbook health checks | Minh (SRE) | Ongoing | Monthly review + update |
```

### Substep 3E: Sign-off & Communication

Document approvals and communicate decision:

```markdown
## Sign-offs & Approvals

**Prepared by**: Minh (SRE)
**Date**: 2026-03-17
**Service**: {{SERVICE_NAME}}
**Owner**: {{SERVICE_OWNER}}

### Approval Signatures

- **SRE**: ☐ Minh — ___________ (date)
- **Architecture**: ☐ Khang — ___________ (date)
- **Service Owner**: ☐ {{SERVICE_OWNER}} — ___________ (date)
- **VP Eng** (if conditional): ☐ {{APPROVER}} — ___________ (date)

### Distribution

- [x] Email PRR to service owner + team (decision summary)
- [x] Schedule post-PRR sync (30 min) with team to answer questions
- [x] Add P0/P1 action items to backlog (link in PRR)
- [x] Schedule deployment kickoff meeting (if GO/CONDITIONAL-GO)
```

---

## Deliverables from This Step

**Output**: Completed PRR Report (use `production-readiness.template.md`)

```
├─ PRR Summary (1-2 pages)
│  ├─ Service overview
│  ├─ Scorecard (9 dimensions, color-coded)
│  ├─ P0 blockers (if any)
│  ├─ P1 risks (if any)
│  └─ Final decision (GO/NO-GO/CONDITIONAL-GO)
│
├─ Detailed Findings (per-dimension analysis)
│  ├─ Reliability
│  ├─ Observability
│  ├─ Performance
│  ├─ Security
│  ├─ Capacity
│  ├─ Data
│  ├─ Runbooks
│  ├─ Dependencies
│  └─ Rollback
│
├─ Action Items (prioritized, assigned, deadlined)
│
└─ Sign-offs (approvals documented)
```

## Success Criteria

- [ ] All 9 dimensions scored with clear rationale
- [ ] Blockers (P0 & P1) categorized and prioritized
- [ ] Final decision clearly stated (GO/NO-GO/CONDITIONAL-GO)
- [ ] If CONDITIONAL, conditions explicitly listed and approved
- [ ] Action items assigned with clear owners and deadlines
- [ ] All required stakeholders have signed off
- [ ] Service owner understands next steps

## Workflow Complete

This is the **FINAL STEP** of the Production Readiness Review workflow.

```
"Production Readiness Review complete!

Steps Completed:
  1. Step 01: Initialize PRR (service context, dimensional framework)
  2. Step 02: Deep Technical Review (9 dimensions scored)
  3. Step 03: Final Decision (GO/NO-GO/CONDITIONAL-GO)

Outcome:
- ✅ Service is approved for production (GO or CONDITIONAL-GO), OR
- ❌ Service requires blockers to be resolved before re-evaluation (NO-GO)

Next Workflow: Proceed to Production Deployment or Security Audit as needed."

Production Readiness Review Workflow: COMPLETE
```

**Next Steps**:
1. Share PRR report with all stakeholders
2. If CONDITIONAL-GO: ensure all conditions are tracked and monitored
3. If NO-GO: schedule follow-up PRR once blockers are resolved
4. Proceed to Production Deployment or Security Audit workflows as needed

---

## Checkpoint

**Select an Option:**
**[C] Complete Workflow**
**[A] Advanced Elicitation**
**[P] Party Mode**
**[Y] YOLO**

Menu Handling:
- IF C: Update frontmatter (add this step to stepsCompleted, update lastStep, lastSaved), save document, then present Workflow Complete summary
- IF A: Perform advanced elicitation on current step output
- IF P: Invite other agents to party mode
- IF Y: Proceed without stopping
- IF Any other: respond and redisplay menu

**EXECUTION RULES:**
- 🛑 ALWAYS halt and wait for user input after presenting checkpoint menu
- 🛑 NEVER present "Workflow Complete" until user selects 'C'
- 🛑 ALWAYS update frontmatter BEFORE presenting completion
- 📖 Read the entire output document before completion summary
