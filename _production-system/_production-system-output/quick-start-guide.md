# Quick Start Guide: Using PSM in a New Project

> How to onboard the Production Systems & MLOps (PSM) module into any project.
> Estimated setup time: 10-15 minutes
> Owner: DoanNgocCuong | Last updated: 2026-03-19

---

## Overview

PSM (Production Systems & MLOps) is a BMAD module that brings structured production engineering workflows to your Claude Opus projects. With PSM, you get:

- **8 workflows** covering architecture review, service setup, incident response, MLOps deployment, and more
- **20 skill domains** (5.01-5.20) for deep production engineering knowledge
- **5 specialized agents** (Khang, Minh, Ha, Linh) with bilingual Vietnamese-English communication
- **Session continuation** — never lose progress if a session is interrupted
- **Checkpoint saves** — progressive saves after each workflow step
- **BMAD v6.2 compliance** — 98% aligned with BMAD methodology

---

## Step 1: Copy PSM to Your Project

Copy the `_production-system/` folder into your project root:

```
{your-project-root}/
├── your-project-files/
├── _bmad-prod/                    ← Create this folder
│   └── _production-system/       ← Copy PSM here
└── _bmad-output/                  ← Create this (runtime outputs)
    └── psm-artifacts/             ← AI saves outputs here
```

Alternatively, copy directly:

```bash
cp -r _production-system/ /path/to/your-project/_bmad-prod/_production-system/
```

---

## Step 2: Configure Your Project

Edit `_bmad-prod/_production-system/_config/config.yaml`:

```yaml
project_context:
  user_name: "Your Name"           # ← Update this
  organization: "Your Company"     # ← Update this
  project_name: "Your Project"     # ← Update this
  communication_language: "Vietnamese-English"  # or "English"
  environment: "development"

workflow_defaults:
  output_folder: "{project-root}/_bmad-output"
  timestamp_format: "YYYY-MM-DD"
```

---

## Step 3: Point Claude to the Master Agent

In your Claude Opus session, load the production master:

```
LOAD the FULL instructions from: _bmad-prod/_production-system/core/agents/prod-master.md
```

Or use the `.claude/skills/` layer for workflow discovery:

```
# Quick architecture review
LOAD the FULL instructions from: _bmad-prod/_production-system/.claude/skills/psm-review-architecture.md

# Quick diagnose
LOAD the FULL instructions from: _bmad-prod/_production-system/.claude/skills/psm-quick-diagnose.md
```

---

## Step 4: Choose a Workflow

Once the master agent is loaded, you'll see the PSM menu:

