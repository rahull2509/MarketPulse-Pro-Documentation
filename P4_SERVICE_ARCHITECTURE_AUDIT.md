# P4 Service Architecture Audit

## 1. Executive Summary
This report formalizes the Phase 4 Service Architecture for MarketPulse Pro. Following a comprehensive repository audit and forensic review of existing Phase-1 SPECs and Phase-2 IMPLs, the architecture is designed to support 100,000–200,000 DAU. The core decision is to embrace a **Modular Monolith** pattern using Uber Fx, deployed via **Role-Based Profiles**, preventing premature distributed-system complexity while guaranteeing independent failure isolation and scalability.

## 2. Repository Evidence
The existing P3 implementation successfully wired a single Uber Fx binary containing:
- `gin` API router
- `redis` Pub/Sub WebSocket Bridge
- `golang-migrate` managed PostgreSQL schema
- `asynq` worker queue
- `gocron` scheduler
- Next.js frontend with TanStack Query and Zustand.

## 3. SPEC/IMPL Evidence
Governance documents (SPEC_001-012, IMPL_001-014) explicitly mandate standard domains: Authentication, Market Data, Realtime Events, Analytics, Background Processing, and Broker Integrations. Constraints prohibit Kafka/NATS/S3 (unless explicitly authorized) and prioritize Redis/PostgreSQL/ClickHouse.

## 4. Existing P3 Architecture
Currently, all components (API, Worker, WebSocket Hub, Scheduler) start simultaneously in a single node via `bootstrap.go`. While acceptable for development, this lacks independent scaling and failure isolation for production.

## 5. Bounded Context Analysis
We identified 5 strict contexts:
- **Identity/Core** (Auth, User, Strategy, Watchlists)
- **Market Data** (Ingest, ClickHouse analytics)
- **Broker/Trading** (Upstream facade, order states)
- **Realtime** (WebSocket connection pooling)
- **Background** (Asynq/Cron jobs)

## 6. Candidate Service Evaluation
Rather than extracting 5 independent codebases (which introduces cross-repo dependency hell), the candidates evaluated best mapped to isolated execution environments of the same codebase.

## 7. Final Service Count
FINAL SERVICE COUNT: 5 (Logical deployable roles, 1 physical codebase).

## 8. Final Service List
1. Core API Role (`ROLE=API`)
2. Realtime Hub Role (`ROLE=WEBSOCKET`)
3. Worker Role (`ROLE=WORKER`)
4. Market Data Role (`ROLE=MARKETDATA`)
5. Broker Adapter Role (`ROLE=BROKER`)

## 9. Responsibility of Each Service
- **Core API:** Standard REST operations, UI state mutations, user profiles.
- **Realtime Hub:** Persistent WebSocket termination, fast push of Redis Pub/Sub events.
- **Worker:** High-CPU background tasks (Greeks calc, notifications).
- **Market Data:** Tick ingestion, high-read analytics serving.
- **Broker:** Isolated upstream interactions, handling API rate limits/timeouts.

## 10. Data Ownership
- **PostgreSQL:** Exclusively owned by Core API and Broker Roles (Users, Watchlists, Strategies, Orders, Sessions).
- **ClickHouse:** Exclusively owned by Market Data (Ticks, OHLCV, Greeks).

## 11. Redis Ownership
- `ws:ticket:*` → Core API (Write) / Realtime (Read/Del)
- `cache:*` → Various Domains
- `mp:events:*` → Realtime Pub/Sub
- `asynq:*` → Worker Queues

## 12. Event Architecture
- **APPROVED:** `mp:events:system` → `system.health`
- **PROPOSED:** `mp:events:marketdata` → `market.prices`, `market.news`
- **PROPOSED:** `mp:events:orders` → `order.execution`, `order.status`

## 13. Realtime Architecture
Redis Pub/Sub acts as the event backplane. The Realtime Hub service (`ROLE=WEBSOCKET`) subscribes to the backplane and fans out to locally connected WebSocket clients. Authentication uses single-use `GETDEL` tickets.

## 14. Background Processing
The `gocron` scheduler enqueues jobs into Redis. The isolated Worker service pulls from `asynq` queues to process tasks without starving the Core API HTTP threads.

## 15. Broker Integration Boundary
A unified facade will abstract Zerodha, Angel One, and ICICI Direct. The implementation of concrete providers remains blocked pending DEC-ARCH-004C/D/E resolutions.

## 16. API Gateway/BFF Decision
Next.js App Router serves as the native BFF. No dedicated API Gateway is justified at this stage.

## 17. Communication Matrix
- Frontend ↔ Backend: HTTP/REST & WebSockets.
- Backend ↔ Backend: Synchronous internal Go interface calls.
- Backend ↔ Worker: Asynchronous (Asynq/Redis).

## 18. Deployment Topology
A containerized topology:
- N `API` Pods
- N `WEBSOCKET` Pods
- N `WORKER` Pods
- N `MARKETDATA` Pods
- PostgreSQL + Redis + ClickHouse Clusters.

## 19. Scale Analysis
- **WebSocket:** VERY HIGH concurrency (Memory bound).
- **Market Data:** HIGH throughput (IO/CPU bound).
- **Core API:** MEDIUM throughput (Standard CRUD).
This topology scales exactly where needed.

## 20. Failure Isolation
If the Broker API fails, only the Trading features degrade; Market Data and Watchlists remain fully operational. If WebSockets drop, users fall back to HTTP polls/refreshes. 

## 21. Security Boundaries
JWT handles client-to-server auth. Internal services run on a trusted VPC. Upstream broker tokens are encrypted in PostgreSQL.

## 22. Observability
Uber Zap (Logs). Prometheus (Metrics). Traces pending implementation. 

## 23. Testing Architecture
Standard Go `testing`, `testify` for units. Integration tests for Postgres/Redis repositories.

## 24. Microservice vs Modular Monolith Decision
**Modular Monolith with Role-Based Deployments.** Evidence shows this provides the failure isolation of microservices without the network latency and deployment complexity of separate repos.

## 25. Phase 4 Dependency Order
1. Service Flags (P4-01)
2. Core Domains (P4-02, P4-03)
3. Market Data (P4-04, P4-05)
4. Broker Facade (P4-07)

## 26. Pending Decisions
- DEC-ARCH-004C/D/E (Broker Specifications)
- DEC-ARCH-005 (S3)
- DEC-ARCH-006 (Parquet)
- DEC-ARCH-008 (Deployment Compute Vendor)

## 27. Architecture Risks
- Broker rate limits throttling the unified facade.
- Redis Pub/Sub memory limits under high market volatility.

## 28. Final Recommendation
Proceed with Phase 4 Implementation adopting the Modular Monolith Role-Based topology.

---

FINAL SERVICE COUNT:
5

FINAL SERVICE MODEL:
MODULAR MONOLITH

SERVICES:
1. Core API
2. Realtime Hub
3. Worker
4. Market Data
5. Broker Integration

CORE SERVICES:
Core API, Realtime Hub, Worker

OPTIONAL/FUTURE SERVICES:
Market Data, Broker Integration (if broker limits mandate physical separation)

DEFERRED SERVICES:
Notification Engine (Deferred to late P4)

PENDING ARCHITECTURAL DECISIONS:
DEC-ARCH-004C/D/E, DEC-ARCH-005, DEC-ARCH-006, DEC-ARCH-008

PHASE 4 READY:
YES

ARCHITECTURE CONFIDENCE:
HIGH

FINAL VERDICT:
P4 SERVICE ARCHITECTURE = READY
