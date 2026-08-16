# P4-06 Governance Approval Register

| ID | Decision | Proposed Value | Status | Source Artifact |
|----|----------|----------------|--------|-----------------|
| 1 | Pricing Model | Black-Scholes-Merton (BSM) | APPROVED BY PROJECT OWNER | P4-06-FINAL-PROJECT-APPROVAL.md |
| 2 | Greeks Scope | Delta, Gamma, Theta, Vega, IV | APPROVED | P4-06-FINAL-PROJECT-APPROVAL.md |
| 3 | Risk-Free Rate | Application Configuration | APPROVED BY PROJECT OWNER | P4-06-FINAL-PROJECT-APPROVAL.md |
| 4 | Dividend Yield | Application Configuration | APPROVED BY PROJECT OWNER | P4-06-FINAL-PROJECT-APPROVAL.md |
| 5 | Volatility Source | IV solved from option market LTP | APPROVED BY PROJECT OWNER | P4-06-FINAL-PROJECT-APPROVAL.md |
| 6 | IV Solver | Newton-Raphson with Bisection fallback | APPROVED BY PROJECT OWNER | P4-06-FINAL-PROJECT-APPROVAL.md |
| 7 | Time-to-Expiry | Calendar days to 15:30 IST | APPROVED BY PROJECT OWNER | P4-06-FINAL-PROJECT-APPROVAL.md |
| 8 | Day-Count Convention | Actual/365 | APPROVED BY PROJECT OWNER | P4-06-FINAL-PROJECT-APPROVAL.md |
| 9 | Price Units | Absolute INR prices | APPROVED BY PROJECT OWNER | P4-06-FINAL-PROJECT-APPROVAL.md |
| 10 | Contract Identity | Explicit mapping in DB/Model | APPROVED BY PROJECT OWNER | P4-06-FINAL-PROJECT-APPROVAL.md |
| 11 | ClickHouse Schema | ReplacingMergeTree(timestamp) | APPROVED | P4-06-FINAL-PROJECT-APPROVAL.md |
| 12 | ClickHouse ORDER BY | (underlying, expiry, strike, option_type, timestamp) | APPROVED BY PROJECT OWNER | P4-06-FINAL-PROJECT-APPROVAL.md |
| 13 | ClickHouse Partitioning | toYYYYMM(timestamp) | APPROVED | P4-06-FINAL-PROJECT-APPROVAL.md |
| 14 | Option Chain REST API | GET /api/v1/options/chain | APPROVED BY PROJECT OWNER | P4-06-FINAL-PROJECT-APPROVAL.md |
| 15 | API Request Parameters | `underlying`, `expiry` | APPROVED BY PROJECT OWNER | P4-06-FINAL-PROJECT-APPROVAL.md |
| 16 | API Response Structure | OptionChainEntry[] (call/put nested) | APPROVED BY PROJECT OWNER | P4-06-FINAL-PROJECT-APPROVAL.md |
| 17 | Authentication | JWT Middleware | APPROVED | P4-06-FINAL-GOVERNANCE-DECISION.md |
| 18 | Authorization | Authenticated Users | APPROVED | P4-06-FINAL-GOVERNANCE-DECISION.md |
| 19 | Underlying Spot Canonical Source | Redis latest-spot (from MarketTick) | APPROVED BY PROJECT OWNER | P4-06-FINAL-PROJECT-APPROVAL.md |
| 20 | Underlying Instrument Mapping | Explicit string parser or lookup | APPROVED BY PROJECT OWNER | P4-06-FINAL-PROJECT-APPROVAL.md |
| 21 | Spot Timestamp Alignment | Latest valid spot <= option | APPROVED BY PROJECT OWNER | P4-06-FINAL-PROJECT-APPROVAL.md |
| 22 | Maximum Spot Staleness | 1 second | APPROVED BY PROJECT OWNER | P4-06-FINAL-PROJECT-APPROVAL.md |
| 23 | Missing Spot Semantics | Graceful reject, do not fabricate | APPROVED BY PROJECT OWNER | P4-06-FINAL-PROJECT-APPROVAL.md |
| 24 | Invalid Spot Semantics | Reject calculation | APPROVED BY PROJECT OWNER | P4-06-FINAL-PROJECT-APPROVAL.md |
| 25 | Calculation Trigger | Async decoupled worker | APPROVED | P4-06-FINAL-GOVERNANCE-DECISION.md |
| 26 | Worker Failure Semantics | Drop gracefully, don't crash | APPROVED | P4-06-FINAL-GOVERNANCE-DECISION.md |
| 27 | Realtime Greeks | No realtime events | DEFERRED | P4-06-FINAL-GOVERNANCE-DECISION.md |
| 28 | Frontend Scope | No UI implementation | DEFERRED | P4-06-FINAL-GOVERNANCE-DECISION.md |
| 29 | Broker Boundary | Internal ticks only | DEFERRED | P4-06-FINAL-GOVERNANCE-DECISION.md |

## P4-06 GOVERNANCE GATE

**P4-06 GOVERNANCE = APPROVED**
**P4-06 IMPLEMENTATION = AUTHORIZED**
**P4-06 READINESS = READY**
