# Framework Architecture

## 1. Purpose

This document defines the architecture principles, boundaries, and decision rules for an enterprise test automation platform that validates UI, API, database, event-driven, contract, and integration risks.

The framework is not treated as a collection of test utilities. It is a **quality engineering platform** designed to provide fast feedback, reliable evidence, maintainable automation, and reusable capabilities across multiple teams.

---

## 2. Architecture Goals

The architecture optimizes for:

- **Fast feedback** at the cheapest reliable test layer.
- **Maintainability** through clear boundaries and reusable abstractions.
- **Deterministic execution** with isolated state and controlled data.
- **Observability** so failures can be diagnosed without rerunning tests manually.
- **Scalability** across services, teams, environments, and CI/CD pipelines.
- **Governance** through common standards without blocking team autonomy.
- **Testability** as a product architecture concern, not only a QA concern.

The architecture does **not** optimize for maximum test count, maximum abstraction, or automation percentage for its own sake.

---

## 3. Architecture Decision Principles

### 3.1 Test at the Lowest Reliable Layer

Validate a behavior at the lowest layer that can provide trustworthy evidence.

Examples:

- Business rule → unit/API layer.
- Service compatibility → contract layer.
- Persistence rule → API + DB validation.
- Critical customer journey → limited E2E/UI validation.

Duplicating the same assertion across UI, API, and DB is avoided unless each layer protects a different risk.

### 3.2 Separate Domain Intent from Tooling

Business intent must not depend directly on Selenium, RestAssured, JDBC, Pact, or Kafka client APIs.

Tests should express intent through domain-focused clients, page objects, validators, and builders. This reduces migration cost when tools or implementations change.

### 3.3 Keep Orchestration Thin

Cucumber step definitions, test methods, and runners orchestrate behavior. They do not contain business logic, SQL construction, HTTP plumbing, or complex retry logic.

### 3.4 Prefer Explicit Boundaries over Generic Abstractions

A small amount of duplication is preferable to an abstraction that hides domain behavior or creates coupling between unrelated services.

A new abstraction is justified when it:

1. removes repeated infrastructure behavior,
2. improves diagnostics or consistency, and
3. can be understood without tracing multiple inheritance or utility layers.

### 3.5 Design for Failure Diagnosis

Every automated test should make failure investigation cheaper. Diagnostic evidence should be captured automatically where practical:

- request/response details,
- screenshots,
- correlation IDs,
- event payloads,
- SQL/query evidence,
- environment metadata,
- retry/flakiness classification.

---

## 4. Architecture Boundaries

| Layer | Owns | Must Not Own |
|---|---|---|
| UI/Page layer | user interactions, locators, UI state | API calls, DB queries, release logic |
| API client layer | HTTP operations, serialization, service endpoints | test assertions unrelated to protocol/domain |
| DB validation layer | read-only queries, data assertions, reconciliation | test setup that bypasses business workflows |
| Event layer | publish/consume, schema, ordering, idempotency checks | hidden business orchestration |
| Contract layer | consumer/provider compatibility | full integration behavior |
| Test orchestration | scenario intent and cross-layer sequence | low-level framework implementation |
| CI/CD layer | suite selection, gates, artifacts, execution policy | product business logic |

Cross-layer interaction is allowed when the scenario requires it, but implementation details remain inside their owning layer.

---

## 5. Reference Architecture

```text
                         +-----------------------------+
                         |       CI/CD & Governance     |
                         | Gates • Evidence • Policies  |
                         +---------------+-------------+
                                         |
                 +-----------------------+-----------------------+
                 |                                               |
                 v                                               v
       +----------------------+                      +----------------------+
       | Test Orchestration   |                      | Observability Hooks  |
       | TestNG / Cucumber    |                      | Logs / Traces / Data |
       +----------+-----------+                      +----------+-----------+
                  |                                                 |
      +-----------+------------+------------+------------+-----------+
      |                        |            |            |           |
      v                        v            v            v           v
+-----------+            +-----------+ +---------+ +---------+ +-----------+
| UI Layer  |            | API Layer | | DB      | | Events  | | Contracts |
| Selenium  |            |RestAssured| | JDBC    | |Kafka/SQS| | Pact      |
+-----------+            +-----------+ +---------+ +---------+ +-----------+
      |                        |            |            |           |
      +------------------------+------------+------------+-----------+
                               |
                               v
                    +-----------------------+
                    | System Under Test     |
                    | + Virtualized Deps    |
                    +-----------------------+
```

