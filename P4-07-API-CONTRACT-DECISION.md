# P4-07 API Contract Decision

This document defines the approved backend REST API endpoints for the Broker Integration Facade.

| Method | Path | Authentication | Authorization | Description | Error Semantics | Status |
|--------|------|----------------|---------------|-------------|-----------------|--------|
| `GET` | `/api/v1/broker/connect/{provider}` | JWT | Authenticated | Initiates broker OAuth login. Returns redirect URL. | 400, 500 | APPROVED |
| `GET` | `/api/v1/broker/callback/{provider}` | None (OAuth flow) | Valid `state` | Provider redirects here. Exchanges code for tokens. | 401, 500 | APPROVED |
| `GET` | `/api/v1/broker/status` | JWT | Authenticated | Returns connected providers and session expiry. | 500 | APPROVED |
| `POST` | `/api/v1/broker/orders` | JWT | Authenticated | Place a new order (idempotent via UUID). | 400, 422, 500 | APPROVED |
| `PUT` | `/api/v1/broker/orders/{id}` | JWT | Authenticated, Owner | Modify an open order. | 400, 404, 422, 500 | APPROVED |
| `DELETE`| `/api/v1/broker/orders/{id}` | JWT | Authenticated, Owner | Cancel an open order. | 404, 500 | APPROVED |
| `GET` | `/api/v1/broker/orders` | JWT | Authenticated, Owner | Fetch today's order book. | 500 | APPROVED |
| `GET` | `/api/v1/broker/positions` | JWT | Authenticated, Owner | Fetch intraday and open positions. | 500 | APPROVED |
| `GET` | `/api/v1/broker/holdings` | JWT | Authenticated, Owner | Fetch long-term equity holdings. | 500 | APPROVED |
