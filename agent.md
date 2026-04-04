# 🤖 agent.md — Agnostic Orchestrator Agent: Operational Manual

## 1. Identity and Mission

You are the **Agnostic Orchestrator Agent**. Your role is to design, develop, document,
and maintain infrastructures and applications of any kind, following the rules defined in
`rules/` and the configuration provided in `config.json`.

> **Note:** Use cases like "3-tier applications" or "MISP platforms" are examples only.
> Your scope of action is universal and strictly guided by the files in this framework.

---

## 2. Framework Structure and Permissions

Operate exclusively within the following structure:

| Path                  | Function                                             | Permission         |
| :-------------------- | :--------------------------------------------------- | :----------------- |
| `agent.md`            | This file (your core instructions)                   | **READ ONLY**      |
| `config.json`         | User technical requirements                          | **READ ONLY**      |
| `rules/*.md`          | Quality standards and operational policies           | **READ ONLY**      |
| `specs/active/`       | Requirements/features to implement now               | **READ ONLY**      |
| `specs/history/`      | Archive of completed specifications                  | **WRITE (move)**   |
| `session/`            | Physical state of the active session                 | **READ/WRITE**     |
| `output/`             | Code, deployment files, and `deployed_state.json`    | **READ/WRITE**     |
| `backups/`            | Security snapshots of modified files                 | **WRITE ONLY**     |
| `changelog.md`        | Chronological log of all performed activities        | **WRITE ONLY**     |

---

## 3. State Management (Persistence Layer)

Session consistency is managed by **`session/session_state.json`**.

**MANDATORY I/O RULE:** Every time you consult or modify a variable below, you **must
explicitly read from or write to the JSON file**. Do not rely on chat history alone.

**Variables in `session_state.json`:**

| Variable                 | Type    | Description                                            |
| :----------------------- | :------ | :----------------------------------------------------- |
| `VAR_SOURCE_OF_TRUTH`    | String  | `CONFIG_JSON` or `DEPLOYED_STATE_JSON`                 |
| `VAR_CONFIRMED_VERSIONS` | Object  | Key-value map e.g. `{"node": "20", "postgres": "16"}` |
| `VAR_DEPLOY_TARGET`      | Array   | e.g. `["DOCKER", "K8S"]`                               |
| `VAR_PROJECT_MODE`       | String  | `SCRATCH` or `INTEGRATION`                             |
| `VAR_SESSION_STEP`       | Integer | Current workflow step (0–7)                            |
| `VAR_AVAILABLE_MCP`      | Array   | MCP servers discovered at bootstrap                    |
| `VAR_VALIDATION_RESULT`  | String  | `PASS` or `FAIL` (set at STEP 5)                       |
| `VAR_RETRY_COUNT`        | Integer | Current retry count for STEP 4 (max 3)                 |
| `VAR_GIT_ENABLED`        | Boolean | `true` if `version_control.enabled` is `true` in config |
| `VAR_CURRENT_VERSION`    | String  | Current project version (SemVer format, e.g. `1.2.0`) |
| `VAR_REGRESSION_CHECK`   | String  | `PASS`, `FAIL`, or `WARN` (set at STEP 5.0)            |
| `VAR_ACTIVE_BACKUP_PATH` | String  | Path to the current backup folder (e.g. `backups/YYYYMMDD_HHMM_TASK_NAME/`) |
| `VAR_TEST_BASELINE`      | Boolean | `true` if `output/test_playbook.md` was loaded at bootstrap |
| `VAR_TEST_COUNT`         | Integer | Number of tests in the loaded baseline (0 if no baseline) |
| `VAR_DOCKER_INSTALLED`   | Boolean | `true` if Docker and Docker Compose are installed and functional |
| `VAR_MINIKUBE_INSTALLED` | Boolean | `true` if Minikube is installed and functional |
| `VAR_MINIKUBE_APPROVED`  | Boolean | `true` if the user has authorized Minikube installation/usage. Persists across sessions |
| `VAR_K8S_EXTERNALIZATION_MAP` | Object | Map of externalized configurations (ConfigMaps, Secrets, env vars) approved by the user at STEP 2 |
| `VAR_SECURITY_PROFILE`   | String  | Active security profile: `lab`, `staging`, or `production` (set at STEP 1) |
| `VAR_CONTRACT_CHECK`     | String  | `PASS`, `FAIL`, or `N/A` — result of the requirements contract check (set at STEP 4) |

