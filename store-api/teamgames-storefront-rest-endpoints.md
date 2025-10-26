---
description: >-
  This reference walks through the two API calls required to power a custom
  TeamGames storefront: fetching the public catalog and submitting a cart for
  checkout.
---

# TeamGames Storefront REST Endpoints

### Quick Start

1. **Copy your API key** from the TeamGames dashboard.
2. **Fetch the catalog** with the Base64-encoded key to populate your store UI.
3. **Build a `cartItems` payload** using the catalog’s numeric `id` values.
4. **Submit checkout** via `POST /api/v2/client/global/checkout/complete` and redirect the player to the returned URL.
5. **Claim purchases** with `POST /api/v4/store/transaction/update` (reuse the Base64 `Authorization` header) once the player returns in-game—or `POST /api/v3/store/transaction/update` if you still depend on the legacy response format.

### Endpoint Overview

| Step | Purpose                                 | Method & Path                                  | Auth                                      |
| ---- | --------------------------------------- | ---------------------------------------------- | ----------------------------------------- |
| 1    | Fetch active catalog                    | `POST /api/v2/client/global/products`          | `Authorization: <base64-encoded API key>` |
| 2    | Submit cart & initiate payment          | `POST /api/v2/client/global/checkout/complete` | `Authorization: <base64-encoded API key>` |
| 3a   | Deliver purchases in-game (recommended) | `POST /api/v4/store/transaction/update`        | `Authorization: <base64-encoded API key>` |
| 3b   | Deliver purchases in-game (legacy)      | `POST /api/v3/store/transaction/update`        | `x-api-key: <server API key>`             |

***

### 1. Fetch the Product Catalog

```http
POST /api/v2/client/global/products HTTP/1.1
Host: <your-domain>
Content-Type: application/json
Authorization: BASE64_ENCODED_API_KEY

{}
```

* Encode the raw API key with Base64 (Node: `Buffer.from(apiKey).toString('base64')`, bash: `echo -n "API_KEY" | base64`).
* The body may be `{}`; some HTTP clients require a minimal payload.
* Disabled products and items outside their scheduled availability window are excluded automatically.

#### Sample Response

```json
{
  "message": "SUCCESS",
  "products": [
    {
      "id": 123,
      "productId": "sword_01",
      "name": "Super Sword",
      "price": 14.99,
      "quantity": 1,
      "description": "Best damage dealer.",
      "image": "https://cdn.example.com/assets/sword.png",
      "disabled": false,
      "sales": [
        {
          "saleType": "percentage",
          "discountPercentage": 10
        }
      ]
    }
  ]
}
```

Key fields:

* `id`: Numeric identifier required by checkout (use this in your cart payload).
* `productId`: Your own stable identifier for server-side logic.
* `sales`: Only active promotions within their date windows.

If the API key is invalid you receive:

```json
{
  "products": [],
  "message": "GAME_SERVER_NOT_FOUND",
  "extendedMessage": "The game server does not exist. Create one in the TeamGames dashboard."
}
```

Cache the catalog response and refresh when you publish product changes to avoid hitting rate limits.

***

### 2. Build the Cart Payload

Create an array of `{id, quantity}` objects using product `id` values. Before sending, JSON-stringify the array and place it in the `cartItems` field.

```json
[
  { "id": 123, "quantity": 2 },
  { "id": 456, "quantity": 1 },
  { "id": 789, "quantity": 3 }
]
```

In JavaScript that becomes:

```js
const cartItems = [
  { id: 123, quantity: 2 },
  { id: 456, quantity: 1 },
  { id: 789, quantity: 3 },
];

const cartPayload = JSON.stringify(cartItems);
```

Ship only the fields required by the API—extra data is ignored and may trigger validation errors.

***

### 3. Complete Checkout

```http
POST /api/v2/client/global/checkout/complete HTTP/1.1
Host: <your-domain>
Content-Type: application/json
Authorization: BASE64_ENCODED_API_KEY

{
  "username": "Player123",
  "cartItems": "[{\"id\":123,\"quantity\":2},{\"id\":456,\"quantity\":1},{\"id\":789,\"quantity\":3}]"
}
```

The API derives the game server from your key—no extra identifiers are required.

#### Success Payload

```json
{
  "statusCode": "200",
  "status": "SUCCESS",
  "redirect": "https://www.paypal.com/cgi-bin/webscr?token=EC-123...",
  "transactionId": "c5f3b8c4-4f74-4ac3-ae53-63a90e4cf50f",
  "isFree": false
}
```

