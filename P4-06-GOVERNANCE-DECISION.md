# P4-06 Governance Decision

## 1. Purpose
This document formally evaluates and resolves the architectural and governance gaps identified in the P4-06 Forensic Discovery phase for the Option Chain & Greeks Foundation. It strictly evaluates existing repository artifacts to classify required decisions.

## 2. Existing Governance Evidence
- `PHASE4_ARCHITECTURE_ROADMAP.md`: Explicitly dictates "Implement Worker jobs for Greeks/IV calculation and ClickHouse persistence."
- `IMPL_013_v2.0.md`: States "Black-Scholes calculation must happen on the backend or via a heavily optimized WASM/JS library on the client. [DECISION REQUIRED]".
- `EVENT_CATALOG.md`: Excludes any mention of `market.greeks` or realtime option chains.
- `Volume_2_Requirements.md`: Requires interactive option chain with real-time greeks (but actual distribution boundaries are deferred by P4-04/P4-05).

## 3. Existing Repository Evidence
- `Backend/internal/modules/marketdata/models/market.go`: Contains `OptionGreeks` (Delta, Gamma, Theta, Vega, IV) and `OptionChainEntry` (Strike, Expiry).
- `Backend/migrations/clickhouse/000002_create_option_greeks.up.sql`: Contains only `symbol` and `timestamp` alongside the 5 greeks.
- Frontend `/analyse` and `/trade`: Explicitly deferred via placeholders. Charting explicitly blocked.

## 4. Decision Matrix

| Decision | Evidence | Current Status | Proposed Resolution | Governance Required |
|----------|----------|----------------|---------------------|---------------------|
| 1. Calculation Location | `ROADMAP`, `IMPL_013` | EXISTING GOVERNANCE | Backend Go worker jobs | NO |
| 2. Mathematical Model | `IMPL_013` | GOVERNANCE GAP | Define Black-Scholes vs BSM & parameters | YES |
| 3. Greeks Scope | `market.go` | EXISTING REPOSITORY CAPABILITY | Delta, Gamma, Theta, Vega, IV | NO |
| 4. Contract Identity | `000002_create_option_greeks.up.sql` | GOVERNANCE GAP | Must define if schema expands or symbol parsed | YES |
| 5. ClickHouse Schema | `000002_create_option_greeks.up.sql` | GOVERNANCE GAP | Must define SQL migration for Strike/Expiry | YES |
| 6. Calculation Trigger | `ROADMAP`, P4-04 Ingestion | ARCHITECTURAL INFERENCE | Async worker post-ingestion. Must not block ClickHouse | NO |
| 7. Option Chain API | `routes.go` | GOVERNANCE GAP | Define exact REST API contract | YES |
| 8. Realtime Greeks | `EVENT_CATALOG.md` | GOVERNANCE GAP | Deferred. No Redis event creation. | NO |
| 9. Frontend Scope | `/analyse`, `ChartContainer` | DEFERRED | Placeholder preservation. No charting. | NO |
| 10. Broker Boundary | P4-07/08 Governance | DEFERRED | Internal market ticks only. No live brokers. | NO |

## 5. Calculation Architecture Decision
- **Evidence**: `PHASE4_ARCHITECTURE_ROADMAP.md` explicitly lists "Implement Worker jobs for Greeks/IV calculation and ClickHouse persistence."
- **Status**: EXISTING GOVERNANCE.
- **Resolution**: Calculations MUST live in the Go backend as worker jobs. Frontend WASM is rejected based on roadmap supersedence.
- **Constraints**: Calculation engine ownership belongs to the Backend. Calculations must execute asynchronously and MUST NOT block P4-04 ingestion latency.

## 6. Mathematical Model Decision
- **Evidence**: `IMPL_013_v2.0.md` mentions "Black-Scholes", but mathematical assumptions are completely undefined.
- **Status**: GOVERNANCE GAP.
- **Resolution**: Explicit governance is required for:
  - Exact formula (e.g., Black-Scholes-Merton)
  - Risk-Free Rate assumption (hardcoded 10% vs dynamic)
  - Dividend Yield assumption (hardcoded 0% vs dynamic)
  - Volatility source (Historical vs Implied derivation)
  - Time-to-Expiry convention (Trading days vs Calendar days)
  - Day-count convention (e.g., Actual/365)
  - Price units