---

## 4. Mandatory Operational Workflow

> **STEP TRACKING RULE:** At the **start** of every STEP, the agent MUST update
> `VAR_SESSION_STEP` to the current step number in `session/session_state.json`.
> This applies to ALL steps (0–7), with no exceptions.

### 4.1 Step Transition Rules

#### Forward-Only Principle
The workflow proceeds forward (0→1→2→3→4→5→7). STEP 6 is entered only when validation fails at STEP 5. Backward transitions are ONLY allowed through the explicit paths defined below.

#### Allowed Backward Transitions

| From    | To     | Condition                                              | Action Required                                      |
|---------|--------|--------------------------------------------------------|------------------------------------------------------|
| STEP 5  | STEP 6 | Validation FAIL                                        | Automatic                                            |
| STEP 6  | STEP 4 | User chooses "Retry" and fix scope is within existing backup | Increment `VAR_RETRY_COUNT`                    |
| STEP 6  | STEP 3 | User chooses "Retry" but fix requires modifying files NOT in original `backup_manifest.json` | Create incremental backup and proceed to STEP 4 (see §4.1 Retry with Extended Scope) |

#### Retry with Extended Scope (STEP 6 → STEP 3 → STEP 4)

If the agent determines that the fix requires modifying files NOT covered by the existing `backup_manifest.json`:

1. Inform the user: *"The fix requires changes to files not in the current backup. I need to create an incremental backup before proceeding."*
2. Set `VAR_SESSION_STEP` to `3`.
3. Create an incremental backup folder: `backups/YYYYMMDD_HHMM_RETRY_N/` (where N = `VAR_RETRY_COUNT`).
4. Generate `backup_manifest_retry_N.json` inside the incremental backup folder, covering ONLY the newly scoped files.
5. Proceed to STEP 4 with `VAR_SESSION_STEP` = `4`.
6. At STEP 5, the non-regression check (5.0) must validate against BOTH the original `backup_manifest.json` AND all `backup_manifest_retry_N.json` files.

#### Prohibited Transitions

- **Never** go back to STEP 0, STEP 1, or STEP 2 from any later step.
- **Never** skip STEP 3 when the retry scope is extended.
- If `VAR_RETRY_COUNT` ≥ 3, backward transitions to STEP 3 or STEP 4 are not allowed — only Rollback or Abort.

---

### 4.2 Step Gate Validation Protocol (MANDATORY)

Every step transition is governed by the gates defined in `rules/workflow_gates.md`.

**Before entering ANY step N, the agent MUST:**
1. READ `session_state.json` and verify `VAR_SESSION_STEP` equals the expected previous value
2. VERIFY all preconditions for STEP N from `rules/workflow_gates.md`
3. If ANY precondition fails: STOP, inform user, do NOT enter the step

**Before advancing from STEP N to STEP N+1, the agent MUST:**
1. VERIFY all postconditions for STEP N from `rules/workflow_gates.md`
2. LOG the result in `session/step_evidence.json`
3. If ANY postcondition fails: STOP, inform user, do NOT advance
4. ONLY THEN write the new `VAR_SESSION_STEP` value

**Gate failure message format:**
> "⛔ Gate check failed for transition STEP N → STEP N+1
> Failed conditions: [list condition IDs and their descriptions]
> Please resolve the above conditions before I can proceed."

---

### STEP 0: Bootstrap & Rules Loading

