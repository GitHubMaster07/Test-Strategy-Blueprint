# Test Layers and Coverage Strategy

## 1. Purpose

This document defines how functional risks are distributed across unit, API, contract, integration, database, event, UI, and end-to-end testing.

The objective is not to maximize coverage at every layer. The objective is to place each risk at the **cheapest reliable layer that can detect it with sufficient confidence**, while using higher-cost tests only where they provide unique evidence.

---

## 2. Layer Selection Principles

### 2.1 Cheapest Reliable Layer

A test belongs at the lowest layer that can validate the intended risk without creating false confidence.

Examples:

| Risk | Preferred Layer | Why |
|---|---|---|
| calculation/business rule | Unit/API | fast and deterministic |
| API schema compatibility | Contract/API | catches breaking changes early |
| persistence/reconciliation | API + DB | validates behavior and stored state |
| service-to-service behavior | Integration/Event | validates real interaction |
| critical customer journey | UI/E2E | confirms system works from user perspective |
| visual/usability issue | UI/manual/accessibility | cannot be proven by backend tests |

### 2.2 Avoid Duplicate Assertions

The same business rule should not be repeated across every layer unless each test protects a different failure mode.

### 2.3 Higher Layers Require Stronger Justification

As tests move upward toward E2E, they become slower, more environment-dependent, and more expensive to maintain. The required business-risk justification therefore increases.

---

## 3. Coverage Model

The blueprint uses a **risk-adjusted test pyramid**, not fixed percentages.

Typical direction:

```text
              E2E / UI
          limited critical journeys
                 /\
                /  \
       Integration / Events
              /      \
         API / Contracts
            /          \
           Unit / Component
```

Percentages such as “10% UI” or “5% E2E” may be useful heuristics, but are not governance targets. A system with complex UI behavior may require more UI coverage; an API platform may require almost none.

---

## 4. Unit and Component Testing

### Purpose
Validate isolated logic and component behavior with minimal infrastructure.

### Best suited for
- calculations,
- validation rules,
- transformations,
- state transitions,
- error handling,
- domain logic.

### Quality expectations
- very fast execution,
- deterministic behavior,
- no external network dependency,
- critical logic covered by meaningful assertions.

### Architect decision
Do not use line coverage alone as a quality metric. Coverage percentage identifies unexecuted code; it does not prove correct behavior.

### Anti-patterns
- mocking so heavily that production behavior is no longer represented,
- testing implementation details instead of observable behavior,
- treating a coverage threshold as sufficient release evidence.

---

## 5. API Testing

### Purpose
Validate service behavior, business rules, security boundaries, and response contracts without UI cost.

### Best suited for
- positive/negative business flows,
- authentication and authorization,
- validation/error handling,
- idempotency,
- pagination/filtering,
- data contracts,
- service-level regression.

### Coverage expectations
Critical endpoints receive deeper risk-based coverage, including edge cases and failure behavior. Lower-risk endpoints may receive representative rather than exhaustive combinations.

### Architect decision
Prefer API automation over UI automation when both provide equivalent business confidence.

### Anti-patterns
- validating only status code `200`,
- duplicating the entire API regression through UI,
- hardcoded tokens/environment values,
- treating schema validation as a substitute for business assertions.

---

## 6. Contract Testing

### Purpose
Detect incompatible changes between consumers and providers before full integration.

### Best suited for
- request/response structure,
- required fields and types,
- provider expectations,
- backward compatibility.

### Boundary
Contract tests answer **“can these services still communicate as agreed?”** They do not answer whether the entire business workflow works.

### Anti-patterns
- using contracts as full API functional tests,
- publishing contracts without provider verification,
- maintaining contracts that no real consumer uses.

---

## 7. Database and Data Validation

### Purpose
Validate persistence, integrity, transformations, and data movement when stored data is itself a business risk.

### Best suited for
- persistence checks,
- source-to-target reconciliation,
- referential integrity,
- ETL transformations,
- transactional outcomes,
- audit/compliance data.

### Decision rule
Validate the database only when backend state provides evidence not already available from supported interfaces.

### Anti-patterns
- direct DB writes that bypass application behavior,
- asserting internal implementation details that may legitimately change,
- using SQL as the only proof of an end-user workflow.

---

## 8. Integration Testing

### Purpose
Validate the real behavior of connected components and services.

### Best suited for
- API → database behavior,
- service → service communication,
- external dependency integrations,
- transaction boundaries,
- error propagation,
- authentication between services.

### Virtualization decision
Use a real dependency when compatibility/integration is the risk. Use virtualization when deterministic behavior, fault injection, or dependency instability is the greater concern.

### Anti-patterns
- calling every external dependency in every CI test,
- hiding integration behavior behind mocks only,
- treating an integration test as an E2E test merely because multiple components are involved.

