# YieldRoutes Backend API Reference — Overview

Welcome to the **YieldRoutes Backend API Reference**. This documentation provides a comprehensive guide for frontend engineers, protocol integrators, and automated agents interacting with the YieldRoutes REST API.

---

## 1. Base URL & Environments

| Environment | Base URL | Description |
| :--- | :--- | :--- |
| **Production** | `https://api.yieldroutes.com/api/v1` | Public Stellar Mainnet indexing and optimization service |
| **Testnet / Staging** | `https://testnet-api.yieldroutes.com/api/v1` | Stellar Testnet simulation and development endpoint |
| **Local Dev** | `http://localhost:4000/api/v1` | Default local backend server |

---

## 2. Authentication

All private and state-changing endpoints require an authenticated API key. Provide the API key in the HTTP request headers using either format:

### Option A: `X-API-Key` Header (Recommended)
```http
GET /api/v1/vault/status HTTP/1.1
Host: api.yieldroutes.com
X-API-Key: yr_live_9f83a04b82c14092bdf87e2a901f4c7d
Content-Type: application/json
```

### Option B: `Authorization` Bearer Token
```http
GET /api/v1/vault/status HTTP/1.1
Host: api.yieldroutes.com
Authorization: Bearer yr_live_9f83a04b82c14092bdf87e2a901f4c7d
Content-Type: application/json
```

Requests without a valid key on protected endpoints return `401 Unauthorized`.

---

## 3. Standard Response Format

Every API endpoint returns a standardized JSON envelope:

### Successful Response (`2xx`)
```json
{
  "success": true,
  "data": {
    "totalTvlUsd": "15420000.50",
    "activePools": 12
  },
  "timestamp": "2026-10-08T09:30:00Z"
}
```

### Error Response (`4xx` / `5xx`)
```json
{
  "success": false,
  "error": {
    "code": "INVALID_PARAMETER",
    "message": "Pool ID 'invalid-id' was not found in active registry.",
    "details": null
  },
  "timestamp": "2026-10-08T09:30:00Z"
}
```

---

## 4. HTTP Status Codes

| Status Code | Meaning | Description |
| :--- | :--- | :--- |
| `200 OK` | Success | The request succeeded and returned requested data. |
| `201 Created` | Created | A new resource (or intent) was successfully generated. |
| `400 Bad Request` | Validation Error | Missing required fields, malformed JSON, or out-of-bounds parameters. |
| `401 Unauthorized` | Auth Failure | Missing, expired, or invalid API key. |
| `403 Forbidden` | Permission Denied | API key lacks permission for the requested administrative action. |
| `404 Not Found` | Resource Missing | Specified pool, user, or route record does not exist. |
| `429 Too Many Requests` | Rate Limited | Request rate exceeded allowed quotas. |
| `500 Server Error` | Internal Failure | Unexpected server condition or downstream RPC node timeout. |

---

## 5. Rate Limits

Standard rate limits per API key:
- **Public Read Endpoints**: 120 requests / minute
- **Optimization & Calculation Endpoints**: 30 requests / minute
- **Transaction Intent Generation**: 20 requests / minute

Rate limit headers returned on all responses:
```http
X-RateLimit-Limit: 120
X-RateLimit-Remaining: 114
X-RateLimit-Reset: 1775640660
```

---

## 6. Route Groups Reference

Explore dedicated documentation for each functional route group:

1. [**Pools API (`docs/api/pools.md`)**](./pools.md) — Query liquidity pools, historical APY metrics, and TVL telemetry.
2. [**Oracle API (`docs/api/oracle.md`)**](./oracle.md) — Access oracle price feeds, cross-asset rates, and freshness triggers.
3. [**Route API (`docs/api/route.md`)**](./route.md) — Dynamic yield optimization, route rebalancing, and portfolio allocation.
4. [**Vault API (`docs/api/vault.md`)**](./vault.md) — User share balances, vault metrics, deposit intents, and withdrawal flows.
