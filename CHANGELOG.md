# CHANGELOG — BMAD Production Systems & MLOps

> Nhật ký chi tiết toàn bộ quá trình xây dựng package từ zero → production-ready.
> Owner: Đoàn Ngọc Cường | Started: 2026-03-18

---

## [2.0.0] — 2026-03-18 (Session cuối — Hoàn thiện theo BMAD chuẩn)

### BMAD Compliance Fixes

**Restructure toàn bộ theo đúng BMAD template (core/ + psm/):**
- Di chuyển `agents/prod-master.md` → `core/agents/prod-master.md` (master orchestrator thuộc core, không thuộc module)
- Di chuyển `agents/architect.md`, `sre.md`, `security.md`, `mlops.md` → `psm/agents/`
- Di chuyển `workflows/` → `psm/workflows/`
- Di chuyển `templates/` → `psm/templates/`
- Di chuyển `knowledge-base/` → `psm/knowledge-base/`
- Tạo `psm/data/`, `psm/teams/`, `psm/config.yaml`, `psm/module-help.csv`
- Tạo `core/config.yaml`, `core/module-help.csv`

**Tạo `.claude/skills/` — Layer 1 IDE Discovery (20 stubs):**
- Phát hiện thiếu hoàn toàn tầng "cầu nối IDE → runtime" sau khi đọc `FlowDataBMAD.md`
- 5 agent stubs: `psm-master`, `psm-architect`, `psm-sre`, `psm-security`, `psm-mlops`
- 8 workflow stubs: `psm-review-architecture`, `psm-setup-new-service`, `psm-incident-response`, `psm-production-readiness`, `psm-security-audit`, `psm-mlops-deployment`, `psm-quick-diagnose`, `psm-quick-review`
- 6 task stubs: `psm-advanced-elicitation`, `psm-adversarial-review`, `psm-incident-analysis`, `psm-knowledge-lookup`, `psm-production-checklist`, `psm-risk-assessment`
- 1 core workflow stub: `psm-party-mode`
- Format đúng BMAD: agent stubs dùng `<agent-activation>`, workflow stubs dùng `LOAD the FULL...`, task stubs dùng `Follow the instructions in...`

**Tạo 6 core/tasks (reusable across workflows):**
- `production-checklist/SKILL.md` — Quick 4-dimension check
- `adversarial-review/SKILL.md` — Phản biện production artifacts
- `advanced-elicitation/SKILL.md` — Ép AI đào sâu (dùng qua [A] menu)
- `incident-analysis/SKILL.md` — Phân tích logs/metrics/traces
- `risk-assessment/SKILL.md` — Đánh giá rủi ro thay đổi production
- `knowledge-lookup/SKILL.md` — Tìm kiếm trong 20 SKILL.md
- Mỗi task có `bmad-skill-manifest.yaml` đi kèm

**Tạo core/workflows:**
- `party-mode/workflow.md` — Multi-agent discussion (4 agents: Khang, Minh, Hà, Linh)

**Tạo _config/ đầy đủ theo BMAD chuẩn (12 items):**
- Thêm `bmad-help.csv` — Help routing (15 question patterns)
- Thêm `task-manifest.csv` — Registry 6 core tasks
- Thêm `tool-manifest.csv` — Header only (chưa có tools)
- Thêm `files-manifest.csv` — Index 49 files quan trọng
- Thêm `_config/agents/` — 5 customize yamls (override per-project)
- Thêm `_config/ides/` — `claude-code.yaml` + `cursor.yaml`
- Thêm `_config/custom/` — Placeholder cho user overrides

**Fix `_memory/` theo BMAD gốc:**
- Copy `tech-writer-sidecar/documentation-standards.md` nguyên từ BMAD (223 dòng)
- Copy `config.yaml` format từ BMAD
- Tạo `sre-sidecar/production-standards.md` (56 dòng) — SRE-specific persistent memory theo tech-writer pattern
- Xóa `.gitkeep.md` placeholder

**Fix `psm/teams/` theo BMAD format:**
- Tạo `default-party.csv` — CSV format đúng BMAD (persona columns: name, displayName, title, icon, role, identity, communicationStyle, principles, module, path)
- Tạo `team-production.yaml` — Bundle format (giống `team-fullstack.yaml` của BMAD)
- Fix `default-team.yaml` — Sửa mapping tên ↔ vai trò bị đảo (Khang ← SRE→Architect, Minh ← Architect→SRE, Linh ← Product→MLOps)

**Fix `psm/agents/bmad-skill-manifest.yaml`:**
- Sửa skill IDs sai (5.20, 5.21, 5.30, 5.31 → đúng: 5.04-5.10, 5.11-5.14, 5.15-5.20)
- Thêm canonicalId cho mỗi agent
- Thêm đầy đủ workflows mapping

