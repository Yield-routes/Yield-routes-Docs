# YieldRoutes Backend API Reference — Oracle Route Group

The **Oracle** route group manages external price feeds, TWAP benchmarks, and Soroban on-chain oracle freshness checks required for multi-asset yield calculations.

---

## Endpoints Summary

| Method | Path | Auth | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/api/v1/oracle/rates` | Public | List current exchange rates and prices for all supported tokens |
| `GET` | `/api/v1/oracle/rates/:pair` | Public | Get the latest price, confidence interval, and timestamp for a pair |
| `POST` | `/api/v1/oracle/refresh` | Private (`X-API-Key`) | Trigger an on-demand on-chain oracle update cycle |

---

## 1. List All Exchange Rates

### `GET /api/v1/oracle/rates`

Returns all active asset prices relative to USD.

#### Query Parameters

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `source` | `string` | No | Filter by oracle provider (`reflector`, `pyth`, `blended`) |

#### Example Request
```bash
curl -X GET "https://api.yieldroutes.com/api/v1/oracle/rates" \
  -H "Accept: application/json"
```

#### Example Response (`200 OK`)
```json
{
  "success": true,
  "data": {
    "baseCurrency": "USD",
    "updatedAt": "2026-10-08T09:28:45Z",
    "rates": {
      "XLM": {
        "priceUsd": "0.1245",
        "confidence": 0.0002,
        "source": "reflector",
        "decimals": 7
      },
      "USDC": {
        "priceUsd": "1.0001",
        "confidence": 0.0001,
        "source": "pyth",
        "decimals": 7
      },
      "AQUA": {
        "priceUsd": "0.00084",
        "confidence": 0.00001,
        "source": "blended",
        "decimals": 7
      }
    }
  },
  "timestamp": "2026-10-08T09:30:00Z"
}
```

---

## 2. Get Single Pair Rate

### `GET /api/v1/oracle/rates/:pair`

Fetches pricing metrics and 24h change for a trading pair.

#### Path Parameters

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `pair` | `string` | Yes | Token pair formatted as `ASSET_BASE-ASSET_QUOTE` (e.g. `XLM-USDC`) |

#### Example Request
```bash
curl -X GET "https://api.yieldroutes.com/api/v1/oracle/rates/XLM-USDC" \
  -H "Accept: application/json"
```

#### Example Response (`200 OK`)
```json
{
  "success": true,
  "data": {
    "pair": "XLM-USDC",
    "rate": "0.1245",
    "twap30m": "0.1244",
    "twap24h": "0.1238",
    "priceChange24hPercent": 1.25,
    "lastLedgerSequence": 54291083,
    "updatedAt": "2026-10-08T09:28:45Z"
  },
  "timestamp": "2026-10-08T09:30:00Z"
}
```

---

## 3. Trigger Oracle Price Refresh

### `POST /api/v1/oracle/refresh`

Triggers immediate synchronization with on-chain Soroban oracle contracts (e.g., Reflector) and updates Redis cache buffers.

#### Headers

| Header | Value | Description |
| :--- | :--- | :--- |
| `X-API-Key` | `<SECRET_API_KEY>` | Administrative or partner API key |

#### Request Body (JSON)

```json
{
  "pairs": ["XLM-USDC"],
  "forceUpdate": false
}
```

#### Example Request
```bash
curl -X POST "https://api.yieldroutes.com/api/v1/oracle/refresh" \
  -H "X-API-Key: yr_live_9f83a04b82c14092bdf87e2a901f4c7d" \
  -H "Content-Type: application/json" \
  -d '{"pairs": ["XLM-USDC"]}'
```

#### Example Response (`200 OK`)
```json
{
  "success": true,
  "data": {
    "syncedPairs": ["XLM-USDC"],
    "oracleLedgerSequence": 54291085,
    "durationMs": 42
  },
  "timestamp": "2026-10-08T09:30:00Z"
}
```
