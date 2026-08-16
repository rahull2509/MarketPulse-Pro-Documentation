# MarketPulse Pro Enterprise Blueprint

## Project Context & Identity

This workspace contains the complete Enterprise Blueprint for the **MarketPulse Pro** platform. 
MarketPulse Pro is an enterprise-grade financial market analysis platform and is a SEPARATE PROJECT from any older "MarketPulse" iterations. 
These documents are the absolute governing specification for every engineering task. 

## Project Boundaries

- DO NOT import, reuse, assume, infer, or silently copy architecture, business logic, data pipelines, infrastructure, algorithms, APIs, database schemas, or implementation details from any older "MarketPulse" project.
- Any information from an older MarketPulse project is OUT OF SCOPE unless it is explicitly present in the current MarketPulse Pro documentation.
- Do not use historical project memory as a source of truth. 

## Document Hierarchy & Source-of-Truth

Follow this priority order when resolving requirements:
1. **LEVEL 1** — Explicit current MarketPulse Pro requirements
2. **LEVEL 2** — Approved Phase-1 SPEC documents
3. **LEVEL 3** — Approved Phase-2 IMPL documents
4. **LEVEL 4** — Current MarketPulse Pro product/UI requirements
5. **LEVEL 5** — User-provided Sensibull reference screenshots and extracted observations
6. **LEVEL 6** — Existing master documentation
7. **LEVEL 7** — General engineering knowledge

*General engineering knowledge must NEVER override an explicit project requirement. If two sources conflict, do not silently choose one. Record the conflict, and if unresolved, mark as `[DECISION REQUIRED]`.*

### Volume Hierarchy
1. 00_Enterprise_AI_Operating_Manual.md
2. Volume_1_Business_Foundation.md
3. Volume_2_Requirements.md
4. Volume_3_Enterprise_Architecture.md
5. Volume_4_Engineering_Delivery.md
6. Volume_5_Implementation_Architecture.md
7. Volume_6_Engineering_Governance.md

## Operating Rules

### Explicit Preparation Rule
**Before implementing a feature, the agent MUST identify the applicable SPEC, IMPL, architecture document, UI/UX requirement, API contract, and dependency requirements.**

### Phase Rules
- **Phase 1 (SPEC)**: Requirements and specification definition.
- **Phase 2 (IMPL)**: Technical blueprints and implementation architecture.
- **Phase 3 (CODE)**: Implementation phase. Do not begin Phase 3 without explicit clearance that Phase 3 Readiness is met.

### Technical & Governance Rules
- **Coding Rules**: Write enterprise-grade, clean, testable code conforming to approved IMPL documents.
- **Architecture Rules**: Follow `ARCHITECTURE_MAP.md` and `Volume_5_Implementation_Architecture.md`. If a decision is unapproved, explicitly mark `[DECISION REQUIRED]`.
- **Security Rules**: No hardcoded secrets. Follow secure coding practices defined in governance.
- **Testing Rules**: Comprehensive unit and integration testing is mandatory. Do not bypass tests.
- **UI/UX Rules**: Follow references strictly as defined in `UI_UX_REQUIREMENT_MATRIX.md`. Sensibull references dictate structure/behavior but do not permit copying proprietary backend/algorithms.
- **Database Rules**: No schema modifications without proper migration files and governance approval.
- **API Rules**: Respect API contracts. Do not introduce breaking changes without formal deprecation.
- **Realtime Rules**: Follow specific WebSocket/realtime specs where strictly required.
- **Observability Rules**: Include metrics, logging, and tracing where specified.
- **Dependency Rules**: Follow `DEPENDENCY_GRAPH.md`. Do not introduce arbitrary dependencies.
- **Git Rules**: No force-pushing to protected branches. Respect branching strategies.
- **Review Rules**: All features require a formal AI and human review checkpoint before merging.

### Prohibited Behaviors
An AI agent is strictly prohibited from:
- Inventing undocumented features or requirements.
- Changing approved architecture without authorization.
- Mixing old MarketPulse information into the Pro project.
- Deleting requirements to make implementation easier.
- Bypassing tests or security standards.
- Hardcoding secrets.
- Introducing arbitrary or unapproved dependencies.
- Changing UI behavior without explicit specification.
- Modifying database schemas without formal migration.
- Breaking API contracts.
- Silently changing business logic.

### Definition of Done
A task is done only when:
1. It implements the exact SPEC/IMPL without unauthorized scope expansion.
2. It passes all testing rules.
3. UI/UX matches the reference matrix faithfully.
4. No legacy contamination is introduced.
5. All documentation has been updated to reflect the new state.
6. A review checkpoint has been passed.

If additional information is required, explicitly state what is missing instead of making assumptions.
## Mandatory Architecture & Governance Checks
Before implementing any feature, the agent MUST check:
1. `DECISION_REGISTER.md`
2. applicable SPEC
3. applicable IMPL
4. applicable ADR
5. architecture documents
6. UI/UX requirements
7. dependency requirements

The agent MUST NOT:
- use S3 by default
- use Parquet by default
- introduce Upstox
- assume broker APIs are implemented
- assume Zerodha/Angel One/ICICI Direct APIs are approved
- invent unresolved architecture decisions

The agent must stop and report:
`[DECISION REQUIRED]`
when a blocking decision is required.
