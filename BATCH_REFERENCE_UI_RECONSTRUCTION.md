# BATCH-1 & BATCH-2 REFERENCE UI RECONSTRUCTION

This document maps the reconstructed UI structural requirements from the original reference material (Sensibull-inspired Batch-1/Batch-2) to the current MarketPulse Pro product scope.

## 1. Global Navigation
- **Top Navigation Bar:** Persistent across all `/app/*` routes. Contains branding (MarketPulse Pro) and primary product links.
- **Links:** Home, Trade, Analyse, Watchlist, Positions, Orders.
- **User Menu:** Logout functionality.

## 2. Route Map
- `/` (Landing Page)
- `/auth/login` (Login Page)
- `/auth/register` (Register Page)
- `/app/home` (Home Dashboard)
- `/app/trade` (Strategy Builder & Trading Tools)
- `/app/analyse` (Option Chain & Advanced Analytics)
- `/app/watchlist` (Custom Watchlists)
- `/app/positions` (Open/Closed Positions)
- `/app/orders` (Order Management)

## 3. Page Map & Governance Status

| Screen / Feature | Location | Reconstructed Purpose | Status |
|------------------|----------|-----------------------|--------|
| Home Dashboard | `/app/home` | Command center with global market indices summary (NIFTY/BANKNIFTY) and quick links. | APPROVED |
| Option Chain | `/app/analyse` | Dense data table for options data (CE/PE, Volume, OI, LTP). Contains filters and instrument selection. | `REFERENCE-DERIVED` / APPROVED (SPEC_001) |
| Strategy Builder | `/app/trade` | Builder workflow with legs, payoff chart, P&L stats, Greeks. | APPROVED (SPEC_011) |
| Watchlist | `/app/watchlist` | List/Table of tracked instruments with LTP and % change. | `REFERENCE-DERIVED` |
| Positions | `/app/positions` | Table of open/closed positions with P&L metrics. | APPROVED (SPEC_010 integration) |
| Orders | `/app/orders` | Table of pending, executed, or cancelled orders. | APPROVED (SPEC_010 integration) |
| Multi-strike OI | `/app/analyse` | Advanced charting for open interest tracking. | `REFERENCE-DERIVED` |
| Broker Login | Modal/Auth | Zerodha, Angel One, ICICI Direct broker selection buttons. | APPROVED (DEC-ARCH-004C/D/E) |

## 4. Shared Components
- `Button`, `Input`, `Card`, `PageHeader`, `TopNav`, `EmptyState`
- `Table` (Reusable data grid for financial data)
- `ChartContainer` (Reusable wrapper for Recharts/similar chart lib)
- `Select` / `DropdownMenu` (For underlying/expiry selection)

## 5. UI Structure Breakdown

### 5.1 Trade UI (Strategy Builder)
- **Top Bar:** Select Underlying, Select Expiry.
- **Left Panel:** Option Chain view or Strike Selector (CE/PE, Buy/Sell toggles).
- **Right Panel (Analytics):**
  - Strategy Legs summary.
  - Payoff Chart.
  - P&L and Breakeven metrics.
  - Greeks and IV stats.
  - Save Strategy CTA.

### 5.2 Analyse UI (Option Chain)
- **Controls:** Instrument, Expiry.
- **Layout:** Central strike column with Calls (Left) and Puts (Right).
- **Data Points:** OI, Volume, LTP, Bid/Ask.

### 5.3 Home Dashboard UI
- **Widgets:** Index trackers (NIFTY 50, BANK NIFTY).
- **Navigation:** Quick entry cards for Option Chain and Strategy Builder.
- **Market State:** Bullish/Bearish indicators (mocked/empty).

### 5.4 Watchlist, Positions, Orders
- Standardized dense financial tables with clear typography.
- Status badges (Executed, Pending).
- Colored numeric cells (Green for profit/gain, Red for loss).

## 6. Loading, Empty, and Error States
Every page must safely handle missing data (since broker execution is not implemented in P4-01/P4-02).
- **Empty State:** "No data available."
- **Loading State:** Skeleton loaders for tables and charts.

## 7. Responsive Rules
- Container: `max-w-7xl` centered.
- Table handling: Horizontal scroll overflow on smaller screens (`overflow-x-auto`).
- Grids: Collapse from multi-column (e.g. Strategy Builder left/right panels) to single column stacked on mobile.

## 8. Broker Integration Scope
UI strictly limited to **Zerodha**, **Angel One**, and **ICICI Direct**. Upstox is explicitly banned. Broker UI will be visually represented but functionally mocked or restricted until Phase 5 backend implementation.
