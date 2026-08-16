# P4-07 Blocker Approval Matrix

This matrix requires explicit Project Owner approval to resolve the database schema mismatch and interface conflict identified during the final readiness audit.

| ID | Blocker | Existing Reality | Approved Decision | Proposed Correction | Owner Approval |
|----|---------|------------------|-------------------|---------------------|----------------|
| 1 | User ID type | `users.id` is BIGINT/BIGSERIAL | P4-07 specifies UUID | BIGINT matching existing users.id. | APPROVED |
| 2 | Broker OAuth architecture | `Authenticate(credentials)` is synchronous | OAuth 2.0 Redirection | Introduce a dedicated OAuth/auth orchestration boundary while keeping provider-specific BrokerAdapter implementations focused on broker capabilities. | APPROVED |
