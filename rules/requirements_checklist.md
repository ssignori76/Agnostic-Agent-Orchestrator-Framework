# 📋 rules/requirements_checklist.md — Contract Check and Requirements Checklist

> **MANDATORY:** Every STEP 4 implementation must be validated against the requirements
> declared at STEP 2. The agent MUST run the contract check before advancing to STEP 5.
> A failing contract check is BLOCKING — not advisory.

---

## 1. Purpose

The Requirements Checklist mechanism ensures that every functional requirement declared
in the execution plan at STEP 2 has been **concretely implemented** before the workflow
advances to STEP 5.

This prevents silent omissions where an agent passes gate checks without having
implemented all required features (e.g. a shopping cart module declared in the plan
but never created in `output/`).

---

## 2. `session/requirements_checklist.json` — Structure

The agent creates this file at the end of STEP 2 (after user "GO") and updates it
at the end of STEP 4 with per-requirement verification results.

```json
{
  "checklist_version": "1.0",
  "generated_at": "<ISO-8601 timestamp>",
  "step": 2,
  "execution_plan_summary": "Brief summary of the agreed execution plan",
  "requirements": [
    {
      "id": "REQ-001",
      "step": 4,
      "category": "functional",
      "description": "Human-readable description of the requirement",
      "artifacts": [
        "output/src/module/file.js"
      ],
      "verification": "How to verify this requirement is satisfied (what to check)",
      "status": "pending",
      "evidence": ""
    }
  ]
}
```

### Field Descriptions

| Field | Type | Description |
|-------|------|-------------|
| `checklist_version` | String | Always `"1.0"` for this schema version |
| `generated_at` | String | ISO-8601 timestamp when the file was created |
| `step` | Integer | Step at which this file was generated (always `2`) |
| `execution_plan_summary` | String | Short summary of the agreed execution plan |
| `requirements` | Array | List of all functional requirements to implement |
| `requirements[].id` | String | Unique requirement ID (e.g. `REQ-001`, `REQ-002`) |
| `requirements[].step` | Integer | Step responsible for implementing this requirement (always `4`) |
| `requirements[].category` | String | One of: `functional`, `infrastructure`, `security`, `testing` |
| `requirements[].description` | String | Plain-language description of the requirement |
| `requirements[].artifacts` | Array | List of file paths (relative to repo root) that must exist to satisfy this requirement |
| `requirements[].verification` | String | How to verify satisfaction (e.g. "File exists and exports a createCart() function") |
| `requirements[].status` | String | `pending` (initial), `pass`, `fail`, or `waived` (set at STEP 4 contract check; `waived` requires explicit user approval) |
| `requirements[].evidence` | String | Notes recorded during verification (populated at STEP 4) |

---

## 3. When to Generate the Checklist (STEP 2)

At the end of STEP 2, **after the user says "GO"**, the agent MUST:

1. Extract all functional, infrastructure, security, and testing requirements from the approved execution plan.
2. For each requirement, identify:
   - Which source files or artifacts are expected to be created in `output/`
   - A clear, verifiable verification criterion
3. Write `session/requirements_checklist.json` with all requirements having `status: "pending"`.
4. Confirm to the user: *"✅ Requirements checklist created with N requirements. All will be verified at the end of STEP 4 before advancing to STEP 5."*

> **Note:** If the execution plan is vague about specific file locations, use the
> declared module structure and naming conventions to infer reasonable artifact paths.
> Document the inference in the `verification` field.

---

## 4. Contract Check Protocol (STEP 4)

At the end of STEP 4, **before advancing to STEP 5**, the agent MUST run the
**Contract Check**:

### 4.1 Contract Check Steps

1. **READ** `session/requirements_checklist.json`.
2. For **each requirement** in the list:
   a. Verify that **all declared artifacts** exist in the file system.
   b. Verify that each artifact file is **non-empty** (size > 0).
   c. Apply any additional verification defined in the `verification` field
      (e.g. check that a specific function is exported, an endpoint is registered).
   d. Set `status` to `"pass"` if all checks pass; `"fail"` otherwise.
   e. Record verification notes in the `evidence` field.
3. **Count** passing and failing requirements.
4. **Set `VAR_CONTRACT_CHECK`** in `session_state.json`:
   - `"PASS"` if ALL requirements have `status == "pass"` or `status == "waived"`
   - `"FAIL"` if ANY requirement has `status == "fail"` or `status == "pending"`
5. **Write** the updated `session/requirements_checklist.json` with all statuses filled in.
6. **Log** the contract check result in `session/step_evidence.json` under the
   transition from STEP 4 to STEP 5 (see §5).

### 4.2 Contract Check Failure

If `VAR_CONTRACT_CHECK` is `"FAIL"`:

- **STOP** immediately. Do NOT advance to STEP 5.
- Display the following message:

```
⛔ Contract Check FAILED — transition STEP 4 → STEP 5 blocked.

The following requirements declared in the execution plan have not been implemented:

[For each failed requirement:]
  ❌ REQ-XXX: <description>
     Missing artifacts: <list of missing files>
     Verification criterion: <verification text>

Please implement the missing requirements before advancing to STEP 5.
```

- Wait for the user's instruction (fix and re-run, or explicit waiver).
- If the user explicitly waives a requirement, update its `status` to `"waived"` and
  record the user's reason in the `evidence` field. Waived requirements do NOT block
  the transition.

### 4.3 Partial Passes

All requirements must pass (or be explicitly waived) before `VAR_CONTRACT_CHECK`
can be set to `"PASS"`. There is no partial-pass concept — the check is binary.

---

## 5. Integration with `session/step_evidence.json`

When logging the transition from STEP 4 to STEP 5, the agent MUST include a
`contract_check` field in the evidence entry:

```json
{
  "step_transitions": [
    {
      "from": 4,
      "to": 5,
      "timestamp": "<ISO-8601>",
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

---

## 6. Requirement Categories

| Category | Description | Examples |
|----------|-------------|---------|
| `functional` | Business logic features | User authentication, shopping cart, product listing |
| `infrastructure` | Deployment and runtime setup | Docker compose file, database migration scripts |
| `security` | Security controls | Input validation, password hashing, CORS config |
| `testing` | Test coverage obligations | Unit tests for auth, integration tests for cart |

---

## 7. Naming Conventions for Requirement IDs

Use the prefix `REQ-` followed by a zero-padded three-digit number:

- `REQ-001`, `REQ-002`, ..., `REQ-009`, `REQ-010`, ...

Group requirements by category in the list (all `functional` first, then
`infrastructure`, then `security`, then `testing`) for readability.

---

## 8. Re-running the Contract Check

If the agent returns to STEP 4 after a STEP 6 rollback gate:

1. **Do NOT delete** `session/requirements_checklist.json`.
2. Reset the `status` of all failed requirements back to `"pending"`.
3. Preserve `"pass"` status for requirements that were already satisfied.
4. Run the contract check again at the end of the new STEP 4 iteration.

This allows incremental progress — requirements satisfied in earlier iterations
are not re-checked from scratch unless their artifacts have changed.
