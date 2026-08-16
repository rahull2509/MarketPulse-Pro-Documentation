# Decision Register

This register records architectural and product decisions, ensuring strict traceability to approved MarketPulse Pro sources.

## Decision Summary
The repository establishes a clear separation between resolved platform architecture and pending product-specific integrations. The approved core stack, modular monolith pattern, WebSocket realtime model, and multi-broker UI scope are treated as fixed decision anchors, while broker-adapter contracts and deployment specifics remain intentionally deferred until they are documented and approved.

## Resolved Decisions

**DECISION ID**: DEC-ARCH-001
**DECISION**: Next.js / React
**EVIDENCE FILE**: `Volume_5_Implementation_Architecture.md`
**EVIDENCE SECTION**: Line 3924 (Enterprise Next.js Frontend Architecture, UI Composition & Client Application Specification)
**RATIONALE**: Explicitly approved and titled in Volume 5 as the enterprise architecture standard for the platform's client.
**SCOPE**: All frontend UI components and client applications in MarketPulse Pro.
**STATUS**: RESOLVED

**DECISION ID**: DEC-ARCH-002
**DECISION**: PostgreSQL (with Redis caching)
**EVIDENCE FILE**: `Volume_6_Engineering_Governance.md`, `Volume_3_Enterprise_Architecture.md`
**EVIDENCE SECTION**: Vol 6 Line 1943 (PostgreSQL Architecture), Vol 3 Line 2284 (PostgreSQL schemas)
**RATIONALE**: Explicitly cited as the primary relational store and architectural layer in the overarching Enterprise Volumes.
**SCOPE**: Primary relational database for user, configuration, and transactional data.
**STATUS**: RESOLVED

**DECISION ID**: DEC-ARCH-003
**DECISION**: WebSocket
**EVIDENCE FILE**: `Phase-2/IMPL_007_v2.0.md`
**EVIDENCE SECTION**: Line 22 (Realtime WebSocket & Event Distribution)
**RATIONALE**: A dedicated, approved Phase-2 IMPL exists solely for standardizing WebSocket as the realtime communication layer.
**SCOPE**: All realtime market data streaming and client updates.
**STATUS**: RESOLVED

**DECISION ID**: DEC-ARCH-004A
**DECISION**: Broker Login/Product UI Scope (Zerodha, Angel One, ICICI Direct)
**EVIDENCE FILE**: `Volume_1_Business_Foundation.md`
**EVIDENCE SECTION**: Sensibull UX Reference material
**RATIONALE**: The MarketPulse Pro UI must strictly match the approved Sensibull reference broker set.
**SCOPE**: Supported broker login options presented in the frontend UI.
**STATUS**: RESOLVED

**DECISION ID**: DEC-ARCH-004B
**DECISION**: Multi-Broker Backend Architecture
**EVIDENCE FILE**: `ADR/ADR-001-Multi-Broker-Architecture.md`
**EVIDENCE SECTION**: Entire Document
**RATIONALE**: MarketPulse Pro will use a provider-agnostic Broker Integration Layer with separate broker adapters for Zerodha, Angel One, and ICICI Direct. Provider-specific API implementation details remain subject to their corresponding technical specifications.
**SCOPE**: Backend integration layer boundaries, security, authentication, and isolation handling for multiple brokers.
**STATUS**: RESOLVED

## Pending / Deferred Decisions

**DECISION ID**: DEC-ARCH-004C
**DECISION**: Zerodha API Integration Specification
**EVIDENCE FILE**: N/A
**EVIDENCE SECTION**: N/A
**RATIONALE**: The Multi-Broker architecture defines the layer boundary, but the specific technical contract, SDK usage, and payloads for Zerodha are not yet documented.
**SCOPE**: Zerodha specific API adapter implementation.
**STATUS**: PENDING (BLOCKING - BROKER INTEGRATION)

**DECISION ID**: DEC-ARCH-004D
**DECISION**: Angel One API Integration Specification
**EVIDENCE FILE**: N/A
**EVIDENCE SECTION**: N/A
**RATIONALE**: The Multi-Broker architecture defines the layer boundary, but the specific technical contract, SDK usage, and payloads for Angel One are not yet documented.
**SCOPE**: Angel One specific API adapter implementation.
**STATUS**: PENDING (BLOCKING - BROKER INTEGRATION)

