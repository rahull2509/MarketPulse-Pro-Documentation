# P4-09 SECURITY DECISION

**Status:** [PENDING PROJECT OWNER APPROVAL]
**Date:** 2026-08-10
**Decision ID:** DEC-ARCH-P409-G

## 1. Objective
Ensure that P4-09 portfolio aggregation does not compromise the strict user and provider isolation established in P4-07 and P4-08.

## 2. Security Proposals

### 2.1 User Isolation
- All aggregation queries must enforce `user_id` isolation via the JWT middleware. The `BrokerManager` must verify that the active `BrokerSession` strictly matches the `userID` provided in the JWT context. No cross-user data leakage is permissible.

### 2.2 Provider Credentials
- Portfolio aggregation endpoints will NEVER expose provider session tokens, API keys, or underlying provider credentials to the frontend.
- Provider responses will be scrubbed and mapped to generic `BrokerPosition` / `BrokerHolding` structs before JSON serialization.

### 2.3 Redis Event Isolation
- Realtime WebSocket updates mapped to Redis Pub/Sub MUST be published to isolated channels prefixed by user ID (e.g., `user.{user_id}.portfolio`).
- The WebSocket server MUST enforce that the connected socket only subscribes to its authenticated user's channels.

**Note:** These security controls are proposals and remain strictly [PENDING PROJECT OWNER APPROVAL]. Implementation cannot begin until these are explicitly governed.
