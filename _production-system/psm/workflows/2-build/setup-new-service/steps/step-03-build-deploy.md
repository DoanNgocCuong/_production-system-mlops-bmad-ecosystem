---
step_number: 3
step_name: "Build, Testing & Deployment Pipeline"
next_step_file: "steps/step-04-reliability.md"
estimated_time: "2.5-3 hours"
lead_agent: "SRE Minh"
supporting_agents: ["Architect Khang"]
skill_loads:
  - "references/5.04 - BUILD - Deployment & CI-CD/SKILL.md"
  - "references/5.05 - BUILD - Testing/SKILL.md"
  - "references/5.06 - BUILD - Code Quality & Maintainability/SKILL.md"
is_final_step: false
outputFile: "{project-root}/psm/outputs/setup-new-service/{service_name}/ci-cd-configuration.md"
workflowType: "setup-new-service"
---

# STEP 3: Build, Testing & Deployment Pipeline

## STEP GOAL

Establish automated CI/CD pipeline, containerization, and testing strategy:

- **CI/CD pipeline**: GitHub Actions / GitLab CI setup
- **Docker containerization**: Multi-stage builds, image optimization
- **Testing strategy**: Unit, integration, E2E test ratios and coverage targets
- **Code quality gates**: Linting, type checking, static analysis
- **Artifact registry**: Image storage and versioning

**Output**: `ci-cd-configuration.md` + `dockerfile` + `test-strategy.md` ready for step-04

## Skill Files to Load

```
Load: references/5.04 - BUILD - Deployment & CI-CD/SKILL.md
      → CI/CD platforms, deployment strategies, blue-green, canary

Load: references/5.05 - BUILD - Testing/SKILL.md
      → Testing pyramid, test ratios, coverage targets, E2E frameworks

Load: references/5.06 - BUILD - Code Quality & Maintainability/SKILL.md
      → Linting, formatting, type checking, static analysis
```

## MANDATORY RULES

1. **Test-first mindset** — Define tests before implementation
2. **Automated everything** — No manual deployment steps
3. **Fail fast** — Quality gates must run early in pipeline
4. **Clear stages** — Build → Test → Quality → Deploy phases
5. **Artifact versioning** — Every build produces versioned artifact

## BMAD v6.2 Checkpoint Protocol

**Before proceeding, ensure output document exists with YAML frontmatter:**
```yaml
---
stepsCompleted: ["step-01-architecture.md", "step-02-api-database.md"]
lastStep: "step-03-build-deploy.md"
lastSaved: "{current-timestamp}"
workflowType: "setup-new-service"
---
```

**SAVING RULE:** After completing all execution sections, BEFORE loading the next step:
1. Update output document: append current step's output to the document
2. Update frontmatter: add this step to `stepsCompleted`, update `lastStep` and `lastSaved`
3. Save the document

---

## EXECUTION PROTOCOL

### Phase 1: CI/CD Platform Selection (15 min)

**Platform Options**:

**Option A: GitHub Actions**
- Pros: Native GitHub integration, free for public, good for small teams
- Cons: Limited concurrency on free tier, workflow complexity
- Best for: GitHub-hosted projects

**Option B: GitLab CI**
- Pros: Free runners, excellent debugging, built-in Docker registry
- Cons: Learning curve, less ecosystem
- Best for: Self-hosted, full control needed

**Option C: Jenkins**
- Pros: Ultimate flexibility, mature ecosystem, self-hosted
- Cons: Complex setup, maintenance overhead
- Best for: Complex enterprises, existing Jenkins infrastructure

**Decision**: SELECTED = [GitHub Actions | GitLab CI | Jenkins]

**Rationale**: [Your justification]

### Phase 2: Docker Multi-Stage Build (25 min)

**Multi-stage Dockerfile Template**:

```dockerfile
# Stage 1: Builder
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build
RUN npm run test

# Stage 2: Runtime
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s \
  CMD node healthcheck.js || exit 1
CMD ["node", "dist/index.js"]
```

**Key Practices**:

```
1. Multi-stage: Separates build dependencies from runtime
2. Layer caching: Order Dockerfile by change frequency
3. Image size: Minimal base images (alpine, distroless)
4. Health checks: Always include HEALTHCHECK
5. Security: Non-root user, minimal privileges
6. Versioning: Tag images with semantic version + git commit hash
```