WireMock and other virtualization tools sit at dependency boundaries and are used when control, repeatability, fault injection, or isolation provides more value than using a live dependency.

---

## 6. Core Components

### 6.1 UI Layer

**Responsibilities**
- Page/component interactions.
- Stable locator strategy.
- Synchronization and browser lifecycle.
- Failure screenshots and browser diagnostics.

**Design decisions**
- Page Objects expose user-oriented actions rather than raw locators.
- Assertions remain in test/validation layers unless a component-specific invariant clearly belongs with the component.
- `data-testid`/stable semantic selectors are preferred over brittle DOM paths.

**Do not** build generic “click anything” wrappers that hide meaningful UI intent.

### 6.2 API Layer

**Responsibilities**
- Service-specific clients.
- Request specifications and authentication.
- Serialization/deserialization.
- Schema/contract helpers.
- Request/response diagnostics.

**Design decisions**
- One client per bounded service/domain rather than one global API utility.
- Payload builders model valid defaults and explicit overrides.
- Negative tests remain readable and do not depend on hidden mutation helpers.

### 6.3 Database Layer

**Responsibilities**
- Parameterized read-only queries.
- Source-to-target and persistence validation.
- Reusable data validators.
- Connection lifecycle.

**Decision rule**
Use DB validation when data state itself is the risk. Do not use direct DB writes to bypass behavior that should be validated through supported product interfaces.

### 6.4 Event Layer

**Responsibilities**
- Event production/consumption.
- Schema compatibility.
- Ordering, retries, idempotency, and DLQ behavior.
- Correlation across asynchronous workflows.

Tests must use bounded polling with diagnostic timeouts rather than arbitrary sleeps.

### 6.5 Contract Layer

**Responsibilities**
- Consumer/provider expectations.
- Compatibility verification.
- Breaking-change detection before integration.

Contract tests reduce—not replace—integration testing.

### 6.6 Service Virtualization

Use WireMock or equivalent when:

- dependency availability is unreliable,
- deterministic negative behavior is required,
- latency/fault scenarios must be injected,
- third-party cost or rate limits make live execution inefficient.

Do not virtualize a dependency when the integration itself is the primary risk being validated.

---

## 7. Configuration and Secrets

Configuration precedence should be explicit and predictable:

1. repository-safe defaults,
2. environment profile,
3. runtime/system properties,
4. CI/CD secret injection.

Secrets are never committed to source control or embedded in test data. Environment configuration should fail fast when required values are missing.

---

## 8. Test Data Architecture

Test data is treated as a first-class dependency.

### Principles
- Deterministic where repeatability matters.
- Unique where parallel isolation matters.
- Synthetic or masked where privacy/compliance matters.
- Created through supported APIs/services when possible.
- Cleaned up only when cleanup is safe and necessary.

### Builders and Factories
Builders provide valid defaults; factories represent domain-specific creation strategies. Random values must be seedable or logged so failures can be reproduced.

---

## 9. Parallel Execution and Isolation

Parallelism is enabled only when isolation can be guaranteed.

Required controls may include:

- isolated browser instances,
- unique test identities/data,
- independent API/session state,
- safe DB connections,
- correlation IDs for async flows,
- no mutable global state.

Increasing thread count is not considered a performance optimization if it increases flakiness or environment contention.

---

## 10. Observability by Design

A mature framework should answer **why a test failed** without requiring an immediate rerun.