---

## 9. Event-Driven Testing

### Purpose
Validate asynchronous business behavior and messaging guarantees.

### Best suited for
- event schema,
- publish/consume behavior,
- ordering,
- idempotency,
- retries,
- DLQ routing,
- eventual consistency,
- correlation across services.

### Architecture considerations
Timing assertions should reflect the system’s SLOs rather than arbitrary sleeps. Polling must be bounded and diagnostics must retain timestamps, keys, correlation IDs, and consumed payloads.

### Anti-patterns
- fixed sleeps,
- hardcoded offsets,
- ignoring duplicate delivery,
- validating only event existence without validating meaning.

---

## 10. UI Testing

### Purpose
Validate behavior that is unique to the user interface and confirm selected critical user journeys.

### Best suited for
- navigation and interaction,
- client-side validation,
- browser-specific behavior,
- accessibility hooks,
- critical user workflows,
- UI-only business risk.

### Automation decision
A UI test is justified when lower layers cannot provide equivalent evidence or when user-level integration confidence is specifically required.

### Anti-patterns
- reproducing every API scenario in UI,
- unstable locator strategies,
- assertions hidden in generic page utilities,
- long multi-purpose UI tests,
- using retries to tolerate unstable synchronization.

---

## 11. End-to-End Testing

### Purpose
Provide limited evidence that a high-value business capability works across the deployed system.

### Selection criteria
An E2E test should normally satisfy several of these conditions:

- critical revenue/compliance/customer impact,
- multiple services must cooperate,
- production incidents would be severe,
- lower-layer tests cannot prove orchestration,
- the workflow is stable enough to automate economically.

### E2E budget
E2E tests are treated as a **scarce quality budget**, not the default regression layer.

### Anti-patterns
- using E2E as the main regression suite,
- testing every permutation through UI,
- long chains that fail for unrelated environmental reasons,
- accepting high flakiness because the workflow is “important.”

---

## 12. Cross-Layer Coverage Matrix

A traceability view should show which layers protect each risk.

Example:

| Business Capability / Risk | Unit | API | Contract | DB | Integration/Event | UI/E2E |
|---|---:|---:|---:|---:|---:|---:|
| Authentication rules | ✓ | ✓ |  |  | ✓ | smoke |
| Profile persistence | ✓ | ✓ |  | ✓ |  | critical flow |
| Provider compatibility |  | ✓ | ✓ |  | ✓ |  |
| Payment/event processing | ✓ | ✓ | ✓ | ✓ | ✓ | critical journey |
| Accessibility |  |  |  |  |  | ✓ |

Blank cells are intentional when another layer provides sufficient evidence.

---

## 13. Coverage Gap Analysis

A coverage gap exists when:

- a critical risk has no owning test layer,
- only a high-cost E2E test detects a defect that could be detected earlier,
- a contract/integration boundary is unverified,
- production incidents reveal untested failure modes,
- automation exists but does not provide diagnostic evidence.

Coverage reviews should therefore ask **“what risk is unprotected?”**, not only “what percentage is automated?”

---

## 14. Duplication Review

Duplicate tests may be retained when they deliberately provide different evidence, such as:

- API test proves business rule,
- contract test proves compatibility,
- E2E test proves orchestration.

Duplicate tests should be removed when they repeat the same assertion with higher maintenance cost and no additional risk reduction.

---

## 15. Change-Based Test Selection

Not every change requires every layer.

Examples:

| Change | Minimum Expected Validation |
|---|---|
| isolated business rule | unit + targeted API |
| API schema change | API + contract + affected integration |
| DB transformation | API/data + DB reconciliation |
| UI-only styling | component/UI + accessibility/visual as relevant |
| cross-service workflow | contract + integration/event + critical E2E |
| authentication change | unit/API + security + targeted E2E |

CI/CD can use this mapping to optimize feedback while nightly/release suites provide broader assurance.

---

## 16. Layer Ownership

Developers, QA/SDETs, and platform teams share responsibility for coverage.

- Developers primarily own unit/component quality and testability.
- Product/QE teams own business-risk coverage across API, data, integration, and UI.
- Architecture/platform teams help define contract, environment, and observability standards.
- No layer is “QA-only” if another engineering role is best positioned to prevent the defect earlier.

---

## 17. Review Questions for Test Architects

During design or test-plan review, ask:

1. What failure are we trying to detect?
2. What is the cheapest reliable layer for that failure?
3. Are we duplicating evidence already provided elsewhere?
4. What happens when a dependency fails or returns invalid data?
5. Can the test run deterministically in CI?
6. What evidence will be available when it fails?
7. Is this automation still worth maintaining six months from now?

These questions keep layer selection tied to risk and business value rather than tooling preference.