**Tạo 6 checklist.md (validation gates):**
- `review-architecture/checklist.md` — 23 items, 5 categories
- `setup-new-service/checklist.md` — 33 items, 6 categories
- `incident-response/checklist.md` — 32 items, 5 categories
- `production-readiness-review/checklist.md` — 30 items, 5 categories
- `security-audit/checklist.md` — 28 items, 5 categories
- `mlops-deployment/checklist.md` — 27 items, 4 categories
- Tổng: 173 checkbox items, format `- [ ]` với emoji headers

**Thêm outputFile vào 8 workflow.md:**
- Tất cả workflows giờ có `outputFile` frontmatter trỏ tới `{output_folder}/psm-artifacts/`
- AI tự biết save output đi đâu khi chạy workflow

**Tạo web-bundles/ (Claude.ai Projects):**
- `sre-bundle.md` — SRE agent + incident-response + production-readiness (single-file)
- `architect-bundle.md` — Architect agent + review-architecture (single-file)

**Tạo psm/data/:**
- `severity-matrix.csv` — SEV1-4 classification
- `prr-dimensions.csv` — 9 PRR dimensions với weights
- `project-context-template.md` — Template cho project context mới

**Xóa `_output/`:**
- BMAD gốc không có `_output/` bên trong `_bmad/`
- Output nằm ở `{project-root}/_bmad-output/` bên ngoài package
- Fix `config.yaml`: `output_folder` trỏ tới `{project-root}/_bmad-output`

**Đổi master agent:**
- `prod-master.md` name: "Production Master" → "Đoàn Ngọc Cường"
- Identity + communication style cập nhật: "refers to himself as Cường"
- `agent-manifest.csv` displayName cập nhật

**Tạo 8 workflow bmad-skill-manifest.yaml:**
- Mỗi workflow có manifest riêng (name, type, lead_agent, skills_used, trigger_phrases)

---

## [1.0.0] — 2026-03-18 (Session đầu — Build từ zero)

### Phase 1: Deep Research BMAD

**Nghiên cứu BMAD gốc (`_bmad/` local + GitHub):**
- Đọc toàn bộ 282 files trong `_bmad/` (installed)
- Phân tích cấu trúc: `_config/` (manifests), `bmm/` (agents + workflows), `core/` (tasks)
- Đọc key files: `bmad-master.md`, `analyst.md`, `architect.md`, `config.yaml`, manifests
- Đọc workflow chi tiết: `create-product-brief/` (6 steps), `create-prd/` (12 steps), `code-review/`

**Nghiên cứu BMAD GitHub repo:**
- Fork: `https://github.com/DoanNgocCuong/BMAD-METHOD`
- Phân tích: `src/` structure (bmm, core, utility), `tools/`, `docs/`
- Đọc `CONTRIBUTING.md` (PR < 800 lines, trunk-based dev)
- Đọc `TRADEMARK.md` (community module naming convention)
- Phân tích module.yaml format, agent compilation pipeline
- Hiểu 3-layer: `.claude/skills/` → `_bmad/` → `steps/templates/data/`

**Phân tích FlowDataBMAD.md:**
- 2 cơ chế routing: Agent→Workflow (exec menu) vs Workflow trực tiếp (inline Your Role)
- `.claude/skills/` = Layer 1 bridge cho IDE discovery
- Workflow KHÔNG trỏ ngược ra agent (chỉ có inline role reinforcement)

**Output: `00-BMAD-MASTER-GUIDE.md` (1,988 dòng)**
- Deep research tổng hợp toàn bộ BMAD architecture
- Hướng dẫn từ zero → build workflow system giống BMAD

### Phase 2: Đánh giá 20 Skills vs Chuẩn quốc tế

**So sánh 20 production domains với 10 frameworks:**
- Google SRE Book, SRE Workbook, DORA Metrics, AWS Well-Architected (6 pillars), Azure Well-Architected, 12-Factor App, ISO 25010, NIST Cybersecurity Framework 2.0, CNCF Cloud Native Maturity Model, Platform Engineering Maturity Model

**Kết quả: 80% coverage (Senior IC level)**
- Xác định 5 critical gaps: Supply Chain Security, Team Structure, Business Metrics, Data Pipelines, Env Parity
- Xác định 5 medium gaps: Data Privacy, Load Balancing, IaC/GitOps, Multi-Cloud, LLMOps

### Phase 3: Quyết định chiến lược

**Fork vs Tách riêng → Làm cả hai:**
- Tầng 1: Standalone package (ưu tiên) — self-contained, copy đi dùng ngay
- Tầng 2: Cherry-pick 3-4 generic workflows → PR upstream vào BMAD repo
- Tầng 3: Community Module trên npm (tương lai)

**Lý do không chỉ fork:**
- BMAD = process framework (SDLC), PSM = knowledge framework (Production Engineering) — bản chất khác
- PR size limit 800 dòng, package > 25,000 dòng → 30+ PRs
- Knowledge rất specific (Langfuse, Docker troubleshooting cá nhân)

