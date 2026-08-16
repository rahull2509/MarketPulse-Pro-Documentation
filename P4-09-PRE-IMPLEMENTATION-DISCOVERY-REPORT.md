# P4-09 PRE-IMPLEMENTATION DISCOVERY REPORT

**Date:** 2026-08-10
**Phase:** P4-09 - Trading & Portfolio Aggregation
**Status:** [BLOCKED]

## 1. P4-09 Scope
- **Phase Name:** Trading & Portfolio Aggregation
- **Implementation Scope:** Expose execution APIs and sync upstream positions to the frontend (as per `PHASE4_ARCHITECTURE_ROADMAP.md`).
- **Existing Code:** REST routes for `/broker/orders`, `/broker/positions`, `/broker/holdings` were already scaffolded during P4-07/P4-08.
- **Excluded Scope:** Ungoverned.

## 2. P4-09 Objective
To expose execution APIs and aggregate/sync upstream positions to the frontend.

## 3. Dependencies
- **P4-07:** Core Broker Facade and Data Models (Closed/Passed).
- **P4-08:** Concrete Provider Implementations (Closed/Passed).
- **P4-09** depends on both layers functioning correctly to aggregate across providers.

## 4. Authoritative Governance Documents
**None.** No P4-09 specific governance documents, decision records, or API contracts exist in the repository.

## 5. Decision Matrix
| Decision | Required Value | Evidence | Status |
|---|---|---|---|
| Cross-Broker Aggregation API Contract | REST/WebSocket Definition | None | UNKNOWN |
| Position Caching Strategy | Database/Redis/Live | None | UNKNOWN |
| Realtime Position Sync Mechanism | Redis Channels/Events | None | UNKNOWN |
| Aggregation Failure Semantics | Partial vs Full Failure | None | UNKNOWN |

## 6. Existing Repository Architecture
The current architecture contains fully functional, tested components from P4-07/P4-08 that P4-09 can reuse:
- **Interfaces:** `BrokerManager`, `BrokerAdapter`
- **Models:** `BrokerOrder`, `BrokerPosition`, `BrokerHolding`
- **Routes:** `GET /broker/positions`, `GET /broker/holdings`, `POST /broker/orders`, `PUT /broker/orders/:id`, `DELETE /broker/orders/:id`, `GET /broker/orders`
- **Handlers:** `BrokerHandler` is mapped and protected via JWT.

## 7. Database Impact
- **Required:** UNKNOWN. It is not governed whether `BrokerPosition` or `BrokerHolding` should be cached/persisted in PostgreSQL (requiring migrations/schema) or always fetched live.

## 8. API Impact
- **Required:** UNKNOWN. Existing endpoints query a single provider (`?provider=zerodha`). It is not governed whether a unified `/broker/portfolio` cross-provider aggregation endpoint is required, or if the frontend is responsible for parallel queries.

## 9. Security Impact
- **Required:** UNKNOWN. Aggregation across multiple active provider sessions requires clear authorization bounds and secret handling, especially if caching balances.

## 10. Redis/WebSocket Impact
- **Required:** UNKNOWN. "Sync upstream positions to the frontend" strongly implies realtime push capabilities (WebSocket/Redis PubSub), but no event payloads, channels, or lifecycle events are governed for position syncs.

## 11. Failure/Resilience Requirements
- **Required:** UNKNOWN. If a user queries an aggregated portfolio and 1 out of 3 connected brokers times out, the degraded mode/partial failure semantics are entirely ungoverned.

## 12. Testing Requirements
When unblocked, P4-09 will require:
- Unit tests for aggregation logic.
- Integration tests for partial provider failures.
- Concurrency tests for live position syncing.

## 13. P4-07 Compatibility
Any P4-09 implementation must not modify the `BrokerManager` interface or `BrokerSession` state transitions governed under P4-07.

## 14. P4-08 Compatibility
Any P4-09 implementation must respect the P4-08 rate limit CircuitBreakers and must not blindly retry provider fetch calls if rate-limited.

## 15. Implementation Risks
- Proceeding without explicit cross-provider error boundaries risks catastrophic frontend failure if one broker's API degrades.
- Implementing database caching without governed refresh cadences risks stale data rendering.

## 16. Blocking Unknowns/Pending Decisions
1. **API Contract:** Single-provider vs Multi-provider aggregation.
2. **Data Storage:** Live fetch vs Persistent cache for holdings/positions.
3. **Realtime Sync:** Definition of WebSocket event payloads for position updates.
4. **Resilience:** Partial failure behavior during cross-provider fetches.

## 17. Required Project Owner Approvals
The Project Owner must explicitly propose and approve the P4-09 Technical Governance specifications before implementation can begin.

## 18. Recommended Implementation Sequence
*(Blocked until Governance is ratified)*
1. Define Portfolio Aggregation API Contract.
2. Define Position Caching Strategy.
3. Define Realtime Sync WebSocket Contract.

---

## FINAL VERDICT

P4-09 DISCOVERY = COMPLETE
P4-09 GOVERNANCE = BLOCKED
P4-09 IMPLEMENTATION = NOT STARTED
P4-09 READINESS = BLOCKED
