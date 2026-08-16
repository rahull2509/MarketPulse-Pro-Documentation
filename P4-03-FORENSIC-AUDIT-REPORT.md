# P4-03 Forensic Audit Report

## 1. Executive Summary
This document provides the final forensic audit of the MarketPulse Pro P4-03 (Core Domain & Data Integration) implementation. The audit inspects the actual repository code against the documented enterprise governance, design constraints, and traceability requirements.

**Primary Finding:** The backend domain implementation (Watchlist and Strategy CRUD) is architecturally sound and correctly enforces strict user ownership boundaries. However, the implementation introduces a **TRACEABILITY ERROR** by citing incorrect specifications, and a **PRODUCT LOGIC CONTAMINATION** by allowing the frontend UI to persist hardcoded "mock" strategy legs into the production database under the guise of an actual feature.

## 2. Repository Evidence
- **Backend:** `golang-migrate` schemas remain untouched and authoritative. `WatchlistService` and `StrategyService` correctly extract the `userID` context and compare it against the target database record.
- **Frontend:** TanStack query hooks (`use-watchlists.ts`, `use-strategies.ts`) replace the previous mock implementations. The `trade/page.tsx` UI was updated to fire live mutations to the backend.

## 3. Traceability Audit
The P4-03 Implementation Plan and Report claimed traceability mappings that do not exist in the official `FEATURE_TRACEABILITY.md` or `Volume_2_Requirements.md`.

| Requirement | Claimed SPEC | Actual SPEC Definition | Actual Implementation | Verdict |
|---|---|---|---|---|
| User Profile | `SPEC_005` | Parquet / Option Chain | `useProfileQuery` | **TRACEABILITY ERROR** |
| Watchlist | `SPEC_006` | Upstox Integration | `WatchlistService` | **TRACEABILITY ERROR** |
| Strategy | `SPEC_007` | Redis Pub/Sub Bridge | `StrategyService` | **TRACEABILITY ERROR** |

*Note: The actual valid Strategy specification is `SPEC_011` → `IMPL_013_v2.0` (Strategy Builder). Watchlist is a core business requirement (`REQ-WATCHLIST-001`) but does not have a dedicated `Phase-1/SPEC_xxx.md` file.*

## 4. Watchlist Audit
- **Model / Migration:** Correctly defined via `000004_create_watchlists_table.up.sql`. No `AutoMigrate` usage.
- **Repository / Service:** Implemented `ListByUser`, `Create`, `GetByID`, `Delete`, `AddItem`, `RemoveItem`.
- **Frontend Flow:** `useWatchlistsQuery` integrates seamlessly into `/watchlist/page.tsx`.
- **Verdict:** **SAFE**. Conceptually, User A attempting to access User B's watchlist is rejected natively at the service layer by checking `watchlist.UserID != userID`.

## 5. Strategy Audit
- **Model / Migration:** Correctly defined via `000005_create_strategies_table.up.sql`.
- **Frontend Flow:** The `trade/page.tsx` page correctly loads and creates Strategies via `useStrategiesQuery`.
- **Mock Data Generation:** The UI includes `+ Buy CE` and `+ Sell PE` buttons. These buttons execute `handleAddMockLeg`, which submits a hardcoded NIFTY strike (22000) to the **live production API** (`POST /api/v1/strategies/:id/legs`).
- **Verdict:** **PRODUCT LOGIC CONTAMINATION**. While intended as a "development placeholder" to bypass the unbuilt Option Chain, it presents fake data generation as a primary user action and permanently contaminates the actual database with mock records. It is not sandboxed.

## 6. User/Profile Audit
- **Implementation:** `GET /api/v1/user/profile` correctly extracts `user_id` from the JWT and returns the User model. Password hashes are correctly omitted from the response struct.
- **Integration:** The `TopNav` component consumes `useProfileQuery` to display the user's name.
- **Verdict:** **SAFE**. No sensitive fields leaked.

## 7. Authorization Audit
- **Mechanism:** JWT middleware extracts `user_id` and sets it in the Gin context. Handlers extract this integer and pass it explicitly to the domain services.
- **Enforcement:** `GetByID(ctx, id, userID uint)` enforces `if entity.UserID != userID { return Forbidden }`. All mutations (Delete, AddLeg, RemoveItem) depend on `GetByID`, ensuring IDOR vulnerabilities are mitigated system-wide. The backend explicitly ignores any `user_id` passed in the request body.
- **Verdict:** **SAFE**. Security constraints correctly enforced.

