# Changelog

All notable changes to the AAOF project are documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
AAOF uses [Semantic Versioning](https://semver.org/).

---

## [Unreleased]

---

## [0.5.0] — 2026-04-05

### Added

**Requirements Checklist & Contract Check (PR #20)**
- `rules/requirements_checklist.md` — Contract check mechanism between execution plan and produced artifacts
- `session/requirements_checklist.json` — Per-step requirements tracking with pass/fail/waived status
- `VAR_CONTRACT_CHECK` session variable — Blocks STEP 4→5 transition if requirements are not satisfied
- Example file: `session/requirements_checklist.json.example`

**Mandatory Artifacts Validation (PR #21)**
- Gate check validation for mandatory artifact existence, non-emptiness, minimum content length, and required sections
- Configurable per-step artifact requirements

**Requirements Traceability Matrix (PR #22, #23)**
- Explicit mapping between functional requirements and test files
- Gate check TDD validation: every requirement must have at least one associated test
- Coverage report (covered/uncovered) integrated in `step_evidence.json`

**Namespace Separation (PR #24)**
- Clear separation between framework artifacts (`.aaof/`) and project artifacts (`output/`)
- Gate check validation prevents writing project artifacts in framework paths and vice versa

**Template Scaffolding & Directory Validation (PR #25)**
- Automatic directory and file stub generation based on project configuration
- Gate check validation for empty directories (declared directories must contain application logic)

**Optional Refinement Loop (PR #26)**
- Step 5+ optional refinement loop with gap analysis between requirements and produced artifacts
- Automatic gap report with missing requirements, files, and corrective actions

**Auto-Scoring System**
- Automatic quality scoring (0–100) based on: artifact coverage, test coverage, security compliance, module complexity
- `output/scoring_report.md` generated at end of process with per-category scores and recommendations

**Multi-Session Orchestration**
- `AGENT_ORCHESTRATOR.md` — Complete guide for AI orchestrator agents
- `prompts/resume_prompt.md` — Resume instructions for relaunched sub-agents
- Turn limit recovery: orchestrator monitors `session_state.json` and relaunches sub-agents as needed
- Orchestrator state persistence in `.aaof/orchestrator_state.json`

---

## [Unreleased] (pre-v0.5.0)

### Added

**Testing Rules — Self-Execution Enforcement**
- `rules/testing_rules.md` §4.0 — Self-Execution Rule: agent MUST execute all tests
  itself inside Docker containers, MUST NOT delegate to user. Test playbook is a
  reproducibility document, not a delegation mechanism. Manual-only tests documented
  separately when agent cannot automate them

**Validation Hardening**
- `agent.md` STEP 5.5 — Evidence-Based Validation: `VAR_VALIDATION_RESULT` can only be
  set after agent directly observes executed test results. Explicitly prohibits setting
  PASS based on code review or user confirmation
- `agent.md` STEP 5.4.1 — Kubernetes Validation sub-step: mandatory second validation
  cycle on Minikube when `VAR_DEPLOY_TARGET` includes K8S (apply manifests, verify pods,
  health check, functional tests on K8s endpoints). When `VAR_MINIKUBE_APPROVED` is
  `false`, sub-step is skipped with a WARN logged; `VAR_VALIDATION_RESULT` is then based
  on Docker validation results alone

**Workflow Hardening**
- `agent.md` STEP 2 — Minimum 2 alternatives enforcement: plan must include at least 2
  architectural/technological alternatives with trade-offs per design review rules
- `agent.md` STEP 3 — SCRATCH Project Baseline: empty `backup_manifest.json` generated
  for new projects to establish non-regression baseline

**Prerequisites & Environment Checks**
- Docker Prerequisite Check at STEP 0 — mandatory verification of Docker and Docker Compose
  installation with auto-installation guidance. Blocking: framework stops if Docker is not
  available. Session variable: `VAR_DOCKER_INSTALLED`
- Minikube Conditional Check at STEP 0 — optional verification when `deploy_targets` includes
  Kubernetes. User choice persisted in `VAR_MINIKUBE_APPROVED` across sessions. Agent reminds
  user of declined installation at each new session

**K8s Configuration Externalization**
- `rules/kubernetes_rules.md` §10 — "Nothing Hardcoded Inside" principle: every configuration
  file, parameter, or value that may vary between environments must be externalized as ConfigMap,
  Secret, or environment variable. Classification rules, framework-specific examples (Spring
  Boot, Apache, Nginx, Node.js), default-to-ConfigMap rule
- STEP 2 — K8s Externalization Analysis: agent scans project code, classifies config items,
  presents externalization map to user for approval. Stored in `VAR_K8S_EXTERNALIZATION_MAP`
- STEP 4 — K8s manifest generation follows approved externalization map
- STEP 5 — Non-regression check verifies externalization map compliance
- STEP 7 — Kubernetes deploy gated on `VAR_MINIKUBE_APPROVED`

**Docker Rules**
- `rules/docker_rules.md` §0 — Docker Prerequisite Verification section with OS-specific
  auto-installation commands, daemon health checks, and blocking behavior documentation

**Session Variables — New**
- `VAR_DOCKER_INSTALLED` (Boolean) — Docker installation and daemon status
- `VAR_MINIKUBE_INSTALLED` (Boolean) — Minikube installation status
- `VAR_MINIKUBE_APPROVED` (Boolean) — User authorization for Minikube (persists across sessions)
- `VAR_K8S_EXTERNALIZATION_MAP` (Object) — Approved K8s externalization map

**Golden Rules**
- Added "Nothing Hardcoded Inside" rule to §5

### Changed

- STEP 0 — Added Docker prerequisite check (blocking) and Minikube conditional check before
  rules loading. Renumbered existing steps 2-7 → 4-9
- STEP 2 — Added K8s externalization analysis with user approval gate
- STEP 4 — Added K8s manifest generation from externalization map
- STEP 7 — Added K8s deploy step gated on `VAR_MINIKUBE_APPROVED`. Renumbered Git Release step
- `rules/docker_rules.md` — Renumbered existing sections 1-9 → 2-10 to accommodate new §0
- `rules/kubernetes_rules.md` §1 — Added Minikube prerequisite check subsection
- `GEMINI.md` — Updated scope limitation to allow Docker/Minikube installation when authorized

---

## [0.3.0] — 2026-03-28

### Added

**Rules Library — New Files**
- `rules/git_rules.md` — Branching model, Conventional Commits, SemVer algorithm, tagging, PR
  standards, repository hygiene. Conditionally loaded when `version_control.enabled = true`.
  Includes fallback for repos without a `develop` branch; `develop` created only on explicit
  user request
- `rules/testing_rules.md` — Mandatory test coverage by service type (Web/API, Backend,
  Database), test playbook/results artifacts, TDD Iron Law (RED-GREEN-REFACTOR), Verification
  Before Completion gate
- `rules/debugging_rules.md` — Systematic Debugging Protocol: 4-phase Investigation Protocol
  (Evidence → Root Cause → Targeted Fix → Post-Fix Verification), Three-Strike Rule,
  Anti-Pattern Detection table
- `rules/design_review_rules.md` — Design Review Protocol: mandatory context gathering,
  approach proposal (2–3 alternatives with trade-offs), No Placeholders Rule, self-review
  checklist, user approval gate before STEP 3

**Core Framework Enhancements (`agent.md`)**
- §4.1 Step Transition Rules — Forward-only principle with documented allowed backward
  transitions, Retry with Extended Scope (STEP 6 → STEP 3 → STEP 4), incremental backup
  support (`backup_manifest_retry_N.json`), prohibited transitions
- STEP TRACKING RULE — Global mandate: `VAR_SESSION_STEP` must be updated as the first action
  of every step (0–7)
- STEP 3 — Pre-Backup Inventory: mandatory `backup_manifest.json` with SHA-256 hashes, public
  methods, Docker volumes, K8s resources, exposed ports, env vars. Backup completeness gate
  before STEP 4. `VAR_ACTIVE_BACKUP_PATH` session variable
- STEP 4 — TDD mandatory: RED-GREEN-REFACTOR per function, failing test written first, test +
  implementation committed together. Git branching (`feature/` or `fix/`) when `VAR_GIT_ENABLED`
- STEP 5.0 — Non-Regression Check: diff current `output/` vs `backup_manifest.json`, FAIL on
  missing files/removed methods/removed volumes/PVCs/endpoints, WARN on env vars. Test-count
  guard with `VAR_TEST_BASELINE` / `VAR_TEST_COUNT`
- STEP 5.4 — Functional Tests sub-step
- STEP 6 — Retry (Extended) option, references `rules/debugging_rules.md` Investigation Protocol
- STEP 7 — Git Release: merge via PR when CI/CD configured, SemVer bump, annotated tag (gated
  on `version_control.auto_tag`), `VAR_CURRENT_VERSION` update
- §5 Golden Rules — Added `Test-First` and `Evidence Over Claims`

**Configuration**
- `config.json` — New `version_control` block (`enabled`, `provider`, `repository`,
  `default_branch`, `auto_tag`, `versioning`) with `_required_when_enabled` documentation

**Session Variables — New**
- `VAR_GIT_ENABLED` (Boolean) — Git integration active flag
- `VAR_CURRENT_VERSION` (String) — SemVer current version
- `VAR_REGRESSION_CHECK` (String) — `PASS`/`FAIL`/`WARN` from STEP 5.0
- `VAR_TEST_BASELINE` (Boolean) — Test baseline loaded flag
- `VAR_TEST_COUNT` (Integer) — Number of tests in baseline
- `VAR_ACTIVE_BACKUP_PATH` (String) — Path to active backup folder

### Changed

- STEP 0 — Rules loading now excludes `git_rules.md` unconditionally; git rules loaded only
  when `version_control.enabled = true`. Added conditional test baseline loading
- STEP 2 — References `rules/design_review_rules.md` quality standards
- STEP 5 — Renumbered sub-steps (5.0–5.6) to accommodate non-regression check and functional
  tests
- STEP 6 — Added Retry (Extended) option alongside existing Retry/Rollback/Abort
- `GEMINI.md` — Bootstrap reading list updated with all new rule files and conditional
  `git_rules.md`

---

## [0.2.0] — 2024-01-15

### Added — Phase 1 Foundation

**Core Framework**
- `agent.md` — Complete 7-step operational workflow for the Agnostic Orchestrator Agent
- STEP 5: Validation & Test (build → run → health check → PASS/FAIL)
- STEP 6: Rollback Gate (notify → retry/rollback/abort → wait for user)
- STEP 7: Consolidation (renumbered from old STEP 5)
- New session state variables: `VAR_AVAILABLE_MCP`, `VAR_VALIDATION_RESULT`, `VAR_RETRY_COUNT`

**Rules Library**
- `rules/development_rules.md` — Code quality, file size limits (differentiated by type), naming conventions, modularity
- `rules/docker_rules.md` — Container-first mandate, dev container pattern, multi-stage Dockerfiles, Compose best practices
- `rules/kubernetes_rules.md` — Minikube-first, 1 resource per YAML, Kustomize, health probes
- `rules/security_rules.md` — No hardcoded secrets, non-root users, minimal images, .gitignore/.dockerignore enforcement
- `rules/error_handling_rules.md` — Error categories, auto-fix vs ask policy, retry strategy (max 3), rollback procedure
- `rules/mcp_rules.md` — MCP inventory at bootstrap, tool preference order, missing MCP policy

**Templates**
- `templates/Dockerfile.dev` — Development container template
- `templates/Dockerfile.app` — Multi-stage production Dockerfile template
- `templates/docker-compose.yml` — Production Compose template with health checks and resource limits
- `templates/docker-compose.dev.yml` — Dev override with volume mounts and debug ports
- `templates/.env.example` — Environment variables template
- `templates/k8s/namespace.yaml` — Kubernetes namespace template
- `templates/k8s/deployment.yaml` — Deployment with probes and security context
- `templates/k8s/service.yaml` — Service template
- `templates/k8s/configmap.yaml` — ConfigMap template
- `templates/k8s/kustomization.yaml` — Kustomize base configuration

**Project Structure**
- `config.json` — Project configuration template
- `session/session_state.json.example` — Session state template with all variables
- `output/deployed_state.json.example` — Deployed state template
- `specs/active/`, `specs/history/`, `session/`, `output/`, `backups/` directories
- `GEMINI.md` — Gemini CLI auto-loaded instructions
- `CONTRIBUTING.md` — Contributing guidelines
- `.gitignore` — Appropriate ignores for AAOF projects
- `docs/guides/gemini-cli-setup.md` — Step-by-step Gemini CLI integration guide

**Documentation**
- `README.md` — Complete rewrite with project structure, workflow, platforms, prerequisites, quick start, and roadmap

### Changed

- Renamed `Agent.md` → `agent.md` (lowercase convention)
- Updated line limit rule: differentiated by file type (200 for source, 300 for docs, logical separation for infra)
- `VAR_SESSION_STEP` range extended from 0-5 to 0-7

---

## [0.1.0] — 2024-01-01

### Added — Initial Commit

- `Agent.md` — Initial operational manual with 5-step workflow
- `README.md` — Initial project description
