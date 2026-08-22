# Risk-Based Testing Model

## 1. Purpose

Risk-based testing allocates test depth according to the potential business impact and likelihood of failure. It prevents teams from spending equal effort on unequal risks.

The model is intentionally lightweight so it can be used during refinement, architecture review, sprint planning, and release readiness.

---

## 2. Risk Scoring

### 2.1 Base Formula

**Risk Score = Probability x Impact**

Score Probability and Impact from 1 to 5.

### Probability
| Rating | Meaning |
|---:|---|
| 1 | Rare / highly stable area |
| 2 | Unlikely |
| 3 | Possible / moderate change history |
| 4 | Likely / complex or defect-prone |
| 5 | Very likely / new, unstable, or high-change area |

### Impact
| Rating | Meaning |
|---:|---|
| 1 | Cosmetic or negligible business effect |
| 2 | Minor inconvenience / workaround available |
| 3 | Material user or operational impact |
| 4 | Major business capability impaired |
| 5 | Critical outage, security/privacy issue, financial/data loss, or regulatory exposure |

### Risk Bands
| Score | Level | Default Response |
|---:|---|---|
| 16-25 | Critical | Release-blocking evidence; multi-layer validation |
| 10-15 | High | Strong automated regression and negative-path coverage |
| 5-9 | Medium | Focused coverage; automation based on value |
| 1-4 | Low | Exploratory/manual or lightweight checks |

---

## 3. Risk Modifiers

Raise the effective risk when a change involves:
- Authentication or authorization
- Shared libraries or common platform services
- Data migrations, ETL, transformations, or reconciliation
- Public APIs / contracts / schemas
- Payment, eligibility, pricing, entitlement, or regulated workflows
- High concurrency or high-volume paths
- Async/event-driven processing
- Third-party dependencies
- Historically unstable components
- Limited rollback capability

Lower the effective risk when:
- The change is isolated and easily reversible
- Strong lower-layer coverage already exists
- Blast radius is constrained by feature flags
- Observability and rollback are proven

---

## 4. Test Depth by Risk

### Critical
Expected evidence may include:
- Unit/component coverage
- API/service automation
- Contract/schema validation
- Integration and database checks
- Security/authorization scenarios
- Resilience/recovery checks where relevant
- Critical UI/E2E journey
- Production monitoring and rollback criteria

### High
Expected evidence may include:
- API/integration automation
- Negative and boundary cases
- Data integrity checks
- Targeted UI/E2E
- Performance or security checks when architecture warrants them

### Medium
- Focused functional coverage
- Representative negative cases
- Automation when repeated execution or regression value is meaningful

### Low
- Exploratory/manual checks
- Unit/component validation
- Automation only when execution frequency makes it economical

---

## 5. Example Risk Register

| Risk | P | I | Score | Controls / Test Response | Residual Risk Owner |
|---|---:|---:|---:|---|---|
| Authentication outage | 4 | 5 | 20 | API auth suite, UI smoke, security checks, resilience validation | Engineering/Product |
| Data corruption after update | 3 | 5 | 15 | API + DB reconciliation + negative/idempotency tests | Engineering |
| Breaking API response | 3 | 5 | 15 | Pact/schema checks in PR pipeline | Service owner |
| Third-party timeout | 3 | 4 | 12 | WireMock fault injection, timeout/retry validation | Engineering/SRE |
| Cosmetic spacing defect | 2 | 1 | 2 | Exploratory/manual visual review | Product |

---

## 6. Risk Review Cadence

Review risk:
- During refinement for material stories/epics
- When architecture changes
- Before major releases
- After production incidents
- When defect trends indicate a weak control

Risk scores are not permanent. A previously low-risk component can become high-risk after increased traffic, architectural coupling, or repeated incidents.

---

## 7. Anti-Patterns

Avoid:
- Testing every requirement with equal depth
- Treating severity after a defect as the same thing as pre-release risk
- Using a risk score without documenting assumptions
- Letting a numeric score replace engineering judgment
- Ignoring residual risk because all automated tests passed

---

## 8. Decision Principle

**Test effort should follow business risk, not feature size or test-count targets.**