* `redirect`: Send the player here immediately.
* `transactionId`: Persist for reconciliation and webhook handling.
* `isFree`: `true` when the total is zero—no redirect required.

#### Validation Responses

| Status                 | Meaning                                                                     | Recommended Action                                                        |
| ---------------------- | --------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| `INVALID_STOCK`        | Product availability changed during checkout.                               | Refresh the catalog data, update the cart, and ask the player to confirm. |
| `ERROR` (with message) | Gateway prerequisites unmet (e.g., minimum charge, gateway not configured). | Surface the message and let the player adjust their input.                |
| `error` property       | Coupon validation failed.                                                   | Display the error text and allow the player to fix it.                    |
| `statusCode: "400"`    | Basket violates gateway limits or integration is incomplete.                | Prompt the player to try another payment method or retry later.           |

#### HTTP Errors

* **401 Unauthorized**: Missing or incorrect API key—double-check your Base64 encoding.
* **404 Not Found**: Payload could not be parsed (malformed JSON or missing fields). Log the response body, rebuild the cart from the latest catalog snapshot, and try again.

***

### 4. Claim Purchased Items (v4)

Once checkout finishes and the player returns in-game, call the v4 claim endpoint to deliver pending purchases.

```http
POST /api/v4/store/transaction/update HTTP/1.1
Host: <your-domain>
Content-Type: application/json
Authorization: BASE64_ENCODED_API_KEY

{
  "playerName": "Player123",
  "preview": false
}
```

* `playerName` must match the value provided during checkout.
* Base64-encode the same server API key used for catalog/checkout requests and send it via the `Authorization` header.
* Set `preview` to `true` to perform a dry run (no items are granted and state remains unchanged).

#### Sample Response

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
    "rawTransactions": [
      {
        "productid": "42",
        "price": "14.99",
        "pricewithdiscount": "12.99",
        "quantity": 1,
        "givequantity": 5,
        "allowreclaim": 0,
        "username": "Player123",
        "invoice": "INV-12345",
        "paymenttype": "Stripe",
        "paymentstatus": "Paid",
        "deliverystatus": "Delivered"
      }
    ]
  }
}
```

`data.claims` mirrors the legacy response for quick integrations. `data.rawTransactions` exposes the original database rows with lowercase column names so you can inspect invoices, payment types, or other audit details. The v4 endpoint reuses the Base64 `Authorization` header pattern from catalog/checkout to keep authentication consistent.

Common `code` values:

| Code                | Meaning                                    |
| ------------------- | ------------------------------------------ |
| `SUCCESS`           | Items were delivered successfully.         |
| `PREVIEW`           | Dry-run mode; no state changes were made.  |
| `NO_ITEMS`          | No undelivered purchases found.            |
| `RATE_LIMIT`        | Per-player rate limit hit (10 seconds).    |
| `ALREADY_CLAIMED`   | Items were already delivered on this node. |
| `PRODUCT_NOT_FOUND` | Underlying product record is missing.      |
| `INVALID_PLAYER`    | Player name failed validation.             |
| `SERVER_NOT_FOUND`  | Supplied API key does not map to a server. |

***

### 5. Claim Purchased Items (v3 Legacy)

`POST /api/v3/store/transaction/update` remains available for older integrations. It returns the original array payload, does not support preview mode, and lacks machine-friendly status codes. Authenticate with the raw server key via the `x-api-key` header (legacy behavior retained for backwards compatibility).

```http
POST /api/v3/store/transaction/update HTTP/1.1
Host: <your-domain>
Content-Type: application/json
x-api-key: YOUR_SERVER_API_KEY

{
  "playerName": "Player123"
}
```

```json
[
  {
    "player_name": "Player123",
    "product_id": 42,
    "product_amount": 5,
    "amount_purchased": 1,
    "product_name": "Super Sword",
    "product_price": 14.99
  }
]
```

After you grant the items in-game, the backend automatically marks the transactions as delivered.

***

### Next Steps

1. **Redirect immediately** when you receive a `redirect` URL.
2. **Handle callbacks/IPN**: keep `transactionId` handy so you can reconcile gateway notifications.
3. **Fulfill purchases** using `POST /api/v4/store/transaction/update` once payments settle (see above and `docs/store-server-integration.md`). The legacy v3 endpoint remains available for older integrations.
