######################################################################################################################## 

############################################ PHASE 2

################################### ENTERPRISE IMPLEMENTATION BLUEPRINT

############################################## IMPL-006 v2.0

############################################ ANALYTICS ENGINE

######################################################################################################################## 

========================================================================================================================
==================================================== PART 1
============================================================
============================================== ANALYTICS ARCHITECTURE
==================================================
========================================================================================================================

TITLE

Analytics Engine (Go Edition)

DOCUMENT TYPE

Implementation Blueprint

STATUS

Approved

VERSION

2.0

PRIORITY

Critical

EXECUTION ORDER

IMPL-006

DEPENDENCIES

IMPL-001 v2.0 IMPL-002 v2.0 IMPL-003 v2.0 IMPL-005 v2.0

TECHNOLOGY

Go 1.25+ Gin GORM PGX PostgreSQL Redis Asynq gocron/v2 Uber Fx Zap Viper
Prometheus OpenTelemetry AWS S3

######################################################################################################################## 

MISSION

Analytics Engine shall transform validated market data into real-time,
deterministic and observable market intelligence.

The engine shall calculate

Market Sentiment

Premarket Gain

Day Gain

MoM Gain

Delta Volume

High Delta Volume Alerts

FII-related Metrics

Market Breadth

Index Analytics

Instrument Analytics

Aggregated Market Signals

######################################################################################################################## 

ANALYTICS FLOW

Market Data

↓

Validation

↓

Normalization

↓

Analytics Engine

↓

Feature Calculation

↓

Signal Calculation

↓

Aggregation

↓

Redis Cache

↓

PostgreSQL / S3

↓

WebSocket

↓

Frontend

######################################################################################################################## 

DESIGN PRINCIPLES

Analytics shall be

Deterministic

Stateless where possible

Idempotent

Concurrent

Low Latency

Observable

Testable

Recoverable

######################################################################################################################## 

ANALYTICS MODULES

internal/modules/analytics/

    sentiment/

    premarket/

    gain/

    volume/

    fii/

    breadth/

    indicators/

    aggregation/

    alerts/

    snapshots/

Every analytics module shall have a single responsibility.

######################################################################################################################## 

INPUT CONTRACT

Analytics Engine shall consume

Instrument

Open

High

Low

Close

Last Price

Volume

Average Price

Open Interest

Net Change

Total Buy Quantity

Total Sell Quantity

Previous Volume

Delta Volume

Fetch Timestamp

Trading Session

Provider Metadata

######################################################################################################################## 

OUTPUT CONTRACT

Analytics Engine shall produce

Calculated Metrics

Signals

Scores

Alerts

Snapshots

Analytics Events

Cache Objects

Persistence Objects

######################################################################################################################## 

ANALYTICS EXECUTION

Input

↓

Feature Extraction

↓

Metric Calculation

↓

Signal Calculation

↓

Score Aggregation

↓

Validation

↓

Publish

All calculations shall be reproducible.

######################################################################################################################## 

REALTIME PROCESSING

During market hours the engine shall support continuous updates.

Target trading window

09:15 IST

to

15:30 IST

Processing frequency shall follow the available market data update
frequency.

######################################################################################################################## 

MARKET SENTIMENT

Market Sentiment shall combine the configured sentiment inputs.

Core model

Market Sentiment

=

Premarket Component

-   

FII Component

-   

Configured Dynamic Signals

Final aggregation rules shall remain centralized.

######################################################################################################################## 

SIGNAL SCORE

Official score mapping

Strong Bullish = +2

Bullish = +1

Neutral = 0

Bearish = -1

Strong Bearish = -2

######################################################################################################################## 

AGGREGATE SCORE

Score \>= 3

Strong Bullish

Score \>= 1

Bullish

Score == 0

Neutral

Score \>= -3 AND \< 0

Bearish

Score \< -3

Strong Bearish

######################################################################################################################## 

ANALYTICS STATE

Realtime analytics state shall be maintained in Redis.

Persistent analytics shall be stored according to the relevant
database/S3 retention policy.

Redis shall not become the permanent source of truth.

######################################################################################################################## 

NEXT

PART 2

