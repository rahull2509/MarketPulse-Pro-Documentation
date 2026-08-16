######################################################################################################################## 
# SPEC_011: STRATEGY BUILDER
######################################################################################################################## 

## DOCUMENT INFORMATION
Directive ID: SPEC-011
Document Name: Strategy Builder Specification
Priority: Critical
Status: APPROVED (Conditionally based on DEC-ARCH resolutions)
Dependencies: `Volume_2_Requirements.md`, `Volume_5_Implementation_Architecture.md`

######################################################################################################################## 
## 1. SCOPE & RESPONSIBILITIES
The Strategy Builder module provides tools for users to construct, analyze, and save multi-leg options strategies.

######################################################################################################################## 
## 2. FUNCTIONAL REQUIREMENTS

### 2.1 Strategy Construction
- **Underlying Selection**: The system must allow users to select an underlying asset. [CONFIRMED REQUIREMENT]
- **Expiry Selection**: The system must allow selection of valid expiries for the underlying. [CONFIRMED REQUIREMENT]
- **Strike Selection**: The system must present available strikes around ATM. [CONFIRMED REQUIREMENT]
- **Option Leg Construction**: Users must be able to add multiple legs (CE/PE, Buy/Sell, Quantity). [CONFIRMED REQUIREMENT]
- **Draft/Save Behavior**: Users must be able to save constructed strategies as drafts. [CONFIRMED REQUIREMENT]

### 2.2 Strategy Analysis & Visualization
- **Payoff Visualization**: The system must display a payoff chart indicating profit and loss across underlying prices. [CONFIRMED REQUIREMENT]
- **Profit/Loss Analysis**: The system must calculate Max Profit, Max Loss, and Risk/Reward ratio. [CONFIRMED REQUIREMENT]
- **Breakeven**: The system must calculate and display breakeven points. [CONFIRMED REQUIREMENT]
- **Greeks**: The system must calculate and display aggregate strategy Greeks (Delta, Gamma, Theta, Vega). [CONFIRMED REQUIREMENT]
- **IV Charts**: The system must display IV percentile/rank context. [REFERENCE-DERIVED REQUIREMENT]

### 2.3 User Flows & States
- **Loading State**: The system must display skeleton loaders during underlying/option chain fetch. [REFERENCE-DERIVED REQUIREMENT]
- **Empty State**: Prompts the user to "Add Leg" or select from pre-built strategies. [REFERENCE-DERIVED REQUIREMENT]
- **Error State**: Displays validation errors (e.g., "Invalid Quantity", "Expiry Mismatch"). [PROPOSED BEHAVIOR]

######################################################################################################################## 
## 3. DATA & API REQUIREMENTS

### 3.1 Data Required
- Realtime LTP, Bid/Ask, and IV for selected legs.
- End-of-day settlement prices for P&L calculations.

### 3.2 API Required
- `GET /api/v1/market/instruments`: Fetch available underlyings.
- `GET /api/v1/market/options/chain`: Fetch strikes for expiry.
- `POST /api/v1/user/strategies`: Save draft strategy.

######################################################################################################################## 
## 4. VALIDATION & SECURITY
- **Validation**: Max 10 legs per strategy. Cannot mix different underlyings in a single strategy. [PROPOSED BEHAVIOR]
- **Security**: Draft saving requires an authenticated session. [CONFIRMED REQUIREMENT]

######################################################################################################################## 
## 5. PERFORMANCE & TESTING
- **Performance**: Realtime P&L updates must calculate within 50ms upon receiving WebSocket ticks. [CONFIRMED REQUIREMENT]
- **Testing**: Requires unit testing for Greek/Payoff math, and integration testing for save strategy flows. [CONFIRMED REQUIREMENT]
