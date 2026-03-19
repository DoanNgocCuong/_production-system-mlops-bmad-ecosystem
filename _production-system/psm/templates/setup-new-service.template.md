---
template_name: setup-new-service
template_version: "1.0.0"
created_date: 2026-03-19
workflow_id: setup-new-service
description: Service setup report documenting all phases from architecture to production readiness
---

# New Service Setup Report

**Service**: {{SERVICE_NAME}}
**Owner**: {{SERVICE_OWNER}}
**SRE Lead**: {{SRE_LEAD}} (Minh)
**Setup Date**: {{DATE}}
**Target Environment**: {{PRODUCTION | STAGING | DEVELOPMENT}}
**Target Go-Live**: {{TARGET_DATE}}

---

## Executive Summary

{{1-2 paragraphs summarizing the service setup, architecture decisions, and readiness assessment}}

**Overall Assessment**: {{READY | CONDITIONAL | NOT_READY}}

**PRR Readiness Score**: {{X}}/10

---

## Phase 1: Architecture Design

### Service Purpose

{{Brief description of what this service does and why it exists}}

### Architecture Pattern

**Pattern**: {{MICROSERVICES | MONOLITH | SERVERLESS | EVENT-DRIVEN | HYBRID}}

**Justification**: {{Why this pattern was chosen for this use case}}

### High-Level Design

{{Include or link to architecture diagram}}

### Service Components

| Component | Technology | Responsibility |
|-----------|------------|----------------|
| {{Component 1}} | {{Tech stack}} | {{Responsibility}} |
| {{Component 2}} | {{Tech stack}} | {{Responsibility}} |
| {{Component 3}} | {{Tech stack}} | {{Responsibility}} |

### SLO Definition

| Metric | Target | Measurement |
|--------|--------|-------------|
| Availability | {{X}}% | Monthly |
| Latency P99 | {{X}}ms | Per request |
| Error Rate | {{X}}% | Per hour |

### Dependencies

| Dependency | Type | Fallback Strategy |
|------------|------|-------------------|
| {{Dependency 1}} | {{INTERNAL|EXTERNAL}} | {{Fallback approach}} |
| {{Dependency 2}} | {{INTERNAL|EXTERNAL}} | {{Fallback approach}} |

**Phase 1 Status**: {{✅ COMPLETE | ⚠️ IN PROGRESS | ❌ INCOMPLETE}}

---

## Phase 2: API & Database Design

### API Design

#### Endpoints

| Method | Endpoint | Purpose | Auth Required |
|--------|----------|---------|--------------|
| {{GET|POST|PUT|DELETE}} | {{/api/path}} | {{Purpose}} | {{YES|NO}} |
| {{GET|POST|PUT|DELETE}} | {{/api/path}} | {{Purpose}} | {{YES|NO}} |
| {{GET|POST|PUT|DELETE}} | {{/api/path}} | {{Purpose}} | {{YES|NO}} |

#### API Standards Compliance

- [ ] RESTful conventions followed
- [ ] Proper HTTP status codes used
- [ ] Input validation implemented
- [ ] Rate limiting configured
- [ ] API versioning strategy defined

### Database Design

**Database Type**: {{POSTGRESQL|MYSQL|MONGODB|DYNAMODB|REDIS|OTHER}}

**Justification**: {{Why this database was chosen}}

#### Schema / Data Model

{{Include or link to database schema or data model}}

#### Migrations

- [ ] Migration scripts version-controlled
- [ ] Rollback strategy defined
- [ ] Tested in staging environment

#### Backup & Recovery

- [ ] Automated backups configured
- [ ] Backup retention policy: {{X}} days
- [ ] Recovery procedure documented
- [ ] Recovery time objective (RTO): {{X}} hours
- [ ] Recovery point objective (RPO): {{X}} minutes

**Phase 2 Status**: {{✅ COMPLETE | ⚠️ IN PROGRESS | ❌ INCOMPLETE}}

---

## Phase 3: Build & Deployment Pipeline

### CI/CD Pipeline

#### Build Stage

- **Build Tool**: {{DOCKER|MAVEN|GRADLE|NPM|OTHER}}
- **Build Trigger**: {{PUSH TO BRANCH | PR | MANUAL | SCHEDULED}}
- **Artifact**: {{DOCKER IMAGE | JAR | BINARY | OTHER}}

#### Test Stage

- [ ] Unit tests: {{Coverage: X%}}
- [ ] Integration tests: {{Status}}
- [ ] Security scans: {{TRIVY|AQUA|SNYK|OTHER}}
- [ ] Code quality checks: {{SONARQUBE|OTHER}}

#### Deployment Stage

