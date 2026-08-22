# Release Readiness & Go / No-Go Framework

## 1. Purpose

Release readiness converts test results, defects, operational evidence, and residual risk into a clear recommendation.

QA does not approve a release solely because a regression percentage is green. The release decision should consider **business criticality, known gaps, production safeguards, rollback capability, and explicit risk ownership**.

---

## 2. Evidence Categories

A release review should consider:
- Critical business-flow results
- API/contract/schema compatibility
- Integration and data-integrity evidence
- Open defects by severity and business impact
- Security findings
- Performance/SLO evidence
- Flaky/blocked tests and untested scope
- Environment/configuration readiness
- Observability and alerting readiness
- Rollback / recovery readiness
- Known operational dependencies

---

## 3. Decision States

### GO
Use when:
- Critical journeys pass
- No unaccepted critical defects exist
- Compatibility/security/performance evidence is within agreed tolerance
- Known gaps are low risk
- Monitoring and rollback are appropriate for the change

### CONDITIONAL GO
Use when a release has known residual risk but the business chooses to proceed with explicit controls.

Required documentation:
- Risk description
- Business impact
- Why release is still justified
- Mitigation/workaround
- Monitoring signal
- Named risk owner
- Rollback/containment trigger
- Expiration/follow-up date

### NO-GO
Typical triggers:
- Critical customer journey fails
- Data corruption/integrity risk is unresolved
- Authentication/authorization/security control is broken
- Breaking contract/schema change is unapproved
- Critical performance degradation exceeds tolerance
- Release cannot be safely rolled back/recovered
- Test evidence is insufficient for a high-risk change

---

## 4. Example Release Scorecard

| Evidence Area | Status | Notes |
|---|---|---|
| Critical Journeys | PASS | 12/12 protected flows green |
| API / Contracts | PASS | No breaking changes |
| Regression | WARN | 97.2%; 3 known non-critical failures classified |
| Defects | WARN | 1 Sev-3 with workaround |
| Performance | PASS | p95 within SLO |
| Security | PASS | No critical/high unaccepted findings |
| Data Integrity | PASS | Reconciliation complete |
| Rollback | PASS | Validated in staging |
| Observability | PASS | Dashboards/alerts confirmed |

**Recommendation: CONDITIONAL GO**

**Residual Risk:** Known Sev-3 display issue affects a non-critical admin workflow.

**Mitigation:** Workaround documented; fix scheduled for next patch.

**Risk Owner:** Product Owner.

---

## 5. Release Decision Record

For material releases, capture:

```text
Release:
Date:
Change Scope:
Overall Risk: Low / Medium / High / Critical

Critical Evidence:
- ...

Known Defects / Gaps:
- ...

Residual Risk:
- ...

Mitigations / Monitoring:
- ...

Rollback Trigger:
- ...

QA Recommendation: GO / CONDITIONAL GO / NO-GO
Product Decision:
Engineering Decision:
Risk Owner(s):
```

---

## 6. Important Distinction: Gate vs Decision

A **quality gate** is an automated or procedural condition.

A **release decision** evaluates the complete evidence and risk context.

Examples:
- A 95% regression pass rate does not automatically mean GO.
- A 94% pass rate does not automatically mean NO-GO if failures are understood, low-risk, and explicitly accepted.
- A 100% pass rate can still be NO-GO if critical scope was not tested or observability/rollback is missing.

---

## 7. Decision Principle

**QA provides evidence and a risk recommendation. Accountable business and engineering stakeholders own the decision to accept residual risk.**
