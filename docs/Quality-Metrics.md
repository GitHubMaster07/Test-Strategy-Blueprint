# Quality Metrics & Decision Framework

## 1. Purpose

Quality metrics should help teams answer three questions:
1. Where is risk increasing?
2. Is our feedback system trustworthy and fast enough?
3. What action should we take next?

Metrics that do not influence a decision should not become management targets.

---

## 2. Recommended Metrics

### 2.1 Product / Release Quality
| Metric | Interpretation |
|---|---|
| Defect leakage | Defects that escaped pre-production controls |
| Escaped severity | Business impact of escaped defects |
| Change failure rate | Releases causing incident, rollback, or urgent remediation |
| Reopen rate | Potential weakness in fixes or verification |
| Repeated root-cause category | Systemic weakness that deserves prevention work |

### 2.2 Automation Health
| Metric | Interpretation |
|---|---|
| Flaky test rate | Trustworthiness of automated feedback |
| Time to actionable feedback | How quickly a developer gets useful evidence |
| Failure classification time | Cost of diagnosing failed automation |
| Critical-flow coverage | Protection of high-risk customer/business workflows |
| Quarantined-test age | Whether temporary exceptions are becoming permanent debt |

### 2.3 Architecture / Integration Quality
| Metric | Interpretation |
|---|---|
| Contract-breaking change count | Compatibility risk between services |
| Schema drift incidents | Data/API evolution control |
| Integration failure trend | Cross-service reliability |
| Performance SLO violations | Capacity / latency risk |

---

## 3. Metrics to Avoid as Standalone Targets

### Number of Test Cases
More tests can mean more duplication and maintenance rather than more confidence.

### Automation Percentage
A high percentage may hide weak coverage of critical risks or excessive UI automation.

### Raw Pass Rate
A 99% pass rate is meaningless if the 1% failure is a critical payment/authentication flow.

### Bugs Found per Tester
This can reward late defect discovery and discourage prevention/collaboration.

### Code Coverage Alone
Coverage shows code execution, not assertion quality or business-risk coverage.

---

## 4. Example Quality Scorecard

| Dimension | Signal | Status | Action |
|---|---|---|---|
| Critical flows | 100% pass | Green | No action |
| Flakiness | 2.4% | Red | Stabilization work required |
| Defect leakage | Rising for data workflows | Red | Root-cause review + new controls |
| Time to feedback | 14 min smoke | Amber | Parallelize / move checks lower |
| Contract health | 100% verified | Green | No action |
| Performance | p95 +18% vs baseline | Amber | Investigate before release |

---

## 5. Trend Over Snapshot

Prefer trends over single values:
- 4-week flakiness trend
- Escaped defects by component over releases
- Regression duration trend
- Contract/schema failures by service
- Defect root-cause Pareto

A stable trend is often more meaningful than whether a metric crossed a threshold once.

---

## 6. Metric Ownership

Each metric should have:
- Definition
- Data source
- Owner
- Review cadence
- Threshold/expected range (if useful)
- Required action when unhealthy

Without an owner and action, a dashboard becomes decoration.

---

## 7. Decision Principle

**Do not measure what is easy to count. Measure what helps the team change a quality decision.**
