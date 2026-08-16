# Technology Stack Baseline

## 1. Document Information
- **Title**: MarketPulse Pro Technology Baseline
- **Version**: 1.1
- **Status**: AUTHORITATIVE
- **Purpose**: Establishes the definitive technology stack for MarketPulse Pro based on governed architecture artifacts.

## 2. Scope
This document covers all technology layers and explicitly decouples Architectural Approval from Implementation Status, establishing a scope-specific readiness model for Phase 3.

## 3. Technology Governance Rules
1. **Four-Dimensional Tracking**: Every technology is tracked by Document Status, Architecture Status, Implementation Status, and Phase 3 Status.
2. **Implementation vs. Approval**: Repository codebase being zero (E0) means Implementation = `NOT YET IMPLEMENTED`. It does NOT downgrade Architecture Status.
3. **Hierarchy**: `DECISION_REGISTER.md` > Explicit Approved Architecture > Implementation Spec > Foundation Specs. However, older artifacts remain historically valid for their bounded scopes unless explicitly superseded.

## 4. Technology Evidence Matrix

| Technology | Category | Document Status | Architecture Status | Implementation Status | Phase 3 Status | Decision/Authority |
|---|---|---|---|---|---|---|
| **Go (1.25+)** | Backend | High (109 mentions) | **APPROVED** | **NOT YET IMPLEMENTED** | **GREEN** | Vol 5, IMPL_006 |
| **Gin** | API | High | **APPROVED** | **NOT YET IMPLEMENTED** | **GREEN** | IMPL_006 |
| **GORM** | Database | High | **APPROVED** | **NOT YET IMPLEMENTED** | **GREEN** | IMPL_003 |
| **Uber Fx** | Core | High | **APPROVED** | **NOT YET IMPLEMENTED** | **GREEN** | IMPL_002 |
| **Viper** | Config | High | **APPROVED** | **NOT YET IMPLEMENTED** | **GREEN** | IMPL_006 |
| **Zap / Uber Zap**| Logging | High | **APPROVED** | **NOT YET IMPLEMENTED** | **GREEN** | IMPL_001 |
| **PostgreSQL** | Database | High (118 mentions) | **APPROVED** | **NOT YET IMPLEMENTED** | **GREEN** | DEC-ARCH |
| **ClickHouse** | Analytics | High (60 mentions) | **APPROVED** | **NOT YET IMPLEMENTED** | **GREEN** | Vol 5, SPEC_001 |
| **Redis** | Cache | High (374 mentions) | **APPROVED** | **NOT YET IMPLEMENTED** | **GREEN** | Vol 5, IMPL_006 |
| **Gorilla WebSocket**| Realtime | High | **APPROVED** | **NOT YET IMPLEMENTED** | **GREEN** | IMPL_007 |
| **Redis Pub/Sub** | Messaging | High (30 mentions) | **APPROVED** | **NOT YET IMPLEMENTED** | **GREEN** | SPEC_007, IMPL_005 |
| **Asynq** | Queue | High | **APPROVED** | **NOT YET IMPLEMENTED** | **GREEN** | IMPL_002 |
| **gocron/v2** | Scheduler | High | **APPROVED** | **NOT YET IMPLEMENTED** | **GREEN** | IMPL_001 |
| **Prometheus** | Metrics | High | **APPROVED** | **NOT YET IMPLEMENTED** | **GREEN** | IMPL_002 |
| **Grafana** | Dashboards | High | **APPROVED** | **NOT YET IMPLEMENTED** | **GREEN** | IMPL_002 |
| **OpenTelemetry** | Tracing | High | **APPROVED** | **NOT YET IMPLEMENTED** | **GREEN** | IMPL_005 |
| **Sentry** | Error Tracking | High | **APPROVED** | **NOT YET IMPLEMENTED** | **GREEN** | IMPL_012 |
| **Next.js** | Frontend | High | **APPROVED** | **NOT YET IMPLEMENTED** | **GREEN** | Vol 5, DEC-ARCH |
| **React** | Frontend | High | **APPROVED** | **NOT YET IMPLEMENTED** | **GREEN** | Vol 5 |
| **TanStack Query** | Frontend State | High | **APPROVED** | **NOT YET IMPLEMENTED** | **GREEN** | DEC-FRONTEND-001 |
| **Zustand** | Frontend State | High | **APPROVED** | **NOT YET IMPLEMENTED** | **GREEN** | DEC-FRONTEND-001 |
| **golang-migrate** | DB Migration | High | **APPROVED** | **NOT YET IMPLEMENTED** | **GREEN** | IMPL_003 |