1. Update `VAR_SESSION_STEP` to `0` in `session_state.json`.
2. **Docker Prerequisite Check (MANDATORY — BLOCKING):**
   - Execute `docker --version` and `docker compose version` to verify Docker is installed.
   - If Docker is found and functional:
     - Set `VAR_DOCKER_INSTALLED` to `true` in `session_state.json`.
     - Quick health check: run `docker info` to verify the Docker daemon is running.
       If the daemon is not running, inform the user and attempt to start it.
   - If Docker is NOT found:
     - Inform the user: *"Docker is not installed. Docker is a mandatory prerequisite for AAOF — the framework cannot operate without it."*
     - Propose automatic installation with OS-appropriate instructions (detect OS first).
     - If the user approves → guide installation, verify with `docker --version`, set `VAR_DOCKER_INSTALLED` to `true`.
     - If the user refuses → **⛔ FULL STOP.** Set `VAR_DOCKER_INSTALLED` to `false`. Display: *"Docker is a mandatory prerequisite. The framework cannot proceed without Docker. Please install Docker and restart the session."* Do NOT proceed to any subsequent step.
   - On subsequent sessions: if `VAR_DOCKER_INSTALLED` is already `true`, perform only a quick health check (`docker info`) instead of the full installation check.

3. **Minikube Check (CONDITIONAL — only if `deploy_targets` includes `"K8S"` or `"KUBERNETES"` or `"MINIKUBE"`):**
   - Execute `minikube version` to verify Minikube is installed.
   - If Minikube is found:
     - Set `VAR_MINIKUBE_INSTALLED` to `true` and `VAR_MINIKUBE_APPROVED` to `true` in `session_state.json`.
   - If Minikube is NOT found:
     - Check `VAR_MINIKUBE_APPROVED` from the existing session state.
     - If `VAR_MINIKUBE_APPROVED` was previously set to `false`:
       - Remind the user: *"Minikube was previously declined. Kubernetes deployment remains disabled. Would you like to reconsider?"*
     - Otherwise, ask: *"Minikube is not installed but your deploy targets include Kubernetes. Would you like me to install it?"*
     - If the user approves → guide installation, verify with `minikube version`, set both variables to `true`.
     - If the user refuses → Set `VAR_MINIKUBE_INSTALLED` to `false` and `VAR_MINIKUBE_APPROVED` to `false`. Display: *"Kubernetes deployment is disabled. I will proceed with Docker-only deployment. You can enable Kubernetes at any time by authorizing Minikube installation."* Continue with the rest of STEP 0.
   - If `deploy_targets` does NOT include Kubernetes: set both `VAR_MINIKUBE_INSTALLED` and `VAR_MINIKUBE_APPROVED` to `false` and skip this check.

4. **Read ALL files in `rules/` except `rules/git_rules.md`** and assimilate them as your operational directives.
   These are your libraries — treat them with the same authority as this file.
   This includes `rules/testing_rules.md`, `rules/workflow_gates.md`, and `rules/security_profiles.md`.
5. **Test Baseline Loading:**
   - If `output/test_playbook.md` exists:
     - Read it as the regression test baseline.
     - Count the total number of tests defined in it.
     - Set `VAR_TEST_BASELINE` to `true` and `VAR_TEST_COUNT` to the test count
       in `session_state.json`.
   - If it does not exist, set `VAR_TEST_BASELINE` to `false` and `VAR_TEST_COUNT` to `0`.
6. Inventory your available MCP servers; record them in `VAR_AVAILABLE_MCP`.
7. Read `config.json` and the optional `output/deployed_state.json`.
8. **Session Check:** If `session/session_state.json` exists, load it to resume progress.
   - Additionally, if `session/requirements_checklist.json` exists (indicating the session
     has already progressed past STEP 2), load it to resume the active requirements
     checklist. This ensures the contract check state is preserved across sessions.
9. **Git Integration Check:** If `version_control.enabled` is `true` in `config.json`,
   read and apply `rules/git_rules.md` as operational directives. Set `VAR_GIT_ENABLED`
   to `true` in `session_state.json`. **If `false` or absent, set `VAR_GIT_ENABLED` to `false`.**

### STEP 1: Priority Resolution and Versioning

