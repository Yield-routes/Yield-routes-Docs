# YieldRoutes Backend API Reference — Pools Route Group

The **Pools** route group provides real-time information and historical telemetry on all supported Soroban and Stellar DEX liquidity and yield-generating pools tracked by YieldRoutes.

---

## Endpoints Summary

| Method | Path | Auth | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/api/v1/pools` | Public | List all indexed liquidity pools with current APY and TVL |
| `GET` | `/api/v1/pools/:id` | Public | Get detailed metadata and status for a specific pool |
| `GET` | `/api/v1/pools/:id/apy-history` | Public | Retrieve historical APY time series for trend analysis |
| `GET` | `/api/v1/pools/:id/tvl` | Public | Retrieve historical TVL metric breakdown |

---

## 1. List All Pools

### `GET /api/v1/pools`

Returns an array of all active liquidity pools tracked by the YieldRoutes indexing engine.

#### Query Parameters

| Parameter | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| `asset` | `string` | No | — | Filter pools containing this asset code or contract (e.g. `USDC`) |
| `minApy` | `number` | No | `0` | Minimum current annual percentage yield filter |
| `status` | `string` | No | `active` | Filter by status (`active`, `paused`, `deprecated`) |
| `limit` | `number` | No | `50` | Maximum items to return (1-100) |
| `offset` | `number` | No | `0` | Pagination offset |

#### Example Request
```bash
curl -X GET "https://api.yieldroutes.com/api/v1/pools?asset=USDC&minApy=5.0" \
  -H "Accept: application/json"
```

#### Example Response (`200 OK`)
```json
{
  "success": true,
  "data": {
    "total": 3,
    "pools": [
      {
        "id": "pool-soroswap-usdc-xlm",
        "name": "SoroSwap USDC/XLM LP",
        "protocol": "Soroswap",
        "contractAddress": "CCW67TSZV3SSS2HXMBQ5JFGCKJNXKZM7UQUWUZPUTHXSTZLEO7SJMI75",
        "assets": ["USDC", "XLM"],
        "tvlUsd": "4520300.20",
        "currentApy": 8.45,
        "baseApy": 4.10,
        "rewardApy": 4.35,
        "status": "active",
        "lastUpdated": "2026-10-08T09:15:00Z"
      },
      {
        "id": "pool-blend-usdc",
        "name": "Blend USDC Lending Pool",
        "protocol": "Blend",
        "contractAddress": "CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC",
        "assets": ["USDC"],
        "tvlUsd": "8100500.00",
        "currentApy": 6.80,
        "baseApy": 6.80,
        "rewardApy": 0.00,
        "status": "active",
        "lastUpdated": "2026-10-08T09:15:00Z"
      }
    ]
  },
  "timestamp": "2026-10-08T09:30:00Z"
}
```

---

## 2. Get Pool Details

### `GET /api/v1/pools/:id`

Retrieves deep configuration, contract addresses, token reserve ratios, and risk score for a single pool.

#### Path Parameters

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `id` | `string` | Yes | Unique pool identifier (e.g. `pool-blend-usdc`) or contract address |

#### Example Request
```bash
curl -X GET "https://api.yieldroutes.com/api/v1/pools/pool-blend-usdc" \
  -H "Accept: application/json"
```

#### Example Response (`200 OK`)
```json
{
  "success": true,
  "data": {
    "id": "pool-blend-usdc",
    "name": "Blend USDC Lending Pool",
    "protocol": "Blend",
    "contractAddress": "CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC",
    "assets": [
      {
        "code": "USDC",
        "issuer": "GA5ZSEJYB37JRC5AVCIA5MOP4RHTM335X2KGX3IHOJAPP5RE34K4KZVN",
        "reserve": "81005000000000"
      }
    ],
    "tvlUsd": "8100500.00",
    "currentApy": 6.80,
    "riskScore": "A+",
    "utilizationRate": 0.78,
    "status": "active",
    "feeBps": 15,
    "lastUpdated": "2026-10-08T09:15:00Z"
  },
  "timestamp": "2026-10-08T09:30:00Z"
}
```

---

## 3. Retrieve Historical APY

### `GET /api/v1/pools/:id/apy-history`

Provides historical APY data points over a specified timeframe for charting and forecasting.

#### Query Parameters

| Parameter | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| `timeframe` | `string` | No | `30d` | Window duration: `24h`, `7d`, `30d`, `90d`, `1y` |
| `interval` | `string` | No | `1d` | Sample interval: `1h`, `4h`, `1d` |

#### Example Request
```bash
curl -X GET "https://api.yieldroutes.com/api/v1/pools/pool-blend-usdc/apy-history?timeframe=7d&interval=1d" \
  -H "Accept: application/json"
```

#### Example Response (`200 OK`)
```json
{
  "success": true,
  "data": {
    "poolId": "pool-blend-usdc",
    "timeframe": "7d",
    "history": [
      { "timestamp": "2026-10-01T00:00:00Z", "apy": 6.72 },
      { "timestamp": "2026-10-02T00:00:00Z", "apy": 6.75 },
      { "timestamp": "2026-10-03T00:00:00Z", "apy": 6.81 },
      { "timestamp": "2026-10-04T00:00:00Z", "apy": 6.79 },
      { "timestamp": "2026-10-05T00:00:00Z", "apy": 6.84 },
      { "timestamp": "2026-10-06T00:00:00Z", "apy": 6.82 },
      { "timestamp": "2026-10-07T00:00:00Z", "apy": 6.80 }
    ]
  },
  "timestamp": "2026-10-08T09:30:00Z"
}
```

---

## 4. Retrieve Historical TVL

### `GET /api/v1/pools/:id/tvl`

Returns time series TVL in USD and raw token denomination.

#### Example Request
```bash
curl -X GET "https://api.yieldroutes.com/api/v1/pools/pool-blend-usdc/tvl?timeframe=7d" \
  -H "Accept: application/json"
```

#### Example Response (`200 OK`)
```json
{
  "success": true,
  "data": {
    "poolId": "pool-blend-usdc",
    "currentTvlUsd": "8100500.00",
    "history": [
      { "timestamp": "2026-10-01T00:00:00Z", "tvlUsd": "7950000.00" },
      { "timestamp": "2026-10-07T00:00:00Z", "tvlUsd": "8100500.00" }
    ]
  },
  "timestamp": "2026-10-08T09:30:00Z"
}
```
