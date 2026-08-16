# P4-08 ERROR SEMANTICS DECISION

**Status:** [APPROVED]
**Date:** 2026-08-10

## 1. Governance Principles
- The internal P4-07 `BrokerAdapter` error taxonomy is the authoritative standard.
- Provider-specific HTTP status codes and JSON error codes must be translated explicitly into standardized internal errors.
- Do not invent mappings; map only documented errors.
- **Write operations MUST remain non-retryable.**

## 2. Angel One Error Normalization Matrix
| Provider Condition | Internal Normalized Error | Retry Allowed? | CircuitBreaker Counted? |
|---|---|---|---|
| Invalid Quantity (`AB4014`) | `ErrInvalidQuantity` | NO | NO |
| Insufficient Funds ("Insufficient Funds") | `ErrInsufficientFunds` | NO | NO |
| Invalid Instrument (Missing Token) | `ErrUnsupportedInstrument` | NO | NO |
| Rejected Order (General `false` status) | `ErrOrderRejected` | NO | NO |
| Invalid TOTP (`AB1050`) | `ErrInvalidCredentials` | NO (Prompt UI) | NO |
| Invalid Token (`AG8001`) | `ErrSessionExpired` | YES (Auth Flow) | NO |
| Invalid API Key (`AG8004`) | `ErrInvalidCredentials` | NO | NO |
| Rate Limit Exceeded (HTTP 403) | `ErrRateLimitExceeded` | NO | NO |
| Provider Internal Error (HTTP 5xx / `AB2001`) | `ErrProviderInternalError` | NO | YES |
| Timeout / Dial Error | `ErrProviderNetworkTimeout` | NO | YES |
| Bad JSON Envelope | `ErrProviderMalformedResponse` | NO | YES |

## 3. ICICI Direct Error Normalization Matrix
| Provider Condition | Internal Normalized Error | Retry Allowed? | CircuitBreaker Counted? |
|---|---|---|---|
| Invalid Instrument (Missing CSV Token) | `ErrUnsupportedInstrument` | NO | NO |
| Rejected Order | `ErrOrderRejected` | NO | NO |
| Invalid Token/Session (HTTP 401) | `ErrSessionExpired` | YES (Auth Flow) | NO |
| Invalid Checksum (HTTP 401) | `ErrInvalidSignature` | NO | NO |
| Rate Limit Exceeded (HTTP 429 / 403) | `ErrRateLimitExceeded` | NO | NO |
| Time Sync Error >60s (HTTP 400) | `ErrInvalidTimestamp` | NO | NO |
| Generic Failure (HTTP 500) | `ErrProviderInternalError` | NO | YES |
| Timeout / Dial Error | `ErrProviderNetworkTimeout` | NO | YES |
| Bad JSON Envelope | `ErrProviderMalformedResponse` | NO | YES |

## 4. Generic Business Error Fallback (Gap Resolved)
For provider HTTP 4xx responses (such as Invalid Price, Invalid Order Parameters, ICICI Insufficient Funds, ICICI Invalid Quantity) where the provider documentation does not define a specific JSON business-error code:
- **Map response to:** `UNKNOWN_PROVIDER_BUSINESS_ERROR`
- **Retry:** NO
- **CircuitBreaker increment:** NO
- **Order retry:** NO
- **Provider-specific error code:** DO NOT FABRICATE
- **Original provider HTTP status:** PRESERVE
- **Provider response:** REDACTED and preserved only where safe for diagnostics.
