# P4-06 Data Model Decision

This document formally proposes the required changes to the OptionGreeks domain model and ClickHouse schema to support P4-06 Option Chain queries.

## 1. Option Contract Identity
**Status:** PROPOSED GOVERNANCE DECISION (Approval Required)

The existing `OptionGreeks` struct and ClickHouse table (`option_greeks`) contain only a raw `symbol` string (e.g., `NIFTY24DEC21000CE`). Relying on runtime string parsing within ClickHouse queries to group options into chains (by underlying, expiry, and sorting by strike) is highly inefficient and brittle.

**Proposed Resolution:** Explicit contract columns MUST be added to the canonical identity.

## 2. Proposed Conceptual Schema
**Status:** PROPOSED GOVERNANCE DECISION (Approval Required)

The following schema is required for performant Option Chain grouping:
- `symbol` (String)
- `underlying` (String)
- `strike` (Float64)
- `expiry` (Date or DateTime)
- `option_type` (String: 'CE', 'PE')
- `timestamp` (DateTime64(3, 'UTC'))
- `delta` (Float64)
- `gamma` (Float64)
- `theta` (Float64)
- `vega` (Float64)
- `iv` (Float64)

## 3. ClickHouse Requirements
**Status:** PROPOSED GOVERNANCE DECISION (Approval Required)

- **Engine:** `ReplacingMergeTree(timestamp)` (Preserved)
- **Partitioning:** `PARTITION BY toYYYYMM(timestamp)` (Preserved)
- **Ordering (ORDER BY):** `(underlying, expiry, strike, option_type, symbol, timestamp)`
  *(Note: `symbol` is explicitly included to preserve distinct unparseable legacy rows without fabricating contract identity.)*
- **Deduplication Key:** Inherited from the `ORDER BY` clause.

*No SQL migration will be created until this decision is explicitly approved by project leadership.*
