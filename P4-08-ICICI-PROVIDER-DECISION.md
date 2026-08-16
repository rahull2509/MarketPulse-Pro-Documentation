# P4-08 ICICI DIRECT PROVIDER DECISION (PROPOSAL)

**Status:** [PENDING PROJECT OWNER APPROVAL]
**Date:** 2026-08-10

## 1. Authentication Mechanism
ICICI Direct Breeze API uses a redirect-based Publisher Login flow to obtain a Session Token. It is **not** standard OAuth 2.0.

## 2. Connect / Login Flow
- **Base API URL:** `https://api.icicidirect.com/breezeapi/api/v1`
- **Login URL:** `https://api.icicidirect.com/apiuser/login?api_key=YOUR_API_KEY`
- **Flow:** The user is redirected to the ICICI login page. Upon successful authentication, ICICI redirects back to the configured callback URL, providing an `API_Session` (Session Token) typically in the response.

## 3. Required Credentials
- `AppKey` (API Key)
- `Secret Key`
- `Session Token` (API_Session, expires in 24 hours)

## 4. Request Headers (Mandatory)
Every API request must include the following headers for authentication and security:
- `Content-Type: application/json`
- `X-AppKey: <AppKey>`
- `X-SessionToken: <SessionToken>`
- `X-Timestamp: <Current_ISO8601_Timestamp>`
- `X-Checksum: token <SHA256_Hash>`

## 5. Checksum / Signature Requirements
All requests MUST be cryptographically signed using the Secret Key.
- **Algorithm:** `SHA256`
- **Payload Input:** `Timestamp + JSON_Body + SecretKey`
- **Format:** The resulting hash must be prefixed with the string `"token "` and placed in the `X-Checksum` header.

## 6. Token / Session Lifecycle
- The `API_Session` is valid for 24 hours.
- There is no automated refresh token flow; the user must re-authenticate via the UI redirect once the session expires.

## 7. Order / Position / Holding APIs
- **Order Placement:** `POST /order`
- **Modify Order:** `PUT /order`
- **Cancel Order:** `DELETE /order`
- **Order Book:** `GET /order`
- **Positions:** `GET /portfolio/positions`
- **Holdings:** `GET /portfolio/holdings`
- **Response Envelope:** Usually `{ "Success": [...], "Status": 200, "Error": ... }`

## 8. SDK vs HTTP
**Recommendation:** Direct HTTP (`net/http`). 
- Given the strict P4-07 requirements for custom CircuitBreaker wrapping, direct control over the HTTP context, and explicit ban on automatic HTTP retries (especially for write operations), using the official Breeze SDK is not recommended. Implementing the checksum generation locally is trivial and ensures full architectural compliance.

## 9. Security Requirements
- **Storage:** `API_Session` (Session Token) must be encrypted at rest via `CryptoService` (AES-256-GCM).
- **Redaction:** Tokens, AppKeys, and Secret Keys must never be logged.
- **CSRF Protection:** If ICICI provides a mechanism for passing an opaque state parameter through their login flow, it MUST be used with the P4-07 OAuth State Redis validator. If ICICI drops the state parameter, we must document this limitation and securely bind the session token strictly to the active JWT session upon receipt.

## 10. Unsupported Capabilities
- Features outside the P4-07 `BrokerAdapter` interface are unsupported.

*This document is a governance proposal and must be approved before implementation begins.*
