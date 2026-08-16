# P4-06 Forensic Discovery Report

## 1. Executive Summary
This forensic discovery evaluates the governance, architecture, and existing codebase for Phase P4-06 (Option Chain & Greeks Foundation). The audit determined that while P4-06 is the designated phase for these capabilities, critical architectural and governance gaps exist. Specifically, calculation models, database schemas for querying chains, and API/Realtime contracts are completely undefined. **P4-06 is BLOCKED pending explicit governance decisions.**

## 2. Actual P4-06 Governed Scope
According to `PHASE4_ARCHITECTURE_ROADMAP.md`:
- **Scope**: "Implement Worker jobs for Greeks/IV calculation and ClickHouse persistence."
- **Governed**: Backend calculation of Greeks/IV, ClickHouse persistence.
- **Not Governed/Deferred**: Live broker feeds (P4-07/08), Realtime Greeks distribution, Chart library integration.

## 3. Repository Evidence
- **Models**: `Backend/internal/modules/marketdata/models/market.go` defines `OptionGreeks` and `OptionChainEntry`.
- **Database**: `Backend/migrations/clickhouse/000002_create_option_greeks.up.sql` defines the persistence layer.
- **API/Routes**: No option chain or greeks retrieval endpoints exist.
- **Calculation Logic**: No `BlackScholes` or pricing libraries exist.
- **Frontend**: `/analyse` and `/trade` contain explicit placeholders deferring Option Chain logic. Chart usage is blocked in `ChartContainer`.

## 4. Governance Evidence
- `IMPL_013_v2.0.md` explicitly lists Black-Scholes location (Backend vs WASM) as `[DECISION REQUIRED]`.
- `EVENT_CATALOG.md` has no events for realtime greeks or option chains.
- `PHASE4_ARCHITECTURE_ROADMAP.md` delegates worker jobs to P4-06.
- `Volume_2_Requirements.md` REQ-ANALYSE-001 requires an interactive option chain with real-time greeks.

## 5. Traceability Matrix
- Option Chain Worker Jobs: **GOVERNED** (`PHASE4_ARCHITECTURE_ROADMAP.md`)
- Black-Scholes Calculation Model: **GOVERNANCE GAP** (`IMPL_013_v2.0.md`)
- ClickHouse Greeks Persistence: **EXISTING REPOSITORY CAPABILITY** (`000002_create_option_greeks.up.sql`)
- Option Chain REST API: **GOVERNANCE GAP** (No specs)
- Realtime Greeks Distribution: **GOVERNANCE GAP** (Not in `EVENT_CATALOG.md`)
- Option Chain UI Integration: **GOVERNANCE GAP** (Deferred, no component specs)
- Broker Live Feeds: **DEFERRED** (P4-07/08)
- Fake Market Data: **NOT FOUND** (Strictly prohibited)

## 6. Existing Market Data Architecture
P4-04 established buffered ClickHouse ingestion. P4-05 established realtime tick distribution decoupled from ClickHouse success. Option Greeks ingestion exists via `ProcessGreeksBatch`, but no component *generates* or *calculates* these Greeks yet.

## 7. OptionGreeks Model Audit
Existing struct in `market.go`:
- `Symbol` (string)
- `Delta`, `Gamma`, `Theta`, `Vega`, `IV` (float64)
- `Timestamp` (time.Time)
**Missing Fields**: `Rho`, `Strike`, `Expiry`, `OptionType` (CE/PE), `UnderlyingSymbol`.
*Status*: Technically insufficient for performant Option Chain grouping without explicit string parsing logic.

## 8. Option Chain Model Audit
Existing struct `OptionChainEntry` encapsulates `StrikePrice`, `ExpiryDate`, `CallData`, `PutData`. 
*Status*: Present, but no mechanism to populate it from ClickHouse exists.

## 9. Greeks Calculation Audit
**NOT FOUND**. No mathematical models exist in the repository.

## 10. IV Calculation Audit
**NOT FOUND**. No implied volatility solvers (e.g., Newton-Raphson, Bisection) exist.

## 11. ClickHouse Audit
Table `option_greeks`:
- `symbol String`
- `timestamp DateTime64(3, 'UTC')`
- `delta, gamma, theta, vega, iv Float64`
*Status*: To query an "Option Chain" (all strikes for an underlying expiry), ClickHouse would require parsing `symbol` at query time (e.g., extracting "NIFTY", "24DEC", "21000", "CE" from "NIFTY24DEC21000CE"). A schema migration adding `underlying`, `strike`, `expiry`, and `option_type` is practically required for performance, but this is a **Governance Decision**.

## 12. API Audit
**GOVERNANCE GAP**. No REST APIs exist for fetching Option Chains.

## 13. Redis/WebSocket Audit
**GOVERNANCE GAP**. `EVENT_CATALOG.md` lacks `market.greeks` or `option.chain` definitions.

## 14. Frontend Audit
- `/analyse` and `/trade` tabs have placeholders.
- `ChartContainer` exists but charting libraries (TradingView, Recharts) are strictly prohibited pending governance.

## 15. Broker Boundary Audit
Brokers (Zerodha, Angel One) are explicitly deferred. Greeks must be calculated from internal ticks, not broker APIs.

## 16. Security Audit
All APIs must use existing JWT architecture. WS must use one-time tickets.

## 17. Failure Semantics
Calculations must not block tick ingestion.

## 18. Performance Considerations
Worker jobs calculating Greeks across thousands of strikes per tick require high concurrency and memory efficiency.

## 19. Data Integrity Risks
Calculating Greeks on delayed/dropped ticks causes severe IV skew.

## 20. Governance Gaps
1. **Calculation Model**: Black-Scholes vs BSM? Backend vs WASM? Interest rate/dividend assumptions?
2. **Schema Migration**: Must ClickHouse `option_greeks` be migrated to include individual contract attributes (Strike, Expiry)?
3. **API Contracts**: What are the REST boundaries for fetching chains?
4. **Realtime Contracts**: Are Greeks streamed over Redis?

## 21. Deferred Capabilities
Broker integration, Charting.

## 22. Exact File Inventory
Inspected:
- `Backend/internal/modules/marketdata/models/market.go`
- `Backend/internal/modules/marketdata/services/ingest_service.go`
- `Backend/migrations/clickhouse/000002_create_option_greeks.up.sql`
- `Architecture/PHASE4_ARCHITECTURE_ROADMAP.md`
- `Architecture/EVENT_CATALOG.md`
- `Architecture/Phase-2/IMPL_013_v2.0.md`
- `Frontend/src/app/(authenticated)/analyse/page.tsx`
- `Frontend/src/app/(authenticated)/trade/page.tsx`

## 23. Readiness Matrix
- DISCOVERY: COMPLETE
- GOVERNANCE: BLOCKED
- IMPLEMENTATION: NOT STARTED

## 24. Final Verdict
**P4-06 READINESS = BLOCKED.** Implementation cannot begin until explicit architectural decisions are made regarding calculation methodology, database schemas, and API contracts.
