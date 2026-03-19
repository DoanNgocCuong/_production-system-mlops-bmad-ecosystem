# Incident Resolution — Validation Checklist

**Definition of Done**: This checklist confirms the incident is fully resolved and postmortem is complete.

## 🚨 Triage

- [ ] Incident severity classified (SEV1, SEV2, SEV3, SEV4)
- [ ] On-call engineer notified immediately
- [ ] Incident commander designated
- [ ] Affected services and users documented
- [ ] Incident channel created (Slack, Teams, or equivalent)
- [ ] Initial stakeholder communication sent

## 🔍 Diagnosis

- [ ] Metrics collected and analyzed (CPU, memory, latency, error rates)
- [ ] Logs reviewed and filtered by timestamp range
- [ ] Distributed traces examined for bottlenecks
- [ ] Root cause identified with supporting evidence
- [ ] Hypotheses documented (considered and rejected causes)
- [ ] All team members informed of diagnosis

## 🔧 Fix & Recovery

- [ ] Fix applied to production (or rollback executed)
- [ ] Verification passed (metrics returned to baseline)
- [ ] User impact confirmed resolved (manual testing or monitoring)
- [ ] Rollback tested and validated if fix was deployed
- [ ] Monitoring confirmed stable for minimum 10 minutes post-fix
- [ ] All-clear communication sent to stakeholders

## 📝 Postmortem

- [ ] Incident timeline documented (detection, triage, fix, recovery)
- [ ] Root cause analysis completed with supporting evidence
- [ ] Action items created with owners and deadlines
- [ ] P0 action items (must fix immediately)
- [ ] P1 action items (must fix in next sprint)
- [ ] P2 action items (nice to fix)
- [ ] Lessons learned captured (what went well, what to improve)
- [ ] Postmortem document drafted

## 🔚 Final Status

- [ ] Incident status updated in tracking system (Resolved/Closed)
- [ ] Sprint/release status synced if deployment was reverted
- [ ] Follow-up action items added to backlog
- [ ] Re-audit/re-check scheduled if action items deferred
- [ ] Postmortem shared with team for feedback
- [ ] Postmortem published to knowledge base
