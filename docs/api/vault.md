# Vault API (`/vault`)

The Vault API provides endpoints for tokenized yield-bearing vaults implementing the **SEP-56** standard on Stellar. Users deposit underlying assets (e.g. USDC) to receive vault shares that continuously appreciate in value as fees and yield are harvested.

---

## 1. Get Share Price

Returns the current price per vault share, scaled by $10^9$ (e.g. `1000000000` = 1.000000000 underlying token per share).

- **Method**: `GET`
- **Path**: `/vault/share-price`
- **Auth**: Public

### Response (`200 OK`)

```json
{
  "success": true,
  "data": {
    "sharePriceRaw": "1050000000",
    "sharePrice": 1.05,
    "scalingFactor": "1000000000",
    "totalAssets": "11500000000000",
    "totalShares": "10952380950000",
    "timestamp": 1729960000000
  }
}
```

### Example `curl`

```bash
curl https://api.yieldroutes.org/api/v1/vault/share-price
```

---

## 2. Simulate Deposit

Calculates the exact amount of vault shares that will be minted for a specified deposit amount without broadcasting a transaction.

- **Method**: `POST`
- **Path**: `/vault/simulate-deposit`
- **Auth**: Public

### Request Body

```json
{
  "amount": "10000000000"
}
```

### Response (`200 OK`)

```json
{
  "success": true,
  "data": {
    "depositAmount": "10000000000",
    "expectedShares": "9523809523",
    "currentSharePrice": 1.05,
    "feeDeducted": "0"
  }
}
```

---

## 3. Query User Position

Returns the share balance, underlying asset entitlement, and cumulative yield earned by a specific Stellar account.

- **Method**: `GET`
- **Path**: `/vault/position/:accountAddress`
- **Auth**: Public

### Response (`200 OK`)

```json
{
  "success": true,
  "data": {
    "account": "GDZX4B7K7VHQXNQ4K7Q9M2R6X7B4N2Q8F1A3E5C7D9E1F2A3B4C5D6E7",
    "sharesHeld": "9523809523",
    "currentAssetValue": "10000000000",
    "depositedPrincipal": "9600000000",
    "unrealizedYield": "400000000",
    "yieldPercent": "4.16%"
  }
}
```

---

## 4. List Harvest History

Returns historical harvest events where protocol fees and external yields were injected into the vault.

- **Method**: `GET`
- **Path**: `/vault/harvests`
- **Auth**: Public

### Query Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `limit` | `integer` | `10` | Records per page (Max: `50`). |
| `page` | `integer` | `1` | Page number. |

### Response (`200 OK`)

```json
{
  "success": true,
  "data": {
    "harvests": [
      {
        "id": "hrv_55b2_8910",
        "yieldAmount": "3500000000",
        "preSharePrice": 1.0465,
        "postSharePrice": 1.0500,
        "transactionHash": "8f3e2b...64hex...",
        "timestamp": "2026-10-26T06:00:00Z"
      }
    ],
    "rollingApy": "11.25%"
  }
}
```
