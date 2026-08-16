# P4-07 FINAL FORENSIC AUDIT REPORT

**Date:** 2026-08-10
**Audit Status:** FAILED
**Implementation Status:** PARTIAL / BLOCKED

## Executive Summary
An independent, strictly read-only forensic audit of the P4-07 codebase has been conducted following the Correction Implementation phase.

**Verdict:** The codebase has successfully remediated many structural defects (implementing a functional Circuit Breaker, removing symbol-guessing, and correctly isolating the Zerodha adapter). However, the implementation **STILL CONTAINS PRODUCTION MOCKS** in the REST API handler, violating strict governance rules. Furthermore, Angel One and ICICI Direct correctly halted implementation due to unverified documentation, meaning the overall implementation is only Functionally Partial.

---

## 1. Governance Verification

| Governance Decision | Status | Forensic Evidence |
|---------------------|--------|-------------------|
| `P4-07-BROKER-AUTH-DECISION.md` | FAILED | `auth_service.go` correctly isolates Zerodha token exchange using `sha256` checksums. However, `broker_handler.go` hardcodes `userID := int64(1)` during the callback, violating OAuth state protection and user isolation. |
| `P4-07-DATA-MODEL-DECISION.md` | PASSED | `BrokerSession`, `BrokerOrder`, etc. correctly utilize `BIGINT` user_ids mapped to core users, with correct foreign keys and `ON DELETE CASCADE`. |
| `P4-07-INSTRUMENT-MAPPING-DECISION.md` | PASSED | `InstrumentRepository` strictly maps `InternalSymbol` to `BrokerSymbol`/`BrokerToken`. Missing mappings return `ErrUnsupportedInstrument` without falling back. |
| `P4-07-API-CONTRACT-DECISION.md` | PASSED | All REST handlers use governed HTTP semantics. Internal orchestrator uses explicit `ProviderOrderRequest` boundary model. |
| `P4-07-FAILURE-SEMANTICS-DECISION.md` | PASSED | `CircuitBreaker` correctly implements >50% 5xx failure threshold on a 1-minute window, with exact 30-second OPEN states and 1-probe HALF-OPEN logic. Order retries are explicitly banned. |
| `P4-07-SECURITY-DECISION.md` | PASSED | `CryptoService` uses `AES-256-GCM` with Nonce uniqueness. Tokens are stored encrypted. |

## 2. Database Forensic Audit
**Status: PASSED**
- Verified `migrations/000006_create_broker_schema.up.sql`.
- Correctly uses `BIGINT` for user_id.
- Correctly enforces `REFERENCES users(id) ON DELETE CASCADE`.
- Includes `encryption_version`.

## 3. Cryptography Audit
**Status: PASSED**
- `CryptoService` strictly implements `AES-256-GCM`.
- Loads exactly 32-byte key from `BROKER_ENCRYPTION_KEY`.
- Uses unique nonces via `rand.Reader`.
- No plaintexts are persisted or logged.

## 4. Broker Authentication Audit
**Status: FAILED**
- **Zerodha:** The `auth_service.go` correctly implements SHA-256 exchange.
- **Handler Violation:** The HTTP Handler `ProviderCallback` hardcodes the user session: `userID := int64(1) // For this mock compile...`. This violates user isolation and OAuth state protection.
- **Angel One & ICICI Direct:** Safely return `ErrUnsupportedCapability`.

## 5. Provider HTTP Implementation Audit
**Status: PARTIAL**
- **Zerodha:** Functional `PlaceOrder`, `ModifyOrder`, and `CancelOrder` with real `net/http` calls, body serialization, and error parsing. However, `GetPositions`, `GetOrders`, and `GetHoldings` hit the correct endpoints but return empty models (`[]models.BrokerPosition{}`) without parsing the response body, making them partially implemented.
- **Angel One & ICICI Direct:** Halt safely with `ErrUnsupportedCapability`.

## 6. Instrument Resolution Audit
**Status: PASSED**
- Traced `PlaceOrder` -> `manager.go` -> `GetBrokerMapping`.
- Fails securely if mapping is missing. No symbol guessing exists.
- Broker tokens and symbols are explicitly passed to adapters via `ProviderOrderRequest`.

