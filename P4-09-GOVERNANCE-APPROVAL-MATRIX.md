# P4-09 GOVERNANCE APPROVAL MATRIX

**Status:** [APPROVED]
**Date:** 2026-08-10

| Decision | Project Owner Decision | Status | Blocking? |
|---|---|---|---|
| **1A Quantity aggregation** | net_quantity = SUM(provider_position.quantity) | APPROVED | NO |
| **1B Long positions** | long_quantity = max(net_quantity, 0) | APPROVED | NO |
| **1C Short positions** | short_quantity = abs(min(net_quantity, 0)) | APPROVED | NO |
| **1D Zero quantity** | MUST NOT appear in active list (audit only) | APPROVED | NO |
| **1E Average price** | SUM(quantity_i * average_price_i) / SUM(quantity_i) (Same direction, Null if zero) | APPROVED | NO |
| **1F Unrealized P&L** | SUM(provider_unrealized_pnl) (Do not fabricate) | APPROVED | NO |
| **1G Realized P&L** | SUM(provider_realized_pnl) (Independent from Unrealized) | APPROVED | NO |
| **1H Duplicate instruments**| Canonical P4-04 InternalSymbol is the identity | APPROVED | NO |
| **1I Conflicting mappings** | P4-04 remains authoritative. No guessing permitted. | APPROVED | NO |
| **2A Redis key format** | mp:broker:portfolio:{user_id} and :{provider} | APPROVED | NO |
| **2B Cache TTL** | 5 seconds (Explicit PO Decision) | APPROVED | NO |
| **2C-K Cache behavior** | Miss = Parallel Fetch, Timeout = Stale Cache, Ephemeral | APPROVED | NO |
| **3 Partial failure** | 200 OK, partial_success, Failed Providers returned/Stale allowed | APPROVED | NO |
| **4 Realtime/WebSocket** | position.update, Sequence Monotonic, REST reconnect | APPROVED | NO |
| **5 Frontend ownership** | P4-09 FRONTEND SCOPE = NONE (Backend Only) | APPROVED | NO |
| **6 Fan-out/concurrency** | Max 3, 5000ms Timeout, errgroup, No Retries | APPROVED | NO |
| **7 Test governance** | Full test matrix mandated. | APPROVED | NO |
| **8 Security** | Strict User/Session/Redis isolation. P4-07 Compatibility | APPROVED | NO |

## Final Gate
All implementation-critical P4-09 governance parameters are explicitly approved.
**P4-09 GOVERNANCE = APPROVED**
**P4-09 IMPLEMENTATION = AUTHORIZED**
**P4-09 READINESS = READY FOR IMPLEMENTATION**
