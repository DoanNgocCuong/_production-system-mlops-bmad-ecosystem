# Production Incident Response Workflow

## Overview

Complete BMAD pattern incident response workflow for Production Systems. This is the most critical workflow - handles triage, diagnosis, recovery, and learning from production incidents.

**Status**: PRODUCTION READY
**Version**: 6.2.0
**Lead Agent**: SRE Minh
**Phase**: 3-Run (Emergency Response & Recovery)

## Files Created

### Main Entry Point
- **workflow.md** (162 lines)
  - Entry point with overall context, severity scale, navigation options
  - Configuration loading instructions
  - Workflow architecture explanation (micro-file design)
  - Skill references to 5.07, 5.08, 5.09, 5.10, 5.14

### Step Files (4 atomic steps)

1. **step-01-triage.md** (234 lines)
   - Goal: Gather initial symptoms, assess severity, classify
   - Execution: 5 critical questions → severity classification → triage report
   - Time: 5-10 minutes
   - Decision: [A] Advanced Elicitation or [C] Continue to diagnosis
   - Skills: 5.08 Observability, 5.10 Production Readiness

2. **step-02-diagnose.md** (298 lines)
   - Goal: Systematic root cause analysis using observability data
   - Execution: Metrics dashboard → logs → traces → root cause hypothesis
   - Time: 10-30 minutes
   - Decision: [A] Advanced Investigation or [C] Continue to fix
   - Skills: 5.08 Observability, 5.07 Reliability, 5.09 Error Handling

3. **step-03-fix.md** (358 lines)
   - Goal: Apply fix safely with rollback plan, verify recovery
   - Execution: Select strategy → prepare rollback → apply fix → verify metrics
   - Time: 5-20 minutes
   - Decision: [A] Advanced Verification or [C] Continue to postmortem
   - Skills: 5.07 Reliability, 5.09 Error Handling, 5.10 Production Readiness
   - Fix strategies: Dependency recovery, resource management, database fixes, rollback, config changes, fallback

4. **step-04-postmortem.md** (385 lines)
   - Goal: Document incident, identify system-level prevention
   - Execution: Timeline reconstruction → 5-why analysis → action items → lessons learned
   - Time: 15-30 minutes
   - Decision: [A] Advanced Review or [C] Complete postmortem
   - Skills: 5.14 Documentation, 5.10 Production Readiness
   - Final step (no next_step_file)

## Key Features

### BMAD Pattern Compliance
- ✓ Micro-file architecture (each step is separate file)
- ✓ State tracking via YAML frontmatter
- ✓ Just-in-time skill loading (references to SKILL.md files)
- ✓ Quality gates at each step (A/C menu system)
- ✓ Clear next step navigation (nextStepFile in frontmatter)

### Execution Model
- Entry point: workflow.md → step-01
- State preserved between steps
- Resume capability (if interrupted, restart from exact step)
- Final output: Incident postmortem + action items

### Severity Scale
- **SEV1**: Complete service down, revenue-blocking → Page all on-call immediately
- **SEV2**: Major degradation, significant users affected → Escalate quickly
- **SEV3**: Moderate impact, some users affected → Assign to senior engineer
- **SEV4**: Minor issue, limited users → Can defer to business hours

### Knowledge Integration
- **5.07 Reliability**: Circuit breaker patterns, fallback strategies, timeout management
- **5.08 Observability**: Structured logging, metrics queries, distributed tracing
- **5.09 Error Handling**: Error classification, graceful degradation, recovery patterns
- **5.10 Production Readiness**: Incident prevention checklist, alerting setup, SLO/SLA
- **5.14 Documentation**: Postmortem templates, incident reports, runbooks

## Usage Flow

```
1. [NEW-INC] Incident detected
   ↓
2. workflow.md → Load step-01-triage.md
   ↓
3. Step-01: Ask 5 questions, classify severity (SEV1-4)
   ↓
4. Step-02: Analyze metrics + logs + traces, identify root cause
   ↓
5. Step-03: Apply fix with rollback plan, verify recovery
   ↓
6. Step-04: Document postmortem, create prevention actions
   ↓
7. INCIDENT RESOLVED + Lessons learned captured
```

## Metrics & SLOs

**Target MTTR (Mean Time To Recovery)**:
- SEV1: < 15 minutes
- SEV2: < 30 minutes
- SEV3: < 1 hour
- SEV4: Next business day

**Triage**: < 5 minutes to severity classification
**Diagnosis**: < 20 minutes to root cause identification
**Recovery**: Fix applied + verified within 20 minutes of root cause identification
**Postmortem**: Completed within 24 hours of incident resolution

## Integration Points

This workflow integrates with:
- Alert systems (detection trigger)
- On-call schedules (escalation)
- Slack #incidents channel (communication)
- Metrics dashboards (Prometheus, Datadog, CloudWatch)
- Distributed tracing (Jaeger, Datadog APM)
- Log aggregation (ELK, Splunk, Datadog)
- Incident tracking (Jira, Linear)
- Wiki/Knowledge base (Confluence, Notion)

## Best Practices Encoded

1. **Speed first**: Triage in < 5 minutes
2. **Root cause identification**: Find system issue, not just surface fix
3. **Rollback readiness**: Always have undo plan before applying fix
4. **Evidence-based**: All decisions supported by observability data
5. **Blameless postmortem**: Focus on systems, not people
6. **Prevention actions**: 5-why analysis to find system-level fixes
7. **Continuous improvement**: Learn from every incident

## Next Steps

To integrate this workflow:

1. Copy to BMAD workflow registry
2. Update _config/workflow-manifest.csv with entry:
   ```
   W-INCIDENT-001,Production Incident Response,incident,/workflows/3-run/incident-response/workflow.md
   ```
3. Train team on 4-step workflow
4. Assign primary/backup incident commanders
5. Set up escalation contacts in config.yaml
6. Practice with tabletop incident scenarios

---

**Created**: 2026-03-17
**Total Lines**: 1,437 (workflow.md + 4 step files)
**Format**: BMAD v6.2.0 compliant
**Language**: Vietnamese-English (Tiếng Việt-English)
