# P4-07 Data Ownership Decision

This document defines the strict ownership and normalization boundaries between the internal market data pipeline (P4-04/P4-06) and the external broker facade (P4-07).

| Data Type | Canonical Source | Allowed Consumers | Broker Supplementation Allowed? | Broker Replacement Allowed? | Normalization Boundary | Failure Behavior | Governance Status |
|-----------|------------------|-------------------|---------------------------------|-----------------------------|------------------------|------------------|-------------------|
| **Market Ticks (Live)** | P4-04 Ingestion | Realtime Hub, P4-06 Greeks | NO (Broker ticks do not enter P4-04) | NO (P4-04 is authoritative) | Strictly Internal | Fallback to prior tick | APPROVED |
| **Underlying Spot** | P4-04 Redis Cache | P4-06 Greeks | NO (Broker spot must not alter internal cache) | NO | Strictly Internal | Block Greeks Calculation | APPROVED |
| **Option Ticks (Live)** | P4-04 Ingestion | Realtime Hub, P4-06 Greeks | NO | NO | Strictly Internal | Drop gracefully | APPROVED |
| **Instruments / Master** | Internal DB/CSV (P4-04) | Parsers, UIs | YES (Broker API may be used to map token IDs) | NO (Internal symbol is primary key) | Facade Mapping Layer | Drop mapping | APPROVED |
| **Greeks** | P4-06 Worker | Option Chain API, UI | NO (Broker Greeks must not replace internal BSM) | NO | Strictly Internal | Recalculate / NaN | APPROVED |
| **Orders** | P4-07 Broker API | UI, User App | N/A (Broker is sole source) | N/A | Facade Normalization | Standard API Error | APPROVED |
| **Positions** | P4-07 Broker API | UI, User App | N/A (Broker is sole source) | N/A | Facade Normalization | Standard API Error | APPROVED |
| **Holdings** | P4-07 Broker API | UI, User App | N/A (Broker is sole source) | N/A | Facade Normalization | Standard API Error | APPROVED |