**DECISION ID**: DEC-ARCH-004E
**DECISION**: ICICI Direct API Integration Specification
**EVIDENCE FILE**: N/A
**EVIDENCE SECTION**: N/A
**RATIONALE**: The Multi-Broker architecture defines the layer boundary, but the specific technical contract, SDK usage, and payloads for ICICI Direct are not yet documented.
**SCOPE**: ICICI Direct specific API adapter implementation.
**STATUS**: PENDING (BLOCKING - BROKER INTEGRATION)

**DECISION ID**: DEC-ARCH-005
**DECISION**: Cloud Object Storage Provider (S3 vs Alternatives)
**EVIDENCE FILE**: `Phase-2/IMPL_010_v2.0.md`
**EVIDENCE SECTION**: Various mentions of S3 Adapter
**RATIONALE**: Mentions of S3 exist in IMPLs, but it is ambiguous if this is inherited legacy architecture or an explicit new MarketPulse Pro decision.
**SCOPE**: Blob storage for exports/reports.
**STATUS**: PENDING (NON-BLOCKING)

**DECISION ID**: DEC-ARCH-006
**DECISION**: Data Pipeline Format (Parquet)
**EVIDENCE FILE**: `Phase-1/SPEC_005.md`
**EVIDENCE SECTION**: Historical Data Pipelines
**RATIONALE**: Parquet is mentioned for analytics data dumps, but needs explicit confirmation against new MarketPulse Pro requirements.
**SCOPE**: Historical data lake storage format.
**STATUS**: PENDING (DEFERRED)


### DEC-ARCH-007
**DECISION**: Event Bus & Realtime Message Distribution (Redis Pub/Sub)
**STATUS**: **RESOLVED**
**EVIDENCE SECTION**: SPEC_007 (Redis Pub/Sub Bridge), IMPL_005.
**RATIONALE**: While higher-level enterprise artifacts (Volume 3) theoretically mention Kafka and NATS for future scale, the explicit implementation specifications (SPEC_007, IMPL_005) actively approve and define Redis Pub/Sub for all realtime fan-out and messaging. Kafka and NATS are therefore PROPOSED/DEFERRED for future scalability and are NOT approved for Phase 3.
**IMPACT**: Phase 3 implementation must use Redis Pub/Sub for all internal publish/subscribe capabilities.

### DEC-ARCH-008
**DECISION**: Core Infrastructure & Deployment Stack (EC2, Docker, Nginx, GitHub Actions, systemd)
**STATUS**: **PENDING (NON-BLOCKING)**
**EVIDENCE SECTION**: IMPL_012 defines EC2, Docker, and Nginx for deployment.
**RATIONALE**: IMPL_012 specifies these technologies as deployment details, but a central enterprise infrastructure decision has not formally ratified the cloud/compute vendor for the target architecture. This decision is pending, but does not block Phase 3 application code development.
**IMPACT**: Application codebase (Go, Next.js) may proceed as container-agnostic implementations. Deployment implementations remain on hold.

### DEC-FRONTEND-001
**DECISION**: Frontend State Management — TanStack Query + Zustand
**STATUS**: **RESOLVED**
**EVIDENCE SECTION**: User/Architect approval (P3 Foundation Remediation R7). IMPL_013 L20 mentions "Redux/Zustand or equivalent." IMPL_014 L20 mentions "SWR/React Query for clientside hydration." Volume 5 L4179: "Do not mandate specific state management libraries" — decision resolved by explicit architect approval.
**RATIONALE**: TanStack Query handles server state (API caching, background refetch, mutations). Zustand handles client state (auth, UI). This separation aligns with React best practices and Next.js App Router compatibility.
**SCOPE**: All frontend state management in the MarketPulse Pro Next.js application.
**IMPACT**: `@tanstack/react-query` and `zustand` are approved as GREEN dependencies for Phase 3 frontend implementation.

### DEC-ARCH-009
**DECISION**: Service Architecture Model — Modular Monolith with Role-Based Deployments
**STATUS**: **RESOLVED**
**EVIDENCE SECTION**: P4 Service Architecture Audit and P4 Architectural Plan Approval.
**RATIONALE**: Ensures failure isolation and independent scalability (100k-200k DAU) without introducing unnecessary distributed-system complexity and multi-repository overhead.
**SCOPE**: All Go backend deployments in Phase 4.
**IMPACT**: The existing Uber Fx application will be refactored to accept role profiles (e.g., API, WEBSOCKET, WORKER) rather than splitting into separate binaries.
