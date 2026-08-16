# P4-07 Execution Plan Final Review

## 1. Previous Findings
The preliminary forensic review identified critical deviations from the approved governance in the execution plan:
1. False protocol normalization assuming all providers use OAuth 2.0 `ExchangeToken`.
2. Missing database security constraint `encryption_version` for token key rotation.
3. Incorrect REST API endpoints (`/auth/:provider/login` instead of `/connect/{provider}`).
4. Unauthorized event ownership (publishing Redis events directly from `BrokerAdapter`).
5. Overly broad 5xx retry logic that risked duplicate order execution on timeouts.
6. Generic Dependency Injection bindings without explicit constructors.

## 2. Corrections Applied
The `implementation_plan.md` has been successfully updated to incorporate all six required corrections:
1. **Provider Authentication**: The plan now specifies normalizing the internal `BrokerSession` rather than forcing uniform provider protocols. Angel One and ICICI Direct implementations will use their specific auth patterns.
2. **Database Security**: Migration `000006_create_broker_schema.up.sql` now explicitly requires `encryption_version INTEGER NOT NULL` in the `broker_sessions` table to support future AES key rotation. `user_id` remains `BIGINT REFERENCES users(id) ON DELETE CASCADE`.
3. **REST API Contract**: All proposed routes have been updated to strictly match the API contract (`/connect/{provider}` and `/callback/{provider}`).
4. **Realtime Event Ownership**: Event publication (`order.execution`, `order.status`) has been correctly reassigned to the `BrokerManager` (Facade), ensuring provider-specific data does not leak into the Redis pipeline.
5. **Retry / Order Safety**: The plan now explicitly bans retries on `PlaceOrder` to prevent duplicate executions. Retries are restricted to idempotent read operations. The governed circuit breaker policy (>50% failure over 1 min) is explicitly stated.
6. **Fx Dependency Injection**: The plan now lists exact `fx.Provide` constructor strings mapped precisely to the repository structure.

## 3. Governance Evidence
All applied corrections conform perfectly to:
- `P4-07-FINAL-PROJECT-APPROVAL.md`
- `P4-07-FINAL-GOVERNANCE-CONSISTENCY-AUDIT.md`
- `P4-07-DATA-MODEL-DECISION.md`
- `P4-07-BROKER-AUTH-DECISION.md`
- `P4-07-INSTRUMENT-MAPPING-DECISION.md`
- `P4-07-REALTIME-CONTRACT-DECISION.md`
- `P4-07-API-CONTRACT-DECISION.md`
- `P4-07-FAILURE-SEMANTICS-DECISION.md`
- `P4-07-SECURITY-DECISION.md`

## 4. Repository Evidence
- The `users` table uses `id BIGSERIAL PRIMARY KEY`.
- `Backend/internal/bootstrap/bootstrap.go` utilizes the exact `fx.Provide` structure.
- Highest current migration is `000005`, confirming `000006` is next.

## 5. Remaining Legitimate Provider Unknowns
- Exact JSON response structures from Zerodha, Angel One, and ICICI Direct for edge-case errors.
- Exact rate limit headers and `Retry-After` semantics per provider.
- These must be dynamically discovered during code implementation by inspecting official provider API documentation.

## 6. Final Execution-Plan Verdict

**P4-07 EXECUTION PLAN = APPROVED**
**P4-07 IMPLEMENTATION = AUTHORIZED**
