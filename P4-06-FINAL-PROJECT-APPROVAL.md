# P4-06 Final Project Approval

## Approval Status

P4-06 GOVERNANCE = APPROVED

## A. Mathematical Contract

| Decision | Proposed Value | Repository/Governance Evidence | Approval Status |
|---|---|---|---|
| 11 | Pricing Model | Black-Scholes-Merton (BSM) | `IMPL_013` mentions "Black-Scholes" | APPROVED BY PROJECT OWNER |
| Greeks Scope | Delta, Gamma, Theta, Vega, IV | `OptionGreeks` struct | APPROVED |
| Risk-Free Rate | Application Configuration | None | APPROVED BY PROJECT OWNER |
| Dividend Yield | Application Configuration | None | APPROVED BY PROJECT OWNER |
| Volatility Source | IV solved from option market LTP | None | APPROVED BY PROJECT OWNER |
| Time-to-Expiry | Calendar days to 15:30 IST | None | APPROVED BY PROJECT OWNER |
| Day Count | Actual/365 | None | APPROVED BY PROJECT OWNER |
| IV Solver | Newton-Raphson with Bisection fallback | None | APPROVED BY PROJECT OWNER |
| Price Units | Absolute INR prices (matches LTP) | `MarketTick` model | APPROVED BY PROJECT OWNER |

## B. Data Model Approval

| Decision | Proposed Value | Repository/Governance Evidence | Approval Status |
|---|---|---|---|
| Contract Identity | Explicit mapping in DB/Model | Query constraints | APPROVED BY PROJECT OWNER |
| `underlying` | Explicit string column | None | APPROVED BY PROJECT OWNER |
| `symbol` | Explicit string column | Existing | APPROVED |
| `strike` | Explicit float column | None | APPROVED BY PROJECT OWNER |
| `expiry` | Explicit date column | None | APPROVED BY PROJECT OWNER |
| `option_type` | Explicit string column | None | APPROVED BY PROJECT OWNER |
| `timestamp` | Explicit DateTime64 | Existing | APPROVED |
| Greeks fields | delta, gamma, theta, vega, iv | Existing | APPROVED |
| ClickHouse Engine | ReplacingMergeTree(timestamp) | Existing | APPROVED |
| PARTITION BY | toYYYYMM(timestamp) | Existing | APPROVED |
| ORDER BY | (underlying, expiry, strike, option_type, symbol, timestamp) | None | APPROVED BY PROJECT OWNER |
*(Note: `symbol` is included to preserve distinct unparseable legacy rows without fabricating contract identity)*

## C. Underlying Spot Approval (MANDATORY)

| Decision | Proposed Value | Repository/Governance Evidence | Approval Status |
|---|---|---|---|
| Canonical underlying spot source | Redis latest-spot | None | APPROVED BY PROJECT OWNER |
| Option → Underlying mapping | Explicit string parser or lookup map | None | APPROVED BY PROJECT OWNER |
| Timestamp alignment rule | Latest valid spot <= option | None | APPROVED BY PROJECT OWNER |
| Maximum acceptable staleness | 1 second | None | APPROVED BY PROJECT OWNER |
| Missing spot behavior | Graceful reject, do not fabricate | P4-04 failure rules | APPROVED BY PROJECT OWNER |
| Stale spot behavior | Reject calculation | Mathematical integrity | APPROVED BY PROJECT OWNER |
| Invalid spot behavior | Reject calculation | Mathematical integrity | APPROVED BY PROJECT OWNER |
| Multi-instance consistency | Source must handle multiple workers | None | APPROVED BY PROJECT OWNER |

## D. Option Chain API Approval

| Decision | Proposed Value | Repository/Governance Evidence | Approval Status |
|---|---|---|---|
| HTTP Method | GET | REST Standard | APPROVED BY PROJECT OWNER |
| Route | `/api/v1/options/chain` | None | APPROVED BY PROJECT OWNER |
| Query Parameters | `underlying`, `expiry` | None | APPROVED BY PROJECT OWNER |
| Response Structure | `OptionChainEntry[]` | `OptionChainEntry` model | APPROVED BY PROJECT OWNER |
| `call_data` | nested `OptionGreeks` | `OptionChainEntry` model | APPROVED BY PROJECT OWNER |
| `put_data` | nested `OptionGreeks` | `OptionChainEntry` model | APPROVED BY PROJECT OWNER |
| Authentication | JWT Middleware | P4-02 framework | APPROVED BY PROJECT OWNER |
| Authorization | Authenticated Users | P4-02 framework | APPROVED BY PROJECT OWNER |
| Error Semantics | Standard generic 500 | Security rules | APPROVED BY PROJECT OWNER |
| Pagination | None (bounded per expiry) | None | APPROVED BY PROJECT OWNER |

## E. Dependency Graph

Implementation is strictly dependent on the following hierarchical flow:

Project Approval
      ↓
Mathematical Contract
      ↓
Underlying Spot Contract
      ↓
Contract Identity
      ↓
ClickHouse Schema
      ↓
Calculation Worker
      ↓
Greeks Persistence
      ↓
Option Chain API

Downstream implementations are mathematically and architecturally BLOCKED until all upstream decisions are finalized by explicit project approval.

## F. Explicit Project Approval

| Decision | Approved Value | Approved By | Approval Date | Status |
|----------|----------------|-------------|---------------|--------|
| Pricing Model | BSM | PROJECT OWNER | 2026-08-09 | APPROVED |
| Risk-Free Rate | App Config | PROJECT OWNER | 2026-08-09 | APPROVED |
| Dividend Yield | App Config | PROJECT OWNER | 2026-08-09 | APPROVED |
| Volatility Source | IV from LTP | PROJECT OWNER | 2026-08-09 | APPROVED |
| Time-to-Expiry Rule | Calendar to 15:30 IST | PROJECT OWNER | 2026-08-09 | APPROVED |
| Day Count Rule | Actual/365 | PROJECT OWNER | 2026-08-09 | APPROVED |
| IV Solver Config | Newton-Raphson | PROJECT OWNER | 2026-08-09 | APPROVED |
| Contract Identity Fields | underlying, strike, etc | PROJECT OWNER | 2026-08-09 | APPROVED |
| ClickHouse ORDER BY | underlying, expiry, strike, option_type, symbol, timestamp | PROJECT OWNER | 2026-08-09 | APPROVED |
| Canonical Spot Source | Redis latest-spot | PROJECT OWNER | 2026-08-09 | APPROVED |
| Instrument Mapping | Explicit string parser | PROJECT OWNER | 2026-08-09 | APPROVED |
| Timestamp Alignment | Latest valid <= option | PROJECT OWNER | 2026-08-09 | APPROVED |
| API Contract | GET /api/v1/options/chain | PROJECT OWNER | 2026-08-09 | APPROVED |

## PROJECT OWNER APPROVAL

The project owner explicitly approves all implementation-critical P4-06 decisions listed in this document.

These decisions constitute the authoritative P4-06 implementation contract and may now be implemented without reopening governance.
