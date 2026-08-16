# P4-02: Application Authentication & User/Session Integration

## 1. Executive Summary
This document outlines the implementation plan for P4-02, which transitions the existing backend authentication foundation and UI application shell into a fully integrated, session-aware authentication flow. It bridges the gap between the Next.js frontend (Zustand/Cookies) and the Go/Gin backend (JWT/Bearer), ensuring secure route protection, correct logout semantics, and a seamless user experience.

## 2. Current Repository Auth State
Based on forensic inspection of the repository:
- **Backend:** A robust foundation exists. `users` table is defined. `auth.Handler` supports `POST /auth/register` and `POST /auth/login`, returning a 24h JWT and user payload. `user.Handler` supports `GET /user/profile`.
- **Frontend:** `auth-store.ts` (Zustand) persists auth state to `localStorage`. `login/page.tsx` manually sets a plain `auth_token` cookie to satisfy `middleware.ts`. `register/page.tsx` forces a manual login step after registration.
- **Critical Disconnect:** The frontend `logout` function clears Zustand but fails to clear the `auth_token` cookie, leaving the user effectively "logged in" according to the middleware. `lib/api.ts` does not intercept `401 Unauthorized` responses to clear expired sessions.

## 3. Evidence Inventory
- `Backend/migrations/000001_create_users_table.up.sql`
- `Backend/internal/core/auth/jwt.go`
- `Backend/internal/modules/auth/handler.go`
- `Backend/internal/routes/routes.go`
- `Frontend/src/middleware.ts`
- `Frontend/src/stores/auth-store.ts`
- `Frontend/src/lib/api.ts`
- `Frontend/src/app/auth/login/page.tsx`
- `Frontend/src/app/auth/register/page.tsx`
- `Frontend/src/components/navigation/top-nav.tsx`

## 4. Architecture Compliance Review
The proposed changes strictly adhere to the established architecture:
- No new state management libraries (using existing Zustand).
- No new database tables or GORM `AutoMigrate` (existing schema is sufficient).
- Broker authentication (`DEC-ARCH-004`) remains strictly segregated in the TopNav BrokerModal and is explicitly excluded from this application auth flow.

## 5. Authentication Flow
- **Register:** Submit credentials -> Backend creates user & returns JWT -> Frontend stores JWT in Zustand & sets Cookie -> Auto-login & redirect to `/dashboard`.
- **Login:** Submit credentials -> Backend validates & returns JWT -> Frontend stores JWT & sets Cookie -> Redirect to `/dashboard`.
- **Session:** `lib/api.ts` injects `Authorization: Bearer <token>`. If a `401 Unauthorized` is returned, the API client triggers a global session clear.
- **Logout:** User clicks Logout -> Frontend clears Zustand state -> Frontend explicitly deletes `auth_token` cookie -> Calls backend `POST /api/v1/auth/logout` -> Redirects to `/auth/login`.

## 6. Session/JWT Architecture
The backend uses a stateless JWT with a 24-hour expiration.
- **Transport:** JWT is returned in the JSON payload, stored in `localStorage` by Zustand, and copied to a client-side cookie (`auth_token`) for Next.js middleware routing. API requests use the `Authorization: Bearer <token>` header.
- **Correction:** The frontend must align the cookie expiration with the JWT expiration (24h) and strictly delete the cookie on logout.

## 7. Database/User Model Review
The `users` table (`000001_create_users_table.up.sql`) is fully compliant:
- Primary Key: `id` (BIGSERIAL)
- Uniqueness: `email` (UNIQUE)
- Security: `password` (VARCHAR)
- Timestamps: `created_at`, `updated_at`
**No database migrations are required for P4-02.**

## 8. API Contract Matrix
| Method | Route | Auth Req | Changes |
|---|---|---|---|
| POST | `/api/v1/auth/register` | No | None (Existing) |
| POST | `/api/v1/auth/login` | No | None (Existing) |
| GET | `/api/v1/user/profile` | Yes | None (Existing) |
| POST | `/api/v1/auth/logout` | Yes | **[NEW]** Returns 200 OK. Serves as a placeholder for future token invalidation logic. |

## 9. Frontend Integration Architecture
- **`lib/api.ts`:** Will be enhanced to detect `401` status codes. Upon detection, it will invoke `useAuthStore.getState().logout()` and force a window reload or redirect to `/auth/login`.
- **`auth-store.ts`:** The `logout` action will be upgraded to explicitly delete the `auth_token` cookie (`document.cookie = "auth_token=; path=/; max-age=0"`).
- **`register/page.tsx`:** Will consume the token returned by the backend and log the user in immediately, eliminating the redundant manual login step.

