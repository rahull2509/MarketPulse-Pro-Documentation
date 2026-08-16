# P4-05 Event Contract Decision

## Status
APPROVED

## Evidence Reviewed
- `Architecture/EVENT_CATALOG.md` (Events marked PROPOSED).
- `Architecture/PHASE4_ARCHITECTURE_ROADMAP.md` (Governing P4-05 Redis Pub/Sub mapping).
- `Backend/internal/modules/marketdata/models/market.go` (Canonical models: `MarketTick`).
- `Backend/internal/modules/marketdata/services/ingest_service.go` (P4-04 Ingest boundary).

## Decision
The authoritative payload schemas for realtime market data distribution over Redis Pub/Sub are hereby locked and approved for P4-05 execution.

## market.prices

### Event Identity
- **Event Name**: `market.prices`
- **Event Version**: `1.0`
- **Event Category**: Market Data
- **Publisher**: Market Data Ingest Service (P4-04 Boundary)
- **Transport**: Redis Pub/Sub -> WebSocket Hub

### Envelope
The event uses a standard JSON envelope strictly enforcing versioning, routing, and temporal tracking:
```json
{
  "event": "market.prices",
  "version": "1.0",
  "timestamp": "2023-10-01T12:00:00Z",
  "data": { ... }
}
```

### Payload
Derived directly from the authoritative P4-04 `models.MarketTick` structure to prevent transformation overhead:
```json
{
  "symbol": "NIFTY",
  "ltp": 19500.50,
  "open": 19400.00,
  "high": 19600.00,
  "low": 19350.00,
  "close": 19450.00,
  "volume": 1500000,
  "timestamp": "2023-10-01T12:00:00Z"
}
```

### Required Fields
- `symbol` (String, Non-null)
- `ltp` (Numeric/Float64, Non-null)
- `timestamp` (String/RFC3339, Non-null)

### Optional Fields
- `open`, `high`, `low`, `close`, `volume` (Numeric/Float64/Int64). May be omitted or zero if the upstream provider only emits trade ticks rather than aggregate OHLCV candles.

### Types
All price values are strictly standard IEEE 754 `float64` to match the Go backend definitions. Volumes are `int64`.

### Timestamp Semantics
Timestamps MUST reflect the actual exchange trade/tick time (origin time), strictly formatted as an ISO-8601/RFC3339 string. They do NOT represent the ingestion time.

### Versioning
Schema evolution is handled by the `version` field in the envelope. Breaking changes to `data` require a bump to `2.0`. Additive changes preserve `1.x`.

### Redis Channel
`market.prices.{symbol}` (e.g., `market.prices.NIFTY`).
This prevents a global firehose bottleneck and allows granular subscription filtering via Redis `PSUBSCRIBE`.

### WebSocket Routing
The Hub internal channel mirrors the Redis channel exactly: `market.prices.{symbol}`.

---

## market.news

### Status
GOVERNANCE GAP — INSUFFICIENT EVIDENCE

The repository lacks any data model, ingestion pathway, or upstream provider for `market.news`. P4-04 exclusively implemented ticks and Greeks. There is no structural evidence to define a news schema without inventing non-governed requirements.

P4-05 implementation is explicitly authorized to proceed **WITHOUT** `market.news`. The `market.news` event is deferred to a future phase.

---

## Rejected Alternatives
1. **Global Firehose (`market.prices`)**: Publishing all ticks to a single channel was rejected. It would force the WebSocket hub and browser clients to filter thousands of irrelevant ticks per second, causing massive memory/CPU exhaustion.
2. **Binary Envelopes (Protobuf/MessagePack)**: Rejected. JSON is strictly required for native browser compatibility and observability, aligning with the P4 architectural simplicity mandate.

## Compatibility
Fully compatible with P4-04 `MarketTick`. The JSON payload matches the Go JSON tags natively, requiring no additional struct mapping or allocations during serialization.
