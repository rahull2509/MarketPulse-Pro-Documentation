# P4-10 GOVERNANCE APPROVAL MATRIX
**Phase:** P4-10 Production Hardening

| Decision | Proposed Value | Evidence | Status | Blocking? |
|---|---|---|---|---|
| Circuit Breaker Scope | No global CB; existing P4-07 CB remains untouched. | P4-10 PO Decisions | APPROVED | NO |
| Distributed Tracing | OTEL Collector, trace context, explicit spans. | P4-10 PO Decisions | APPROVED | NO |
| Telemetry Security | Explicit Denylist (tokens, passwords) & Allowlist. | P4-10 PO Decisions | APPROVED | NO |
| Frontend Scope | FRONTEND SCOPE = NONE. | P4-10 PO Decisions | APPROVED | NO |
| Load Test Framework | k6 | P4-10 PO Decisions | APPROVED | NO |
| 100K DAU Workload | 5,000 peak users, 500 max RPS, p99<=1s | P4-10 PO Decisions | APPROVED | NO |
| Capacity / Hardware | Result-driven based on load tests. | P4-10 PO Decisions | APPROVED | NO |
| Failure Semantics | Observability failure != business failure. DB/Redis failure semantics preserved. | P4-10 PO Decisions | APPROVED | NO |
| Observability Metrics | Defined list of explicit HTTP/Redis/WS metrics. | P4-10 PO Decisions | APPROVED | NO |
| Deployment Hardening | Startup probes, config validation, graceful shutdown. | P4-10 PO Decisions | APPROVED | NO |
| Graceful Shutdown Timeout | 30 seconds | P4-10 PO Decisions | APPROVED | NO |
| Test Governance | 16 mandatory tests, latency SLA <= 1% error. | P4-10 PO Decisions | APPROVED | NO |
| Regression Lock | P4-07, P4-08, P4-09 contracts strictly preserved. | P4-10 PO Decisions | APPROVED | NO |
