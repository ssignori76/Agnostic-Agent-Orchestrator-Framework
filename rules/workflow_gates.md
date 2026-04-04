# 🚦 rules/workflow_gates.md — Step Gate Machine

> **MANDATORY:** Every step transition is governed by the gates defined in this file.
> The agent MUST verify ALL preconditions before entering a step and ALL postconditions
> before advancing to the next step. Gate failures are BLOCKING — not advisory.

---

## STEP 0: Bootstrap & Rules Loading

### Preconditions (ALL must be TRUE to enter this step)

| ID | Condition | How to verify |
|----|-----------|---------------|
| PRE-0.1 | None (entry point) | Always satisfied |

### Postconditions (ALL must be TRUE to advance to STEP 1)

| ID | Condition | How to verify |
|----|-----------|---------------|
| POST-0.1 | `VAR_DOCKER_INSTALLED` == true | Read `session_state.json`, check value |
| POST-0.2 | All `rules/*.md` loaded (except `git_rules.md` if `version_control.enabled` is false) | Confirm each file was read during this step |
| POST-0.3 | `session_state.json` initialized (created or loaded) | File exists in `session/` |
| POST-0.4 | `VAR_SESSION_STEP` == 0 | Read `session_state.json`, check value |

### Gate Failure Action

If any precondition fails: STOP, inform user which condition failed, do NOT enter the step.
If any postcondition fails: STOP, inform user which condition failed, do NOT advance.

---

## STEP 1: Priority Resolution and Versioning

### Preconditions (ALL must be TRUE to enter this step)

| ID | Condition | How to verify |
|----|-----------|---------------|
| PRE-1.1 | `VAR_SESSION_STEP` == 0 | Read `session_state.json`, check value |
| PRE-1.2 | `VAR_DOCKER_INSTALLED` == true | Read `session_state.json`, check value |
| PRE-1.3 | `session/session_state.json` exists | File system check |

### Postconditions (ALL must be TRUE to advance to STEP 2)

| ID | Condition | How to verify |
|----|-----------|---------------|
| POST-1.1 | `VAR_SOURCE_OF_TRUTH` is set (non-empty string) | Read `session_state.json` |
| POST-1.2 | `VAR_CONFIRMED_VERSIONS` is non-empty object | Read `session_state.json`, check at least 1 key |
| POST-1.3 | `VAR_SECURITY_PROFILE` is set to `lab`, `staging`, or `production` | Read `session_state.json` |
| POST-1.4 | `config.json` reflects the actual project stack being built | Verify stack fields match the spec in `specs/active/` |

### Gate Failure Action

If any precondition fails: STOP, inform user which condition failed, do NOT enter the step.
If any postcondition fails: STOP, inform user which condition failed, do NOT advance.

---

## STEP 2: Execution Plan (Proposal)

### Preconditions (ALL must be TRUE to enter this step)

| ID | Condition | How to verify |
|----|-----------|---------------|
| PRE-2.1 | `VAR_SESSION_STEP` == 1 | Read `session_state.json`, check value |
| PRE-2.2 | `VAR_CONFIRMED_VERSIONS` is non-empty | Read `session_state.json` |
| PRE-2.3 | `specs/active/` contains ≥1 file | List directory; count files |

### Postconditions (ALL must be TRUE to advance to STEP 3)

| ID | Condition | How to verify |
|----|-----------|---------------|
| POST-2.1 | Execution plan presented with ≥2 alternatives (or documented reason for single approach) | Plan text was output to user during this step |
| POST-2.2 | User explicit "GO" received | User typed "GO" or equivalent approval during this step |
| POST-2.3 | `VAR_SESSION_STEP` == 2 | Read `session_state.json`, check value |
| POST-2.4 | `session/requirements_checklist.json` created with all functional requirements from the execution plan (all statuses set to `pending`) | File exists in `session/` and contains ≥1 requirement entry |

### Gate Failure Action

If any precondition fails: STOP, inform user which condition failed, do NOT enter the step.
If any postcondition fails: STOP, inform user which condition failed, do NOT advance.

---

## STEP 3: Backup Protocol