### Phase 4: Build Package v1.0

**Tạo cấu trúc folder ban đầu:**
- `_config/` — 5 manifests (config.yaml, manifest.yaml, agent/workflow/skill-manifest.csv)
- `agents/` — 5 agents (prod-master, architect, sre, security, mlops)
- `workflows/` — 8 workflows, 26 step files
- `templates/` — 3 templates (postmortem, PRR, security audit)
- `knowledge-base/` — 20 SKILL.md (copy từ `docs/4-production-quality/`)

**5 Agents (XML activation block format):**
- `prod-master.md` — Entry point, menu 13 items, routes to 4 agents
- `architect.md` — "Khang" (5.01-5.03), calm/pragmatic
- `sre.md` — "Minh" (5.04-5.10), metric-driven/SLO-first
- `security.md` — "Hà" (5.11-5.14, 5.18), thorough/zero-trust
- `mlops.md` — "Linh" (5.15-5.20), data-driven/experimental

**8 Workflows (26 step files tổng):**
1. `review-architecture/` — 3 steps (context → analysis → recommendations)
2. `setup-new-service/` — 6 steps (architecture → API/DB → build → reliability → security → readiness)
3. `incident-response/` — 4 steps (triage → diagnose → fix → postmortem)
4. `production-readiness-review/` — 3 steps (init → deep-review → decision)
5. `security-audit/` — 3 steps (scope → audit → report)
6. `mlops-deployment/` — 3 steps (model-validation → deploy → monitor)
7. `quick-diagnose/` — 2 steps (gather → diagnose)
8. `quick-review/` — 2 steps (context → review)

**3 Templates:**
- `incident-postmortem.template.md` — Timeline, RCA, impact, action items, lessons
- `production-readiness.template.md` — 9-dimension scorecard, GO/NO-GO decision
- `security-audit-report.template.md` — Findings by severity, compliance status

**20 SKILL.md knowledge base (12,992 dòng tổng):**
- Copy từ `docs/4-production-quality/5.01-5.20/SKILL.md`
- Mỗi file 275-1,221 dòng chứa: Overview, Core Principles, Best Practices, Decision Frameworks, Checklist, Common Mistakes, Code Examples

---

## Tiền sử — Trước khi build package

### 2026-03-13 — Tạo 20 domains từ 15 PHAN cũ

**Bối cảnh:** 301 files trong `_CKP_OLD/` đổ đống, đánh số La Mã PHAN_I → PHAN_XV

**Quyết định:**
- Chọn lifecycle grouping (DESIGN/BUILD/RUN/CROSS/SPECIALIZED) thay vì tech stack hay learning order
- Map 15 PHAN → 5 groups
- Tách PHAN_X (Database 76 files chứa cả RabbitMQ) → 5.03 Database + 5.17 Message Queues
- Gộp PHAN_I + PHAN_II + PHAN_IX → 5.01 Architecture + 5.13 Infrastructure
- Thêm 5 domains mới (Testing, Caching, Compliance, Chaos, Cost)
- Kết quả: 20 domains MECE, 227 files phân loại + 6 empty gaps

### 2026-03-15 — Bổ sung files từ NOTE-TECH

- 18 files tech → đúng domain trong 5.xx
- 8 files non-tech → redirect về MINDGame

### 2026-03-17 — Thêm skill template structure

- Top-level: `scripts/`, `references/`, `assets/`
- Tạo PowerShell script `RESTRUCTURE-5xx.ps1` cho restructure 20 sub-folders

---

## Stats tại thời điểm hoàn thành

| Metric | Value |
|--------|-------|
| Total files | 150 |
| Total size | 1.2 MB |
| Agents | 5 (1 core + 4 domain) |
| Workflows | 8 + 1 core (party-mode) |
| Step files | 26 |
| Checklist files | 6 (173 checkbox items) |
| Core tasks | 6 |
| Knowledge base | 20 SKILL.md (12,992 dòng) |
| Templates | 3 |
| Skill stubs (.claude/skills/) | 20 |
| Config/manifest files | 12 |
| Agent customize yamls | 5 |
| Web bundles | 2 |
| IDE configs | 2 |
| Sidecar memory files | 2 |

---

## Cần làm trên Windows (manual)

- [ ] Xóa `_output/` (thừa — output nằm ngoài package)
- [ ] Xóa `_memory/.gitkeep.md` (đã có config.yaml thật)
- [ ] Xóa `psm/teams/default-team.yaml` nếu muốn (đã có bản fix, nhưng giữ cũng được)
- [ ] Xóa `ide-configs/` cũ (đã có `_config/ides/` mới)
- [ ] Xóa `output - DeepResearchBMAD.md` ở root (file thừa)
- [ ] Xóa các empty ghost folders: `agents/`, `workflows/`, `tasks/`, `data/` ở root (overlay artifacts)