Feature Calculations

Premarket

Gain

Volume

OI

Market Sentiment

========================================================================================================================
==================================================== PART 2
============================================================
============================================ FEATURE CALCULATION
========================================================
========================================================================================================================

FEATURE ENGINE

Feature Engine shall calculate all reusable market features before
signal generation.

######################################################################################################################## 

PREMARKET GAIN

Premarket Gain shall be calculated once at the beginning of the trading
session.

Official calculation window

09:15 IST

Premarket Gain shall then be cached for the trading session.

It shall NOT be unnecessarily recalculated on every tick.

######################################################################################################################## 

PREMARKET INPUTS

Required values

Premarket/Open Price

Previous Close

Instrument

Trading Date

Missing values shall be handled explicitly.

Invalid numeric values shall not enter the calculation.

######################################################################################################################## 

GAIN CALCULATION

Day Gain

=

(Current Live Price / Previous Close) - 1

Result shall be represented according to the platform's configured
precision.

######################################################################################################################## 

MOM GAIN

Minute-over-Minute Gain

=

(Current Minute Live Price / Previous Minute Live Price) - 1

Previous value shall be maintained in the analytics state.

######################################################################################################################## 

VOLUME ANALYTICS

Volume analytics shall calculate

Current Volume

Previous Volume

Delta Volume

High Delta Volume

Volume Change

Volume Acceleration

######################################################################################################################## 

DELTA VOLUME

Delta Volume

=

Current Volume - Previous Volume

Negative values shall be handled according to the configured market-data
policy.

######################################################################################################################## 

HIGH DELTA VOLUME

The engine shall compare Delta Volume against the configured threshold.

Example baseline

DELTA_VOLUME_THRESHOLD

=

100000

Threshold shall remain configuration driven.

######################################################################################################################## 

OPEN INTEREST

OI analytics shall calculate

Current OI

Previous OI

OI Change

OI Change %

OI Day High

OI Day Low

######################################################################################################################## 

BUY/SELL PRESSURE

The engine shall consume

Total Buy Quantity

Total Sell Quantity

and derive configured market-pressure features.

######################################################################################################################## 

PREMARKET CACHE

Premarket result shall contain

Instrument

Trading Date

Premarket Value

Calculation Timestamp

Version

Status

Cache TTL

######################################################################################################################## 

FII COMPONENT

FII-related data shall be consumed according to the approved system
design.

If official FII data is available, the configured source shall be used.

If the project specification defines a derived FII metric from
market-book values, that calculation shall remain isolated inside the
FII module.

######################################################################################################################## 

MARKET SENTIMENT PIPELINE

Premarket

↓

FII

↓

Dynamic Market Signals

↓

Signal Scores

↓

Aggregation

↓

Final Sentiment

######################################################################################################################## 

SENTIMENT OUTPUT

Output shall contain

Score

Label

Premarket Contribution

FII Contribution

Dynamic Contribution

Calculation Timestamp

Version

######################################################################################################################## 

INDEX ANALYTICS

Engine shall support

NIFTY

SENSEX

Other configured indices

Index analytics shall use the same standardized feature pipeline.

######################################################################################################################## 

INSTRUMENT ANALYTICS

Each instrument may produce

Price Metrics

Volume Metrics

OI Metrics

Momentum Metrics

Sentiment Contribution

Alert State

######################################################################################################################## 

NULL HANDLING

Analytics shall explicitly handle

Missing

NA

Null

Zero

Invalid

Infinite

values.

Invalid values shall never cause an application panic.

######################################################################################################################## 

PRECISION

Financial calculations shall use deterministic precision.

Money-related persistence shall use PostgreSQL NUMERIC/ DECIMAL where
applicable.

Floating point calculations shall be controlled and tested.

######################################################################################################################## 

NEXT

PART 3

Signal Engine

Sentiment Aggregation

Alerts

Ranking

Scoring

========================================================================================================================
==================================================== PART 3
============================================================
============================================== SIGNAL ENGINE
===========================================================
========================================================================================================================

SIGNAL ENGINE

Signal Engine shall transform calculated features into standardized
market signals.

######################################################################################################################## 

SIGNAL FLOW

