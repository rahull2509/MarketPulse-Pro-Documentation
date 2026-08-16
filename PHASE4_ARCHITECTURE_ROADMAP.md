# Phase 4 Architecture Roadmap

This roadmap defines the implementation order for the Phase 4 Enterprise Service Architecture, derived from bounded-context dependency analysis.

## Execution Sequence

| Phase | Milestone | Dependencies | Primary Goal |
|---|---|---|---|
| **P4-00** | Architecture Lock | None | Finalize Service Boundaries, Data Ownership, and Event Catalog. |
| **P4-01** | Core API & Fx Deployment Roles | P4-00 | Refactor `bootstrap.go` to accept environment `ROLE` flags (`API`, `WORKER`, `WEBSOCKET`). |
| **P4-02** | User & Watchlist Domains | P4-01 | Implement full CRUD for Watchlists and Items against PostgreSQL. |
| **P4-03** | Strategy Builder Domain | P4-02 | Implement custom Strategy and Leg persistence. |
| **P4-04** | Market Data Ingest Foundation | P4-00 | Establish ClickHouse schema and live tick ingestion pipeline. |
| **P4-05** | Realtime Market Data Distribution | P4-04 | Map ClickHouse aggregates to Redis Pub/Sub (`market.prices`, `market.news`). |
| **P4-06** | Option Chain & Analytics Calculation | P4-05 | Implement Worker jobs for Greeks/IV calculation and ClickHouse persistence. |
| **P4-07** | Broker Integration Layer (Facade) | P4-03 | Implement the internal unified Broker Interface and generic Orders DB schema. |
| **P4-08** | Zerodha/AngelOne Adapters | P4-07, `DEC-ARCH-004C/D` | Implement concrete provider APIs behind the facade. |
| **P4-09** | Trading & Portfolio Aggregation | P4-08 | Expose execution APIs and sync upstream positions to the frontend. |
| **P4-10** | Production Hardening | All above | Circuit breakers, distributed tracing, load testing at 100k DAU profiles. |
