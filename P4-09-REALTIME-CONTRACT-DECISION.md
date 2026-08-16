# P4-09 REALTIME CONTRACT DECISION

**Status:** [APPROVED]
**Date:** 2026-08-10

## 1. Objective
Define WebSocket payloads, events, and Redis channels.

## 2. Realtime Synchronization Governance (APPROVED)

- **Redis Channel:** `mp:events:portfolio`
- **Event Name:** `position.update`
- **WebSocket Event Name:** `position.update`
- **Payload Schema:**
```json
{
  "event": "position.update",
  "user_id": "<authenticated-user-id>",
  "provider": "<provider>",
  "internal_symbol": "<P4-04-internal-symbol>",
  "quantity": 0,
  "average_price": null,
  "unrealized_pnl": 0,
  "realized_pnl": 0,
  "snapshot_id": "<snapshot-id>",
  "snapshot_timestamp": "<RFC3339 timestamp>",
  "sequence": 0
}
```
*Note: Fields must be populated authoritatively. Do not fabricate values.*

- **Event Scoping:** Events strictly scoped to authenticated user. Clients MUST reject alien events.
- **4A. Event Ordering:** Monotonically increasing sequence. Older sequences are ignored.
- **4B. Duplicate Events:** Ignored if `user_id` + `snapshot_id` + `sequence` match.
- **4C. Stale Events:** Older sequences MUST NOT overwrite newer state.
- **4D. WebSocket Reconnect:** Client MUST obtain fresh portfolio via REST API. Missed WebSocket events cannot be reconstructed from stream.

**Status:** P4-09 REALTIME CONTRACT = APPROVED.
