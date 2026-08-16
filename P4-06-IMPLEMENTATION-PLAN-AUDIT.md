# P4-06 Implementation Plan Audit

## 1. Overall Verdict
**BLOCKED BY GOVERNANCE**

The proposed implementation plan contains severe undocumented assumptions that violate the strict governance boundary, particularly concerning mathematical data dependencies (Underlying Spot Price) and data migration strategies.

## 2. Approved Plan Elements
- **Mathematical Scope:** Implementation of BSM, Delta, Gamma, Theta, Vega, and IV.
- **Contract Identity Requirements:** Identifying the need to persist `underlying`, `strike`, `expiry`, and `option_type`.
- **Option Chain REST API:** Route `GET /api/v1/options/chain` with query parameters `underlying` and `expiry`, using JWT authentication.
- **Asynchronous Execution:** Worker-based architecture decoupled from P4-04 tick ingestion.

## 3. Unauthorized Assumptions
1. **Risk-Free Rate & Dividend Yield Values:** The plan assumed `0.10` (10%) and `0.00` (0%) as defaults. Governance required them to be configurable but did not explicitly approve these specific defaults.
2. **Symbol Parsing Strategy:** The plan assumed it should parse `symbol` strings (e.g., `NIFTY24DEC21000CE`) at runtime to populate the contract identity. The source-of-truth mapping for this parsing was not governed.
3. **Underlying Spot Price Source [CRITICAL FATAL FLAW]:** The plan proposed passing `ltp` to the BSM engine, completely omitting the Underlying Spot Price (`S`). An ingested tick for `NIFTY24DEC21000CE` only contains the option's LTP. Without synchronously retrieving the `NIFTY` spot price for the exact timestamp of the option tick, Greeks and IV cannot be calculated. The plan assumed this data magically existed.
4. **IV Solver Numerical Constraints:** The plan assumed it could handle "numerical edge cases" without defining the specific solver tolerance, maximum iterations, valid bounds, and Bisection fallback criteria.
5. **Expiry Representation & TTE:** The plan omitted definitions for timestamp timezone handling and exact continuous day calculations.
6. **Repository Interface:** The plan assumed a new `OptionsRepository` interface could be introduced. Evidence shows the existing `MarketDataRepository` in `interfaces/repository.go` already handles `InsertOptionGreeks`.

## 4. Governance Gaps

### Requires explicit project approval
- **Underlying Spot Synchronization:** How the Greeks worker retrieves the authoritative Underlying Spot Price (`S`) for a given option tick timestamp (e.g., Redis fast-cache vs ClickHouse query vs embedded in tick).
- **Contract Symbol Parsing Rules:** The authoritative regex or mapping logic to extract `underlying`, `strike`, `expiry`, and `option_type` from `symbol`.
- **IV Solver Parameters:** Explicit mathematical tolerance (e.g., `1e-6`) and maximum iteration limits (e.g., `100`).
- **Data Backfill Strategy:** Existing rows in `option_greeks` only have `symbol`. The migration cannot authoritatively backfill `strike` and `expiry` without parsing the symbol. If symbol parsing is error-prone, a backfill strategy must be explicitly approved.

### Can be resolved from existing repository conventions
- **Repository Interface:** The query method `GetOptionChain` should simply be added to the existing `interfaces.MarketDataRepository`.
- **JSON Tags:** `models/market.go` conclusively uses `snake_case`.

## 5. Data Integrity Risks
- **Incorrect IV / Greeks:** Attempting to calculate BSM without a perfectly synchronized Underlying Spot price will result in catastrophic IV skew and incorrect Greeks.
- **Historical Backfill Corruption:** Renaming the ClickHouse table without migrating existing data causes data fragmentation; attempting to migrate it via unapproved string parsing risks data corruption.
- **Worker Saturation:** Undefined IV solver bounds risk infinite loops or excessive CPU burn, saturating the worker pool and stalling the pipeline.

## 6. Corrected Implementation Plan
*Implementation is aborted. A corrected plan cannot be produced until the Underlying Spot Price architecture and Symbol Parsing rules are explicitly governed.*

## 7. Pre-Implementation Approval Checklist
- [ ] **Underlying Spot Architecture:** Approval on how the Greeks worker fetches the underlying spot price (e.g., Redis snapshot).
- [ ] **Symbol Parsing Rules:** Approval of the canonical parser logic mapping `NIFTY24DEC21000CE` to its components.
- [ ] **IV Solver Constraints:** Approval of max iterations and mathematical tolerance.
- [ ] **Migration Data Strategy:** Approval of whether to backfill old `option_greeks` rows via string parsing or abandon them in the new schema.
- [ ] **Numeric Defaults:** Explicit approval if 10% Risk-Free Rate and 0% Dividend Yield should be the fallback defaults when config is missing.

## 8. Final Status
**P4-06 IMPLEMENTATION = BLOCKED**
