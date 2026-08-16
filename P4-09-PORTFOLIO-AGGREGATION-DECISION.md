# P4-09 PORTFOLIO AGGREGATION DECISION

**Status:** [APPROVED]
**Date:** 2026-08-10

## 1. Objective
Define the cross-broker portfolio aggregation mathematical rules and snapshot semantics.

## 2. Aggregation Mathematics (APPROVED)

- **1A. Quantity Aggregation:** For the same canonical P4-04 InternalSymbol: `net_quantity = SUM(provider_position.quantity)`. Provider positions must first be normalized.
- **1B. Long Positions:** A position is LONG when `net_quantity > 0`. Formula: `long_quantity = max(net_quantity, 0)`.
- **1C. Short Positions:** A position is SHORT when `net_quantity < 0`. Formula: `short_quantity = abs(min(net_quantity, 0))`.
- **1D. Zero-Quantity Positions:** Positions where `net_quantity == 0` MUST NOT appear in the aggregated active-position list (raw data remains for audit).
- **1E. Average Price:** Only quantities of the same direction are used: `weighted_average_price = SUM(quantity_i * average_price_i) / SUM(quantity_i)`. If the denominator is zero, average price MUST be null.
- **1F. Unrealized P&L:** `aggregated_unrealized_pnl = SUM(provider_unrealized_pnl)`. If invalid, do not fabricate.
- **1G. Realized P&L:** `aggregated_realized_pnl = SUM(provider_realized_pnl)`. Must not combine with unrealized P&L.

## 3. Instrument Identity (APPROVED)

- **1H. Duplicate Instruments:** Merged by canonical P4-04 InternalSymbol. Broker-specific symbols/tokens are ignored for aggregation.
- **1I. Conflicting Symbol Mappings:** P4-04 InternalSymbol remains authoritative. Unresolvable mappings are not aggregated (`ErrUnsupportedInstrument`); no symbol guessing is permitted.

**Status:** P4-09 PORTFOLIO AGGREGATION = APPROVED.
