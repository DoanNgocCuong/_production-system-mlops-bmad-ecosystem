---
step_id: SA-S02
step_name: Security Assessment
workflow_id: SA001
sequence: 2
type: "assessment"
agent: "Hà (Security)"
description: "Execute security checklist across domains, identify vulnerabilities, grade findings"
estimated_time: "60-90 minutes"
is_final_step: false
outputFile: "{project-root}/psm/outputs/security-audit/{audit_id}/security-audit-report.md"
workflowType: "security-audit"
---

# Step 02: Execute Security Assessment

## Objective

Conduct systematic security assessment across all 5 domains:
- Score each domain (COMPLIANT/FINDINGS/CRITICAL)
- Identify specific vulnerabilities and control gaps
- Assign severity levels (Critical, High, Medium, Low)
- Generate remediation recommendations

## BMAD v6.2 Checkpoint Protocol

**Before proceeding, ensure output document exists with YAML frontmatter:**
```yaml
---
stepsCompleted: ["step-01-scope.md"]
lastStep: "step-02-audit.md"
lastSaved: "{current-timestamp}"
workflowType: "security-audit"
---
```

**SAVING RULE:** After completing all execution sections, BEFORE loading the next step:
1. Update output document: append current step's output to the document
2. Update frontmatter: add this step to `stepsCompleted`, update `lastStep` and `lastSaved`
3. Save the document

---

## Execution Flow

### Domain 1: Authentication & Authorization

**Skill Reference**: 5.11 Security

Evaluate:
- [ ] Authentication mechanism defined? (JWT, OAuth2, SAML, multi-factor)
- [ ] User identity verified before access? (no guest mode for sensitive operations)
- [ ] Authorization (RBAC, ABAC) implemented and tested?
- [ ] Session management secure? (expiration, invalidation, secure cookies)
- [ ] Default credentials changed? (no hardcoded admin/password)
- [ ] API authentication enforced? (all endpoints require auth token)
- [ ] Cross-origin requests (CORS) properly restricted?

**Scoring Criteria**:
- **COMPLIANT**: Auth/authz implemented, multi-factor available, sessions secure, tested
- **FINDINGS**: Basic auth present, but gaps in scope (e.g., some endpoints unprotected)
- **CRITICAL**: No auth, default credentials, or trivial bypass

**Prompt for Service Owner**:
> "How do you verify a user is who they claim to be? Do you use API keys, tokens, or something else? Can a user with role X access data they shouldn't?"

---

### Domain 2: API Security

**Skill Reference**: 5.11 Security, 5.12 Config Management

Evaluate:
- [ ] Input validation on all endpoints? (type, length, format checks)
- [ ] Sanitization against injection attacks? (SQL, NoSQL, command injection)
- [ ] Rate limiting / throttling implemented?
- [ ] API versioning and deprecation strategy?
- [ ] Error responses don't leak sensitive info? (no stack traces, PII)
- [ ] CORS headers properly configured?
- [ ] Security headers present? (CSP, X-Frame-Options, etc.)
- [ ] Request/response logging without exposing secrets?

**Scoring Criteria**:
- **COMPLIANT**: Input validation, rate limiting, secure error messages, security headers
- **FINDINGS**: Validation present but incomplete; rate limiting missing; headers partially configured
- **CRITICAL**: No input validation, SQL injection possible, error responses leak secrets

**Prompt for Service Owner**:
> "What happens if someone sends you a really long string as input? Can they break your database? Do you have rate limiting so someone can't hammer your API?"

---

### Domain 3: Secrets Management

**Skill Reference**: 5.12 Config Management

Evaluate:
- [ ] No hardcoded secrets in code? (scanned with git-secrets, TRUFFLEHO3)
- [ ] Secrets stored in secure vault? (AWS Secrets Manager, HashiCorp Vault, Azure Key Vault)
- [ ] Secrets rotated regularly? (expiration dates, rotation policy)
- [ ] Access to secrets logged and auditable? (who accessed what, when)
- [ ] Secrets never logged or exposed in debug output?
- [ ] Different secrets per environment? (dev ≠ prod)
- [ ] Backup credentials available in case of emergency?

**Scoring Criteria**:
- **COMPLIANT**: Vault-managed secrets, rotation policy, audit logging, environment separation
- **FINDINGS**: Secrets in vault but not rotated; audit logging incomplete; some env reuse
- **CRITICAL**: Hardcoded secrets, plaintext storage, or shared credentials across environments

**Prompt for Service Owner**:
> "Where do you store API keys, database passwords, and other secrets? If I clone your GitHub repo, would I find any hardcoded secrets?"

---

### Domain 4: Encryption

**Skill Reference**: 5.11 Security, 5.12 Config Management

Evaluate:

**In Transit**:
- [ ] All external communication over HTTPS/TLS 1.2+?
- [ ] Internal service-to-service communication encrypted?
- [ ] Certificate pinning or mutual TLS (mTLS) for high-risk connections?
- [ ] No fallback to HTTP (HSTS enforced)?
- [ ] Cipher suite strong? (no weak ciphers like RC4, DES)

