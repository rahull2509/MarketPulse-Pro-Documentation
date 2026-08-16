# P4-08 ANGEL ONE PROVIDER DECISION

**Status:** [APPROVED]
**Date:** 2026-08-10

## 1. Authentication Mechanism
Angel One SmartAPI does **not** use a standard OAuth 2.0 redirect flow. It utilizes a direct credential-based POST login (Publisher Login) requiring a Time-based One-Time Password (TOTP).

## 2. Connect / Login Flow
- **Base URL:** `https://apiconnect.angelone.in`
- **Endpoint:** `POST /rest/auth/angelbroking/user/v1/loginByPassword`

## 3. Request Body (Exact JSON)
```json
{
  "clientcode": "<client code>",
  "password": "<PIN/password as governed>",
  "totp": "<current TOTP>",
  "state": "<optional governed state value>"
}
```
*The exact field names (`clientcode`, `password`, `totp`, `state`) MUST be preserved.*

## 4. Required Headers (Mandatory)
- `Content-Type: application/json`
- `Accept: application/json`
- `X-UserType: USER`
- `X-SourceID: WEB`
- `X-ClientLocalIP: <resolved client local IP>`
- `X-ClientPublicIP: <resolved public IP>`
- `X-MACAddress: <resolved MAC address>`
- `X-PrivateKey: <Angel One API key>`

## 5. IP/MAC Resolution Policy
**Resolution:** Explicit Application Configuration.
- `X-ClientPublicIP` MUST be sourced from explicit configuration `ANGELONE_CLIENT_PUBLIC_IP`.
- `X-ClientLocalIP` MUST be sourced from explicit configuration `ANGELONE_CLIENT_LOCAL_IP`.
- `X-MACAddress` MUST be sourced from explicit configuration `ANGELONE_CLIENT_MAC_ADDRESS`.

**Security Rules:**
- The backend MUST NOT trust or use `X-Forwarded-For` or browser IP metadata for these headers.
- Dynamic network-interface discovery MUST NOT be used.
- These values MUST be validated at application startup.
- If missing or invalid, the authentication MUST fail safely.
- Configuration values MUST NOT be logged.
- These values remain provider-specific to Angel One.

## 6. Login Response Contract
```json
{
  "status": true,
  "message": "SUCCESS",
  "errorcode": "",
  "data": {
    "jwtToken": "...",
    "refreshToken": "...",
    "feedToken": "...",
    "state": "..."
  }
}
```
*The implementation must validate `status`, validate required token fields, never log plaintext tokens, encrypt tokens using `CryptoService`, persist via `BrokerSession`, and preserve `encryption_version`.*

## 7. Credential Security
- `clientcode`, `password`, `totp`, `X-PrivateKey`, `jwtToken`, `refreshToken`, and `feedToken` MUST NOT be logged.
- Credentials must remain in memory only for the minimum required duration.
- Undocumented 4xx errors map to `UNKNOWN_PROVIDER_BUSINESS_ERROR` (No retry, no CB increment).

## 8. Authentication Boundary
Angel One direct authentication must occur via `DirectLogin -> BrokerSession`. It MUST NOT overload the OAuth `Callback` method.
