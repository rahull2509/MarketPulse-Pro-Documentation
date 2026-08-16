# P4-06 Implementation Plan

## 1. Goal
Establish the backend Option Chain and Greeks calculation foundation, persisting computed analytical data to ClickHouse, and exposing it to the frontend.

## 2. Governed Scope
- Backend worker jobs for Greeks/IV calculation.
- Persistence of calculated Greeks to ClickHouse.
- (All other P4-06 scopes are currently blocked by missing governance).

## 3. Explicit Non-Goals
- Live broker feed integration (P4-07/08).
- Fake data generation.
- UI Charting (blocked pending library governance).

## 4. Architecture
**BLOCKED**. Calculation architecture (Backend vs. WASM) requires a governance decision as explicitly noted in `IMPL_013_v2.0.md`.

## 5. Data Flow
**BLOCKED**. Whether calculations happen synchronously during ingestion, asynchronously via queue, or entirely on-demand via REST is undefined.

## 6. Domain Model
**BLOCKED**. The current `OptionGreeks` struct and ClickHouse table lack `strike`, `expiry`, `option_type`, and `underlying_symbol`. A decision on schema expansion vs string parsing is required.

## 7. Calculation Model
**BLOCKED**. The mathematical model (Black-Scholes, BSM, Binomial) and constants (Risk-Free Rate, Dividend Yield) are undefined. 

## 8. Persistence Strategy
ClickHouse remains the authoritative store, but `option_greeks` schema migration requires governance approval.

## 9. API Contract
**BLOCKED**. No endpoints (e.g., `GET /api/v1/options/chain`) are governed.

## 10. Realtime Contract
**BLOCKED**. No Redis event envelopes for Greeks/Option Chain are present in `EVENT_CATALOG.md`.

## 11. Frontend Integration
**BLOCKED**. Pending API contracts.

## 12. Security
Existing JWT and WS Ticket auth will be used.

## 13. Failure Semantics
Calculations must fail gracefully without halting primary tick ingestion.

## 14. Performance
High-throughput worker pool required if calculations are performed on every tick.

## 15. Testing Strategy
- Unit test mathematical precision of the chosen model against known baselines.
- Integration test ClickHouse persistence.

## 16. Regression Strategy
Validate P4-04 ingestion latency remains unaffected. Validate P4-05 tick distribution remains unimpeded.

## 17. Exact File Inventory
Files to be modified/created:
- `Backend/internal/modules/marketdata/models/market.go`
- `Backend/internal/modules/marketdata/services/greeks_worker.go` (TBD)
- `Backend/internal/modules/marketdata/handlers/options_handler.go` (TBD)
- ClickHouse migrations (TBD)

## 18. Implementation Units
1. Finalize Governance (Prerequisite)
2. Define Domain Models & Schemas
3. Implement Calculation Engine
4. Integrate with Ingestion / Worker Jobs
5. Expose REST APIs

## 19. Risks
- Calculating Greeks synchronously could throttle ingestion throughput.
- Relying on string parsing for `symbol` in ClickHouse will degrade query performance.

## 20. Pending Decisions
1. **Calculation Location**: Backend vs Frontend WASM?
2. **Mathematical Model**: Exact formula and environment variables (interest rate, dividend yield)?
3. **Database Schema**: Approval for ClickHouse migration to add explicit contract fields.
4. **API Contracts**: REST schemas for fetching Option Chains.
5. **Realtime Contracts**: Redis events for streaming Greeks.

## 21. Readiness Matrix
- P4-06 DISCOVERY = COMPLETE
- P4-06 IMPLEMENTATION = NOT STARTED
- P4-06 READINESS = BLOCKED

## 22. Approval Gate
Implementation MUST NOT begin until the decisions in Section 20 are explicitly resolved by project governance.
