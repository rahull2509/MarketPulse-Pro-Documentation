# P4-10 IMPLEMENTATION PLAN
**Phase:** P4-10 Production Hardening
**Status:** IMPLEMENTATION = AUTHORIZED

*This document is the AUTHORIZED implementation plan for P4-10. All implementation-critical governance decisions have been explicitly approved by the Project Owner.*

## 1. Proposed Implementation Sequence
1. **Load Test Harness:** Create non-production k6 load testing scripts covering REST and WebSocket scenarios governed by the 100k DAU profile.
2. **Infrastructure Scaffolding:** Add OTEL and Metrics exporters to `bootstrap.go`, bound strictly to configuration flags.
3. **Context Propagation:** Ensure `context.Context` is strictly propagated through all application layers.
4. **Telemetry Instrumentation:** Instrument handlers, DB/Redis repositories, and Broker adapters with OTEL spans and metrics.
5. **Security Enforcement:** Implement strict scrubbing middleware to enforce the absolute telemetry denylist (tokens, PII, passwords).
6. **Deployment Hardening:** Add liveness/readiness endpoint routes and configure a strict 30-second graceful shutdown mechanism in the main server loop.

## 2. Affected Components
- `bootstrap.go`
- `middleware/telemetry.go` (new)
- `middleware/security_redactor.go` (new)
- DB/Redis repositories
- WebSocket connection lifecycle hooks

## 3. Regression Checks
The P4-10 changes MUST NOT:
- Replace the P4-07 Broker CircuitBreaker
- Change P4-07 order retry semantics
- Bypass P4-08 provider isolation
- Bypass P4-08 rate-limit handling
- Modify P4-09 aggregation mathematics
- Modify P4-09 Redis portfolio semantics
- Modify P4-09 `position.update` contract
- Weaken JWT user isolation
- Expose broker credentials

*Any conflict in these areas is a BLOCKING regression.*
