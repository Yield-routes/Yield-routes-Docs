# YieldRoutes Backend REST API Reference

The YieldRoutes Backend API provides public endpoints for querying swap routes across Soroban AMM pools, managing tokenized yield vaults (SEP-56), inspecting liquidity pools, and accessing time-weighted average price (TWAP) feeds on the Stellar network.

---

## Base URLs

| Environment | Base URL | Network |
|---|---|---|
| **Local Development** | `http://localhost:3004/api/v1` | Standalone / Futurenet |
| **Testnet** | `https://testnet-api.yieldroutes.org/api/v1` | Stellar Testnet |
| **Mainnet** | `https://api.yieldroutes.org/api/v1` | Stellar Mainnet |

---

## API Route Groups

The API is structured into four core functional domains:

1. [**Routes API (`/routes`)**](routes.md) — Quote generation, optimal multi-hop routing, slippage simulation, and swap execution tracking.
2. [**Pools API (`/pools`)**](pools.md) — Registered Soroban AMM liquidity pools, volume metrics, reserve depths, and admin registrations.
3. [**Vault API (`/vault`)**](vault.md) — Tokenized SEP-56 vault deposits, withdrawals, share price calculations, harvest event tracking, and APY history.
4. [**Oracle API (`/oracle`)**](oracle.md) — Spot price feeds, 1-hour/24-hour TWAP aggregations, historical price snapshots, and reporter status.

---

## Authentication & Authorization

- **Public Endpoints**: Most read endpoints (`GET /routes/quote`, `GET /pools`, `GET /vault/share-price`, `GET /oracle/twap`) are completely public and require no authentication.
- **Admin & Write Endpoints**: Endpoints that invoke state-modifying smart contract calls or register protocol infrastructure require:
  - An `Authorization: Bearer <ADMIN_JWT>` header, or
  - Signed Soroban transaction envelopes (`XDR`) signed by the relevant Stellar account keypair (`Address::require_auth`).

---

## Standard Response Format

All responses are returned in JSON format with standard HTTP status codes:

### Success Response
```json
{
  "success": true,
  "data": { ... },
  "timestamp": 1729960000000
}
```

### Error Response
```json
{
  "success": false,
  "error": {
    "code": "INVALID_PARAMETERS",
    "message": "amountIn must be a positive integer formatted as a base-10 string or number",
    "details": {
      "field": "amountIn",
      "received": -50
    }
  },
  "timestamp": 1729960000000
}
```

---

## Common Error Codes

| HTTP Status | Error Code | Description |
|---|---|---|
| `400` | `INVALID_PARAMETERS` | One or more query parameters or body fields failed validation. |
| `404` | `RESOURCE_NOT_FOUND` | The requested route, pool, transaction, or pair does not exist. |
| `408` | `QUOTE_EXPIRED` | The provided `quoteId` exceeded the 60-second validity TTL. |
| `422` | `NO_ROUTE_FOUND` | No registered AMM pool path could be found satisfying the pair and depth. |
| `429` | `RATE_LIMIT_EXCEEDED` | Request rate exceeded tier limits (Default: 60 requests/minute per IP). |
| `500` | `INTERNAL_SERVER_ERROR` | An unexpected error occurred while interacting with the Soroban RPC. |
