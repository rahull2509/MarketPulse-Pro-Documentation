# Technology Conflict Register

This register documents all explicitly detected architectural and technology conflicts within MarketPulse Pro, ensuring no conflict is silently overwritten without formal governance.

## TECH-CONFLICT-001
**Technology**: AWS S3 / Cloud Object Storage
**Artifact A**: `SPEC_010`, `IMPL_005` (S3 integration is explicitly referenced and defined)
**Artifact B**: `DECISION_REGISTER.md` (DEC-ARCH-005 marks Cloud Object Storage as PENDING NON-BLOCKING)
**Conflict Description**: Historical Phase-1 and Phase-2 documents define S3, but DEC-ARCH-005 holds the active architectural decision in a PENDING state.
**Resolution Status**: DEC-ARCH-005 governs the current active status, meaning new adoption is paused. The historical references in SPEC/IMPL documents are preserved and remain valid as historical blueprints, but do not override the central Decision Register's PENDING state for Phase 3 implementation.

## TECH-CONFLICT-002
**Technology**: Parquet
**Artifact A**: `SPEC_005` (Parquet defined as standard analytical format)
**Artifact B**: `DECISION_REGISTER.md` (DEC-ARCH-006 marks Data Pipeline Format as PENDING DEFERRED)
**Conflict Description**: SPEC-005 establishes explicit Parquet requirements, but the Decision Register has deferred the pipeline format decision.
**Resolution Status**: The historical SPEC-005 requirements are preserved. However, DEC-ARCH-006 defers *new* adoption and implementation for Phase 3. Parquet is valid for its historically bounded workload if reactivated, but remains DEFERRED in current execution scopes.

## TECH-CONFLICT-003
**Technology**: Upstox
**Artifact A**: `SPEC_006`, `SPEC_010` (Upstox integration defined)
**Artifact B**: `DECISION_REGISTER.md` (UI Scope strictly limited to Zerodha, Angel One, ICICI Direct)
**Conflict Description**: Upstox appears heavily in legacy specs but was explicitly rejected by the updated broker UI scope.
**Resolution Status**: RESOLVED. Upstox is explicitly REJECTED/LEGACY. Old specifications are preserved for history but marked as conflicts.

## TECH-CONFLICT-004
**Technology**: Kafka / NATS vs Redis Pub/Sub
**Artifact A**: `Volume 3` (Mentions Kafka/NATS topics conceptually)
**Artifact B**: `SPEC_007`, `IMPL_005` (Redis Pub/Sub explicitly integrated and defined for message distribution)
**Conflict Description**: High-level enterprise architecture conceptually references Kafka/NATS for event bus, but detailed specifications explicitly require Redis Pub/Sub.
**Resolution Status**: RESOLVED. Kafka and NATS are merely conceptual references (NOT SELECTED). Redis Pub/Sub is the explicit, active, approved realtime event distribution mechanism.

## TECH-CONFLICT-005
**Technology**: EC2 / Docker / Nginx / systemd / GitHub Actions
**Artifact A**: `IMPL_012` (Defines these technologies for deployment)
**Artifact B**: Global Infrastructure Decision (Missing / Pending)
**Conflict Description**: While IMPL-012 mentions these technologies as deployment implementations, they are deployment architecture details that lack a central Decision Register entry governing the core infrastructure.
**Resolution Status**: REQUIRES GOVERNANCE DECISION. Docker and Nginx are PROVISIONALLY APPROVED for containerization, but EC2, GitHub Actions, and systemd remain PENDING infrastructure decisions. They are not unconditionally GREEN for Phase 3.
