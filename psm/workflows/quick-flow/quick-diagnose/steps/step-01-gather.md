---
step_id: QD-S01
step_name: Gather Context
workflow_id: QD001
sequence: 1
type: "triage"
agent: "Minh (SRE)"
description: "Collect symptom info, run quick metrics check, gather baseline"
estimated_time: "5-10 minutes"
---

# Step 01: Gather Symptom Context

## Objective

Rapidly collect symptom information and baseline metrics:
1. Understand what is broken
2. Gather quick metrics snapshot
3. Identify timeline and impact
4. Establish baseline for diagnosis

## Execution Flow

### Quick Triage Questions

As Minh (SRE), ask in this order (takes 2-3 minutes):

```
1. WHAT'S BROKEN?
   "What are you seeing? (e.g., slow responses, errors, timeouts, no responses)"

2. WHICH SERVICE?
   "Which service/component? (e.g., API, database, cache, payment processor)"

3. WHEN DID IT START?
   "When? (just now, 5 min ago, 1 hour ago)"

4. IMPACT?
   "How bad? (all users, some users, specific feature, revenue impact?)"

5. ERROR MESSAGE?
   "Any error message? (HTTP 500, timeout, database connection refused, etc.)"
```

### Quick Metrics Snapshot

Immediately pull metrics:

```bash
# For API services:
- Response time (last 5 min): p50, p95, p99
- Error rate (last 5 min): %
- Request rate (last 5 min): req/sec
- HTTP status distribution (last 5 min): 2xx, 4xx, 5xx

# For databases:
- Query latency (last 5 min): p99
- Connection pool: used/max
- Slow query count (last 5 min): count
- Replication lag (if applicable): ms

# For infrastructure:
- CPU utilization: %
- Memory utilization: %
- Disk space: % used
- Network I/O: Mbps in/out
```

### Establish Timeline

```markdown
Timeline:
├─ T-0:00 — All normal (baseline)
├─ T-0:15 — First user complaint
├─ T-0:20 — Error rate jumped to 5%
├─ T-0:22 — Received alert
└─ T-now — Still ongoing (20 min duration)
```

### Document Symptom Pattern

```
Pattern Analysis:
├─ Is it constant or intermittent?
├─ Affects all traffic or specific subset?
├─ Affects all services or just one?
├─ Has error rate been increasing or flat?
```

### Transition to Step 02

```
"Got it. Let me pull dashboards and logs for [service].
I'll analyze the root cause and suggest a fix in about 5 minutes."
```

**Proceed to [Step 02](step-02-diagnose.md)**

---

**Navigation**: [← Back to Workflow Overview](../workflow.md), [Next: Step 02 →](step-02-diagnose.md)