## 7. Circuit Breaker Forensic Audit
**Status: PASSED**
- Evaluates `count5xx / totalRequests > 0.5`.
- 1-minute rolling window implemented via `now.Add(-1 * time.Minute)`.
- OPEN state enforced for exactly `30*time.Second`.
- HALF-OPEN permits exactly `1` probe.

## 8. Order Safety Audit
**Status: PASSED**
- `PlaceOrder`, `ModifyOrder`, and `CancelOrder` do not contain internal retry loops. 
- Passed through `cb.Execute` exactly once.

## 9. Read Retry Audit
**Status: PASSED**
- No automatic read retries were implemented, meaning no forbidden retries exist.

## 10. REST API Audit
**Status: FAILED**
- Exact governed routes (`GET /api/v1/broker/connect/:provider` and `/callback/:provider`) are registered.
- However, the callback route handler contains a production mock (`userID := int64(1)`).

## 11. Redis / Realtime Audit
**Status: PASSED**
- Publishes strictly to `mp:events:orders`.
- No sensitive credentials, access tokens, or refresh tokens are pushed to Redis.

## 12. FX / Dependency Injection Audit
**Status: PASSED**
- Constructors for `InstrumentRepository` and `CircuitBreaker` correctly provided to the `BrokerManager` in `module.go`.

## 13. Test Audit
**Status: PASSED**
- `adapter_test.go` uses `httptest.Server` to mock Zerodha APIs.
- `circuit_breaker_test.go` verifies the transition states of the Circuit Breaker.

## 14. Production Mock / Stub Forensic Search
**Status: FAILED**
A forensic search for production mocks identified the following violations:
- `internal/modules/broker/handlers/broker_handler.go` contains:
  ```go
  // Ensure this matches the callback domain state validation to get the exact user.
  // For this mock compile, we'll just parse an ID from state or mock it.
  userID := int64(1)
  ```

## 15. P4-04 / P4-05 / P4-06 Regression Audit
**Status: PASSED**
- No broker code was found to overwrite or mutate P4-04 Ingestion, P4-05 Publication, or P4-06 Greeks.

## 16. Report Truth Audit
- Claim: "All tests use gomock or httptest.Server. No production mocks or hardcoded behavior exist in application logic." -> **FALSE** (Production mock exists in `broker_handler.go`).

## 17. Provider Capability Matrix

| Provider | Auth | Orders | Positions | Holdings | Instrument Mapping | Status |
|----------|------|--------|-----------|----------|--------------------|--------|
| Zerodha  | FUNCTIONAL | FUNCTIONAL | PARTIAL | PARTIAL | FUNCTIONAL | FUNCTIONALLY PARTIAL |
| Angel One| UNSUPPORTED| UNSUPPORTED| UNSUPPORTED| UNSUPPORTED| UNSUPPORTED| UNSUPPORTED |
| ICICI Direct| UNSUPPORTED| UNSUPPORTED| UNSUPPORTED| UNSUPPORTED| UNSUPPORTED| UNSUPPORTED |

## 18. Final Decision

**FINAL VERDICT: FAILED**
P4-07 = FUNCTIONALLY PARTIAL AND GOVERNANCE BLOCKED

**Rationale:** The correction phase successfully established robust architectural boundaries (Circuit Breaker, Instrument Resolution). However, the implementation is explicitly blocked from closing because a production REST handler contains a mocked `userID := int64(1)`, violating user isolation rules and the strict "No Mocks" governance mandate. Furthermore, `GetPositions` and `GetHoldings` for Zerodha only execute HTTP requests without parsing the data.

## 19. REQUIRED CORRECTIONS

1. **Remove the Production Mock:** `BrokerHandler.ProviderCallback` MUST correctly resolve the `userID` from the OAuth `state` parameter or a verified JWT context. `userID := int64(1)` must be removed.
2. **Complete Data Parsing:** Zerodha's `GetPositions`, `GetOrders`, and `GetHoldings` must actually parse their JSON responses into models, or be explicitly stubbed with `ErrUnsupportedCapability` if implementation is deferred.

No code modifications were made during this audit.
