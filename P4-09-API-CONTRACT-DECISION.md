# P4-09 API CONTRACT DECISION

**Status:** [PENDING PROJECT OWNER APPROVAL]
**Date:** 2026-08-10
**Decision ID:** DEC-ARCH-P409-A

## 1. Objective
Define the API contract for the unified portfolio and trading endpoints.

## 2. Proposed Endpoints

### 2.1 Unified Portfolio Aggregation
**Endpoint:** `GET /api/v1/broker/portfolio`
**Authentication:** Required (JWT)
**Authorization:** User-isolated
**Query Parameters:**
- `refresh` (boolean, optional): If `true`, bypasses cache to force live fetch.
**Response Body (200 OK):**
```json
{
  "snapshot_id": "uuid-v4",
  "timestamp": "2026-08-10T11:15:00Z",
  "partial_success": false,
  "failed_providers": [],
  "aggregated_positions": [
    {
      "internal_symbol": "RELIANCE",
      "net_quantity": 150,
      "weighted_average_price": 2500.50,
      "providers": [
        {"broker": "ZERODHA", "quantity": 100, "average_price": 2500.00},
        {"broker": "ANGELONE", "quantity": 50, "average_price": 2501.50}
      ]
    }
  ],
  "aggregated_holdings": []
}
```

### 2.2 Broker-Specific Data (Existing P4-07 Routes)
**Endpoints:** 
- `GET /api/v1/broker/positions?provider=zerodha`
- `GET /api/v1/broker/holdings?provider=zerodha`
- `GET /api/v1/broker/orders?provider=zerodha`
**Proposed Change:** Standardize responses to include `snapshot_id` and `timestamp`.

### 2.3 Order Execution (P4-07 Extension)
**Endpoint:** `POST /api/v1/broker/orders`
**Proposed Change:** None at this phase. The existing P4-07 interface (`interfaces.OrderRequest`) already handles single-provider execution. P4-09 does not propose cross-broker algorithmic splitting.

## 3. Error Semantics
- **500 Internal Server Error:** If ALL connected providers fail to respond.
- **206 Partial Content / 200 OK with `partial_success: true`:** If >0 but <ALL providers respond successfully.

**Note:** All endpoints and structures are proposals and remain strictly [PENDING PROJECT OWNER APPROVAL].
