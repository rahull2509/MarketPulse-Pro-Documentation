# P3 Foundation Remediation — Master Implementation Plan

## Document Information
- **Status**: PLAN — NOT YET EXECUTED
- **Authority**: P3_01_TO_P3_09_FORENSIC_AUDIT.md
- **Scope**: Foundation remediation only — NO product features
- **Governance**: DECISION_REGISTER.md, TECHNOLOGY_STACK.md, ADR-001, AGENTS.md, IMPL_003, IMPL_007

---

## 1. Executive Summary

### Current State (Repository-Verified)

The repository contains **structural scaffolding** but lacks operational foundation:

| Layer | What Exists | What Is Missing |
|-------|-------------|-----------------|
| Bootstrap | `main.go` → Fx → `bootstrap.go` wires config, logger, DBs, server, auth, cache, pubsub | `routes.RegisterRoutes` is **never called** — orphaned. `websocket.Hub` is **not wired** into Fx. |
| Database | GORM connection with pooling, TxManager | Zero migrations, zero schemas, zero tables |
| Auth | JWT generate/validate (`jwt.go`), middleware extracts `Bearer` token (`middleware/auth.go`) | No login/register handlers, no password hashing, hardcoded dev secret in `jwt.go` L45 |
| Domain | User model+interface+repository, Strategy/StrategyLeg models, Watchlist/WatchlistItem models, MarketTick/OptionGreeks structs | No services, no business logic. Only User has interface+repository. Strategy/Watchlist have only models. |
| API | `routes.go` defines `/api/v1` with `/ping` + commented-out routes | Never called from bootstrap. No handlers, no controllers, no request/response DTOs |
| WebSocket | `Hub` struct with Register/Unregister/Subscribe/Broadcast (uses sync.RWMutex + channels) | Not imported by bootstrap, not attached to HTTP upgrade, **no gorilla/websocket import in any `.go` file**, no Redis integration |
| Pub/Sub | Publisher and Subscriber wrappers around go-redis | Wired into Fx but never used by any code path |
| Background | **Nothing** — Asynq and gocron/v2 are **not in `go.mod`** | Entire subsystem absent |
| Frontend | `layout.tsx` + `page.tsx` (placeholder text) + `api.ts` (fetch wrapper) + `globals.css` (dark theme tokens) | No pages, no components, no state management, no auth, no WS client |
| Pkg | `pkg/errors/errors.go` (AppError struct), `pkg/response/response.go` (Success/Created/Error helpers) | Functional but unused by any handler |

### Audit Discrepancy Report

The forensic audit stated `gorilla/websocket` is "Actually Imported: YES." This is **incorrect**. `gorilla/websocket v1.5.3` is in `go.mod` as an `// indirect` dependency (pulled by ClickHouse driver) but **no `.go` source file imports it**. The `websocket` package (`internal/websocket/hub.go`) uses only `sync` and `zap`. No actual WebSocket protocol handling exists.

### Target State

After remediation, the repository will have:
- PostgreSQL tables for foundation entities created via `golang-migrate` (per IMPL_003)
- Working authentication flow (register, login, JWT issuance)
- Domain service layer bridging handlers → repositories
- Operational API endpoints for auth + user + watchlist + strategy CRUD
- Redis Pub/Sub → WebSocket Hub pipeline with actual HTTP upgrade via gorilla/websocket
- Asynq/gocron foundation with at least one health-check task
- Frontend application shell with state management (`[DECISION REQUIRED]` gate), auth context, and API integration

---

## 2. Governance Constraints

### MUST Preserve
| Technology | Status | Evidence |
|-----------|--------|----------|
| Go | GREEN | `go.mod` — `go 1.26.1` |
| Gin | GREEN | `go.mod` — `v1.12.0`, used in `server.go`, `routes.go`, `middleware/auth.go` |
| Uber Fx | GREEN | `go.mod` — `v1.24.0`, used in `bootstrap.go` |
| Viper | GREEN | `go.mod` — `v1.21.0`, used in `config.go` |
| Zap | GREEN | `go.mod` — `v1.28.0`, used throughout |
| PostgreSQL/GORM | GREEN | `go.mod` — GORM `v1.31.2`, `postgres.go` connects, models use GORM tags |
| Redis | GREEN | `go.mod` — `go-redis v9.22.0`, `redis.go` connects |
| ClickHouse | GREEN | `go.mod` — `clickhouse-go v2.48.0`, `clickhouse.go` connects |
| Gorilla WebSocket | GREEN | `go.mod` — `v1.5.3` (indirect — must become direct) |
| JWT | GREEN | `go.mod` — `golang-jwt v5.3.1` (indirect — must become direct) |
| Next.js/React | GREEN | `package.json` — Next `^15.3.0`, React `^19.1.0` |
| Testify | GREEN | `go.mod` — `v1.11.1`, used in existing tests |
| Docker (dev) | YELLOW | `docker-compose.dev.yml` exists (provisionally approved) |
| golang-migrate | GREEN | IMPL_003 L2193–2195 explicitly names it as "Official Migration Tool" |

### MUST NOT Introduce
| Prohibited | Reason |
|-----------|--------|
| Upstox | DEC-ARCH-004A — REJECTED |
| Python / APScheduler | LEGACY/RED |
| Kafka / NATS | NOT SELECTED — Redis Pub/Sub is approved (DEC-ARCH-007) |
| S3 | DEC-ARCH-005 — PENDING |
| Parquet | DEC-ARCH-006 — DEFERRED |
| Zerodha/AngelOne/ICICI Direct concrete API calls | DEC-ARCH-004C/D/E — PENDING |

### MUST NOT Resolve (Pending Decisions)

| Decision | Status | Impact on Remediation | Can Remediation Proceed? |
|----------|--------|----------------------|--------------------------|
| DEC-ARCH-004C (Zerodha API) | PENDING BLOCKING | No concrete Zerodha adapter | YES — broker layer is excluded from foundation |
| DEC-ARCH-004D (Angel One API) | PENDING BLOCKING | No concrete Angel One adapter | YES — broker layer is excluded |
| DEC-ARCH-004E (ICICI Direct API) | PENDING BLOCKING | No concrete ICICI adapter | YES — broker layer is excluded |
| DEC-ARCH-005 (Cloud Storage) | PENDING NON-BLOCKING | No S3 or storage modules | YES — no storage needed for foundation |
| DEC-ARCH-006 (Data Format) | DEFERRED | No Parquet pipelines | YES — no data pipelines in foundation |
| DEC-ARCH-008 (Deployment Infra) | PENDING NON-BLOCKING | No production Docker/Nginx/CI changes | YES — only dev infra used |
| Secret Management | UNRESOLVED | Use `[DECISION REQUIRED]` boundary — config-based for dev | YES — dev defaults acceptable |
| Frontend State Management | `[DECISION REQUIRED]` | Recommendation provided in R7; must not be treated as RESOLVED until user approves | YES — R8 implementation conditional on R7 approval |
| Database Migration Tool | **NOW RESOLVED** — IMPL_003 specifies `golang-migrate` | Plan uses `golang-migrate`, not GORM AutoMigrate | YES |

---

## 3. Remediation Units

### Execution Order (Dependency-Derived)

```
R1 — Database Operationalization (golang-migrate + schemas)
 ↓
R2 — Domain Service Layer
 ↓
R3 — Authentication Flow
 ↓
R4 — API Operationalization
 ↓
R5 — Realtime Pipeline (Redis Pub/Sub → WebSocket)
 ↓
R6 — Background Processing Foundation (Asynq + gocron/v2)
 ↓
R7 — Frontend State Architecture Decision [DECISION REQUIRED GATE]
 ↓
R8 — Functional Frontend Foundation (blocked by R7 approval)
 ↓
R9 — Architecture Compliance Hardening
```

**Rationale**: Tables must exist before repositories can query them. Services must exist before handlers can call them. Auth flow requires the user service. The realtime pipeline is independent of auth/API but builds on Redis. Background processing depends on Redis. Frontend depends on backend APIs being operational AND state management decision being approved.

---

## R1 — Database Operationalization

### Objective
Create foundation entity tables in PostgreSQL using `golang-migrate` per IMPL_003. Add JWT config to Viper. Add relationship tags for FK constraints.

### Migration Strategy — Governance-Compliant

**AGENTS.md Line 53**: "No schema modifications without proper migration files and governance approval."

**IMPL_003 L2193–2195**: "Official Migration Tool: golang-migrate. All database schema changes shall be managed exclusively through version-controlled migration files."

**IMPL_003 L2240–2260**: Migration directory structure: `migrations/sql/up/`, `migrations/sql/down/`.

**IMPL_003 L2264–2280**: Migration rules: One Purpose, Atomic, Reversible, Idempotent, Reviewed, Tested. Migration history shall never be rewritten.