- **Deployment Tool**: {{KUBERNETES|HELM|TERRAFORM|ANSIBLE|OTHER}}
- **Environment Strategy**: {{BLUE-GREEN | CANARY | ROLLING | RECREATE}}
- **Rollback Strategy**: {{AUTOMATIC | MANUAL | BOTH}}

### Infrastructure as Code

- [ ] Terraform/Ansible/IaC scripts version-controlled
- [ ] Infrastructure changes reviewed via PR
- [ ] State management configured

### Containerization

- [ ] Dockerfile created and optimized
- [ ] Image size < {{X}}MB
- [ ] Non-root user configured
- [ ] Health check defined
- [ ] Resource limits set (CPU: {{X}}, Memory: {{X}})

**Phase 3 Status**: {{✅ COMPLETE | ⚠️ IN PROGRESS | ❌ INCOMPLETE}}

---

## Phase 4: Reliability Patterns

### High Availability

- [ ] Multiple replicas configured: {{X}} replicas
- [ ] Pod disruption budget defined
- [ ] Anti-affinity rules configured
- [ ] Multi-zone deployment: {{YES|NO}}

### Circuit Breaker

- [ ] Circuit breaker implemented for external calls
- [ ] Threshold settings: {{Failure threshold: X, Timeout: Xms}}
- [ ] Fallback responses defined

### Retry & Timeout Strategy

| External Call | Retry Attempts | Backoff | Timeout |
|---------------|----------------|---------|---------|
| {{Service 1}} | {{X}} | {{EXPONENTIAL|LINEAR}} | {{X}}ms |
| {{Service 2}} | {{X}} | {{EXPONENTIAL|LINEAR}} | {{X}}ms |

### Rate Limiting

- [ ] Rate limiting configured at API gateway
- [ ] Limit: {{X}} requests per {{minute|hour}}
- [ ] Response on limit exceeded: {{429 TOO MANY REQUESTS with retry-after}}

### Graceful Shutdown

- [ ] Graceful shutdown signal handling implemented
- [ ] In-flight requests drained before termination
- [ ] Drain timeout: {{X}} seconds

**Phase 4 Status**: {{✅ COMPLETE | ⚠️ IN PROGRESS | ❌ INCOMPLETE}}

---

## Phase 5: Security & Infrastructure Hardening

### Network Security

- [ ] Network policies defined (Kubernetes)
- [ ] Service mesh: {{YES ({{ISTIO|LINKERD|OTHER}})|NO}}
- [ ] mTLS between services: {{YES|NO}}
- [ ] Ingress/Egress controls configured
- [ ] Firewall rules defined

### Secrets Management

- [ ] Secrets stored in: {{VAULT|AWS SECRETS MANAGER|GCP SECRET MANAGER|K8S SECRETS}}
- [ ] No secrets in code or environment variables (except injected at runtime)
- [ ] Secrets rotation policy: {{Frequence}}

### Authentication & Authorization

- [ ] Authentication mechanism: {{JWT|OAUTH2|MTLS|SAML}}
- [ ] Authorization model: {{RBAC|ABAC}}
- [ ] Service-to-service auth: {{YES ({{Method}})|NO}}

### Security Scanning

- [ ] Container image scanned on build
- [ ] Dependency vulnerability scanning enabled
- [ ] SAST/DAST in CI pipeline: {{YES|NO}}

### Compliance

{{Applicable standards: GDPR, PCI-DSS, SOC 2, HIPAA, etc.}}

- [ ] {{Standard 1}}: {{Status}}
- [ ] {{Standard 2}}: {{Status}}

**Phase 5 Status**: {{✅ COMPLETE | ⚠️ IN PROGRESS | ❌ INCOMPLETE}}

---

## Phase 6: Production Readiness

### Observability Setup

#### Logging

- [ ] Structured logging implemented (JSON format)
- [ ] Log level configurable
- [ ] Log aggregation: {{ELK|GRAFANA LOKI|CLOUDWATCH|OTHER}}
- [ ] Sensitive data masked

#### Metrics

- [ ] Key metrics exposed: {{RED method (Rate, Errors, Duration)}}
- [ ] Custom business metrics: {{YES|NO}}
- [ ] Metrics collection: {{PROMETHEUS|DATADOG|CLOUDWATCH|OTHER}}

#### Tracing

- [ ] Distributed tracing enabled
- [ ] Tool: {{JAEGER|ZIPKIN|OPENTELEMETRY|OTHER}}
- [ ] Sampling rate: {{X}}%

#### Dashboards

- [ ] Service overview dashboard created
- [ ] Key SLO/SLI dashboards: {{YES|NO}}
- [ ] Alerting rules configured

### Alerting

