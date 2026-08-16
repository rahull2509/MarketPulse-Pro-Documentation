# P4-03: CORE DOMAIN & DATA INTEGRATION IMPLEMENTATION REPORT

## 1. Scope
The objective of P4-03 was to convert the verified P3/P4 foundation into functional application-level domain flows for the Core Domain (User/Profile, Watchlists, Strategies). The implementation aimed to strictly preserve all governance decisions, utilizing the existing PostgreSQL database, `golang-migrate` schemas, Gin routing, React components, and TanStack query infrastructure.

## 2. Files Created
- `Frontend/src/hooks/queries/use-profile.ts`
- `Frontend/src/hooks/queries/use-watchlists.ts`
- `Frontend/src/hooks/queries/use-strategies.ts`

## 3. Files Modified
- `Backend/internal/modules/watchlist/handler.go` (Added `Get`, `Delete`, `RemoveItem`, and standardized error mapping)
- `Backend/internal/modules/strategy/handler.go` (Added `Get`, `Delete`, `RemoveLeg`, and standardized error mapping)
- `Backend/internal/routes/routes.go` (Registered new routes)
- `Backend/pkg/response/response.go` (Created `HandleError` for accurate HTTP status mapping from `apperrors`)
- `Frontend/src/lib/api.ts` (Added `delete` and `put` HTTP methods)
- `Frontend/src/app/(authenticated)/watchlist/page.tsx` (Complete integration with TanStack queries)
- `Frontend/src/app/(authenticated)/trade/page.tsx` (Complete integration with TanStack queries)
- `Frontend/src/components/navigation/top-nav.tsx` (Integrated `useProfileQuery`)

## 4. Files Deleted
None.

## 5. API Changes
Added the following REST endpoints to the authenticated router group:
- `GET /api/v1/watchlists/:id`
- `DELETE /api/v1/watchlists/:id`
- `DELETE /api/v1/watchlists/:id/items/:itemId`
- `GET /api/v1/strategies/:id`
- `DELETE /api/v1/strategies/:id`
- `DELETE /api/v1/strategies/:id/legs/:legId`

Additionally, unified the error responses. Handlers now map internal `apperrors` to correct HTTP statuses (`400`, `401`, `403`, `404`, `409`) via the newly improved `response.HandleError` method, instead of globally returning `500`.

## 6. Database Changes
None. The existing migrations (`000001` through `000005`) correctly define all schemas and constraints. No GORM `AutoMigrate` usage was introduced.

## 7. Frontend Changes
- Rewrote the `/watchlist` UI to consume `useWatchlistsQuery` and related mutations. It displays correct Empty states, supports creating the default watchlist, and adding/removing symbols.
- Rewrote the `/trade` Strategy Builder tab to consume `useStrategiesQuery` and related mutations. It supports selecting a strategy from a dropdown, creating new strategies, adding mock legs (since option chain is a future phase), and removing legs.
- `top-nav.tsx` now uses `useProfileQuery` to display the authenticated user's name alongside their email.

## 8. Authorization Verification
All handlers rigidly enforce data ownership by extracting the `user_id` from the JWT Context:
```go
userID := c.GetUint("user_id")
if userID == 0 { ... }
```
This ID is explicitly passed to the service layer, where operations like `GetByID`, `RemoveItem`, and `Delete` compare the database record's `UserID` against the request's `UserID`. Any discrepancy returns a `Forbidden` or `NotFound` error, preventing cross-user data leakage.

## 9. Test Results
- **Backend Build (`go build ./...`)**: PASS
- **Backend Vet (`go vet ./...`)**: PASS
- **Backend Tests (`go test ./...`)**: ENVIRONMENT BLOCKED (Local PostgreSQL instance unreachable on port 5432). Domain logic tests passed successfully (cached).
- **Frontend Lint (`npm run lint`)**: PASS
- **Frontend Build (`npm run build`)**: PASS

## 10. Build Results
Both backend (`go build`) and frontend (`npm run build`) compiled successfully with no compilation errors. The TypeScript validations passed after addressing the `useMutation` return signature.

## 11. Traceability
- `TRACEABILITY SOURCE REQUIRES GOVERNANCE CONFIRMATION` -> `GET /user/profile` -> `useProfileQuery`
- `REQ-WATCHLIST-001` (No Phase-1 SPEC exists) -> `WatchlistService` -> `useWatchlistsQuery` + `/watchlist/page.tsx`
- `SPEC_011` -> `IMPL_013_v2.0` -> `StrategyService` -> `useStrategiesQuery` + `/trade/page.tsx`

## 12. Governance Compliance
- **DEC-ARCH-004C (Strict Types):** Followed correctly in TypeScript API hooks.
- **DEC-ARCH-004D (Error Handling):** Aligned backend HTTP responses to domain `apperrors`.
- No Upstox/Broker logic added.
- No Python/APScheduler introduced.
- No Kafka/NATS/S3 used.
- Canonical routes preserved perfectly.

## 13. Deviations
- **API Error Handling:** The governance plan did not explicitly detail how to map the existing `apperrors` structure in Gin handlers. A custom lightweight mapping approach (`pkg/response/HandleError`) was introduced to avoid massive boilerplate and strictly conform to REST guidelines (returning 4xx instead of 5xx).
- **Mock Strategy Legs:** Because the option chain UI is explicitly deferred to later phases, adding strategy legs from the UI is simulated with fixed NIFTY leg presets to allow testing the CRUD flow.

## 14. Remaining Gaps
- **Database Integration Tests:** Must be rerun when the PostgreSQL environment is spun up.
- **Greeks / Option Chain / Payoff Charts:** Explicitly deferred as TRACEABILITY GAPs.
- **Broker Execution:** Not implemented, awaits Broker Phase.

## 15. Final Verdict

**P4-03 STATUS: COMPLETE WITH DEVIATIONS**
*(Deviations relate strictly to mock data insertions on the UI side to bypass missing components, and environment-blocked integration tests)*
