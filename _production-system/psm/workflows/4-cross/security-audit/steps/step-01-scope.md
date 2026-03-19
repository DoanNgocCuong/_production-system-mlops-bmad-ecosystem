---
step_id: SA-S01
step_name: Scope & Threat Model
workflow_id: SA001
sequence: 1
type: "scoping"
agent: "Hà (Security)"
description: "Define audit scope, identify threat model, compliance requirements"
estimated_time: "30-40 minutes"
is_final_step: false
outputFile: "{project-root}/psm/outputs/security-audit/{audit_id}/security-audit-report.md"
workflowType: "security-audit"
---

# Step 01: Scope Security Audit & Threat Model

## Objective

Define the scope and boundaries of the security audit:
1. Understand service architecture and data flows
2. Identify threat model (threats/assets/actors)
3. Determine applicable compliance requirements
4. Establish baseline security posture
5. Plan detailed assessment in Step 02

## BMAD v6.2 Checkpoint Protocol

**Before proceeding, ensure output document exists with YAML frontmatter:**
```yaml
---
stepsCompleted: []
lastStep: "step-01-scope.md"
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

### Substep 1A: Introduction & Scope Setting

As Hà (Security Lead), introduce the audit:

```
"Hi! I'm Hà, your security lead for this audit.

We'll do a comprehensive security evaluation covering:
- Authentication & Authorization
- API Security
- Secrets Management
- Encryption (in-transit and at-rest)
- PII & Data Protection

This is not a penetration test—we're reviewing your design and controls.
After this audit, we'll have a clear picture of your security posture
and a prioritized remediation plan."
```

### Substep 1B: Gather Service Architecture

Collect architectural information:

```
1. Service Overview
   - What does the service do?
   - Who are the users/consumers?
   - What data does it process/store?

2. Architecture Diagram
   - Components (frontend, API, database, cache, queues, etc.)
   - Data flows (request/response, persistence, sync)
   - External integrations (payment processor, auth provider, etc.)
   - Deployment topology (single region, multi-region, edge)

3. Data Sensitivity
   - What PII does the service handle? (names, emails, payment info, etc.)
   - Sensitive business data? (API keys, credentials, source code)
   - Who can access what data? (users, admins, employees)

4. Trust Boundaries
   - Where does untrusted data enter the system? (user input, APIs, webhooks)
   - Where does trusted data reside? (databases, configuration)
   - What are the trust relationships? (client-to-service, service-to-service)
```

Document responses clearly for threat modeling in Substep 1C.

### Substep 1C: Threat Modeling

Using collected architecture, identify threats:

**Assets to Protect**:
- [ ] User credentials
- [ ] PII (names, emails, payment info)
- [ ] API keys / service credentials
- [ ] Sensitive business data
- [ ] Transaction history
- [ ] System availability

**Potential Threat Actors**:
- [ ] External attackers (internet)
- [ ] Malicious insiders (employees)
- [ ] Compromised upstream dependencies
- [ ] Compliance violations (accidental)

**Common Attack Vectors**:
- [ ] SQL injection / NoSQL injection
- [ ] Cross-site scripting (XSS)
- [ ] Cross-site request forgery (CSRF)
- [ ] Man-in-the-middle (MITM)
- [ ] Credential theft / brute force
- [ ] Data exfiltration
- [ ] Denial of service
- [ ] Dependency vulnerabilities
- [ ] Misconfiguration (exposed secrets, open S3 buckets)

**Likely Scenarios**:
- If database is compromised, what data is exposed?
- If API key is leaked, what can attacker do?
- If user password is weak, can it be brute-forced?
- If HTTPS is not enforced, can traffic be eavesdropped?

### Substep 1D: Compliance & Regulatory Requirements

Determine applicable compliance frameworks:

```
1. Data Protection
   - GDPR? (if serving EU users)
   - CCPA? (if serving California residents)
   - Data residency requirements?

2. Payment Card Industry (PCI)
   - Handling credit cards? (triggers PCI-DSS)
   - Using payment processor? (may handle compliance)

3. Industry Standards
   - Healthcare (HIPAA)?
   - Finance (FINRA, SOX)?
   - Government (FedRAMP, NIST)?

4. Internal Policy
   - Company security standards?
   - Data classification policy?
   - Access control standards?

5. Contractual
   - Customer contracts require specific security?
   - SLAs mention uptime or compliance?
```

Map each requirement to specific controls needed in Step 02.

### Substep 1E: Establish Baseline

Document current security posture:

```
Questions for Service Owner/Platform Team:

Current State Assessment:
- [ ] Do you have any security controls in place today?
- [ ] Are there known vulnerabilities or debt?
- [ ] Have you had a security review before?
- [ ] What's your biggest security concern?

Assumptions & Constraints:
- [ ] What's the deployment timeline?
- [ ] Are there architectural constraints we need to know about?
- [ ] Budget constraints for security tooling?
```

### Substep 1F: Transition to Step 02

Confirm scope and proceed:

```
"Great! I have a clear picture of your service now.

In Step 02, I'll do a detailed security assessment across these domains:
1. Authentication & Authorization
2. API Security
3. Secrets Management
4. Encryption (in-transit & at-rest)
5. PII & Data Protection

I'll check your design against best practices and compliance requirements,
then score each domain with recommended remediation actions.

Ready to dive deep?"
```

**On confirmation → Proceed to [Step 02](step-02-audit.md)**

---

## Checkpoint

**Select an Option:**
**[C] Continue to step-02-audit.md**
**[A] Advanced Elicitation**
**[P] Party Mode**
**[Y] YOLO**

Menu Handling:
- IF C: Update frontmatter (add this step to stepsCompleted, update lastStep, lastSaved), save document, then read fully and follow `./step-02-audit.md`
- IF A: Perform advanced elicitation on current step output
- IF P: Invite other agents to party mode
- IF Y: Proceed without stopping
- IF Any other: respond and redisplay menu

**EXECUTION RULES:**
- 🛑 ALWAYS halt and wait for user input after presenting checkpoint menu
- 🛑 NEVER load the next step file until user selects 'C'
- 🛑 ALWAYS update frontmatter BEFORE loading next step
- 📖 Read the entire next step file before execution

## Deliverables from This Step

1. **Audit Scope Document**
   - Service overview and architecture diagram
   - Data sensitivity classification
   - Trust boundaries identified

2. **Threat Model**
   - Assets to protect
   - Potential threat actors
   - Attack vectors
   - Risk scenarios

3. **Compliance Mapping**
   - Applicable frameworks (GDPR, PCI, etc.)
   - Compliance requirements → control mapping
   - Timeline for remediation

4. **Baseline Assessment**
   - Current security controls (if any)
   - Known vulnerabilities or debt
   - Constraints to consider during assessment

## Key Questions

- "What's the most sensitive data in this service?"
- "Who shouldn't have access to this data, and how do you prevent that?"
- "If a malicious user tried to break this service, what would be their first target?"
- "What compliance standards apply to your business?"
