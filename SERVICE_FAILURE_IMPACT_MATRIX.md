# Service Failure Impact Matrix

This matrix defines the blast radius and system degradation behavior if a specific logical service or infrastructure component fails.

| Failing Component | Affected Features | Remaining Operational Features | Retry / Fallback Behavior |
|---|---|---|---|
| **Core API Service** | Login, Registration, Watchlist CRUD, Strategy CRUD | Established WebSockets remain open but tickets cannot be generated. | Next.js displays offline state. Retries on standard HTTP exponential backoff. |
| **Realtime Hub Service** | Live market data ticks, realtime order updates | Core API (Watchlist, Strategy creation, Portfolio snapshot) | Client WebSocket SDK automatically attempts reconnection with binary exponential backoff (Max 5 attempts). |
| **Worker Service** | Scheduled analytics, email notifications | Core API, Realtime Hub, active Trading | Asynq automatically retries failed jobs. gocron skips missed ticks. |
| **Market Data Service** | Option Chains, Screener queries, Live Quotes | Core API, Trading execution | Client queries fallback to cached snapshots if available. Realtime Hub degrades gracefully. |
| **Broker Integration Service** | Order placement, Portfolio syncing | Market Data, Strategy Builder, Watchlists | Circuit breakers trigger on upstream broker timeouts to prevent thread starvation. |
| **PostgreSQL** | All transactional state (Auth, Portfolios) | Market Data streaming (if cached), WebSocket broadcasts | Hard failure for Core API. Services halt writes. |
| **ClickHouse** | Analytics, Screener, Historical Charts | Core API, Trading, Realtime Quotes | Market Data service returns degraded state or cache. |
| **Redis** | WebSocket tickets, Pub/Sub, Asynq | Core API (DB reads only) | Critical failure for Realtime Hub and Worker execution. Caches bypass to DB. |
