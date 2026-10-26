# Pools API (`/pools`)

The Pools API manages registered Soroban AMM liquidity pools used by the `RouteAggregator`. Integrators can query liquidity distribution, swap fees, reserve balances, and historical trading volume.

---

## 1. List Registered Pools

Returns all AMM liquidity pools currently whitelisted for route execution.

- **Method**: `GET`
- **Path**: `/pools`
- **Auth**: Public

### Query Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `limit` | `integer` | `20` | Number of pool records to return (Max: `100`). |
| `offset` | `integer` | `0` | Pagination offset index. |
| `token` | `string` | — | Filter pools containing this token address. |

### Response (`200 OK`)

```json
{
  "success": true,
  "data": {
    "total": 42,
    "pools": [
      {
        "poolAddress": "CBX2G55M67LQQF2K4Q7E3VZXNQ4K7Q9M2R6X7B4N2Q8F1A3E5C7D9E1F",
        "tokenA": {
          "address": "CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC",
          "symbol": "USDC",
          "decimals": 7,
          "reserve": "5420000000000"
        },
        "tokenB": {
          "address": "CAS3J7GYLGXMF6TDJBBYYSE3HQ6BBSMLNUQ34T6TZMYMW2EVH34XOWMA",
          "symbol": "XLM",
          "decimals": 7,
          "reserve": "52900000000000"
        },
        "feeBps": 30,
        "volume24hUsd": "128450.00",
        "registeredAt": "2026-08-14T10:00:00Z"
      }
    ]
  }
}
```

### Example `curl`

```bash
curl "https://api.yieldroutes.org/api/v1/pools?token=CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC"
```

---

## 2. Get Pool By Address

Retrieves detailed runtime metrics and reserve ratios for a specific pool contract.

- **Method**: `GET`
- **Path**: `/pools/:poolAddress`
- **Auth**: Public

### Response (`200 OK`)

```json
{
  "success": true,
  "data": {
    "poolAddress": "CBX2G55M67LQQF2K4Q7E3VZXNQ4K7Q9M2R6X7B4N2Q8F1A3E5C7D9E1F",
    "tokenA": "CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC",
    "tokenB": "CAS3J7GYLGXMF6TDJBBYYSE3HQ6BBSMLNUQ34T6TZMYMW2EVH34XOWMA",
    "tvlUsd": "1084000.00",
    "apy30d": "8.45%",
    "contractVersion": "v1.2.0"
  }
}
```

---

## 3. Register New Pool (Admin)

Registers an AMM pool contract for symmetric routing. Must be called by the `RouteAggregator` admin key.

- **Method**: `POST`
- **Path**: `/pools/register`
- **Auth**: Admin (`Bearer <JWT>`)

### Request Body

```json
{
  "tokenA": "CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC",
  "tokenB": "CAS3J7GYLGXMF6TDJBBYYSE3HQ6BBSMLNUQ34T6TZMYMW2EVH34XOWMA",
  "poolAddress": "CBX2G55M67LQQF2K4Q7E3VZXNQ4K7Q9M2R6X7B4N2Q8F1A3E5C7D9E1F"
}
```

### Response (`201 Created`)

```json
{
  "success": true,
  "data": {
    "registered": true,
    "transactionHash": "4a7f2e1b...64hex...",
    "ledgerSequence": 5420194
  }
}
```
