# P4-04 Forensic Discovery Report

## 1. Executive Summary
This report analyzes the repository state to define the precise scope of MarketPulse Pro Phase 4, Milestone 4 (P4-04). Based on the `PHASE4_ARCHITECTURE_ROADMAP.md`, P4-04 is officially defined as the **Market Data Ingest Foundation**. It focuses exclusively on establishing the ClickHouse schema, connection lifecycle, and backend data-ingestion mechanisms (internal webhook and memory-buffered batching) to safely capture and store incoming market ticks. No live broker integration, fake market data generation, or chart libraries are authorized.

## 2. Actual P4-04 Scope
**P4-04 ACTUAL GOVERNED SCOPE:** Market Data Ingest Foundation
- **Domain:** Market Data (Tick Ingestion & Storage)
- **User-facing capability:** None (Backend infrastructure only). Existing UIs remain in a "Deferred / Integration Pending" state.
- **Backend capability:** 
  1. ClickHouse connection lifecycle enhancement (migrations support using `golang-migrate`).
  2. ClickHouse schemas for ticks and option greeks.
  3. Internal `marketdata` repository for ClickHouse inserts.
  4. Internal `/api/v1/internal/ingest/ticks` webhook/handler to simulate future broker adapters pushing data into the system.
- **Frontend capability:** No changes. Chart libraries remain blocked (`[BLOCKED — DECISION REQUIRED]`).
- **Database requirements:** ClickHouse. PostgreSQL is entirely untouched in this phase.
- **Service role ownership:** Market Data Role.
- **Deferred dependencies:** Live broker feeds (P4-07/08), Redis Pub/Sub distribution (P4-05), Option Chain Greeks calculations (P4-06).

## 3. Repository Evidence
- **Backend:** `Backend/internal/core/clickhouse/clickhouse.go` exists and connects successfully. `github.com/golang-migrate/migrate/v4` v4.19.1 is present and natively supports ClickHouse.
- **Models:** `Backend/internal/modules/marketdata/models/market.go` defines `MarketTick` and `OptionGreeks`.
- **Frontend:** `/trade` and `/analyse` exist but explicitly defer Option Chain and Charting to future phases. `ChartContainer` explicitly blocks chart library usage.

## 4. Traceability Matrix

| Requirement | REQ | SPEC | IMPL | Evidence | Current State | P4-04 Action |
|-------------|-----|------|------|----------|---------------|--------------|
| Market Data Ingest Foundation | N/A | `NO DEDICATED SPEC` | `NO DEDICATED IMPL` | `PHASE4_ARCHITECTURE_ROADMAP.md (P4-04)` | Models & Connection exist | Establish Ingest Pipeline (DIRECTLY GOVERNED) |

*Note: Previous mappings to SPEC_005/006 were TRACEABILITY ERRORS and have been purged, as those SPECs apply to legacy/deferred historical data lakes and legacy pricing engines not part of this exact roadmap scope.*

## 5. ClickHouse Architecture & Schema (Evidence-Based)
Based on `MarketTick` (symbol, ltp, open, high, low, close, volume, timestamp) and time-series query requirements (filtering by symbol, ranges by time):
- **Table:** `market_ticks`
- **Engine:** `ReplacingMergeTree(timestamp)` to deduplicate identical ticks inherently without complex application logic.
- **PARTITION BY:** `toYYYYMM(timestamp)` (Standard for ticks, assuming high frequency but keeping partitions reasonably sized per month).
- **ORDER BY:** `(symbol, timestamp)` for optimal primary index performance when querying specific instrument timelines.
- **TTL:** `BLOCKED — DECISION REQUIRED`. No data retention TTL is applied as it is not explicitly governed in the current artifacts.

## 6. Realtime & Background Processing Audit
- **WebSocket:** Excluded from P4-04 (Deferred to P4-05).
- **Background Processing Decision:** The ingestion pipeline will use **Go buffered channels + batch worker**. 
  - *Justification:* ClickHouse requires large batch inserts (10k+ rows) for performance. Asynq stores every job in Redis; using Asynq for high-frequency tick streams would catastrophically overwhelm Redis. Since Kafka/NATS are explicitly prohibited, in-memory Go batching with a periodic flush is the only scalable and governed architectural choice.

## 7. Ingestion Pipeline Failure Semantics
- **Batch Size:** 1,000 ticks.
- **Flush Interval:** 1 second.
- **Channel Capacity:** 10,000 (allows 10 seconds of buffering at 1k ticks/sec).
- **Overflow Behavior:** If channel is full, the webhook returns `503 Service Unavailable` (backpressure to the future broker adapter).
- **Graceful Shutdown:** The context cancellation triggers a final flush of the buffer before the server terminates.
- **Partial Failure / ClickHouse Unavailable:** Ticks are dropped and logged as errors. ClickHouse relies on the upstream provider for truth. 

## 8. Internal Ingest Security & Router Isolation
- **Authentication:** Normal user JWTs are REJECTED. The endpoint must use an explicit `InternalAuthMiddleware` that requires an `X-Internal-API-Key` header matching an injected environment variable (`INTERNAL_API_KEY`). Missing/invalid keys return `401 Unauthorized`.
- **Router Isolation:** Registered on a completely separate Gin router group (`/api/v1/internal`) that is strictly isolated from public endpoints. The middleware is applied strictly to this group.

## 9. Frontend Audit
- **Current UI:** `trade`, `dashboard`, `analyse` are all correctly stubbed.
- **Fake Market Data:** Zero fake data will be inserted or generated. The API is a pure boundary.

## 10. Exact File Inventory
- **CREATE:** `Backend/migrations/clickhouse/000001_create_market_ticks.up.sql` / `.down.sql` (Schema implementation).
- **CREATE:** `Backend/internal/middleware/internal_auth.go` (Mandatory security).
- **CREATE:** `Backend/internal/modules/marketdata/repositories/clickhouse_repository.go` (ClickHouse inserts).
- **CREATE:** `Backend/internal/modules/marketdata/services/ingest_service.go` (Go channel batching).
- **CREATE:** `Backend/internal/modules/marketdata/handlers/ingest_handler.go` (Internal webhook).
- **MODIFY:** `Backend/internal/core/database/migrate.go` (To conditionally support `clickhouse://` URLs using existing `golang-migrate`).
- **MODIFY:** `Backend/internal/routes/routes.go` (Register `/api/v1/internal/*` group).
- **MODIFY:** `Backend/internal/config/config.go` (Add `InternalAPIKey`).

## 11. Final Readiness Matrix

| Area | Status | Evidence | Blocker |
|------|--------|----------|---------|
| Governance | GREEN | Phase 4 Roadmap (P4-04) | None |
| Traceability | GREEN | Explicitly mapped to Roadmap | None |
| Database | GREEN | PostgreSQL safe, ClickHouse defined | None |
| Security | GREEN | Internal Auth Middleware defined | None |
| Background | GREEN | Memory batching justified | None |
| Frontend | GREEN | Excluded | None |

**P4-04 DISCOVERY CORRECTION = COMPLETE**
