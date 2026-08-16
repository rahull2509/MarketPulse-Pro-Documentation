# P4-03: CORE DOMAIN & DATA INTEGRATION IMPLEMENTATION PLAN

## 1. Forensic Inspection Summary
A deep forensic audit of the `MarketPulse Pro` repository was conducted to assess the readiness of the P4-03 domain integration. 
- **Backend:** The PostgreSQL schemas (`migrations/000002` through `000005`) are perfectly aligned with the `gorm` models in `watchlist.go` and `strategy.go`. The repositories and services enforce strict user-ownership validation. The Gin handlers exist but are missing critical endpoints (`GET /:id`, `DELETE /:id`, `DELETE /:id/items|legs`). The route registrations in `routes.go` are similarly incomplete.
- **Frontend:** `@tanstack/react-query` is installed and globally provided via `layout.tsx`. However, the UI pages (`/watchlist`, `/trade`) use mock structures and display `EmptyState` components referencing a "TRACEABILITY GAP". 
- **API Formats:** All existing handlers currently return direct `c.JSON()` objects. The `pkg/response` package is unused and out-of-sync with `pkg/apperrors`. Standardization will continue using direct `c.JSON()` to maintain consistency with existing working code.

## 2. Evidence Matrix

| Area | Governance Requirement | Actual Repository State | Evidence File | Status |
|------|------------------------|-------------------------|---------------|--------|
| PostgreSQL | Schema matches models | Exact match | `Backend/migrations/*.up.sql` | APPROVED |
| Migrations | No GORM AutoMigrate | `golang-migrate` used exclusively | `Backend/migrations/` | APPROVED |
| Fx DI | Dependency injection | Used to wire handlers & routes | `Backend/internal/routes/routes.go` | APPROVED |
| Identity | Scoped by JWT `user_id` | `c.GetUint("user_id")` used everywhere | `Backend/internal/modules/watchlist/handler.go` | APPROVED |
| Watchlist UI | Implement domain UI | Hardcoded empty state | `Frontend/src/app/(authenticated)/watchlist/page.tsx` | PENDING |
| Strategy UI | Implement domain UI | Hardcoded empty state | `Frontend/src/app/(authenticated)/trade/page.tsx` | PENDING |
| State Mgmt | TanStack Query + Zustand | Both installed and configured | `Frontend/package.json`, `layout.tsx` | APPROVED |

## 3. Current Architecture State
The repository adheres to the **Modular Monolith** pattern.
- **Repositories:** Abstracted via interfaces. GORM used exclusively inside repos.
- **Services:** Contain all business rules (e.g., maximum 50 items per watchlist) and ownership checks (`if entity.UserID != userID`). Return custom `apperrors`.
- **Handlers:** Extract identity, bind JSON, invoke services, return HTTP responses. Do not contain domain logic.
- **Data Boundaries:** Users, Watchlists, and Strategies are completely segregated into their respective modules.

## 4. P4-03 Scope
Transform the verified P3/P4 foundation into functional core application flows for:
1. User Profile (`/user/profile`)
2. Watchlist Management (CRUD watchlists + items)
3. Strategy Management (CRUD strategies + legs)
*Note: No broker integration, execution engine, or live market data ingestion is included in this phase.*

## 5. Domain Model Assessment
Models for `Watchlist`, `WatchlistItem`, `Strategy`, and `StrategyLeg` are complete, relational, and properly tagged for GORM mapping and JSON serialization.

## 6. Database Assessment
**READY.** Migrations `000001` through `000005` define primary keys, cascading foreign keys (`ON DELETE CASCADE`), indexes on `user_id`, and appropriate constraints. No further migrations are required for P4-03.

## 7. API Assessment
The following API endpoints must be implemented/completed:
- `DELETE /api/v1/watchlists/:id`
- `GET /api/v1/watchlists/:id`
- `DELETE /api/v1/watchlists/:id/items/:itemId`
- `DELETE /api/v1/strategies/:id`
- `GET /api/v1/strategies/:id`
- `DELETE /api/v1/strategies/:id/legs/:legId`
*(Routes and Handler methods need to be wired).*