## 8. Error Handling Audit
- **Implementation:** `pkg/response/HandleError` was introduced to map domain `apperrors` to standard HTTP statuses (400, 401, 403, 404, 409).
- **Leakage:** The fallback branch for non-domain errors executes `c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})`. This violates the governance rule that internal errors must never leak to the client, as raw SQL or panic strings could be exposed.
- **Verdict:** **REQUIRED CORRECTION (MEDIUM)**.

## 9. Database Audit
- **Migrations:** `golang-migrate` is strictly used. `AutoMigrate` was not introduced.
- **Verdict:** **SAFE**.

## 10. Frontend Data Flow Audit
- **Architecture:** UI → Query Hook (`useStrategiesQuery`) → `api.ts` → API.
- **Storage:** No fake server state exists. No duplicate state stored in Zustand (Zustand strictly manages the JWT).
- **Verdict:** **SAFE**.

## 11. UI Audit
- **Behavior:** The P4 Master UI shell is perfectly preserved. Loading, Empty, and Error states handle gracefully. Canonical routes remain untouched.
- **Verdict:** **VERIFIED WITH DEVIATIONS**. As noted in Section 5, the UI presents mock market data generation as live user actions.

## 12. API Contract Audit
- **Contracts:** Frontend accurately hits `GET`, `POST`, and `DELETE` paths matching the updated Gin router paths. `lib/api.ts` was properly updated to include `delete` and `put`.
- **Verdict:** **SAFE**.

## 13. Testing Audit
- **Backend Build/Vet:** **PASS**.
- **Backend Tests:** **ENVIRONMENT BLOCKED**. The `TestRunMigrations` database test fails because a local PostgreSQL instance is unavailable on port 5432. (Domain tests passed via cache).
- **Frontend Lint/Build:** **PASS**.
- **Verdict:** **SAFE**, accurately reported.

## 14. Regression Audit
- **Impact:** JWT login, middleware, and the UI Shell were untouched.
- **Verdict:** **SAFE**.

## 15. Prohibited Technology Audit
- Search confirmed no usage of: Upstox, Python, APScheduler, Kafka, NATS, S3, Parquet, AutoMigrate, go-chi.
- **Verdict:** **SAFE**.

## 16. Governance Audit
- **DEC-ARCH-004D (Error Handling):** Attempted resolution via `response.HandleError`, but the implementation is flawed (leaks internal errors).
- **Verdict:** Governance remains largely respected, but Error Handling requires fixing.

## 17. Deviations
- **Traceability Hallucination:** P4-03 claimed `SPEC_005/006/007` mapped to Profile/Watchlist/Strategy. This is entirely false.
- **Mock Data Contamination:** The Trade UI sends fake NIFTY legs to the real database.

## 18. Risk Register
| Risk | Severity | Description |
|---|---|---|
| IDOR / Data Leakage | LOW | Service layer ownership checks (`UserID != request.UserID`) thoroughly mitigate IDOR. |
| DB Contamination | HIGH | Mock strategy legs populate production database tables under real user accounts. |
| Error Leakage | MEDIUM | `HandleError` fallback exposes raw `err.Error()` to clients on 500s. |
| Governance Drift | LOW | Hallucinated traceability mappings dilute the source of truth. |

## 19. Required Corrections
1. **Fix Error Leakage:** Update `pkg/response/response.go` to return a generic "Internal Server Error" string for 500s instead of `err.Error()`.
2. **Isolate Mock Data:** Remove the `+ Buy CE` / `+ Sell PE` buttons that mutate the live database with fake data. Replace them with a pure UI placeholder or an `alert("Option chain integration deferred to future phase")`.
3. **Correct Traceability:** Update the implementation documentation to accurately reflect `SPEC_011` for Strategy, and mark Watchlist as a direct Business Requirement (`REQ-WATCHLIST-001`) without a SPEC phase-1 document.

## 20. Final Verdict

**P4-03 FORENSIC AUDIT = P4-03 VERIFIED WITH REQUIRED CORRECTIONS**

## 21. P4-04 Readiness

**P4-04 READINESS = BLOCKED**
*Reason: The product logic contamination (persisting mock legs to the DB) and the error leakage (exposing raw 500 errors) must be fixed before the repository is cleared to advance to P4-04.*
