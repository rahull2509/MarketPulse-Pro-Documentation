# P4-06 Mathematical Contract

This document formally proposes the mathematical pricing model and assumptions required for P4-06 Greeks and IV calculation. 

## 1. Pricing Model
**Status:** PROPOSED GOVERNANCE DECISION (Approval Required)
**Proposed:** Black-Scholes-Merton (BSM) for European options, as it supports continuous dividend yield. 
*Note: Existing governance mentions "Black-Scholes" generally, but BSM is the standard extension for indices (NIFTY/BANKNIFTY) which require dividend yield handling.*

## 2. Greeks Scope
**Status:** APPROVED BY EXISTING GOVERNANCE
**Approved:** Delta, Gamma, Theta, Vega, IV. 
*Evidence: Existing `OptionGreeks` model in `Backend/internal/modules/marketdata/models/market.go`.*

## 3. Risk-Free Rate
**Status:** PROPOSED GOVERNANCE DECISION (Approval Required)
**Proposed:** Application configuration value (e.g., loaded via `config.yaml` or Environment Variable). Hardcoding without configuration is discouraged.

## 4. Dividend Yield
**Status:** PROPOSED GOVERNANCE DECISION (Approval Required)
**Proposed:** Application configuration value. Hardcoding without configuration is discouraged.

## 5. Volatility Source
**Status:** PROPOSED GOVERNANCE DECISION (Approval Required)
**Proposed:** Implied Volatility (IV) calculated iteratively from market option prices (LTP) observed in ingested market ticks. Historical volatility is explicitly excluded from this phase.

## 6. Time To Expiry
**Status:** PROPOSED GOVERNANCE DECISION (Approval Required)
**Proposed:** Calendar days calculated continuously until expiry date cutoff (e.g., 15:30 IST on expiry day) relative to the `timestamp` of the ingested tick.

## 7. Day Count
**Status:** PROPOSED GOVERNANCE DECISION (Approval Required)
**Proposed:** Actual/365 convention.

## 8. Price Units
**Status:** PROPOSED GOVERNANCE DECISION (Approval Required)
**Proposed:** Absolute INR prices (e.g., `21000.50`), maintaining consistency with the existing `MarketTick` float64 `LTP`.

## 9. IV Solver
**Status:** PROPOSED GOVERNANCE DECISION (Approval Required)
**Proposed:** Newton-Raphson with Bisection fallback (or Brent's method), bounded between `0.01` and `5.00` (1% to 500%), with a max iteration limit to prevent worker thread saturation.