## 7. Greeks Scope Decision
- **Evidence**: `Backend/internal/modules/marketdata/models/market.go` defines `OptionGreeks`.
- **Status**: EXISTING REPOSITORY CAPABILITY.
- **Resolution**: Delta, Gamma, Theta, Vega, IV are approved. Rho is excluded.

## 8. Option Contract Identity Decision
- **Evidence**: Performant grouping into an Option Chain requires `underlying`, `strike`, `expiry`, and `option_type`. Existing model only has `symbol`.
- **Status**: GOVERNANCE GAP.
- **Resolution**: Project leadership must approve a decision whether to rely on runtime string parsing of `symbol` in ClickHouse (highly discouraged for performance), or approve a schema addition for explicit contract identity columns.

## 9. ClickHouse Schema Decision
- **Evidence**: `000002_create_option_greeks.up.sql` lacks contract identity columns.
- **Status**: GOVERNANCE GAP.
- **Resolution**: If explicit columns are approved in Decision 8, a new migration must be governed adding `underlying`, `strike`, `expiry`, `option_type` to `option_greeks`. DO NOT execute this until approved.

## 10. Calculation Trigger Decision
- **Evidence**: P4-04 established atomic, unblocked batch ingestion.
- **Status**: ARCHITECTURAL INFERENCE.
- **Resolution**: Calculation MUST happen via asynchronous background workers drawing from successfully persisted ticks or a decoupled internal queue (e.g., Go channels). It must absolutely not block `InsertMarketTicks`.

## 11. Option Chain REST API Decision
- **Evidence**: No endpoints exist.
- **Status**: GOVERNANCE GAP.
- **Resolution**: A strict API contract (e.g., `GET /api/v1/options/chain`) including request parameters and response payload shapes must be defined and governed before implementation.

## 12. Realtime Greeks Event Decision
- **Evidence**: `EVENT_CATALOG.md` has no greek events.
- **Status**: GOVERNANCE GAP.
- **Resolution**: Do NOT introduce `market.greeks` or `option.chain` Redis events. Realtime greek distribution is explicitly excluded from P4-06 pending governance.

## 13. Frontend Scope Decision
- **Evidence**: `/analyse` and `/trade` defer option chain logic.
- **Status**: DEFERRED.
- **Resolution**: P4-06 remains a backend-only phase. Frontend placeholders are preserved. Charting libraries are strictly prohibited.

## 14. Broker Boundary Decision
- **Evidence**: P4-07/P4-08 govern broker integration.
- **Status**: DEFERRED.
- **Resolution**: P4-06 must NOT integrate with Zerodha, Angel One, or Upstox. All calculations must be derived purely from internal P4-04 `market_ticks`.

## 15. Security Decision
- **Evidence**: Existing P4-02 framework.
- **Status**: EXISTING GOVERNANCE.
- **Resolution**: Any new REST API must use the existing JWT middleware.

## 16. Failure Semantics
- **Evidence**: P4-04 and P4-05 graceful degradation.
- **Status**: ARCHITECTURAL INFERENCE.
- **Resolution**: Mathematical panics, invalid option chains, or worker saturation must drop gracefully and log. They must never crash the ingest service.

## 17. Performance Requirements
- **Evidence**: High-frequency ticks.
- **Status**: ARCHITECTURAL INFERENCE.
- **Resolution**: Worker pools must utilize lock-free paradigms where possible and buffer calculations to avoid degrading the 10,000-capacity tick channel.

## 18. Explicit Non-Goals
- No fake market data.
- No broker tokens.
- No UI charting.
- No Kafka/NATS.

## 19. Remaining Governance Gaps
The following must be formally approved before any code is written:
1. Mathematical Model & Assumptions (Decision 6)
2. Option Contract Identity & Schema (Decisions 8 & 9)
3. REST API Contract (Decision 11)

## 20. Final P4-06 Approval Gate
P4-06 GOVERNANCE = BLOCKED

P4-06 IMPLEMENTATION = NOT STARTED
P4-06 READINESS = BLOCKED
ARCHITECTURE CONFIDENCE = HIGH