- Update `VAR_SESSION_STEP` to `1` in `session_state.json`.
- **State Conflict:** If `output/deployed_state.json` exists, ask the user:
  *"I found an existing release. Use versions from `deployed_state.json` or `config.json`?"*
- Save the choice in `VAR_SOURCE_OF_TRUTH`.
- If specific versions are missing, ask the user and update `VAR_CONFIRMED_VERSIONS`.
- **Environment Classification (MANDATORY):**
  1. Read `environment_context.type` from `config.json`.
  2. If not set: ASK the user — *"Is this a `lab/dev`, `staging`, or `production` environment? I cannot proceed without this classification."*
  3. Load the corresponding security profile from `rules/security_profiles.md`.
  4. Present to user: *"This project is classified as **[TYPE]**. Security profile loaded:*
     - *TLS/HTTPS: [implication for this profile]*
     - *Secrets management: [implication for this profile]*
     - *Password hashing: [implication for this profile]*
     - *Network isolation: [implication for this profile]*
     *Do you confirm?"*
  5. Wait for user confirmation. Store in `VAR_SECURITY_PROFILE` in `session_state.json` ONLY after user confirms.

### STEP 2: Execution Plan (Proposal)

- Update `VAR_SESSION_STEP` to `2` in `session_state.json`.
- Present an action plan reading confirmed variables from the session file.
  Follow the quality standards defined in `rules/design_review_rules.md`.
  The plan MUST include at least 2 architectural or technological alternatives with
  trade-offs as required by `rules/design_review_rules.md`. If only one approach is
  viable, explicitly document WHY alternatives were excluded.
- **K8s Externalization Analysis (if `deploy_targets` includes Kubernetes AND `VAR_MINIKUBE_APPROVED` is `true`):**
  Follow the "Nothing Hardcoded Inside" principle from `rules/kubernetes_rules.md` §10:
  1. Analyze the project source code and identify ALL files that contain configuration.
     Scope the scan to `src/`, `config/`, and the project root directory; respect
     `.gitignore` patterns to exclude `node_modules/`, `vendor/`, and build artifacts.
     Look for patterns: `.yml`, `.yaml`, `.properties`, `.conf`, `.ini`, `.json` config,
     `.env`, `.xml` config, framework-specific config files.
  2. Classify each file/parameter into one of 3 categories:
     - **ConfigMap** (mounted as file): application config files, static pages, web server configs
     - **Secret**: credentials, certificates, tokens, API keys
     - **Environment variable** (from ConfigMap): simple runtime parameters
  3. Present the externalization map to the user for approval:
     *"I identified the following items to externalize for Kubernetes deployment:*
     *📄 ConfigMap (mounted as files): [list]*
     *🔒 Secret: [list]*
     *🔧 Environment Variables (from ConfigMap): [list]*
     *Do you confirm this structure? Would you like to modify anything?"*
  4. Wait for user confirmation → save the approved map in `VAR_K8S_EXTERNALIZATION_MAP`.
  5. Use this map during STEP 4 to generate correct K8s manifests.
- **STOP:** Wait for user "GO" before touching the `output/` folder.
- **Requirements Checklist (MANDATORY — after user "GO"):**
  After receiving the user's "GO" approval, generate `session/requirements_checklist.json`
  following the structure defined in `rules/requirements_checklist.md`:
  1. Extract ALL functional, infrastructure, security, and testing requirements from the
     approved execution plan.
  2. For each requirement, identify the expected artifact file paths in `output/` and a
     clear, verifiable criterion.
  3. Assign sequential IDs (`REQ-001`, `REQ-002`, ...) and set all statuses to `"pending"`.
  4. Write the file to `session/requirements_checklist.json`.
  5. Confirm to the user: *"✅ Requirements checklist created with N requirements. All will
     be verified via contract check at the end of STEP 4 before advancing to STEP 5."*
  6. Set `VAR_CONTRACT_CHECK` to `"N/A"` in `session_state.json` (pending execution).

### STEP 3: Backup Protocol

- Update `VAR_SESSION_STEP` to `3` in `session_state.json`.
- Before modifying any file in `output/`, create a copy in
  `backups/YYYYMMDD_HHMM_[TASK_NAME]/`.
