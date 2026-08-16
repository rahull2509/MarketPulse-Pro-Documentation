# P4-10 TEST GOVERNANCE DECISION
**Phase:** P4-10 Production Hardening

## 1. Mandatory Tests
1. OpenTelemetry propagation
2. Secret/PII redaction
3. Telemetry exporter failure isolation
4. PostgreSQL failure behavior
5. Redis failure behavior
6. Broker provider failure behavior
7. WebSocket observability
8. concurrent request tracing
9. k6 REST load test
10. k6 WebSocket load test
11. spike test
12. soak test
13. 100k DAU workload profile
14. regression tests for P4-07
15. regression tests for P4-08
16. regression tests for P4-09

## 2. Load-test Pass Criteria
- p50 <= 200 ms
- p95 <= 500 ms
- p99 <= 1000 ms
- error rate <= 1%
