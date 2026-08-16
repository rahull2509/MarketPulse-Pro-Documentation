# P4-07 Final Pre-Implementation Gate

## 1. Governance Verification
The execution plan aligns structurally with the updated P4-07 governance artifacts. It successfully isolates provider protocols, correctly assigns realtime ownership to the BrokerManager, and restricts retries to idempotent read operations.

## 2. Repository Verification (Constructor Existence)
- **Finding**: The execution plan explicitly lists `brokerrepo.NewPostgresBrokerSessionRepository`, `zerodha.NewAdapter`, `broker.NewAuthService`, etc. A directory listing of `Backend/internal/modules/broker` confirms that these packages and constructors **do not currently exist** in the repository.
- **Verification**: The plan is structurally correct in intending to use Fx to wire these components. However, the plan must be understood as *creating* these constructors during Phases 5-7 prior to Phase 12 (Fx wiring). No existing constructors can be leveraged for this.

## 3. Provider Verification
- **Zerodha**: Plan correctly identifies request_token/access_token lifecycle.
- **Angel One**: Plan correctly acknowledges publisher login/TOTP and avoids forcing an OAuth 2.0 `ExchangeToken` pattern.
- **ICICI Direct**: Plan correctly identifies SessionToken and checksum requirements.

## 4. Database Verification
- **Migration**: 000006 is confirmed as the next available migration sequence.
- **Fields**: `user_id BIGINT REFERENCES users(id) ON DELETE CASCADE` and `encryption_version INTEGER NOT NULL` are correctly specified. 
- **Verification**: The plan delegates the exact field definitions to `P4-07-DATA-MODEL-DECISION.md`. During implementation, all VARCHAR, DECIMAL, and TIMESTAMP fields must strictly match the governed document.

## 5. Security Verification
- **Encryption**: AES-256-GCM token encryption, tracked by `encryption_version`, is correctly mandated.
- **Key**: Plan references the environment-based master key mechanism, which is approved in the security decision.

## 6. REST Verification
- **Finding / Correction Required**: The execution plan specifies registering the endpoints `/api/v1/broker/connect/{provider}` and `/callback/{provider}`, but **omits the HTTP methods**. 
- **Required Correction**: The plan must strictly enforce `GET /api/v1/broker/connect/{provider}` and `GET /api/v1/broker/callback/{provider}` as defined in `P4-07-API-CONTRACT-DECISION.md`.

## 7. Realtime Verification
- **Ownership**: `BrokerManager` correctly owns event publication.
- **Payloads**: The plan correctly references the JSON payloads and `mp:events:orders` channel.
- **Isolation**: P4-04 and P4-05 events remain untouched.

## 8. Failure/Resilience Verification
- **Retry**: The plan explicitly prohibits retrying `PlaceOrder` on 5xx/timeouts, guaranteeing protection against duplicate execution ambiguity.
- **Circuit Breaker**: The plan explicitly lists the exact governed threshold (>50% over 1 min).

## 9. Fx Verification
- **Dependency Graph**: The execution plan successfully separates `BrokerAuthService` from `BrokerManager`, avoiding circular dependencies. Repositories and adapters are correctly layered.
- **Correction Required**: As noted in Section 2, the exact constructor names (`zerodha.NewAdapter`, etc.) must be dynamically created to match this Fx wiring step.

## 10. Testing Verification
- The plan explicitly mandates `gomock` over real credentials and provides a comprehensive list of 17 test domains covering error normalization, state transitions, and rate limiting.

## 11. Exact Remaining Discrepancies
1. **HTTP Methods Missing**: The execution plan must explicitly define the REST API methods (`GET`, `POST`, `PUT`, `DELETE`) alongside the routes to ensure no HTTP method guessing occurs during implementation.
2. **Constructor Creation**: The plan assumes constructor names for Fx injection. The implementation must guarantee these exact packages and signatures are created before wiring.

## 12. Final Verdict
**P4-07 PRE-IMPLEMENTATION GATE = APPROVED WITH REQUIRED CORRECTIONS**

Implementation is authorized to begin, provided the exact HTTP methods are strictly followed from the API Contract Decision during the routing implementation, and the missing constructor packages are explicitly created before Fx injection.
