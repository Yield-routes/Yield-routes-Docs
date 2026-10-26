# Routes API (`/routes`)

The Routes API allows frontend interfaces and trading bots to find optimal multi-hop swap routes across registered Soroban AMM liquidity pools, simulate price impacts, generate cryptographically time-bounded quotes, and monitor on-chain swap execution.

---

## 1. Request Swap Quote

Calculates the optimal trade execution route across registered pools for a given token pair and input amount.

- **Method**: `POST`
- **Path**: `/routes/quote`
- **Auth**: Public

### Request Body

| Field | Type | Required | Description |
|---|---|---|---|
| `tokenIn` | `string` | Yes | Stellar contract address of the input token (56-character string). |
| `tokenOut` | `string` | Yes | Stellar contract address of the target token. |
| `amountIn` | `number \| string` | Yes | Amount of `tokenIn` to swap (scaled to integer base units, e.g. stroops/wei). |
| `maxHops` | `number` | No | Maximum intermediary hops allowed (Default: `3`, Max: `5`). |
| `slippageToleranceBps` | `number` | No | User's slippage tolerance in basis points (e.g. `50` = 0.50%). Default: `50`. |

```json
{
  "tokenIn": "CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC",
  "tokenOut": "CAS3J7GYLGXMF6TDJBBYYSE3HQ6BBSMLNUQ34T6TZMYMW2EVH34XOWMA",
  "amountIn": "1000000000",
  "maxHops": 3,
  "slippageToleranceBps": 50
}
```

### Response (`200 OK`)

```json
{
  "success": true,
  "data": {
    "quoteId": "quote_9d8f3e2b_7a14",
    "tokenIn": "CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC",
    "tokenOut": "CAS3J7GYLGXMF6TDJBBYYSE3HQ6BBSMLNUQ34T6TZMYMW2EVH34XOWMA",
    "amountIn": "1000000000",
    "expectedOut": "997100000",
    "minimumOut": "992114500",
    "protocolFeeAmount": "1000000",
    "priceImpactBps": 5,
    "hops": 1,
    "path": [
      {
        "pool": "CBX2G55M67LQQF2K4Q7E3VZXNQ4K7Q9M2R6X7B4N2Q8F1A3E5C7D9E1F",
        "tokenIn": "CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC",
        "tokenOut": "CAS3J7GYLGXMF6TDJBBYYSE3HQ6BBSMLNUQ34T6TZMYMW2EVH34XOWMA",
        "feeBps": 30
      }
    ],
    "validUntil": "2026-10-26T19:45:00.000Z",
    "ttlSeconds": 60
  }
}
```

### Example `curl`

```bash
curl -X POST https://api.yieldroutes.org/api/v1/routes/quote \
  -H "Content-Type: application/json" \
  -d '{
    "tokenIn": "CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC",
    "tokenOut": "CAS3J7GYLGXMF6TDJBBYYSE3HQ6BBSMLNUQ34T6TZMYMW2EVH34XOWMA",
    "amountIn": "1000000000",
    "maxHops": 3
  }'
```

---

## 2. Inspect Quote Details

Retrieves the parameters and live expiry countdown of a previously generated quote.

- **Method**: `GET`
- **Path**: `/routes/quote/:quoteId`
- **Auth**: Public

### Response (`200 OK`)

```json
{
  "success": true,
  "data": {
    "quoteId": "quote_9d8f3e2b_7a14",
    "status": "ACTIVE",
    "expectedOut": "997100000",
    "remainingTtlSeconds": 42,
    "validUntil": "2026-10-26T19:45:00.000Z"
  }
}
```

---

## 3. Build Swap Transaction (Pre-flight Envelope)

Generates an unsigned Soroban transaction envelope (`XDR`) ready for Freighter / Albedo signing.

- **Method**: `POST`
- **Path**: `/routes/build-tx`
- **Auth**: Public

### Request Body

```json
{
  "quoteId": "quote_9d8f3e2b_7a14",
  "senderAddress": "GDZX4B7K7VHQXNQ4K7Q9M2R6X7B4N2Q8F1A3E5C7D9E1F2A3B4C5D6E7",
  "minOut": "992114500"
}
```

### Response (`200 OK`)

```json
{
  "success": true,
  "data": {
    "unsignedXdr": "AAAAAgAAAAD...",
    "networkPassphrase": "Test SDF Network ; September 2015",
    "estimatedFee": "15000",
    "sorobanResourceFee": "450000"
  }
}
```

---

## 4. Query Cumulative Route Volume

Returns protocol-wide volume metrics routed through the `RouteAggregator`.

- **Method**: `GET`
- **Path**: `/routes/metrics`
- **Auth**: Public

### Response (`200 OK`)

```json
{
  "success": true,
  "data": {
    "totalVolumeUsd": "14523900.50",
    "totalSwapsExecuted": 84920,
    "totalProtocolFeesCollectedUsd": "14523.90",
    "last24hVolumeUsd": "342500.00"
  }
}
```