### Preconditions (ALL must be TRUE to enter this step)

| ID | Condition | How to verify |
|----|-----------|---------------|
| PRE-3.1 | `VAR_SESSION_STEP` == 2 | Read `session_state.json`, check value |
| PRE-3.2 | User "GO" was received at STEP 2 (session log) | `POST-2.2` was satisfied — verifiable from step_evidence.json |

### Postconditions (ALL must be TRUE to advance to STEP 4)

| ID | Condition | How to verify |
|----|-----------|---------------|
| POST-3.1 | `VAR_ACTIVE_BACKUP_PATH` is set and non-empty | Read `session_state.json` |
| POST-3.2 | `backup_manifest.json` exists in backup folder | File system check at `VAR_ACTIVE_BACKUP_PATH` |
| POST-3.3 | Backup completeness verified (all files in manifest are present in backup folder) | Compare manifest file list against actual backup folder contents |

### Gate Failure Action

If any precondition fails: STOP, inform user which condition failed, do NOT enter the step.
If any postcondition fails: STOP, inform user which condition failed, do NOT advance.

---

## STEP 4: Implementation

### Preconditions (ALL must be TRUE to enter this step)

| ID | Condition | How to verify |
|----|-----------|---------------|
| PRE-4.1 | `VAR_SESSION_STEP` == 3 | Read `session_state.json`, check value |
| PRE-4.2 | `VAR_ACTIVE_BACKUP_PATH` is set | Read `session_state.json` |
| PRE-4.3 | `backup_manifest.json` exists | File system check at `VAR_ACTIVE_BACKUP_PATH` |

### Postconditions (ALL must be TRUE to advance to STEP 5)

| ID | Condition | How to verify |
|----|-----------|---------------|
| POST-4.1 | Source files exist in `output/` | List `output/` directory; confirm non-empty |
| POST-4.2 | TDD evidence recorded (test files created before/with implementation, or explicit user-approved exception documented in `session_state.json`) | Check test file timestamps or user approval record |
| POST-4.3 | All code follows `rules/development_rules.md` (file headers, line limits, error handling) | Manual review or automated check |
| POST-4.4 | Contract check passed — all requirements in `session/requirements_checklist.json` have `status == "pass"` or `"waived"` (i.e. `VAR_CONTRACT_CHECK` == `PASS`) | Read `session_state.json`; read `session/requirements_checklist.json` for detail |

### Gate Failure Action

If any precondition fails: STOP, inform user which condition failed, do NOT enter the step.
If any postcondition fails: STOP, inform user which condition failed, do NOT advance.

---

## STEP 5: Validation & Test

### Preconditions (ALL must be TRUE to enter this step)

| ID | Condition | How to verify |
|----|-----------|---------------|
| PRE-5.1 | `VAR_SESSION_STEP` == 4 | Read `session_state.json`, check value |
| PRE-5.2 | Source files exist in `output/` | List `output/` directory |
| PRE-5.3 | `backup_manifest.json` is available for regression check | File exists at `VAR_ACTIVE_BACKUP_PATH` |

### Postconditions (ALL must be TRUE to advance to STEP 6 or STEP 7)

| ID | Condition | How to verify |
|----|-----------|---------------|
| POST-5.1 | `VAR_VALIDATION_RESULT` is set to `PASS` or `FAIL` (based on executed evidence, not assumptions) | Read `session_state.json` |
| POST-5.2 | `output/test_playbook.md` exists | File system check |
| POST-5.3 | `output/test_results.md` exists | File system check |
| POST-5.4 | `output/compliance_report.md` exists | File system check (generated at STEP 5.0.5) |
| POST-5.5 | `VAR_REGRESSION_CHECK` is set to `PASS`, `FAIL`, or `WARN` | Read `session_state.json` |

### Gate Failure Action

If any precondition fails: STOP, inform user which condition failed, do NOT enter the step.
If any postcondition fails: STOP, inform user which condition failed, do NOT advance.

---

## STEP 6: Rollback Gate

### Preconditions (ALL must be TRUE to enter this step)

