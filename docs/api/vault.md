# YieldRoutes Backend API Reference — Vault Route Group

The **Vault** route group exposes endpoints for querying YieldRoutes ERC-4626 / SEP-41 vault state, individual user positions, and generating unsigned transaction XDR envelopes for Freighter or wallet signature.

---

## Endpoints Summary

| Method | Path | Auth | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/api/v1/vault/status` | Public | Get aggregate vault metrics, total assets, and current share price |
| `GET` | `/api/v1/vault/shares/:user` | Public | Get a specific user's vault share balance and unclaimed yield |
| `POST` | `/api/v1/vault/deposit-intent` | Public | Build an unsigned deposit transaction XDR envelope |
| `POST` | `/api/v1/vault/withdraw-intent` | Public | Build an unsigned withdrawal transaction XDR envelope |

---

## 1. Get Aggregate Vault Status

### `GET /api/v1/vault/status`

Returns total underlying assets, active share supply, and current exchange rate.

#### Example Request
```bash
curl -X GET "https://api.yieldroutes.com/api/v1/vault/status" \
  -H "Accept: application/json"
```

#### Example Response (`200 OK`)
```json
{
  "success": true,
  "data": {
    "vaultContractAddress": "CA7...VAULT_CONTRACT_ID",
    "underlyingAsset": "USDC",
    "totalAssetsStroops": "12500000000000",
    "totalShares": "11800000000000",
    "sharePriceStroops": "1.0593220",
    "blendedApy": 7.84,
    "vaultStatus": "active",
    "lastHarvestTimestamp": "2026-10-08T06:00:00Z"
  },
  "timestamp": "2026-10-08T09:30:00Z"
}
```

---

## 2. Get User Vault Position

### `GET /api/v1/vault/shares/:user`

Retrieves a user wallet's share balance, corresponding underlying assets, and lifetime yield earnings.

#### Path Parameters

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `user` | `string` | Yes | Stellar account public key (`G...`) |

#### Example Request
```bash
curl -X GET "https://api.yieldroutes.com/api/v1/vault/shares/GA5ZSEJYB37JRC5AVCIA5MOP4RHTM335X2KGX3IHOJAPP5RE34K4KZVN" \
  -H "Accept: application/json"
```

#### Example Response (`200 OK`)
```json
{
  "success": true,
  "data": {
    "userAddress": "GA5ZSEJYB37JRC5AVCIA5MOP4RHTM335X2KGX3IHOJAPP5RE34K4KZVN",
    "shares": "50000000000",
    "underlyingAssetsStroops": "52966100000",
    "underlyingAssetsUsd": "5296.61",
    "unrealizedYieldStroops": "2966100000",
    "firstDepositTimestamp": "2026-08-15T12:00:00Z"
  },
  "timestamp": "2026-10-08T09:30:00Z"
}
```

---

## 3. Generate Deposit Intent

### `POST /api/v1/vault/deposit-intent`

Constructs an unsigned Soroban contract transaction envelope that deposits underlying tokens into the vault and mints shares to the user.

#### Request Body (JSON)

| Field | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `userAddress` | `string` | Yes | Stellar wallet address (`G...`) |
| `amountStroops` | `string` | Yes | Amount of underlying asset to deposit (integer string) |
| `slippageBps` | `number` | No | Maximum allowed slippage on share minting (default `50`) |

#### Example Request
```bash
curl -X POST "https://api.yieldroutes.com/api/v1/vault/deposit-intent" \
  -H "Content-Type: application/json" \
  -d '{
    "userAddress": "GA5ZSEJYB37JRC5AVCIA5MOP4RHTM335X2KGX3IHOJAPP5RE34K4KZVN",
    "amountStroops": "10000000000"
  }'
```

#### Example Response (`200 OK`)
```json
{
  "success": true,
  "data": {
    "unsignedXdr": "AAAAAgAAAABn8y...BASE64_TRANSACTION_ENVELOPE...",
    "expectedSharesMinted": "9439999900",
    "feeStroops": "100",
    "timeoutSeconds": 300
  },
  "timestamp": "2026-10-08T09:30:00Z"
}
```

---

## 4. Generate Withdrawal Intent

### `POST /api/v1/vault/withdraw-intent`

Constructs an unsigned Soroban transaction envelope that burns user vault shares and redeems underlying assets back to the wallet.

#### Request Body (JSON)

| Field | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `userAddress` | `string` | Yes | Stellar wallet address (`G...`) |
| `sharesToBurn` | `string` | Yes | Number of vault shares to burn (integer string) |

#### Example Request
```bash
curl -X POST "https://api.yieldroutes.com/api/v1/vault/withdraw-intent" \
  -H "Content-Type: application/json" \
  -d '{
    "userAddress": "GA5ZSEJYB37JRC5AVCIA5MOP4RHTM335X2KGX3IHOJAPP5RE34K4KZVN",
    "sharesToBurn": "5000000000"
  }'
```

#### Example Response (`200 OK`)
```json
{
  "success": true,
  "data": {
    "unsignedXdr": "AAAAAgAAAABn8y...BASE64_WITHDRAW_XDR...",
    "expectedAssetsRedeemed": "5296610000",
    "feeStroops": "100"
  },
  "timestamp": "2026-10-08T09:30:00Z"
}
```
