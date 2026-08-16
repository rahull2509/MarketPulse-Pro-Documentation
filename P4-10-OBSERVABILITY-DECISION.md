# P4-10 OBSERVABILITY DECISION
**Phase:** P4-10 Production Hardening

## 1. Required Metrics
- HTTP request count, HTTP latency, HTTP error count
- PostgreSQL latency/errors
- Redis latency/errors
- Broker provider latency/errors, Broker CircuitBreaker state
- Portfolio cache hit/miss, Portfolio aggregation latency
- WebSocket connection count, WebSocket disconnect count, WebSocket event count, WebSocket error count

## 2. Required Identifiers
- request ID
- trace ID
*(Do NOT expose user IDs or credentials as metric labels).*