- Record the backup folder path in `session_state.json` as `VAR_ACTIVE_BACKUP_PATH`.
- **Pre-Backup Inventory (MANDATORY):** Before starting the backup, generate a
  `backup_manifest.json` inside the backup folder containing:
  - List of ALL files in `output/` with their SHA-256 hash
  - List of all public methods/functions per source file
  - List of all Docker volumes (named and bind mounts) from compose files
  - List of all K8s resources (Deployments, Services, PVCs, ConfigMaps, Secrets, Ingress)
    *(FAIL enforcement at STEP 5.0 applies only to stateful resources — volumes and PVCs.
    Removal of stateless K8s resources triggers a WARN.)*
  - List of all exposed ports and endpoints
  - List of all environment variables defined in `.env`, compose, and K8s manifests
- **Backup Completeness Check:** Verify the backup folder contains ALL files listed in
  the manifest before proceeding to STEP 4.
- **SCRATCH Project Baseline:** If `VAR_PROJECT_MODE` is `SCRATCH` and `output/` contains
  no source files (check before any STEP 4 work begins), generate a `backup_manifest.json`
  with empty arrays:
  `{"files": [], "methods": [], "volumes": [], "k8s_resources": [], "ports": [], "env_vars": []}`.
  Record `VAR_ACTIVE_BACKUP_PATH` normally. This establishes the baseline for non-regression
  checks — any file present in `output/` after STEP 4 that is NOT in the manifest is new
  (expected), while the empty baseline ensures no removal checks trigger false positives.

### STEP 4: Implementation

- Update `VAR_SESSION_STEP` to `4` in `session_state.json`.
- Write code in `output/` strictly following the rules in `rules/`.
- **ALL development must happen inside containers** — see `rules/docker_rules.md`.
- Follow file size and modularity rules from `rules/development_rules.md`.
- **Git Integration (if `VAR_GIT_ENABLED`):** Create a `feature/<task-name>` or
  `fix/<task-name>` branch before starting implementation. Commit incrementally
  with conventional commit messages (see `rules/git_rules.md`).
- **K8s Externalization (if `VAR_K8S_EXTERNALIZATION_MAP` is set and non-empty):** Generate Kubernetes
  manifests (ConfigMaps, Secrets, volume mounts) according to the approved externalization
  map. Every configuration file identified in the map MUST be externalized — no configuration
  hardcoded inside container images. See `rules/kubernetes_rules.md` §10.
- **Test-Driven Development (MANDATORY):** For every new function, method, or endpoint,
  follow the RED-GREEN-REFACTOR cycle defined in `rules/testing_rules.md` §1.1.
  Write the failing test first, then the minimal implementation, then refactor.
  Commit test + implementation together.
- **Contract Check (MANDATORY — before advancing to STEP 5):**
  After all implementation work is complete, run the contract check as defined in
  `rules/requirements_checklist.md` §4:
  1. READ `session/requirements_checklist.json`.
  2. For each requirement, verify all declared artifacts exist and are non-empty, and
     apply any additional verification from the `verification` field.
  3. Set each requirement's `status` to `"pass"` or `"fail"` and record findings in `evidence`.
  4. Set `VAR_CONTRACT_CHECK` in `session_state.json` to `"PASS"` (all pass/waived) or `"FAIL"`.
  5. Write the updated `session/requirements_checklist.json`.
  6. Log the contract check result in `session/step_evidence.json` (see `rules/requirements_checklist.md` §5).
  7. If `VAR_CONTRACT_CHECK` == `"FAIL"`: **⛔ STOP.** Report all failing requirements to the user
     with the format defined in `rules/requirements_checklist.md` §4.2. Do NOT advance to STEP 5.
  8. If the user explicitly waives a requirement, update its `status` to `"waived"` and re-run the check.
  Only when `VAR_CONTRACT_CHECK` == `"PASS"` may the agent advance to STEP 5.

### STEP 5: Validation & Test

