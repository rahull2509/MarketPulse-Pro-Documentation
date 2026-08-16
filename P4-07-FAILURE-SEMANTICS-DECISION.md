# P4-07 Failure & Resilience Semantics Decision

This document defines the approved error handling, resilience patterns, and failure semantics for the Broker Integration Facade.

| Scenario | Proposed Policy | Governance Status |
|----------|-----------------|-------------------|
| **API Request Timeout** | Facade enforces strict Context timeouts (e.g. 5s for orders, 10s for data) | APPROVED |
| **API Retry Policy** | 0 retries for Order Placement. Up to 3 retries with exponential backoff for reads (Positions/Holdings). | APPROVED |
| **Broker Rate Limits** | Provider-specific limits MUST be obtained from official provider documentation/configuration. | APPROVED |
| **Circuit Breaker** | Open circuit if provider returns >50% 5xx errors over a 1-minute window. | APPROVED |
| **Token Expiry** | Proactive refresh attempt if token expires in < 5 mins; else synchronously refresh on 401. | APPROVED |
| **Broker Outage** | Bubble up `503 Service Unavailable` with specific `ERR_BROKER_OUTAGE` code. | APPROVED |
| **Malformed Response** | Log raw payload as `ERROR` and return `502 Bad Gateway` to client. | APPROVED |
| **Duplicate Order Response** | Idempotency key (UUID) prevents duplicate dispatch; broker rejection returned as standard error. | APPROVED |
| **Partial Execution** | Treated as valid state transition (`order.execution`); status remains `OPEN` until fully filled or cancelled. | APPROVED |
| **Network Failure** | Bubble up as `504 Gateway Timeout` or `502 Bad Gateway`. | APPROVED |
| **Provider Maintenance** | Map provider's maintenance code to `503 Service Unavailable` with `ERR_BROKER_MAINTENANCE`. | APPROVED |
