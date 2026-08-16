# Documentation Changelog

This log records all structural and contextual modifications made to the MarketPulse Pro documentation to ensure traceability.

| Date | File | Section | Old State | New State | Reason | Source | Impact | Status |
|---|---|---|---|---|---|---|---|---|
| 2026-08-08 | `DECISION_REGISTER.md` | All | Bare decision IDs | Full evidence-backed closure for DEC-ARCH-001,002,003,004 | Resolve architectural blockers for Phase 3 | Volume 5, 6, IMPL 007, SPEC 010 | Unblocks Phase 3 conditionally | RESOLVED |
| 2026-08-08 | `SPEC_011.md` | N/A | Missing | Created Strategy Builder SPEC | Close functional traceability gap | Volume 2, Sensibull | Enables IMPL_013 | CREATED |
| 2026-08-08 | `IMPL_013_v2.0.md` | N/A | Missing | Created Strategy Builder IMPL | Close technical traceability gap | SPEC_011 | Unblocks Code Phase | CREATED |
| 2026-08-08 | `SPEC_012.md` | N/A | Missing | Created Home Dashboard SPEC | Close functional traceability gap | Volume 2 | Enables IMPL_014 | CREATED |
| 2026-08-08 | `IMPL_014_v2.0.md` | N/A | Missing | Created Home Dashboard IMPL | Close technical traceability gap | SPEC_012 | Unblocks Code Phase | CREATED |
| 2026-08-08 | `UI_UX_REQUIREMENT_MATRIX.md` | Strategy/Home rows | PENDING / REFERENCE-DERIVED | APPROVED | Matched statuses to new explicitly approved SPECs | SPEC_011, SPEC_012 | High | TRACEABLE |
| 2026-08-08 | `FEATURE_TRACEABILITY.md` | Option Chain / Strategy / Home | NOT YET DEFINED | `OptionChain.spec.ts` mapped, `TRACEABLE` | Establish E2E traceability for tests | Prompt instruction | High | TRACEABLE |
| 2026-08-08 | `DEPENDENCY_GRAPH.md` | Tech Dependencies | Missing | Added Option Chain, Broker, Strategy, Home dependencies | Document explicit module-to-module data flows | Architecture Audit | High | VALIDATED |
| 2026-08-08 | `MASTER_INDEX.md` | Phase 1 & 2 Lists | Missing | Added explicit links to all 12 SPECs and 14 IMPLs | Correct index consistency | Directory audit | High | VALIDATED |
| 2026-08-09 | `DECISION_REGISTER.md` | DEC-ARCH-004 | Upstox Approved | DEC-ARCH-004A (UI: Zerodha, Angel One, ICICI - Approved) & DEC-ARCH-004B (Backend: PENDING BLOCKING) | Disambiguate UI requirement from backend API approval | Sensibull UI Ref | Blocks Phase 3 CODE conditionally | SPLIT & RECLASSIFIED |
| 2026-08-09 | `SPEC_012.md` & `IMPL_014_v2.0.md` | Broker / API | Upstox hardcoded | Abstracted to Broker Integration (Zerodha/Angel One/ICICI) | Align with DEC-ARCH-004A | User Directive | High | CORRECTED |
| 2026-08-09 | Phase-1 & Phase-2 Specs | Upstox sections | Original | Prepended `[LEGACY MARKETPULSE CONFLICT]` warnings | Preserve historical artifacts while invalidating conflicting architecture | User Directive | High | FLAGGED |
| 2026-08-09 | `DEPENDENCY_GRAPH.md` | Broker Node | Upstox Adapter | Broker Integration Layer -> Zerodha/Angel One/ICICI | Separate abstract layer from specific provider adapters | User Directive | High | CORRECTED |
| 2026-08-09 | `UI_UX_REQUIREMENT_MATRIX.md` | Login Rows | Upstox / Missing | Explicitly mapped Zerodha, Angel One, ICICI as REFERENCE-DERIVED | Align UI constraints with Sensibull | User Directive | High | CORRECTED |
| 2026-08-09 | `FEATURE_TRACEABILITY.md` | Broker Login | Missing | Mapped Login UI as TRACEABLE, Backend as DECISION REQUIRED | Maintain distinction between UI and backend integration | User Directive | High | TRACEABLE |
| 2026-08-09 | `ADR-001-Multi-Broker-Architecture.md` | N/A | Missing | Created formal architecture for Broker Integration Layer | Resolve DEC-ARCH-004B abstract architecture | User Directive | Unblocks Core Phase 3 | CREATED |
| 2026-08-09 | `DECISION_REGISTER.md` | DEC-ARCH-004B | PENDING | RESOLVED (Core Arch), but DEC-ARCH-004C/D/E created as PENDING | Separation of architecture design from provider implementation | ADR-001 | Unblocks Core Phase 3 | RESOLVED |
| 2026-08-09 | `AGENTS.md` | Governance | N/A | Added strict rules for AI regarding DECISION_REGISTER check | Enforce governance constraints on code generation | User Directive | High | CREATED |
| 2026-08-09 | `00_Enterprise_AI_Operating_Manual.md` | Decision States | N/A | Defined precise semantics for RESOLVED, PENDING, BLOCKING, etc. | Align agent behavior with architectural tracking | User Directive | High | UPDATED |
| 2026-08-09 | `ARCHITECTURE_MAP.md` | Core & Infra | Undefined/Legacy | Added Next.js, Postgres, Redis, WebSocket. Marked Storage/Data Format TBD. | Align with core enterprise stack and pending storage decisions | User Directive | High | CORRECTED |
| 2026-08-09 | `FEATURE_TRACEABILITY.md` | Broker Login | Flattened | Explicit layering: UI (Approved), Arch (Approved), API (Pending), Impl (Not yet implemented) | Ensure UI approval doesn't fabricate API readiness | User Directive | High | REFINED |
| 2026-08-09 | `UI_UX_REQUIREMENT_MATRIX.md` | Broker Rows | Mixed API details | Explicitly mapped UI elements for Zerodha/Angel/ICICI ONLY, stripped backend columns | Prevent UI mapping from implying backend API readiness | User Directive | High | REFINED |
| 2026-08-09 | `DEPENDENCY_GRAPH.md` | Broker Node | Hardcoded status | Marked Adapters strictly as PENDING | Ensure abstraction boundaries are clear | User Directive | High | REFINED |