```
╔══════════════════════════════════════════════════════════════╗
║           PSM — Production Systems & MLOps (v6.2)           ║
║              Owner: DoanNgocCuong | BMAD: 98%                ║
╠══════════════════════════════════════════════════════════════╣
║  DESIGN & PLANNING                                            ║
║  [1] Review Architecture       — Evaluate system design      ║
║  [2] Setup New Service         — Build production service   ║
║                                                              ║
║  OPERATIONS                                                  ║
║  [3] Incident Response         — Triage → Diagnose → Fix    ║
║  [4] Production Readiness      — Pre-deploy validation     ║
║  [5] Security Audit           — Vulnerability assessment   ║
║                                                              ║
║  SPECIALIZED                                                  ║
║  [6] MLOps Deployment          — Deploy ML models to prod  ║
║                                                              ║
║  QUICK FLOW                                                  ║
║  [7] Quick Diagnose            — Fast production triage     ║
║  [8] Quick Review              — Rapid code/arch review     ║
║                                                              ║
║  CORE                                                        ║
║  [P] Party Mode                — Multi-agent discussion     ║
║  [A] Advanced Elicitation     — Deep analysis technique    ║
║  [K] Knowledge Lookup         — Search 20 skill domains    ║
║                                                              ║
║  Type number or letter to select workflow                    ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Key Features

### Session Continuation

If your Claude session is interrupted, simply reload the workflow and the AI will automatically detect and resume from where you left off:

```
LOAD the FULL instructions from: _bmad-prod/_production-system/psm/workflows/1-design/review-architecture/workflow.md
```

The AI checks for `step-N-output.md` files and `progress.yaml` to determine where to resume.

### Checkpoint Saves

After each workflow step, the AI saves a checkpoint and presents a menu:

```
[A] Advanced Elicitation — Go deeper on this step
[C] Continue — Move to next step
[P] Party Mode — Get multi-agent input
[Y] YOLO — Skip to end
[S] Save & Exit — Save checkpoint and pause
```

### BMAD v6.2 State Tracking

Every output document includes frontmatter tracking your progress:

```yaml
---
workflow_id: RA001
workflow_name: Architecture Review Report
stepsCompleted: [1, 2]
lastStep: 2
lastSaved: "2026-03-19T16:45:00+07:00"
---
```

### 20 Skill Domains

PSM comes with deep knowledge in 20 production engineering domains:

| Skill | Domain |
|-------|--------|
| 5.01 | Architecture Principles |
| 5.02 | System Design Patterns |
| 5.03 | Database (PostgreSQL, MongoDB, Redis) |
| 5.04 | Observability & Monitoring |
| 5.05 | Reliability Engineering |
| 5.06 | Performance Optimization |
| 5.07 | Incident Management |
| 5.08 | CI/CD & Deployment |
| 5.09 | Container Orchestration |
| 5.10 | Cloud Infrastructure |
| 5.11 | Security Hardening |
| 5.12 | API Security |
| 5.13 | Infrastructure as Code |
| 5.14 | Compliance & Governance |
| 5.15 | MLOps Fundamentals |
| 5.16 | Feature Engineering |
| 5.17 | Message Queues & Streaming |
| 5.18 | Cost Optimization |
| 5.19 | Chaos Engineering |
| 5.20 | Testing & Quality Assurance |

---

## Common Workflows

### Quick Diagnose (15-25 minutes)

When something breaks in production:

```
LOAD the FULL instructions from: _bmad-prod/_production-system/psm/workflows/quick-flow/quick-diagnose/workflow.md
```

Provide: symptom description, affected service, timeline, impact.

### Architecture Review (3-4 hours)

When designing or evaluating a system:

```
LOAD the FULL instructions from: _bmad-prod/_production-system/psm/workflows/1-design/review-architecture/workflow.md
```

Provide: architecture diagrams, decision logs, deployment topology, constraints.

### MLOps Deployment (3-4 hours)

When deploying a model to production:

```
LOAD the FULL instructions from: _bmad-prod/_production-system/psm/workflows/5-specialized/mlops-deployment/workflow.md
```

Provide: model artifact, performance metrics, serving infrastructure, A/B test plan.

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "Workflow not found" | Check path — ensure `_bmad-prod/_production-system/` is correct |
| "Skills not loading" | Run `SKILL: knowledge-lookup` from the prod-master menu |
| "Session lost progress" | Reload the workflow.md — AI auto-detects via `step-N-output.md` files |
| "Agent not responding" | Use `[P]` Party Mode to engage multiple agents |

---

## File Structure Reference

```
_bmad-prod/_production-system/
├── _config/                    # Manifests and configuration
│   ├── config.yaml            # Project settings
│   ├── agent-manifest.csv     # 5 agent definitions
│   ├── workflow-manifest.csv  # 8 workflow definitions
│   ├── skill-manifest.csv    # 20 skill definitions
│   └── agents/               # Per-agent customization
├── core/                      # Core layer (always needed)
│   └── agents/
│       └── prod-master.md    # Entry point — load this first
├── psm/                       # Production Systems module
│   ├── agents/               # 4 domain agents (Khang, Minh, Ha, Linh)
│   ├── workflows/            # 8 workflows
│   │   ├── 1-design/
│   │   ├── 2-build/
│   │   ├── 3-run/
│   │   ├── 4-cross/
│   │   ├── 5-specialized/
│   │   └── quick-flow/
│   ├── tasks/               # 6 reusable tasks
│   ├── templates/           # Output templates
│   └── teams/               # Team configurations
└── .claude/skills/          # IDE discovery layer (20 stubs)
```

---

## Next Steps

1. Read the sample outputs in `_production-system-output/` to see what each workflow produces
2. Try the **Quick Diagnose** workflow on your next production incident
3. Run an **Architecture Review** before your next major deployment
4. Explore the 20 skill domains with `SKILL: knowledge-lookup`

---

**BMAD Version**: v6.2
**PSM Module**: 5 - Production Systems & MLOps
**Compliance**: 98%
**Owner**: DoanNgocCuong
