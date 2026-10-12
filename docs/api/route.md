# YieldRoutes Backend API Reference — Route Optimization Group

The **Route** route group houses YieldRoutes' mathematical optimization engine, which solves multi-pool capital allocation equations to maximize net blended APY subject to slippage, gas cost, and liquidity concentration constraints.

---

## Endpoints Summary

| Method | Path | Auth | Description |
| :--- | :--- | :--- | :--- |
| `POST` | `/api/v1/route/optimize` | Public | Compute mathematically optimal capital allocation weights across pools |
| `GET` | `/api/v1/route/active` | Public | Fetch current active on-chain route distribution |
| `GET` | `/api/v1/route/rebalance-history` | Public | List historical vault rebalancing transactions and execution metrics |

---

## 1. Compute Optimal Yield Route

### `POST /api/v1/route/optimize`

Calculates the optimal capital allocation for a given deposit amount or total portfolio size using the gradient ascent convex optimization model.

#### Request Body (JSON)

| Field | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| `asset` | `string` | Yes | — | Asset symbol or SAC contract ID (e.g. `USDC`) |
| `amount` | `string` | Yes | — | Total capital amount in stroops (integer string) |
| `maxPools` | `number` | No | `5` | Maximum number of candidate pools to allocate across |
| `riskTolerance` | `string` | No | `moderate` | Risk tier: `conservative`, `moderate`, `aggressive` |
| `maxSlippageBps` | `number` | No | `50` | Maximum acceptable slippage in basis points |

#### Example Request
```bash
curl -X POST "https://api.yieldroutes.com/api/v1/route/optimize" \
  -H "Content-Type: application/json" \
  -d '{
    "asset": "USDC",
    "amount": "100000000000",
    "riskTolerance": "moderate",
    "maxPools": 3
  }'
```

#### Example Response (`200 OK`)
```json
{
  "success": true,
  "data": {
    "asset": "USDC",
    "totalCapitalStroops": "100000000000",
    "projectedNetApy": 7.92,
    "allocation": [
      {
        "poolId": "pool-blend-usdc",
        "poolName": "Blend USDC Lending",
        "weightPercentage": 60.0,
        "amountStroops": "60000000000",
        "marginalApy": 6.80
      },
      {
        "poolId": "pool-soroswap-usdc-xlm",
        "poolName": "SoroSwap USDC/XLM LP",
        "weightPercentage": 40.0,
        "amountStroops": "40000000000",
        "marginalApy": 9.60
      }
    ],
    "estimatedGasFeeStroops": "250000",
    "computedAt": "2026-10-08T09:30:00Z"
  },
  "timestamp": "2026-10-08T09:30:00Z"
}
```

---

## 2. Get Active Route Strategy

### `GET /api/v1/route/active`

Returns the current on-chain capital allocation ratio implemented by the YieldRoutes Vault.

#### Query Parameters

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `asset` | `string` | No | Target asset filter (default: `USDC`) |

#### Example Request
```bash
curl -X GET "https://api.yieldroutes.com/api/v1/route/active?asset=USDC" \
  -H "Accept: application/json"
```

#### Example Response (`200 OK`)
```json
{
  "success": true,
  "data": {
    "asset": "USDC",
    "currentBlendedApy": 7.84,
    "totalAllocatedStroops": "12500000000000",
    "lastRebalancedAt": "2026-10-08T06:00:00Z",
    "routeDistribution": [
      { "poolId": "pool-blend-usdc", "share": 0.62 },
      { "poolId": "pool-soroswap-usdc-xlm", "share": 0.38 }
    ]
  },
  "timestamp": "2026-10-08T09:30:00Z"
}
```

---

## 3. List Rebalancing History

### `GET /api/v1/route/rebalance-history`

Lists automated rebalance transactions executed by the backend keeper bot.

#### Query Parameters

| Parameter | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| `limit` | `number` | No | `20` | Max entries to return |
| `offset` | `number` | No | `0` | Pagination offset |

#### Example Request
```bash
curl -X GET "https://api.yieldroutes.com/api/v1/route/rebalance-history?limit=5" \
  -H "Accept: application/json"
```

#### Example Response (`200 OK`)
```json
{
  "success": true,
  "data": {
    "events": [
      {
        "id": "reb-20261008-01",
        "txHash": "4a5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c",
        "rebalancedAt": "2026-10-08T06:00:00Z",
        "previousApy": 7.35,
        "newApy": 7.84,
        "deltaApy": 0.49,
        "reallocatedStroops": "1500000000000"
      }
    ]
  },
  "timestamp": "2026-10-08T09:30:00Z"
}
```
