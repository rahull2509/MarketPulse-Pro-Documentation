# P4-10 FAILURE SEMANTICS DECISION
**Phase:** P4-10 Production Hardening

## 1. Semantics
- **PostgreSQL failure:** Existing governed service-specific error behavior remains unchanged. No global CircuitBreaker is introduced.
- **Redis failure:** Redis-dependent cache/realtime functionality must fail according to its existing P4-09 semantics. Redis failure MUST NOT expose credentials or corrupt portfolio data.
- **Broker provider failure:** Preserve P4-07/P4-08 semantics exactly.
- **Provider 429:** Preserve `ErrRateLimitExceeded`. Do not convert it into a 5xx failure.
- **Provider 5xx:** Preserve existing provider CircuitBreaker semantics.
- **WebSocket failure:** Preserve P4-05/P4-09 reconnect and sequence semantics.
- **Tracing / Metrics failure:** MUST NOT fail the business request.
- **Application restart:** Business correctness MUST NOT depend on in-memory telemetry state.
