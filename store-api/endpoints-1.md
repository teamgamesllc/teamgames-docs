---
hidden: true
---

# Endpoints

## TeamGames Storefront REST Endpoints

This reference walks through the two API calls required to power a custom TeamGames storefront: fetching the public catalog and submitting a cart for checkout.

### Quick Start

1. **Copy your API key** from the TeamGames dashboard.
2. **Fetch the catalog** with the Base64-encoded key to populate your store UI.
3. **Build a `cartItems` payload** using the catalog’s numeric `id` values.
4. **Submit checkout** via `POST /api/v2/client/global/checkout/complete` and redirect the player to the returned URL.
5. **Claim purchases** with `POST /api/v3/store/transaction/update` once the player returns in-game.

### Endpoint Overview

| Step | Purpose                        | Method & Path                                  | Auth                                      |
| ---- | ------------------------------ | ---------------------------------------------- | ----------------------------------------- |
| 1    | Fetch active catalog           | `POST /api/v2/client/global/products`          | `Authorization: <base64-encoded API key>` |
| 2    | Submit cart & initiate payment | `POST /api/v2/client/global/checkout/complete` | `Authorization: <base64-encoded API key>` |

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

### 4. Claim Purchased Items

Once checkout finishes and the player returns in-game, fetch undelivered items using the v3 claim endpoint.

```http
POST /api/v3/store/transaction/update HTTP/1.1
Host: <your-domain>
Content-Type: application/json
x-api-key: YOUR_SERVER_API_KEY

{
  "playerName": "Player123"
}
```

* `playerName` must match the value provided during checkout.
* The API key is the same one used for catalog/checkout requests.

#### Sample Response

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

Return values are legacy fields expected by existing claim scripts. Treat an empty array as “no items to claim.” Every response (including errors) uses HTTP 200 with a `message` field:

| Message                                                                              | Meaning                                 |
| ------------------------------------------------------------------------------------ | --------------------------------------- |
| `"Please wait a few seconds before trying again"`                                    | Per-player rate limit hit (10 seconds). |
| `"There are currently no items to claim."`                                           | No undelivered purchases found.         |
| `"The game server could not be found. This could be that the Api Key is incorrect."` | API key mismatch or inactive server.    |

After you grant the items in-game, the backend automatically marks the transactions as delivered.

***

### Next Steps

1. **Redirect immediately** when you receive a `redirect` URL.
2. **Handle callbacks/IPN**: keep `transactionId` handy so you can reconcile gateway notifications.
3. **Fulfill purchases** using `POST /api/v3/store/transaction/update` once payments settle (see above and `docs/store-server-integration.md`).