Features

↓

Rules

↓

Individual Signals

↓

Scores

↓

Aggregation

↓

Final Signal

######################################################################################################################## 

SIGNAL TYPES

Supported signals

Bullish

Bearish

Strong Bullish

Strong Bearish

Neutral

Volume Alert

Momentum Alert

OI Alert

Market Pressure Alert

######################################################################################################################## 

SIGNAL SCORE

Every signal shall map to a numeric score.

Strong Bullish

+2

Bullish

+1

Neutral

0

Bearish

-1

Strong Bearish

-2

######################################################################################################################## 

SIGNAL RULES

Rules shall remain

Centralized

Versioned

Testable

Configuration Driven

Business logic shall not be duplicated across handlers.

######################################################################################################################## 

AGGREGATION

Individual instrument signals

↓

Component Scores

↓

Aggregate Score

↓

Sentiment Classification

Aggregation shall be deterministic.

######################################################################################################################## 

SENTIMENT CLASSIFICATION

Aggregate score mapping

> = 3

Strong Bullish

> = 1

Bullish

== 0

Neutral

> = -3

Bearish

\< -3

Strong Bearish

Boundary conditions shall be covered by automated tests.

######################################################################################################################## 

ALERT ENGINE

Alert Engine shall generate

High Delta Volume

OI Change

Price Movement

Momentum

Market Sentiment

Configured Threshold Alerts

######################################################################################################################## 

HIGH DELTA VOLUME ALERT

When

Delta Volume \>= Threshold

the engine shall generate High Delta Volume Alert.

Alert shall contain

Instrument

Delta Volume

Threshold

Timestamp

Severity

Alert Type

######################################################################################################################## 

ALERT DEDUPLICATION

Identical alerts shall not be generated repeatedly within the configured
deduplication window.

Redis may maintain the deduplication state.

######################################################################################################################## 

RANKING ENGINE

Ranking shall support

Top Gainers

Top Losers

Highest Volume

Highest Delta Volume

Highest OI

Highest OI Change

Strongest Bullish

Strongest Bearish

######################################################################################################################## 

RANKING RULES

Ranking shall define

Primary Metric

Secondary Metric

Sort Direction

Tie Breaker

Timestamp

Results shall be deterministic.

######################################################################################################################## 

MARKET BREADTH

Breadth analytics may include

Advancing

Declining

Unchanged

Advance/Decline Ratio

Breadth Percentage

Breadth shall be calculated from validated instrument data.

######################################################################################################################## 

AGGREGATION LEVELS

Instrument

↓

Sector (Future)

↓

Index

↓

Market

Analytics architecture shall support additional aggregation levels.

######################################################################################################################## 

VERSIONING

Every analytics result shall carry

Algorithm Version

Configuration Version

Calculation Timestamp

Trading Date

This allows historical recalculation and debugging.

######################################################################################################################## 

DETERMINISM

Same input

-   

Same configuration

-   

Same algorithm version

=

Same output

Determinism is mandatory.

######################################################################################################################## 

NEXT

PART 4

Realtime Analytics

Redis

Events

WebSocket

S3

PostgreSQL

========================================================================================================================
==================================================== PART 4
============================================================
========================================== REALTIME ANALYTICS
==========================================================
========================================================================================================================

REALTIME ENGINE

Analytics shall process incoming market updates with minimum unnecessary
latency.

######################################################################################################################## 

REALTIME FLOW

Market Update

↓

Analytics Worker

↓

Feature Calculation

↓

Signal Calculation

↓

Aggregation

↓

Redis Update

↓

Analytics Event

↓

WebSocket

######################################################################################################################## 

ASYNC PROCESSING

Heavy analytics workloads shall use

Asynq

Workers shall support

Retries

Concurrency

Timeout

Dead Letter Queue

Priority

######################################################################################################################## 

REDIS ANALYTICS CACHE

Redis shall maintain

Latest Analytics

Latest Sentiment

Latest Rankings

Latest Alerts

Latest Market Breadth

Latest Instrument Metrics

######################################################################################################################## 

CACHE KEY STANDARD

Keys shall follow a consistent namespace.

Example