| ID | Condition | How to verify |
|----|-----------|---------------|
| PRE-6.1 | `VAR_VALIDATION_RESULT` == `FAIL` | Read `session_state.json`, check value |

### Postconditions (ALL must be TRUE to conclude STEP 6)

| ID | Condition | How to verify |
|----|-----------|---------------|
| POST-6.1 | User decision recorded in `session_state.json` (one of: `Retry`, `Retry Extended`, `Rollback`, `Abort`) | Read `session_state.json` |

### Gate Failure Action

If any precondition fails: STOP, inform user which condition failed, do NOT enter the step.
If any postcondition fails: STOP, inform user which condition failed, do NOT advance.

---

## STEP 7: Consolidation, State Backup, and Cleanup

### Preconditions (ALL must be TRUE to enter this step)

| ID | Condition | How to verify |
|----|-----------|---------------|
| PRE-7.1 | `VAR_VALIDATION_RESULT` == `PASS` | Read `session_state.json`, check value |
| PRE-7.2 | `VAR_REGRESSION_CHECK` != `FAIL` | Read `session_state.json`, check value |

### Postconditions (ALL must be TRUE to conclude STEP 7)

| ID | Condition | How to verify |
|----|-----------|---------------|
| POST-7.1 | `output/deployed_state.json` updated with final snapshot | File exists and timestamp is from this session |
| POST-7.2 | `specs/active/` archived (moved to `specs/history/`) | `specs/active/` is empty or contains only new unprocessed specs |
| POST-7.3 | `changelog.md` updated with a new entry for this session | Latest entry in `changelog.md` has today's date |

### Gate Failure Action

If any precondition fails: STOP, inform user which condition failed, do NOT enter the step.
If any postcondition fails: STOP, inform user which condition failed, do NOT advance.

---

## Step Transition Validation Protocol

Before changing `VAR_SESSION_STEP` from N to N+1, the agent MUST:

1. **READ** `session_state.json` — verify current value equals N
2. **CHECK** all postconditions for STEP N — each must be TRUE
3. **CHECK** all preconditions for STEP N+1 — each must be TRUE
4. **LOG** the gate check result in `session/step_evidence.json`
5. **ONLY THEN** write the new step value to `session_state.json`

If ANY condition fails:
- STOP immediately
- Inform the user: "⛔ Gate check failed for transition STEP N → STEP N+1"
- List which specific conditions (by ID) failed
- Do NOT write the new step value
- Wait for user instruction

### `session/step_evidence.json` Format

The agent creates and appends to this file at every step transition:

```json
{
  "step_transitions": [
    {
      "from": 0,
      "to": 1,
      "timestamp": "ISO-8601",
      "postconditions_checked": {
        "POST-0.1": { "condition": "VAR_DOCKER_INSTALLED == true", "result": "PASS" },
        "POST-0.2": { "condition": "All rules loaded", "result": "PASS" },
        "POST-0.3": { "condition": "session_state.json initialized", "result": "PASS" },
        "POST-0.4": { "condition": "VAR_SESSION_STEP == 0", "result": "PASS" }
      },
      "preconditions_checked": {
        "PRE-1.1": { "condition": "VAR_SESSION_STEP == 0", "result": "PASS" },
        "PRE-1.2": { "condition": "VAR_DOCKER_INSTALLED == true", "result": "PASS" },
        "PRE-1.3": { "condition": "session_state.json exists", "result": "PASS" }
      },
      "gate_result": "PASS"
    },
    {
      "from": 4,
      "to": 5,
      "timestamp": "ISO-8601",
      "postconditions_checked": {
        "POST-4.1": { "condition": "output/ non-empty", "result": "PASS" },
        "POST-4.2": { "condition": "TDD evidence recorded", "result": "PASS" },
        "POST-4.3": { "condition": "Code follows development_rules.md", "result": "PASS" },
        "POST-4.4": { "condition": "Contract check passed", "result": "PASS" }
      },
      "contract_check": {
        "total_requirements": 5,
        "passed": 5,
        "failed": 0,
        "waived": 0,
        "result": "PASS",
        "failed_ids": [],
        "waived_ids": []
      },
      "gate_result": "PASS"
    }
  ]
}
```
