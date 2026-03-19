# Production System Full Kit - BMAD Method

> A comprehensive production systems knowledge base built on the BMAD Method framework.
> "Full combo" version with everything needed for production-ready software development.
> 5 Agents, 8 Workflows, 20 Skill domains.

---

## Quick Start (3 steps)

### 1. Copy to your project

```bash
# Copy entire folder to your project
cp -r bmad-production-systems/ /path/to/your/project/_bmad-prod/
```

Or on Windows:
```powershell
Copy-Item -Recurse bmad-production-systems\ C:\your\project\_bmad-prod\
```

### 2. Edit config

Open `_config/config.yaml` and change:

```yaml
project_name: "YOUR_PROJECT_NAME"
user_name: "YOUR_NAME"
communication_language: en  # or Vietnamese-English
knowledge_base: "{project-root}/knowledge-base"  # or external path
```

### 3. Use it

Point your AI assistant (Claude Code, Cursor, etc.) to `agents/prod-master.md` as the entry point. The master agent will present a menu of all available workflows.

---

## What's Inside - Full Kit Components

```
production-system-full-kit-bmad-method/
├── README.md                          ← You are here
├── 00-BMAD-MASTER-GUIDE.md            ← Deep research: how BMAD works + how to build your own
├── CHANGELOG.md                       ← Version history
├── Draft.md                           ← Working drafts and notes
├── evaluation-BMAD-ProductionQuality.md ← Evaluation benchmarks
├── output - DeepResearchBMAD.md        ← Deep research output
├── _config/                           ← Configuration files
│   ├── config.yaml                    ← Project settings
│   ├── manifest.yaml                  ← Version info
│   ├── agent-manifest.csv             ← Agent registry
│   ├── workflow-manifest.csv          ← Workflow registry
│   └── skill-manifest.csv             ← 20 skill domains registry
├── _memory/                           ← Agent persistent memory
├── core/                              ← Core BMAD components
├── psm/                               ← PSM module (standalone)
├── workflows/                         ← Production workflows
├── ide-configs/                       ← Claude, Cursor, VSCode configs
├── web-bundles/                       ← Web UI bundles
└── knowledge-base/                    ← 20 SKILL.md files (optional)
```

## For Who

- Development teams building production systems
- SREs and DevOps engineers
- MLOps practitioners
- Anyone needing comprehensive production knowledge

## Part of BMAD Ecosystem

BMAD = Breakthrough Method for Agile AI-Driven Development

This is the extended/enterprise version complementing the core PSM module.

---

## Quick Start (3 steps)

### 1. Copy to your project

```bash
# Copy entire folder to your project
cp -r production-system-full-kit-bmad-method/ /path/to/your/project/_bmad-prod/
```

Or on Windows:
```powershell
Copy-Item -Recurse production-system-full-kit-bmad-method\ C:\your\project\_bmad-prod\
```

### 2. Edit config

Open `_config/config.yaml` and change:

```yaml
project_name: "YOUR_PROJECT_NAME"
user_name: "YOUR_NAME"
communication_language: en  # or Vietnamese-English
knowledge_base: "{project-root}/knowledge-base"  # or external path
```

### 3. Use it

Point your AI assistant (Claude Opus, Cursor, etc.) to `agents/prod-master.md` as the entry point. The master agent will present a menu of all available workflows.

---

## What's Inside - Core Module

```
bmad-production-systems/
├── README.md                          ← You are here
├── 00-BMAD-MASTER-GUIDE.md            ← Deep research: how BMAD works + how to build your own
├── _config/
│   ├── config.yaml                    ← Project settings (EDIT THIS)
│   ├── manifest.yaml                  ← Version info
│   ├── agent-manifest.csv             ← Agent registry
│   ├── workflow-manifest.csv          ← Workflow registry
│   └── skill-manifest.csv             ← 20 skill domains registry
├── _memory/                           ← Agent persistent memory (auto-populated)
├── _output/                           ← Generated artifacts land here
├── agents/
│   ├── prod-master.md                 ← Entry point — routes to all workflows
│   ├── architect.md                   ← "Khang" — Architecture, API, Database
│   ├── sre.md                         ← "Minh" — Reliability, Observability, Deployment
│   ├── security.md                    ← "Hà" — Security, Infrastructure, Compliance
│   └── mlops.md                       ← "Linh" — MLOps, Performance, Async
├── workflows/
│   ├── 1-design/review-architecture/  ← 3 steps
│   ├── 2-build/setup-new-service/     ← 6 steps (full service from zero)
│   ├── 3-run/
│   │   ├── incident-response/         ← 4 steps (triage → diagnose → fix → postmortem)
│   │   └── production-readiness-review/ ← 3 steps (9-dimension PRR)
│   ├── 4-cross/security-audit/        ← 3 steps
│   ├── 5-specialized/mlops-deployment/ ← 3 steps
│   └── quick-flow/
│       ├── quick-diagnose/            ← 2 steps (fast incident diagnosis)
│       └── quick-review/              ← 2 steps (fast architecture review)
├── templates/
│   ├── incident-postmortem.template.md
│   ├── production-readiness.template.md
│   └── security-audit-report.template.md
└── knowledge-base/                    ← Link or copy your 20 SKILL.md files here
```