## 10. Route Protection Model
The canonical routes remain protected by `middleware.ts`. The logic (`isProtectedPage && !token`) is sound, provided the cookie lifecycle is correctly managed by the auth store.

## 11. Security Review
- **Password Hashing:** Handled securely via `bcrypt` in `userService.Register`.
- **XSS Exposure:** Storing JWTs in `localStorage` and non-HttpOnly cookies exposes them to XSS. This is a known architectural tradeoff for the current phase to allow Next.js middleware to read the token without requiring a BFF (Backend For Frontend). *[Smallest Compliant Correction: We will maintain this transport but ensure strict 24h expiration on the cookie].*
- **Logging:** Passwords and tokens are strictly excluded from `zap.Logger`.

## 12. Error Handling
The backend uses standard HTTP status codes (400, 401). `lib/api.ts` translates these into user-friendly `apiError` strings which are displayed natively in the Login/Register forms.

## 13. Observability
Backend handlers will log authentication lifecycle events:
- Registration success/failure.
- Login success/failure.
- Logout event.

## 14. Testing Strategy
- **Backend:** Add unit tests for `POST /auth/logout` and ensure existing `register/login` tests pass.
- **Frontend Integration:** Ensure manual verification of:
  - Register auto-login.
  - Logout clearing cookies and redirecting to login.
  - Expired token triggering a 401 and redirecting to login.

## 15. Exact File Change Inventory

### Backend
1. **`Backend/internal/modules/auth/handler.go`**
   - **MODIFY:** Add `Logout` method returning `200 OK`.
2. **`Backend/internal/routes/routes.go`**
   - **MODIFY:** Register `POST /auth/logout` under the protected route group.

### Frontend
3. **`Frontend/src/stores/auth-store.ts`**
   - **MODIFY:** Update `logout` function to delete `document.cookie = 'auth_token=; path=/; max-age=0'`. Update `setAuth` to set the cookie with a 24h `max-age`.
4. **`Frontend/src/lib/api.ts`**
   - **MODIFY:** Intercept `401` responses. If `401` is received, call `useAuthStore.getState().logout()` and redirect.
5. **`Frontend/src/app/auth/register/page.tsx`**
   - **MODIFY:** Capture the `token` and `user` payload from the successful API response, invoke `setAuth`, and redirect to `/dashboard` directly.
6. **`Frontend/src/app/auth/login/page.tsx`**
   - **MODIFY:** Remove manual cookie setting (now handled by `setAuth` in the store).
7. **`Frontend/src/components/navigation/top-nav.tsx`**
   - **MODIFY:** Ensure the `handleLogout` function invokes the backend `/api/v1/auth/logout` endpoint in addition to clearing the local store.

## 16. Dependency Changes
None.

## 17. Migration Requirements
None.

## 18. Implementation Sequence
1. Modify `auth-store.ts` to manage the cookie lifecycle centrally.
2. Update `login/page.tsx` and `register/page.tsx` to use the updated store.
3. Enhance `lib/api.ts` to intercept `401` errors.
4. Add the `Logout` endpoint to the backend (`handler.go` & `routes.go`).
5. Wire the `TopNav` logout button to the new backend endpoint and frontend store.
6. Run `npm run lint`, `npm run build`, and `go test ./...`.

## 19. Risks
- Edge cases in `lib/api.ts` triggering infinite redirect loops if `401` handling is not debounced or isolated from the `/auth/login` route itself. (Mitigation: Ensure `401` interceptor ignores requests on the login page).

## 20. Traceability Matrix
- Matches `SPEC_004` (Application Authentication Flow).

## 21. Pending Decisions / Blockers
None.

## 22. Verification Plan
- Register a new user -> Confirm immediate redirect to `/dashboard`.
- Logout -> Confirm redirect to `/auth/login` and inability to navigate back to `/dashboard`.
- Simulate `401` by manually deleting token from `localStorage` -> Confirm automatic logout on next API call.

## 23. Expected Final State
A cohesive, robust application authentication system where frontend and backend lifecycles are perfectly synchronized.

---

P4-02 PLAN STATUS: READY FOR IMPLEMENTATION
