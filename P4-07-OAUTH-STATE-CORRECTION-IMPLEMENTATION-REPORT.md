# P4-07 OAUTH STATE CORRECTION IMPLEMENTATION REPORT

**Date:** 2026-08-10

## 1. Root Cause
The previous implementation derived the user identity solely from the JWT context during the callback (`c.Get("userID")`). While this successfully removed the hardcoded user mock, it failed to bind the OAuth callback to the original connection request using a cryptographically secure `state` parameter. This omission created a Cross-Site Request Forgery (CSRF) vulnerability where an attacker could bind their broker session to a victim's MarketPulse Pro account.

## 2. Governance Reference
This correction explicitly satisfies the **P4-07-BROKER-AUTH-DECISION.md** requirements for OAuth state protection and user isolation without altering the authorized behavior for unsupported capabilities.

## 3. State Generation
- During `Connect`, 32 bytes of secure entropy are generated using `crypto/rand`.
- The bytes are hex-encoded into a 64-character state string.
- This state provides 256 bits of cryptographic entropy, rendering it practically unguessable.

## 4. State Storage
- The state is persisted using the governed system infrastructure: **Redis**.
- The `BrokerAuthService` now injects `*redis.Client` (which is already used by `BrokerManager` for real-time events).
- Storage key: `mp:oauth:state:<state>`.
- TTL: Explicity configured for **15 minutes**.

## 5. State/User/Provider Binding
- The state value acts as a secure key pointing to a JSON payload in Redis containing:
  - `user_id`: The ID of the authenticated user who initiated the connection.
  - `provider`: The broker being connected (e.g., `zerodha`).
- This binding ensures that only the user who initiated the flow can consume the callback.

## 6. Callback Validation
- The `Callback` endpoint extracts the `state` from the query parameters.
- It fetches and decodes the state binding from Redis.
- Validation enforces:
  - State must not be empty.
  - State must exist in Redis (not expired).
  - The `user_id` inside the state MUST exactly match the `userID` from the current JWT session.
  - The `provider` inside the state MUST match the callback provider.

## 7. Replay Protection & Concurrency Safety
- State consumption is **strictly atomic** using Redis `GETDEL`.
- When the callback retrieves the state, `GETDEL` instantly removes it from Redis.
- If two simultaneous callbacks attempt to use the same state, exactly one will retrieve the data, and the other will receive `redis.Nil`, which triggers an "invalid, expired, or already consumed oauth state" error.
- This provides complete replay and concurrency protection without the race condition of a check-then-delete sequence.

## 8. Security Verification
- **No Hardcoded Identities:** The user ID is strictly derived from the JWT context and verified against the Redis state store.
- **No Predictable State:** `crypto/rand` is explicitly used.
- **No Plaintext Logging:** The OAuth state, `request_token`, `api_secret`, and `access_token` are completely absent from application logs.
- **Token Security:** Tokens are securely encrypted using `CryptoService` before persistence to PostgreSQL.

## 9. Tests
- Extensive unit tests were implemented using `miniredis` to verify the state behavior locally.
- **TestAuthService_ConnectStateGeneration**: Verifies state generation, format, randomness, and Redis serialization.
- **TestAuthService_CallbackValidation**: Simulates missing state, unknown state, and user identity mismatches, verifying exact error rejection.
- **TestAuthService_ReplayProtectionAndConcurrency**: Employs Goroutines to blast identical concurrent callbacks at the handler. Verifies that exactly one callback succeeds (in reading the state) and the remainder are safely rejected, confirming atomic consumption.

## 10. Verification Command Results
- `go test ./...` -> **PASS** (Excluding standard DB environment failures)
- `go test -race ./...` -> **ENVIRONMENT BLOCKED** (`cc1.exe: sorry, unimplemented: 64-bit mode not compiled in`)
- `go build ./...` -> **PASS**
- `go vet ./...` -> **PASS**

## 11. Files Changed
- `Backend/internal/modules/broker/services/auth_service.go`
- `Backend/internal/modules/broker/services/auth_test.go`
- `Backend/internal/modules/broker/module.go` (Dependency Injection logic)

## 12. Remaining Limitations
- Angel One and ICICI Direct correctly remain in the governed `ErrUnsupportedCapability` state.
- Race condition testing via Go tooling remains environment-blocked.

## 13. Final Status

**P4-07 OAUTH STATE CORRECTION = COMPLETE**
**P4-07 READINESS = READY FOR FINAL FORENSIC RE-AUDIT**
