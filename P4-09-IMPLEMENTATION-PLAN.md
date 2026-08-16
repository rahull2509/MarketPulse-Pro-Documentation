# P4-09 IMPLEMENTATION PLAN

**Status:** [APPROVED - READY FOR IMPLEMENTATION]
**Date:** 2026-08-10

## 1. Objective
Define the implementation and testing sequence for Phase 4-09 (Trading & Portfolio Aggregation).

## 2. Testing Governance (APPROVED)
The implementation MUST include tests for the following matrix (implementation-time requirements):
- single provider
- multiple providers
- duplicate instruments
- conflicting mappings
- long positions
- short positions
- zero quantity
- average price
- realized P&L
- unrealized P&L
- provider timeout
- provider 429
- partial failure
- stale cache
- cache miss
- Redis failure
- empty broker state
- concurrent aggregation
- per-user single-flight
- realtime publication
- WebSocket reconnect
- duplicate events
- stale events
- cross-user isolation

## 3. Security Boundaries (APPROVED)
- JWT identity is the sole user identity source.
- Redis keys MUST contain `user_id`.
- Broker sessions MUST never cross user boundaries.
- WebSocket events MUST be user-scoped.
- Broker access tokens MUST never be in Redis portfolio payloads or exposed to frontend.
- No cross-user aggregation permitted.
- Existing AES-256-GCM token storage and OAuth semantics remain unchanged.

## 4. Execution Sequence
Implementation is AUTHORIZED. Development may proceed following the approved governance boundaries.
