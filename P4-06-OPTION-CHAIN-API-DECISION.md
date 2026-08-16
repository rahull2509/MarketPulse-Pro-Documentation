# P4-06 Option Chain API Decision

This document formally proposes the REST API contract required for fetching Option Chains in P4-06.

## 1. Option Chain REST API
**Status:** PROPOSED GOVERNANCE DECISION (Approval Required)

The following contract is proposed for exposing Option Chain data stored in ClickHouse.

### Route & Method
`GET /api/v1/options/chain`

### Authentication & Authorization
- **Authentication:** Standard P4-02 JWT middleware.
- **Authorization:** Global access for authenticated users (no user-specific restrictions for market data).

### Request Parameters (Query)
- `underlying` (string, required): e.g., `NIFTY`
- `expiry` (string, required): e.g., `2024-12-26`

### Response Structure
Based on the existing `OptionChainEntry` and `OptionGreeks` models (and respecting governance constraints that OI, bid/ask are deferred):

```json
{
  "status": "success",
  "data": {
    "underlying": "NIFTY",
    "expiry": "2024-12-26",
    "timestamp": "2024-12-10T15:30:00Z",
    "chain": [
      {
        "strike_price": 21000,
        "expiry_date": "2024-12-26",
        "call_data": {
          "symbol": "NIFTY24DEC21000CE",
          "delta": 0.85,
          "gamma": 0.02,
          "theta": -10.5,
          "vega": 15.2,
          "iv": 0.12,
          "timestamp": "2024-12-10T15:30:00Z"
        },
        "put_data": {
          "symbol": "NIFTY24DEC21000PE",
          "delta": -0.15,
          "gamma": 0.02,
          "theta": -8.5,
          "vega": 15.2,
          "iv": 0.13,
          "timestamp": "2024-12-10T15:30:00Z"
        }
      }
    ]
  }
}
```

### Errors
Standard application error conventions (e.g., `400 Bad Request` for missing parameters, `401 Unauthorized` for missing JWT, `500 Internal Server Error`).

### Performance / Pagination
- **Pagination:** Not required. A single expiry chain for a specific underlying is a naturally bounded dataset (typically <200 strikes). It should be returned as a single JSON array to facilitate immediate frontend rendering.
