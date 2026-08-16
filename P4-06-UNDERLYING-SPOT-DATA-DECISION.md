# P4-06 Underlying Spot Data Decision

## 1. Problem Statement
The Black-Scholes-Merton (BSM) formula and Implied Volatility (IV) calculations require the Underlying Spot Price (`S`) at the exact or nearest timestamp to the option market price. Currently, the P4-04 ingestion pipeline processes ticks containing an option's `Symbol` and `LTP` (e.g., `NIFTY24DEC21000CE`, `150.50`), but it does not synchronously possess or cache the underlying asset's spot price (e.g., `NIFTY`, `21050.00`). Without `S`, mathematical calculation of Greeks is impossible.

## 2. Repository Evidence
- **P4-04 `MarketTick` model:** Contains `Symbol`, `LTP`, `Timestamp`. It lacks any embedded underlying price.
- **P4-04 `IngestService`:** Flushes batches of ticks directly to ClickHouse. It does not cache the latest spot price in memory or Redis.
- **ClickHouse `market_ticks`:** Stores historical ticks, but querying ClickHouse synchronously per option tick to find the latest underlying spot price would introduce severe latency and violate P4-06 non-blocking constraints.
- **P4-07/08 Broker boundary:** Explicitly deferred. We cannot query an external broker API for spot prices during calculation.

## 3. Candidate Sources

| Candidate | Repository Evidence | Governance Status | Suitable? |
|---|---|---|---|
| In-Memory Cache (Ingest Service) | None | Ungoverned | High risk of stale data if load-balanced across multiple instances. |
| Redis Fast-Cache (e.g., HSET) | None | Ungoverned | Highly suitable, but requires explicit architecture approval to implement Redis state holding. |
| ClickHouse Synchronous Query | `market_ticks` table | Existing but blocking | Unsuitable. Violates P4-06 non-blocking rules. |
| Broker API (Live) | Deferred to P4-07 | Deferred | Unsuitable. Violates P4-06 boundaries. |

## 4. Canonical Source
CANONICAL SOURCE = REQUIRES EXPLICIT PROJECT APPROVAL

## 5. Instrument Resolution
Mapping an option symbol (e.g., `NIFTY24DEC21000CE`) to its underlying instrument (e.g., `NIFTY`) requires authoritative parsing rules or an explicit instrument mapping dictionary. Currently, this does not exist.
UNDERLYING RESOLUTION = REQUIRES EXPLICIT PROJECT APPROVAL

## 6. Timestamp Relationship
The difference in timestamps between the option tick and the most recently cached underlying spot tick can cause IV skew if the market is moving rapidly. A maximum acceptable staleness threshold must be defined.
TIMESTAMP ALIGNMENT = REQUIRES EXPLICIT PROJECT APPROVAL

## 7. Missing/Stale Data
If the underlying spot price is completely missing or exceeds the staleness threshold, the Greeks cannot be calculated.
FAILURE SEMANTICS = REQUIRES EXPLICIT PROJECT APPROVAL

## 8. Mathematical Dependency Contract
To successfully calculate P4-06 Greeks, the worker MUST possess:
- `S` = Underlying Spot Price (Currently blocked)
- `K` = Strike (Requires explicit contract identity)
- `T` = Time to expiry (Requires explicit contract identity)
- `r` = Risk-free rate (Requires config approval)
- `q` = Dividend yield (Requires config approval)
- `option price` = Tick LTP (Available)

## 9. Final Decision
**UNDERLYING SPOT SOURCE = REQUIRES EXPLICIT PROJECT APPROVAL**
