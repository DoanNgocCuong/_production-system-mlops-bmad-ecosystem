# Setup Production Service Workflow — BMAD

**Version**: 6.2.0
**Framework**: BMAD (Breakthrough Method for Agile AI-Driven Development)
**Created**: 2026-03-17
**Status**: Production Ready

---

## Overview

This workflow walks through setting up a **production-grade service from scratch**, covering all critical dimensions: architecture, API design, database selection, CI/CD, reliability, security, and production readiness.

**Total workflow**: ~2,158 lines of content across 7 files
**Estimated duration**: 12-20 hours
**Team**: SRE Minh (Lead), Architect Khang (Co-lead), Mary Analyst (Research)

---

## Workflow Structure

```
setup-new-service/
├── workflow.md                          [Entry point - 115 lines]
├── README.md                            [This file]
└── steps/
    ├── step-01-architecture.md          [192 lines - 2-3 hours]
    ├── step-02-api-database.md          [260 lines - 2-3 hours]
    ├── step-03-build-deploy.md          [270 lines - 2.5-3 hours]
    ├── step-04-reliability.md           [367 lines - 2.5-3 hours]
    ├── step-05-security-infra.md        [484 lines - 2.5-3 hours]
    └── step-06-readiness.md             [470 lines - 1.5-2 hours]
```

---

## Quick Start

1. **Begin workflow**: Open `workflow.md`
2. **Choose entry point**: Select `[NEW]` to start fresh
3. **Follow step chain**: Each step loads the next step automatically
4. **Save outputs**: Documents generated in `./outputs/setup-new-service-{service_name}/`

---

## Step-by-Step Breakdown

### Step 1: Architecture & Requirements Gathering
**Lead**: Architect Khang | **Duration**: 2-3 hours

Understand the service requirements and select architecture pattern:
- Business context and goals
- Technical requirements (scale, latency, availability)
- Architecture pattern selection (Monolith/Microservices/Serverless)
- Service boundaries and API contracts

**Output**: `architecture-design-document.md`

---

### Step 2: API Design & Database Strategy
**Lead**: Architect Khang | **Duration**: 2-3 hours

Finalize API contracts and database technology selection:
- API protocol selection (REST/GraphQL/gRPC)
- API contract specification with request/response schemas
- Database technology selection (PostgreSQL/MongoDB/Redis/DynamoDB)
- Schema design and indexing strategy

**Output**: `api-design-document.md` + `database-schema.md`

---

### Step 3: Build, Testing & Deployment Pipeline
**Lead**: SRE Minh | **Duration**: 2.5-3 hours

Establish automated CI/CD and testing infrastructure:
- CI/CD platform selection (GitHub Actions/GitLab CI/Jenkins)
- Docker multi-stage builds and image optimization
- Testing strategy (unit/integration/E2E test ratios and coverage)
- Code quality gates (linting, type checking, static analysis)
- Artifact registry and versioning

**Output**: `ci-cd-configuration.md` + `dockerfile` + `test-strategy.md`

---

### Step 4: Reliability, Observability & Error Handling
**Lead**: SRE Minh | **Duration**: 2.5-3 hours

Build resilient service with comprehensive observability:
- Resilience patterns (circuit breaker, retry, timeout, fallback)
- Structured logging (JSON logs with correlation IDs)
- Metrics & alerting (Prometheus, Grafana, alert rules)
- Distributed tracing (OpenTelemetry instrumentation)
- Error handling strategy and categorization

**Output**: `reliability-plan.md` + `observability-config.md` + `error-handling.md`

---

### Step 5: Security, Infrastructure & Deployment
**Lead**: SRE Minh | **Duration**: 2.5-3 hours

Secure service and prepare infrastructure for production:
- Authentication/Authorization (JWT, OAuth2, RBAC)
- Secret management (HashiCorp Vault setup and rotation)
- Kubernetes deployment manifests (complete YAML with best practices)
- Network security (Ingress, NetworkPolicies, TLS/mTLS)
- Data protection (encryption at rest and in transit)

**Output**: `security-plan.md` + `kubernetes-deployment.yaml` + `infrastructure-config.md`

---

### Step 6: Production Readiness Review & Go/No-Go Decision
**Lead**: SRE Minh | **Duration**: 1.5-2 hours

Comprehensive production readiness assessment:
- Production Readiness Checklist (50+ criteria across all dimensions)
- Runbook creation (common scenarios and escalation procedures)
- Incident response plan (SLA classification, communication flow)
- Team readiness (on-call rotation, training, knowledge transfer)
- Final go/no-go decision with signed approval

**Output**: `production-readiness-report.md` + `runbook.md` + `incident-response-guide.md`

---

## BMAD Pattern Applied

This workflow follows the **BMAD (Breakthrough Method for Agile AI-Driven Development)** pattern:

### Key Features

1. **Metadata-driven**: Each file has YAML frontmatter with clear metadata
2. **Step-chain architecture**: Each step knows the next step (no jumping)
3. **Mandatory rules**: Non-negotiable requirements at each step
4. **State tracking**: Progress saved in YAML for resumability
5. **Agent collaboration**: Lead agent + supporting agents specified
6. **Skill loading**: Each step auto-loads relevant SKILL files
7. **Menu-driven execution**: [A] Accept or [C] Continue refining

