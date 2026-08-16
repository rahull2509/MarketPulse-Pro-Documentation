# P4-10 CAPACITY AND SIZING DECISION
**Phase:** P4-10 Production Hardening

## 1. Result-Driven Sizing
- P4-10 MUST NOT hardcode a final production EC2/VM size before load-test results. Capacity sizing is RESULT-DRIVEN.
- The load test must determine: CPU saturation, memory saturation, PostgreSQL saturation, Redis saturation, network saturation, and WebSocket connection capacity.
- Production sizing will be approved only after the 100k DAU load-test results are available. No arbitrary hardware recommendation may be treated as final capacity.
