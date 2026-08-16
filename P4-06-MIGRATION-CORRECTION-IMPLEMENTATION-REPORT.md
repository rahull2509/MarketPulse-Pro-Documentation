# P4-06 MIGRATION CORRECTION IMPLEMENTATION REPORT

## 1. Root Cause
The forensic audit identified two critical migration defects in `000003_update_option_greeks_schema.up.sql`:
1. **Underlying Extraction**: The regex `^([A-Z0-9]+)` used in ClickHouse's `extract` function was too greedy and consumed the entire symbol (e.g. `NIFTY24DEC21000CE`), corrupting the `underlying` column.
2. **Unparseable Legacy Rows (Data Collapse)**: Using default identity placeholders `('', 1970-01-01, 0, '')` for unparseable legacy symbols created a data-integrity flaw where ClickHouse's `ReplacingMergeTree` would silently collapse and delete distinct legacy rows sharing the same timestamp, fundamentally violating the legacy data preservation requirement.

## 2. Exact SQL Correction (Underlying Extraction)
The greedy regex issue was resolved by replacing it with a precise, non-greedy subpattern bounded by the remainder of the option contract structure. 
**Corrected SQL Extraction:**
```sql
extract(symbol, '^([A-Z0-9]+?)[0-9]{2}[A-Z]{3}[0-9]+(?:CE|PE)$')
```
This correctly extracts only the underlying (e.g., `NIFTY`, `BANKNIFTY`, `RELIANCE`) without consuming the expiry, strike, or option type.

## 3. Legacy Row Preservation Strategy (Architectural Conflict)
**STATUS: BLOCKED BY GOVERNANCE CONFLICT**

In attempting to resolve the unparseable row data collapse without fabricating contract metadata, a fundamental architectural conflict was identified within the approved P4-06 governance.

The approved `ReplacingMergeTree` deduplication key (defined by `ORDER BY`) is:
```sql
ORDER BY (underlying, expiry, strike, option_type, timestamp)
```
Notice that `symbol` is **not** included in the `ORDER BY`.

If a legacy row is genuinely unparseable, we are forbidden from inventing synthetic identity. This implies we must leave `underlying`, `expiry`, `strike`, and `option_type` empty/default. 
If we do this, ANY two unparseable rows (e.g. `symbol='A'` and `symbol='B'`) that share the identical `timestamp` will be assigned the exact same `ORDER BY` tuple: `('', 1970-01-01, 0, '', timestamp)`.

ClickHouse `ReplacingMergeTree` mechanically collapses identical `ORDER BY` tuples during background merges. Thus, distinct legacy rows will be permanently deleted. 

**Conclusion:** It is mathematically impossible to preserve distinct unparseable legacy rows using the approved schema without either:
1. Fabricating synthetic identity (e.g., artificially setting `underlying = symbol`).
2. Weakening the governance by modifying the `ORDER BY` clause to include `symbol`.

Per the explicit instructions, I have STOPPED and am reporting this architectural conflict rather than inventing a workaround.

## 4. ReplacingMergeTree Collision Analysis
As detailed above, deduplication collisions are structurally guaranteed by the exclusion of `symbol` from the `ORDER BY` tuple for any unparseable rows sharing a timestamp. 

## 5. Test Evidence
- **UNIT TESTS:** PASS (All P4-06 tests pass, including the rigorous symbol parser unit tests)

## 6. Build/Vet Evidence
- `go build ./...`: PASS
- `go vet ./...`: PASS

## 7. Environment Limitations
- **INTEGRATION TESTS:** ENVIRONMENT BLOCKED (`core/database` tests fail solely due to the missing PostgreSQL/ClickHouse instances).

## 8. Files Changed
- `Backend/migrations/clickhouse/000003_update_option_greeks_schema.up.sql`

## 9. Regression Verification
The `SpotCache`, `GreeksWorker`, and Option Parser were not modified during this step. No regressions were introduced.

## FINAL VERDICT
**STATUS:** P4-06 MIGRATION CORRECTION = BLOCKED
**READINESS:** P4-07 READINESS = BLOCKED

The migration correction is completely blocked pending governance resolution for the unparseable row deduplication collision.