**IMPL_003 L2304–2326**: Application startup shall detect pending migrations, validate order, execute, verify schema version, then continue.

**IMPL_012 L523**: "golang-migrate shall manage database migrations."

Therefore: **GORM AutoMigrate is NOT authorized.** The plan must use `golang-migrate` with versioned SQL migration files. GORM is used only as the ORM for runtime queries, not for schema management.

### Prerequisites
- PostgreSQL running (via `docker-compose.dev.yml`)
- `golang-migrate/migrate` added to `go.mod`

### Foundation Tables

| Table | Model | Source | Materialize Now? |
|-------|-------|--------|-------------------|
| `users` | `User` | `internal/modules/user/models/user.go` | YES |
| `strategies` | `Strategy` | `internal/modules/strategy/models/strategy.go` | YES |
| `strategy_legs` | `StrategyLeg` | `internal/modules/strategy/models/strategy.go` | YES |
| `watchlists` | `Watchlist` | `internal/modules/watchlist/models/watchlist.go` | YES |
| `watchlist_items` | `WatchlistItem` | `internal/modules/watchlist/models/watchlist.go` | YES |
| `schema_migrations` | (golang-migrate internal) | automatic | YES |

### Tables NOT Materialized
| Reason | Models |
|--------|--------|
| ClickHouse responsibility (analytics) | `MarketTick`, `OptionGreeks`, `OptionChainEntry` |
| Broker data (blocked by DEC-ARCH-004C/D/E) | `Session`, `Position`, `Order` from `broker/interfaces.go` |

### Migration File Structure (per IMPL_003 L2240–2252)
```
Backend/
  migrations/
    000001_create_users_table.up.sql
    000001_create_users_table.down.sql
    000002_create_watchlists_table.up.sql
    000002_create_watchlists_table.down.sql
    000003_create_watchlist_items_table.up.sql
    000003_create_watchlist_items_table.down.sql
    000004_create_strategies_table.up.sql
    000004_create_strategies_table.down.sql
    000005_create_strategy_legs_table.up.sql
    000005_create_strategy_legs_table.down.sql
```

### Migration SQL Design

**000001_create_users_table.up.sql**:
```sql
CREATE TABLE IF NOT EXISTS users (
    id BIGSERIAL PRIMARY KEY,
    email VARCHAR(255) NOT NULL,
    name VARCHAR(255) NOT NULL,
    password VARCHAR(255) NOT NULL,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE UNIQUE INDEX IF NOT EXISTS idx_users_email ON users (email);
```

**000001_create_users_table.down.sql**:
```sql
DROP TABLE IF EXISTS users;
```

Similar pattern for watchlists (FK: `user_id → users.id`), watchlist_items (FK: `watchlist_id → watchlists.id`), strategies (FK: `user_id → users.id`), strategy_legs (FK: `strategy_id → strategies.id`).

### Schema Enhancements to Models

**Strategy model** — add `Legs []StrategyLeg` relationship via GORM `has-many` tag so GORM can Preload. This does NOT change the database schema (the FK is created by the migration SQL). It enables GORM's `Preload("Legs")` at runtime.

**Watchlist model** — add `Items []WatchlistItem` relationship via GORM `has-many` tag. Same rationale.

### Startup Migration Integration

Per IMPL_003 L2304–2326, the application startup must:
1. Detect pending migrations
2. Validate migration order
3. Execute migrations
4. Verify schema version
5. Continue startup

This is implemented via a `RunMigrations(cfg, log)` function called in `bootstrap.go` via `fx.Invoke`, before the HTTP server starts.

### File-Level Plan

| Action | File Path | Purpose | Dep |
|--------|-----------|---------|-----|
| CREATE | `Backend/migrations/000001_create_users_table.up.sql` | Users table DDL | — |
| CREATE | `Backend/migrations/000001_create_users_table.down.sql` | Users table rollback | — |
| CREATE | `Backend/migrations/000002_create_watchlists_table.up.sql` | Watchlists table DDL | 000001 |
| CREATE | `Backend/migrations/000002_create_watchlists_table.down.sql` | Watchlists table rollback | — |
| CREATE | `Backend/migrations/000003_create_watchlist_items_table.up.sql` | Watchlist items DDL | 000002 |
| CREATE | `Backend/migrations/000003_create_watchlist_items_table.down.sql` | Watchlist items rollback | — |
| CREATE | `Backend/migrations/000004_create_strategies_table.up.sql` | Strategies table DDL | 000001 |
| CREATE | `Backend/migrations/000004_create_strategies_table.down.sql` | Strategies table rollback | — |
| CREATE | `Backend/migrations/000005_create_strategy_legs_table.up.sql` | Strategy legs DDL | 000004 |
| CREATE | `Backend/migrations/000005_create_strategy_legs_table.down.sql` | Strategy legs rollback | — |
| CREATE | `Backend/internal/core/database/migrate.go` | `RunMigrations` function using golang-migrate | — |
| MODIFY | `Backend/internal/modules/strategy/models/strategy.go` | Add `Legs` relationship field | — |
| MODIFY | `Backend/internal/modules/watchlist/models/watchlist.go` | Add `Items` relationship field | — |
| MODIFY | `Backend/internal/bootstrap/bootstrap.go` | Add `fx.Invoke(database.RunMigrations)` | — |
| MODIFY | `Backend/go.mod` | Add golang-migrate + promote indirect deps + add asynq, gocron | — |

### Tests

| Test | File | Description |
|------|------|-------------|
| `TestMigrationsUpDown` | `Backend/internal/core/database/migrate_test.go` | Requires live PostgreSQL. Applies all up migrations, verifies tables exist, applies all down migrations, verifies tables removed. |

### Acceptance Criteria
- `docker-compose up -d` then `go run ./cmd/api` runs all migrations automatically
- `SELECT table_name FROM information_schema.tables WHERE table_schema='public'` returns: `users`, `watchlists`, `watchlist_items`, `strategies`, `strategy_legs`, `schema_migrations`
- FK constraints exist between child → parent tables
- Down migrations successfully reverse each up migration
- Migration files are version-controlled, sequential, immutable (per IMPL_003 L2254–2260)

---

## R2 — Domain Service Layer

### Objective
Create service structs that encapsulate business logic for each domain, sitting between handlers and repositories.

### Dependencies
- R1 (tables must exist for repositories to work)

### Existing Code (Repository-Verified)
- **User**: Model (`user.go`), Interface (`interfaces/repository.go` — `UserRepository`), Repository Implementation (`repositories/user_repository.go` — `PostgresUserRepository`). All three exist. Service is missing.
- **Strategy**: Model only (`models/strategy.go` — `Strategy`, `StrategyLeg`). No interface, no repository, no service.
- **Watchlist**: Model only (`models/watchlist.go` — `Watchlist`, `WatchlistItem`). No interface, no repository, no service.

### Service Definitions

#### UserService
- **File**: `Backend/internal/modules/user/services/user_service.go`
- **Dependencies**: `interfaces.UserRepository` (existing), `*zap.Logger`
- **Methods**:
  - `Register(ctx, name, email, password) (*User, error)` — hash password with bcrypt, check duplicate email, create user
  - `GetByID(ctx, id) (*User, error)` — retrieve user
  - `GetByEmail(ctx, email) (*User, error)` — retrieve user
  - `Authenticate(ctx, email, password) (*User, error)` — lookup by email, compare bcrypt hash
- **Validation**: Email format, password minimum length (8 chars), name non-empty
- **Errors**: `ErrDuplicateEmail`, `ErrInvalidCredentials`, `ErrUserNotFound`
- **Transaction**: Single Create, no TX needed

#### WatchlistService
- **File**: `Backend/internal/modules/watchlist/services/watchlist_service.go`
- **Dependencies**: `interfaces.WatchlistRepository` (to be created), `*zap.Logger`
- **Methods**:
  - `Create(ctx, userID, name) (*Watchlist, error)`
  - `ListByUser(ctx, userID) ([]Watchlist, error)`
  - `GetByID(ctx, id, userID) (*Watchlist, error)` — ownership check
  - `Delete(ctx, id, userID) error` — ownership check
  - `AddItem(ctx, watchlistID, userID, symbol) (*WatchlistItem, error)` — ownership check
  - `RemoveItem(ctx, itemID, userID) error` — ownership check
- **Validation**: Name non-empty, symbol non-empty, max 50 items per watchlist
- **Transaction**: `AddItem` and `RemoveItem` do single writes, TX not required. Ownership check + write could use TX if atomicity is needed.

