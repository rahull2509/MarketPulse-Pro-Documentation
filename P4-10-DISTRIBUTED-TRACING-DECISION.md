# P4-10 DISTRIBUTED TRACING DECISION
**Phase:** P4-10 Production Hardening

## 1. Tracing Standard
- Standard: OpenTelemetry
- Propagation: W3C Trace Context

## 2. Tracing Scope
- Incoming HTTP requests
- Outgoing HTTP requests
- PostgreSQL operations
- Redis operations
- Broker provider HTTP calls
- WebSocket lifecycle/events

## 3. Span Naming & Sampling
- Naming: Use stable operation-based names. Do not include user IDs, tokens, symbols containing sensitive data, credentials, or request payloads in span names.
- Sampling: Production sampling MUST be governed by configuration (no hardcoded percentage).

## 4. Architecture & Exporter
- Exporter: OpenTelemetry Collector. Application services MUST export telemetry to the collector.
- Trace retention: Do NOT implement application-level trace persistence. Retention is owned by the telemetry backend/collector infrastructure.

## 5. Tracing Failure
- Telemetry/exporter failure MUST NEVER fail business requests. Telemetry is best-effort.
