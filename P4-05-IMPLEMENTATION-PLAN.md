# P4-05 Implementation Plan: Realtime Market Data Distribution

## 1. Goal
Establish the Realtime Market Data Distribution layer, bridging the ClickHouse market data ingestion pipeline (P4-04) to the WebSocket Realtime Hub for live frontend consumption.

## 2. Scope
- Definition of WebSocket C2S (Client-to-Server) subscription protocol.
- Registration of WebSocket clients to dynamic Redis Pub/Sub channels.
- Emitting market data events from the ingestion layer to Redis Pub/Sub.
- Bridging Redis Pub/Sub channels to active WebSocket clients.

## 3. Non-goals
- Broker API integration (P4-07/08).
- Option Chain/Greeks calculations (P4-06).
- Frontend visual redesigns.
- Kafka/NATS migration.

## 4. Existing Architecture
- **Market Data Ingest (P4-04)**: Processes ticks and batches them to ClickHouse.
- **WebSocket Hub**: Handles concurrent client connections, authenticates via one-time tickets, manages Ping/Pong, but currently drops all C2S messages.
- **Redis Pub/Sub**: Configured and bridged to WebSocket (`bridge.go`), currently only listening to `mp:events:system`.

## 5. P4-04 Boundary
P4-04 ends at the successful ClickHouse insertion within `IngestService`. P4-05 will hook into this layer to concurrently publish the ingested ticks to Redis Pub/Sub without blocking the ClickHouse batching pipeline.

## 6. Realtime Hub Architecture
The Realtime Hub is physically present within the monolithic backend (`Backend/internal/websocket`). It uses Gorilla WebSockets. P4-05 will expand `client_conn.go`'s `ReadPump` to parse C2S JSON commands (e.g., `{"action": "subscribe", "channel": "market.prices"}`) and invoke `hub.Subscribe()`.

## 7. WebSocket Architecture
- **Authentication**: JWT -> `/auth/ws-ticket` -> `/ws?ticket=XXX` (Already implemented).
- **Origin Check**: Currently `return true`. Must be tightened for production.
- **Client Registry**: Managed by `Hub`.
- **Connections**: Handled by `ReadPump` and `WritePump`. 

## 8. Event Architecture
The `market.prices` payload schema is formally approved via `P4-05-EVENT-CONTRACT-DECISION.md`. It strictly mirrors the `models.MarketTick` structure wrapped in a standard JSON envelope. `market.news` is explicitly deferred.

## 9. Internal Transport Architecture
- **Mechanism**: Redis Pub/Sub. 
- **Flow**: `IngestService` -> Redis Publish (`market.prices.{symbol}`) -> `pubsub.Subscriber` (`bridge.go`) -> `hub.Broadcast()` -> WebSocket Client.

## 10. Authentication
Broker authentication is excluded. Application authentication is already handled securely via the one-time ticket system mapped to JWTs.

## 11. Authorization
All channels under `market.prices.*` are considered PUBLIC for any authenticated user. Fine-grained premium authorization is deferred.

## 12. Fan-out / Backpressure
- **Slow Clients**: `hub.Broadcast` uses non-blocking channel selects (`default: drop`). This provides At-Most-Once delivery and protects the server from OOM.
- **Redis to Hub**: `pubsub` channels buffer messages.

## 13. Failure Semantics
- **Best Effort**: Realtime data is transient. If Redis fails, or WebSockets drop, the data is lost for the client, but historically preserved in ClickHouse. Reconnecting clients must query the REST API for snapshots.

## 14. Role-based Deployment
The Realtime Hub runs co-located with the Core API. No microservice extraction is occurring in P4-05.

## 15. Observability
Use existing `zap.Logger` to track dropped messages in `hub.Broadcast()`, total active clients, and Redis Pub/Sub disconnects.

## 16. Security
- Tighten `CheckOrigin` in `upgrade.go` via environment configuration.
- Message size limits already exist (`maxMessageSize = 512`).

## 17. Exact Files
**Files to Modify:**
- `Backend/internal/websocket/client_conn.go` (Add C2S command parsing for JSON `subscribe`/`unsubscribe` actions).
- `Backend/internal/websocket/bridge.go` (Update to use `PSUBSCRIBE "market.prices.*"`).
- `Backend/internal/modules/marketdata/services/ingest_service.go` (Publish ticks asynchronously to Redis `market.prices.{symbol}`).

## 18. Implementation Units
1. Implement WebSocket C2S Subscription logic.
2. Hook `IngestService` to Redis Publisher.
3. Expand `Bridge` to dynamically route Redis wildcard events to WebSocket subscribers.

## 19. Testing Strategy
- **Unit Tests**: Mock Redis publisher to verify `IngestService` fan-out.
- **WebSocket Tests**: Send a `subscribe` command, verify `hub.ChannelCount`. 
- **Integration**: Verify dropping slow client messages.

## 20. Regression Strategy
- Ensure `mp:events:system` health checks continue to flow to the frontend.
- Ensure ClickHouse batch sizes and flush intervals are not delayed by Redis publishing.

## 21. Traceability
Maps to Roadmap `P4-05`. Governed by `P4-05-EVENT-CONTRACT-DECISION.md` and `P4-05-WEBSOCKET-SUBSCRIPTION-DECISION.md`.

## 22. Pending Decisions (Blockers)
- **None.** All governance gaps have been resolved.

## 23. Risks
- Synchronous Redis publishing inside `IngestService` could bottleneck ClickHouse ingestion. Must be asynchronous (e.g., fire-and-forget goroutine or buffered queue).

## 24. Readiness Matrix
| Decision | Status |
|---|---|
| `market.prices` schema | APPROVED |
| `market.news` schema | DEFERRED |
| Event envelope | APPROVED |
| Redis channel mapping | APPROVED (`market.prices.{symbol}`) |
| C2S subscribe protocol | APPROVED |
| C2S unsubscribe protocol | APPROVED |
| Subscription scope | APPROVED (Per-connection) |
| Authentication | APPROVED (One-time ticket) |
| Authorization | APPROVED (Public to authenticated users) |
| Reconnect semantics | APPROVED (Client-driven resubscribe) |
| Failure semantics | APPROVED (Best effort, drop on full buffer) |
| P4-04 boundary | APPROVED (Asynchronous publish post-ingest) |

## 25. Final Verdict
**P4-05 READINESS = READY**
