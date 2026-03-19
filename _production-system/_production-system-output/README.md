# _production-system-output/

> Reference directory for PSM (Production Systems & MLOps) workflow outputs.
> Last updated: 2026-03-19 | Owner: DoanNgocCuong

---

## Purpose

This directory contains **sample/reference outputs** from PSM workflows. These are realistic examples showing what the AI produces when running each workflow.

**Important**: At runtime, actual BMAD outputs land in `{project-root}/_bmad-output/psm-artifacts/`. This directory (`_production-system-output/`) is for **documentation and reference only** — it lives inside the `_production-system/` package for team reference, not as a runtime output destination.

---

## Sample Files

| File | Workflow | Description |
|------|----------|-------------|
| `sample-architecture-review.md` | review-architecture | Full 3-step architecture review report for a microservices platform |
| `sample-service-setup.md` | setup-new-service | 6-step service setup output (auth-service) |
| `sample-mlops-deployment.md` | mlops-deployment | 3-step ML model deployment report (recommendation model) |
| `sample-quick-diagnose.md` | quick-diagnose | 2-step diagnosis for high-latency issue (N+1 query problem) |
| `sample-quick-review.md` | quick-review | 2-step quick code review (API endpoint) |
| `quick-start-guide.md` | — | How to onboard PSM into a new project in 4 steps |

---

## Output Naming Convention

All workflow outputs follow the pattern:

```
{project-root}/_bmad-output/psm-artifacts/{workflow}-{identifier}-{date}.md
```

Examples:
- `architecture-review-{{SERVICE_NAME}}-2026-03-19.md`
- `service-setup-{{SERVICE_NAME}}-2026-03-19.md`
- `mlops-deploy-{{MODEL_NAME}}-2026-03-19.md`
- `quick-diagnose-{{DATE}}.md`
- `quick-review-{{DATE}}.md`

---

## BMAD Compliance

- **Version**: v6.2
- **Compliance**: 98%
- **State tracking**: `stepsCompleted`, `lastStep`, `lastSaved` frontmatter in all outputs
- **Checkpoint protocol**: Progressive save + menu after each step
- **Session continuation**: Detected via `step-01b-continue.md` files