### Menu Pattern

Each step offers exactly 2 menu options:

- **[A] Accept**: Confirms decisions and moves to next step
- **[C] Continue**: Loops back to refine and ask more questions

This prevents premature advancement while keeping momentum.

---

## Mandatory Workflow Rules

1. **No skipping steps** — Each step must be executed in order
2. **Evidence required** — Decisions must be documented with rationale
3. **Validation checkpoints** — Each step validates against team standards
4. **Handoff protocol** — Context clearly transferred between steps
5. **Output artifacts** — Every step produces tangible documents
6. **Time-boxing** — Steps are designed to fit estimated durations
7. **Team alignment** — Lead agent + supporting agents coordinate

---

## Configuration

The workflow automatically loads configuration from `_config/config.yaml`:

```yaml
project_context:
  user_name: "[your name]"
  organization: "[organization]"
  environment: "production"

workflow_defaults:
  communication_language: "Vietnamese"
  output_folder: "./outputs/setup-new-service-{service_name}"
  timestamp: "2026-03-17"
```

Customize these values before starting the workflow.

---

## Output Structure

Upon completion, workflow generates structured outputs:

```
outputs/setup-new-service-{service_name}/
├── architecture-design-document.md
├── api-design-document.md
├── database-schema.md
├── ci-cd-configuration.md
├── dockerfile
├── test-strategy.md
├── reliability-plan.md
├── observability-config.md
├── error-handling.md
├── security-plan.md
├── kubernetes-deployment.yaml
├── infrastructure-config.md
├── production-readiness-report.md
├── runbook.md
├── incident-response-guide.md
└── workflow-progress.yaml          [State tracking]
```

---

## Agent Roles

### SRE Minh (Lead for Infrastructure/Operations)
- Expertise: Kubernetes, DevOps, SRE practices
- Primary in: Steps 3, 4, 5, 6
- Focus: Reliability, observability, deployment

### Architect Khang (Lead for Design/Architecture)
- Expertise: System design, technology selection
- Primary in: Steps 1, 2
- Focus: Architecture patterns, API design, database selection

### Mary Analyst (Supporting Researcher)
- Expertise: Requirements analysis, stakeholder interview
- Primary in: Step 1 (supporting)
- Focus: Requirements gathering, risk assessment

---

## How to Resume

If the workflow is interrupted:

1. Load `workflow-progress.yaml` from outputs folder
2. System auto-detects completed steps
3. Resume from next incomplete step
4. All previous context is preserved

---

## Skill Files Referenced

The workflow automatically loads and references these SKILL files:

```
5.01 Architecture & Design
5.02 API Design
5.03 Database & Data Management
5.04 Deployment & Infrastructure
5.05 Testing Strategy
5.06 Code Quality & Standards
5.07 Reliability & Resilience
5.08 Observability & Monitoring
5.09 Error Handling & Recovery
5.10 Production Readiness
5.11 Security & Compliance
5.12 Configuration Management
5.13 Infrastructure & Deployment
5.14 Documentation & Runbooks
```

These files provide deep-dive content for each domain.

---

## Key Deliverables at Each Step

| Step | Primary Output | Key Decisions | Artifacts |
|------|---|---|---|
| 1 | Architecture document | Pattern selection, tech stack | Requirements, boundaries |
| 2 | API + Database design | Protocol & DB choice | Schemas, contracts |
| 3 | CI/CD + Testing | Platform, test ratios | Dockerfile, pipeline config |
| 4 | Reliability plan | Resilience patterns, alerts | Monitoring, traces |
| 5 | Infrastructure + Security | K8s manifests, secrets | YAML configs, policies |
| 6 | PRR + Runbooks | Go/no-go decision | Runbook, incident plan |

---

## Common Scenarios

### Scenario: Resume from Step 3

1. Locate `workflow-progress.yaml` in outputs
2. Open `step-03-build-deploy.md`
3. System loads context from previous steps
4. Continue from Step 3

### Scenario: Refine Architecture Decision

1. At Step 1, choose [C] Continue
2. Answer additional clarifying questions
3. Re-evaluate pattern selection
4. Choose [A] Accept when ready

### Scenario: Parallel Work

Multiple team members can work on different aspects:
- Architect Khang on Steps 1-2 (design)
- SRE Minh on Steps 3-5 (implementation)
- Mary Analyst on Step 1 (research)

Context is shared via output documents.

---

## Success Metrics

Workflow is successful when:

- ✓ All 50+ items in PRR checklist completed
- ✓ Zero critical blockers identified
- ✓ All output documents generated
- ✓ Runbook created and tested
- ✓ Team alignment on go/no-go decision
- ✓ Service deployed to production within SLA

---

## Support & Questions

For questions about this workflow:

1. Check the BMAD Master Guide: `00-BMAD-MASTER-GUIDE.md`
2. Review step-specific guidance in each step file
3. Reference the SKILL files for deep dives
4. Contact workflow lead: SRE Minh

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 6.2.0 | 2026-03-17 | Initial production release |

---

## License

MIT License - This workflow is part of the BMAD framework.

