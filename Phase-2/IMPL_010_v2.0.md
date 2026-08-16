######################################################################################################################## 

############################################ PHASE 2

################################### ENTERPRISE IMPLEMENTATION BLUEPRINT

############################################## IMPL-010 v2.0

######################################## EXTERNAL INTEGRATION LAYER

######################################################################################################################## 

========================================================================================================================
==================================================== PART 1
============================================================
=========================================== INTEGRATION ARCHITECTURE
===================================================
========================================================================================================================

MISSION

IMPL-010 defines the centralized external integration architecture for
MarketPulse Pro.

The system shall integrate with

Market Data Providers

AWS Services

Email Providers

Storage Providers

Authentication Providers

Monitoring Services

Future External APIs

######################################################################################################################## 

ARCHITECTURE

Application

↓

Integration Gateway

↓

Provider Registry

↓

Provider Adapter

↓

Authentication

↓

External Provider

↓

Response

↓

Normalization

↓

Business Layer

↓

Audit / Metrics

######################################################################################################################## 

CORE PRINCIPLE

Business logic shall never depend directly on provider-specific APIs.

Business Layer

≠

Provider Implementation

######################################################################################################################## 

PROVIDER ABSTRACTION

Every external integration shall provide a standardized interface.

Provider

    Name()

    HealthCheck()

    Authenticate()

    Execute()

    Close()

Provider-specific implementations shall remain isolated.

######################################################################################################################## 

PROVIDER REGISTRY

Registry shall manage

Provider ID

Provider Type

Provider Instance

Configuration

Health State

Capabilities

Version

Status

######################################################################################################################## 

SUPPORTED INTEGRATIONS

Market Data

Upstox

Storage

AWS S3

Email

AWS SES / Configured Email Provider

Monitoring

AWS CloudWatch where required

Secrets

AWS Secrets Manager where required

######################################################################################################################## 

REQUEST FLOW

Business Service

↓

Integration Interface

↓

Provider Registry

↓

Adapter

↓

Authentication

↓

Provider Request

↓

Response

↓

Normalization

######################################################################################################################## 

RESPONSE NORMALIZATION

External provider responses shall be converted into internal DTOs.

Internal systems shall not consume provider-specific response models.

######################################################################################################################## 

TIMEOUT

Every external request shall have

Connection Timeout

Request Timeout

Context Deadline

######################################################################################################################## 

RETRY

Retry shall support

Exponential Backoff

Jitter

Maximum Attempts

Retryable Error Classification

######################################################################################################################## 

CIRCUIT BREAKER

Provider failures shall support

CLOSED

OPEN

HALF_OPEN

Circuit breaker state shall be observable.

######################################################################################################################## 

FAILURE ISOLATION

Failure of one provider shall not stop unrelated integrations.

######################################################################################################################## 

OBSERVABILITY

Every integration shall expose

Metrics

Logs

Traces

Health

Audit

######################################################################################################################## 

NEXT

PART 2

Provider Adapters

Authentication

Upstox

AWS

Email

========================================================================================================================
==================================================== PART 2
============================================================
=========================================== PROVIDER ADAPTERS
==========================================================
========================================================================================================================

UPSTOX ADAPTER

> [!WARNING] [LEGACY MARKETPULSE CONFLICT]
> This adapter design conflicts with the current MarketPulse Pro UI requirement. 
> The approved Sensibull UX reference strictly mandates **Zerodha, Angel One, and ICICI Direct**.
> **Upstox is no longer the approved broker for MarketPulse Pro.** See `DEC-ARCH-004A` and `DEC-ARCH-004B` in `DECISION_REGISTER.md`.

Upstox integration shall support

Authentication

Token Refresh

Market Data

Instrument Data

Provider Health

Rate Limits

Connection Recovery

######################################################################################################################## 

UPSTOX AUTHENTICATION

Authentication flow

Credentials

↓

Authorization

↓

Access Token

↓

Token Validation

↓

Provider Requests

Token secrets shall never be logged.

######################################################################################################################## 

TOKEN REFRESH

Expired token

↓

Refresh

↓

Validate

↓

Update Secure Store

↓

Resume Requests

Failed refresh shall generate an operational alert.

######################################################################################################################## 

AWS S3 ADAPTER

S3 adapter shall support

Upload

Download

Exists

Delete (only authorized workflows)

Metadata

Health Check

Multipart Upload where required

######################################################################################################################## 

