# ADR-001: Multi-Broker Integration Architecture

## 1. Context & Decision
The MarketPulse Pro frontend user experience (based on the Sensibull UX reference) strictly requires supporting **Zerodha, Angel One, and ICICI Direct** as broker login options. Previously, historical artifacts referenced Upstox as the primary broker. This ADR resolves DEC-ARCH-004B by establishing a provider-agnostic Broker Integration Layer to abstract away the specific broker API implementations.

**Decision**: MarketPulse Pro will use a common `Broker Integration Layer` with separate provider adapters for Zerodha, Angel One, and ICICI Direct. Provider-specific API implementation details remain subject to their corresponding technical specifications and are explicitly excluded from this core architecture.

## 2. Conceptual Architecture Hierarchy

```text
MarketPulse Pro
        ↓
Broker Integration Layer
        ↓
Common Broker Contract
        ↓
┌──────────────┬──────────────┬──────────────┐
│              │              │              │
Zerodha       Angel One     ICICI Direct     │
Adapter       Adapter       Adapter          │
│              │              │              │
└──────────────┴──────────────┴──────────────┘
```

The Common Broker Contract defines *only* capabilities required by MarketPulse Pro. Provider-specific behavior belongs exclusively inside each adapter.

## 3. Capability Matrix
The following matrix defines expected broker capabilities. Technical evidence for the exact APIs is currently unavailable; therefore, exact provider support is marked `UNKNOWN` or `NOT YET SPECIFIED`.

| Capability | Zerodha | Angel One | ICICI Direct | Status / Evidence |
|------------|---------|-----------|--------------|-------------------|
| Authentication | UNKNOWN | UNKNOWN | UNKNOWN | NOT YET SPECIFIED |
| Profile | UNKNOWN | UNKNOWN | UNKNOWN | NOT YET SPECIFIED |
| Funds | UNKNOWN | UNKNOWN | UNKNOWN | NOT YET SPECIFIED |
| Positions | UNKNOWN | UNKNOWN | UNKNOWN | NOT YET SPECIFIED |
| Orders | UNKNOWN | UNKNOWN | UNKNOWN | NOT YET SPECIFIED |
| Order Placement | UNKNOWN | UNKNOWN | UNKNOWN | NOT YET SPECIFIED |
| Order Modification | UNKNOWN | UNKNOWN | UNKNOWN | NOT YET SPECIFIED |
| Order Cancellation | UNKNOWN | UNKNOWN | UNKNOWN | NOT YET SPECIFIED |
| Market Data (REST) | UNKNOWN | UNKNOWN | UNKNOWN | NOT YET SPECIFIED |
| Realtime Data (Tick) | UNKNOWN | UNKNOWN | UNKNOWN | NOT YET SPECIFIED |

## 4. Authentication Architecture
Application authentication is strictly segregated from broker authentication.

```text
User
 ↓
MarketPulse Pro Authentication (Application Session)
 ↓
Broker Connection
 ↓
Broker-Specific Authentication (OAuth/Token)
 ↓
Secure Credential/Token Handling
 ↓
Broker Session
```

**Rule**: Application Identity must never leak into or merge with the Broker Connection state.

## 5. Security Architecture Boundary
- **No Hardcoded Secrets**: Broker credentials (API keys, secrets) must never be stored in source code.
- **Log Sanitization**: Credentials and access tokens must never be logged.
- **Storage**: Secure credential storage with at-rest encryption is an architectural requirement.
- **Session Lifecycle**: The architecture must handle token expiration, revocation, and logout flows explicitly.
- **Least Privilege**: Adapters must only request scopes necessary for the capabilities outlined in the Common Broker Contract.
- **Auditability**: All state-modifying actions via the broker layer require secure audit logging.

## 6. Failure Isolation Architecture
One broker failing must NOT bring down the entire Broker Integration Layer or the core application.
- `Zerodha failure ≠ Angel One failure ≠ ICICI Direct failure`.
- **Architectural Requirements**: 
  - Circuit Breaker mechanisms
  - Provider-specific Retry Policies
  - Strict Timeouts
  - Fallback / Degraded State handling
  - Error Normalization (translating provider-specific HTTP errors to a standard internal error taxonomy).

*(Note: Exact numeric timeout/retry values are `[DECISION REQUIRED]` during provider implementation).*

## 7. Data Model Boundary
Conceptual entities (not finalized database schemas):
- `BrokerAccount`: User's linkage to a specific broker.
- `BrokerConnection`: Active linkage state.
- `BrokerSession`: Temporal authentication state.
- `BrokerCredentialReference`: Abstracted reference to secure storage.
- `BrokerCapability`: Registered support flags for a specific adapter.

## 8. API Boundary
Provider-neutral backend APIs conceptually required:
- `Broker Connection`
- `Broker Account`
- `Broker Profile`
- `Broker Funds`
- `Broker Positions`
- `Broker Orders`

## 9. Realtime Architecture Boundary
If a broker supports realtime streaming, the architecture uses the following normalization pipeline to integrate with the approved MarketPulse WebSocket layer:

```text
Broker-specific realtime source
        ↓
Broker Adapter
        ↓
Normalized Event (MarketPulse Standard Tick)
        ↓
MarketPulse Realtime Layer
        ↓
WebSocket (DEC-ARCH-003)
        ↓
Frontend
```

*(Note: It is `UNKNOWN` if all three brokers provide identical realtime capabilities. Adapters must handle capability degradation gracefully).*

## 10. Testing Architecture Requirements
- **Broker Abstraction**: Must test the Common Broker Contract independently of providers.
- **Adapters**: Require mock testing of provider-specific APIs (no live credentials in CI).
- **Authentication**: E2E testing of the two-tier auth state.
- **Failure Isolation**: Chaos testing/fault injection for circuit breakers and timeouts.
- **Realtime**: Testing event normalization correctness.
