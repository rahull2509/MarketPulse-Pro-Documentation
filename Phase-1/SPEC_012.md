######################################################################################################################## 
# SPEC_012: HOME DASHBOARD
######################################################################################################################## 

## DOCUMENT INFORMATION
Directive ID: SPEC-012
Document Name: Home Dashboard Specification
Priority: High
Status: PARTIALLY DEFINED
Dependencies: `Volume_2_Requirements.md`

######################################################################################################################## 
## 1. SCOPE & RESPONSIBILITIES
The Home Dashboard is the primary landing page after user authentication. It provides market summaries, quick entry points to tools, and an overview of active strategies.

######################################################################################################################## 
## 2. FUNCTIONAL REQUIREMENTS

### 2.1 Navigation & Global Elements
- **Global Navigation**: The system must provide a primary navigation bar (Home, Trade, Analyze, Watchlist, Positions, Profile). [CONFIRMED REQUIREMENT]
- **Broker Login CTA**: If a supported broker (Zerodha, Angel One, ICICI Direct) is disconnected, display a prominent call-to-action to connect the broker. [CONFIRMED REQUIREMENT]

### 2.2 Market Summary Widgets
- **Index Overview**: Display realtime mini-charts and LTP for NIFTY, BANKNIFTY, and FINNIFTY. [REFERENCE-DERIVED REQUIREMENT]
- **Market Heatmap Entry**: Provide a summary widget linking to the full market heatmap. [REFERENCE-DERIVED REQUIREMENT]

### 2.3 Tool Discovery
- **Strategy Entry Points**: Quick links to launch the Strategy Builder, Easy Options, and Option Chain. [CONFIRMED REQUIREMENT]
- **Verified P&L**: (Where approved via settings) Display a summary of trader's P&L scorecard. [DECISION REQUIRED]

### 2.4 User Flows & States
- **Loading State**: The system must display skeleton loaders for index widgets during initial fetch. [REFERENCE-DERIVED REQUIREMENT]
- **Responsive Behavior**: The dashboard must degrade gracefully into a single-column layout on mobile devices. [CONFIRMED REQUIREMENT]

######################################################################################################################## 
## 3. DATA & API REQUIREMENTS

### 3.1 Data Required
- Realtime index ticks.
- Broker connection status.

### 3.2 API Required
- `GET /api/v1/market/indices/summary`: Fetch daily snapshot of major indices.
- `GET /api/v1/user/broker/status`: Check current broker connection token validity (e.g., Zerodha, Angel One, ICICI Direct).
