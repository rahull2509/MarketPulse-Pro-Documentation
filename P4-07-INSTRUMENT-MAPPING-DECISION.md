# P4-07 Instrument Mapping Decision

This document defines the approved governance policies for mapping internal symbols (P4-04 standard) to external broker symbols and instrument tokens.

| Decision | Proposed Policy | Evidence | Approval Required |
|----------|-----------------|----------|-------------------|
| **Mapping Ownership** | Backend Provider Facade (`BrokerAdapter`) | ADR-001 (Normalization) | APPROVED |
| **Internal Symbol Definition**| P4-04 Standard (e.g. `NIFTY24DEC21000CE`) | Existing P4-06 | APPROVED |
| **Broker Symbol Definition** | Specific to provider (e.g. `NIFTY24DEC21000CE` for Zerodha) | Provider Specs (Pending) | APPROVED |
| **Broker Instrument Token**| Specific to provider (numeric or string ID) | Provider Specs (Pending) | APPROVED |
| **Mapping Source** | Daily fetch from Provider's Instrument Master API | Standard Industry Practice | APPROVED |
| **Mapping Storage (Cache)** | PostgreSQL `broker_instruments` table or Redis | High-speed lookup required | APPROVED |
| **Update Frequency** | Once per day (Pre-market) | Standard Market Hours | APPROVED |
| **Missing Mapping Behavior**| Reject broker action with `ERR_INSTRUMENT_NOT_MAPPED` | Strict Integrity | APPROVED |
| **Ambiguous Mapping** | Reject broker action; log alert | Strict Integrity | APPROVED |
| **Stale Mapping Behavior** | If token changes intra-day, facade fails until re-synced | Strict Integrity | APPROVED |
| **Timestamp Normalization** | Facade maps provider timestamps to UTC `DateTime64` | General DB standard | APPROVED |