#### StrategyService
- **File**: `Backend/internal/modules/strategy/services/strategy_service.go`
- **Dependencies**: `interfaces.StrategyRepository` (to be created), `*zap.Logger`
- **Methods**:
  - `Create(ctx, userID, name, strategyType) (*Strategy, error)`
  - `ListByUser(ctx, userID) ([]Strategy, error)`
  - `GetByID(ctx, id, userID) (*Strategy, error)` — ownership check, loads legs
  - `Delete(ctx, id, userID) error` — ownership check, cascades legs via FK
  - `AddLeg(ctx, strategyID, userID, leg) (*StrategyLeg, error)`
  - `RemoveLeg(ctx, legID, userID) error`
- **Validation**: Name non-empty, type from allowed list, legs have valid side/quantity
- **Transaction**: Delete strategy (which cascades legs) should use TX

### File-Level Plan

| Action | File Path | Purpose | Dep |
|--------|-----------|---------|-----|
| CREATE | `internal/modules/watchlist/interfaces/repository.go` | WatchlistRepository interface | — |
| CREATE | `internal/modules/watchlist/repositories/watchlist_repository.go` | GORM-based WatchlistRepository | R1 |
| CREATE | `internal/modules/watchlist/services/watchlist_service.go` | Watchlist business logic | ^ |
| CREATE | `internal/modules/strategy/interfaces/repository.go` | StrategyRepository interface | — |
| CREATE | `internal/modules/strategy/repositories/strategy_repository.go` | GORM-based StrategyRepository | R1 |
| CREATE | `internal/modules/strategy/services/strategy_service.go` | Strategy business logic | ^ |
| CREATE | `internal/modules/user/services/user_service.go` | User business logic (bcrypt) | — |

### Tests

| Test File | Coverage |
|-----------|----------|
| `internal/modules/user/services/user_service_test.go` | Register, Authenticate, duplicate email, invalid credentials |
| `internal/modules/watchlist/services/watchlist_service_test.go` | Create, List, ownership check, item limit |
| `internal/modules/strategy/services/strategy_service_test.go` | Create, List, legs, ownership check |

- Services tested with mock repositories (interface-based DI) using `testify/mock`
- User repository already exists — no mock needed for that, but service tests should use mock for isolation

