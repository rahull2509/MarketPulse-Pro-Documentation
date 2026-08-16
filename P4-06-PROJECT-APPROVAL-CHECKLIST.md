# P4-06 Project Approval Checklist

> [!WARNING]
> For the final explicit sign-off matrix resolving all blocked items, see [P4-06-FINAL-PROJECT-APPROVAL.md](file:///c:/Users/rahul/Code/MarketPulse%20Pro/Architecture/P4-06-FINAL-PROJECT-APPROVAL.md).

## A. Mathematical Contract Approval

| Decision | Current Proposal | Evidence | Explicit Approval Required | Status |
|---|---|---|---|---|
| Pricing Model | Black-Scholes-Merton (BSM) | `IMPL_013` mentions "Black-Scholes" | YES | APPROVED BY PROJECT OWNER |
| Greeks Scope | Delta, Gamma, Theta, Vega, IV (No Rho) | `OptionGreeks` struct | NO (Already governed) | APPROVED |
| Risk-Free Rate | Application Configuration | None | YES | APPROVED BY PROJECT OWNER |
| Dividend Yield | Application Configuration | None | YES | APPROVED BY PROJECT OWNER |
| Volatility Source | Implied Volatility solved from tick LTP | None | YES | APPROVED BY PROJECT OWNER |
| Time-to-Expiry | Calendar days until expiry cutoff | None | YES | APPROVED BY PROJECT OWNER |
| Day-Count Convention | Actual/365 | None | YES | APPROVED BY PROJECT OWNER |
| IV Solver | Newton-Raphson with Bisection fallback | None | YES | APPROVED BY PROJECT OWNER |
| Price Units | Absolute INR prices (consistent with LTP) | `MarketTick` model | YES | APPROVED BY PROJECT OWNER |

## B. Option Contract Identity Approval

| Decision | Current Proposal | Evidence | Explicit Approval Required | Status |
|---|---|---|---|---|
| Contract Identity | Explicit Schema Columns (Avoid symbol parsing) | Query performance constraints | YES | APPROVED BY PROJECT OWNER |
| Schema Fields | `underlying`, `symbol`, `strike`, `expiry`, `option_type`, `timestamp`, Greeks | Missing from `option_greeks` table | YES | APPROVED BY PROJECT OWNER |

## C. ClickHouse Schema Approval

| Decision | Current Proposal | Evidence | Explicit Approval Required | Status |
|---|---|---|---|---|
| Engine | `ReplacingMergeTree(timestamp)` | Existing | YES | APPROVED BY PROJECT OWNER |
| Partitioning | `PARTITION BY toYYYYMM(timestamp)` | Existing | YES | APPROVED BY PROJECT OWNER |
| Ordering | `(underlying, expiry, strike, option_type, timestamp)` | Optimal Chain retrieval | YES | APPROVED BY PROJECT OWNER |
| Migration Strategy | Add explicit identity columns | Missing from schema | YES | APPROVED BY PROJECT OWNER |

## D. Option Chain REST API Approval

| Decision | Current Proposal | Evidence | Explicit Approval Required | Status |
|---|---|---|---|---|
| Route | `GET /api/v1/options/chain` | Standard REST convention | YES | APPROVED BY PROJECT OWNER |
| Parameters | `underlying`, `expiry` | Query constraints | YES | APPROVED BY PROJECT OWNER |
| Response | `OptionChainEntry[]` (call_data, put_data) | `OptionChainEntry` model | YES | APPROVED BY PROJECT OWNER |
| Auth & Authz | JWT Middleware, Authenticated users | P4-02 framework | YES | APPROVED BY PROJECT OWNER |
| Pagination | None (Bounded dataset per expiry) | Performance requirements | YES | APPROVED BY PROJECT OWNER |

## E. Underlying Spot Price Approval

| Decision | Candidate Proposals | Evidence | Explicit Approval Required | Status |
|---|---|---|---|---|
| Canonical Source | Redis latest spot from actual MarketTick | No existing governed canonical source | YES | APPROVED BY PROJECT OWNER |

## F. Underlying Instrument Resolution Approval

| Decision | Current Proposal | Evidence | Explicit Approval Required | Status |
|---|---|---|---|---|
| Mapping logic | Explicit contract resolution parser/map | Missing from governance | YES | APPROVED BY PROJECT OWNER |

## G. Timestamp Alignment Approval

| Decision | Candidate Proposals | Evidence | Explicit Approval Required | Status |
|---|---|---|---|---|
| Spot/Option alignment | Latest valid underlying spot <= option (Max 1s stale) | Missing from governance | YES | APPROVED BY PROJECT OWNER |

## H. Missing / Stale Spot Data Approval

| Decision | Current Proposal | Evidence | Explicit Approval Required | Status |
|---|---|---|---|---|
| Missing Spot Semantics | Fail gracefully, do not fabricate, no zero values | P4-04 failure rules | YES | APPROVED BY PROJECT OWNER |
| Stale Spot Semantics | Reject calculation if max staleness exceeded | Mathematical integrity | YES | APPROVED BY PROJECT OWNER |

## I. Architectural Dependencies

Decisions must be resolved strictly in this order to unblock downstream implementation:

1. **Underlying Spot Source**
        ↓
2. **Underlying Instrument Resolution**
        ↓
3. **Timestamp Alignment**
        ↓
4. **Mathematical Contract**
        ↓
5. **Greeks Calculation**
        ↓
6. **ClickHouse Schema**
        ↓
7. **Option Chain REST API**

## J. Already Approved Architecture

The following decisions are strictly governed by prior phases and MUST NOT be reopened:
- **Calculation Location:** Backend Go workers
- **Trigger:** Calculations must be asynchronous
- **Persistence:** ClickHouse remains historical analytical store
- **Source of Truth:** P4-04 ingestion remains authoritative
- **Distribution:** P4-05 realtime tick distribution remains separate
- **Realtime Greeks:** Deferred
- **Frontend Charting:** Deferred
- **Broker Integrations:** P4-07/P4-08
- **Mock Data:** No fake market data
- **Brokers:** No Kafka, No NATS
- **Security:** Existing JWT authentication
- **Frontend Scope:** No frontend implementation in P4-06

## K. Explicit Approval Statement

All implementation-critical items have been explicitly APPROVED BY PROJECT OWNER.

P4-06 IMPLEMENTATION IS AUTHORIZED.
