# P4-05 WebSocket Subscription Model Decision

## Status
APPROVED

## Evidence Reviewed
- `Backend/internal/websocket/hub.go` (Natively supports `channel string` via `h.subscriptions[channel][clientID]`).
- `Backend/internal/websocket/bridge.go` (Natively broadcasts to exact matching channels).
- `Frontend/src/lib/ws.ts` (Natively routes messages via `channel` in JSON payload).
- `Architecture/P4-05-EVENT-CONTRACT-DECISION.md` (Locks Redis channels to `market.prices.{symbol}`).

## Decision
The authoritative Client-to-Server (C2S) WebSocket subscription model is **Symbol-specific channel-based routing**.

## Connection Authentication
The existing architecture handles WebSocket authentication via a one-time ticket:
1. Client requests ticket via `/auth/ws-ticket` (HTTP with JWT).
2. Client connects to `/ws?ticket=XXX`.
3. Backend upgrades and registers the client ID.

This mechanism is APPROVED and sufficient for P4-05. No secondary authentication payload is required in the WebSocket stream.

## C2S Protocol
Clients interact with the Realtime Hub exclusively by passing JSON action payloads.

### subscribe
Requests delivery of a specific data channel.
```json
{
  "action": "subscribe",
  "channel": "market.prices.NIFTY"
}
```

### unsubscribe
Stops delivery of a specific data channel.
```json
{
  "action": "unsubscribe",
  "channel": "market.prices.NIFTY"
}
```

## Subscription Scope
- Subscriptions are strictly **per WebSocket connection**. 
- If a client disconnects, the Hub automatically purges the client from all `h.subscriptions` (already natively supported by `hub.Unregister`).

## Channel Naming
Channels map exactly 1:1 with Redis channels (e.g., `market.prices.NIFTY`).

## Symbol Routing
When a client sends a `subscribe` command, the `ClientConn.ReadPump` parses the JSON, validates the action, and synchronously invokes `hub.Subscribe(c.id, req.Channel)`. No client-side multiplexing logic is needed beyond exact string matching.

## Validation
- **Missing/Invalid Fields**: The server silently ignores malformed JSON payloads.
- **Unknown Action**: Actions other than `subscribe` or `unsubscribe` are silently ignored.
- **Duplicate Subscription**: `hub.Subscribe` is idempotent (uses a boolean map). Duplicate requests are safe and ignored.
- **Unknown Channel**: Clients can subscribe to channels that do not yet exist or have no traffic. The Hub simply records the intent.

## Error Semantics
The Realtime Hub operates strictly on a fire-and-forget, best-effort basis.
- The server does NOT send ACK/NACK responses for `subscribe` actions.
- Malformed payloads do NOT drop the WebSocket connection.
- A `503` or Redis failure upstream results in silence on the channel; the WebSocket connection remains healthy.

## Reconnect Semantics
- Subscriptions are **NOT** automatically restored by the backend.
- Upon reconnection, the frontend (`Frontend/src/lib/ws.ts`) is strictly responsible for re-transmitting all active `subscribe` commands across the new connection.

## Authorization
- All channels under `market.prices.*` are considered PUBLIC for any authenticated user. 
- Fine-grained premium authorization is deferred to a future phase and is NOT required for P4-05 readiness.

## Backpressure
- Handled via `hub.Broadcast()`'s default select case. If a client's 256-message buffer is full, the server drops the message rather than blocking the hub or degrading other clients.

## Security
- `maxMessageSize` is already restricted to 512 bytes, mitigating DoS via oversized C2S payloads.
- **Origin Validation**: `CheckOrigin` in `upgrade.go` currently `return true`. This MUST be restricted via an environment variable (e.g., `ALLOWED_ORIGINS`) during the P4-05 implementation to prevent Cross-Site WebSocket Hijacking (CSWSH).

## Rejected Alternatives
1. **Multi-symbol subscribe (`symbols: ["NIFTY", "BANKNIFTY"]`)**: Rejected to minimize backend array-iteration complexity and to keep the C2S protocol tightly coupled with `hub.go`'s native single-channel string handling.
2. **Watchlist-based server subscriptions**: Rejected. Resolving a watchlist ID to a set of symbols server-side during the websocket read loop introduces database IO latency into the realtime path. Watchlists must be resolved to symbols by the frontend REST API, followed by explicit symbol subscriptions over WebSocket.

## P4-04 Compatibility
The WebSocket Hub acts strictly as a downstream consumer of the Market Data Ingest layer. It exerts zero backpressure on P4-04 pipelines.

## P4-05 Implementation Consequences
- `client_conn.go` must be updated to parse C2S JSON instead of dropping all reads.
- `bridge.go` must be updated to use Redis `PSUBSCRIBE "market.prices.*"` so it dynamically captures all incoming symbol ticks without needing to resync subscriptions with Redis.
