# P4-10 CIRCUIT BREAKER DECISION
**Phase:** P4-10 Production Hardening

## 1. Scope
**Global CircuitBreakers:**
P4-10 SHALL NOT introduce a new global CircuitBreaker.
- PostgreSQL: NO
- Redis: NO
- Broker providers: NO
- Market Data provider calls: NO
- Other dependencies: NO

**Existing P4-07 Broker CircuitBreaker:**
- MUST remain completely unchanged.
- MUST remain provider-isolated.
- MUST NOT be replaced by a global CircuitBreaker.
P4-10 production hardening must preserve the existing P4-07 provider-isolated CircuitBreaker as-is.
