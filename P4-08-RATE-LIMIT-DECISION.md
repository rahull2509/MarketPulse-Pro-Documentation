# P4-08 RATE LIMIT DECISION

**Status:** [PENDING PROJECT OWNER APPROVAL]
**Date:** 2026-08-10

## 1. Governance Principles
- The BrokerAdapter implementation must respect the authoritative provider limits to avoid automated IP/account bans.
- If a rate limit is exceeded, the adapter must return a normalized error (`ErrRateLimitExceeded`) that the P4-07 Circuit Breaker can safely classify without treating it as a provider system failure.

## 2. Angel One Rate Limits (Gap Resolved)
| Capability | Limit | Time Window | Action if Exceeded |
|---|---|---|---|
| Order Placement | 10 requests | 1 Second | Returns 403. Adapter must yield `ErrRateLimitExceeded`. |
| WebSocket Subscriptions | 1,000 tokens | Per Session | Connections dropped or refused. |
| **Read APIs** (getPosition, getOrders, getHolding) | **1 request** | **1 Second** | Returns 403. Adapter must yield `ErrRateLimitExceeded`. |

## 3. ICICI Direct Rate Limits
| Capability | Limit | Time Window | Action if Exceeded |
|---|---|---|---|
| Order APIs (All) | 10 requests | 1 Second | Adapter must yield `ErrRateLimitExceeded`. |
| Global API Calls | 100 requests | 1 Minute | Adapter must yield `ErrRateLimitExceeded`. |
| Global API Calls | 5,000 requests | 1 Day | Adapter must yield `ErrRateLimitExceeded`. |
| Time Sync | Delta < 60s | N/A | Request Rejected. |

## 4. Implementation Directives
- **No Queuing:** We do **not** queue orders or read requests in the Facade if the rate limit is exceeded; we return a fast failure (`429/403` mapping to `ErrRateLimitExceeded`) to the user.
- **No Retries:** Rate limits are passed directly back to the caller. The BrokerAdapter MUST NOT implement internal retry or backoff loops.
- **Circuit Breaker:** Rate limit 429/403 responses MUST NOT increment the HTTP 5xx failure count in the P4-07 Circuit Breaker.

*This document is a governance proposal and must be approved before implementation begins.*
