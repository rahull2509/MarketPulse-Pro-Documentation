# P4-07 Broker Authentication Decision

This document defines the approved governance policies for handling broker authentication, session lifecycles, and credential storage within the Provider Facade.

## OAuth Orchestration Boundary
The architecture separates OAuth orchestration from provider-specific broker capabilities. The required conceptual flow is:
`Frontend -> Broker OAuth/Auth Orchestrator -> BrokerAdapter -> Provider OAuth/API`

The `BrokerAdapter` must implement the following OAuth lifecycle methods (instead of a single synchronous login method):
- `GetAuthURL`
- `ExchangeToken`
- `RefreshToken` (if applicable)
- `RevokeToken` / Disconnect (if applicable)

Unsupported capabilities for specific providers must fail safely.

| Decision | Proposed Policy | Evidence | Approval Required |
|----------|-----------------|----------|-------------------|
| **Broker Login Flow** | External OAuth 2.0 / Token Redirection | ADR-001 (App Auth Separation) | APPROVED |
| **Callback Handling** | Dedicated backend `GET /api/v1/broker/callback/{provider}` | Standard OAuth flow | APPROVED |
| **OAuth State / CSRF**| Secure cryptographic `state` parameter bound to App Session | General App Security | APPROVED |
| **Access-Token Storage**| PostgreSQL `broker_sessions` table (Encrypted at rest) | ADR-001 | APPROVED |
| **Refresh-Token Storage**| PostgreSQL `broker_sessions` table (Encrypted at rest) | ADR-001 | APPROVED |
| **Encryption at Rest** | AES-256-GCM authenticated encryption | Standard InfoSec | APPROVED |
| **Encryption Key Mgmt**| Provisioned via Environment Variable (`BROKER_ENCRYPTION_KEY`) | General 12-factor | APPROVED |
| **Token Expiry** | Polled/derived from Provider response; enforced via session | Provider Specs | APPROVED |
| **Token Refresh** | Automatic background refresh or synchronous refresh on 401 | Provider Specs | APPROVED |
| **Logout / Revocation**| Soft-delete token locally + API call to revoke at provider | Standard Security | APPROVED |
| **Credential Redaction**| Absolute exclusion of all Tokens from application logs | ADR-001 | APPROVED |
| **Audit Logging** | Record connection/disconnection timestamps without token data | ADR-001 | APPROVED |
