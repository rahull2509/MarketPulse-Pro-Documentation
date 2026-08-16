# Service Dependency Graph

This document illustrates the dependencies between logical services in the MarketPulse Pro architecture.

```mermaid
graph TD
    %% Clients
    ClientWeb[Web Client / Next.js BFF]
    ClientMobile[Mobile Client]

    %% Logical Services (Modular Monolith Roles)
    CoreAPI[Core API Service]
    RealtimeHub[Realtime Hub Service]
    WorkerService[Background Worker Service]
    MarketData[Market Data Service]
    BrokerService[Broker Integration Service]

    %% Databases and Infrastructure
    PostgreSQL[(PostgreSQL)]
    ClickHouse[(ClickHouse)]
    Redis[(Redis)]

    %% External Systems
    ExternalBroker[External Broker APIs]
    ExternalMarketData[External Market Data Feeds]

    %% Client Connections
    ClientWeb -->|REST HTTP| CoreAPI
    ClientWeb -->|WebSocket| RealtimeHub
    ClientMobile -->|REST HTTP| CoreAPI
    ClientMobile -->|WebSocket| RealtimeHub

    %% Service to Database
    CoreAPI -->|Read/Write| PostgreSQL
    BrokerService -->|Read/Write| PostgreSQL
    WorkerService -->|Read/Write| PostgreSQL

    MarketData -->|Read/Write| ClickHouse
    WorkerService -->|Read/Write| ClickHouse
    CoreAPI -->|Read| ClickHouse

    %% Service to Cache/State
    CoreAPI -->|Cache / Tickets| Redis
    RealtimeHub -->|Validate Tickets| Redis
    WorkerService -->|Asynq Queue| Redis

    %% Event Bus (Redis Pub/Sub)
    WorkerService -.->|Publish mp:events:system| Redis
    MarketData -.->|Publish mp:events:marketdata (PROPOSED)| Redis
    BrokerService -.->|Publish mp:events:orders (PROPOSED)| Redis
    Redis -.->|Subscribe| RealtimeHub

    %% External Connections
    BrokerService -->|REST/WS| ExternalBroker
    MarketData -->|TCP/WS| ExternalMarketData
```

## Synchronous vs Asynchronous Communication Matrix

| Source Service | Communication Mechanism | Target Service | Reason |
|---|---|---|---|
| API Gateway / BFF | REST/HTTP | Core API | Standard synchronous CRUD operations and JWT issuance. |
| Core API | Redis Pub/Sub | Realtime Hub | Fire-and-forget notification delivery to active WS connections. |
| Core API | Asynq (Redis) | Worker Service | Offload heavy analytical tasks (e.g. strategy backtesting). |
| Realtime Hub | WebSocket | Client Browser | Bidirectional persistent stream for live data. |
| Market Data | Internal Go Call (or HTTP) | Core API | Synchronous retrieval of option chain metadata if co-located. |
| Broker Integration | Internal Go Call | Core API | Synchronous execution of order placement. |
