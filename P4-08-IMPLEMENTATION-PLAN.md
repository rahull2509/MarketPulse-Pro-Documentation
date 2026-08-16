# P4-08 IMPLEMENTATION PLAN

**Status:** [AUTHORIZED]
**Date:** 2026-08-10

## 1. Goal
Provide concrete API implementations for the Angel One and ICICI Direct brokers behind the P4-07 Broker Facade, strictly adhering to the explicitly approved governance specifications.

## 2. Governance Status
P4-08 Governance is **APPROVED**. 
All implementation-critical blockers have been explicitly resolved and governed by the Project Owner.

## 3. P4-07 Compatibility Analysis & Implementation Steps

### A. Angel One
**Authentication:**
- Implement a custom REST handler to accept `clientcode`, `password`, and `totp_secret` (or pre-generated `totp`).
- Securely invoke the Angel One login endpoint using the new `DirectLogin` method in `BrokerAuthService`.
- Apply exact JSON body and explicit required headers.
- Populate `X-ClientLocalIP`, `X-ClientPublicIP`, and `X-MACAddress` strictly from explicit configurations (`ANGELONE_CLIENT_LOCAL_IP`, `ANGELONE_CLIENT_PUBLIC_IP`, `ANGELONE_CLIENT_MAC_ADDRESS`).
- Validate configurations at application startup.
- Encrypt the resulting `jwtToken` using `CryptoService.Encrypt()`.
- Store the encrypted token in a new `BrokerSession` via `sessionRepo.Save()`.

**Orders & Positions:**
- Update `angelone/adapter.go`.
- Implement `PlaceOrder`, `ModifyOrder`, and `CancelOrder` by directly constructing `http.Request` objects, applying the `jwtToken` Bearer header, and wrapping the execution in the BrokerManager's `CircuitBreaker`.
- Implement Instrument mapping via the downloaded JSON Master file. Filter on `exchange`. Run at startup and 08:30 AM via Cron.
- Explicitly map 403 Rate Limit errors to `ErrRateLimitExceeded` (Do not trigger circuit breaker OPEN state). 10 orders/sec limit, 1 read/sec limit.
- Explicitly map undocumented 4xx responses to `UNKNOWN_PROVIDER_BUSINESS_ERROR` (No retry, No CB increment).
- Ensure NO internal retry loops exist for write operations.
- Map internal `ProviderOrderRequest` to the Angel One JSON format.

### B. ICICI Direct
**Authentication:**
- Implement the ICICI redirect callback endpoint to capture the `API_Session` token.
- Generate the `SHA256(timestamp + payload + secret_key)` checksum for every outgoing request.
- Encrypt the `API_Session` token via `CryptoService`.

**Orders & Positions:**
- Update `icicidirect/adapter.go`.
- Implement order management endpoints.
- Map internal `InternalSymbol` to the `stock_code` or `X.Y!Token` from the Security Master CSV. Run at startup and 08:30 AM via Cron.
- Ensure strict compliance with the CircuitBreaker and NO retry policies.
- Enforce the 100 req/min and 10 orders/sec rate limits explicitly, mapping to `ErrRateLimitExceeded`.
- Explicitly map undocumented 4xx responses to `UNKNOWN_PROVIDER_BUSINESS_ERROR` (No retry, No CB increment).

## 4. Security Review
- **Authentication Secrets:** TOTP Secrets and ICICI Secret Keys must be injected securely (e.g. env vars or secure vault) and NEVER passed plaintext via logs or unencrypted databases.
- **Token Encryption:** All tokens (JWT, Refresh, API_Session) must be encrypted via AES-256-GCM.
- **CSRF Protection:** If ICICI supports standard OAuth state strings, utilize the existing Redis state logic. Otherwise, implement an equivalent secure binding mechanism.
- **Redaction:** `X-PrivateKey`, `X-AppKey`, and `jwtToken` are strictly forbidden from appearing in any application log. Provider responses generating `UNKNOWN_PROVIDER_BUSINESS_ERROR` must be strictly redacted except where safe for diagnostics.

## 5. Verification Plan
- We will execute full integration tests to verify precise JSON mapping.
- We will verify that state protections (or equivalent secure protocols) are applied.
- We will execute the standard P4-07 CI command suite (`go test`, `go vet`, `go build`).
