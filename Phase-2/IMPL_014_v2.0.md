######################################################################################################################## 
# IMPL_014: HOME DASHBOARD IMPLEMENTATION ARCHITECTURE
######################################################################################################################## 

## DOCUMENT INFORMATION
Directive ID: IMPL-014
Document Name: Home Dashboard Implementation Architecture
Version: 2.0
Priority: High
Status: APPROVED
Dependencies: `Phase-1/SPEC_012.md`, `Volume_5_Implementation_Architecture.md`

######################################################################################################################## 
## 1. SCOPE
This document outlines the technical blueprint for the frontend Home Dashboard and its aggregate data fetching requirements.

######################################################################################################################## 
## 2. COMPONENT ARCHITECTURE (FRONTEND)
- **Framework**: Next.js (React)
- **Data Fetching**: Next.js App Router (React Server Components) for initial static shell, with SWR/React Query for clientside hydration of realtime index data.
- **Components**:
  - `HomeDashboardLayout`: Grid-based responsive layout container.
  - `GlobalNavbar`: Sticky top navigation with user profile and broker connection state.
  - `IndexSummaryWidget`: Subscribes to WebSocket index ticks.
  - `QuickToolsPanel`: Grid of static links pointing to `/trade/strategy-builder`, `/analyze/option-chain`, etc.
  - `BrokerConnectionBanner`: Conditionally rendered warning if `brokerStatus === 'disconnected'`.

######################################################################################################################## 
## 3. BACKEND DOMAIN ARCHITECTURE
- **Domain**: `market-service` (for indices) and `user-service` (for broker state).
- **APIs**:
  - `GET /api/v1/market/indices/summary`: Backend retrieves daily snapshots from Redis cache (populated by tick workers) and returns JSON.
  - `GET /api/v1/user/broker/status`: Queries PostgreSQL `user_broker_tokens` table for the active broker session (e.g., Zerodha, Angel One, ICICI Direct).

######################################################################################################################## 
## 4. REALTIME DATA FLOW
- **WebSocket Subscriptions**: The `IndexSummaryWidget` issues a `SUBSCRIBE` command for NIFTY, BANKNIFTY, and FINNIFTY instrument tokens on mount.

######################################################################################################################## 
## 5. OBSERVABILITY & ERRORS
- **Telemetry**: Track "Dashboard Load Time" and "Broker Connection Banner Clicked".
- **Errors**: If WebSocket fails, fallback to polling `GET /api/v1/market/indices/summary` every 10 seconds.
