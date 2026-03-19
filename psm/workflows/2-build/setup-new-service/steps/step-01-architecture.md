---
step_number: 1
step_name: "Architecture & Requirements Gathering"
next_step_file: "steps/step-02-api-database.md"
estimated_time: "2-3 hours"
lead_agent: "Architect Khang"
supporting_agents: ["SRE Minh", "Mary Analyst"]
skill_loads:
  - "references/5.01 - DESIGN - Architecture & Design/SKILL.md"
---

# STEP 1: Architecture & Requirements Gathering

## STEP GOAL

Hiểu sâu về service mới và quyết định architecture pattern phù hợp:

- **Business context**: Vấn đề service giải quyết, objectives, users
- **Technical requirements**: Load expectations, data volume, latency requirements
- **Tech stack**: Language, framework, database types, infrastructure
- **Architecture pattern**: Monolith vs. Microservice vs. Serverless
- **Service boundaries**: APIs, domains, integration points

**Output**: `architecture-design-document.md` ready for step-02

## Skill Files to Load

```
Load: references/5.01 - DESIGN - Architecture & Design/SKILL.md
      → Architecture patterns, design principles, tech selection frameworks
```

## MANDATORY RULES

1. **Ask ALL clarifying questions** — không make assumptions
2. **Document rationale** — mỗi decision phải có reasoning rõ ràng
3. **Validate with stakeholders** — confirm requirements trước moving forward
4. **Consider scalability** — design for 10x growth từ day 1
5. **Time-box**: Max 120 minutes total execution

## EXECUTION PROTOCOL

### Phase 1: Service Context & Business Goals (20 min)

**Greeting**:

Xin chào! Tôi là Architect Khang. Chúng ta sẽ xây dựng architecture cho service mới.

Mình cần hiểu:
- Service này giải quyết vấn đề gì?
- Ai sẽ sử dụng?
- Business objectives là gì?
- Timeline deployment?

**Elicitation Questions**:

1. Service purpose: "Describe the primary business problem this service solves"
2. Primary users: "Who are the main users? Internal/external?"
3. Scale: "Expected users/requests per day at launch? Year 1?"
4. Criticality: "How critical is this service for business?"
5. Budget/Timeline: "What's the timeline and budget?"

**Document**: Capture responses in `business-context.md`

### Phase 2: Technical Requirements (25 min)

**Key Questions**:

1. Latency: "What's acceptable p99 latency?"
2. Throughput: "Expected RPS (requests per second)?"
3. Data volume: "Estimated data size at year 1?"
4. Availability: "Uptime SLA requirement?"
5. Compliance: "Any regulatory requirements (SOC2, HIPAA, GDPR)?"
6. Data residency: "Geographic constraints?"
7. Integration: "What systems need to integrate?"

**Non-Functional Matrix**:

| Aspect | Requirement | Justification |
|--------|-------------|---------------|
| Latency | < 200ms p99 | [fill in] |
| Throughput | 10k RPS peak | [fill in] |
| Uptime | 99.95% | [fill in] |
| Data Consistency | Strong/Eventual | [fill in] |

**Document**: Capture in `technical-requirements.md`

### Phase 3: Architecture Pattern Selection (35 min)

**Pattern Options to Evaluate**:

**Option A: Monolith**
- All logic in single process
- Pros: Simple deployment, easier debugging, lower ops overhead
- Cons: Scaling challenges, deployment rigidity
- Best for: < 5M users, simple domain, single team

**Option B: Microservices**
- Decomposed into multiple services per domain
- Pros: Independent scaling, polyglot, team autonomy
- Cons: Complex testing, network latency, distributed tracing overhead
- Best for: > 10M users, complex domains, multiple teams

**Option C: Serverless**
- Function-as-a-service (AWS Lambda, Google Cloud Functions)
- Pros: Zero infrastructure, pay-per-execution, auto-scaling
- Cons: Cold starts, vendor lock-in, state management complexity
- Best for: Event-driven, variable load, low volume

**Selection Decision Tree**:

```
1. Expected RPS at peak?
   → < 1k RPS → Consider Monolith
   → 1k-10k RPS → Consider Microservices with careful boundaries
   → > 10k RPS → Microservices required

2. Data consistency requirements?
   → Strong consistency → Monolith better
   → Eventual OK → Microservices viable

3. Team size?
   → < 5 engineers → Monolith simpler
   → 5-20 engineers → Microservices if well-designed
   → > 20 engineers → Microservices necessary

4. Operational maturity?
   → Low → Monolith + managed services (RDS, CDN)
   → High → Microservices with full K8s
```

**Decision Format**:

```
SELECTED PATTERN: [Monolith | Microservices | Serverless]

Rationale:
- [Reason 1]
- [Reason 2]
- [Reason 3]

Trade-offs Accepted:
- [Trade-off 1]
- [Trade-off 2]
```

### Phase 4: Service Boundaries & APIs (25 min)

**For Monolith**:
- Define core domains/modules
- Document API contract (REST/GraphQL endpoints)
- Define data model conceptually

**For Microservices**:
- Decompose into services by domain
- Define service boundaries
- Document API contracts between services
- Plan integration patterns (sync/async)

**API Contract Template**:

```
Service: [Name]

Endpoints:
- POST /v1/[resource] → Create
- GET /v1/[resource]/{id} → Retrieve
- PUT /v1/[resource]/{id} → Update
- DELETE /v1/[resource]/{id} → Delete

Dependencies:
- Calls: [Other services or systems]
- Called by: [Who depends on this]
```

**Document**: Create `service-boundaries.md` with all service definitions

## MENU - Choose Your Path

### [A] Accept architecture decision
- Confirms architecture pattern selected
- Moves to document generation
- Proceeds to step-02

### [C] Continue refining
- Asks more questions
- Re-evaluates pattern selection
- Loops back to clarification

---

**Bấm [A] để chấp nhận hoặc [C] để refine thêm**