- [ ] Alert rules defined for critical paths
- [ ] Alert routing configured (PagerDuty/OpsGenie)
- [ ] On-call rotation established
- [ ] Alert thresholds documented

### Runbooks

- [ ] Deployment runbook created
- [ ] Rollback runbook created
- [ ] Troubleshooting runbook created
- [ ] Incident response runbook created
- [ ] Escalation contacts documented

### Documentation

- [ ] README.md created with setup instructions
- [ ] API documentation (OpenAPI/Swagger)
- [ ] Architecture decision records (ADRs)
- [ ] Onboarding guide for new team members

**Phase 6 Status**: {{✅ COMPLETE | ⚠️ IN PROGRESS | ❌ INCOMPLETE}}

---

## Checklist Summary

### Must-Have (PRR Gate)

- [ ] Architecture approved
- [ ] API design reviewed
- [ ] Database schema approved
- [ ] CI/CD pipeline functional
- [ ] Security review completed
- [ ] Observability configured
- [ ] Runbooks created
- [ ] On-call rotation established
- [ ] SLOs defined and documented

### Should-Have (Best Practices)

- [ ] Load testing completed
- [ ] Chaos engineering tested
- [ ] DR/BCP documented
- [ ] Cost estimation done
- [ ] Documentation complete

### Nice-to-Have (Post-Launch)

- [ ] Performance optimization
- [ ] Advanced security hardening
- [ ] Feature flag system
- [ ] A/B testing framework

---

## PRR Readiness Score

### Score Calculation

| Category | Weight | Score | Weighted |
|----------|--------|-------|----------|
| Reliability | 20% | {{X}}/10 | {{X}} |
| Observability | 15% | {{X}}/10 | {{X}} |
| Security | 20% | {{X}}/10 | {{X}} |
| Deployment | 15% | {{X}}/10 | {{X}} |
| Documentation | 10% | {{X}}/10 | {{X}} |
| Testing | 10% | {{X}}/10 | {{X}} |
| Operations | 10% | {{X}}/10 | {{X}} |

**Total PRR Score**: {{X}}/10

### Scoring Guide

| Score | Readiness | Action |
|-------|-----------|--------|
| 8-10 | READY | Can proceed to production |
| 5-7 | CONDITIONAL | Proceed with documented risks |
| <5 | NOT READY | Significant work required |

---

## Open Issues

### Critical Issues (Must Fix Before Go-Live)

- [ ] **{{Issue 1}}** — {{Description}}
  - Owner: {{Name}} | Deadline: {{DATE}}

### Technical Debt

| Item | Severity | Owner | Target Fix |
|------|----------|-------|------------|
| {{Debt item}} | {{HIGH|MED|LOW}} | {{Name}} | {{DATE}} |

---

## Deployment Plan

### Pre-Deployment Checklist

- [ ] All critical issues resolved
- [ ] Security approval received
- [ ] On-call team notified
- [ ] Rollback plan tested
- [ ] Staging environment validated
- [ ] Stakeholders informed

### Deployment Timeline

| Step | Action | Duration | Owner |
|------|--------|----------|-------|
| 1 | {{Pre-deployment validation}} | {{X}} min | {{Name}} |
| 2 | {{Deploy to production}} | {{X}} min | {{Name}} |
| 3 | {{Verification checks}} | {{X}} min | {{Name}} |
| 4 | {{Monitor for X hours}} | {{X}} hours | {{Name}} |

### Rollback Plan

- [ ] Trigger conditions defined
- [ ] Rollback procedure documented
- [ ] Rollback tested in staging

---

## Sign-offs & Approvals

- [ ] **SRE Lead** ({{NAME}}) — Production readiness approved
  - Signature: ________________________ Date: __________

- [ ] **Architecture Lead** ({{NAME}}) — Architecture validated
  - Signature: ________________________ Date: __________

- [ ] **Security Lead** ({{NAME}}) — Security review approved
  - Signature: ________________________ Date: __________

- [ ] **Service Owner** ({{NAME}}) — Acknowledged and committed
  - Signature: ________________________ Date: __________

---

## Post-Launch Plan

### First 24 Hours

- [ ] SRE on-call monitoring closely
- [ ] Hourly health checks
- [ ] Daily standup with service team
- [ ] Be ready to rollback if needed

### First Week

- [ ] Daily metrics review
- [ ] Monitor for data drift or unusual behavior
- [ ] Collect initial user feedback

### First Month

- [ ] Monthly review with service team
- [ ] Validate SLO achievement
- [ ] Address technical debt

---

**Report prepared by**: {{SRE_LEAD}}
**Report date**: {{DATE}}
**Last updated**: {{DATE}}
