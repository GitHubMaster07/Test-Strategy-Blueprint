# Automation Decision Framework

## 1. Purpose

This framework helps teams decide **what to automate, what not to automate, and at which layer automation provides the best return**.

Automation is an engineering investment. A test should not be automated simply because it is repeatable.

---

## 2. Decision Criteria

Score each automation candidate from 1 to 5.

| Factor | Weight | High Score Means |
|---|---:|---|
| Business / quality risk | 30% | Failure has significant customer, financial, data, security, or regulatory impact |
| Execution frequency | 20% | Test runs frequently across PRs, regression, releases, or environments |
| Manual effort | 15% | Manual execution is slow, repetitive, or error-prone |
| Interface stability | 15% | Behavior and interface are stable enough to maintain automation economically |
| Detection / feedback value | 10% | Failure gives early, actionable information |
| Maintainability | 10% | Test can be isolated, deterministic, observable, and inexpensive to maintain |

### Example Formula

`Automation Value = sum(score/5 x weight)`

Suggested interpretation:
- **>= 75:** Automate
- **50-74:** Evaluate / automate selectively
- **< 50:** Prefer manual, exploratory, or lower-layer validation

Thresholds are guidance, not policy.

---

## 3. What to Automate First

Prioritize:
- Critical business journeys
- API/service business rules
- Authentication and authorization controls
- Contracts and schemas
- High-frequency regression scenarios
- Data integrity and reconciliation checks
- Deterministic negative/boundary scenarios
- Deployment smoke checks
- Repetitive cross-browser checks where business value exists

---

## 4. What Not to Automate by Default

Prefer manual/exploratory approaches for:
- One-time validations
- Rapidly changing prototypes
- Subjective usability/aesthetic evaluation
- Discovery-oriented exploratory testing
- Very low-frequency scenarios with high setup/maintenance cost
- Tests dependent on uncontrolled third parties when virtualization is unavailable
- Scenarios whose automation produces more noise than actionable evidence

A decision not to automate should be intentional, not accidental.

---

## 5. Choose the Lowest Valuable Layer

Before creating a UI test, ask:
1. Can the rule be verified in a unit/component test?
2. Can an API test provide equivalent confidence faster?
3. Is a contract test better for compatibility risk?
4. Is DB/event validation required for data integrity?
5. Does the behavior truly require the browser/user interface?

### Example
| Scenario | Preferred Layer | Reason |
|---|---|---|
| Required API field validation | API | Fast, deterministic, precise failure |
| Provider response compatibility | Contract | Detects breaking change before integration |
| Pricing persisted correctly | API + DB | Validates behavior and data integrity |
| Login page accessibility | UI + accessibility engine | Browser semantics required |
| End-to-end purchase journey | Small E2E set | Verifies critical system orchestration |

---

## 6. Automation Lifecycle

Every automated test has ongoing cost. Periodically classify tests as:
- **Keep** — continues to protect meaningful risk
- **Refactor** — valuable but unreliable/expensive
- **Move Down** — better validated at API/component/contract layer
- **Quarantine Temporarily** — diagnosed with owner and expiration date
- **Delete** — no longer protects relevant risk

Automation deletion is a quality improvement when a test no longer provides useful evidence.

---

## 7. Definition of a Good Automated Test

A good test is:
- Business-relevant
- Deterministic
- Independent
- Fast enough for its pipeline stage
- Diagnosable on failure
- Maintainable
- Secure with test data
- Located at the right layer
- Traceable to a risk, requirement, or quality objective

---

## 8. Anti-Patterns

Avoid:
- “Automate 100% of regression” as a goal
- Duplicating the same assertion at every layer
- Large UI suites for API/business-rule coverage
- Retrying failures until the pipeline turns green
- Keeping obsolete tests to preserve automation-count metrics
- Automating unstable features before requirements settle

---

## 9. Decision Principle

**The best automation portfolio is not the largest one. It is the smallest maintainable set that provides fast, trustworthy evidence for the most important risks.**
