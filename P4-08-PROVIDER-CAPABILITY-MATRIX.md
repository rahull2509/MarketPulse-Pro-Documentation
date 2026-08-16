# P4-08 PROVIDER CAPABILITY MATRIX

**Status:** [PENDING PROJECT OWNER APPROVAL]
**Date:** 2026-08-10

| Capability | Angel One | ICICI Direct |
|---|---|---|
| Authentication | PROVIDER-VERIFIED | PROVIDER-VERIFIED |
| Orders | PROVIDER-VERIFIED | PROVIDER-VERIFIED |
| Modify Order | PROVIDER-VERIFIED | PROVIDER-VERIFIED |
| Cancel Order | PROVIDER-VERIFIED | PROVIDER-VERIFIED |
| Order Status | PROVIDER-VERIFIED | PROVIDER-VERIFIED |
| Positions | PROVIDER-VERIFIED | PROVIDER-VERIFIED |
| Holdings | PROVIDER-VERIFIED | PROVIDER-VERIFIED |
| Instrument Mapping | UNKNOWN — GOVERNANCE REQUIRED | UNKNOWN — GOVERNANCE REQUIRED |
| Market Data | UNSUPPORTED | UNSUPPORTED |
| WebSocket | UNKNOWN — GOVERNANCE REQUIRED | UNKNOWN — GOVERNANCE REQUIRED |
| Rate Limits | UNKNOWN — GOVERNANCE REQUIRED | UNKNOWN — GOVERNANCE REQUIRED |
| SDK | UNSUPPORTED | UNSUPPORTED |

## Notes
- **Instrument Mapping:** Requires defining exact token identifiers for both brokers against MarketPulse Pro's canonical master contract format.
- **Market Data:** P4-04 established a different data source; individual broker data feeds are not consumed here.
- **WebSocket:** We only manage standard REST APIs in the BrokerAdapter. Websockets for execution events are currently unverified for these two providers.
- **SDK:** Explicitly marked UNSUPPORTED as we mandate direct HTTP usage to strictly control failure semantics and CircuitBreaker integrations.