**Image Versioning Strategy**:

```
Tag Format: registry.example.com/service-name:v1.2.3-abc1234

Components:
- v1.2.3 → Semantic version from git tag
- abc1234 → First 7 chars of git commit SHA
- Timestamp → Latest build timestamp
```

### Phase 3: Testing Strategy (25 min)

**Testing Pyramid**:

```
                    /\
                   /  \        E2E Tests (5-10%)
                  /----\       - Full user journeys
                 /      \      - Slow, expensive
                /        \
               /----------\
              /            \   Integration Tests (15-20%)
             /              \  - Service interactions
            /                \ - Database, API calls
           /                  \
          /--------------------\ Unit Tests (70-80%)
         /                        \- Fast, isolated, single function
        /                          \- Mocks external dependencies
```

**Test Ratios**:

| Test Type | Coverage | Speed | Cost |
|-----------|----------|-------|------|
| Unit | 80% | < 100ms | Low |
| Integration | 40% | 100ms-2s | Medium |
| E2E | 20% | 2s-30s | High |

**Example Test Configuration**:

```yaml
# jest.config.js
module.exports = {
  testEnvironment: 'node',
  coverageThreshold: {
    global: {
      branches: 75,
      functions: 75,
      lines: 75,
      statements: 75
    }
  },
  collectCoverageFrom: [
    'src/**/*.{js,ts}',
    '!src/**/*.test.{js,ts}',
    '!src/index.{js,ts}'
  ]
};
```

**Test Coverage Goals**:

- Critical business logic: 90%+
- API handlers: 80%+
- Database queries: 75%+
- Error paths: 80%+
- E2E critical flows: 100%

### Phase 4: Code Quality Gates (20 min)

**Linting & Formatting**:

```yaml
# .eslintrc.json
{
  "extends": "eslint:recommended",
  "parserOptions": {
    "ecmaVersion": 2022,
    "sourceType": "module"
  },
  "rules": {
    "no-console": ["warn", { "allow": ["error", "warn"] }],
    "no-unused-vars": "error",
    "prefer-const": "error"
  }
}
```

**Type Checking** (TypeScript):

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  }
}
```

**Static Analysis** (SonarQube):

```
Security issues: 0 blocker
Code smell: < 10 minor
Technical debt: < 5%
Test coverage: > 75%
```

**CI/CD Pipeline Stages**:

```
Commit → Code Quality → Build → Unit Tests → Integration Tests →
Security Scan → Deploy Staging → E2E Tests → Deploy Production
```

### Phase 5: Artifact Registry & Versioning (15 min)

**Docker Registry Setup**:

```bash
# Example: Push to Docker Hub
docker tag service-name:latest myregistry/service-name:v1.2.3
docker push myregistry/service-name:v1.2.3

# Or: GitHub Container Registry
docker tag service-name:latest ghcr.io/myorg/service-name:v1.2.3
docker push ghcr.io/myorg/service-name:v1.2.3
```

**Semantic Versioning**:

```
v1.2.3
│ │ └─ Patch: Bug fixes
│ └──── Minor: New features, backward compatible
└────── Major: Breaking changes
```

**Document**: Create `docker-build-pipeline.md` with all configurations

## MENU - Choose Your Path

### [A] Accept CI/CD and testing design
- Confirms pipeline structure
- Validates test ratios
- Proceeds to step-04

### [C] Continue refining
- Revisits platform choice
- Adjusts test coverage targets
- Refines quality gates

---

## Checkpoint

**Select an Option:**
**[C] Continue to step-04-reliability.md**
**[A] Advanced Elicitation**
**[P] Party Mode**
**[Y] YOLO**

Menu Handling:
- IF C: Update frontmatter (add this step to stepsCompleted, update lastStep, lastSaved), save document, then read fully and follow `./step-04-reliability.md`
- IF A: Perform advanced elicitation on current step output
- IF P: Invite other agents to party mode
- IF Y: Proceed without stopping
- IF Any other: respond and redisplay menu

**EXECUTION RULES:**
- 🛑 ALWAYS halt and wait for user input after presenting checkpoint menu
- 🛑 NEVER load the next step file until user selects 'C'
- 🛑 ALWAYS update frontmatter BEFORE loading next step
- 📖 Read the entire next step file before execution
