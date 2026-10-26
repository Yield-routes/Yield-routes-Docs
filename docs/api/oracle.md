# Oracle API (`/oracle`)

The Oracle API provides tamper-resistant price feeds derived from rolling time-weighted average prices (TWAP) and authorized reporter submissions. Prices are scaled by $10^9$ for integer arithmetic.

---

## 1. Get TWAP Price

Calculates the time-weighted average price over a rolling 1-hour window for a target token pair.

- **Method**: `GET`
- **Path**: `/oracle/twap`
- **Auth**: Public

### Query Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `base` | `string` | Yes | Contract address of the base token. |
| `quote` | `string` | Yes | Contract address of the quote token. |

### Response (`200 OK`)

```json
{
  "success": true,
  "data": {
    "base": "CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC",
    "quote": "CAS3J7GYLGXMF6TDJBBYYSE3HQ6BBSMLNUQ34T6TZMYMW2EVH34XOWMA",
    "twapPriceRaw": "1050300000",
    "twapPrice": 1.0503,
    "sampleCount": 10,
    "windowSeconds": 3600,
    "oldestSampleTimestamp": 1729956400,
    "newestSampleTimestamp": 1729959950
  }
}
```

### Example `curl`

```bash
curl "https://api.yieldroutes.org/api/v1/oracle/twap?base=CDLZ...&quote=CAS3..."
```

---

## 2. Get Spot Price

Returns the single most recent price sample recorded by authorized reporters.

- **Method**: `GET`
- **Path**: `/oracle/spot`
- **Auth**: Public

### Response (`200 OK`)

```json
{
  "success": true,
  "data": {
    "spotPriceRaw": "1055000000",
    "spotPrice": 1.055,
    "timestamp": 1729959950,
    "reporter": "GBK7...44chars..."
  }
}
```

---

## 3. Submit Price Sample (Reporter Only)

Authorized oracle node submission endpoint. Must be accompanied by a valid reporter signature.

- **Method**: `POST`
- **Path**: `/oracle/submit`
- **Auth**: Reporter Signature

### Request Body

```json
{
  "reporterAddress": "GBK74B7K7VHQXNQ4K7Q9M2R6X7B4N2Q8F1A3E5C7D9E1F2A3B4C5D6E7",
  "baseToken": "CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC",
  "quoteToken": "CAS3J7GYLGXMF6TDJBBYYSE3HQ6BBSMLNUQ34T6TZMYMW2EVH34XOWMA",
  "price": "1055000000",
  "signedPayload": "AAAAAg..."
}
```

### Response (`200 OK`)

```json
{
  "success": true,
  "data": {
    "submitted": true,
    "newSampleCount": 11,
    "currentTwap": 1.0507
  }
}
```