analytics:instrument:{id}

analytics:sentiment:{market}

analytics:ranking:{type}

analytics:alert:{id}

Key naming shall remain versioned where necessary.

######################################################################################################################## 

CACHE INVALIDATION

Cache shall update when

New Market Data

New Analytics Result

Trading Session Change

Configuration Change

Algorithm Version Change

######################################################################################################################## 

ANALYTICS EVENTS

Events shall include

AnalyticsUpdated

SentimentUpdated

RankingUpdated

AlertGenerated

BreadthUpdated

SnapshotGenerated

######################################################################################################################## 

EVENT STRUCTURE

Every event shall contain

Event ID

Event Type

Timestamp

Trading Date

Instrument (When Applicable)

Algorithm Version

Payload

Correlation ID

######################################################################################################################## 

EVENT IDEMPOTENCY

Every event shall support duplicate detection.

Event IDs shall be unique.

Consumers shall safely process repeated events.

######################################################################################################################## 

WEBSOCKET

Analytics results shall be published to WebSocket clients through the
approved WebSocket layer.

Analytics Engine shall not manage raw WebSocket connections.

######################################################################################################################## 

WEBSOCKET FLOW

Analytics Event

↓

Redis Pub/Sub

↓

WebSocket Hub

↓

Subscribed Clients

Analytics Engine shall remain decoupled from client sessions.

######################################################################################################################## 

POSTGRESQL

PostgreSQL shall store persistent analytics metadata and records
required by the application.

High-frequency transient state shall remain in Redis where appropriate.

######################################################################################################################## 

AWS S3

S3 shall store

Historical Analytics

Daily Snapshots

Large Analytics Datasets

Parquet Outputs

Reports

S3 objects shall be versioned and partitioned according to the storage
standard.

######################################################################################################################## 

PARQUET

Analytics Parquet shall support

Trading Date

Market Segment

Instrument

Timestamp

Algorithm Version

Calculated Metrics

Files shall remain immutable after finalized.

######################################################################################################################## 

SNAPSHOTS

Analytics snapshots shall capture

Timestamp

Market Sentiment

Breadth

Top Gainers

Top Losers

Volume Leaders

OI Leaders

Configured Market Metrics

######################################################################################################################## 

SNAPSHOT FREQUENCY

Snapshot frequency shall be configuration driven.

High-frequency snapshots shall remain in cache.

Finalized snapshots shall be persisted to durable storage.

######################################################################################################################## 

CONSISTENCY

Analytics shall guarantee

Input Consistency

Calculation Consistency

Cache Consistency

Event Consistency

Persistence Consistency

######################################################################################################################## 

FAILURE ISOLATION

Failure in

WebSocket

S3

Redis

Analytics Persistence

shall not corrupt the core calculation pipeline.

Recovery shall be handled independently.

######################################################################################################################## 

NEXT

PART 5

Reliability

Performance

Recovery

Observability

Operational Controls

========================================================================================================================
==================================================== PART 5
============================================================
========================================= RELIABILITY & OBSERVABILITY
==================================================
========================================================================================================================

RELIABILITY

Analytics Engine shall tolerate

Invalid Data

Provider Failure

Redis Failure

Database Failure

Queue Failure

WebSocket Failure

Temporary Storage Failure

######################################################################################################################## 

FAILURE CLASSIFICATION

Errors shall be classified as

Validation

Transient

Retryable

Permanent

Configuration

Infrastructure

Calculation

Unknown

######################################################################################################################## 

RETRY POLICY

Retryable analytics jobs shall support

Exponential Backoff

Jitter

Maximum Attempts

Timeout

Dead Letter Queue

######################################################################################################################## 

DEAD LETTER QUEUE

Failed analytics jobs shall be routed to Asynq dead-letter processing.

DLQ records shall contain

Task ID

Error

Attempt Count

Timestamp

Payload Reference

Algorithm Version

######################################################################################################################## 

RECOVERY

Recovery shall support

Job Retry

Cache Rebuild

Snapshot Rebuild

Analytics Replay

Historical Recalculation

Event Replay

######################################################################################################################## 

REPLAY

Historical analytics may be replayed using

Stored Market Data

