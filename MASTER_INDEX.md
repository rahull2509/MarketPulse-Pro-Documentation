# Master Index

This is the authoritative navigation index for MarketPulse Pro.

## Project Overview
- [README.md](README.md)
- [AGENTS.md](AGENTS.md)
- [00_Enterprise_AI_Operating_Manual.md](00_Enterprise_AI_Operating_Manual.md)

## Evidence & Quality Gates
- Architecture is treated as the authoritative specification for MarketPulse Pro, not a loose working note.
- All decisions are mapped to explicit evidence files, with unresolved items clearly marked as pending or deferred.
- Phase 3 readiness is gated on documentation audits, product scope alignment, dependency validation, and decision traceability.
- Architecture work intentionally excludes legacy assumptions from older MarketPulse work unless they are explicitly approved in the current project.

## Core Governance & Traceability
- [DOCUMENTATION_CHANGELOG.md](DOCUMENTATION_CHANGELOG.md)
- [DECISION_REGISTER.md](DECISION_REGISTER.md)
- [FEATURE_TRACEABILITY.md](FEATURE_TRACEABILITY.md)
- [UI_UX_REQUIREMENT_MATRIX.md](UI_UX_REQUIREMENT_MATRIX.md)

## Architecture
- [ARCHITECTURE_MAP.md](ARCHITECTURE_MAP.md)
- [DEPENDENCY_GRAPH.md](DEPENDENCY_GRAPH.md)

## Architecture Decision Records (ADRs)
- [ADR-001: Multi-Broker Architecture](ADR/ADR-001-Multi-Broker-Architecture.md)

## Enterprise Volumes
- [Volume 1: Business Foundation](Volume_1_Business_Foundation.md)
- [Volume 2: Requirements Engineering](Volume_2_Requirements.md)
- [Volume 3: Enterprise Architecture](Volume_3_Enterprise_Architecture.md)
- [Volume 4: Engineering Delivery](Volume_4_Engineering_Delivery.md)
- [Volume 5: Implementation Architecture](Volume_5_Implementation_Architecture.md)
- [Volume 6: Engineering Governance](Volume_6_Engineering_Governance.md)

## Phase-1 Functional Specifications
- [SPEC_001](Phase-1/SPEC_001.md)
- [SPEC_002](Phase-1/SPEC_002.md)
- [SPEC_003](Phase-1/SPEC_003.md)
- [SPEC_004](Phase-1/SPEC_004.md)
- [SPEC_005](Phase-1/SPEC_005.md)
- [SPEC_006](Phase-1/SPEC_006.md)
- [SPEC_007](Phase-1/SPEC_007.md)
- [SPEC_008](Phase-1/SPEC_008.md)
- [SPEC_009](Phase-1/SPEC_009.md)
- [SPEC_010](Phase-1/SPEC_010.md)
- [SPEC_011: Strategy Builder](Phase-1/SPEC_011.md)
- [SPEC_012: Home Dashboard](Phase-1/SPEC_012.md)

## Phase-2 Implementation Blueprints
- [IMPL_001_v2.0](Phase-2/IMPL_001_v2.0.md)
- [IMPL_002_v2.0](Phase-2/IMPL_002_v2.0.md)
- [IMPL_003_v2.0](Phase-2/IMPL_003_v2.0.md)
- [IMPL_004_v2.0](Phase-2/IMPL_004_v2.0.md)
- [IMPL_005_v2.0](Phase-2/IMPL_005_v2.0.md)
- [IMPL_006_v2.0](Phase-2/IMPL_006_v2.0.md)
- [IMPL_007_v2.0](Phase-2/IMPL_007_v2.0.md)
- [IMPL_008_v2.0](Phase-2/IMPL_008_v2.0.md)
- [IMPL_009_v2.0](Phase-2/IMPL_009_v2.0.md)
- [IMPL_010_v2.0](Phase-2/IMPL_010_v2.0.md)
- [IMPL_011_v2.0](Phase-2/IMPL_011_v2.0.md)
- [IMPL_012_v2.0](Phase-2/IMPL_012_v2.0.md)
- [IMPL_013_v2.0: Strategy Builder](Phase-2/IMPL_013_v2.0.md)
- [IMPL_014_v2.0: Home Dashboard](Phase-2/IMPL_014_v2.0.md)

## PHASE 3 READINESS
Phase 3 (CODE) begins only after the following conditions are met:
- [x] Documentation audit complete
- [x] SPEC/IMPL consistency verified
- [x] Product scope verified
- [x] UI/UX reference requirements captured
- [x] Architecture verified
- [x] Dependency graph verified
- [x] Master index verified
- [x] Agent instructions verified
- [x] Security requirements verified
- [x] Testing strategy verified
- [x] Open architectural decisions identified
- [x] No critical documentation conflicts remain
*(Only BLOCKING decisions prevent Phase 3 implementation. Non-blocking/Deferred do not).*

## PUBLICATION STATUS
This repository is published as the authoritative Architecture package for MarketPulse Pro and is intentionally scoped to architecture, governance, and evidence artifacts only. The branch represents the corrected Architecture-only source of truth and is kept separate from unrelated project code and legacy repo roots.
