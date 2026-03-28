# Design Review Rules

> **Purpose:** Ensure the agent produces a thoughtful, validated design before
> writing any code, preventing rework and improving implementation quality.
>
> **Scope:** All AAOF projects. Referenced at STEP 2 (Execution Plan) of the
> agent workflow.

---

## 1. Core Design Principle

```
NO CODE WITHOUT AN APPROVED DESIGN FIRST
```

Before the agent writes any implementation code (STEP 4), it MUST have a design
that has been reviewed and approved by the user at STEP 2.

---

## 2. Design Exploration Phase (STEP 2 Pre-requisite)

Before presenting the execution plan, the agent MUST:

### 2.1 Context Gathering

1. **Read all active specs** in `specs/active/` — understand what is being requested.
2. **Inventory existing output** — read `output/deployed_state.json` and list current
   services, endpoints, volumes, and dependencies.
3. **Identify impact scope** — which existing components will be affected by the change?
4. **Check for conflicts** — does the new requirement conflict with existing functionality?

### 2.2 Approach Proposal

Present **2-3 alternative approaches** with trade-offs:

| Approach | Description | Pros | Cons | Estimated Complexity |
|----------|-------------|------|------|---------------------|
| A        | ...         | ...  | ...  | Low / Medium / High |
| B        | ...         | ...  | ...  | Low / Medium / High |

- **STOP:** Ask the user which approach to pursue before proceeding.
- If the user doesn't choose, the agent MUST NOT proceed.

### 2.3 Design Validation Checklist

Before finalizing the execution plan, verify:

- [ ] All requirements from `specs/active/` are addressed
- [ ] No existing functionality is unintentionally removed
- [ ] File structure follows `rules/development_rules.md` (modularity, size limits)
- [ ] Docker/K8s implications are identified (new services, volumes, ports)
- [ ] Test strategy is outlined (what will be tested and how)
- [ ] Security implications are assessed per `rules/security_rules.md`

---

## 3. Execution Plan Quality Standards

The execution plan presented at STEP 2 MUST contain:

### 3.1 Required Sections

1. **Goal:** One sentence describing what this change achieves.
2. **Approach:** Which approach was selected and why.
3. **File Map:** List of files to create, modify, or delete with one-line descriptions.
4. **Implementation Order:** Numbered list of changes in dependency order.
5. **Test Outline:** Which tests will be written (aligned with TDD cycle).
6. **Rollback Impact:** What happens if this change fails — what gets restored.

### 3.2 No Placeholders Rule

The plan MUST NOT contain:
- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" (specify WHAT error handling)
- "Write tests for the above" (specify WHICH tests)
- "Similar to X" (repeat the details — the plan must be self-contained)
- Vague descriptions like "update configuration as needed"

Every item in the plan must be specific and actionable.

---

## 4. Plan Self-Review

Before presenting the plan to the user, the agent MUST self-review:

1. **Spec Coverage:** For each requirement in `specs/active/`, point to the plan
   item that addresses it. If any requirement has no plan item, add one.
2. **Consistency Check:** Do file names, function names, and variable names used
   in later steps match what was defined in earlier steps?
3. **Completeness Check:** Does the plan cover creation, testing, documentation,
   and cleanup for every component?

---

## 5. User Approval Gate

The execution plan is a **hard gate**:

- The agent presents the plan and says: *"Here is the execution plan. Do you approve? (GO / suggest changes)"*
- If the user says "GO" → proceed to STEP 3.
- If the user suggests changes → update the plan and re-present.
- The agent MUST NOT start STEP 3 or STEP 4 without explicit user approval.

This rule is already implicit in `agent.md` STEP 2 ("STOP: Wait for user GO")
but this file makes the quality standards for the plan explicit.
