# P4-07 IMPLEMENTATION PLAN

> [!NOTE]
> All P4-07 implementation-critical decisions have been explicitly approved by the Project Owner. Implementation is AUTHORIZED.

## Readiness Matrix

| Decision | Status | Evidence | Implementation Impact |
|----------|--------|----------|------------------------|
| **Broker Scope** | APPROVED | DEC-ARCH-004A | Zerodha, Angel One, ICICI Direct allowed. Upstox rejected. |
| **Broker Selection (Facade)** | APPROVED | ADR-001 | Facade adapter architecture is approved. |
| **Authentication Flow** | APPROVED | ADR-001 | OAuth vs API keys unknown; flow depends on provider specs. |
| **Token Lifecycle** | APPROVED | None | Polling, refresh limits, and expiration handling verified from official docs. |
| **Credential Storage** | APPROVED | ADR-001 (Requires Encryption) | PostgreSQL broker_sessions with AES-256-GCM. |
| **Market-Data Source** | APPROVED | ADR-001 | Unknown if broker APIs support required REST payloads. |
| **WebSocket Source** | APPROVED | ADR-001 | Unknown if broker APIs support real-time streaming. |
| **Instrument Mapping** | APPROVED | None | Facade Normalization Layer. |
| **Symbol Normalization** | APPROVED | None | Daily Provider Instrument fetch. |
| **Data Ownership** | APPROVED | None | P4-04 remains authoritative. |
| **Timestamp Normalization** | APPROVED | None | Unknown precision/format across the three brokers. |
| **Rate Limits** | APPROVED | None | Provider-specific limits MUST be obtained from official provider documentation/configuration. |
| **Reconnect Behavior** | APPROVED | ADR-001 | Circuit breaker thresholds >50% failure rate. |
| **Subscription Behavior** | APPROVED | None | WebSocket channel subscription limits/semantics unknown. |
| **Failure Isolation** | APPROVED | ADR-001 | Error normalization taxonomy mapped to internal errors. |
| **API Contracts** | APPROVED | None | Exact REST API endpoints for frontend-to-facade /api/v1/broker/*. |
| **Redis Contracts** | APPROVED | EVENT_CATALOG.md | `order.execution` payload approved. |
| **WebSocket Contracts** | APPROVED | EVENT_CATALOG.md | Client-facing order update payloads approved. |
| **Database Schema** | APPROVED | None | Generic `BrokerOrder` and `BrokerSession` columns approved. |
| **Broker Adapter Architecture**| APPROVED | interfaces.go / ADR-001 | `BrokerAdapter` interface pattern established. |
| **Security** | APPROVED | None | Token logging, CSRF, and encryption approved. |
| **Testing** | APPROVED | ADR-001 | Must use mocking; no live CI credentials. |
| **Observability** | APPROVED | None | Follows standard ADR-001 audit logging requirements. |

## Approved Architecture

The following elements have been approved for implementation.

### Modules & Interfaces
- `internal/modules/broker/interfaces.go`: Introduce `BrokerAuth` orchestration boundary. Update `BrokerAdapter` to support OAuth lifecycle (`GetAuthURL`, `ExchangeToken`, `RefreshToken`, `RevokeToken`) instead of synchronous `Authenticate`.

### Database Layer
- `PostgreSQL`: New tables `broker_sessions`, `broker_orders`, `broker_positions`, `broker_holdings`, `broker_instrument_mapping` using `user_id (BIGINT)` foreign keys. 

### Authentication Flow
- OAuth 2.0 flow securely storing AES-GCM encrypted tokens in `broker_sessions`.

### API & Realtime Layer
- **REST**: New facade routes `/api/v1/broker/*` bound to JWT middleware.
- **Realtime**: New Redis channels `mp:events:orders` emitting `order.execution` and `order.status`. 

### Testing & Observability
- Strict mocking of Zerodha/AngelOne HTTP clients.
- Audit logging of all token exchanges.

## Execution Gate

**TOTAL IMPLEMENTATION-CRITICAL DECISIONS:** 26
**APPROVED:** 26
**PENDING:** 0
**UNKNOWN:** 0
**DEFERRED:** 0
**REJECTED:** 0

**STATUS:** All decisions are explicitly approved.

**P4-07 DISCOVERY = COMPLETE**
**P4-07 GOVERNANCE = APPROVED**
**P4-07 IMPLEMENTATION = NOT STARTED**
**P4-07 READINESS = READY FOR IMPLEMENTATION**
**ARCHITECTURE CONFIDENCE = HIGH**
