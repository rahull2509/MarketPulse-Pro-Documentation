# Event Catalog

This document defines the event taxonomy for MarketPulse Pro over Redis Pub/Sub. All realtime distribution flows through this bus. 

Kafka and NATS are explicitly NOT approved for this phase.

## Current Approved Events

| Event Channel | Event Name | Publisher | Consumers | Payload Owner | Delivery Requirement | Ordering Requirement | Status |
|---|---|---|---|---|---|---|---|
| `mp:events:system` | `system.health` | Worker / Scheduler | Realtime Hub | Core API | At-most-once | None | APPROVED (P3 Foundation) |

## Proposed Events (Pending Explicit Decision)

> [!WARNING]
> These events reflect architectural expectations for Phase 4 Market Data and Broker integration, but are strictly PROPOSED pending final data payload specifications.

| Event Channel | Event Name | Publisher | Consumers | Payload Owner | Delivery Requirement | Ordering Requirement | Status |
|---|---|---|---|---|---|---|---|
| `mp:events:marketdata` | `market.prices` | Market Data | Realtime Hub | Market Data | At-most-once | Timestamped | PROPOSED |
| `mp:events:marketdata` | `market.news` | Market Data | Realtime Hub | Market Data | At-most-once | Timestamped | PROPOSED |
| `mp:events:orders` | `order.execution` | Broker Integration | Realtime Hub | Broker Integration | At-least-once | Timestamped | PROPOSED |
| `mp:events:orders` | `order.status` | Broker Integration | Realtime Hub | Broker Integration | At-least-once | Timestamped | PROPOSED |