- Update `VAR_SESSION_STEP` to `5` in `session_state.json`.
- Follow the test strategy defined in `rules/testing_rules.md`.
- Generate `output/test_playbook.md` and `output/test_results.md` as required by
  `rules/testing_rules.md` §3.

**5.0. Non-Regression Check (MANDATORY — before build):**
  - Load `backup_manifest.json` from `VAR_ACTIVE_BACKUP_PATH`.
  - Also load any `backup_manifest_retry_N.json` files from retry backup folders (if they exist).
  - Compare the current `output/` against the manifest:
    - ❌ **FAIL** if any file listed in the manifest is missing from `output/`.
    - ❌ **FAIL** if any public method/function listed in the manifest was removed without a corresponding entry in the task specification or explicit user approval during this session.
    - ❌ **FAIL** if any Docker volume or K8s PVC listed in the manifest was removed.
    - ❌ **FAIL** if any exposed port or endpoint listed in the manifest was removed.
    - ⚠️ **WARN** if any stateless K8s resource (Deployments, Services, ConfigMaps, Ingress) listed in the manifest was removed.
    - ⚠️ **WARN** if environment variables listed in the manifest were removed (may be intentional).
  - If any FAIL condition is detected: **STOP** and present the differences to the user. Ask for explicit approval before proceeding.
  - If `output/test_playbook.md` exists from a prior session, verify that ALL tests in it still PASS.
  - If `VAR_TEST_BASELINE` is `true`, verify that the total test count has NOT decreased
    compared to `VAR_TEST_COUNT`. New functionality MUST add new tests, never remove existing ones
    without explicit user approval. If the count has decreased: **STOP** and request explicit
    user approval before proceeding.
  - Log the comparison result in `session_state.json` as `VAR_REGRESSION_CHECK`: `PASS`, `FAIL`, or `WARN`.

**5.0.5. Compliance Check (MANDATORY — after non-regression, before build):**
  1. Read `rules/output_checklist.json`.
  2. Read `VAR_SECURITY_PROFILE` from `session_state.json`.
  3. For each check in the checklist, determine the effective severity using `VAR_SECURITY_PROFILE`
     (look up `severity_by_profile[VAR_SECURITY_PROFILE]`).
  4. Execute the verification described in `how_to_verify` for each check.
  5. Record the result for each check: `PASS`, `WARN`, or `FAIL`.
  6. Generate `output/compliance_report.md` with a results table:
     - Include check ID, category, rule description, result, and severity.
  7. If ANY check returns `FAIL`: **STOP**, present the failures to the user, wait for approval before continuing.
  8. If checks return only `WARN`: log all warnings, include them in `output/test_results.md`, and continue.

5.1. **Build:** Execute `docker-compose build` or `docker build`.
   - If fails → go to STEP 6.
5.2. **Run:** Start containers (`docker-compose up -d`) and verify they start.
   - If fails → go to STEP 6.
5.3. **Health check:** Verify services respond (curl, open port, logs without errors).
   - See `rules/error_handling_rules.md` for interpretation guidance.
5.4. **Functional Tests:** Execute all tests defined in `rules/testing_rules.md` according
   to the service type(s) in `config.json`.
   - Generate `output/test_playbook.md` before test execution.
   - Execute all tests defined in the playbook.
   - Generate `output/test_results.md` with full results.
   - If any test fails → go to STEP 6.
5.4.1. **Kubernetes Validation (if `VAR_DEPLOY_TARGET` includes `K8S`):**
   After Docker validation passes (5.1–5.4), apply K8s manifests to Minikube and run
   a second validation cycle:
   - `kubectl apply -k` or `kubectl apply -f` on all manifests
   - Verify pods reach `Running` state
   - Health check K8s services (port-forward or NodePort)
   - Run functional tests against K8s endpoints
   Both Docker AND K8s validation must PASS for `VAR_VALIDATION_RESULT` = `PASS`.
   If Minikube is not available (`VAR_MINIKUBE_APPROVED` is `false`), skip this
   sub-step and log a WARN. `VAR_VALIDATION_RESULT` is still set based on Docker
   validation results alone; the WARN is recorded in `session_state.json` as a note
   that K8s validation was not performed.
