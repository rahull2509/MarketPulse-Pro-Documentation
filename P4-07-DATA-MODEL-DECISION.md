# P4-07 Data Model Decision

This document defines the approved generic database schemas required to support the Multi-Broker Architecture (P4-07).

## 1. BrokerSession
Maintains the OAuth state and active session details for a user's broker connection.
* **Status**: APPROVED
* **Fields**:
  - `id` (UUID, Primary Key)
  - `user_id` (BIGINT, Foreign Key to `users`)
  - `broker_name` (VARCHAR, e.g., 'zerodha', 'angel_one')
  - `broker_user_id` (VARCHAR, provider's identifier for the account)
  - `access_token` (TEXT, Encrypted at rest)
  - `refresh_token` (TEXT, Encrypted at rest, Nullable)
  - `expires_at` (TIMESTAMP WITH TIME ZONE)
  - `created_at` (TIMESTAMP WITH TIME ZONE)
  - `updated_at` (TIMESTAMP WITH TIME ZONE)
* **Constraints**: `UNIQUE (user_id, broker_name)`

## 2. BrokerOrder
Generic abstraction for orders placed through the Broker Integration Layer.
* **Status**: APPROVED
* **Fields**:
  - `id` (UUID, Primary Key)
  - `user_id` (BIGINT, Foreign Key to `users`)
  - `broker_name` (VARCHAR)
  - `broker_order_id` (VARCHAR, Provider's order identifier)
  - `internal_symbol` (VARCHAR, mapped to P4-04 standard)
  - `broker_symbol` (VARCHAR)
  - `side` (VARCHAR, 'BUY', 'SELL')
  - `order_type` (VARCHAR, 'MARKET', 'LIMIT', 'SL', etc.)
  - `quantity` (INTEGER)
  - `filled_quantity` (INTEGER)
  - `price` (DECIMAL, for limit/SL)
  - `avg_execution_price` (DECIMAL)
  - `status` (VARCHAR, 'OPEN', 'COMPLETED', 'REJECTED', 'CANCELLED')
  - `status_message` (TEXT, Error/rejection details)
  - `placed_at` (TIMESTAMP WITH TIME ZONE)
  - `updated_at` (TIMESTAMP WITH TIME ZONE)
* **Constraints**: `UNIQUE (broker_name, broker_order_id)`

## 3. BrokerPosition
Generic abstraction for real-time intra-day and carry-forward positions.
* **Status**: APPROVED
* **Fields**:
  - `id` (UUID, Primary Key)
  - `user_id` (BIGINT, Foreign Key to `users`)
  - `broker_name` (VARCHAR)
  - `internal_symbol` (VARCHAR)
  - `quantity` (INTEGER, positive for long, negative for short)
  - `average_price` (DECIMAL)
  - `realized_pnl` (DECIMAL)
  - `unrealized_pnl` (DECIMAL, typically derived but cached here)
  - `product_type` (VARCHAR, 'MIS', 'CNC', 'NRML')
  - `updated_at` (TIMESTAMP WITH TIME ZONE)
* **Constraints**: `UNIQUE (user_id, broker_name, internal_symbol, product_type)`

## 4. BrokerHolding
Generic abstraction for long-term equity/asset holdings.
* **Status**: APPROVED
* **Fields**:
  - `id` (UUID, Primary Key)
  - `user_id` (BIGINT, Foreign Key to `users`)
  - `broker_name` (VARCHAR)
  - `internal_symbol` (VARCHAR)
  - `isin` (VARCHAR)
  - `quantity` (INTEGER)
  - `average_price` (DECIMAL)
  - `updated_at` (TIMESTAMP WITH TIME ZONE)
* **Constraints**: `UNIQUE (user_id, broker_name, isin)`

## 5. BrokerInstrumentMapping
Cache for resolving internal symbols to provider-specific instrument tokens.
* **Status**: APPROVED
* **Fields**:
  - `broker_name` (VARCHAR)
  - `internal_symbol` (VARCHAR)
  - `broker_symbol` (VARCHAR)
  - `broker_token` (VARCHAR)
  - `exchange` (VARCHAR, 'NSE', 'NFO', 'BSE')
  - `updated_at` (TIMESTAMP WITH TIME ZONE)
* **Constraints**: `PRIMARY KEY (broker_name, internal_symbol)`
