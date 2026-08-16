# P4-08 INSTRUMENT MAPPING DECISION

**Status:** [PENDING PROJECT OWNER APPROVAL]
**Date:** 2026-08-10

## 1. Governance Principles
- The internal `InternalSymbol` (P4-04 standard) remains the authoritative canonical identifier.
- The `InstrumentRepository` must maintain a 1:1 strict mapping between `InternalSymbol` and the provider's token.
- **NEVER** guess or programmatically reconstruct a provider symbol. If a mapping is missing, return `ErrUnsupportedInstrument`.

## 2. Angel One Instrument Mapping
- **Source:** Angel One provides a daily JSON OpenAPI Scrip Master (`OpenAPIScripMaster.json`).
- **Identifier:** `symbolToken` (String).
- **Exchange Identifier:** Must specify the correct `exchange` (e.g., `NSE`, `BSE`, `NFO`) as `symbolToken` can be non-unique across exchanges.
- **Mapping Logic:** Filter by `exchange` and `segment`, map against our `InternalSymbol`.

## 3. ICICI Direct Instrument Mapping
- **Source:** Security Master file updated daily at 8:00 AM (available via Breeze portal).
- **Identifier:** Token (e.g., `4.1!12345` for WS, `stock_code` for REST).
- **Exchange Identifier:** `NSE`, `BSE`, `NDX`.
- **Mapping Logic:** Parse CSV/TXT, map `stock_code` to `InternalSymbol`.

## 4. Refresh Mechanism & Lifecycle (Gap Resolved)
- **Refresh Schedule:** The `InstrumentRepository` MUST download and parse the master files at application startup, and thereafter on a daily Cron schedule at **08:30 AM IST** (ensuring both Angel One and ICICI Direct 08:00 AM updates are available).
- **Stale/Inactive Mappings:** Mappings are completely purged and rebuilt during the 08:30 AM refresh. Mappings fetched during the morning remain valid and immutable for the duration of the current trading session.
- **Missing Mapping Behavior:** If a symbol exists in P4-04 but disappears from the provider's master file (e.g. delisted/expired), any attempt to trade/read it MUST return `ErrUnsupportedInstrument`.
- **Failure Behavior:** If the provider instrument-master download fails, the repository MUST log a CRITICAL error and retain the previously cached mappings to allow the application to function. Do NOT purge if the download fails.

*This document is a governance proposal and must be approved before implementation begins.*