## 5 Agents

| Agent | Name | Domain | Skills |
|-------|------|--------|--------|
| prod-master | — | Orchestrator | Routes to all agents |
| architect | Khang | System Design | 5.01 Architecture, 5.02 API, 5.03 Database |
| sre | Minh | Site Reliability | 5.04-5.10 (Build + Run) |
| security | Hà | Security & Infra | 5.11-5.14, 5.18 (Cross + Compliance) |
| mlops | Linh | MLOps & Performance | 5.15-5.20 (Specialized) |

## 8 Workflows

| Workflow | Steps | Agent | Use When |
|----------|-------|-------|----------|
| Review Architecture | 3 | Khang | "Review our architecture" |
| Setup New Service | 6 | Minh+Khang | "Set up new service from scratch" |
| Incident Response | 4 | Minh | "Production is down!" |
| Production Readiness | 3 | Minh | "Are we ready to go live?" |
| Security Audit | 3 | Hà | "Run security audit" |
| MLOps Deployment | 3 | Linh | "Deploy ML model to production" |
| Quick Diagnose | 2 | Minh | "Something is slow/broken" |
| Quick Review | 2 | Khang | "Quick review this code/design" |

## 20 Skill Domains

Organized by production lifecycle:

| Phase | Domains |
|-------|---------|
| DESIGN | 5.01 Architecture, 5.02 API Design, 5.03 Database |
| BUILD | 5.04 CI/CD, 5.05 Testing, 5.06 Code Quality |
| RUN | 5.07 Reliability, 5.08 Observability, 5.09 Error Handling, 5.10 Production Readiness |
| CROSS | 5.11 Security, 5.12 Config & Secrets, 5.13 Infrastructure, 5.14 Documentation |
| SPECIALIZED | 5.15 MLOps, 5.16 Caching, 5.17 Async/Queues, 5.18 Compliance, 5.19 Chaos Eng, 5.20 Cost |

## How to Connect Your Knowledge Base

Option A — Symlink (recommended):
```bash
ln -s /path/to/your/5-Production-Systems knowledge-base
```

Option B — Copy SKILL.md files:
```bash
mkdir -p knowledge-base
cp /path/to/5.01*/SKILL.md knowledge-base/5.01-SKILL.md
cp /path/to/5.02*/SKILL.md knowledge-base/5.02-SKILL.md
# ... etc
```

Option C — Edit config to point externally:
```yaml
knowledge_base: "/absolute/path/to/5 - Production Systems & MLOps"
```

## Extending the Framework

### Add a new workflow
1. Create `workflows/{phase}/{workflow-name}/workflow.md`
2. Create `workflows/{phase}/{workflow-name}/steps/step-01-xxx.md`
3. Add entry to `_config/workflow-manifest.csv`
4. Add menu item to relevant agent's `.md` file

### Add a new agent
1. Create `agents/{agent-name}.md` following XML activation pattern
2. Add entry to `_config/agent-manifest.csv`
3. Add routing in `agents/prod-master.md` menu

### Add a new skill domain
1. Add row to `_config/skill-manifest.csv`
2. Create SKILL.md in knowledge base
3. Reference from relevant workflow steps

---

## Credits

- Framework pattern: [BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD) v6.2
- BMAD = Breakthrough Method for Agile AI-Driven Development
- Content: DoanNgocCuong's Production Systems knowledge base (227 files, 20 domains)
- Built: 2026-03-18
