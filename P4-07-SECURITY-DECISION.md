# P4-07 Security Governance Decision

This document defines the approved security boundaries and controls for the Broker Integration Facade.

| Domain | Proposed Security Policy | Governance Status |
|--------|--------------------------|-------------------|
| **Credential Encryption** | Access and Refresh tokens must be encrypted at rest using AES-256-GCM. | APPROVED |
| **Key Management** | Master encryption key provided via secure environment variable (`BROKER_ENCRYPTION_KEY`); not stored in DB. | APPROVED |
| **Token Isolation** | Tokens are strictly bound to `user_id`. No cross-user token sharing is permitted. | APPROVED |
| **OAuth State (CSRF)** | OAuth `state` parameter must contain a signed JWT or secure random nonce tied to the user's application session to prevent CSRF login attacks. | APPROVED |
| **Callback Validation** | The `state` parameter must be validated before exchanging the authorization code. | APPROVED |
| **Logging Redaction** | All structured logging must use strict redaction middlewares ensuring `access_token`, `refresh_token`, and `password` never reach stdout or log files. | APPROVED |
| **Secret Rotation** | The application must support rotating the master encryption key (requires key versioning in the DB schema). | APPROVED |
| **Audit Trail** | All broker connections, disconnections, and placed orders must emit an immutable audit log entry. | APPROVED |
| **Authorization Boundaries**| Only authenticated application users can interact with the broker API. Users can only act upon their own linked broker account. | APPROVED |
