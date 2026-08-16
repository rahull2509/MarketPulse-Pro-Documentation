# P4-09 FAILURE SEMANTICS DECISION

**Status:** [APPROVED]
**Date:** 2026-08-10

## 1. Objective
Define partial failure handling, fan-out constraints, and timeout parameters.

## 2. Partial Failure Governance (APPROVED)
**Example:** Provider A = SUCCESS, Provider B = TIMEOUT, Provider C = 429
**Response:**
- `HTTP 200 OK`
- `partial_success = true`
- Successful provider data returned.
- `failed_providers` contains provider, error_code, error classification.
- **Provider B (Timeout):** Stale data MAY be returned if previous cache exists, with `stale = true` and original cache timestamp.
- **Provider C (429):** `ErrRateLimitExceeded`. No retry.

*Rule:* If ALL providers fail, return the governed complete-failure response, NOT an empty portfolio. Stale data must never falsely claim freshness.

## 3. Fan-Out & Concurrency (APPROVED)
- **Maximum Provider Fan-out:** 3 concurrent calls.
- **Aggregation Timeout:** 5000 milliseconds. Provider calls MUST use context cancellation.
- **Concurrency Mechanism:** Go `errgroup`-style parallel execution.
- **Retries:** No automatic retries permitted.
- **CircuitBreaker:** 429 responses DO NOT increment P4-07 CB 5xx counter. Existing CB remains authoritative.

**Status:** P4-09 FAILURE SEMANTICS = APPROVED.
