######################################################################################################################## 
# IMPL_013: STRATEGY BUILDER IMPLEMENTATION ARCHITECTURE
######################################################################################################################## 

## DOCUMENT INFORMATION
Directive ID: IMPL-013
Document Name: Strategy Builder Implementation Architecture
Version: 2.0
Priority: Critical
Status: APPROVED
Dependencies: `Phase-1/SPEC_011.md`, `Volume_5_Implementation_Architecture.md`

######################################################################################################################## 
## 1. SCOPE
This document outlines the technical blueprint for the Strategy Builder frontend and backend components.

######################################################################################################################## 
## 2. COMPONENT ARCHITECTURE (FRONTEND)
- **Framework**: Next.js (React)
- **State Management**: Redux/Zustand or equivalent central state for holding the active strategy legs.
- **Components**:
  - `StrategyBuilderLayout`: Container for the module.
  - `UnderlyingSelector`: Autocomplete search for underlyings.
  - `LegManager`: Table/list UI to add, edit, and remove option legs.
  - `PayoffChart`: Interactive SVG/Canvas chart using a library like Recharts or D3 for visualizing the P&L curve.
  - `StrategySummary`: Displays max profit, max loss, breakevens, and net Greeks.

######################################################################################################################## 
## 3. BACKEND DOMAIN ARCHITECTURE
- **Domain**: `strategy-service` (or integrated within `user-service` if monolith).
- **Database Model (PostgreSQL)**:
  - Table: `user_strategies` (id, user_id, name, underlying_symbol, created_at, updated_at).
  - Table: `user_strategy_legs` (id, strategy_id, symbol, side, qty, price, type, expiry, strike).
- **APIs**:
  - `POST /api/v1/user/strategies`: JSON body with array of legs. Validates session token, stores transactionally.
  - `GET /api/v1/user/strategies`: Returns list of saved strategies.

######################################################################################################################## 
## 4. REALTIME DATA FLOW
- **WebSocket Subscriptions**: When a leg is added, the frontend issues a `SUBSCRIBE` command for that specific instrument token to the Realtime Layer (WebSocket).
- **State Updates**: Tick data (LTP, Bid/Ask) updates the leg state, which reactively triggers a recalculation of the PayoffChart and StrategySummary.

######################################################################################################################## 
## 5. ALGORITHMS & CALCULATIONS
- **P&L Calculation**: `Net P&L = Sum(Leg P&L)`.
- **Greeks**: Black-Scholes calculation must happen on the backend or via a heavily optimized WASM/JS library on the client. `[DECISION REQUIRED]`

######################################################################################################################## 
## 6. OBSERVABILITY & ERRORS
- **Telemetry**: Track "Strategy Saved", "Leg Added" events via standard APM.
- **Errors**: Return HTTP 400 with strict JSON Schema validation errors if strategy submission is invalid (e.g., mismatched underlyings).