Algorithm Version

Configuration Version

Trading Date

Replay shall never overwrite production data without an explicit
operation.

######################################################################################################################## 

CHECKPOINTS

Long-running analytics processing may use checkpoints.

Checkpoint shall contain

Trading Date

Instrument

Timestamp

Processing Position

Algorithm Version

######################################################################################################################## 

PERFORMANCE

Analytics shall optimize for

Low Latency

High Throughput

Low Memory Allocation

Controlled Concurrency

Minimal Database Calls

Minimal Redis Round Trips

######################################################################################################################## 

CONCURRENCY

Go concurrency shall use

Goroutines

Channels

Worker Pools

Context Cancellation

Concurrency limits shall remain configurable.

######################################################################################################################## 

RACE SAFETY

Shared mutable state shall be minimized.

Race-prone state shall use

Mutex

Atomic Operations

Channel Ownership

Immutable Data

Race detection shall run in CI.

######################################################################################################################## 

PERFORMANCE METRICS

Measure

Calculation Latency

Queue Latency

Redis Latency

Database Latency

Event Latency

End-to-End Latency

Records/Second

Memory Usage

CPU Usage

######################################################################################################################## 

OBSERVABILITY

Analytics Engine shall expose

Prometheus Metrics

OpenTelemetry Traces

Structured Logs

Health Checks

Audit Events

######################################################################################################################## 

METRICS

Metrics shall include

analytics_records_total

analytics_errors_total

analytics_duration_seconds

analytics_queue_depth

analytics_cache_hits_total

analytics_cache_misses_total

analytics_alerts_total

analytics_events_total

analytics_replay_total

######################################################################################################################## 

TRACING

Analytics traces shall include

Request ID

Correlation ID

Trace ID

Instrument

Trading Date

Algorithm Version

Calculation Duration

Storage Duration

Publishing Duration

######################################################################################################################## 

LOGGING

Structured logs shall contain

Timestamp

Level

Module

Operation

Instrument

Trading Date

Algorithm Version

Duration

Status

Error

Sensitive data shall never be logged.

######################################################################################################################## 

HEALTH

Analytics health shall verify

Worker Status

Redis

PostgreSQL

Queue

Configuration

Algorithm Registry

Health state

Healthy

Degraded

Critical

######################################################################################################################## 

ALERTING

Alerts shall trigger on

High Error Rate

High Calculation Latency

Queue Backlog

Redis Failure

Database Failure

Repeated Calculation Failure

Missing Analytics Updates

######################################################################################################################## 

CAPACITY

Capacity planning shall monitor

Instruments/Minute

Calculations/Second

Concurrent Workers

Queue Depth

Redis Memory

Database Writes

S3 Growth

######################################################################################################################## 

OPERATIONAL SAFETY

Analytics configuration changes shall be

Versioned

Validated

Audited

Rollbackable

######################################################################################################################## 

NEXT

PART 6

Testing

Quality Gates

Acceptance Criteria

Implementation Checklist

========================================================================================================================
==================================================== PART 6
============================================================
=============================================== TESTING & ACCEPTANCE
====================================================
========================================================================================================================

TESTING STANDARD

Every analytics component shall be tested before deployment.

######################################################################################################################## 

UNIT TESTING

Unit tests shall cover

Premarket Calculation

Day Gain

MoM Gain

Delta Volume

OI Change

Signal Mapping

Sentiment Aggregation

Ranking

Breadth

Alert Rules

######################################################################################################################## 

BOUNDARY TESTING

Tests shall cover

Zero

Negative

Missing

NA

Maximum Values

Minimum Values

Threshold Boundaries

Floating Precision

######################################################################################################################## 

SENTIMENT TESTING

Verify

+2

+1

0

-1

-2

and aggregate boundaries

> = 3

> = 1

0

> = -3

\< -3

######################################################################################################################## 

INTEGRATION TESTING

Integration tests shall cover

Redis

PostgreSQL

Asynq

S3

Market Data Module

WebSocket Event Pipeline

######################################################################################################################## 

REALTIME TESTING

Verify

Market Update

↓

Analytics

↓

Redis

↓

Event

↓

