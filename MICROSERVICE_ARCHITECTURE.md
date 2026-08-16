# Enterprise Service Architecture

## 1. Architectural Strategy: Modular Monolith with Role-Based Deployments

MarketPulse Pro adopts a **Modular Monolith** pattern using Uber Fx, deployed via **Role-Based Profiles**. This architecture perfectly balances development velocity with independent scalability, satisfying the 100,000–200,000 DAU target without introducing premature distributed-system complexity.

### Rationale
- **One Codebase:** A single Go binary, unified `go.mod`, and single CI/CD pipeline.
- **Physical Isolation:** The binary is launched with specific environment profiles (e.g., `ROLE=API`, `ROLE=WEBSOCKET`, `ROLE=WORKER`).
- **Independent Scaling:** WebSocket pods can scale to 200k concurrent connections without unnecessarily scaling the background workers.
- **Strict Bounded Contexts:** Domains (Auth, MarketData, Trading) remain strictly isolated within Go packages, communicating via interfaces.

## 2. Logical Service Boundaries (The Roles)

### Role A: Core API Service (`ROLE=API`)
- **Responsibility:** Handles all synchronous REST/HTTP requests from the Next.js frontend and mobile clients.
- **Domains:** Auth, User, Watchlist, Strategy Builder, Portfolio.
- **Data:** Primary owner of PostgreSQL transactional tables.
- **Scaling Profile:** Medium (scales with standard user interaction patterns).

### Role B: Realtime Hub Service (`ROLE=WEBSOCKET`)
- **Responsibility:** Handles millions of persistent WebSocket connections. Pushes realtime market data and order updates to clients.
- **Domains:** WebSocket connection management, ticket validation.
- **Data:** Redis (Pub/Sub backplane). No direct PostgreSQL access required.
- **Scaling Profile:** Very High (memory-bound per connection, scales horizontally behind load balancer).

### Role C: Background Worker Service (`ROLE=WORKER`)
- **Responsibility:** Executes asynchronous, CPU-intensive background jobs via Asynq and gocron.
- **Domains:** Scheduled analytics calculation, notifications, maintenance.
- **Data:** Connects to PostgreSQL, ClickHouse, and Redis.
- **Scaling Profile:** High (scales horizontally based on queue depth).

### Role D: Market Data & Analytics Service (`ROLE=MARKETDATA`) *(Future Extraction)*
- **Responsibility:** Ingests live market data, calculates Option Chains, updates ClickHouse.
- **Domains:** Market Data, Option Chains, Screener.
- **Data:** Primary owner of ClickHouse.
- **Scaling Profile:** High (compute/read IO bound).

### Role E: Broker Integration Service (`ROLE=BROKER`) *(Future Extraction)*
- **Responsibility:** Dedicated adapter for upstream brokers (Zerodha, Angel One, ICICI Direct).
- **Domains:** Trading, Order Execution, Broker Sessions.
- **Data:** PostgreSQL (broker tokens).
- **Scaling Profile:** Medium (rate-limited by upstream brokers, requires strict failure isolation).

## 3. Communication Patterns
- **Client to Backend:** HTTP/REST (API) and WebSocket (Realtime Hub).
- **Frontend BFF:** Next.js Server Components inherently act as a BFF for web clients.
- **Internal Synchronous:** Go interface calls within the modular monolith.
- **Internal Asynchronous:** Redis Pub/Sub (Fire-and-forget events) and Asynq (Guaranteed execution tasks).

## 4. Pending Decisions Preserved
- `DEC-ARCH-004C/D/E` (Broker APIs): Provider-specific implementations remain pending.
- `DEC-ARCH-005` (S3/Object Storage): Pending.
- `DEC-ARCH-006` (Parquet): Pending.
- `DEC-ARCH-008` (Deployment Topology): Pending.
