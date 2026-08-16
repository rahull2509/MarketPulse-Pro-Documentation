# UI/UX Requirement Matrix

This document maps features and screens to concrete UI elements and requirements.
*Note: Any requirements derived from Sensibull screenshots are marked as `REFERENCE-DERIVED` until formally accepted. This matrix maps UI scope ONLY, and implies no backend API support.*

| Feature | Screen | UI Element | User Action | Expected Result | Data Required | Status |
|---|---|---|---|---|---|---|
| Option Chain | Option Chain | Instrument Selector | User selects NIFTY | Chain reloads | Strike prices, Ltp, OI | `REFERENCE-DERIVED` |
| Strategy Builder | Strategy Builder | Underlying Selector | User selects underlying | Option legs become available | Underlying list | APPROVED |
| Strategy Builder | Strategy Builder | Payoff Chart | User adds a leg | Payoff curve recalculates | Max Profit, Max Loss | APPROVED |
| Strategy Builder | Strategy Builder | IV Chart context | User clicks IV | Displays IV rank | IV Data | `REFERENCE-DERIVED` |
| Home Dashboard | Dashboard Home | Global Navigation | User clicks Analyze | Navigates to Analyze tools | N/A | APPROVED |
| Home Dashboard | Dashboard Home | Index Widgets | User loads page | Displays NIFTY/BANKNIFTY | Index LTP | APPROVED |
| Screener | Screener | Filters Sidebar | User applies filter | Table filters results | Ticker data | PENDING |
| Watchlist | Watchlist | Ticker Row | User hovers on ticker | Shows Quick Actions | Ticker data | `REFERENCE-DERIVED` |
| Broker Integration | Login Modal | Broker List | User opens login | Shows Zerodha option | N/A | APPROVED |
| Broker Integration | Login Modal | Broker List | User opens login | Shows Angel One option | N/A | APPROVED |
| Broker Integration | Login Modal | Broker List | User opens login | Shows ICICI Direct option | N/A | APPROVED |
