# Data Ownership Matrix

This document defines explicit ownership for every database and cache boundary in MarketPulse Pro. Multiple services are strictly prohibited from directly writing to the same database table. Cross-domain data mutations must occur via synchronous APIs or asynchronous events.

## PostgreSQL Ownership (Transactional Store)

| Schema / Table | Logical Owner Service | Read Path | Write Path | Description |
|---|---|---|---|---|
| `users` | Core API (Auth) | Core API | Core API | User identities and profile metadata. |
| `watchlists` | Core API (Watchlist) | Core API | Core API | User-defined watchlists. |
| `watchlist_items` | Core API (Watchlist) | Core API | Core API | Individual instruments in a watchlist. |
| `strategies` | Core API (Strategy) | Core API | Core API | Custom option strategies created by users. |
| `strategy_legs` | Core API (Strategy) | Core API | Core API | Individual legs of an option strategy. |
| `broker_sessions` | Broker Integration | Broker Integration | Broker Integration | Upstream authentication tokens (Zerodha, etc.). |
| `orders` | Broker Integration | Broker Integration | Broker Integration | Master order record mapping internal IDs to broker IDs. |
| `positions` | Broker Integration | Broker Integration | Broker Integration | Flattened position summary synced from broker. |

## ClickHouse Ownership (Analytical Store)

| Dataset | Logical Owner Service | Read Path | Write Path | Retention Consideration |
|---|---|---|---|---|
| `market_ticks` | Market Data | Market Data | Market Data (Ingest) | High-volume time-series, partitioned by day. |
| `ohlcv_candles` | Market Data | Market Data, Core API | Market Data (Rollup) | Aggregated price history, kept indefinitely. |
| `option_greeks` | Market Data | Market Data, Worker | Worker (Compute) | Calculated greeks, high churn, temporary value. |

## Redis Ownership (Cache & State)

| Namespace | Owner Service | Readers | Writers | TTL | Purpose |
|---|---|---|---|---|---|
| `ws:ticket:*` | Core API | Realtime Hub | Core API | 30s | Single-use WebSocket connection tickets (GETDEL). |
| `cache:user:*` | Core API | Core API | Core API | Varies | Caching user profile or preferences. |
| `cache:strategy:*`| Core API | Core API | Core API | Varies | Caching complex strategy evaluations. |
| `mp:events:*` | (Event Bus) | Realtime Hub | All Services | N/A | Pub/Sub topic namespace. |
| `asynq:*` | Worker | Worker | Core API, Scheduler | N/A | System queues for Asynq background jobs. |
