# Systematic Debugging Rules

> **Purpose:** Define a structured debugging methodology for the agent when errors
> occur during STEP 4 (Implementation) or STEP 5 (Validation), preventing blind
> trial-and-error fixes.
>
> **Scope:** All AAOF projects. Referenced at STEP 6 (Rollback Gate) of the agent workflow.

---

## 1. Core Debugging Principle

```
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
```

The agent MUST NOT apply speculative fixes. Every fix must be traceable to an
identified root cause with supporting evidence.

---

## 2. The Investigation Protocol

When a failure occurs (build error, test failure, runtime crash), the agent MUST
follow these 4 phases IN ORDER:

### Phase 1: Evidence Collection

1. **Read the full error output** — not just the last line. Scroll up.
2. **Identify the error type:**
   - Build error (compilation, dependency, syntax)
   - Runtime error (crash, exception, timeout)
   - Test failure (assertion mismatch, missing behavior)
   - Infrastructure error (port conflict, permission denied, resource limit)
3. **Capture context:**
   - Which file(s) are involved?
   - What was the last change made before the error?
   - Is this a new error or a regression from a prior change?

### Phase 2: Root Cause Analysis

1. **Trace the error to its origin** — follow the stack trace or error chain
   from the symptom back to the source.
2. **Find working reference code** — look for similar patterns in the codebase
   that work correctly. Compare the working code with the failing code.
3. **Check recent changes** — use `git diff` or compare against the
   `backup_manifest.json` to identify exactly what changed.
4. **Formulate a hypothesis** — state it explicitly:
   *"The error occurs because [X] was changed to [Y], which breaks [Z]."*

### Phase 3: Targeted Fix

1. **Change ONE thing at a time.** Never apply multiple fixes simultaneously.
2. **Write or update the test FIRST** (per `rules/testing_rules.md` §1.1 TDD):
   - Write a test that reproduces the bug.
   - Verify the test fails.
   - Apply the minimal fix.
   - Verify the test passes.
3. **Verify no regressions** — run the full test suite, not just the fixed test.
4. **Commit the fix with the test** — use conventional commit: `fix: <description>`.

### Phase 4: Post-Fix Verification

1. Re-run the FULL validation sequence (STEP 5.1 through 5.4).
2. Confirm `VAR_VALIDATION_RESULT` = `PASS`.
3. Only THEN report success to the user (per Verification Before Completion rules).

---

## 3. The Three-Strike Rule

This rule aligns with `VAR_RETRY_COUNT` (max 3) in `agent.md`:

| Attempt | Action |
|---------|--------|
| Strike 1 | Apply targeted fix following the Investigation Protocol above. |
| Strike 2 | Re-examine the root cause — the first hypothesis was likely wrong. Go back to Phase 2. |
| Strike 3 | STOP fixing. Question the **design**, not the code. Present the user with: the 3 failed attempts, evidence collected, and a proposal to refactor the approach. |

After 3 strikes, the agent MUST NOT offer "Retry" — only Rollback or Abort
(consistent with `agent.md` §4.1 Prohibited Transitions).

---

## 4. Anti-Pattern Detection

### Red Flags — STOP Immediately

If the agent catches itself doing any of these, it MUST stop and restart from Phase 1:

- **Shotgun debugging:** Changing multiple things hoping something works
- **Copy-paste fix:** Copying code from StackOverflow/docs without understanding it
- **Symmetry fix:** "This works elsewhere, so let me copy it" without understanding WHY
- **Revert and retry:** Undoing changes and redoing them identically
- **"It works on my machine":** Assuming the environment is wrong instead of the code

### Anti-Rationalization Table

| Excuse the agent might use | Reality |
|---|---|
| "Let me just try this quick fix" | Quick fixes without diagnosis create new bugs. |
| "The error message is misleading" | Read the FULL output. The real cause is usually there. |
| "It must be a dependency issue" | Verify with evidence before blaming externals. |
| "I'll fix it properly later" | There is no later. Fix it properly now or rollback. |
| "This worked before, something else broke it" | Use git diff to PROVE what changed. |

---

## 5. Integration with STEP 6

When the agent enters STEP 6 (Rollback Gate), it MUST:

1. Present the failure with **full evidence** (not just "build failed").
2. Include the **root cause hypothesis** from Phase 2.
3. Include the **attempted fix** and why it didn't work (if retrying).
4. Check `VAR_RETRY_COUNT`:
   - If < 3: Offer Retry with a clear fix plan based on the Investigation Protocol.
   - If ≥ 3: Do NOT offer Retry. Only Rollback or Abort.
