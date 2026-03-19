# Production Readiness Review — Sign-off Checklist

**Definition of Done**: This checklist confirms the service has been comprehensively reviewed and decision rendered.

## 📋 Initialization

- [ ] Service context gathered (name, owner, purpose, criticality)
- [ ] PRR framework presented to participant (9 dimensions overview)
- [ ] Scoring system explained (RED/YELLOW/GREEN, P0/P1 categories)
- [ ] Existing dashboards and runbooks identified
- [ ] Known risks and gaps documented from service owner

## 🔍 Deep Review

- [ ] Reliability dimension evaluated (SLA/SLO, error budget, failure modes)
- [ ] Observability dimension evaluated (logging, metrics, tracing, dashboards)
- [ ] Performance dimension evaluated (latency targets, throughput, P99 behavior)
- [ ] Security dimension evaluated (auth, encryption, compliance, audit logging)
- [ ] Capacity dimension evaluated (resource limits, scaling, burst capacity)
- [ ] Data dimension evaluated (backup, schema versioning, retention, governance)
- [ ] Runbooks dimension evaluated (incident procedures, operational playbooks)
- [ ] Dependencies dimension evaluated (external/internal services, contracts)
- [ ] Rollback dimension evaluated (rollback strategy, canary, feature flags)

## 📊 Scoring

- [ ] Each dimension scored RED/YELLOW/GREEN with clear rationale
- [ ] P0 blockers identified (must fix before production)
- [ ] P1 risks documented (should fix, documented caveats accepted)
- [ ] Scorecard visual prepared for presentation
- [ ] Patterns mapped to BMAD framework for consistency

## ✅ Decision

- [ ] GO / NO-GO / CONDITIONAL-GO decision rendered
- [ ] Decision rationale documented
- [ ] Conditions clearly stated (if CONDITIONAL-GO)
- [ ] Risk acceptance obtained (if CONDITIONAL-GO)
- [ ] Escalation path confirmed if NO-GO

## 📝 Sign-off

- [ ] Action items table created (blocker fixes, improvement tasks)
- [ ] Each action item assigned owner and deadline
- [ ] Sign-off chain completed (service owner, architect, SRE, compliance if applicable)
- [ ] Post-production monitoring plan defined (first 72 hours focus areas)
- [ ] PRR report finalized and distributed to stakeholders
- [ ] Next review date scheduled (if GO or CONDITIONAL-GO)
