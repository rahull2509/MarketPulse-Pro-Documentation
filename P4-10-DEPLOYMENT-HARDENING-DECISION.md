# P4-10 DEPLOYMENT HARDENING DECISION
**Phase:** P4-10 Production Hardening

## 1. Required Deployment Semantics
- readiness probe
- liveness probe
- graceful shutdown
- configuration validation
- secret validation
- telemetry configuration validation

## 2. Hardening Rules
- Deployment MUST fail startup when required production security configuration is invalid.
- Graceful shutdown MUST allow active business requests to complete within the governed shutdown period.
- **Graceful shutdown timeout:** 30 seconds. 

## 3. Graceful Shutdown Governance (30 Seconds)
- The application MUST allow up to 30 seconds for active business requests/connections to terminate gracefully during shutdown.
- After the 30-second graceful shutdown period expires, normal termination behavior may proceed.
- This timeout MUST NOT alter P4-07 order execution semantics.
- This timeout MUST NOT introduce retries for broker writes.
- This timeout MUST NOT alter P4-08 provider behavior.
- This timeout MUST NOT alter P4-09 WebSocket sequence, reconnect, or portfolio semantics.
