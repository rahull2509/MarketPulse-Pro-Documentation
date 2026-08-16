# P4-06 Final Implementation Readiness Audit

## 1. Executive Summary
A final strict read-only pre-implementation gate forensic audit has been conducted on the current `P4-06-IMPLEMENTATION-PLAN.md`. While the plan has incorporated all major architectural boundaries and backpressure semantics, it still lacks exact explicit definitions for the ClickHouse migration strategy (to guarantee no data loss), explicit Fx provider signatures, and the exact SpotCache JSON struct. Therefore, the plan is **APPROVED WITH REQUIRED PLAN CORRECTIONS**. Implementation must not start until these final details are locked.

## 2. Governance Compliance
The plan fully complies with the BSM model, Greeks scope (Delta, Gamma, Theta, Vega, IV), Config-injected risk-free rate/dividend yield, Newton-Raphson/Bisection, Calendar Actual/365, absolute INR, and explicit contract identity. It respects the async, post-historical persistence boundary.

## 3. ClickHouse Migration Audit
**REQUIRED CORRECTION**
- **Issue:** The plan proposes `ALTER TABLE ... MODIFY ORDER BY`. Depending on the ClickHouse server version, `MODIFY ORDER BY` on a `ReplacingMergeTree` when adding new columns can be restricted or require part-mutations that risk locking or data loss.
- **Correction:** The plan MUST explicitly use a safe table-recreation migration pattern:
  1. Rename `option_greeks` to `option_greeks_old`.
  2. Create new `option_greeks` with the new schema and new `ORDER BY`.
  3. `INSERT INTO option_greeks SELECT ... FROM option_greeks_old` (mapping new columns to empty strings/zeros).
  4. Drop `option_greeks_old`.
  This guarantees existing data preservation and works across all ClickHouse versions.

## 4. P4-04 Boundary Audit
The plan successfully enforces the correct execution order:
`Tick` -> `P4-04 admission` -> `ClickHouse persistence` -> `ONLY AFTER SUCCESS` -> `Update Spot Cache & Enqueue Option Ticks`.
Redis failures and Greeks worker saturation safely drop calculations without failing P4-04.

## 5. Spot Cache Audit
**REQUIRED CORRECTION**
- **Issue:** The exact Redis value struct and serialization format is not defined.
- **Correction:** The plan must explicitly define the JSON value schema, e.g.:
  ```go
  type SpotState struct {
      SpotPrice float64   `json:"spot_price"`
      Timestamp time.Time `json:"timestamp"`
  }
  ```
  The key must be exactly `market:spot:{underlying}`.

## 6. Underlying Instrument Audit & 7. Contract Parser Audit
The plan mandates a deterministic parser that safely rejects malformed symbols, does not guess, and never panics. It explicitly prevents fabricating missing data.

## 8. Mathematical Engine Audit & 9. IV Solver Audit
BSM Call/Put, Delta, Gamma, Theta, Vega, and IV are defined correctly. Edge cases (NaN, Inf, negative TTE, non-convergence) are planned to be handled defensively. Newton-Raphson with Bisection fallback is included.

## 10. Worker Architecture Audit & 11. Greeks Persistence Audit
The `greeksWorker` correctly employs a bounded `greeksCh` using non-blocking `select { default: }` backpressure. P4-04 `tickWorker` is completely protected from Greeks calculation saturation. Greeks persistence will utilize the existing `InsertOptionGreeks` repository logic, executed in batches.

## 12. Fx Wiring Audit
**REQUIRED CORRECTION**
- **Issue:** The plan loosely states "register the new SpotCache".
- **Correction:** The plan must explicitly define the Fx lifecycle wiring. For example, `fx.Provide(cache.NewSpotCache)` in `Backend/internal/modules/marketdata/module.go`, ensuring the Redis client dependency is injected correctly.

## 13. Option Chain API Audit
Matches governance exactly: `GET /api/v1/options/chain` returning `OptionChainEntry[]`.

## 14. P4-05 Regression Audit & 15. P4-04 Regression Audit
No regressions identified. The realtime publication boundary is correctly preserved after ClickHouse persistence. P4-04 batch limits (10,000 tick buffer, 1,000 batch, 1s flush) remain untouched.

## 16. Test Coverage Audit & 17. Security Audit
Test coverage lists all critical boundaries (Math, IV, Parser, Spot Cache, Worker isolation, Database migration, API). Security is maintained (JWT auth, no ClickHouse error leakage).

## 18. Required Corrections
1. **ClickHouse Migration:** Explicitly detail the safe `CREATE TABLE AS ... INSERT ... DROP` migration strategy instead of `MODIFY ORDER BY` to absolutely guarantee zero data loss.
2. **Spot Cache:** Define the explicit `SpotState` JSON struct used for `market:spot:{underlying}`.
3. **Fx Wiring:** Explicitly name the providers (e.g., `fx.Provide(cache.NewSpotCache)`) in the implementation plan.

## 19. Final Verdict

**APPROVED WITH REQUIRED PLAN CORRECTIONS**
