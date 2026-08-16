# P4-08 FORENSIC DISCOVERY REPORT

**Date:** 2026-08-10

## 1. P4-08 Objective and Boundaries
**Objective:** Phase 4-08 ("Zerodha/AngelOne Adapters") mandates the implementation of concrete provider APIs behind the P4-07 unified Broker Facade.
**Boundaries:**
- Zerodha has already been fully implemented during P4-07.
- P4-08 is therefore scoped exclusively to the remaining governed providers: **Angel One** and **ICICI Direct**.
- Scope includes Auth, GetOrders, GetPositions, GetHoldings, PlaceOrder, ModifyOrder, and CancelOrder operations.

## 2. Dependencies on P4-01 through P4-07
- P4-08 inherently depends on **P4-07 (Broker Integration Layer)** which defines the `BrokerAdapter` interface, generic models, Circuit Breaker mechanics, and AES-256-GCM token encryption requirements.

## 3. Actual Repository Implementation
- The `adapter.go` files for both `angelone` and `icicidirect` currently exist but consist purely of stub methods returning `ErrUnsupportedCapability`.
- No parsing, HTTP invocation, or authentication mechanisms have been written for these providers.

## 4. Missing Architectural Decisions
- `DECISION_REGISTER.md` explicitly lists:
  - **DEC-ARCH-004D** (Angel One API Integration Specification): PENDING (BLOCKING)
  - **DEC-ARCH-004E** (ICICI Direct API Integration Specification): PENDING (BLOCKING)
- **GOVERNANCE GAP:** There are currently no approved specifications detailing the technical contracts, SDK requirements, authentication flow mechanics (e.g. standard OAuth vs. proprietary publisher login), or payload definitions for Angel One or ICICI Direct.

## 5. Architectural Inheritance (from P4-07)
- **Database Schema:** Will utilize the existing `broker_sessions` and `broker_orders` tables with `encryption_version` handling.
- **API Boundaries:** Must integrate seamlessly behind `BrokerManager` without exposing provider-specific logic to the REST API.
- **Security:** Must inherit the strict OAuth state/CSRF validations (or equivalent provider-specific secure flows) and token AES-GCM encryption.
- **Data Ownership:** Provider adapters own their respective orders but do not own normalized application events or market data.
- **Failure Semantics:** Must integrate safely with the 1-minute, >50% failure threshold Circuit Breaker.

## 6. Contradictions
- There are no contradictions between repository reality and governance; the repository accurately reflects the `PENDING` governance state by safely returning `ErrUnsupportedCapability`.

## 7. Readiness Classification
P4-08 is **REQUIRES GOVERNANCE** and is technically **BLOCKED** until DEC-ARCH-004D and DEC-ARCH-004E are formally ratified.
