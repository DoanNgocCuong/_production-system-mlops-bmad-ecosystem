---
step_id: QR-S02
step_name: Review
workflow_id: QR001
sequence: 2
type: "review"
agent: "Khang (Architect)"
description: "Apply checklist, identify findings, recommendations"
estimated_time: "15-20 minutes"
is_final_step: true
---

# Step 02: Apply Checklist & Generate Findings

## Objective

Conduct focused review and generate actionable recommendations:
1. Apply review checklist from Step 01
2. Identify findings (strengths and gaps)
3. Prioritize recommendations
4. Document trade-offs

## Execution Flow

### Example: Architecture Review

Using 5.01 Architectural Patterns framework:

**Checklist**:
- [ ] Single responsibility? (each component has one reason to change)
- [ ] Loose coupling? (components can change independently)
- [ ] High cohesion? (related code is together)
- [ ] Scalability approach clear? (horizontal vs vertical)
- [ ] Failure modes identified? (what breaks, what's the impact)
- [ ] Dependency management? (circular dependencies? clear contracts)
- [ ] Testing strategy? (unit, integration, end-to-end)
- [ ] Observability? (logging, metrics, tracing)

**Apply to Design**:
```
Your Microservices Architecture:

USER SERVICE → ORDER SERVICE → PAYMENT SERVICE → DATABASE

✅ STRENGTHS:
1. Clear single responsibility (user mgmt, order processing, payment)
2. Service boundaries well-defined
3. Async communication via message queue (loose coupling)

⚠️ GAPS:
1. Circular dependency not obvious? (does payment call order? check contracts)
2. Failure modes not documented (what if payment times out?)
3. Testing strategy unclear (how are integration tests structured?)
4. Observability limited (is trace ID propagated across services?)

🎯 RECOMMENDATIONS:
1. Document service contracts (OpenAPI) — [MEDIUM priority]
2. Add failure mode analysis (FMEA) — [HIGH priority]
3. Implement distributed tracing (Jaeger/Datadog) — [HIGH priority]
4. Add integration test suite — [MEDIUM priority]
```

---

### Example: API Design Review

Using 5.11 Security framework:

**Checklist**:
- [ ] Authentication required on all endpoints?
- [ ] Authorization checks per resource?
- [ ] Input validation (type, length, format)?
- [ ] Rate limiting implemented?
- [ ] Versioning strategy defined?
- [ ] Error responses sanitized?
- [ ] API keys rotated?
- [ ] Deprecation path documented?

**Apply to API Spec**:
```
Your REST API Design:

GET    /api/users/{id}
POST   /api/users
PUT    /api/users/{id}
DELETE /api/users/{id}

✅ STRENGTHS:
1. Clear RESTful design (resource-oriented)
2. Versioning in URL path (/api/v1/...)
3. Standard HTTP methods

⚠️ GAPS:
1. Rate limiting not documented
2. Error responses include stack traces (security risk)
3. User can delete other users' data? (authorization missing)
4. No pagination on list endpoint (scalability risk)

🎯 RECOMMENDATIONS:
1. Add authorization checks (verify user can only delete own data) — [CRITICAL]
2. Implement rate limiting (100 req/min per user) — [HIGH]
3. Sanitize error responses (no stack traces) — [HIGH]
4. Add pagination (default limit=50, max=1000) — [MEDIUM]
```

---

### Generate Quick Review Report

Create 1-2 page summary:

```markdown
# Quick Architecture Review Report

**Review Type**: REST API Design
**Reviewed**: User Management API v1
**Reviewer**: Khang (Architect)
**Date**: 2026-03-17

## Summary

Overall assessment: **GOOD with areas for improvement** (7/10)

The API design follows RESTful principles well, but has critical gaps in
authorization and input validation that must be addressed before production.

---

## Findings

### CRITICAL Issues (Must Fix)

**[F1] Missing Authorization Checks**
- Users can delete other users' data
- No permission validation on PUT/DELETE endpoints
- Severity: CRITICAL (data security risk)
- Fix Effort: 2-3 hours
- Recommendation: Implement RBAC middleware, validate user ownership before delete

---

### HIGH Priority Issues (Should Fix)

**[F2] Unprotected Error Responses**
- API returns full stack traces in 5xx errors
- Exposes internal implementation details
- Severity: HIGH (security information disclosure)
- Fix Effort: 30 minutes
- Recommendation: Wrap errors in safe JSON, log stack traces server-side only

**[F3] No Rate Limiting**
- API vulnerable to brute force attacks
- No protection against DoS
- Severity: HIGH (operational risk)
- Fix Effort: 1 hour
- Recommendation: Implement token bucket algorithm, 100 req/min per user

---

### MEDIUM Priority Issues (Good to Have)

**[F4] No Pagination on List Endpoint**
- GET /api/users returns all users (scalability risk)
- Could return 100k+ items on large deployments
- Severity: MEDIUM (performance risk)
- Fix Effort: 1 hour
- Recommendation: Add pagination (limit, offset), default 50, max 1000

---

## Strengths

✅ **RESTful Design** — Clear resource-oriented approach, standard HTTP methods
✅ **Versioning Strategy** — API versioning in URL path (/api/v1/)
✅ **Clear Semantics** — GET, POST, PUT, DELETE used correctly

---

## Trade-offs

**Option 1: Lightweight Authorization (2-3 hours)**
- Pro: Quick to implement, suitable for simple apps
- Con: Less flexible for complex permission systems
- Recommendation: Use this if simple user-level access (own data only)

**Option 2: Full RBAC System (8-12 hours)**
- Pro: Flexible, enterprise-ready
- Con: More complex code, overhead
- Recommendation: Use this if need role-based access (admin, moderator, user)

---

## Action Items

| Priority | Action | Owner | Effort | Deadline |
|----------|--------|-------|--------|----------|
| CRITICAL | Implement authorization checks | API Team | 2-3h | 2026-03-20 |
| HIGH | Sanitize error responses | API Team | 30m | 2026-03-20 |
| HIGH | Implement rate limiting | DevOps | 1h | 2026-03-20 |
| MEDIUM | Add pagination | API Team | 1h | 2026-03-24 |

---

## Next Steps

1. Assign action items to team leads
2. Block production deployment until CRITICAL items fixed
3. Schedule follow-up review after fixes applied
4. Document learnings for future API designs
```

---

## Deliverables

1. **Review Report** (markdown, 1-2 pages)
   - Summary assessment
   - Findings (organized by priority)
   - Strengths and trade-offs
   - Action items with effort/deadline

---

## Workflow Complete

```
"Review complete!

Summary: [overall assessment]

CRITICAL issues that must be fixed:
- [issue 1]
- [issue 2]

HIGH priority improvements:
- [issue 3]

Effort to address all: ~6-8 hours

Should I create detailed action items for your team?"
```

---

**Navigation**: [← Back to Step 01](step-01-context.md), [Back to Workflow Overview](../workflow.md)