## 8. Frontend Assessment
**PENDING.** The frontend requires the creation of standard React Query hooks (`useWatchlistsQuery`, `useCreateWatchlistMutation`, etc.) interacting with `lib/api.ts`. The UI pages must consume these hooks to replace the mocked "TRACEABILITY GAP" states with actual loading, empty, and error states.

## 9. Authorization Assessment
**APPROVED.** The backend rigorously extracts the `user_id` from the JWT middleware. The services enforce ownership by validating `entity.UserID == userID`. Cross-user data leakage is structurally prevented.

## 10. Testing Assessment
**PENDING.** We must ensure robust tests for backend services (ownership validation) and API routes (401/403/404 responses). Frontend requires visual verification of empty states and mutation success.

## 11. Exact Implementation File Inventory

### Backend
- **MODIFY:** `Backend/internal/modules/watchlist/handler.go` (Add `Get`, `Delete`, `RemoveItem`)
- **MODIFY:** `Backend/internal/modules/strategy/handler.go` (Add `Get`, `Delete`, `RemoveLeg`)
- **MODIFY:** `Backend/internal/routes/routes.go` (Map the new endpoints)

### Frontend
- **CREATE:** `Frontend/src/hooks/queries/use-watchlists.ts` (TanStack Query hooks)
- **CREATE:** `Frontend/src/hooks/queries/use-strategies.ts` (TanStack Query hooks)
- **CREATE:** `Frontend/src/hooks/queries/use-profile.ts` (TanStack Query hooks)
- **MODIFY:** `Frontend/src/app/(authenticated)/watchlist/page.tsx` (Wire hooks)
- **MODIFY:** `Frontend/src/app/(authenticated)/trade/page.tsx` (Wire hooks)

## 12. Traceability Matrix
- **Requirement:** User Profile → `TRACEABILITY SOURCE REQUIRES GOVERNANCE CONFIRMATION` → Backend `GET /user/profile` → Frontend `useProfileQuery`
- **Requirement:** Watchlist Domain → `REQ-WATCHLIST-001` (No Phase-1 SPEC exists) → Backend `WatchlistService` → Frontend `useWatchlistsQuery` & UI
- **Requirement:** Strategy Domain → `SPEC_011` → `IMPL_013_v2.0` → Backend `StrategyService` → Frontend `useStrategiesQuery` & UI

## 13. Risk Register
- **API Contract Drift (MEDIUM):** Services return `apperrors`, but handlers return raw HTTP 500s on failure. Handlers must be updated to map `apperrors` types to 400/403/404 statuses to prevent frontend misbehavior.
- **Over-fetching (LOW):** Watchlists eagerly preload all items. Fine for 50 items limit, but must be monitored.
- **Cross-user Data Leakage (LOW):** Protected by service-level ownership checks, but handler parameter parsing must remain vigilant.

## 14. Pending Decisions
- **Standardizing Error Responses:** The repository uses `c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})` instead of mapping `apperrors.AppError` types to 4xx codes. *Decision:* I will implement a lightweight error mapper in the handlers for P4-03 to map NotFound/Forbidden errors correctly to 404/403.

## 15. P4-03 Implementation Sequence
1. Update `watchlist/handler.go` and `strategy/handler.go` with missing endpoints and error-mapping logic.
2. Register new routes in `routes.go`.
3. Create frontend React Query hooks for Profile, Watchlists, and Strategies.
4. Integrate hooks into `/watchlist` and `/trade` pages, replacing mock data.
5. Run tests, build, and verify ownership boundaries locally.

## 16. Final Readiness Verdict

### P4-03 READINESS
- **CORE DOMAIN:** READY
- **DATABASE:** READY
- **WATCHLIST:** READY (Backend partially complete, UI pending)
- **STRATEGY:** READY (Backend partially complete, UI pending)
- **FRONTEND DATA INTEGRATION:** READY (TanStack configured)
- **AUTHORIZATION:** READY
- **TESTING:** READY (Frameworks in place)

**OVERALL: READY FOR IMPLEMENTATION**
