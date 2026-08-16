# P4-10 IMPLEMENTATION PREFLIGHT REPORT
**Phase:** P4-10 Production Hardening

## 1. REPOSITORY INSPECTION
- **Application bootstrap:** `internal/bootstrap/bootstrap.go`
- **HTTP server initialization:** `internal/core/server/server.go`
- **Gin/router initialization:** `internal/core/server/server.go` (setup) and `internal/routes/routes.go` (registration)
- **Middleware chain:** `internal/middleware/` (Currently only uses standard Gin Recovery and a bespoke `requestLogger` in `server.go`)
- **PostgreSQL initialization:** `internal/core/database/db.go`
- **Redis initialization:** `internal/core/redis/redis.go`
- **Broker module:** `internal/modules/broker/module.go`
- **P4-07 CircuitBreaker:** `internal/modules/broker/services/circuit_breaker.go`
- **P4-08 provider adapters:** `internal/modules/broker/providers/angelone/`, `icicidirect/`, `zerodha/`
- **P4-09 portfolio service:** `internal/modules/broker/services/portfolio_service.go`
- **WebSocket infrastructure:** `internal/websocket/` (hub.go, client.go, bridge.go)
- **Configuration loading:** `internal/config/config.go`
- **Logging infrastructure:** `internal/core/logger/logger.go`
- **Existing OpenTelemetry dependency:** Present in `go.mod`
- **Existing metrics infrastructure:** NONE
- **Graceful shutdown implementation:** Exists in `internal/bootstrap/bootstrap.go` (Lines 169-204)
- **Deployment/configuration files:** `configs/config.yaml`

## 2. DISTRIBUTED TRACING IMPACT
OpenTelemetry must be integrated at:
- **Incoming HTTP requests:** `internal/core/server/server.go` (add OTEL Gin middleware).
- **Outgoing HTTP requests:** Modify the HTTP client configuration inside the `BrokerAdapter` constructors (`internal/modules/broker/providers/*`) using `otelhttp` transport, without modifying the interface signature.
- **PostgreSQL:** `internal/core/database/db.go` (GORM OTEL plugin).
- **Redis:** `internal/core/redis/redis.go` (go-redis OTEL hook).
- **Broker provider HTTP calls:** Captured by `otelhttp` or explicitly wrapped in spans in `manager.go`.
- **WebSocket lifecycle/events:** Explicit span creation in `internal/websocket/client.go` and `hub.go`.
- **Confirmations:** W3C Trace Context, OTEL Collector, best-effort telemetry, and failure isolation are structurally possible.

## 3. TELEMETRY SECURITY IMPACT
- **Redaction boundaries:** Telemetry must be scrubbed before export. This requires a custom `SpanProcessor` or strict attribute whitelisting at span creation.
- **High-risk locations:**
  - `broker_handler.go` (JWT tokens in headers)
  - `manager.go` and adapter implementations (`ICICI_SESSION_TOKEN`, `ANGELONE_JWT`, MAC addresses)
  - `portfolio_service.go` (User IDs).
- **Rule:** Raw request/response bodies MUST NOT be added to telemetry.

## 4. OBSERVABILITY IMPACT
- **HTTP metrics:** `server.go` (Gin middleware).
- **PostgreSQL / Redis metrics:** Hooks in `db.go` and `redis.go`.
- **Broker / CircuitBreaker metrics:** `circuit_breaker.go` and `manager.go`.
- **Portfolio metrics:** `portfolio_service.go`.
- **WebSocket metrics:** `websocket/hub.go` and `websocket/client.go`.

## 5. CIRCUIT BREAKER REGRESSION CHECK
- P4-10 WILL NOT modify `internal/modules/broker/services/circuit_breaker.go`.
- Provider-isolated boundaries, >50% 5xx, 30s OPEN, 1 HALF-OPEN probe, and zero-retry semantics will remain unchanged.

## 6. P4-08 REGRESSION CHECK
- Angel One IP/MAC configuration and ICICI checksum flows remain completely unaffected. P4-10 strictly adds spans/metrics without altering logic.

## 7. P4-09 REGRESSION CHECK
- Portfolio aggregation, 5-second cache, stale fallback, singleflight, and `position.update` remain completely unaffected.

## 8. LOAD TEST IMPLEMENTATION
- **k6 tests location:** New directory `tests/load/`.
- **Mapping:**
  - Portfolio requests map to `GET /api/v1/broker/portfolio`.
  - Order requests map to `POST /api/v1/broker/orders`.
  - WebSocket connections map to `/ws`.
- Workload (100k DAU, 5000 peak concurrent, 500 RPS) can be modeled in k6 scenarios.

## 9. DEPLOYMENT HARDENING
- **Readiness/Liveness:** Already exists in `server.go` (`/ready`, `/health`).
- **Graceful Shutdown:** Currently exists in `internal/bootstrap/bootstrap.go:173`.
  - *Status:* It is currently set to `15 * time.Second`.
  - *Required Change:* Must be updated to `30 * time.Second` to meet governance.
- **Config/Secret Validation:** Must be added to `config.go` and `bootstrap.go`.

## 10. FRONTEND
- FRONTEND SCOPE = NONE. No frontend files will be modified.

## 11. DEPENDENCY IMPACT
- `go.opentelemetry.io/otel` is present. Standard OTEL instrumentation packages (e.g., `otelgin`, `otelgorm`) may be required but I am restricted from installing new dependencies during this phase unless approved.

## 12. IMPLEMENTATION FILE MATRIX

| File | Change Type | Purpose | Governance Reference | Risk |
|---|---|---|---|---|
| `internal/bootstrap/bootstrap.go` | MODIFY | Update graceful shutdown to 30s | P4-10-DEPLOYMENT-HARDENING | LOW |
| `internal/core/server/server.go` | MODIFY | Add OTEL Gin middleware, metric middleware | P4-10-OBSERVABILITY | LOW |
| `internal/core/database/db.go` | MODIFY | Add OTEL GORM plugin | P4-10-DISTRIBUTED-TRACING | LOW |
| `internal/core/redis/redis.go` | MODIFY | Add OTEL go-redis hook | P4-10-DISTRIBUTED-TRACING | LOW |
| `tests/load/k6_rest.js` | CREATE | Execute 100k DAU load profiles | P4-10-LOAD-TESTING | LOW |
| `tests/load/k6_ws.js` | CREATE | Execute WS concurrent connections | P4-10-LOAD-TESTING | LOW |

## 13. TEST FILE MATRIX

| Test File | Required Coverage |
|---|---|
| `internal/middleware/telemetry_test.go` | telemetry propagation, redaction, exporter failure |
| `internal/core/database/db_test.go` | PostgreSQL failure, PostgreSQL tracing |
| `internal/core/redis/redis_test.go` | Redis failure, Redis tracing |
| `internal/core/server/server_test.go` | graceful shutdown, HTTP tracing, metrics |
| `tests/load/k6_rest.js` | k6 REST, spike, soak, 100k DAU |
| `tests/load/k6_ws.js` | k6 WebSocket, spike, soak, 100k DAU |
| `tests/regression/...` | regression tests P4-07/08/09 |

## 14. STOP CONDITIONS
- Implementation does NOT require changing P4-07/08/09.
- No governance parameter is missing.
- Security redact boundaries are explicitly defined.
- No frontend modification is necessary.
- No global CircuitBreaker is being implemented.

## FINAL OUTPUT

**P4-10 PREFLIGHT = READY FOR IMPLEMENTATION**
