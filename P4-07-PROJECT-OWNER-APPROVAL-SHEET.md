# P4-07 Project Owner Approval Sheet

This document groups the 26 implementation-critical decisions for P4-07 Broker Integration Layer. 
It requires explicit Project Owner sign-off before any implementation is authorized.

## A. Broker Provider Capability

| ID | Decision | Proposed Value | Evidence | Current Status | Project Owner Decision |
|----|----------|----------------|----------|----------------|------------------------|
| 1 | Broker Selection Scope | Zerodha, Angel One, ICICI Direct | DEC-ARCH-004A | APPROVED BY EXISTING GOVERNANCE | APPROVED |
| 6 | Zerodha API Specification | EXTERNAL PROVIDER VERIFICATION REQUIRED | N/A | APPROVED | APPROVED |
| 7 | Angel One API Specification | EXTERNAL PROVIDER VERIFICATION REQUIRED | N/A | APPROVED | APPROVED |
| 8 | ICICI Direct API Specification | EXTERNAL PROVIDER VERIFICATION REQUIRED | N/A | APPROVED | APPROVED |

## B. Authentication & Token Security

| ID | Decision | Proposed Value | Evidence | Current Status | Project Owner Decision |
|----|----------|----------------|----------|----------------|------------------------|
| 3 | Application Auth vs Broker Auth | Strictly Separated | ADR-001 | APPROVED BY EXISTING GOVERNANCE | APPROVED |
| 9 | Authentication Flow | OAuth 2.0 / Token Redirection | P4-07-BROKER-AUTH-DECISION | APPROVED | APPROVED |
| 11 | Token Storage Strategy | PostgreSQL `broker_sessions` | P4-07-BROKER-AUTH-DECISION | APPROVED | APPROVED |
| 14 | Token Expiry Handling | Provider-specific expiry verified from official docs | N/A | APPROVED | APPROVED |
| 15 | Token Refresh Strategy | Synchronous refresh on 401 | P4-07-BROKER-AUTH-DECISION | APPROVED | APPROVED |

## C. Instrument & Market Data Ownership

| ID | Decision | Proposed Value | Evidence | Current Status | Project Owner Decision |
|----|----------|----------------|----------|----------------|------------------------|
| 4 | Internal vs Broker Market Data | P4-04 remains Authoritative | P4-07-DATA-OWNERSHIP-DECISION | APPROVED | APPROVED |
| 5 | Internal vs Broker Greeks | P4-06 remains Authoritative | P4-07-DATA-OWNERSHIP-DECISION | APPROVED | APPROVED |
| 19 | Instrument Mapping Ownership | Facade Normalization Layer | P4-07-INSTRUMENT-MAPPING-DECISION | APPROVED | APPROVED |
| 20 | Symbol Mapping Cache | Daily Provider Instrument fetch | P4-07-INSTRUMENT-MAPPING-DECISION | APPROVED | APPROVED |

## D. Database & Order Model

| ID | Decision | Proposed Value | Evidence | Current Status | Project Owner Decision |
|----|----------|----------------|----------|----------------|------------------------|
| 17 | BrokerSession Schema | `user_id`, `access_token`, `refresh_token` | P4-07-DATA-MODEL-DECISION | APPROVED | APPROVED |
| 18 | BrokerOrder Schema | Generic unified fields | P4-07-DATA-MODEL-DECISION | APPROVED | APPROVED |

## E. REST API

| ID | Decision | Proposed Value | Evidence | Current Status | Project Owner Decision |
|----|----------|----------------|----------|----------------|------------------------|
| 2 | Facade Architecture | `BrokerAdapter` layer | ADR-001 | APPROVED BY EXISTING GOVERNANCE | APPROVED |
| 23 | REST Facade Contracts | `/api/v1/broker/*` | P4-07-API-CONTRACT-DECISION | APPROVED | APPROVED |

## F. Redis/WebSocket Events

| ID | Decision | Proposed Value | Evidence | Current Status | Project Owner Decision |
|----|----------|----------------|----------|----------------|------------------------|
| 21 | `order.execution` Event | Redis Pub/Sub Payload | P4-07-REALTIME-CONTRACT-DECISION | APPROVED | APPROVED |
| 22 | `order.status` Event | Redis Pub/Sub Payload | P4-07-REALTIME-CONTRACT-DECISION | APPROVED | APPROVED |

## G. Failure & Resilience

| ID | Decision | Proposed Value | Evidence | Current Status | Project Owner Decision |
|----|----------|----------------|----------|----------------|------------------------|
| 24 | Rate Limit Policies | Provider-specific limits MUST be obtained from official docs | P4-07-FAILURE-SEMANTICS-DECISION | APPROVED | APPROVED |
| 25 | Circuit Breaker Thresholds | >50% failure rate | P4-07-FAILURE-SEMANTICS-DECISION | APPROVED | APPROVED |
| 26 | Error Normalization Taxonomy | Map provider HTTP -> internal errors | P4-07-FAILURE-SEMANTICS-DECISION | APPROVED | APPROVED |

## H. Security & Isolation

| ID | Decision | Proposed Value | Evidence | Current Status | Project Owner Decision |
|----|----------|----------------|----------|----------------|------------------------|
| 10 | OAuth CSRF / State Protection | Signed JWT in `state` param | P4-07-SECURITY-DECISION | APPROVED | APPROVED |
| 12 | Token Encryption at Rest | AES-256-GCM authenticated encryption | P4-07-SECURITY-DECISION | APPROVED | APPROVED |
| 13 | Master Key Management | Environment Variable | P4-07-SECURITY-DECISION | APPROVED | APPROVED |
| 16 | Credential Logging | Strict Redaction Middleware | P4-07-SECURITY-DECISION | APPROVED | APPROVED |

## I. Observability & Testing

*(No distinct implementation-critical decisions currently tracked in the top 26 for this boundary. Follows standard ADR-001 mocking and audit logging requirements.)*

---

## PROJECT OWNER APPROVAL SUMMARY

Approved:
26

Pending:
0

Unknown:
0

Deferred:
0

Rejected:
0

P4-07 GOVERNANCE = APPROVED
