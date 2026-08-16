# P4-10 GOVERNANCE GAP RESOLUTION
**Phase:** P4-10 Production Hardening

## 1. CIRCUIT BREAKER GOVERNANCE
- Services receiving global CBs: UNKNOWN — PROJECT OWNER DECISION REQUIRED
- Existing P4-07 Broker CircuitBreaker: MUST REMAIN ISOLATED. MUST NOT BE REPLACED OR MODIFIED.
- New CB parameters (failure condition, rolling window, threshold, OPEN duration, HALF-OPEN transition/probes, recovery, concurrency, retries): UNKNOWN — PROJECT OWNER DECISION REQUIRED

## 2. DISTRIBUTED TRACING
- OpenTelemetry usage, W3C Trace Context, span boundaries, naming, sampling, exporter, collector, environment behavior, retention, failure behavior: UNKNOWN — PROJECT OWNER DECISION REQUIRED
- **CRITICAL SECURITY RULE:** The following MUST NEVER enter traces, logs, span attributes, span events, baggage, or telemetry payloads: Authorization headers, passwords, API keys, client secrets, broker access tokens, broker refresh tokens, request tokens, session tokens, TOTP secrets, cookies, raw OAuth credentials, sensitive PII.

## 3. FRONTEND OBSERVABILITY
- Frontend scope (Included vs Excluded): UNKNOWN — PROJECT OWNER DECISION REQUIRED
- Trace data, privacy, ownership, sampling (if included): UNKNOWN — PROJECT OWNER DECISION REQUIRED

## 4. 100K DAU CAPACITY GOVERNANCE (CRITICAL)
- DAU definition, active-user calculation window, peak/average concurrent users, peak/average requests/sec, WebSocket connections, portfolio/order/broker/market-data requests/sec, cache hit/miss ratio, Redis/Postgres operations/sec, throughput, duration boundaries, acceptable latency (p50/p95/p99), maximum error percentage: UNKNOWN — PROJECT OWNER DECISION REQUIRED

## 5. LOAD TEST FRAMEWORK
- Framework, language, environment, distributed load, test data, simulations, reporting, pass/fail: UNKNOWN — PROJECT OWNER DECISION REQUIRED

## 6. CAPACITY / HARDWARE
- Backend instance type/count, CPU/Memory target, DB/Redis sizing, LB, network, horizontal scaling, autoscaling, storage: UNKNOWN — PROJECT OWNER DECISION REQUIRED

## 7. FAILURE SEMANTICS
- DB/Redis/API outage, 429/5xx, WS/Tracing/Metrics outage, app restart, CB OPEN: UNKNOWN — PROJECT OWNER DECISION REQUIRED
- **CRITICAL:** Telemetry failure MUST NOT automatically become business-service failure.

## 8. OBSERVABILITY
- Logs, metrics, traces, correlation IDs, latency/error/dependency/broker/DB/WS metrics, backend: UNKNOWN — PROJECT OWNER DECISION REQUIRED

## 9. TELEMETRY SECURITY
- Secret redaction, PII masking, Token masking, Authorization masking, broker credential masking: REQUIRED.
- Attribute allowlists, access control, retention, production debug: UNKNOWN — PROJECT OWNER DECISION REQUIRED

## 10. WEBSOCKET HARDENING
- Existing P4-09 `position.update`, sequence, user isolation, REST fallback: MUST REMAIN UNCHANGED.
- Heartbeat, ping/pong, connection limits, backpressure, reconnect, graceful shutdown: UNKNOWN — PROJECT OWNER DECISION REQUIRED

## 11. TEST GOVERNANCE
- Mandatory tests for CB, tracing, redaction, DB/Redis failure, WS, load/stress/soak, DAU profile, pass/fail criteria: UNKNOWN — PROJECT OWNER DECISION REQUIRED

## 12. DEPLOYMENT HARDENING
- Readiness/liveness/startup probes, graceful shutdown, config validation, secrets, resource limits, autoscaling, rollback, monitoring, alerting: UNKNOWN — PROJECT OWNER DECISION REQUIRED

## 13. P4-07 / P4-08 / P4-09 REGRESSION LOCK
**EXPLICITLY PRESERVED:**
- **P4-07:** Broker CircuitBreaker, zero-retry order semantics, OAuth state security, InstrumentRepository rules.
- **P4-08:** Angel One IP/MAC configuration, provider rate limits, provider error semantics, credential isolation.
- **P4-09:** Portfolio aggregation mathematics, Redis 5-second aggregate cache, stale provider fallback, `position.update` contract, sequence handling, JWT isolation, FRONTEND SCOPE = NONE (unless explicitly approved).
