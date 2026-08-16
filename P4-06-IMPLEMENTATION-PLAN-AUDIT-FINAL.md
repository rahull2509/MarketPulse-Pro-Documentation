# P4-06 Implementation Plan Audit Final Report

## 1. Executive Summary
A strict forensic read-only audit of the proposed `implementation_plan.md` has been conducted against the repository and the approved P4-06 governance artifacts. The audit has identified critical ambiguities and gaps in the proposed plan, particularly concerning the P4-04 ingestion boundary, spot cache semantics, and Fx dependency injection. The plan is **APPROVED WITH REQUIRED PLAN CORRECTIONS**.

## 2. Governance Compliance
The plan proposes BSM, Newton-Raphson, absolute INR prices, Actual/365 day count, etc. These align with governance.
However, the plan is too vague on *when* the Redis spot cache is updated relative to ClickHouse persistence, and it misses strict definitions for identifying underlying vs option ticks.

## 3. Repository Compatibility
The models (`MarketTick`, `OptionGreeks`, `OptionChainEntry`) exist in `models/market.go`. The repository interface exists in `interfaces/repository.go`.
However, the plan failed to detail the exact injection via Uber Fx for the new `SpotCache`, `Parser`, and `bsm.go`/`solver.go` components.

## 4. P4-04 Boundary Audit
**CRITICAL BLOCKER IDENTIFIED**
- **A. Is the spot cache update occurring before or after successful ClickHouse persistence?** The plan states "Update flushTicks to update Redis Spot Cache". It must explicitly state this occurs **AFTER** successful ClickHouse persistence.
- **B. Is a spot considered canonical only after successful historical persistence?** Yes, per P4-04.
- **C. Can Greeks calculation consume an option tick that has not yet been successfully persisted?** If enqueued before `s.repo.InsertMarketTicks`, yes. The plan must explicitly enqueue option ticks to `greeksCh` **AFTER** ClickHouse persistence.
- **D. Can Redis failure block ClickHouse ingestion?** If the cache update is synchronous and blocking, it could. Updates must be asynchronous or fail-safe (e.g., logged and ignored without failing `flushTicks`).
- **E. Can Greeks worker failure block ClickHouse ingestion?** If pushing to `greeksCh` is blocking, it will halt the `tickWorker`. Pushes must be non-blocking with capacity checks.

## 5. Spot Cache Audit
**CRITICAL BLOCKER IDENTIFIED**
- **Which ticks update the cache?** The plan does not define how to distinguish an underlying tick (e.g. `NIFTY`) from an option tick (e.g. `NIFTY24DEC21000CE`).
- **Timestamp storage:** Must be explicitly stored within the Redis value to allow the max 1-second staleness check.
- **Redis failure:** Must not affect P4-04 ingestion. Must fail safely.

## 6. Contract Parser Audit
**HIGH SEVERITY ISSUE**
- The repository does not currently contain an authoritative option symbol parser (verified via search). 
- The plan must explicitly define the parsing logic (Regex or string manipulation) for standard option symbols (e.g., `[A-Z]+[0-9]{2}[A-Z]{3}[0-9]+[CP]E`) and gracefully reject non-matching symbols instead of panicking.

## 7. ClickHouse Migration Audit
**MEDIUM SEVERITY ISSUE**
- The proposed migration uses `ALTER TABLE ... MODIFY ORDER BY`. While supported in newer ClickHouse versions, the plan must ensure that the `DOWN` migration modifies the `ORDER BY` *before* dropping the identity columns, as dropping columns used in the sorting key is invalid. The proposed `DOWN` migration in the plan correctly does this, but the risk of syntax failure remains if the ClickHouse version is older.

## 8. Worker Architecture Audit
**HIGH SEVERITY ISSUE**
- The plan must explicitly use a non-blocking `select` when enqueuing to the local Greeks calculation channel to avoid backpressure crashing the P4-04 ingestion worker.

## 9. Mathematical Engine Audit
- The plan correctly proposes BSM, Newton-Raphson, and Bisection fallback.
- Must ensure that application configuration is correctly injected (Risk-Free Rate, Dividend Yield) and not hardcoded.

## 10. API Audit
- `GET /api/v1/options/chain` aligns with governance. Must ensure it is properly registered in the router with JWT middleware.

## 11. Fx Wiring Audit
**HIGH SEVERITY ISSUE**
- The plan completely omits Uber Fx dependency injection updates. `SpotCache`, the new API handler, and the Greeks engine must be provided to the Fx container in the respective `module.go` files.

## 12. P4-05 Regression Audit
- The plan does not modify the realtime publication boundary (`enqueueRealtimeBatch`), which is correct. Realtime events will continue to fire after ClickHouse persistence.

## 13. Test Coverage Audit
- The plan outlines the correct test scenarios, including BSM known vectors, Redis spot storage, and timestamp alignment.

## 14. Security Audit
- JWT authentication is retained. No ClickHouse errors exposed.

## 15. Required Corrections

### REQUIRED CHANGES

1. **Severity:** CRITICAL
   - **Current plan:** "Update flushTicks to update Redis Spot Cache for underlying ticks."
   - **Required correction:** Explicitly state that spot cache updates and Greeks channel enqueueing occur **ONLY AFTER** successful ClickHouse persistence in `flushTicks` to preserve the P4-04 boundary. Redis failures must be logged and ignored without failing `flushTicks`.
   - **Exact file(s) affected:** `Backend/internal/modules/marketdata/services/ingest_service.go`

2. **Severity:** CRITICAL
   - **Current plan:** Push option ticks to local channel.
   - **Required correction:** Must use a non-blocking `select` with a `default` case to drop/log ticks if the Greeks channel is full, preventing backpressure from blocking P4-04 `tickWorker`.
   - **Exact file(s) affected:** `Backend/internal/modules/marketdata/services/ingest_service.go`

3. **Severity:** HIGH
   - **Current plan:** "Implement deterministic resolution from option symbol"
   - **Required correction:** Define explicit regex or parsing logic (e.g., differentiating `NIFTY` from `NIFTY24DEC21000CE`) to route ticks either to Spot Cache or Greeks Calculation.
   - **Exact file(s) affected:** `Backend/internal/modules/marketdata/services/ingest_service.go`, `Backend/internal/modules/marketdata/parser/parser.go`

4. **Severity:** HIGH
   - **Current plan:** No Fx wiring mentioned.
   - **Required correction:** Update `Backend/internal/modules/marketdata/module.go` and application bootstraps to provide `SpotCache` and `OptionChainHandler`.
   - **Exact file(s) affected:** `Backend/internal/modules/marketdata/module.go`

## 16. Final Verdict

**APPROVED WITH REQUIRED PLAN CORRECTIONS**
