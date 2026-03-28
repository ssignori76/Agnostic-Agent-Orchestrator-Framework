# Testing Rules

> **Purpose:** Define test strategies by service type, mandatory test documentation,
> and handoff procedures for subsequent agent sessions.
>
> **Scope:** All AAOF projects. Referenced at STEP 5 (Validation & Test) of the
> agent workflow.

---

## 1. Core Testing Principle

Every deployed service MUST be tested according to its type. The agent does not
choose what to test — this file defines it. Tests are mandatory, not optional.

---

## 1.1 Test-Driven Development (TDD) Iron Law

> **Inspired by:** [obra/superpowers](https://github.com/obra/superpowers) — `skills/test-driven-development`

### The Rule

```
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
```

For every new function, method, endpoint, or behavior the agent implements during STEP 4,
the agent MUST follow the **RED-GREEN-REFACTOR** cycle:

1. **RED:** Write a minimal test that describes the expected behavior. Run it. It MUST fail.
2. **GREEN:** Write the minimum code necessary to make the test pass. Run it. It MUST pass.
3. **REFACTOR:** Clean up the code (remove duplication, improve names). Tests MUST stay green.
4. **COMMIT:** Commit the test + implementation together.

### When to Apply TDD

**Always:**
- New features (new endpoints, new functions, new services)
- Bug fixes (write a test that reproduces the bug FIRST)
- Behavior changes (test the new behavior before changing code)

**Exceptions (require explicit user approval):**
- Pure configuration files (docker-compose.yml, .env, nginx.conf)
- Generated/scaffolded code (boilerplate from frameworks)
- Infrastructure-only changes (Dockerfile, K8s manifests with no logic)

### The Delete Rule

If the agent writes implementation code BEFORE writing the corresponding test:
- **Delete the implementation code.**
- Start over with the TDD cycle.
- Do NOT keep the code "as reference." Do NOT "adapt" it.

### Anti-Rationalization Table

| Excuse the agent might use | Reality |
|---|---|
| "Too simple to test" | Simple code breaks. A test takes 30 seconds to write. |
| "I'll write tests at STEP 5" | Tests written after code pass immediately — that proves nothing. |
| "Tests after achieve the same goal" | Tests-after verify "what does this do?" — Tests-first verify "what SHOULD this do?" |
| "TDD will slow me down" | TDD is faster than debugging. The time saved in STEP 5/6 is massive. |
| "I need to explore first" | Fine — but throw away the exploration and start fresh with TDD. |
| "The test is hard to write" | Hard to test = hard to use. Simplify the design. |
| "Just this once" | No exceptions without explicit user permission. |

### Red Flags — STOP and Start Over

If the agent catches itself in any of these situations, it MUST stop and restart with TDD:
- Code written before test
- Test passes immediately on first run (testing existing behavior, not new)
- Can't explain why the test failed
- Using words like "should work", "probably passes", "looks correct"

---

## 2. Test Strategy by Service Type

### 2.1 Web Services (Frontend / API Gateway / Web App)

For any service that exposes HTTP endpoints to users or other systems:

| Test Category       | What to Test                                      | How                                    |
|---------------------|---------------------------------------------------|----------------------------------------|
| Endpoint Coverage   | ALL exposed URLs/routes                           | HTTP request to each route             |
| HTTP Methods        | GET, POST, PUT, DELETE per endpoint               | `curl` / `httpie` inside dev container |
| Status Codes        | 200, 201, 400, 401, 403, 404, 500                | Assert expected status per scenario    |
| Response Format     | JSON schema / HTML structure                      | Validate response body structure       |
| Auth Endpoints      | Login, logout, token refresh, protected routes    | Test with and without valid credentials|
| CORS                | Cross-origin headers (if applicable)              | Request from different origin          |
| Health Endpoint     | `/health` or `/healthz`                           | Must return 200 + service status       |
| SSL/TLS             | Certificate validity (if applicable)              | `curl --insecure` vs `curl`            |

### 2.2 Backend Services (Business Logic / Microservices / APIs)

For any service that exposes methods or APIs consumed by other services:

| Test Category       | What to Test                                      | How                                    |
|---------------------|---------------------------------------------------|----------------------------------------|
| API Coverage        | ALL public methods/functions exposed via API      | Call each endpoint with valid data     |
| Input Validation    | Invalid/missing parameters per method             | Send malformed requests                |
| Error Handling      | Each error category returns proper response       | Trigger known error conditions         |
| Integration         | Service-to-service communication                  | Test inter-service calls               |
| Idempotency         | Repeated calls produce same result (where needed) | Call same endpoint twice               |
| Rate Limits         | If configured, verify enforcement                 | Rapid successive calls                 |

### 2.3 Database Services

For any database service (relational, NoSQL, or other):

| Test Category       | What to Test                                      | How                                    |
|---------------------|---------------------------------------------------|----------------------------------------|
| Connectivity        | DB accepts connections from app services          | Connection test from app container     |
| Users and Roles     | ALL DB users exist with correct permissions       | List users, verify grants              |
| Tables/Schema       | ALL tables exist with correct schema              | Query `information_schema` / `DESCRIBE`|
| CRUD Operations     | Insert, Select, Update, Delete on each table      | Execute sample queries                 |
| Indexes             | Critical indexes exist                            | Query index metadata                   |
| Data Integrity      | FK constraints, NOT NULL, UNIQUE enforced         | Insert violating data → expect error   |
| Backup/Restore      | DB dump and restore works                         | `pg_dump`/`mongodump` → restore → verify |
| Volume Persistence  | Data survives container restart                   | Insert → restart container → select    |

---

## 3. Mandatory Output Artifacts

The agent MUST produce **two documents** at the end of STEP 5:

### 3.1 Test Results Report: `output/test_results.md`

This document records the outcome of the current test run.

**Required sections:**
- **Header:** Date/time, agent session ID, project name, environment
- **Environment Details:** Container versions, relevant config values
- **Results Table:** One row per test with columns:
  | Test ID | Category | Service | Input/Action | Expected Result | Actual Result | Status |
- **Summary:** Total tests, passed, failed, warnings
- **Failure Details:** For each failed test: full error output, logs, and suggested fix

### 3.2 Test Playbook: `output/test_playbook.md`

This document enables a **human or a subsequent agent** to reproduce ALL tests independently.

**Required sections:**
- **Prerequisites:** Which services must be running, required data state
- **Test Catalog:** For each test:
  1. **Test ID and Name**
  2. **Pre-conditions** (services running, seed data loaded, etc.)
  3. **Exact command** to execute (full `curl` command, SQL query, etc.)
  4. **Expected output** (HTTP status code, response body snippet, row count, etc.)
  5. **How to interpret** the result (what constitutes PASS vs FAIL)
- **Manual Testing Quick Start:** Step-by-step guide to run ALL tests in sequence manually
- **Automated Testing:** If test scripts are generated, how to run them (e.g., `make test` or `docker-compose exec app npm test`)

---

## 4. Test Execution Rules

1. ALL tests MUST run **inside Docker containers** (consistent with `rules/docker_rules.md`).
2. Tests MUST be **repeatable**: no dependency on external state not controlled by the project.
3. The test playbook (`test_playbook.md`) MUST be written **before** test execution.
4. The agent at STEP 5 MUST follow this sequence:
   1. Generate `output/test_playbook.md`
   2. Execute all tests defined in the playbook
   3. Generate `output/test_results.md`
   4. Set `VAR_VALIDATION_RESULT` based on results (PASS only if ALL tests pass)

---

## 5. Handoff to Next Agent Session

When a new agent session starts (STEP 0), if `output/test_playbook.md` exists:

1. The agent MUST read it during bootstrap.
2. The existing test playbook becomes the **regression test baseline**.
3. Any new functionality implemented MUST add new tests to the playbook.
4. The non-regression check (STEP 5.0) MUST also verify that ALL tests
   in the existing playbook still PASS (in addition to the manifest checks).
5. The updated `test_playbook.md` and `test_results.md` replace the previous versions.

---

## 6. Verification Before Completion

> **Inspired by:** [obra/superpowers](https://github.com/obra/superpowers) — `skills/verification-before-completion`

### The Rule

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

Before the agent declares ANY of the following, it MUST have run the verification
command **in that same interaction** and read the full output:

| Claim | Requires | NOT Sufficient |
|---|---|---|
| "Tests pass" | Test command output showing 0 failures | Previous run, "should pass" |
| "Build succeeds" | Build command output with exit 0 | "Linter passed" |
| "Service is healthy" | Health check response (HTTP 200, port open) | "Container started" |
| "Bug is fixed" | Failing test now passes + no regressions | "Code changed, assumed fixed" |
| "All tests still pass" | Full test suite output, 0 failures | Partial test run |

### Prohibited Language

The agent MUST NOT use any of the following before verification:
- "Should work now"
- "Probably passes"
- "Looks correct"
- "I'm confident this fixes it"
- "Great!", "Perfect!", "Done!" (before running verification)

### The Gate Function

```
BEFORE claiming any status:
1. IDENTIFY: What command proves this claim?
2. RUN: Execute the FULL command (not partial)
3. READ: Full output, check exit code
4. VERIFY: Does output confirm the claim?
   - If NO → State actual status with evidence
   - If YES → State claim WITH evidence
5. ONLY THEN: Make the claim
```
