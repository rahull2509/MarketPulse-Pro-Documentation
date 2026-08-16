# P4-09 DATA STORAGE DECISION

**Status:** [APPROVED]
**Date:** 2026-08-10

## 1. Objective
Define Redis caching, keys, TTL, and cache behaviors.

## 2. Redis Cache Governance (APPROVED)

- **2A. Redis Key Format:** 
  - Aggregated: `mp:broker:portfolio:{user_id}`
  - Provider Snapshot: `mp:broker:portfolio:{user_id}:{provider}`
  - User identity strictly enforced.
- **2B. Exact TTL:** 5 seconds. Explicit PO decision.
- **2C. Cache Refresh:** Cache miss or expiry triggers fresh aggregation. Valid cache returns without fan-out.
- **2D. Cache Invalidation:** Invalidate after `PlaceOrder`, `ModifyOrder`, `CancelOrder` for the authenticated user.
- **2E. Cache Miss:** Fetch active providers in parallel.
- **2F. Provider Timeout:** Treated as failed provider. Does not fail entire portfolio.
- **2G. Provider 429:** Maps to `ErrRateLimitExceeded`. DOES NOT increment CircuitBreaker 5xx counter. No automatic retry.
- **2H. Stale Cache:** May return stale cached provider snapshot if provider fails, explicitly marked `stale = true`. Cannot be represented as fresh.
- **2I. Redis Restart:** Ephemeral. Loss = next request is fresh. No PostgreSQL persistence.
- **2J. Application Restart:** Existing Redis caches remain usable if TTL allows.
- **2K. Concurrent Requests:** Per-user single-flight/coalescing mechanism MUST be used for simultaneous cache misses to prevent unbounded fan-out.

**Status:** P4-09 DATA STORAGE = APPROVED.
