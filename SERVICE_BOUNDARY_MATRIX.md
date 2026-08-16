# Service Boundary Matrix

This matrix defines the logical bounded contexts for the MarketPulse Pro Modular Monolith architecture.

| Service Role | Bounded Context | Responsibilities | APIs | Own DB | Own Cache | Publishes | Consumes | External Dependencies | Scaling Profile | Failure Isolation | Status |
|---|---|---|---|---|---|---|---|---|---|---|---|
| **Core API** | Auth, User, Strategy, Watchlist | Handles REST API CRUD, JWT issuance, standard business logic | `POST /auth/*`, `GET /user/*`, `GET /strategies/*`, `GET /watchlists/*` | PostgreSQL | Redis `cache:*` | - | - | - | Medium | Fails independent of Realtime | REQUIRED |
| **Realtime Hub** | WebSocket Connections, Live Streams | Maintains WS connections, validates tickets, pushes Redis Pub/Sub events | `GET /ws` | None | Redis `ws:ticket:*` | - | `mp:events:*` | - | Very High | Fails independent of Core API | REQUIRED |
| **Worker** | Background Processing, Notifications | Executes Asynq tasks, cron jobs, analytics pre-calculations | None | PostgreSQL, ClickHouse | Redis `asynq:*` | `mp:events:system` | - | Third-party SMTP/SMS (Future) | High | Protects API from heavy compute | REQUIRED |
| **Market Data** | Quotes, Options, Screener | Tick data ingestion, Greeks calculation, ClickHouse reads | `GET /marketdata/*` | ClickHouse | Redis | `mp:events:marketdata` (PROPOSED) | - | External Feed Providers | High | Protects transactional DB | RECOMMENDED |
| **Broker Integration** | Orders, Portfolios, Positions | Multi-broker unified facade, session management | `POST /orders/*`, `GET /portfolio/*` | PostgreSQL (Session/Orders) | Redis | `mp:events:orders` (PROPOSED) | - | Zerodha, Angel One, ICICI Direct APIs | Medium | Protects API from upstream timeouts | RECOMMENDED |
| **API Gateway / BFF** | Edge Routing, Aggregation | Next.js Server routes handling edge security, SSR aggregation | Next.js App Router | None | Next.js Cache | - | - | Core API | Medium | - | REQUIRED (Next.js native) |
