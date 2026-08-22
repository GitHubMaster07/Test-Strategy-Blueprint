# CI/CD Pipeline & Quality Gates

This document defines the CI/CD pipeline architecture, execution flow, quality gates, parallelization strategy, artifact management, and failure handling model for automated testing.

The goal is to ensure deterministic, fast, scalable, and audit-ready test execution while producing evidence that supports risk-based release decisions.

---

## 1. CI/CD Overview

### 1.1 Purpose
- Automate build, test, and deployment workflows.
- Provide fast feedback to developers.
- Enforce quality gates before merging or releasing.
- Ensure consistent, reproducible test execution.
- Generate audit‑ready evidence for compliance.

### 1.2 Supported CI/CD Platforms
- GitHub Actions
- Jenkins (optional)
- Local execution via Maven

### 1.3 High-Level Pipeline Diagram
```
+------------------+
|   Pull Request   |
+--------+---------+
|
v
+------------------+
|   Static Checks  |
| Lint, SAST, etc. |
+--------+---------+
|
v
+------------------+
|     Unit Tests   |
+--------+---------+
|
v
+------------------------------+
|  API + Contract + Schema     |
|        Validation            |
+--------------+---------------+
|
v
+------------------------------+
|     Integration Tests        |
| (API → DB, WireMock, Events) |
+--------------+---------------+
|
v
+------------------------------+
|         UI Smoke Tests       |
+--------------+---------------+
|
v
+------------------------------+
|     Full Regression (Nightly)|
+--------------+---------------+
|
v
+------------------------------+
|      Reporting & Artifacts   |
+------------------------------+
```

---

## 2. Pipeline Stages & Execution Order

### 2.1 Stage 1 — Static Analysis
- Linting (Checkstyle, SpotBugs)
- SAST (GitHub CodeQL)
- Dependency vulnerability scan

### 2.2 Stage 2 — Unit Tests
- Fast execution (< 1 min)
- Must pass 100%

### 2.3 Stage 3 — API Tests
- RestAssured tests
- Schema validation
- Authentication/authorization checks

### 2.4 Stage 4 — Contract Tests
- Pact consumer tests
- Publish contracts to Pact Broker
- Provider verification (separate pipeline)

### 2.5 Stage 5 — Integration Tests
- API → DB validation
- WireMock virtualization
- Event emission validation (Kafka/SQS)

### 2.6 Stage 6 — UI Smoke Tests
- Critical flows only
- Parallel execution

### 2.7 Stage 7 — Full Regression (Nightly)
- UI + API + DB + Events
- Parallel matrix execution
- Allure reporting

### 2.8 Stage 8 — Reporting & Artifacts
- Allure report
- Logs
- Screenshots
- Payloads
- Videos (optional)

---

## 3. Parallelization Strategy

### 3.1 TestNG Parallel Execution
- Parallel by classes
- Parallel by methods
- ThreadLocal isolation for:
  - WebDriver
  - API clients
  - DB connections

### 3.2 CI/CD Matrix Builds
- Browser matrix (Chrome, Firefox)
- Environment matrix (QA, Staging)
- Sharded test suites (split by tags)

### 3.3 Distributed Execution
- Selenium Grid / Selenoid
- Parallel API tests (stateless)
- Parallel event tests (isolated topics)

### 3.4 Performance Goals
- Smoke suite: < 10 minutes
- Full regression: < 30 minutes
- Contract tests: < 2 minutes

---

## 4. Quality Gates & Thresholds

### 4.1 Pre-Merge Quality Gates
| Gate | Requirement |
|------|-------------|
| Unit Tests | 100% pass |
| API Tests | 100% pass |
| Contract Tests | 100% pass |
| Static Analysis | No critical issues |
| Linting | No violations |
| Code Coverage | ≥ 80% |
| Secrets Scan | No exposed secrets |

### 4.2 Pre-Release Quality Gates
| Gate | Requirement |
|------|-------------|
| Regression Suite | ≥ 95% pass |
| Flaky Tests | ≤ 1% |
| Performance Baseline | Within SLA |
| Security Scan | No critical vulnerabilities |
| Contract Verification | All providers verified |
| Schema Validation | No drift detected |

### 4.3 Production Deployment Decision Inputs
- Critical business flows have acceptable evidence
- No unaccepted critical defects or security findings
- Contract/schema compatibility is confirmed
- Known test failures are classified; flaky/infra failures do not hide product risk
- Performance is within agreed SLO/SLA tolerance where applicable
- Required evidence is archived
- Observability and rollback/recovery controls are ready

> A green pipeline is necessary evidence, but it is not sufficient by itself to approve a high-risk release. Use the [Release Readiness Framework](./Release-Readiness.md) to classify the recommendation as **GO**, **CONDITIONAL GO**, or **NO-GO**.

---

## 5. Reporting & Artifacts

### 5.1 Allure Reporting
- Steps
- Attachments
- Screenshots
- API payloads
- Event payloads
- History trends
- Categories (flaky, product bugs, infra issues)

### 5.2 CI/CD Artifacts
- Allure results
- Logs (UI, API, DB, events)
- Screenshots
- Videos (optional)
- Pact contracts
- JSON schemas
- WireMock stubs

### 5.3 Artifact Retention Policy
- PR runs: 7 days
- Nightly runs: 30 days
- Release runs: 12 months (audit requirement)

---

## 6. Failure Handling & Reruns

### 6.1 Automatic Reruns
- RetryAnalyzer for known flaky categories
- Max retries: 1
- Only for:
  - UI synchronization issues
  - Network timeouts
  - Transient environment failures

### 6.2 Failure Categorization
- Product defect
- Test defect
- Environment issue
- Flaky test

### 6.3 Failure Triage Workflow
1. Analyze logs, screenshots, payloads
2. Categorize failure
3. Assign owner (QA, Dev, DevOps)
4. Create defect (if applicable)
5. Add evidence to Allure

### 6.4 CI/CD Failure Rules
- Pipeline fails immediately on:
  - Contract test failure
  - Schema drift
  - Security vulnerability
  - Secrets exposure

### 6.5 Notifications
- Slack / Teams alerts
- Email summary (optional)
- GitHub PR status checks



---

## 7. Risk-Based Pipeline Selection

Not every change requires every suite. Pipeline depth should reflect change risk.

| Change Type | Minimum Recommended Evidence |
|---|---|
| Documentation / non-runtime change | Static checks |
| Isolated low-risk service change | Unit + relevant API/component tests |
| API/schema change | Unit + API + schema + contract + integration |
| Critical user-flow change | Unit + API/integration + targeted UI smoke/E2E |
| Data migration/transformation | API/integration + DB reconciliation + rollback validation |
| Auth/security-sensitive change | API + authorization/security checks + critical UI flow |
| High-risk release | Full risk-selected regression + NFR evidence + release-readiness review |

This approach avoids running expensive suites without purpose while ensuring high-risk changes receive deeper evidence.

---

## 8. Quality Gate Exception Policy

A gate may be overridden only when the organization explicitly accepts the residual risk. The exception record should include:
- Failed/waived gate
- Root cause or known limitation
- Business impact
- Mitigation and monitoring
- Named approver / risk owner
- Expiration or follow-up date

Exceptions must not become a mechanism for normalizing flaky tests or bypassing unresolved critical risk.