Minimum evidence for failed critical tests should include relevant combinations of:

- test/environment/build identifiers,
- screenshots or video where useful,
- API request/response payloads with secrets masked,
- correlation/trace IDs,
- event payloads and timestamps,
- DB validation evidence,
- failure category and retry status.

Observability is part of framework architecture, not an afterthought added to reporting.

---

## 11. Retry and Flakiness Policy

Retries are not used to convert unreliable tests into green pipelines.

A retry may be allowed when:

- the failure category is known and transient,
- maximum retry count is controlled,
- the original failure remains visible,
- retry metrics feed a flakiness backlog.

A test that regularly requires retries is treated as a defect in the test, environment, or product dependency.

---

## 12. Extensibility Decision Framework

Before adding a new library, framework layer, or shared utility, ask:

1. What risk or capability is currently unsupported?
2. Can an existing component solve it adequately?
3. What maintenance cost does the new dependency introduce?
4. Will multiple teams genuinely reuse it?
5. How will it be observed, versioned, and supported?
6. What is the exit/migration strategy if the tool changes?

A framework should become broader only when the additional complexity produces measurable quality or delivery value.

---

## 13. Architecture Trade-Offs

| Decision | Benefit | Cost / Risk | Default Position |
|---|---|---|---|
| Heavy UI automation | user-level confidence | slow, brittle, expensive | keep limited to critical journeys |
| Generic shared utilities | less duplication | hidden coupling | prefer domain-specific abstractions |
| Live third-party dependencies | realism | instability, cost | virtualize unless integration risk is primary |
| Aggressive parallelism | faster execution | contention/flakiness | scale after isolation is proven |
| Automatic retries | pipeline resilience | masks defects | max 1 for classified transient failures |
| One framework for every team | consistency | central bottleneck | shared standards + modular ownership |

---

## 14. Framework Ownership Model

A central QA/enablement function may define guardrails, templates, and shared infrastructure, but product teams remain responsible for the quality of their automation.

**Central ownership**
- architecture standards,
- reusable infrastructure,
- CI/CD integration patterns,
- observability conventions,
- upgrade/deprecation guidance.

**Team ownership**
- domain test coverage,
- test maintenance,
- failure triage,
- data quality,
- adherence to agreed gates.

This prevents the framework team from becoming the only group capable of maintaining automation.

---

## 15. Architecture Anti-Patterns

Avoid:

- one giant `BaseTest` with unrelated responsibilities,
- utility classes that hide domain intent,
- assertions inside low-level transport helpers,
- SQL embedded throughout test methods,
- shared mutable test data,
- sleeps used as synchronization,
- broad retries with no failure classification,
- duplicate assertions across every layer,
- tool adoption without a defined problem,
- a framework that only its original author can understand.

---

## 16. Architecture Review Triggers

Revisit the architecture when any of the following occurs:

- flakiness exceeds agreed thresholds,
- feedback time threatens delivery cadence,
- multiple teams create competing abstractions,
- a new architecture style is introduced (events, streaming, mobile, AI/ML),
- test maintenance cost grows faster than product change,
- incidents expose observability or coverage gaps,
- CI/CD constraints require different execution strategies.

Architecture is therefore governed as an evolving capability, not a one-time framework setup.

---

## 17. Example Project Structure

```text
java-selenium-bdd-framework/
├── src/
│   ├── main/java/core/
│   │   ├── driver/
│   │   ├── config/
│   │   ├── observability/
│   │   └── api/
│   ├── test/java/
│   │   ├── pages/
│   │   ├── api/
│   │   ├── db/
│   │   ├── events/
│   │   ├── contracts/
│   │   ├── stepdefs/
│   │   ├── hooks/
│   │   └── runners/
│   └── test/resources/
│       ├── features/
│       ├── schemas/
│       ├── pact/
│       └── testdata/
├── pom.xml
├── .gitignore
└── .gitattributes
```

The structure is illustrative. The architecture principles and boundaries are more important than exact folder names.
