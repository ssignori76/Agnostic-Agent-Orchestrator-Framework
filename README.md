# Agnostic Agent Orchestrator Framework (AAOF)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-0.5.0-blue.svg)](changelog.md)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

> **Give any AI agent a structured operating system — and watch it build production-ready
> infrastructure for you.**

---

## 🎯 What Is AAOF?

AAOF is a **document-based framework** that turns a general-purpose AI agent (Claude,
Gemini, GPT, etc.) into a disciplined, stateful infrastructure engineer.

Instead of typing freeform prompts and hoping for the best, you configure a handful of
files and let the framework guide the AI through a repeatable, auditable workflow:
bootstrap → plan → backup → implement → validate → consolidate.

AAOF enforces this workflow through a **Step Gate Machine**: every step has programmatic
entry conditions (preconditions) and exit conditions (postconditions). The agent cannot
skip steps, cut corners, or advance without meeting all conditions — skipping from
STEP 0 to STEP 4 is as impossible as jumping from floor 0 to floor 4 in an elevator.

**You don't need to be a programmer.** You just need to describe what you want.

---

## 👤 Who Is It For?

- Non-programmers who use AI tools to build software projects
- Developers who want a reproducible AI-assisted workflow
- Teams that need auditable, rollback-safe AI-generated code
- Anyone deploying containerized applications with Docker or Kubernetes

---

## ✨ Key Features

| Feature | Description |
| :--- | :--- |
| **Tech-Agnostic** | Works with any stack — Node.js, Python, Java, Go, PHP… |
| **Stateful Sessions** | JSON-based persistence survives AI context resets |
| **Container-First** | All development happens inside Docker containers |
| **Automatic Backups** | Every change is snapshotted before modification |
| **Step Gate Machine** | Programmatic preconditions and postconditions block every step — no skipping allowed |
| **Security Profiles** | Three tiered profiles (lab/staging/production) adapt rule enforcement to the environment |
| **Compliance Checklist** | Automated machine-verifiable checks at STEP 5.0.5 produce a compliance report |
| **Validation Gate** | Build → Run → Health check before accepting a result |
| **Rollback Gate** | Structured recovery on failure (retry / rollback / abort) |
| **MCP-Aware** | Works with any MCP-enabled agent (Claude Code, Gemini CLI…) |
| **Changelog Enforced** | Every action is logged in `changelog.md` |
| **TDD Iron Law** | RED-GREEN-REFACTOR mandatory cycle — no production code without a failing test first |
| **Non-Regression Gate** | Pre-build diff against backup manifest prevents accidental removal of files, methods, volumes |
| **Git/GitHub Integration** | Optional branching, Conventional Commits, SemVer auto-tagging, PR-based merge |
| **Design Review Gate** | Mandatory context gathering and approach proposal before execution planning |
| **Debugging Protocol** | Systematic 4-phase investigation before any fix attempt |
| **Step Transition Rules** | Documented forward-only principle with safe backward transitions and incremental backups |
| **Auto-Prerequisites** | Docker auto-detected and installed; Minikube optional with persistent user choice |
| **Config Externalization** | K8s configs, secrets, and env vars automatically mapped and externalized — nothing hardcoded inside containers |
| **Requirements Checklist** | Automatic contract check between execution plan and produced artifacts (`rules/requirements_checklist.md`) |
| **Requirements Traceability Matrix** | Explicit mapping of requirements → tests with validation in the TDD gate check |
| **Mandatory Artifacts Validation** | Verifies artifact existence, non-emptiness, minimum length, and required sections for each step |
| **Namespace Separation** | Clear separation between framework artifacts (`.aaof/`) and project artifacts (`output/`) |
| **Template Scaffolding** | Automatic directory structure generation with empty-directory validation |
| **Refinement Loop** | Optional gap analysis step between requirements and produced artifacts, with targeted iteration |
| **Auto-Scoring System** | Automatic execution quality score with detailed report |
| **Multi-Session Orchestration** | Orchestrator agent support with turn limit recovery |

---

