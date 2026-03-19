---
step_id: RA-S01
step_name: Context
workflow_id: RA001
sequence: 1
type: "scoping"
agent: "Khang (Architect)"
description: "Gather architecture artifacts, decisions, constraints"
estimated_time: "45-60 minutes"
is_final_step: false
outputFile: "{project-root}/psm/outputs/review-architecture/architecture-review-report.md"
---

# Step 01: Gather Architecture Context

## Objective

Collect comprehensive architecture information for analysis:
1. Understand system components and relationships
2. Learn historical decisions and constraints
3. Identify business/operational context
4. Establish baseline for dimensional analysis

## BMAD v6.2 Checkpoint Protocol

**Before proceeding, ensure output document exists with YAML frontmatter:**
```yaml
---
stepsCompleted: []
lastStep: "step-01-context.md"
lastSaved: "{current-timestamp}"
workflowType: "review-architecture"
---
```

**SAVING RULE:** After completing all execution sections, BEFORE loading the next step:
1. Update output document: append current step's output to the document
2. Update frontmatter: add this step to `stepsCompleted`, update `lastStep` and `lastSaved`
3. Save the document

---

## Execution Flow

### Substep 1A: Introduction & Scope

As Khang (Architect), introduce the review:

```
"Hi! I'm Khang, your architect for this review.

Over the next 3 hours, we'll evaluate your architecture against
key dimensions:
- Scalability: Can it grow 10x?
- Reliability: Can it survive failures?
- Maintainability: Is the code organized and clear?
- Operational Complexity: Can your team operate it?
- Cost Efficiency: Are resources used well?

By the end, we'll have recommendations and action items for the next quarter."
```

### Substep 1B: Architecture Overview

Collect high-level understanding:

```
QUESTIONS:

1. System Purpose & Domain
   "What does this system do? What's the core business value?"

2. Components & Topology
   "What are the main components? Draw a diagram or describe:
    - Frontend (web, mobile, API clients)
    - Backend services (monolith or microservices?)
    - Data stores (databases, caches, search)
    - External integrations (payment, auth, analytics)"

3. Scale & Growth
   "How many users? How fast is it growing?
    - Current: [X] users, [Y] requests/sec
    - 6-month target: [X*2] users, [Y*2] requests/sec
    - 1-year target: [X*5] users, [Y*5] requests/sec"

4. Tech Stack
   "What technologies?
    - Language/Framework (Java, Python, Node.js, Go?)
    - Primary database (PostgreSQL, MongoDB, DynamoDB?)
    - Message queue (Kafka, RabbitMQ, SQS?)
    - Caching (Redis, Memcached)
    - Infrastructure (Kubernetes, EC2, cloud-native?)"
```

Document in markdown with ASCII diagram:

```markdown
## System Architecture Overview

### Current Topology

```
                    Users (5K DAU, 500 RPS)
                            |
                  [API Gateway / Load Balancer]
                            |
        ____________________|_____________________
        |                   |                     |
    [Auth Service]  [Order Service]  [Payment Service]
      (1 pod)         (3 pods)          (2 pods)
        |                   |                     |
        |___________________|_____________________|
                            |
                  [PostgreSQL Cluster]
                  (1 master, 2 replicas)
                            |
                        [Redis Cache]
                        (single instance)

### Current Tech Stack
- API: Go 1.21, Gin framework
- Database: PostgreSQL 14
- Cache: Redis 7
- Infrastructure: Kubernetes 1.27
- CI/CD: GitHub Actions
```

### Substep 1C: Deployment & Operations

Understand how system runs:

```
QUESTIONS:

1. Deployment Strategy
   "How do you deploy?
    - Manual? Automated? Blue-green? Canary?
    - How long does a deployment take?
    - Can you rollback? How long?"

2. SLOs & Health
   "What's the target SLO?
    - Availability target (99%, 99.9%, 99.99%)
    - Latency targets (p50, p95, p99)
    - Error rate tolerance
    - Current uptime?"

3. Operational Burden
   "How many people operate this?
    - Team size?
    - On-call rotation frequency?
    - Incident frequency (per month)?
    - Time spent on operational tasks?"

4. Scaling Approach
   "How does it scale?
    - Horizontal (add more pods/instances)?
    - Vertical (bigger machines)?
    - Database sharding?
    - Caching strategy?"