| **JWT** | Auth | High | **APPROVED** | **NOT YET IMPLEMENTED** | **GREEN** | SPEC_004 |
| **Docker** | Deployment | Medium (IMPL_012) | **PROVISIONALLY APPROVED** | **NOT YET IMPLEMENTED** | **YELLOW** | IMPL_012 |
| **Nginx** | Reverse Proxy | Medium (IMPL_012) | **PROVISIONALLY APPROVED** | **NOT YET IMPLEMENTED** | **YELLOW** | IMPL_012 |
| **EC2** | Infrastructure | Low | **PENDING** | **NOT YET IMPLEMENTED** | **YELLOW** | IMPL_012 (Proposed) |
| **GitHub Actions**| CI/CD | Low | **PENDING** | **NOT YET IMPLEMENTED** | **YELLOW** | IMPL_012 (Proposed) |
| **systemd** | Deployment | Low | **PENDING** | **NOT YET IMPLEMENTED** | **YELLOW** | IMPL_012 (Proposed) |
| **AWS S3** | Storage | High (Historical) | **PENDING** | **NOT YET IMPLEMENTED** | **YELLOW** | DEC-ARCH-005 |
| **Parquet** | Format | Medium (Historical) | **DEFERRED** | **NOT YET IMPLEMENTED** | **YELLOW** | DEC-ARCH-006 |
| **Kafka / NATS** | Messaging | Low (Conceptual) | **REFERENCE / NOT SELECTED** | **NOT YET IMPLEMENTED** | **YELLOW** | Volume 3 |
| **Python** | Backend | Low (Historical) | **LEGACY** | **NOT YET IMPLEMENTED** | **RED** | Older Phase 2 IMPLs |
| **APScheduler** | Scheduler | Low | **LEGACY** | **NOT YET IMPLEMENTED** | **RED** | Older Python Specs |
| **Upstox** | Integration | High (Historical) | **REJECTED** | **NOT YET IMPLEMENTED** | **RED** | DEC-ARCH-004A |
| **Zerodha API** | Integration | Medium | **PENDING** | **NOT YET IMPLEMENTED** | **YELLOW** | DEC-ARCH-004C |
| **Angel One API** | Integration | Medium | **PENDING** | **NOT YET IMPLEMENTED** | **YELLOW** | DEC-ARCH-004D |
| **ICICI Direct API**| Integration | Medium | **PENDING** | **NOT YET IMPLEMENTED** | **YELLOW** | DEC-ARCH-004E |

## 5. Phase 3 Technology Gate

### 🟢 GREEN (Safe for Phase 3 Implementation)
The core architecture stack is explicitly approved and safe for immediate implementation:
- Go (1.25+), Gin, GORM, Uber Fx, Viper, Zap, PostgreSQL, ClickHouse, Redis, Next.js, React, Gorilla WebSocket, Redis Pub/Sub, Asynq, gocron/v2, Prometheus, Grafana, OpenTelemetry, Sentry, JWT.

### 🟡 YELLOW (Conditional / Decision Required Before Use)
These technologies may not be implemented for their respective domains until explicitly governed:
- **AWS S3**: Cloud Object Storage provider is officially TBD. (Blocks data pipeline/storage modules).
- **Parquet**: Pipeline format is officially DEFERRED. (Blocks analytical export implementations).
- **Kafka / NATS**: Reference concepts only. Redis Pub/Sub must be used for event distribution.
- **Docker / Nginx**: Provisionally approved, conditional on final infrastructure deployment decisions.
- **EC2 / GitHub Actions / systemd**: Infrastructure decisions remain pending.
- **Zerodha / Angel One / ICICI Direct APIs**: Broker adapters are conditional pending 004C/D/E specifications.

### 🔴 RED (Must Not Be Introduced)
These technologies are explicitly prohibited from Phase 3 implementations:
- **Upstox**: Explicitly rejected from current UI scope.
- **Python / APScheduler**: Legacy backend.

## 6. Scope-Specific Readiness Model

The Phase 3 implementation is **NOT globally blocked**. Readiness is evaluated by bounded context:

- **CORE PLATFORM READINESS**: `READY`
  - *Engineering can start building the backend APIs, authentication, core Next.js frontend, database schemas (PostgreSQL/ClickHouse), and realtime web sockets immediately.*

- **DATA PIPELINE READINESS**: `CONDITIONAL / BLOCKED`
  - *Blocked ONLY for modules that actually require the unresolved S3 storage or Parquet data-format architecture (DEC-ARCH-005/006).*

- **BROKER INTEGRATION READINESS**: `CONDITIONAL / BLOCKED`
  - *Blocked ONLY for the provider-specific adapter implementations pending DEC-ARCH-004C/D/E. Abstract architecture implementation is safe.*
