# P4-10 TELEMETRY SECURITY DECISION
**Phase:** P4-10 Production Hardening

## 1. Absolute Telemetry Denylist
Never record the following in logs, traces, span attributes, span events, baggage, metrics labels, or telemetry payloads:
- Authorization headers
- passwords
- API keys, API secrets
- broker access tokens, broker refresh tokens, ICICI_SESSION_TOKEN, ANGELONE_JWT
- request tokens, session tokens, OAuth credentials, TOTP secrets, cookies
- ANGELONE_CLIENT_MAC_ADDRESS
- sensitive PII

## 2. Explicit Allowlist Model
Telemetry attributes MUST follow an explicit allowlist model.
Allowed examples: HTTP method, route template, HTTP status code, latency, provider name, operation name, error classification, CircuitBreaker state, cache hit/miss, database operation class, WebSocket event type.

*(Never place raw request/response bodies into telemetry).*