### Acceptance Criteria
- Each service method has at least one passing unit test
- Services enforce ownership (user A cannot access user B's watchlists)
- Password is bcrypt-hashed before storage, never returned in any service output
- `json:"-"` tag on `User.Password` already prevents JSON serialization

---

## R3 — Authentication Flow

### Objective
Create working register/login API endpoints that produce JWT tokens, wired into the existing middleware.

### Dependencies
- R1 (users table exists)
- R2 (UserService exists)

### Separation: Application Auth vs Broker Auth
Per ADR-001 Section 4: "Application Identity must never leak into or merge with the Broker Connection state." This remediation covers **only** application authentication. Broker authentication remains blocked by DEC-ARCH-004C/D/E.

### Auth Flow Design

```
Register:
  POST /api/v1/auth/register
    → AuthHandler.Register
      → UserService.Register (bcrypt hash, create user)
      → JWTService.GenerateAccessToken
      → Return { token, user }

Login:
  POST /api/v1/auth/login
    → AuthHandler.Login
      → UserService.Authenticate (bcrypt compare)
      → JWTService.GenerateAccessToken
      → Return { token, user }

Protected Request:
  GET /api/v1/user/profile
    → AuthMiddleware (existing — validates JWT, sets user_id in context)
    → UserHandler.GetProfile
      → UserService.GetByID
      → Return { user }
```

### JWT Configuration Enhancement

The existing `jwt.go` already defines a `JWTConfig` struct (L36–40) with `SecretKey`, `AccessExpiryHours`, `RefreshExpiryHours` but it is **never read from config**. The `NewJWTService` function (L43–54) hardcodes `"changeme-insecure-dev-key"`.

#### [MODIFY] `Backend/internal/config/config.go`
- Add `JWT auth.JWTConfig` field to `Config` struct (using the existing `JWTConfig` type)
- Add Viper defaults: `jwt.secret_key` = `"changeme-insecure-dev-key"`, `jwt.access_expiry_hours` = 24, `jwt.refresh_expiry_hours` = 168
- `[DECISION REQUIRED]`: Production secret management platform

#### [MODIFY] `Backend/internal/core/auth/jwt.go`
- `NewJWTService` reads from `cfg.JWT` fields instead of hardcoded values
- Retain `[DECISION REQUIRED]` comment for production secret management

#### [MODIFY] `Backend/configs/config.yaml`
- Add `jwt:` section

#### [MODIFY] `Backend/.env.example`
- Add `MP_JWT_SECRET_KEY`, `MP_JWT_ACCESS_EXPIRY_HOURS`, `MP_JWT_REFRESH_EXPIRY_HOURS`

### Handler Files

| Action | File Path | Purpose |
|--------|-----------|---------|
| CREATE | `Backend/internal/modules/auth/handler.go` | AuthHandler with Register/Login methods |
| CREATE | `Backend/internal/modules/user/handler.go` | UserHandler with GetProfile method |

- `AuthHandler` struct: `UserService`, `JWTService`, `*zap.Logger`
- `RegisterRequest{Name, Email, Password}`, `LoginRequest{Email, Password}`
- `AuthResponse{Token string, User UserResponse}`
- `UserResponse{ID, Email, Name, IsActive, CreatedAt}` — Password excluded
- Uses `pkg/response` for consistent JSON responses
- Uses `pkg/errors` for error mapping

### Security Constraints
- Password: minimum 8 characters, validated in handler
- bcrypt cost: `bcrypt.DefaultCost` (10)
- JWT secret: read from config, not hardcoded in auth code
- `[DECISION REQUIRED]`: Production secret rotation mechanism
- Passwords never logged, never returned in JSON (enforced by `json:"-"` tag on model)
- Rate limiting: **NOT** implemented in foundation (deferred to hardening)

### Tests

| Test File | Coverage |
|-----------|----------|
| `internal/modules/auth/handler_test.go` | Register success, duplicate email, login success, wrong password, missing fields |
| Extend `internal/core/auth/jwt_test.go` | Config-based secret test (verify it reads from config, not hardcode) |

### Acceptance Criteria
- `POST /api/v1/auth/register` with valid body → 201 + JWT token
- `POST /api/v1/auth/login` with correct credentials → 200 + JWT token
- `POST /api/v1/auth/login` with wrong password → 401
- `GET /api/v1/user/profile` without token → 401
- `GET /api/v1/user/profile` with valid token → 200 + user data (no password field)
- Password column in DB contains bcrypt hash, not plaintext
- JWT secret is loaded from config/environment, not hardcoded in `NewJWTService`

---

## R4 — API Operationalization

### Objective
Wire domain handlers into the Gin router. Fix the orphaned `RegisterRoutes`. Implement watchlist and strategy CRUD handlers.

### Dependencies
- R2 (services exist)
- R3 (auth flow exists)

### Critical Fix: Wire Routes into Bootstrap

Currently `routes.RegisterRoutes` is **defined in `routes.go` but never called** from `bootstrap.go`. Additionally, `server.go` L45 creates an orphaned `router.Group("/api/v1")` that is never used.

#### [MODIFY] `Backend/internal/core/server/server.go`
- Remove the orphaned `router.Group("/api/v1")` line (L45)
- No other changes to server.go — routes are registered separately

#### [MODIFY] `Backend/internal/routes/routes.go`
- Refactor to accept handler dependencies via Fx injection
- Register actual routes instead of commented-out placeholders
- New signature: `RegisterRoutes(router *gin.Engine, jwtService, authHandler, userHandler, watchlistHandler, strategyHandler)`

#### [MODIFY] `Backend/internal/bootstrap/bootstrap.go`
- Add `fx.Provide` for: `PostgresUserRepository` (existing), `UserService`, `WatchlistRepository`, `WatchlistService`, `StrategyRepository`, `StrategyService`, `AuthHandler`, `UserHandler`, `WatchlistHandler`, `StrategyHandler`
- Add `fx.Invoke(routes.RegisterRoutes)` — Fx injects all dependencies

### API Route Plan

| Method | Path | Handler | Auth | Classification |
|--------|------|---------|------|----------------|
| GET | `/health` | (existing in server.go) | No | FOUNDATION REQUIRED |
| GET | `/ready` | (existing in server.go) | No | FOUNDATION REQUIRED |
| GET | `/api/v1/ping` | (existing inline in routes.go) | No | FOUNDATION REQUIRED |
| POST | `/api/v1/auth/register` | `AuthHandler.Register` | No | FOUNDATION REQUIRED |
| POST | `/api/v1/auth/login` | `AuthHandler.Login` | No | FOUNDATION REQUIRED |
| GET | `/api/v1/user/profile` | `UserHandler.GetProfile` | Yes | FOUNDATION REQUIRED |
| GET | `/api/v1/watchlists` | `WatchlistHandler.List` | Yes | FOUNDATION REQUIRED |
| POST | `/api/v1/watchlists` | `WatchlistHandler.Create` | Yes | FOUNDATION REQUIRED |
| GET | `/api/v1/watchlists/:id` | `WatchlistHandler.GetByID` | Yes | FOUNDATION REQUIRED |
| DELETE | `/api/v1/watchlists/:id` | `WatchlistHandler.Delete` | Yes | FOUNDATION REQUIRED |
| POST | `/api/v1/watchlists/:id/items` | `WatchlistHandler.AddItem` | Yes | FOUNDATION REQUIRED |
| DELETE | `/api/v1/watchlists/:id/items/:itemId` | `WatchlistHandler.RemoveItem` | Yes | FOUNDATION REQUIRED |
| GET | `/api/v1/strategies` | `StrategyHandler.List` | Yes | FOUNDATION OPTIONAL |
| POST | `/api/v1/strategies` | `StrategyHandler.Create` | Yes | FOUNDATION OPTIONAL |
| GET | `/api/v1/strategies/:id` | `StrategyHandler.GetByID` | Yes | FOUNDATION OPTIONAL |
| DELETE | `/api/v1/strategies/:id` | `StrategyHandler.Delete` | Yes | FOUNDATION OPTIONAL |
| — | `/api/v1/market/*` | — | — | FUTURE PRODUCT FEATURE |
| — | `/api/v1/options/*` | — | — | FUTURE PRODUCT FEATURE |
| — | `/api/v1/broker/*` | — | — | FUTURE PRODUCT FEATURE (BLOCKED) |

### Handler Files

| Action | File Path | Purpose |
|--------|-----------|---------|
| CREATE | `Backend/internal/modules/watchlist/handler.go` | WatchlistHandler — List, Create, GetByID, Delete, AddItem, RemoveItem |
| CREATE | `Backend/internal/modules/strategy/handler.go` | StrategyHandler — List, Create, GetByID, Delete |

### Middleware Files

| Action | File Path | Purpose |
|--------|-----------|---------|
| CREATE | `Backend/internal/middleware/request_id.go` | Generates UUID, sets `X-Request-ID` header + context for Zap logging |
| CREATE | `Backend/internal/middleware/cors.go` | CORS middleware allowing `http://localhost:3000` in dev. `[DECISION REQUIRED]` for production origins. |

### Request/Response DTOs
All request DTOs live alongside their handler files:

| DTO | Fields | Validation |
|-----|--------|------------|
| `CreateWatchlistRequest` | `Name string` | required, max 255 |
| `AddWatchlistItemRequest` | `Symbol string` | required, max 50 |
| `CreateStrategyRequest` | `Name string`, `Type string` | required |

### Error Handling Convention
- Use `pkg/errors.AppError` for all domain errors
- Handler catches `AppError` → sends `pkg/response.Error(c, appErr)`
- Unexpected errors → `response.Error(c, errors.InternalError("...", err))` — never leak stack traces

### Pagination
- Foundation endpoints return user-scoped lists (typically < 100 items). Pagination is **DEFERRED** to product feature phase.

### Tests

| Test File | Coverage |
|-----------|----------|
| `internal/modules/watchlist/handler_test.go` | CRUD via httptest |
| `internal/modules/strategy/handler_test.go` | CRUD via httptest |

### Acceptance Criteria
- All routes in table above return correct HTTP status codes
- Protected routes reject requests without valid JWT (401)
- Validation errors return 400 with descriptive message
- Ownership enforcement: user cannot access another user's resources (403)
- `go build ./...` compiles cleanly
- All handler tests pass

---

## R5 — Realtime Pipeline

### Objective
Create an actual WebSocket upgrade endpoint using gorilla/websocket, wire Hub to Redis Pub/Sub Subscriber, and establish the event pipeline.

### Dependencies
- R1 (Redis connection exists)
- R3 (JWT service exists for auth validation)
- R4 (server and routes are wired)

### WebSocket Authentication Design

**IMPL_007 Upgrade Flow (L432–462)**: HTTP Request → Route Validation → JWT Authentication → Origin Validation → WebSocket Upgrade → Connection Registration → Ready.

**IMPL_007 Connection States (L310–328)**: CONNECTING → AUTHENTICATING → CONNECTED → ...

The IMPL specifies that JWT authentication happens **during the upgrade** (before the WebSocket handshake completes). The IMPL does not prescribe HOW the JWT is transmitted.

#### Security Analysis of JWT Transmission Options

| Option | Mechanism | Security Risk | Compatibility |
|--------|-----------|---------------|---------------|
| **A: Query parameter** (`/ws?token=JWT`) | JWT in URL | Token leaks into server access logs, proxy logs, browser history, Referer headers. OWASP warns against credentials in URLs. | Universal — works in all browsers |
| **B: First-message auth** | Upgrade unauthenticated, client sends `{"type":"auth","token":"JWT"}` as first WS message | Brief unauthenticated connection window. Server must enforce auth timeout. | Universal — works in all browsers |
| **C: Sec-WebSocket-Protocol header** | Client sends token via subprotocol, server echoes back | Abuses the subprotocol mechanism. Token appears in network inspector. Non-standard. | Works but semantically incorrect |
| **D: Short-lived WebSocket ticket** | Client exchanges long-lived JWT for a single-use, short-lived ticket via REST, then uses ticket in query param | No long-lived token in URL. Ticket expires in seconds. Extra HTTP round-trip. | Universal |

#### Selected Approach: D — Short-Lived WebSocket Ticket

**Rationale**: This is the safest option that is compatible with the existing architecture.

1. Client calls `POST /api/v1/auth/ws-ticket` (authenticated with JWT in Authorization header)
2. Server generates a single-use, short-lived ticket (UUID, 30-second TTL, stored in Redis)
3. Client connects to `ws://host:8080/ws?ticket=<TICKET>`
4. Server validates the ticket against Redis (one-time use: delete after validation)
5. If valid → extract user context → proceed with WebSocket upgrade
6. If invalid/expired → reject with 401 before upgrade

**Security properties**:
- Long-lived JWT never appears in URL/logs
- Ticket is single-use (consumed on first use)
- Ticket has short TTL (30 seconds)
- Ticket is bound to user ID (stored in Redis: `ws:ticket:{uuid} → user_id`)
- No new authentication system introduced — uses existing JWT + Redis

### Pipeline Architecture

```
[Any Service] → Publisher.Publish(ctx, "mp:events:{type}", jsonPayload)
        ↓
   Redis Pub/Sub
        ↓
[RealtimeBridge] ← Subscriber.Subscribe(ctx, "mp:events:*")
        ↓
   Deserialize → Event envelope
        ↓
   Hub.Broadcast(channel, message)
        ↓
   Client.Send channel → gorilla/websocket.WriteMessage
        ↓
   Connected Browser
```

### Event Envelope

```go
type Event struct {
    Type      string          `json:"type"`       // e.g. "system.heartbeat", "watchlist.updated"
    Channel   string          `json:"channel"`    // e.g. "system.health"
    Payload   json.RawMessage `json:"payload"`
    Timestamp time.Time       `json:"timestamp"`
}
```

### Topic/Channel Naming
- Redis channels: `mp:events:system`, `mp:events:watchlist` (colon-separated per Redis convention)
- WebSocket subscription channels: `system.health` (dot-separated for client)
- Foundation implements only: `mp:events:system` with a heartbeat event

### File-Level Plan

| Action | File Path | Purpose |
|--------|-----------|---------|
| MODIFY | `Backend/internal/websocket/hub.go` | Add `Unsubscribe`, `ChannelCount` methods |
| CREATE | `Backend/internal/websocket/client_conn.go` | Wraps gorilla/websocket.Conn — ReadPump, WritePump, ping/pong, disconnect cleanup |
| CREATE | `Backend/internal/websocket/upgrade.go` | Gin handler: validates ticket from Redis, upgrades to WS, creates Client |
| CREATE | `Backend/internal/websocket/ticket.go` | `TicketService` — generates and validates single-use WS tickets via Redis |
| CREATE | `Backend/internal/websocket/bridge.go` | `RealtimeBridge` — subscribes to Redis, forwards events to Hub. Fx lifecycle. |
| MODIFY | `Backend/internal/routes/routes.go` | Add `GET /ws` route + `POST /api/v1/auth/ws-ticket` route |
| MODIFY | `Backend/internal/bootstrap/bootstrap.go` | Add `fx.Provide(websocket.NewHub, websocket.NewTicketService, websocket.NewRealtimeBridge)` + lifecycle hooks |

### Connection Lifecycle
1. Client calls `POST /api/v1/auth/ws-ticket` with JWT → receives `{ticket: "uuid"}`
2. Client connects to `ws://host:8080/ws?ticket=<TICKET>`
3. Server validates ticket in Redis (single-use: deleted after lookup)
4. Server upgrades connection, creates `Client{ID: connectionID, UserID: userID}`
5. Client sends `{"action":"subscribe","channel":"system.health"}`
6. Hub adds client to channel subscription
7. Events published to Redis → Bridge receives → Hub broadcasts to subscribed clients
8. On disconnect → Hub.Unregister cleans up all subscriptions

### Backpressure
- Client `Send` channel buffer size: 256
- If buffer full, message dropped, warning logged
- No blocking of the bridge goroutine

### Graceful Shutdown
- Bridge cancels its context → subscription closes → read loop exits
- Fx OnStop handles shutdown ordering

### Tests

| Test File | Coverage |
|-----------|----------|
| Extend `internal/websocket/hub_test.go` | Add Unsubscribe, ChannelCount tests |
| `internal/websocket/ticket_test.go` | Ticket generation, validation, single-use, expiry |
| `internal/websocket/bridge_test.go` | Mock Redis PubSub, verify event reaches Hub |
| `internal/websocket/upgrade_test.go` | httptest server with WebSocket upgrade, verify ticket auth |

### Acceptance Criteria
- `POST /api/v1/auth/ws-ticket` returns a ticket (authenticated)
- WebSocket upgrade succeeds with valid ticket
- WebSocket upgrade fails with invalid/expired/reused ticket
- Event published to Redis is received by subscribed WebSocket client
- Client disconnect cleans up subscriptions
- Ticket cannot be reused (single-use enforced)

---

## R6 — Background Processing Foundation

### Objective
Add Asynq (task queue) and gocron/v2 (scheduler) to the application, wired into Fx lifecycle.

### Dependencies
- R1 (Redis connection for Asynq, `go.mod` updated to include both packages)

### Architecture Verification
- **TECHNOLOGY_STACK.md**: Asynq = GREEN (APPROVED, IMPL_002), gocron/v2 = GREEN (APPROVED, IMPL_001)
- **IMPL_007 L170**: Lists Asynq as the Queue technology
- **go.mod**: Neither dependency exists yet — must be added

### Architecture
```
gocron scheduler
    → enqueues tasks via Asynq client
        → Redis queue
            → Asynq server (worker)
                → task handler
```

### File-Level Plan

| Action | File Path | Purpose |
|--------|-----------|---------|
| CREATE | `Backend/internal/infrastructure/queue/client.go` | `NewAsynqClient` — Fx constructor |
| CREATE | `Backend/internal/infrastructure/queue/server.go` | `NewAsynqServer` + `StartWorker` — Fx lifecycle |
| CREATE | `Backend/internal/infrastructure/queue/tasks.go` | Foundation task: `system:health_check` |
| CREATE | `Backend/internal/infrastructure/scheduler/scheduler.go` | `NewScheduler` + `RegisterFoundationJobs` — Fx lifecycle |
| MODIFY | `Backend/internal/bootstrap/bootstrap.go` | Add Fx providers for queue + scheduler |
| MODIFY | `Backend/internal/config/config.go` | Add `Queue QueueConfig` to Config struct |
| MODIFY | `Backend/configs/config.yaml` | Add `queue:` section |
| MODIFY | `Backend/.env.example` | Add `MP_QUEUE_*` variables |

### Task Processing Policy
- Default retry: 3 attempts with exponential backoff
- Task timeout: 30 seconds
- Failed tasks logged via Zap
- Foundation tasks are idempotent (health check has no side effects)

### Tests

| Test File | Coverage |
|-----------|----------|
| `internal/infrastructure/queue/tasks_test.go` | Health check task handler |
| `internal/infrastructure/scheduler/scheduler_test.go` | Verify job registration |

### Acceptance Criteria
- Asynq server starts alongside the application
- gocron enqueues a health-check task every 5 minutes
- Asynq worker picks up and executes the task
- Task result is logged
- Graceful shutdown stops scheduler then worker

---

## R7 — Frontend State Architecture Decision

### Objective
Provide a recommendation for the frontend state management decision that is currently `[DECISION REQUIRED]`.

### Status: `[DECISION REQUIRED]` — THIS IS A RECOMMENDATION, NOT A RESOLVED DECISION

No existing architecture document specifies a concrete state management library:
- **Volume 5 L4179**: "Do not mandate specific state management libraries."
- **IMPL_013 L20**: "Redux/Zustand or equivalent central state" (directional, not binding)
- **IMPL_014 L20**: "SWR/React Query for clientside hydration" (directional, not binding)
- **TECHNOLOGY_STACK.md**: Lists only Next.js and React. No state library.
- **DECISION_REGISTER.md**: No frontend state management entry exists.

### Recommendation: TanStack Query + Zustand

| Concern | Recommended Solution | Rationale |
|---------|---------------------|-----------|
| Server state (API data) | **TanStack Query** (`@tanstack/react-query`) | Automatic caching, background refetch, stale-while-revalidate, mutations — purpose-built for server state in React. Aligns with IMPL_014's mention of "React Query for clientside hydration." |
| Client state (UI, auth) | **Zustand** | Minimal API, no boilerplate, works with Next.js App Router server/client component boundary, tree-shakeable. Aligns with IMPL_013's mention of "Zustand or equivalent." |
| Realtime state (WS) | TanStack Query `queryClient.setQueryData` from WS messages | Integrates realtime updates into the same cache layer as REST data |

### Why This Combination Fits MarketPulse Pro
1. **Enterprise maintainability**: Zustand stores are simple, testable functions. TanStack Query handles complex cache invalidation automatically.
2. **Next.js App Router compatibility**: Both work cleanly with the server/client component boundary. No React Context provider nesting hell.
3. **Realtime-first**: TanStack Query's cache can be externally mutated by WebSocket events, avoiding stale data.
4. **Scale**: Zustand is O(1) subscription updates. TanStack Query deduplicates identical network requests.
5. **Directional alignment**: IMPL_013 lists "Zustand", IMPL_014 lists "React Query" — this combination satisfies both hints.

### Why NOT Redux
- Excessive boilerplate for this project's bounded state requirements
- TanStack Query eliminates most of what Redux was historically used for (API caching)
- Zustand covers the remaining client-state needs with significantly less complexity

### Why NOT React Context alone
- No built-in caching, deduplication, or background refresh
- Performance degrades with deeply nested providers and frequent updates

### Decision Register Update Required
The implementing agent **MUST NOT** install these dependencies until the user explicitly approves. Upon approval:
1. Add to `DECISION_REGISTER.md`:
   - `DEC-FRONTEND-001`: Frontend State Management — TanStack Query + Zustand
   - Status: RESOLVED
   - Evidence: User approval + IMPL_013/IMPL_014 directional guidance
2. Add to `TECHNOLOGY_STACK.md`: Both libraries with GREEN status

### Dependencies to Install (upon approval only)
```json
{
  "@tanstack/react-query": "^5.x",
  "zustand": "^5.x"
}
```

### `[DECISION REQUIRED]` Gate
**R8 (Functional Frontend Foundation) CANNOT begin until this decision is explicitly approved by the user/architect.** If the recommendation is rejected, the implementing agent must use the approved alternative.

---

## R8 — Functional Frontend Foundation

### Objective
Build the minimum application shell that can render, authenticate, and communicate with the backend.

### Dependencies
- R4 (backend APIs operational)
- R7 (state management decision **APPROVED** — blocks until then)

### Foundation vs Product Scope Boundary

This remediation unit builds **ONLY** the application shell. The following are explicitly OUT OF SCOPE and must NOT be implemented:

| OUT OF SCOPE | Reason |
|-------------|--------|
| Option Chain UI | Product feature (SPEC_001, IMPL_001) |
| Market Data dashboard widgets | Product feature (IMPL_014) — requires broker data feeds |
| Strategy Builder UI | Product feature (IMPL_013) — requires option chain + pricing |
| Broker Login modal | Blocked by DEC-ARCH-004C/D/E |
| Trading functionality | Product feature — requires broker integration |
| Charts / visualization | Product feature — requires market data |
| Real-time market-data display | Product feature — requires broker data feeds |
| Screener | Product feature (SPEC_003) |

What IS in scope:
- Application shell (layout, navigation skeleton, error boundary)
- Authentication pages (login, register) — minimal functional forms
- Route protection middleware
- API client integration with auth token injection
- WebSocket client foundation (connect, subscribe, receive)
- Minimal CRUD pages for watchlists and strategies (smoke-test level only — list + create + delete)
- State management integration (per R7 decision)

### File-Level Plan

#### Application Shell

| Action | File Path | Purpose |
|--------|-----------|---------|
| MODIFY | `Frontend/src/app/layout.tsx` | Add Providers wrapper, font loading |
| MODIFY | `Frontend/src/app/page.tsx` | Redirect to `/dashboard` if authenticated, `/login` if not |
| CREATE | `Frontend/src/app/(auth)/login/page.tsx` | Login form — email + password + submit |
| CREATE | `Frontend/src/app/(auth)/register/page.tsx` | Register form — name + email + password + submit |
| CREATE | `Frontend/src/app/(app)/layout.tsx` | Authenticated layout with sidebar/header |
| CREATE | `Frontend/src/app/(app)/dashboard/page.tsx` | Minimal "Welcome, {name}" page — NO market data widgets |
| CREATE | `Frontend/src/app/(app)/watchlists/page.tsx` | List watchlists + create form — smoke test CRUD only |
| CREATE | `Frontend/src/app/(app)/strategies/page.tsx` | List strategies + create form — smoke test CRUD only |

#### State Management (conditional on R7 approval)

| Action | File Path | Purpose |
|--------|-----------|---------|
| CREATE | `Frontend/src/providers/query-provider.tsx` | TanStack Query client provider (`"use client"`) |
| CREATE | `Frontend/src/stores/auth-store.ts` | Zustand store: `{token, user, login, logout, isAuthenticated}` |
| CREATE | `Frontend/src/hooks/use-auth.ts` | Hook wrapping auth store + localStorage token persistence |

#### API Layer

| Action | File Path | Purpose |
|--------|-----------|---------|
| MODIFY | `Frontend/src/lib/api.ts` | Add auth header injection, typed methods for auth/watchlists/strategies |
| CREATE | `Frontend/src/lib/ws.ts` | WebSocket client: ticket exchange, connect, subscribe/unsubscribe, reconnect with backoff |

#### UI Foundation Components (minimal — NOT a full design system)

| Action | File Path | Purpose |
|--------|-----------|---------|
| CREATE | `Frontend/src/components/ui/button.tsx` | Reusable button with variants |
| CREATE | `Frontend/src/components/ui/input.tsx` | Reusable form input |
| CREATE | `Frontend/src/components/ui/loading.tsx` | Loading spinner |
| CREATE | `Frontend/src/components/ui/error-boundary.tsx` | React error boundary |
| CREATE | `Frontend/src/components/layout/sidebar.tsx` | Navigation sidebar (placeholder links for: Home, Trade, Analyse, Watchlist, Positions, Orders) |
| CREATE | `Frontend/src/components/layout/header.tsx` | Top bar with user info + logout |

#### Route Protection

| Action | File Path | Purpose |
|--------|-----------|---------|
| CREATE | `Frontend/src/middleware.ts` | Next.js middleware — redirects unauthenticated users to `/login` |

#### Configuration

| Action | File Path | Purpose |
|--------|-----------|---------|
| MODIFY | `Frontend/.env.example` | Add `NEXT_PUBLIC_WS_URL` |
| MODIFY | `Frontend/src/styles/globals.css` | Extend design tokens for component system |
| MODIFY | `Frontend/package.json` | Add approved state management dependencies |

### Tests
- `npm run build` must succeed without errors
- `npm run lint` must pass
- Formal frontend testing framework selection is deferred — but all pages must be manually verifiable

### Acceptance Criteria
- `npm run build` completes without errors
- Login page renders, submits credentials, receives JWT, stores in state, redirects to dashboard
- Dashboard shows "Welcome, {name}" — no market data widgets
- Unauthenticated access to `/dashboard` redirects to `/login`
- Watchlist page lists watchlists from API (smoke test)
- WebSocket client connects successfully when user is authenticated
- Navigation sidebar renders with placeholder links for future pages

---

## R9 — Architecture Compliance Hardening

### Objective
Verify and lock down architecture compliance after R1–R8 are implemented.

### Architecture Preservation Matrix

| Technology | Remediation Impact | Must Preserve | Must Not Introduce | Pending Dependency |
|-----------|-------------------|---------------|-------------------|-------------------|
| Go/Gin | Extended with handlers/middleware | Gin router patterns, handler signatures | No alternative HTTP frameworks | — |
| Uber Fx | Extended with more providers/invokes | Module composition pattern | No manual DI | — |
| Viper | Extended with JWT + queue config | Config struct + env override pattern | No alternative config libs | — |
| Zap | Used in all new services/handlers | Structured logging pattern | No alternative loggers | — |
| PostgreSQL/GORM | Tables created, repositories added | Connection pooling, `WithContext` pattern | No raw SQL for CRUD | — |
| golang-migrate | NEW — used for schema management | Versioned SQL files, up/down pattern | No GORM AutoMigrate for schemas | — |
| ClickHouse | Untouched | Non-fatal startup connection | No ClickHouse tables in foundation | — |
| Redis | Used by Pub/Sub, Cache, Asynq, WS tickets | Shared client pattern | No separate Redis libraries | — |
| Redis Pub/Sub | Wired to WebSocket Hub via Bridge | Publisher/Subscriber abstraction | No Kafka/NATS | DEC-ARCH-007 (RESOLVED) |
| Gorilla WebSocket | NOW actually used for HTTP upgrade | Upgrader + ReadPump/WritePump | No alternative WS libraries | — |
| Asynq | NEW — task queue | Redis-backed queue pattern | No Celery/Sidekiq equivalents | — |
| gocron/v2 | NEW — scheduler | Cron-like scheduling | No APScheduler | — |
| JWT | Config-driven, not hardcoded | HS256 signing via golang-jwt | No new auth providers | `[DECISION REQUIRED]` secret mgmt |
| Next.js/React | Extended with pages/components | App Router, server components | No framework changes | — |
| TanStack Query | NEW (conditional on R7) | Server state caching | — | `[DECISION REQUIRED]` R7 |
| Zustand | NEW (conditional on R7) | Client state store | — | `[DECISION REQUIRED]` R7 |
| Docker (dev) | Untouched | `docker-compose.dev.yml` | No production Docker changes | DEC-ARCH-008 |
| Testify | Extended with new tests | assert/require/mock pattern | No alternative test frameworks | — |

### Prohibited Technology Verification
The implementing agent must run these checks after implementation:

```bash
# Must return zero results:
grep -ri "upstox" Backend/ Frontend/ --include="*.go" --include="*.ts" --include="*.tsx"
grep -ri "kafka" Backend/ --include="*.go"
grep -ri "nats" Backend/ --include="*.go"
grep -ri "apscheduler" Backend/ Frontend/
grep -ri "parquet" Backend/ --include="*.go"
grep -ri "automigrate" Backend/ --include="*.go"
```

Note: The last check ensures GORM AutoMigrate is not used for schema management.

### `go.mod` Direct Dependencies Audit
After all remediation, `go.mod` direct `require` block must contain exactly:
- `github.com/gin-gonic/gin` (existing)
- `github.com/redis/go-redis/v9` (existing)
- `github.com/spf13/viper` (existing)
- `github.com/stretchr/testify` (existing)
- `go.uber.org/fx` (existing)
- `go.uber.org/zap` (existing)
- `gorm.io/driver/postgres` (existing)
- `gorm.io/gorm` (existing)
- `github.com/ClickHouse/clickhouse-go/v2` (existing)
- `github.com/golang-jwt/jwt/v5` (promoted from indirect)
- `github.com/gorilla/websocket` (promoted from indirect)
- `golang.org/x/crypto` (promoted from indirect — for bcrypt)
- `github.com/google/uuid` (new — for request IDs and WS tickets)
- `github.com/golang-migrate/migrate/v4` (new — for database migrations)
- `github.com/hibiken/asynq` (new — for task queue)
- `github.com/go-co-op/gocron/v2` (new — for scheduling)

No other new direct dependencies.

### Frontend `package.json` Audit
After remediation, new dependencies must be exactly:
- `@tanstack/react-query` (conditional on R7 approval)
- `zustand` (conditional on R7 approval)

No other new dependencies.

---

## 4. Observability Plan

### Structured Logging (Zap)
All new services and handlers must:
- Log on entry/exit of significant operations at `Debug` level
- Log errors at `Error` level with `zap.Error(err)`
- Include `zap.String("user_id", ...)` where available
- Include `zap.String("request_id", ...)` from middleware context
- **Never log passwords, tokens, secrets, or ticket values**

### Request ID Middleware
- `request_id.go`: Generates UUID per request, sets `X-Request-ID` response header, stores in Gin context
- Downstream handlers and services can retrieve it for structured logging

### Prometheus/OpenTelemetry/Sentry
- **NOT wired in foundation remediation** — integration patterns are not yet defined at sufficient detail for foundation
- Foundation code is structured to allow easy instrumentation later (discrete service methods, consistent error types)
- `[DECISION REQUIRED]`: Exact Prometheus metric naming, OpenTelemetry exporter config, Sentry DSN

### Future Metrics (not implemented now, but architecture supports)
- HTTP request latency (Prometheus histogram)
- Auth failure count (Prometheus counter)
- WebSocket connection count (Prometheus gauge)
- Asynq queue depth (Prometheus gauge)
- DB query latency (GORM plugin)

---

## 5. Security Review

| Area | Current State | Remediation Action | Risk |
|------|--------------|-------------------|------|
| Passwords | Model has `Password string` with `json:"-"` | Service hashes with bcrypt before storage | LOW after fix |
| JWT Secret | Hardcoded `"changeme-insecure-dev-key"` in `jwt.go` L45 | Config-driven via Viper + env override. `[DECISION REQUIRED]` for production rotation. | MEDIUM (dev acceptable) |
| SQL Injection | GORM parameterized queries | Preserved — no raw SQL in handlers/services | LOW |
| Auth Header | Middleware validates `Bearer` token | Already implemented (`middleware/auth.go`), no changes needed | LOW |
| WebSocket Auth | Not implemented | Short-lived ticket via Redis (single-use, 30s TTL). No JWT in URLs. | LOW after fix |
| Error Leakage | `pkg/errors` wraps internal errors | Handlers use `response.Error()` which never exposes internal details | LOW |
| Sensitive Logging | `json:"-"` on `User.Password`, `Session.AccessToken` | Services must not log passwords or tokens. WS tickets must not be logged. | LOW with discipline |
| CORS | Not configured | Dev-only: allow `http://localhost:3000`. `[DECISION REQUIRED]` for production origins. | LOW (dev) |
| Rate Limiting | Not implemented | DEFERRED to hardening phase | MEDIUM |
| Redis Auth | Empty password in dev config | Config supports password field; production must set it. | LOW (dev) |
| Request IDs | Not implemented | Added via middleware — enables audit trail correlation | LOW |
| JWT in Frontend | Stored in client-side state | Zustand + localStorage. `[DECISION REQUIRED]`: HttpOnly cookie approach for production. | MEDIUM |

---

## 6. Performance / Scale Considerations

| Concern | Foundation Approach | Future Scale Path |
|---------|-------------------|-------------------|
| HTTP Statelessness | Gin handlers stateless, JWT auth stateless | Horizontally scalable behind load balancer |
| DB Connection Pool | Configured: 25 open, 10 idle, 5min lifetime (config.go) | Tunable via Viper/env |
| Redis | Single client shared across cache, pubsub, asynq, tickets | Can split into separate Redis instances via config |
| WebSocket Fanout | Hub uses in-memory subscriptions (single-server) | For multi-server: Redis-backed Hub (future, not foundation) |
| Queue Concurrency | Asynq configurable (default: 10 workers) | Horizontally scalable by adding workers |
| ClickHouse | Not used in foundation | Separate analytical workload |
| Migration Performance | golang-migrate with connection pooling | No concern for 5 small tables |

**No premature optimization. No Kafka. No multi-server WebSocket in foundation.**

---

## 7. Risk Register

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| golang-migrate version conflicts with GORM | LOW | MEDIUM | Both use pgx under the hood. golang-migrate manages DDL only, GORM handles DML. No overlap. |
| Frontend state library not approved | MEDIUM | HIGH | R7 is a `[DECISION REQUIRED]` gate. R8 blocks until approval. |
| WebSocket single-server limitation | LOW | LOW | Documented as future enhancement. Redis-backed Hub is an additive change. |
| Secret hardcoded in dev config | HIGH (likelihood) | LOW (impact — dev only) | `[DECISION REQUIRED]` documented. Config override via env vars available for production. |
| Broker API premature implementation | LOW | HIGH | Explicit governance constraint in plan. grep check in R9. |
| Overbuilding R8 frontend beyond foundation | MEDIUM | MEDIUM | Plan explicitly lists OUT OF SCOPE items. No option chain, no charts, no market data. |
| ClickHouse schema drift | LOW | LOW | No ClickHouse schemas in this remediation. |
| Test coverage gaps | MEDIUM | MEDIUM | Each unit has test requirements defined. Integration tests require docker-compose infra. |
| golang-migrate not in go.mod | CERTAIN | LOW | Must be added as direct dependency in R1. |
| `google/uuid` not in go.mod | CERTAIN | LOW | Must be added as direct dependency for request IDs and WS tickets. |

---

## 8. Testing Strategy Summary

| Layer | Framework | Location Pattern | Infrastructure Required |
|-------|-----------|-----------------|------------------------|
| Unit (services) | Testify + mock repos | `internal/modules/{domain}/services/*_test.go` | None |
| Unit (handlers) | Testify + httptest | `internal/modules/{domain}/*handler*_test.go` | None |
| Unit (auth) | Testify | `internal/core/auth/*_test.go` (extend existing) | None |
| Unit (queue tasks) | Testify | `internal/infrastructure/queue/*_test.go` | None |
| Unit (websocket) | Testify | `internal/websocket/*_test.go` (extend existing) | None |
| Unit (migration) | Testify | `internal/core/database/migrate_test.go` | PostgreSQL |
| Integration (API) | Testify + httptest + live PG | Future `tests/integration/` | PostgreSQL, Redis |
| Frontend | `npm run build` + `npm run lint` | — | None |

---

## 9. Definition of Done

A remediation unit is DONE only when:

1. ✅ All planned files are created or modified
2. ✅ `go build ./...` compiles without errors
3. ✅ `go vet ./...` reports no issues
4. ✅ `gofmt -l .` returns empty (all code formatted)
5. ✅ `go test ./...` — all tests pass
6. ✅ `npm run build` (Frontend) — compiles without errors
7. ✅ No hardcoded secrets in source code (except clearly marked dev defaults in config)
8. ✅ No prohibited technologies introduced (verified by grep)
9. ✅ No GORM AutoMigrate used for schema management
10. ✅ No pending decisions silently resolved
11. ✅ `[DECISION REQUIRED]` markers present where governance requires them
12. ✅ Architecture compliance checks pass (R9 grep commands return 0)
13. ✅ Migration files are version-controlled, sequential, and reversible (per IMPL_003)

---

## 10. Final Readiness Gate

The foundation is ready for product-feature implementation when ALL of the following are true:

| Gate | Evidence |
|------|----------|
| Database tables exist | golang-migrate runs all up migrations; 5 tables + schema_migrations in PostgreSQL |
| Auth flow works | `POST /api/v1/auth/register` → `POST /api/v1/auth/login` → `GET /api/v1/user/profile` returns user |
| CRUD APIs work | Watchlist and Strategy CRUD return correct data |
| WebSocket connects | Client exchanges JWT for ticket, connects to `/ws?ticket=UUID`, receives events |
| Background processing runs | Asynq worker processes health-check task |
| Frontend renders | Login → Dashboard → Watchlist navigation works |
| All tests pass | `go test ./...` and `npm run build` succeed |
| Architecture compliant | No prohibited tech, no AutoMigrate, no silently resolved decisions |
| Frontend state decided | DEC-FRONTEND-001 is RESOLVED (user approved) |

**Only when all gates pass is MarketPulse Pro ready for product-feature implementation.**

---

## 11. Complete File Inventory

### Files to CREATE (Backend: 29)

| # | Path | Remediation Unit |
|---|------|-----------------|
| 1 | `migrations/000001_create_users_table.up.sql` | R1 |
| 2 | `migrations/000001_create_users_table.down.sql` | R1 |
| 3 | `migrations/000002_create_watchlists_table.up.sql` | R1 |
| 4 | `migrations/000002_create_watchlists_table.down.sql` | R1 |
| 5 | `migrations/000003_create_watchlist_items_table.up.sql` | R1 |
| 6 | `migrations/000003_create_watchlist_items_table.down.sql` | R1 |
| 7 | `migrations/000004_create_strategies_table.up.sql` | R1 |
| 8 | `migrations/000004_create_strategies_table.down.sql` | R1 |
| 9 | `migrations/000005_create_strategy_legs_table.up.sql` | R1 |
| 10 | `migrations/000005_create_strategy_legs_table.down.sql` | R1 |
| 11 | `internal/core/database/migrate.go` | R1 |
| 12 | `internal/core/database/migrate_test.go` | R1 |
| 13 | `internal/modules/user/services/user_service.go` | R2 |
| 14 | `internal/modules/user/services/user_service_test.go` | R2 |
| 15 | `internal/modules/watchlist/interfaces/repository.go` | R2 |
| 16 | `internal/modules/watchlist/repositories/watchlist_repository.go` | R2 |
| 17 | `internal/modules/watchlist/services/watchlist_service.go` | R2 |
| 18 | `internal/modules/watchlist/services/watchlist_service_test.go` | R2 |
| 19 | `internal/modules/strategy/interfaces/repository.go` | R2 |
| 20 | `internal/modules/strategy/repositories/strategy_repository.go` | R2 |
| 21 | `internal/modules/strategy/services/strategy_service.go` | R2 |
| 22 | `internal/modules/strategy/services/strategy_service_test.go` | R2 |
| 23 | `internal/modules/auth/handler.go` | R3 |
| 24 | `internal/modules/auth/handler_test.go` | R3 |
| 25 | `internal/modules/user/handler.go` | R4 |
| 26 | `internal/modules/watchlist/handler.go` | R4 |
| 27 | `internal/modules/strategy/handler.go` | R4 |
| 28 | `internal/middleware/request_id.go` | R4 |
| 29 | `internal/middleware/cors.go` | R4 |
| 30 | `internal/websocket/client_conn.go` | R5 |
| 31 | `internal/websocket/upgrade.go` | R5 |
| 32 | `internal/websocket/ticket.go` | R5 |
| 33 | `internal/websocket/bridge.go` | R5 |
| 34 | `internal/websocket/ticket_test.go` | R5 |
| 35 | `internal/websocket/bridge_test.go` | R5 |
| 36 | `internal/infrastructure/queue/client.go` | R6 |
| 37 | `internal/infrastructure/queue/server.go` | R6 |
| 38 | `internal/infrastructure/queue/tasks.go` | R6 |
| 39 | `internal/infrastructure/queue/tasks_test.go` | R6 |
| 40 | `internal/infrastructure/scheduler/scheduler.go` | R6 |
| 41 | `internal/infrastructure/scheduler/scheduler_test.go` | R6 |

### Files to MODIFY (Backend: 12)

| # | Path | Remediation Unit |
|---|------|-----------------|
| 1 | `go.mod` (+ `go.sum` via `go mod tidy`) | R1 |
| 2 | `internal/bootstrap/bootstrap.go` | R1, R4, R5, R6 |
| 3 | `internal/config/config.go` | R3, R6 |
| 4 | `configs/config.yaml` | R3, R6 |
| 5 | `.env.example` | R3, R6 |
| 6 | `internal/core/auth/jwt.go` | R3 |
| 7 | `internal/core/server/server.go` | R4 |
| 8 | `internal/routes/routes.go` | R4, R5 |
| 9 | `internal/modules/strategy/models/strategy.go` | R1 |
| 10 | `internal/modules/watchlist/models/watchlist.go` | R1 |
| 11 | `internal/websocket/hub.go` | R5 |
| 12 | `internal/websocket/hub_test.go` | R5 |

### Files to CREATE (Frontend: 18)

| # | Path | Remediation Unit |
|---|------|-----------------|
| 1 | `src/providers/query-provider.tsx` | R8 |
| 2 | `src/stores/auth-store.ts` | R8 |
| 3 | `src/hooks/use-auth.ts` | R8 |
| 4 | `src/lib/ws.ts` | R8 |
| 5 | `src/app/(auth)/login/page.tsx` | R8 |
| 6 | `src/app/(auth)/register/page.tsx` | R8 |
| 7 | `src/app/(app)/layout.tsx` | R8 |
| 8 | `src/app/(app)/dashboard/page.tsx` | R8 |
| 9 | `src/app/(app)/watchlists/page.tsx` | R8 |
| 10 | `src/app/(app)/strategies/page.tsx` | R8 |
| 11 | `src/components/ui/button.tsx` | R8 |
| 12 | `src/components/ui/input.tsx` | R8 |
| 13 | `src/components/ui/loading.tsx` | R8 |
| 14 | `src/components/ui/error-boundary.tsx` | R8 |
| 15 | `src/components/layout/sidebar.tsx` | R8 |
| 16 | `src/components/layout/header.tsx` | R8 |
| 17 | `src/middleware.ts` | R8 |

### Files to MODIFY (Frontend: 5)

| # | Path | Remediation Unit |
|---|------|-----------------|
| 1 | `src/app/layout.tsx` | R8 |
| 2 | `src/app/page.tsx` | R8 |
| 3 | `src/lib/api.ts` | R8 |
| 4 | `src/styles/globals.css` | R8 |
| 5 | `.env.example` | R8 |
| 6 | `package.json` | R7/R8 |

### Inventory Totals
- **Backend CREATE**: 41 files
- **Backend MODIFY**: 12 files
- **Frontend CREATE**: 17 files
- **Frontend MODIFY**: 6 files
- **Grand Total**: 76 files

### Files NOT Modified
- All `Architecture/*.md` documents (read-only — except `DECISION_REGISTER.md` for R7 upon user approval)
- `docker-compose.dev.yml` — no changes needed
- `Makefile` — optionally add `migrate-up` / `migrate-down` targets (not required)
- `internal/modules/broker/interfaces.go` — untouched (blocked)
- `internal/modules/marketdata/models/market.go` — untouched (no DB table, analytics only)

---

## 12. Dependency Audit

| Dependency | New/Existing | Direct/Indirect | Approved | Necessary For | Already in go.mod? |
|-----------|-------------|-----------------|----------|--------------|-------------------|
| `github.com/golang-jwt/jwt/v5` | Existing | Promote to direct | YES (TECHNOLOGY_STACK) | JWT service imports it | Yes (indirect) |
| `github.com/gorilla/websocket` | Existing | Promote to direct | YES (TECHNOLOGY_STACK) | WebSocket upgrade | Yes (indirect) |
| `golang.org/x/crypto` | Existing | Promote to direct | YES (standard lib extension) | bcrypt password hashing | Yes (indirect via pgx) |
| `github.com/google/uuid` | New | Direct | YES (standard utility) | Request IDs, WS tickets | No |
| `github.com/golang-migrate/migrate/v4` | New | Direct | YES (IMPL_003 L2193) | Database migrations | No |
| `github.com/hibiken/asynq` | New | Direct | YES (TECHNOLOGY_STACK — GREEN) | Task queue | No |
| `github.com/go-co-op/gocron/v2` | New | Direct | YES (TECHNOLOGY_STACK — GREEN) | Scheduling | No |
| `@tanstack/react-query` | New (FE) | Direct | `[DECISION REQUIRED]` | Server state management | No |
| `zustand` | New (FE) | Direct | `[DECISION REQUIRED]` | Client state management | No |

---

## 13. Blocked Areas (Cannot Be Implemented)

| Area | Blocker | What Must Not Be Done |
|------|---------|----------------------|
| Zerodha adapter | DEC-ARCH-004C PENDING | No Zerodha API calls, SDK imports, or credential handling |
| Angel One adapter | DEC-ARCH-004D PENDING | No Angel One API calls, SDK imports, or credential handling |
| ICICI Direct adapter | DEC-ARCH-004E PENDING | No ICICI Direct API calls, SDK imports, or credential handling |
| Data pipelines | DEC-ARCH-005/006 PENDING/DEFERRED | No S3 uploads, Parquet writes |
| Production deployment | DEC-ARCH-008 PENDING | No production Dockerfiles, Nginx configs, CI/CD |
| Secret management | Unresolved | No HashiCorp Vault, AWS Secrets Manager, etc. |
| Frontend state (R7) | `[DECISION REQUIRED]` | No installation until user approval |

---

## 14. Critical Pre-Implementation Decisions

Before the implementing agent begins, the following must be explicitly resolved:

1. **Frontend State Management (R7)**: User must approve TanStack Query + Zustand recommendation, or provide an alternative. R8 is blocked until this is resolved.

All other remediation units (R1–R6, R9) can proceed without additional decisions. The migration strategy is already resolved by IMPL_003 (golang-migrate).

---

## 15. Final Recommendation

### Verdict: **B — APPROVED WITH CORRECTIONS**

The original plan contained three governance violations that have been corrected in this version:

| Issue | Original Plan | Corrected Plan | Governance Authority |
|-------|--------------|----------------|---------------------|
| **Migration strategy** | GORM AutoMigrate | `golang-migrate` with versioned SQL files | AGENTS.md L53, IMPL_003 L2193–2195, IMPL_012 L523 |
| **WebSocket auth** | JWT in query parameter (`/ws?token=JWT`) | Short-lived single-use ticket via Redis | OWASP credential-in-URL warning + IMPL_007 upgrade flow |
| **File inventory counts** | Header said "21 CREATE" but table had 29 | Corrected: 41 CREATE (backend), 17 CREATE (frontend), 12 MODIFY (backend), 6 MODIFY (frontend) | Reconciliation error |

Additional corrections applied:
- `google/uuid` added to dependency audit (was missing — needed for request IDs and WS tickets)
- `golang-migrate/migrate/v4` added to dependency audit
- User repository existence acknowledged in R2 (was omitted)
- Foundation vs product scope boundary made explicit in R8 with OUT OF SCOPE table
- Frontend state management strictly kept as `[DECISION REQUIRED]`, not treated as resolved
- Volume 5 L4179 ("Do not mandate specific state management libraries") cited as evidence
- `AutoMigrate` added to prohibited technology grep checks in R9
- Migration file structure aligned to IMPL_003 L2240–2252 specification
