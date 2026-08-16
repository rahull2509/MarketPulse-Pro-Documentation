# P4-07 Realtime Contract Decision

This document defines the approved Redis Pub/Sub contracts and WebSocket payloads for distributing broker execution events.

## 1. order.execution
Broadcast when a fill (partial or complete) occurs on an active order.
* **Status**: APPROVED
* **Redis Channel**: `mp:events:orders`
* **WebSocket Exposure**: Yes (Secured/Filtered by `user_id`)
* **Idempotency**: Consumers must de-duplicate via `execution_id`.
* **Proposed Payload**:
```json
{
  "event_type": "order.execution",
  "version": "1.0",
  "timestamp": "2026-08-10T10:15:30.123Z",
  "data": {
    "user_id": "uuid-1234",
    "broker_name": "zerodha",
    "broker_order_id": "23081000123456",
    "execution_id": "fill-9876",
    "internal_symbol": "NIFTY24DEC21000CE",
    "side": "BUY",
    "filled_quantity": 50,
    "execution_price": 125.50
  }
}
```

## 2. order.status
Broadcast when an order undergoes a state transition (e.g., OPEN -> REJECTED, OPEN -> CANCELLED).
* **Status**: APPROVED
* **Redis Channel**: `mp:events:orders`
* **WebSocket Exposure**: Yes (Secured/Filtered by `user_id`)
* **Idempotency**: Evaluated via `broker_order_id` + `status` timestamp.
* **Proposed Payload**:
```json
{
  "event_type": "order.status",
  "version": "1.0",
  "timestamp": "2026-08-10T10:15:31.000Z",
  "data": {
    "user_id": "uuid-1234",
    "broker_name": "zerodha",
    "broker_order_id": "23081000123456",
    "internal_symbol": "NIFTY24DEC21000CE",
    "status": "REJECTED",
    "status_message": "Insufficient margin",
    "error_code": "ERR_MARGIN"
  }
}
```
