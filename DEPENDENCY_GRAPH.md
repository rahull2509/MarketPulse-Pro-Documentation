# Dependency Graph

This graph maps the formalized dependencies across all phases, domains, and external services for MarketPulse Pro.

## Phase Dependencies

| Source | Target | Type | Reason | Status |
|---|---|---|---|---|
| Phase-2 IMPL | Phase-1 SPEC | Governance | Technical blueprints must fulfill functional specifications. | APPROVED |
| Phase-3 CODE | Phase-2 IMPL | Governance | Code implementation must strictly adhere to the technical blueprint. | PENDING |

## Product & Domain Dependencies

| Source | Target | Type | Reason | Status |
|---|---|---|---|---|
| Product Modules | Frontend Domains | Implementation | Product features map to frontend user experiences. | PENDING |
| Frontend Domains | Backend Domains | Data Flow | UI components consume REST/WebSocket APIs provided by backend domains. | PENDING |
| Backend Domains | Infrastructure | Execution | Backend services require compute and storage provisioned by infrastructure. | PENDING |
| Testing | All Domains | Quality Assurance | All implemented modules require testing coverage before release. | PENDING |

## Specific Technical Dependencies

| Source | Target | Type | Reason | Status |
|---|---|---|---|---|
| Option Chain Domain | Market Data Domain | Data Flow | Option chains require real-time pricing and OI data. | PROPOSED |
| Market Data Domain | Broker Integration Layer | External API | Market Data Domain depends on upstream broker feeds for ticks. | PROPOSED |
| Broker Integration Layer | Zerodha Adapter | External API | Supported broker based on Sensibull UX reference. | PENDING |
| Broker Integration Layer | Angel One Adapter | External API | Supported broker based on Sensibull UX reference. | PENDING |
| Broker Integration Layer | ICICI Direct Adapter | External API | Supported broker based on Sensibull UX reference. | PENDING |
| Strategy Builder (IMPL_013) | Option Chain Domain | Internal API | Strategy Builder requires strikes and Greeks from the Option Chain. | APPROVED |
| Strategy Builder (IMPL_013) | User Domain | Internal API | Strategy Builder requires User Domain to save drafts securely. | APPROVED |
| Home Dashboard (IMPL_014) | Market Data Domain (Indices) | Internal API | Home Dashboard relies on Market Data Domain for index ticks. | APPROVED |
| Home Dashboard (IMPL_014) | Broker Integration Layer | Internal API | Home Dashboard queries broker connection token validity. | APPROVED |