```

### Substep 1D: Known Constraints & Decisions

Document decision history:

```
ARCHITECTURE DECISION LOG:

1. Why microservices instead of monolith?
   - Chosen because: Team structure, independent scaling
   - Trade-off: Complexity vs flexibility

2. Why PostgreSQL instead of MongoDB?
   "
   - Chosen because: ACID transactions needed for orders
   - Trade-off: Less flexible schema vs data consistency

3. Why single Redis instead of cluster?
   - Chosen because: Cost optimization, acceptable for current scale
   - Risk: Single point of failure
   - Mitigation: In-memory fallback for cache misses

4. Current Pain Points?
   "
   - Database scaling is a concern (approaching 1M orders/day)
   - Microservice latency (3 hops = 300ms sometimes)
   - On-call burden (3 incidents/month)"
```

### Substep 1E: Business & Operational Context

Understand priorities:

```
QUESTIONS:

1. Business Context
   "What's the business goal?
    - Growth target (2x revenue)?
    - User retention focus?
    - Cost reduction?
    - New market entry?"

2. Constraints
   "What are the real constraints?
    - Budget ceiling?
    - Team size (can't hire)?
    - Technical debt to address?
    - Regulatory/compliance needs?"

3. Pain Points
   "What keeps stakeholders up at night?
    - Reliability issues?
    - Scaling challenges?
    - Team productivity?
    - Cost overruns?"

4. Success Metrics
   "How do we measure success?
    - Performance improvements (latency, throughput)?
    - Reliability improvements (SLO achievement)?
    - Cost reduction?
    - Team productivity (deployment frequency)?
    - Developer experience?"
```

### Substep 1F: Preparation for Step 02

Summarize context:

```markdown
## Context Summary

**System**: Order Management Platform
**Team Size**: 8 engineers (2 infra, 6 backend)
**Current Scale**: 5K DAU, 500 RPS, 1M orders/month
**SLO Target**: 99.5% availability
**Pain Points**:
  1. Database scaling challenges (approaching single-node limit)
  2. Microservice latency (3-hop calls taking 300ms)
  3. On-call burden (3 incidents/month)
  4. Developer onboarding (new devs need 2 weeks to understand system)

**Key Decision**: Expand from 1 to 3 payment services (for PCI isolation)

**Constraints**:
  - Limited budget for new tools/infrastructure
  - Team growth limited to 2 engineers this year
  - Must maintain compatibility with existing API contracts
```

### Substep 1G: Transition to Step 02

Confirm and proceed:

```
"Excellent! I have a solid understanding of your system.

In Step 02, I'll analyze your architecture against 5 dimensions:
1. Scalability — Can it handle 10x growth?
2. Reliability — Can it survive failures gracefully?
3. Maintainability — Is the code organized and clear?
4. Operational Complexity — Can your team operate it sustainably?
5. Cost Efficiency — Are resources used optimally?

For each dimension, I'll score: GREEN (good), YELLOW (manageable), RED (concern)

Ready to move forward?"
```

**Proceed to [Step 02](step-02-analysis.md)**

## Deliverables from This Step

1. **Architecture Diagram** (ASCII or formal)
   - Components, relationships, data flows
   - Deployment topology
   - Clear labels and annotations

2. **System Context Document**
   - Purpose, scale, tech stack
   - SLOs and operational targets
   - Known pain points

3. **Decision Log**
   - Key architectural choices
   - Trade-offs and rationale
   - Current constraints

4. **Business Context**
   - Goals and success metrics
   - Constraints (budget, team, time)
   - Stakeholder priorities

---

## Checkpoint

**Select an Option:**
**[C] Continue to step-02-analysis.md**
**[A] Advanced Elicitation**
**[P] Party Mode**
**[Y] YOLO**

Menu Handling:
- IF C: Update frontmatter (add this step to stepsCompleted, update lastStep, lastSaved), save document, then read fully and follow `./step-02-analysis.md`
- IF A: Perform advanced elicitation on current step output
- IF P: Invite other agents to party mode
- IF Y: Proceed without stopping
- IF Any other: respond and redisplay menu

**EXECUTION RULES:**
- 🛑 ALWAYS halt and wait for user input after presenting checkpoint menu
- 🛑 NEVER load the next step file until user selects 'C'
- 🛑 ALWAYS update frontmatter BEFORE loading next step
- 📖 Read the entire next step file before execution