## 🏗 Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│  User                                                               │
│   │                                                                 │
│   ├─ Edits config.json  ──────────────────────────────────────┐     │
│   ├─ Writes specs/active/  ──────────────────────────────┐    │     │
│   └─ Types "GO" to approve plans                         │    │     │
│                                                          │    │     │
│  AI Agent (Claude / Gemini / GPT)                        │    │     │
│   │                                                      │    │     │
│   ├─ Reads agent.md  (primary manual)                    │    │     │
│   ├─ Reads rules/*.md (policies)                         │    │     │
│   │   ├─ workflow_gates.md    Step Gate Machine          │    │     │
│   │   ├─ security_profiles.md env-aware enforcement      │    │     │
│   │   ├─ output_checklist.json compliance checks         │    │     │
│   │   └─ (other rules...)                                │    │     │
│   ├─ Reads specs ────────────────────────────────────────┘    │     │
│   ├─ Reads config ────────────────────────────────────────────┘     │
│   │                                                                 │
│   ├─ Reads/Writes session/session_state.json  (persistence layer)   │
│   ├─ Reads/Writes session/step_evidence.json  (gate audit log)      │
│   └─ Writes output/  (generated code and deployment files)          │
│                                                                     │
│  Step Gate Machine                                                   │
│   ├─ RULES: rules/workflow_gates.md  (pre/postconditions per step)  │
│   ├─ PROTOCOL: agent.md §4.2        (how the agent enforces gates)  │
│   ├─ Every step has PRECONDITIONS — entry is blocked if not met     │
│   ├─ Every step has POSTCONDITIONS — advance is blocked if not met  │
│   └─ All gate checks are logged in session/step_evidence.json       │
└─────────────────────────────────────────────────────────────────────┘
```

> **How does the Step Gate Machine work?** The gate conditions for each step (PRE-0.1, POST-0.1, etc.) are defined in [`rules/workflow_gates.md`](rules/workflow_gates.md). The enforcement protocol — *how* the agent checks those conditions at every transition — is defined in [`agent.md` §4.2](agent.md). To add or modify gate conditions, edit `workflow_gates.md`. See [Customizing the Rules](#-customizing-the-rules) below.

---

## 🏗 Framework Structure

```
AAOF/
├── agent.md                        ← The AI's operational manual (read this first)
├── config.json                     ← Your project requirements (incl. environment_context)
├── GEMINI.md                       ← Gemini CLI auto-loaded instructions
├── AGENT_ORCHESTRATOR.md           ← Guide for AI orchestrator agents (multi-session)
├── changelog.md                    ← Chronological activity log
├── CONTRIBUTING.md                 ← How to contribute
│
├── rules/                          ← The AI's "libraries" — operational policies
│   ├── workflow_gates.md           ← Step Gate Machine: pre/postconditions per step
│   ├── security_profiles.md        ← lab/staging/production enforcement rules
│   ├── output_checklist.json       ← Machine-verifiable compliance checks (STEP 5.0.5)
│   ├── requirements_checklist.md   ← Contract check mechanism (STEP 4→5 gate)
│   ├── development_rules.md        ← Code quality, file size, naming conventions
│   ├── docker_rules.md             ← Container-first dev, Compose best practices
│   ├── kubernetes_rules.md         ← Minikube-first, 1 resource per file, Kustomize
│   ├── security_rules.md           ← No secrets in code, non-root users, scanning
│   ├── error_handling_rules.md     ← Retry strategy, rollback procedures, logging
│   ├── mcp_rules.md                ← MCP server discovery and usage conventions
│   ├── git_rules.md                ← Branching model, Conventional Commits, SemVer
│   ├── testing_rules.md            ← Test coverage, TDD Iron Law, verification gate
│   ├── debugging_rules.md          ← Systematic debugging protocol, investigation phases
│   └── design_review_rules.md      ← Design review gate, approach proposals, approval
│
├── specs/
│   ├── active/                     ← Requirements the AI should implement NOW
│   └── history/                    ← Completed specs (archived automatically)
│
├── session/
│   ├── session_state.json          ← Live session state (git-ignored, auto-managed)
│   ├── step_evidence.json          ← Gate check audit log (git-ignored, auto-managed)
│   └── requirements_checklist.json ← Per-step requirements tracking (git-ignored, auto-managed)
│
├── output/                         ← Generated code, Dockerfiles, manifests
│   ├── deployed_state.json         ← Technical snapshot of the last deployment
│   ├── test_playbook.md            ← Test plan (generated at STEP 5)
│   ├── test_results.md             ← Test execution results (generated at STEP 5)
│   ├── compliance_report.md        ← Compliance check results (generated at STEP 5.0.5)
│   └── scoring_report.md           ← Auto-scoring quality report (generated at STEP 7)
│
├── .aaof/                          ← Framework orchestrator artifacts (git-ignored)
│   └── orchestrator_state.json     ← Orchestrator state for turn limit recovery
│
├── prompts/                        ← Ready-to-use prompts for orchestrator patterns
│   └── resume_prompt.md            ← Resume instructions for relaunched sub-agents
│
├── backups/                        ← Pre-change snapshots (git-ignored)
│
├── templates/                      ← Reusable file templates
│   ├── Dockerfile.dev              ← Dev container template
│   ├── Dockerfile.app              ← Multi-stage production Dockerfile
│   ├── docker-compose.yml          ← Compose template with best practices
│   ├── docker-compose.dev.yml      ← Dev override (volume mounts, debug ports)
│   ├── .env.example                ← Environment variables template
│   └── k8s/
│       ├── namespace.yaml
│       ├── deployment.yaml
│       ├── service.yaml
│       ├── configmap.yaml
│       └── kustomization.yaml
│
└── docs/
    └── guides/
        └── gemini-cli-setup.md     ← Step-by-step Gemini CLI integration guide
```

---

## 🔄 The 8-Step Workflow (with Gate Machine)

The AI agent follows a strictly gated 8-step workflow. Every transition between steps
requires meeting both the **exit conditions** of the current step and the **entry conditions**
of the next. The agent will stop and inform you if any gate fails.

### STEP 0 — Bootstrap & Rules Loading

| | |
|---|---|
| **What happens** | Docker check; Minikube check (if K8s target); load all `rules/*.md`; load test baseline; inventory MCP servers; load `config.json`; initialize or restore `session_state.json` |
| **Entry gate** | None (entry point) |
| **Exit gate** | `VAR_DOCKER_INSTALLED` == true; all rules loaded; `session_state.json` initialized; `VAR_SESSION_STEP` == 0 |
| **Gate failure** | If Docker is not installed and user refuses installation: **full stop** — Docker is mandatory |

### STEP 1 — Priority Resolution and Versioning

| | |
|---|---|
| **What happens** | Resolve version conflicts (deployed state vs config); confirm tech versions; **classify environment** (lab/staging/production); load security profile; confirm profile with user |
| **Entry gate** | `VAR_SESSION_STEP` == 0; `VAR_DOCKER_INSTALLED` == true; `session_state.json` exists |
| **Exit gate** | `VAR_SOURCE_OF_TRUTH` set; `VAR_CONFIRMED_VERSIONS` non-empty; `VAR_SECURITY_PROFILE` set and confirmed by user; `config.json` reflects actual stack |
| **Gate failure** | If environment type is unknown: agent **asks the user** and cannot proceed without a confirmed profile |

### STEP 2 — Execution Plan (Proposal)

| | |
|---|---|
| **What happens** | Present architectural alternatives (≥2 options with trade-offs); K8s externalization analysis if applicable; wait for user "GO" |
| **Entry gate** | `VAR_SESSION_STEP` == 1; `VAR_CONFIRMED_VERSIONS` non-empty; `specs/active/` contains ≥1 file |
| **Exit gate** | Plan presented with ≥2 alternatives; user explicit "GO" received; `VAR_SESSION_STEP` == 2 |
| **Gate failure** | Agent will not start implementation without an explicit "GO" from the user |

### STEP 3 — Backup Protocol

| | |
|---|---|
| **What happens** | Create timestamped backup in `backups/`; generate `backup_manifest.json` with file hashes, methods, volumes, ports, env vars; verify backup completeness |
| **Entry gate** | `VAR_SESSION_STEP` == 2; user "GO" was received at STEP 2 |
| **Exit gate** | `VAR_ACTIVE_BACKUP_PATH` set; `backup_manifest.json` exists; backup completeness verified |
| **Gate failure** | Agent will not touch `output/` until backup is confirmed complete |

### STEP 4 — Implementation

| | |
|---|---|
| **What happens** | Write code inside Docker containers following all `rules/`; TDD mandatory (RED-GREEN-REFACTOR); Git branch if enabled; K8s externalization if applicable; generate `session/requirements_checklist.json`; run **Contract Check** before advancing |
| **Entry gate** | `VAR_SESSION_STEP` == 3; `VAR_ACTIVE_BACKUP_PATH` set; `backup_manifest.json` exists |
| **Exit gate** | Source files exist in `output/`; TDD evidence recorded (or user-approved exception); all code follows `rules/development_rules.md`; `VAR_CONTRACT_CHECK` == PASS; all requirements in `requirements_checklist.json` satisfied |
| **Gate failure** | No production code without a failing test first — TDD is the Iron Law; `VAR_CONTRACT_CHECK` == FAIL blocks transition to STEP 5 |

### STEP 5 — Validation & Test

| | |
|---|---|
| **What happens** | **5.0** Non-regression check; **5.0.5** Compliance check (`output_checklist.json`); **5.1** Build; **5.2** Run; **5.3** Health check; **5.4** Functional tests; **5.5** Set `VAR_VALIDATION_RESULT` |
| **Entry gate** | `VAR_SESSION_STEP` == 4; source files in `output/`; `backup_manifest.json` available |
| **Exit gate** | `VAR_VALIDATION_RESULT` set (PASS/FAIL); `test_playbook.md`, `test_results.md`, `compliance_report.md` all exist; `VAR_REGRESSION_CHECK` set |
| **Gate failure** | Any FAIL in compliance checks → agent stops and presents failures; any test failure → go to STEP 6 |

### STEP 6 — Rollback Gate *(entered only when STEP 5 fails)*

| | |
|---|---|
| **What happens** | Debug investigation; present user options: Retry / Retry Extended / Rollback / Abort |
| **Entry gate** | `VAR_VALIDATION_RESULT` == FAIL |
| **Exit gate** | User decision recorded in `session_state.json` |
| **Gate failure** | STEP 6 cannot be entered if validation passed — it is a FAIL-only path |

### STEP 7 — Consolidation, State Backup, and Cleanup

| | |
|---|---|
| **What happens** | Update `deployed_state.json`; session backup; archive `specs/active/`; cleanup `session/`; update `changelog.md`; K8s deploy if applicable; Git release tag if enabled |
| **Entry gate** | `VAR_VALIDATION_RESULT` == PASS; `VAR_REGRESSION_CHECK` != FAIL |
| **Exit gate** | `deployed_state.json` updated; `specs/active/` archived; `changelog.md` updated |
| **Gate failure** | Agent will not consolidate a failed session |

---

## 🔐 Security Profiles

Security enforcement adapts to the deployment environment. The active profile is set in
`config.json` under `environment_context.type` and stored in `VAR_SECURITY_PROFILE`.

### Profile Selection

```json
"environment_context": {
  "type": "lab",
  "tls_required": false,
  "external_access": false,
  "secrets_strictness": "relaxed"
}
```

Valid values: `lab`, `staging`, `production`.

If not set, the agent **must ask the user** before proceeding past STEP 1.

### Enforcement Table

| Rule | Lab/Dev | Staging | Production |
|------|---------|---------|------------|
| TLS/HTTPS | ⬜ Optional (document choice) | ✅ Required (self-signed OK) | ✅ Required (CA-signed) |
| Secrets in docker-compose.yml | ⚠️ WARN | ❌ FAIL | ❌ FAIL |
| DB port exposed on host | ✅ Allowed for debugging | ❌ FAIL | ❌ FAIL |
| Password hashing (application) | ⚠️ WARN | ✅ Required | ✅ Required |
| Non-root containers | ✅ Required always | ✅ Required | ✅ Required |
| `.env.example` present | ✅ Required always | ✅ Required | ✅ Required |
| `.dockerignore` present | ✅ Required always | ✅ Required | ✅ Required |
| Network isolation (Compose) | ⚠️ WARN | ✅ Required | ✅ Required |
| Resource limits (Compose) | ⬜ Optional | ✅ Required | ✅ Required |
| Image scanning | ⬜ Recommended | ✅ Required | ✅ Required + block on CRITICAL |
| Cookie security (HttpOnly, SameSite) | ⚠️ WARN | ✅ Required | ✅ Required |
| Input validation | ⚠️ WARN | ✅ Required | ✅ Required |
| File headers (development_rules §2) | ⬜ Optional | ✅ Required | ✅ Required |
| Read-only filesystem | ⬜ Optional | ⚠️ WARN | ✅ Required |
| XSS prevention (no raw innerHTML) | ⚠️ WARN | ✅ Required | ✅ Required |

**Legend:** ✅ Required (FAIL if violated) | ⚠️ WARN (logged, execution may continue) | ⬜ Optional | ❌ Always FAIL

> **Non-root containers are REQUIRED for ALL profiles** — there is no lab exemption.

See `rules/security_profiles.md` for the complete reference.

---

## ✅ Compliance Checklist (STEP 5.0.5)

At STEP 5.0.5, after non-regression and before build, the agent automatically runs all
checks defined in `rules/output_checklist.json` and generates `output/compliance_report.md`.

Severity is determined by `VAR_SECURITY_PROFILE` — the same check may be WARN in lab
and FAIL in staging/production.

| ID | Category | Rule | Lab | Staging | Production |
|----|----------|------|-----|---------|------------|
| SEC-001 | Security | No hardcoded secrets in docker-compose | WARN | FAIL | FAIL |
| SEC-002 | Security | All Dockerfiles define non-root USER | FAIL | FAIL | FAIL |
| SEC-003 | Security | `.env.example` exists | FAIL | FAIL | FAIL |
| SEC-004 | Security | Database port not exposed on host | WARN | FAIL | FAIL |
| SEC-005 | Security | No XSS via `innerHTML` with untrusted data | WARN | FAIL | FAIL |
| DOCKER-001 | Docker | Network isolation defined in compose (≥2 networks) | WARN | FAIL | FAIL |
| DOCKER-002 | Docker | `.dockerignore` present for each build context | FAIL | FAIL | FAIL |
| DOCKER-003 | Docker | `docker-compose.dev.yml` exists | FAIL | FAIL | FAIL |
| DEV-001 | Development | All source files have header block | WARN | FAIL | FAIL |
| DEV-002 | Development | Source files within 200-line limit | WARN | WARN | FAIL |
| TEST-001 | Testing | Minimum test count met | WARN | FAIL | FAIL |
| TEST-002 | Testing | Negative tests present | WARN | FAIL | FAIL |

If ANY check returns `FAIL` → the agent stops, presents the failures, and waits for user approval.
If checks return only `WARN` → warnings are logged and execution continues.

---

## 🔄 Quick Workflow Summary

```
STEP 0  Bootstrap       Docker check → read rules/ → load test baseline → init session state
STEP 1  Resolution      Resolve versions → classify environment → load security profile → user confirm
STEP 2  Plan            Design review gate → ≥2 alternatives → wait for user GO
STEP 3  Backup          backup_manifest.json → snapshot output/ → verify completeness
STEP 4  Implement       TDD mandatory (RED-GREEN-REFACTOR) → generate code → Contract Check (VAR_CONTRACT_CHECK == PASS)
STEP 5  Validate        Non-regression → Compliance check (5.0.5) → Build → Run → Tests → compliance_report.md
        [Optional]      Refinement Loop — gap analysis between requirements and artifacts, iterate if needed
STEP 6  Rollback Gate   On FAIL: debug protocol → Retry / Retry Extended / Rollback / Abort
STEP 7  Consolidate     Update deployed_state.json → archive specs → changelog → scoring_report.md → Git release
```

The session state (`session/session_state.json`) persists across AI context windows,
so you can pause and resume without losing progress.

---

## 📊 Session State Reference

All session variables are stored in `session/session_state.json` and read/written
exclusively through that file (never from chat history alone).

| Variable | Type | Description |
|:---------|:-----|:------------|
| `VAR_SESSION_STEP` | Integer | Current workflow step (0–7) |
| `VAR_SOURCE_OF_TRUTH` | String | `CONFIG_JSON` or `DEPLOYED_STATE_JSON` |
| `VAR_CONFIRMED_VERSIONS` | Object | Key-value map of confirmed tech versions, e.g. `{"node": "20"}` |
| `VAR_DEPLOY_TARGET` | Array | Deploy targets, e.g. `["DOCKER", "K8S"]` |
| `VAR_PROJECT_MODE` | String | `SCRATCH` or `INTEGRATION` |
| `VAR_AVAILABLE_MCP` | Array | MCP servers discovered at bootstrap |
| `VAR_VALIDATION_RESULT` | String | `PASS` or `FAIL` (set at STEP 5, based on executed evidence) |
| `VAR_RETRY_COUNT` | Integer | Current retry count for STEP 4 (max 3) |
| `VAR_GIT_ENABLED` | Boolean | `true` if `version_control.enabled` is `true` in config |
| `VAR_CURRENT_VERSION` | String | Current project version in SemVer format |
| `VAR_REGRESSION_CHECK` | String | `PASS`, `FAIL`, or `WARN` (set at STEP 5.0) |
| `VAR_ACTIVE_BACKUP_PATH` | String | Path to the current backup folder |
| `VAR_TEST_BASELINE` | Boolean | `true` if `output/test_playbook.md` was loaded at bootstrap |
| `VAR_TEST_COUNT` | Integer | Number of tests in the loaded baseline (0 if none) |
| `VAR_DOCKER_INSTALLED` | Boolean | `true` if Docker and Docker Compose are installed and functional |
| `VAR_MINIKUBE_INSTALLED` | Boolean | `true` if Minikube is installed and functional |
| `VAR_MINIKUBE_APPROVED` | Boolean | `true` if the user has authorized Minikube installation/usage |
| `VAR_K8S_EXTERNALIZATION_MAP` | Object | Map of externalized configs (ConfigMaps, Secrets, env vars) |
| `VAR_SECURITY_PROFILE` | String | Active security profile: `lab`, `staging`, or `production` |
| `VAR_CONTRACT_CHECK` | String | `PASS` or `FAIL` from the contract check at STEP 4 (all requirements satisfied) |

---

## 📋 Prerequisites

Before using AAOF, ensure you have:

- **Docker** and **Docker Compose** — the agent will verify and guide installation at bootstrap if missing (mandatory)
- **Minikube** (for Kubernetes targets) — the agent will verify and offer installation if needed (optional, user choice persisted)
- An **AI agent** that can read files and run terminal commands:
  - [Google Gemini CLI](docs/guides/gemini-cli-setup.md) ← recommended quickstart
  - [Anthropic Claude Code](https://docs.anthropic.com/claude-code)
  - Any other MCP-compatible agent
- **Git** (to clone this repo and track changes)

---

## 🐳 Supported Platforms

| Platform | Status | Notes |
| :--- | :--- | :--- |
| Docker Compose | ✅ Supported | Primary local development and deployment target |
| Minikube (K8s) | ✅ Supported | Local Kubernetes testing via Minikube |
| Cloud K8s (EKS/GKE/AKS) | 🗺 Roadmap | Planned for Phase 2 |
| Nomad / Podman | 🗺 Roadmap | Planned for Phase 3 |

---

## 🚀 Quick Start

### 1. Clone the repository

```bash
git clone https://github.com/ssignori76/Agnostic-Agent-Orchestrator-Framework.git my-project
cd my-project
```

### 2. Configure your project

Edit `config.json` to describe your stack, deployment targets, and environment context:

```json
{
  "project_name": "my-app",
  "project_description": "A Node.js REST API with PostgreSQL",
  "stack": {
    "languages": ["javascript"],
    "frameworks": ["express"],
    "databases": ["postgresql"]
  },
  "deploy_targets": ["DOCKER"],
  "environment_context": {
    "type": "lab",
    "tls_required": false,
    "external_access": false,
    "secrets_strictness": "relaxed"
  }
}
```

### 3. Write your first spec

Create a file in `specs/active/` describing what you want to build:

```
specs/active/001-initial-api.md
```

### 4. Launch your AI agent

Follow the **[Gemini CLI Setup Guide](docs/guides/gemini-cli-setup.md)** to configure
and launch Gemini CLI with AAOF.

For other agents, point them to `agent.md` as their primary instruction file.

### 5. Approve the plan and deploy

The agent will:
1. Bootstrap and classify your environment at STEP 1
2. Present an execution plan at STEP 2 — review it and type **GO**
3. Build and validate your project inside Docker containers
4. Produce `output/compliance_report.md` with security and quality checks

---

## 🛡 Golden Rules

The 8 golden rules that govern every agent action:

| Rule | Plain Language |
|------|----------------|
| **No Assumptions** | If a technical parameter is ambiguous, the agent asks. It never guesses. |
| **Documentation** | Every change must be traceable in `changelog.md`. |
| **Persistence** | `session_state.json` is the only source of truth for session state. Chat history is not reliable. |
| **Container-First** | All development and testing happens inside Docker containers. Nothing is installed on the host. |
| **Security** | `rules/security_rules.md` applies at all times — no exceptions. Profile adjusts severity, never waives rules. |
| **Test-First** | No production code without a failing test first — TDD is the Iron Law. |
| **Evidence Over Claims** | `VAR_VALIDATION_RESULT` = PASS only after the agent has directly observed test results from executed commands. |
| **Nothing Hardcoded Inside** | Every config that may vary between environments must be externalized as ConfigMap, Secret, or environment variable. |

---

## 🔧 Customizing the Rules

The `rules/` files are **policy documents, not code**. The AI agent loads them at STEP 0
and treats them as binding operational directives. You can edit any `rules/*.md` file to
change the agent's behavior — no code changes required.

### How to add or modify step gate conditions

Edit [`rules/workflow_gates.md`](rules/workflow_gates.md) and add rows to the
Preconditions or Postconditions tables for the relevant step. Each condition needs:
- An ID (e.g., `PRE-4.4`)
- A description of what must be true
- A "How to verify" instruction the agent can follow

### How to add a new compliance check

Add an entry to [`rules/output_checklist.json`](rules/output_checklist.json) with:
- An ID (e.g., `SEC-006`)
- A `category` and `rule` description
- A `how_to_verify` instruction
- A `severity_by_profile` object specifying `lab`, `staging`, and `production` severities

### How to change the security profile behavior

Edit [`rules/security_profiles.md`](rules/security_profiles.md) to change what is WARN
vs FAIL for each environment tier. To switch your project's active profile, update
`config.json` → `environment_context.type` to `lab`, `staging`, or `production`.

### How to create a completely new rule file

Create a new file in the `rules/` directory (e.g., `rules/my_custom_rules.md`). The agent
automatically loads **all `rules/*.md` files at STEP 0** (see `agent.md` STEP 0, point 4).
Use clear imperative language ("The agent MUST…") so the AI treats it as a binding directive.

### What NOT to modify (or modify with caution)

[`agent.md`](agent.md) is the **core operational protocol** — changing it alters the
fundamental 8-step workflow. Modify it only if you fully understand the step lifecycle and
gate validation protocol (§4.2). The `session/` and `backups/` directories are
auto-managed and should not be edited manually.

### Quick Reference

| Goal | File to edit | Notes |
|------|-------------|-------|
| Add/change step gate conditions | `rules/workflow_gates.md` | Add rows to PRE/POST tables |
| Add a compliance check | `rules/output_checklist.json` | Include `severity_by_profile` |
| Adjust security enforcement | `rules/security_profiles.md` | Changes WARN/FAIL thresholds |
| Add technology-specific rules | `rules/new_file.md` | Auto-loaded at STEP 0 |
| Change deployment environment | `config.json` | Set `environment_context.type` |
| Modify core workflow | `agent.md` | ⚠️ Advanced — affects all steps |

---

## 🤖 Using AAOF with an Orchestrator Agent

AAOF supports a **multi-session orchestration pattern** where a high-level AI agent (the
*orchestrator*) manages the project lifecycle and delegates the actual implementation work
to *sub-agents*. This pattern solves the **turn limit problem**: each sub-agent has a fresh
budget of turns, so complex projects that would exceed a single session's limits are handled
automatically.

### How it works

```
Orchestrator Agent (Agent A)
  │
  │  1. Configures config.json and specs/
  │
  ├─► Sub-Agent 1 ── Implements until turn limit
  │       └─ Writes session/session_state.json
  │
  ├─► Sub-Agent 2 ── Reads state, continues from VAR_SESSION_STEP
  │       └─ Updates session/session_state.json
  │
  └─► Sub-Agent N ── Finishes at STEP 7, produces scoring_report.md
```

The orchestrator monitors `session/session_state.json` after each sub-agent run. If
`VAR_SESSION_STEP` < 7, the sub-agent did not finish and the orchestrator relaunches it
with the resume prompt. Up to 5 relaunch attempts are supported before the orchestrator
notifies the user.

### Ready-to-use prompts

**Prompt 1 — If you have NOT yet cloned the repo (Italian):**

```
Scarica il repository https://github.com/ssignori76/Agnostic-Agent-Orchestrator-Framework.git
e leggi il file AGENT_ORCHESTRATOR.md. Segui le istruzioni contenute nel file.
```

**Prompt 1 — If you have NOT yet cloned the repo (English):**

```
Download the repository https://github.com/ssignori76/Agnostic-Agent-Orchestrator-Framework.git
and read the AGENT_ORCHESTRATOR.md file. Follow the instructions contained in the file.
```

**Prompt 2 — If you have ALREADY cloned the repo (Italian):**

```
Leggi il file AGENT_ORCHESTRATOR.md in questa directory.
Segui le istruzioni contenute nel file.
```

**Prompt 2 — If you have ALREADY cloned the repo (English):**

```
Read the AGENT_ORCHESTRATOR.md file in this directory.
Follow the instructions contained in the file.
```

See [`AGENT_ORCHESTRATOR.md`](AGENT_ORCHESTRATOR.md) for the full orchestrator guide,
including turn limit recovery, state persistence, and the complete relaunch protocol.

---

## 🗺 Roadmap

### Phase 1 — Foundation (current: v0.5.0)
- [x] Core workflow (agent.md) with 8-step process
- [x] Rules library (development, docker, k8s, security, error handling, MCP)
- [x] Validation & Rollback gates
- [x] Templates for Docker and Kubernetes
- [x] Gemini CLI integration guide
- [x] Git/GitHub integration rules with SemVer auto-tagging
- [x] Testing strategy with TDD Iron Law enforcement
- [x] Non-regression verification and backup hardening
- [x] Step transition rules with extended retry support
- [x] Design review and debugging protocols
- [x] Verification Before Completion enforcement
- [x] Docker prerequisite auto-detection and guided installation
- [x] Minikube optional check with persistent user choice
- [x] K8s configuration externalization principle ("Nothing Hardcoded Inside")
- [x] **Step Gate Machine** — programmatic preconditions/postconditions per step (`rules/workflow_gates.md`)
- [x] **Security Profiles** — lab/staging/production enforcement tiers (`rules/security_profiles.md`)
- [x] **Compliance Checklist** — automated machine-verifiable checks at STEP 5.0.5 (`rules/output_checklist.json`)
- [x] **Environment Context** in `config.json` — explicit `environment_context.type`
- [x] Requirements Checklist with Contract Check mechanism
- [x] Mandatory Artifacts Validation in gate check transitions
- [x] Requirements Traceability Matrix between requirements and tests
- [x] Namespace separation between framework and project artifacts
- [x] Template scaffolding and directory validation
- [x] Optional Refinement Loop step with gap analysis
- [x] Auto-scoring system integrated in the framework
- [x] Multi-session orchestration with Agent Orchestrator Guide

### Phase 2 — Cloud & Advanced Deployments
- [ ] Cloud Kubernetes support (EKS, GKE, AKS)
- [ ] Terraform integration
- [ ] CI/CD pipeline templates (GitHub Actions, GitLab CI)
- [ ] Multi-agent orchestration

### Phase 3 — Ecosystem Expansion
- [ ] Podman / Nomad support
- [ ] Additional AI agent guides (Claude Code, GPT-4o)
- [ ] Web UI for session state visualization
- [ ] Plugin system for custom rules

---

## 🤝 Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) before
submitting a pull request.

Key areas where contributions are most valuable:

- New `rules/` files for additional technologies
- AI agent guides for new platforms (Claude Code, GPT)
- Template improvements
- Real-world usage examples in `specs/`

---

## 📄 License

This project is licensed under the **MIT License** — see [LICENSE](LICENSE) for details.

---

*Created with 💡 to empower non-developers in managing professional AI-driven project
lifecycles.*
