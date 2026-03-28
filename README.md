# Agnostic Agent Orchestrator Framework (AAOF)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-0.3.0-blue.svg)](changelog.md)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

> **Give any AI agent a structured operating system — and watch it build production-ready
> infrastructure for you.**

---

## 🎯 What Is AAOF?

AAOF is a **document-based framework** that turns a general-purpose AI agent (Claude,
Gemini, GPT, etc.) into a disciplined, stateful infrastructure engineer.

Instead of typing freeform prompts and hoping for the best, you configure a handful of
files and let the framework guide the AI through a repeatable, auditable workflow:
plan → backup → build → validate → deploy.

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

---

## 🏗 Framework Structure

```
AAOF/
├── agent.md                        ← The AI's operational manual (read this first)
├── config.json                     ← Your project requirements
├── GEMINI.md                       ← Gemini CLI auto-loaded instructions
├── changelog.md                    ← Chronological activity log
├── CONTRIBUTING.md                 ← How to contribute
│
├── rules/                          ← The AI's "libraries" — operational policies
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
│   └── session_state.json          ← Live session state (git-ignored, auto-managed)
│
├── output/                         ← Generated code, Dockerfiles, manifests
│   └── deployed_state.json         ← Technical snapshot of the last deployment
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

## 🔄 How It Works

The AI agent follows a 7-step workflow defined in `agent.md`:

```
STEP 0  Bootstrap       Read rules/ (conditional git_rules), load test baseline, inventory MCP
STEP 1  Resolution      Resolve version conflicts, confirm technical choices
STEP 2  Plan            Design review gate → present execution plan → wait for user GO
STEP 3  Backup          Pre-backup inventory (backup_manifest.json) → snapshot output/
STEP 4  Implement       TDD mandatory (RED-GREEN-REFACTOR) → generate code following rules/
STEP 5  Validate        Non-regression check → Build → Run → Health check → Functional tests
STEP 6  Rollback Gate   On FAIL: debugging protocol → Retry / Retry Extended / Rollback / Abort
STEP 7  Consolidate     Git release (SemVer tag) → update state → archive specs → changelog
```

The session state (`session/session_state.json`) persists across AI context windows,
so you can pause and resume without losing progress.

---

## 🐳 Supported Platforms

| Platform | Status | Notes |
| :--- | :--- | :--- |
| Docker Compose | ✅ Supported | Primary local development and deployment target |
| Minikube (K8s) | ✅ Supported | Local Kubernetes testing via Minikube |
| Cloud K8s (EKS/GKE/AKS) | 🗺 Roadmap | Planned for Phase 2 |
| Nomad / Podman | 🗺 Roadmap | Planned for Phase 3 |

---

## 📋 Prerequisites

Before using AAOF, ensure you have:

- **Docker** and **Docker Compose** installed ([Get Docker](https://docs.docker.com/get-docker/))
- **Minikube** (for Kubernetes targets) ([Install Minikube](https://minikube.sigs.k8s.io/docs/start/))
- An **AI agent** that can read files and run terminal commands:
  - [Google Gemini CLI](docs/guides/gemini-cli-setup.md) ← recommended quickstart
  - [Anthropic Claude Code](https://docs.anthropic.com/claude-code)
  - Any other MCP-compatible agent
- **Git** (to clone this repo and track changes)

---

## 🚀 Quick Start

### 1. Clone the repository

```bash
git clone https://github.com/ssignori76/Agnostic-Agent-Orchestrator-Framework.git my-project
cd my-project
```

### 2. Configure your project

Edit `config.json` to describe your stack and deployment targets:

```json
{
  "project_name": "my-app",
  "project_description": "A Node.js REST API with PostgreSQL",
  "stack": {
    "languages": ["javascript"],
    "frameworks": ["express"],
    "databases": ["postgresql"]
  },
  "deploy_targets": ["DOCKER"]
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

The agent will present an execution plan at STEP 2. Review it, type **GO**, and watch
your project get built inside Docker containers.

---

## 🗺 Roadmap

### Phase 1 — Foundation (current: v0.3.0)
- [x] Core workflow (agent.md) with 7-step process
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
