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