S3 OBJECT POLICY

Objects shall use

Environment

Trading Date

Market Segment

Data Type

Version

Partitioning shall remain consistent.

######################################################################################################################## 

EMAIL ADAPTER

Email adapter shall support

Recipient

Subject

Template

HTML

Plain Text

Provider Response

Delivery Status

######################################################################################################################## 

AWS SES

If SES is enabled, SES-specific logic shall remain inside the SES
adapter.

Core notification code shall depend only on the provider interface.

######################################################################################################################## 

CLOUDWATCH

CloudWatch integration may expose

Application Metrics

Operational Events

Infrastructure Signals

Existing Prometheus/Grafana observability remains the primary
application monitoring architecture.

######################################################################################################################## 

SECRETS MANAGER

Secrets Manager integration shall support

Secret Retrieval

Version

Rotation Awareness

Health

Secrets shall be cached only according to approved security policy.

######################################################################################################################## 

RATE LIMITS

Provider adapters shall enforce provider-specific rate limits.

Rate-limit errors shall be classified separately.

######################################################################################################################## 

ERROR MAPPING

Provider errors shall map to

TIMEOUT

RATE_LIMIT

AUTHENTICATION

VALIDATION

NOT_FOUND

SERVER_ERROR

NETWORK

UNKNOWN

Business layer shall receive normalized errors.

######################################################################################################################## 

NEXT

PART 3

Integration Gateway

Configuration

Request Management

Normalization

========================================================================================================================
==================================================== PART 3
============================================================
============================================ INTEGRATION GATEWAY
=======================================================
========================================================================================================================

GATEWAY

Integration Gateway shall provide a common entry point for external
provider communication.

######################################################################################################################## 

RESPONSIBILITIES

Gateway shall manage

Provider Selection

Request Context

Timeout

Retry

Circuit Breaker

Authentication

Metrics

Tracing

Error Mapping

######################################################################################################################## 

REQUEST CONTEXT

Every request shall contain

Request ID

Correlation ID

Trace ID

Provider

Operation

Timestamp

Deadline

######################################################################################################################## 

CONFIGURATION

Viper shall manage

Provider URL

Credentials Reference

Timeout

Retry

Rate Limit

Circuit Breaker

Environment

Region

Feature Flags

######################################################################################################################## 

CONFIGURATION VALIDATION

Application startup shall reject

Missing Required Configuration

Invalid URL

Invalid Timeout

Invalid Retry Count

Invalid Provider

Invalid Credentials Reference

######################################################################################################################## 

NORMALIZATION

Provider Response

↓

Adapter DTO

↓

Internal DTO

↓

Business Model

Provider-specific fields shall not leak into core business logic.

######################################################################################################################## 

IDEMPOTENCY

External operations shall use idempotency where supported.

Examples

Upload

Notification

Payment-like future integrations

Critical commands shall avoid duplicate execution.

######################################################################################################################## 

AUDIT

External requests shall record

Provider

Operation

Status

Latency

Error Classification

Correlation ID

Sensitive request payloads shall not be stored unnecessarily.

######################################################################################################################## 

HEALTH CHECK

Every provider shall expose

Connectivity

Authentication

Latency

Availability

######################################################################################################################## 

PROVIDER REGISTRY STATE

AVAILABLE

DEGRADED

UNAVAILABLE

AUTHENTICATION_FAILED

RATE_LIMITED

UNKNOWN

######################################################################################################################## 

NEXT

PART 4

Reliability

Retry

Circuit Breaker

Fallback

Recovery

========================================================================================================================
==================================================== PART 4
============================================================
====================================== RELIABILITY & RECOVERY
=========================================================
========================================================================================================================

RETRYABLE

Retry

Timeout

Temporary Network Error

Provider 5xx

Temporary AWS Error

Rate Limit

NON-RETRYABLE

Invalid Credentials

Invalid Request

Invalid Configuration

Permission Denied

Permanent Validation Error

######################################################################################################################## 

BACKOFF

Retry shall use

Exponential Backoff

Jitter

Maximum Attempts

Maximum Delay

######################################################################################################################## 

CIRCUIT BREAKER

Repeated failures

↓

OPEN

↓

Stop Requests

↓

Cooldown

↓

HALF_OPEN

↓

Health Request

↓

CLOSED

######################################################################################################################## 

FALLBACK

Fallback shall only be used where an approved alternative provider
exists.

No automatic fallback shall change business semantics.