**At Rest**:
- [ ] Database encryption enabled? (transparent data encryption, column-level)
- [ ] File storage encrypted? (S3 server-side encryption, encrypted volumes)
- [ ] Backups encrypted?
- [ ] Encryption keys managed separately from data?
- [ ] Key rotation policy in place?

**Scoring Criteria**:
- **COMPLIANT**: TLS 1.2+ for all external, encrypted at rest with key rotation
- **FINDINGS**: TLS present but weak cipher suite; database encryption missing
- **CRITICAL**: Unencrypted data in transit or at rest, especially for PII

**Prompt for Service Owner**:
> "Is all communication to your API over HTTPS? Is your database encrypted? If someone gets access to your database server, would they be able to read the data directly?"

---

### Domain 5: PII & Data Protection

**Skill Reference**: 5.14 Data Governance & Storage, 5.18 Compliance

Evaluate:
- [ ] PII identified and classified? (what data, who can access)
- [ ] Minimum necessary data collected? (data minimization)
- [ ] Data retention policy? (how long is PII kept)
- [ ] Access controls? (only authorized users can access PII)
- [ ] Audit logging for PII access? (who accessed what, when)
- [ ] Right to deletion implemented? (GDPR/CCPA requirement)
- [ ] Data breach response plan? (notification, investigation)
- [ ] Encryption and masking for PII in logs/dev environments?
- [ ] Third-party data processor agreements? (if outsourcing storage/processing)

**Scoring Criteria**:
- **COMPLIANT**: PII classified, minimized, encrypted, retention policy, audit logs, deletion capability
- **FINDINGS**: PII present but classification incomplete; retention policy missing; audit logging partial
- **CRITICAL**: PII unencrypted, unlimited retention, no access controls, no deletion capability

**Prompt for Service Owner**:
> "What personal information does your service collect? (names, emails, addresses, payment info?) Can users request deletion of their data? Who can see their information?"

---

## Severity Grading Matrix

Score each finding using this framework:

| Severity | CVSS | Description | Timeline |
|----------|------|-------------|----------|
| **Critical** | 9.0-10.0 | Data breach imminent; regulatory violation; service compromise | Fix within 24-48 hours |
| **High** | 7.0-8.9 | Exploitable vulnerability; significant access bypass | Fix within 1 week |
| **Medium** | 4.0-6.9 | Control gap but requires additional factors; partial bypass | Fix within 2-4 weeks |
| **Low** | 0.1-3.9 | Security hygiene issue; best practice gap | Plan for next sprint |

## Example Findings

```markdown
### Finding #1: Missing Input Validation [HIGH]

**Domain**: API Security
**Severity**: HIGH (CVSS 7.5)
**Description**: User input on `/api/search` endpoint not validated for SQL injection.
Attacker can inject SQL queries to access unauthorized data.

**Evidence**:
- Code review shows direct string interpolation in SQL query
- No parameterized queries or ORM protection
- Testing confirms `'; DROP TABLE users; --` bypasses authentication

**Impact**: Data breach, regulatory violation, service unavailability

**Remediation**:
1. Implement parameterized queries for all database access
2. Use ORM or prepared statements
3. Add input validation for expected formats (whitelist approach)
4. Add automated testing for SQL injection payloads

**Owner**: Backend Team
**Deadline**: 2026-03-24
**Effort**: 2-3 days
```

---

## Domain Summary Scorecard

Document all findings:

| Domain | Score | Critical | High | Medium | Low | Status |
|--------|-------|----------|------|--------|-----|--------|
| Auth & Authz | COMPLIANT | 0 | 0 | 1 | 0 | ✅ |
| API Security | FINDINGS | 0 | 2 | 1 | 2 | ⚠️ |
| Secrets Mgmt | COMPLIANT | 0 | 0 | 0 | 0 | ✅ |
| Encryption | FINDINGS | 0 | 0 | 2 | 0 | ⚠️ |
| PII & Data | CRITICAL | 1 | 1 | 0 | 0 | ❌ |

---

## Transition to Step 03

Once all domains assessed:

```
"I've completed the security assessment across all 5 domains.

Summary:
- [#] COMPLIANT domains (no significant findings)
- [#] FINDINGS domains (have remediable issues)
- [#] CRITICAL domains (require immediate action)

Let's move to Step 03 where I'll present the full findings report
with prioritized remediation plan and compliance status."
```

**Proceed to [Step 03](step-03-report.md)**

---

## Checkpoint

**Select an Option:**
**[C] Continue to step-03-report.md**
**[A] Advanced Elicitation**
**[P] Party Mode**
**[Y] YOLO**

Menu Handling:
- IF C: Update frontmatter (add this step to stepsCompleted, update lastStep, lastSaved), save document, then read fully and follow `./step-03-report.md`
- IF A: Perform advanced elicitation on current step output
- IF P: Invite other agents to party mode
- IF Y: Proceed without stopping
- IF Any other: respond and redisplay menu

**EXECUTION RULES:**
- 🛑 ALWAYS halt and wait for user input after presenting checkpoint menu
- 🛑 NEVER load the next step file until user selects 'C'
- 🛑 ALWAYS update frontmatter BEFORE loading next step
- 📖 Read the entire next step file before execution
