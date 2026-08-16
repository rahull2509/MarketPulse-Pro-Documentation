# P4-07 Blocker Resolution Proposal

This document outlines the proposed governance and architectural corrections to resolve the two blockers identified during the final pre-implementation readiness audit.

## 1. Database Type Conflict

- **Existing repository evidence**: `Backend/migrations/000001_create_users_table.up.sql` explicitly defines the core user identity as `id BIGSERIAL PRIMARY KEY` (which translates to `BIGINT`).
- **Approved decision**: The approved `P4-07-DATA-MODEL-DECISION.md` dictates that `BrokerSession`, `BrokerOrder`, `BrokerPosition`, `BrokerHolding`, and `BrokerInstrumentMapping` use `user_id (UUID)` as the foreign key.
- **Exact conflict**: A `UUID` foreign key cannot reference a `BIGINT` primary key. This is a strict database migration blocker.
- **Recommended resolution**: Update the P4-07 data models to use `user_id (BIGINT)` instead of `UUID`. We must adapt P4-07 to the existing authoritative identity model rather than unnecessarily rewriting the existing, functioning user identity system.
- **Impact**: Zero impact on existing infrastructure. Ensures the upcoming P4-07 migrations will successfully apply.
- **Explicit approval required**: YES.

## 2. OAuth Interface Conflict

- **Existing BrokerAdapter interface**: Defined in `Backend/internal/modules/broker/interfaces.go`, the interface currently exposes a synchronous authentication signature: `Authenticate(ctx context.Context, credentials map[string]string) (*Session, error)`.
- **Approved OAuth decision**: The approved `P4-07-BROKER-AUTH-DECISION.md` strictly governs an **OAuth 2.0 / Token Redirection** flow.
- **Exact architectural conflict**: The synchronous `Authenticate` method cannot orchestrate an asynchronous browser redirect flow, which involves generating an authorization URL, handling an external callback, verifying a CSRF `state` nonce, and exchanging an authorization code for an access token.
- **Recommended interface architecture**: 
  Introduce a dedicated OAuth/auth orchestration boundary (e.g., a `BrokerAuthService`). 
  Replace the legacy `Authenticate` method on the `BrokerAdapter` interface with methods that strictly support the OAuth lifecycle. A search confirms there are currently zero usages of `BrokerAdapter` across the repository, meaning it can be safely replaced without backward compatibility concerns.
  Proposed interface additions:
  - `GetAuthURL(state string) string`
  - `ExchangeToken(ctx context.Context, code string) (*Session, error)`
  - `RefreshToken(ctx context.Context, refreshToken string) (*Session, error)`
  - `RevokeToken(ctx context.Context, accessToken string) error`
- **Impact**: Cleanly aligns the `BrokerAdapter` with the governed OAuth 2.0 flow. Zero blast radius on existing codebase.
- **Explicit approval required**: YES.

## 3. Proposed Governance Changes

1. **Update `Architecture/P4-07-DATA-MODEL-DECISION.md`**: Modify the data dictionary to specify `BIGINT` for all `user_id` foreign keys.
2. **Update `Backend/internal/modules/broker/interfaces.go`**: Replace the `Authenticate` method with OAuth-compatible lifecycle methods.

## 4. Non-Goals

The following remain explicitly out of scope for this task:
- no code (application logic)
- no migration creation or execution
- no API implementation
- no broker SDK integration
- no credentials or secrets modification