######################################################################################################################## 

RECOVERY

Provider unavailable

↓

Detect

↓

Alert

↓

Retry

↓

Health Check

↓

Recover

↓

Resume

######################################################################################################################## 

S3 RECOVERY

Upload failure

↓

Retry

↓

Verify Object

↓

Record Success

Incomplete objects shall not be marked finalized.

######################################################################################################################## 

EMAIL RECOVERY

Delivery failure

↓

Classify

↓

Retry

↓

Alternative Provider (if configured)

↓

DLQ

######################################################################################################################## 

AUTHENTICATION RECOVERY

Token failure

↓

Refresh

↓

Validate

↓

Retry Original Request

Authentication failure after refresh shall become a critical provider
error.

######################################################################################################################## 

GRACEFUL SHUTDOWN

Stop New Requests

↓

Finish Safe Requests

↓

Close Provider Connections

↓

Flush Metrics

↓

Shutdown

######################################################################################################################## 

NEXT

PART 5

Security

Observability

Operations

========================================================================================================================
==================================================== PART 5
============================================================
======================================= SECURITY & OBSERVABILITY
=======================================================
========================================================================================================================

SECURITY

Integrations shall enforce

TLS

Secret Protection

Least Privilege

Input Validation

Authorization

Audit

######################################################################################################################## 

SECRETS

Secrets shall come from

Secure Environment

Secrets Manager

Approved Secret Store

Never from source code.

######################################################################################################################## 

LOGGING

Never log

Passwords

Tokens

API Keys

AWS Secrets

Authorization Headers

Sensitive Provider Payloads

######################################################################################################################## 

METRICS

integration_requests_total

integration_success_total

integration_failures_total

integration_retries_total

integration_latency_seconds

integration_rate_limit_total

integration_circuit_open_total

######################################################################################################################## 

TRACING

Trace

Gateway

Adapter

Provider Request

Provider Response

Normalization

Persistence

######################################################################################################################## 

ALERTS

Provider Down

Authentication Failure

High Error Rate

High Latency

Rate Limit

Circuit Open

Repeated S3 Failure

Email Provider Failure

######################################################################################################################## 

DASHBOARD

Provider Availability

Request Rate

Latency

Failures

Retries

Circuit State

Rate Limits

S3 Status

Email Status

######################################################################################################################## 

AUDIT

Provider Configuration

Credential Rotation

Manual Retry

Provider Disable

Provider Enable

Administrative Operations

shall be audited.

######################################################################################################################## 

NEXT

PART 6

Testing

Quality Gates

Acceptance

========================================================================================================================
==================================================== PART 6
============================================================
=============================================== TESTING & ACCEPTANCE
====================================================
========================================================================================================================

TESTING

Unit tests shall cover

Adapters

Gateway

Error Mapping

Retry

Circuit Breaker

Normalization

Configuration

######################################################################################################################## 

INTEGRATION TESTS

Test

Upstox

S3

Email

Secrets

Provider Health

######################################################################################################################## 

FAILURE TESTS

Test

Timeout

Network Failure

Authentication Failure

Rate Limit

5xx

S3 Failure

Email Failure

######################################################################################################################## 

SECURITY TESTS

Test

Secret Leakage

Authorization

TLS

Invalid Credentials

Input Validation

######################################################################################################################## 

PERFORMANCE

Measure

Latency

Throughput

Retries

Connection Usage

Memory

CPU

######################################################################################################################## 

RACE

go test -race

shall pass.

######################################################################################################################## 

CODE QUALITY

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

Critical Integration Logic

100%

######################################################################################################################## 

IMPLEMENTATION CHECKLIST

✓ Integration Gateway ✓ Provider Registry ✓ Upstox Adapter ✓ S3 Adapter
✓ Email Adapter ✓ Authentication ✓ Token Refresh ✓ Retry ✓ Timeout ✓
Circuit Breaker ✓ Error Mapping ✓ Normalization ✓ Health Checks ✓
Metrics ✓ Tracing ✓ Audit ✓ Security ✓ Recovery ✓ Tests ✓ CI Quality
Gates

######################################################################################################################## 

ACCEPTANCE

All external integrations shall

Be isolated

Be testable

Be observable

Be recoverable

Be secure

Be replaceable

######################################################################################################################## 

STATUS

IMPL-010 v2.0 COMPLETED

NEXT

IMPL-011 v2.0

Testing & Quality Assurance

######################################################################################################################## 
