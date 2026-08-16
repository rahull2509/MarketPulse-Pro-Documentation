# P4-07 Execution Plan Forensic Review

## 1. Executive Verdict
**P4-07 EXECUTION PLAN = APPROVED WITH REQUIRED CORRECTIONS**

The proposed execution plan correctly identifies the 15-phase methodology but contains critical deviations from the approved governance regarding API contracts, authentication normalization, and database schema requirements. These must be corrected before implementation begins.

## 2. Governance Compliance
The plan violates governance in three specific areas:
1. **False Protocol Normalization**: Assumes all providers can be forced into an `ExchangeToken` OAuth flow.
2. **API Contract Mismatch**: Proposes `/auth/:provider/login` instead of the governed `/connect/{provider}`.
3. **Database Security Gap**: Misses the mandated key versioning column required for secret rotation.

## 3. Provider Authentication Verification
- **Zerodha**: Supports standard OAuth 2.0. Requires `api_key`, redirects for a `request_token`, and exchanges for an `access_token` (valid for the day) via a checksum using `api_secret`.
- **Angel One**: Uses a `publisher-login` flow returning tokens in the URL, or a server-side `loginByPassword` requiring TOTP. It does NOT strictly support standard OAuth 2.0 `ExchangeToken`.
- **ICICI Direct**: Uses a login redirect to yield a `SessionToken` and requires a checksum signature on all subsequent requests.
- **Correction Required**: The `BrokerAuthService` orchestration boundary must normalize the *internal outcome* (`BrokerSession`), NOT the provider protocol. It must not force Angel One or ICICI into an `ExchangeToken` interface if they provide tokens directly via redirect or require TOTP. 

## 4. Database Migration Verification
- `000006_create_broker_schema.up.sql` is indeed the correct next migration number.
- `user_id` MUST be `BIGINT REFERENCES users(id) ON DELETE CASCADE`.
- **Correction Required**: `P4-07-SECURITY-DECISION.md` explicitly mandates secret rotation capabilities. The `broker_sessions` table MUST include an `encryption_version` (INTEGER) column to track which master key encrypted the token. 

## 5. Security Verification
- The plan to use AES-256-GCM is approved. 
- Using an environment variable (`BROKER_ENCRYPTION_KEY`) is explicitly approved in `P4-07-SECURITY-DECISION.md`.
- **Verification**: Keys must be 32 bytes, nonces uniquely generated per encryption and prepended/appended to ciphertext, and never logged. 

## 6. REST API Verification
- **Correction Required**: The proposed plan lists `/api/v1/broker/auth/:provider/login`. This violates `P4-07-API-CONTRACT-DECISION.md`, which explicitly mandates `/api/v1/broker/connect/{provider}`. All proposed routes must be updated to exactly match the governance artifact.

## 7. Redis/WebSocket Verification
- **Correction Required**: The plan suggests expanding Redis pub/sub inside the `BrokerAdapter`. This is a violation of abstraction. Event publication must be owned by the `BrokerManager` (Facade) to ensure event envelopes remain uniform regardless of which provider executed the order. 
- Events must strictly adhere to `P4-07-REALTIME-CONTRACT-DECISION.md` payloads (`order.execution`, `order.status`). 

## 8. Resilience Verification
- **Correction Required**: "Basic retry logic for 5xx errors" is dangerously broad. `PlaceOrder` MUST NOT be blindly retried on 5xx/timeouts, as it risks executing duplicate orders. Retries are only permitted for read operations (Positions, Holdings, Status).
- Circuit breaker must precisely implement the governed policy: Open circuit if >50% 5xx errors occur over a 1-minute window.

## 9. Instrument Mapping Verification
- `InstrumentResolver` must fetch provider instruments and cache them in `BrokerInstrumentMapping`. 
- **Verification**: The internal `P4-04` symbol remains the absolute authority. If a broker mapping is missing, the facade must safely return `ErrUnsupportedInstrument` rather than fabricating or guessing a provider token.

## 10. Fx Verification
- Generic binding statements are insufficient. The implementation must explicitly register:
  - `fx.Provide(brokerrepo.NewPostgresBrokerSessionRepository)`
  - `fx.Provide(brokerrepo.NewPostgresBrokerOrderRepository)`
  - `fx.Provide(zerodha.NewAdapter)`
  - `fx.Provide(angelone.NewAdapter)`
  - `fx.Provide(icicidirect.NewAdapter)`
  - `fx.Provide(broker.NewAuthService)`
  - `fx.Provide(broker.NewManager)` 
- Circular dependencies between Auth Service and Manager must be avoided by keeping them logically distinct.

## 11. Testing Verification
- "100% regression safety" is hyperbole. 
- **Correction Required**: The plan must replace this with a concrete, evidence-based testing strategy. Tests MUST use `gomock` to simulate Zerodha, Angel One, and ICICI responses. Tests must cover token encryption, order idempotency, mapping errors, and circuit breaker state transitions. No real credentials may be used.

## 12. Exact Required Plan Corrections
1. Do not enforce `ExchangeToken` on providers that do not support OAuth 2.0; standardize on returning a `*BrokerSession`.
2. Add `encryption_version` to the `broker_sessions` database migration.
3. Fix REST API routes to match `/connect/{provider}` and `/callback/{provider}`.
4. Move Redis event publication to the `BrokerManager` facade, not the `BrokerAdapter`.
5. Explicitly prohibit retries on `PlaceOrder`.
6. Use explicit `fx.Provide` constructors in the DI plan.

## 13. Final Readiness Decision
**P4-07 EXECUTION PLAN = APPROVED WITH REQUIRED CORRECTIONS**

Implementation may proceed, strictly adhering to the corrections listed in Section 12. 

Application code modified: NO
Frontend code modified: NO
Database migrations modified: NO
Routes modified: NO
REST APIs implemented: NO
Broker SDKs added: NO
Broker credentials added: NO
Redis events added: NO
WebSocket events added: NO
BrokerAdapter modified: NO
Database schema implemented: NO
Governance decisions modified: NO
