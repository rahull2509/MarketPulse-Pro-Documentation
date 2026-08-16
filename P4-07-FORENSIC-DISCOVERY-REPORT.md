# P4-07 FORENSIC DISCOVERY REPORT

## Objective
A strict read-only forensic inspection was conducted to determine the architectural readiness and governance state for P4-07 (Broker Integration Layer).

## 1. Governance Discovery
P4-07 is defined in `PHASE4_ARCHITECTURE_ROADMAP.md` as:
> "Broker Integration Layer (Facade) - Implement the internal unified Broker Interface and generic Orders DB schema."

The foundational governance is `ADR-001-Multi-Broker-Architecture.md` (DEC-ARCH-004B), which outlines a conceptual abstraction layer above specific providers (Zerodha, Angel One, ICICI Direct). 

**Classification of Requirements:**
- **Supported Brokers:** APPROVED BY EXISTING GOVERNANCE (DEC-ARCH-004A: Zerodha, Angel One, ICICI Direct. Upstox is explicitly REJECTED).
- **Facade Concept:** APPROVED BY EXISTING GOVERNANCE (ADR-001).
- **Authentication Separation:** APPROVED BY EXISTING GOVERNANCE (Application Auth != Broker Auth).
- **Generic Orders DB Schema:** GOVERNANCE GAP (Only abstract entity names like `BrokerOrder` are listed; actual column types, indexes, and relations are missing).
- **Token Encryption At Rest:** GOVERNANCE GAP (ADR-001 requires encryption at rest, but the encryption mechanism/provider is not governed).
- **Token Lifecycle/Refresh:** GOVERNANCE GAP (No rules defined for refresh token polling or expiry handling).
- **Broker Provider API Specs (Zerodha, Angel One, ICICI):** GOVERNANCE GAP (Explicitly marked as PENDING BLOCKING via DEC-ARCH-004C/D/E).

## 2. Repository Forensic Inspection
- **Backend:** `interfaces.go` under `internal/modules/broker` defines a preliminary `BrokerAdapter` interface. It correctly includes comments stating that concrete implementations are blocked pending DEC-ARCH-004C/D/E. No broker-specific code, credentials, or SDKs exist.
- **Frontend:** `BrokerModal` correctly lists Zerodha and Angel One, disables Upstox, and fires alert placeholders for API pending integration.
- **Architecture:** `ADR-001` exists but explicitly labels all provider capabilities (Auth, Orders, Market Data) as `UNKNOWN / NOT YET SPECIFIED`.

## 3. P4-06 Boundary Audit
P4-07 is currently **NOT PERMITTED** to replace or supplement internal market data, underlying spot, or option ticks. ADR-001 mentions a potential realtime normalization pipeline, but its implementation is explicitly deferred due to unknown provider capabilities. Data ownership between the internal P4-04 pipeline and external brokers remains ambiguous.

## 4. Broker Boundary Discovery
For Zerodha, Angel One, and ICICI Direct:
- **Authentication Mechanism:** UNKNOWN
- **Token Storage Requirements:** Encrypted at rest (mechanism UNKNOWN)
- **Market Data APIs:** UNKNOWN
- **Rate Limits:** UNKNOWN
- **Error/Reconnection Semantics:** UNKNOWN

## 5. Domain Boundary
The repository already conceptually introduces `BrokerAdapter` in `interfaces.go`. It does not yet include `InstrumentResolver` or `OrderProvider` interfaces. Given the gaps in provider specs, these interfaces cannot be finalized.

## 6. Data Ownership
- **Live Market Ticks / Greeks:** P4-04 and P4-06 (Internal Pipeline).
- **Broker Account State / Orders:** P4-07 Facade (Schema missing).
- **Conflict:** There is a severe architectural conflict regarding symbol mapping. A broker's `symbol` or `token` might not match the internal P4-04 `symbol`. No mapping resolution layer is governed.

## 7. Security Audit
- **Encryption at rest:** Required by ADR-001, but completely ungoverned (No AES key management, KMS, or Vault specified).
- **Token rotation/expiry:** GOVERNANCE GAP.
- **OAuth state/callback validation:** GOVERNANCE GAP.

## 8. Realtime Audit
`EVENT_CATALOG.md` lists `order.execution` and `order.status` as PROPOSED. No concrete Redis channels or payloads are approved. P4-07 is strictly blocked from introducing real-time events until these payloads are governed.

## 9. Failure & Isolation
ADR-001 requires "Circuit Breakers", "Strict Timeouts", and "Provider-specific Retry Policies", but states that exact numeric values and error normalization mappings are `[DECISION REQUIRED]`.

## 10. Implementation Readiness
P4-07 requires a facade and a generic database schema. However, because the generic schema cannot be designed without knowing the lowest common denominator of the three underlying broker APIs (which are pending/unknown), and because the security requirements for token storage are incomplete, P4-07 is entirely blocked.

**FINAL NO-CODE VERIFICATION:**
- Application code modified: NO
- Frontend code modified: NO
- Database migrations modified: NO
- Routes modified: NO
- REST APIs implemented: NO
- Broker integration implemented: NO

## FINAL VERDICT
**P4-07 DISCOVERY = COMPLETE**
**P4-07 IMPLEMENTATION = NOT STARTED**
**P4-07 READINESS = BLOCKED**
**ARCHITECTURE CONFIDENCE = HIGH**