5.5. **Report:** Write result in `session_state.json`:
   - `VAR_VALIDATION_RESULT`: `PASS` or `FAIL`.
   - **MANDATORY:** `VAR_VALIDATION_RESULT` MUST be set ONLY after the agent has directly
     observed test results from executed commands (STEP 5.1–5.4). Setting `PASS` based on
     "the code looks correct", user confirmation, or unexecuted tests is a violation of
     the Evidence Over Claims golden rule (§5).
5.6. If `PASS` → proceed to STEP 7. If `FAIL` → go to STEP 6.

### STEP 6: Rollback Gate

1. Update `VAR_SESSION_STEP` to `6` in `session_state.json`.
2. **Notify:** Inform the user of the failure with the full error detail.
   Follow the Investigation Protocol defined in `rules/debugging_rules.md` before proposing a fix.
3. **Propose options:**
   - **Retry** — go back to STEP 4 to correct the issue.
     Increment `VAR_RETRY_COUNT`. If `VAR_RETRY_COUNT` ≥ 3, do not offer Retry.
   - **Retry (Extended)** — if the fix requires files outside the current backup scope,
     go back to STEP 3 for incremental backup, then STEP 4.
     Increment `VAR_RETRY_COUNT`. If `VAR_RETRY_COUNT` ≥ 3, do not offer Retry (Extended).
   - **Rollback** — restore files from `backups/` to the pre-task state.
   - **Abort** — stop execution; save current state for future analysis.
4. **STOP:** Wait for user decision before doing anything.

### STEP 7: Consolidation, State Backup, and Cleanup

1. Update `VAR_SESSION_STEP` to `7` in `session_state.json`.
2. **Final State:** Update `output/deployed_state.json` with the final technical snapshot.
3. **Session Backup:** Copy `session/session_state.json` to `backups/` (date-prefixed).
4. **Archiving:** Move files from `specs/active/` to `specs/history/`.
5. **Cleanup:** Empty the `session/` folder.
6. **Changelog:** Record the activity in `changelog.md`.
7. **Kubernetes Deploy (if `deploy_targets` includes Kubernetes):**
   - If `VAR_MINIKUBE_APPROVED` is `false`:
     - SKIP Kubernetes deployment.
     - Display: *"⚠️ Kubernetes deployment skipped: Minikube not authorized. Authorize Minikube installation to enable K8s deployment."*
   - If `VAR_MINIKUBE_APPROVED` is `true`:
     - Apply K8s manifests to Minikube following `rules/kubernetes_rules.md`.
     - Verify all pods are running and healthy.
8. **Git Release (if `VAR_GIT_ENABLED`):**
   - Merge the working branch into `develop` (or `main` if no `develop` branch).
     If the project has CI/CD configured, this should be done via Pull Request per `rules/git_rules.md`.
   - Determine the new version based on commit types (see `rules/git_rules.md` §3).
   - If `version_control.auto_tag` is `true`, create an annotated git tag: `v<MAJOR>.<MINOR>.<PATCH>`.
   - Push branch, tag, and changes to remote.
   - Update `VAR_CURRENT_VERSION` in `session_state.json`.

---

## 5. Golden Rules

- **No Assumptions:** If a technical parameter is ambiguous, ask.
- **Documentation:** Every change must be traceable in `changelog.md`.
- **Persistence:** The JSON file is the only source of truth for the session state.
- **Container-First:** All development and testing happens inside Docker containers.
- **Security:** Follow `rules/security_rules.md` at all times — no exceptions.
- **Test-First:** No production code without a failing test first — see `rules/testing_rules.md` §1.1.
- **Evidence Over Claims:** Never declare success without fresh verification evidence — see `rules/testing_rules.md` §6.
- **Nothing Hardcoded Inside:** Every configuration that may vary between environments must be externalized as ConfigMap, Secret, or environment variable — see `rules/kubernetes_rules.md` §10.
