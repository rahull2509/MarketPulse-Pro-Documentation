# P4-08 GOVERNANCE GAP RESOLUTION

**Status:** [APPROVED]
**Date:** 2026-08-10

## 1. Objective
This document resolves the explicit governance gaps for P4-08, including the final missing specification: Angel One IP/MAC resolution.

## 2. Gap: Instrument Mapping
**Status:** [APPROVED]

## 3. Gap: Rate Limits
**Status:** [APPROVED]

## 4. Gap: Error Semantics
**Status:** [APPROVED]

## 5. Gap: Angel One Authentication Specification
**Status:** [APPROVED] (Exact JSON body, endpoint, headers, and credential boundary all approved).

## 6. Gap: Angel One IP/MAC Resolution Policy
Angel One mandates that requests use a registered static public IP. Relying on network discovery is unstable and insecure.
**Resolution Approved:**
- `X-ClientLocalIP`, `X-ClientPublicIP`, and `X-MACAddress` MUST be populated strictly from explicit backend environment configurations (`ANGELONE_CLIENT_LOCAL_IP`, `ANGELONE_CLIENT_PUBLIC_IP`, `ANGELONE_CLIENT_MAC_ADDRESS`).
- The backend MUST explicitly ignore client-provided headers (e.g., `X-Forwarded-For`) for these values.
- Dynamic network-interface discovery MUST NOT be used.
- Startup validation MUST enforce the presence of these configs.
- **Status:** [APPROVED] by Project Owner.

*All governance gaps have been formally resolved.*
