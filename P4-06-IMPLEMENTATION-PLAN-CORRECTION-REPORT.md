# P4-06 Implementation Plan Correction Report

## 1. Original Audit Findings
The Final Implementation Readiness Audit (`P4-06-FINAL-IMPLEMENTATION-READINESS-AUDIT.md`) identified three critical required corrections before implementation could begin:
1. Exact ClickHouse safe migration strategy (table-recreate) instead of `MODIFY ORDER BY`.
2. Exact `SpotState` JSON schema and Redis value contract.
3. Explicit Uber Fx constructor signatures for dependency injection.

## 2. ClickHouse Migration Correction
The plan has been corrected to explicitly mandate the safe table-recreation strategy for ClickHouse migration `000003_update_option_greeks_schema.up.sql`. 
- **Legacy Data Preservation Strategy:** It will rename `option_greeks` to `option_greeks_old`, create the new table with the correct identity columns and `ORDER BY`, and then explicitly insert existing historical rows by mapping missing identity fields (underlying, strike, expiry, option_type) to safe zero-values/empty-strings (`'', 0, toDateTime64('1970-01-01', 3, 'UTC'), ''`). It will NOT fabricate historical contract metadata.
- **Rollback:** The `DOWN` migration uses the exact same safe recreate-and-copy strategy to restore the original table without silently destroying data.

## 3. SpotState Contract
The plan now explicitly defines the JSON serialization contract for Redis spot values:
```go
type SpotState struct {
    SpotPrice float64   `json:"spot_price"`
    Timestamp time.Time `json:"timestamp"`
}
```
- Missing key, invalid JSON, or invalid/non-finite SpotPrice will trigger a calculation rejection.
- Redis failures will safely log and swallow errors without blocking P4-04.

## 4. Redis Key Contract
The key `market:spot:{underlying}` is explicitly confirmed.

## 5. Timestamp Alignment
The calculation alignment rules (`T_spot <= T_option` and `T_option - T_spot <= 1 second`) are strictly enforced. Any spot older than 1s or in the future will result in calculation rejection.

## 6. Fx Provider Wiring
The plan accurately targets `Backend/internal/bootstrap/bootstrap.go` and explicitly defines the required Fx provider additions: `fx.Provide(cache.NewSpotCache, marketdatahandler.NewOptionChainHandler)`.

## 7. P4-04 Boundary Confirmation
The corrected plan strictly respects the historical persistence boundary. Calculations and cache updates execute **only after** successful ClickHouse tick persistence. Backpressure from the Greeks worker (`select { default: drop }`) ensures the `tickWorker` is never blocked.

## 8. P4-05 Boundary Confirmation
P4-05 realtime publication continues to execute independently after successful historical persistence. It is not reordered or blocked by P4-06.

## 9. Governance Compliance
The plan complies entirely with the BSM model, Greeks configuration, actual/365 math, JWT authentication, and absolute INR data structures.

## 10. Files Modified
- `C:\Users\rahul\.gemini\antigravity-ide\brain\d1aacd3f-b56f-4d52-bb26-35b6c890e159\implementation_plan.md`
- `Architecture/P4-06-IMPLEMENTATION-PLAN-CORRECTION-REPORT.md` (This file)

## 11. Files Untouched
- Application Code (`.go` files)
- Database Migrations (`.sql` files)
- Frontend Code (`.tsx`, `.ts` files)
- REST/Route Definitions
- Config
- Governance Decisions

## 12. Final Verdict
**P4-06 IMPLEMENTATION PLAN = CORRECTED**
**P4-06 CODE IMPLEMENTATION = NOT STARTED**
**P4-06 READINESS = READY FOR IMPLEMENTATION**
