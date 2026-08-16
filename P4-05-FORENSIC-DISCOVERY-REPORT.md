# P4-05 Forensic Discovery Report: Realtime Market Data Distribution

## 1. Objective and Boundary Validation
The goal of this discovery is to establish the current state of the repository regarding realtime distribution and map it strictly to existing governance. 

**P4-04 Boundary:** P4-04 successfully implemented the `MarketDataIngestService`, memory channels, batch processing, and ClickHouse (`ReplacingMergeTree(timestamp)`) persistence for `MarketTick` and `OptionGreeks`. It exposes the internal ingestion endpoint. P4-04 ends once data is successfully persisted to ClickHouse. It does NOT currently emit events to Redis or WebSockets.

## 2. Realtime Hub Role Audit
- **Physical Existence**: The Realtime Hub Role *physically exists* within the Modular Monolith under `Backend/internal/websocket`.
- **WebSocket Foundation**: Implemented via Gorilla WebSocket.
- **Client Registry**: Exists (`hub.clients`).
- **Connection Lifecycle**: `ReadPump` and `WritePump` manage connections, Ping/Pong, and deadlines successfully.
- **Auth**: Uses a one-time ticket system (`/auth/ws-ticket` -> `/ws?ticket=XXX`) mapped to authenticated JWT sessions.
- **Broadcast**: Exists (`hub.Broadcast`), utilizing a channel-based subscription map.
- **Defects/Gaps**: 
  1. `ReadPump` explicitly drops all Client-to-Server (C2S) messages.
  2. Because C2S messages are dropped, there is NO way for clients to subscribe to channels.
  3. `hub.Subscribe()` is *never* called in production code. Thus, no client ever receives any broadcast.

## 3. Redis / PubSub Audit
- **Existence**: Redis Pub/Sub infrastructure physically exists (`Backend/internal/infrastructure/pubsub`).
- **Integration**: `bridge.go` successfully links Redis Pub/Sub to the WebSocket Hub. It listens to `mp:events:system` and broadcasts payloads to the corresponding WebSocket channel.
- **Governance**: `PHASE4_ARCHITECTURE_ROADMAP.md` explicitly approves Redis Pub/Sub for P4-05: *"Map ClickHouse aggregates to Redis Pub/Sub (`market.prices`, `market.news`)."*

## 4. Event Catalog & Data Semantics
- **System Events**: `mp:events:system` -> `system.health` is APPROVED and handled by the frontend.
- **Market Events**: `market.prices` payload is now formally APPROVED via `Architecture/P4-05-EVENT-CONTRACT-DECISION.md`.
- **Finding**: `market.news` is deferred due to insufficient repository evidence.

## 5. Subscription Model
- **Finding**: The subscription model is now explicitly governed as **Symbol-specific channel-based routing**.
- **Status**: SUBSCRIPTION MODEL = APPROVED via `Architecture/P4-05-WEBSOCKET-SUBSCRIPTION-DECISION.md`.

## 6. Frontend Audit
- **Implementation**: `Frontend/src/lib/ws.ts` provides a robust client with exponential backoff reconnects, heartbeat handling, and client-side channel routing.
- **Usage**: The Dashboard currently subscribes to `mp:events:system` client-side, but receives nothing because the backend drops subscriptions.

## 7. Security & Architecture Vulnerabilities
- CORS in `upgrade.go` currently allows all origins (`return true`).
- Goroutine leaks are prevented by select default cases (dropping messages for slow clients).
- Redis failure semantics are not explicitly handled beyond basic disconnects.

## 8. Exact Implementation Inventory
**Files to CREATE (Eventually):**
- Event payload schema definitions in Governance.
- Market Data Redis publisher service.

**Files to MODIFY (Eventually):**
- `Backend/internal/websocket/client_conn.go` (to process C2S subscribe messages).
- `Backend/internal/websocket/bridge.go` (to subscribe to new market channels).
- `Frontend/src/lib/ws.ts` (to send subscribe commands).

**Files that MUST REMAIN UNTOUCHED:**
- P4-04 Ingest Service.
- Broker Integrations.
- Strategy Business Logic.
