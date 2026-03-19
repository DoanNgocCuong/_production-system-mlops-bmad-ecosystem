# Security Audit — Completion Checklist

**Definition of Done**: This checklist confirms the security audit is thorough and report is ready for delivery.

## 📋 Scope Definition

- [ ] Audit scope clearly defined (service boundaries, systems included/excluded)
- [ ] Threat model created (attack vectors, threat actors, risk categories)
- [ ] Assets inventoried (data stores, APIs, infrastructure, dependencies)
- [ ] Compliance requirements identified (GDPR, CCPA, SOC2, industry-specific)
- [ ] Stakeholders and approval chain identified

## 🔍 Audit Execution

- [ ] Authentication mechanisms reviewed (OAuth2, JWT, SAML, MFA implementation)
- [ ] Authorization controls verified (RBAC, policy enforcement, least privilege)
- [ ] API security evaluated (rate limiting, input validation, CORS, API key management)
- [ ] Secrets management assessed (storage, rotation, access logging)
- [ ] Encryption validated (TLS config, key management, at-rest encryption)
- [ ] PII handling assessed (classification, access controls, retention, compliance)

## 📊 Findings

- [ ] All findings categorized by severity (CRITICAL, HIGH, MEDIUM, LOW)
- [ ] Evidence documented for each finding (logs, code snippets, test results)
- [ ] CVE/CWE references included where applicable
- [ ] Remediation plan created per finding
- [ ] Effort and priority estimated for each remediation
- [ ] Business impact articulated (data breach risk, compliance violation, etc.)

## 📝 Report & Compliance

- [ ] Security audit report generated (scope, findings, recommendations)
- [ ] Findings prioritized by severity and business impact
- [ ] Compliance status documented (compliant / non-compliant / in-progress)
- [ ] Risk acceptance obtained for deferred findings (if applicable)
- [ ] Executive summary prepared for stakeholders
- [ ] Technical appendix included for implementation teams

## 🔚 Final Status

- [ ] Security report delivered to service owner and stakeholders
- [ ] Remediation action items created and tracked in backlog
- [ ] Responsible parties assigned to each remediation
- [ ] Critical findings escalated to leadership
- [ ] Re-audit scheduled (typically 6-12 months from completion)