WebSocket

End-to-end propagation shall be tested.

######################################################################################################################## 

CONCURRENCY TESTING

Test

Parallel Instruments

Parallel Calculations

Concurrent Cache Updates

Concurrent Events

Worker Pool Saturation

Context Cancellation

######################################################################################################################## 

PERFORMANCE TESTING

Measure

Calculation Latency

Throughput

Memory

CPU

Redis Latency

Database Latency

Event Latency

######################################################################################################################## 

LOAD TESTING

Test scenarios shall include

Thousands of Instruments

Peak Market Load

High Volume Updates

Large Analytics Batches

Concurrent Consumers

######################################################################################################################## 

FAILURE TESTING

Test

Redis Down

Database Down

Queue Down

S3 Failure

Invalid Input

Provider Failure

Worker Crash

Event Failure

Recovery shall be verified.

######################################################################################################################## 

REPLAY TESTING

Historical replay shall verify

Deterministic Output

Version Compatibility

Duplicate Prevention

Snapshot Integrity

######################################################################################################################## 

RACE TESTING

Required command

go test -race

All analytics packages shall pass race detection.

######################################################################################################################## 

BENCHMARKING

Required command

go test -bench

Critical calculations shall have benchmarks.

######################################################################################################################## 

SECURITY TESTING

Verify

Input Validation

Authorization

Sensitive Data Handling

Configuration Security

Secret Protection

Audit Logging

######################################################################################################################## 

CODE QUALITY

Required

gofmt

goimports

go vet

staticcheck

golangci-lint

govulncheck

######################################################################################################################## 

COVERAGE

Minimum

90%

Critical calculation logic

100%

Signal mapping

100%

Sentiment aggregation

100%

Alert rules

100%

######################################################################################################################## 

CI QUALITY GATES

CI shall fail when

Tests Fail

Coverage Fails

Race Detection Fails

Static Analysis Fails

Security Scan Fails

Benchmark Regression exceeds configured threshold

######################################################################################################################## 

IMPLEMENTATION CHECKLIST

✓ Analytics architecture defined

✓ Feature engine implemented

✓ Premarket calculation implemented

✓ Day Gain implemented

✓ MoM Gain implemented

✓ Delta Volume implemented

✓ OI analytics implemented

✓ Signal engine implemented

✓ Sentiment aggregation implemented

✓ Ranking engine implemented

✓ Breadth engine implemented

✓ Alert engine implemented

✓ Redis analytics cache implemented

✓ Asynq processing implemented

✓ Analytics events implemented

✓ WebSocket integration implemented

✓ PostgreSQL integration implemented

✓ S3 integration implemented

✓ Replay support implemented

✓ Recovery implemented

✓ Prometheus metrics enabled

✓ OpenTelemetry enabled

✓ Structured logging enabled

✓ Unit tests completed

✓ Integration tests completed

✓ Load tests completed

✓ Race detection passed

✓ CI quality gates configured

######################################################################################################################## 

GENERATED ARTIFACTS

Analytics Engine

Feature Engine

Signal Engine

Sentiment Engine

Ranking Engine

Breadth Engine

Alert Engine

Analytics Cache

Analytics Event Contracts

Replay Engine

Recovery Framework

Analytics Test Suite

Performance Benchmarks

Operational Dashboards

######################################################################################################################## 

ACCEPTANCE CRITERIA

IMPL-006 shall be considered complete only when

All calculations are deterministic

All approved formulas are implemented

Sentiment mapping is correct

Realtime analytics are processed

Redis state is updated

Analytics events are published

Historical analytics can be persisted

Failures are recoverable

Tests pass

Race detection passes

Quality gates pass

Observability is enabled

######################################################################################################################## 

PHASE COMPLETION

IMPLEMENTATION

IMPL-006 v2.0

STATUS

COMPLETED

READINESS

APPROVED

TECHNOLOGY BASELINE

Go Enterprise Stack

######################################################################################################################## 

NEXT DOCUMENT

IMPL-007 v2.0

Realtime WebSocket & Event Distribution (Go Edition)

######################################################################################################################## 

END OF IMPL-006 v2.0

######################################################################################################################## 
