# P4-04 Implementation Plan (Market Data Ingest Foundation)

## 1. Goal
Establish the backend foundation for Market Data Ingest strictly adhering to the P4-04 roadmap. This involves creating ClickHouse migrations, defining analytical schemas (Ticks, Greeks), and implementing high-throughput memory-batch ingestion mechanisms behind an isolated, securely authenticated internal webhook.

## 2. Scope Constraints
- **NO Frontend Changes:** The frontend remains exactly as it is (stubbed for charts/options).
- **NO Live Broker Data:** Broker adapters are governed by P4-07/08.
- **NO Fabricated DB Records:** Test data will not be inserted into the production database.
- **NO Redis Pub/Sub:** Realtime distribution is governed by P4-05.
- **NO Asynq for Ingestion:** Memory-buffered Go channels will be used instead to prevent Redis exhaustion.

## 3. Implementation Units

### P4-04-R1: ClickHouse Migration Infrastructure
**Goal:** Extend the application bootstrap to securely run ClickHouse migrations using `golang-migrate` alongside PostgreSQL.
**Files:**
- **MODIFY** `Backend/internal/core/database/migrate.go`: Introduce conditional logic to run ClickHouse migrations if the dialect is ClickHouse.
- **MODIFY** `Backend/internal/bootstrap/bootstrap.go`: Invoke ClickHouse migrations on startup.
- **CREATE** `Backend/migrations/clickhouse/000001_create_market_ticks.up.sql`: Create table using `ReplacingMergeTree(timestamp)` ordered by `(symbol, timestamp)` partitioned by `toYYYYMM(timestamp)`.
- **CREATE** `Backend/migrations/clickhouse/000002_create_option_greeks.up.sql`: Same engine/ordering for Option Greeks.

### P4-04-R2: Internal Security & Router Isolation
**Goal:** Guarantee that public users cannot invoke internal ingestion APIs.
**Files:**
- **MODIFY** `Backend/internal/config/config.go`: Add `InternalAPIKey` field.
- **CREATE** `Backend/internal/middleware/internal_auth.go`: Middleware rejecting any request lacking `X-Internal-API-Key` matching config. Returns `401`.
- **MODIFY** `Backend/internal/routes/routes.go`: Create `internalGroup := router.Group("/api/v1/internal")` and apply `InternalAuthMiddleware`.

### P4-04-R3: Market Data Repository Layer
**Goal:** Create the ClickHouse persistence logic.
**Files:**
- **CREATE** `Backend/internal/modules/marketdata/repositories/clickhouse_repository.go`
**Details:** Expose `InsertMarketTicks(ctx context.Context, ticks []models.MarketTick) error`. Uses `clickhouse-go` batch abstraction for fast native inserts.

### P4-04-R4: Ingestion Service (Memory Batching)
**Goal:** Process incoming ticks via a buffered channel to prevent database locking, flushing in optimal batches.
**Files:**
- **CREATE** `Backend/internal/modules/marketdata/services/ingest_service.go`
**Details:** 
- **Semantics:** 10k capacity channel. 1,000 tick batch size OR 1-second flush interval.
- **Backpressure:** Returns `503` immediately if the channel is full.
- **Graceful Shutdown:** `Stop()` closes channel and flushes remaining buffer.

### P4-04-R5: Webhook & Wiring
**Goal:** Expose the internal endpoint and wire the Fx module.
**Files:**
- **CREATE** `Backend/internal/modules/marketdata/handlers/ingest_handler.go`: Handles `POST /api/v1/internal/ingest/ticks`.
- **CREATE** `Backend/internal/modules/marketdata/module.go`: Fx wiring.

## 4. Testing Strategy
- **Backend Unit Tests:** Test the batching logic (`ingest_service_test.go`) utilizing in-memory fixtures. Ensure timer and capacity flushes behave correctly.
- **Security Tests:** Verify `/api/v1/internal/*` rejects missing/wrong keys.
- **Build:** `go build` and `npm run build` must pass.

## 5. Regression Checklist
- [ ] PostgreSQL migrations still execute cleanly.
- [ ] Public API routes remain accessible.
- [ ] Watchlist and Strategy CRUD remain untouched.
- [ ] Existing JWT middleware remains untouched.

## 6. Final Readiness Matrix

| Area | Status | Evidence | Blocker |
|------|--------|----------|---------|
| Governance | GREEN | Phase 4 Roadmap (P4-04) | None |
| Traceability | GREEN | Explicitly mapped | None |
| Database | GREEN | PostgreSQL safe, ClickHouse schema governed | None |
| Security | GREEN | Isolated internal auth | None |
| Background | GREEN | Memory batching justified | None |
| Frontend | GREEN | Excluded | None |

## 7. Approval Request
This plan implements the mandated corrections for P4-04 Discovery. Approval is required to proceed with code implementation.
