---
description: >-
  This guide shows how your backend can call the TeamGames v4 claim endpoint to
  fetch undelivered purchases for a player and hand them out inside your own
  game or service.
---

# TeamGames Claim API

1. Sign in to the TeamGames dashboard.
2. Copy the **Server API Key**. Keep this key private; do not ship it in client builds.
3. Official TeamGames SDKs accept the raw key and handle Base64 encoding automatically. When you call the REST endpoint yourself, Base64-encode the raw key and send it in the `Authorization` header.

***

### 2. Make the Claim Request (v4)

```http
POST /api/v4/store/transaction/update HTTP/1.1
Host: <your-api-domain>
Content-Type: application/json
Authorization: BASE64_ENCODED_API_KEY  # raw key when using an SDK

{
  "playerName": "Player123",
  "preview": false,
  "includeRawTransactions": false
}
```

| Field                    | Description                                                                                           |
| ------------------------ | ----------------------------------------------------------------------------------------------------- |
| `playerName`             | Required. The username supplied during checkout. Case matters.                                        |
| `preview`                | Optional. `true` runs a dry run—returns the items but does not mark them delivered.                   |
| `includeRawTransactions` | Optional. `true` adds the database rows (invoice, payment type, etc.). Leave `false` for normal play. |

> **Rate limiting:** Each player is rate-limited to one claim call every ten seconds (per game server). Surface the response message verbatim so players know when to retry.

***

### 3. Interpret the Response

```json
{
  "status": "SUCCESS",
  "code": "SUCCESS",
  "message": "Transaction has been completed. Thank you for the purchase!",
  "data": {
    "claims": [
      {
        "player_name": "Player123",
        "product_id": 42,
        "product_id_string": "42",
        "product_amount": 5,
        "amount_purchased": 1,
        "product_name": "Super Sword",
        "product_price": 14.99
      }
    ],
    "rawTransactions": []
  }
}
```

* `status` / `code` describe the outcome (`SUCCESS`, `NO_ITEMS`, `RATE_LIMIT`, `SERVER_NOT_FOUND`, etc.). HTTP status is always 200.
* `message` is a player-friendly string you can show as-is.
* `data.claims` contains everything you need to grant the items—each entry mirrors the legacy array format.
* `data.rawTransactions` is populated only when `includeRawTransactions` is `true`; use it for auditing or reconciliation.

#### Common codes

| Code                | What it means                      | Your move                                     |
| ------------------- | ---------------------------------- | --------------------------------------------- |
| `SUCCESS`           | Items delivered.                   | Grant them and log the fulfillment.           |
| `PREVIEW`           | Dry run only.                      | Do not grant items; show the message.         |
| `NO_ITEMS`          | Nothing pending.                   | Tell the player there’s nothing to claim yet. |
| `RATE_LIMIT`        | Too many requests.                 | Ask the player to wait \~10 seconds.          |
| `ALREADY_CLAIMED`   | Another node already fulfilled it. | Treat as success.                             |
| `SERVER_NOT_FOUND`  | API key invalid or missing server. | Verify the key, rotate if needed.             |
| `PRODUCT_NOT_FOUND` | Product was deleted or disabled.   | Restore the product or escalate.              |

***

### 4. Production Checklist

* [ ] Store the raw API key in an environment variable or secrets manager on your server.
* [ ] Call the claim endpoint over HTTPS only.
* [ ] Validate player names before sending them to the API.
* [ ] Cache claim timestamps locally to avoid hammering the endpoint.
* [ ] Grant items atomically and log invoice/product IDs for support investigations.
* [ ] Monitor claim attempts, successes, and failures to catch key leaks early.

***

### 5. Quick cURL Test

```bash
curl -X POST "https://your-api-domain/api/v4/store/transaction/update" \
  -H "Content-Type: application/json" \
  -H "Authorization: $(printf %s "$TEAMGAMES_API_KEY" | base64)" \
  -d '{
    "playerName":"Player123",
    "preview": false,
    "includeRawTransactions": false
  }'
```

Replace `your-api-domain` with the host where your TeamGames API lives. Set `TEAMGAMES_API_KEY` (raw value) before running the command.

***

### 6. Legacy Array Response (v3)

Need to keep an older integration alive? `POST /api/v3/store/transaction/update` still returns the historical array and expects the `x-api-key` header. It does **not** include the status envelope or preview support, so plan your migration to v4 when possible.
