---
hidden: true
---

# Custom Interface Endpoints

## TeamGames Storefront API Guide

Build a custom storefront or game-launcher shop that stays in sync with your TeamGames dashboard. This guide combines the three public endpoints you need—product catalog, cart management, and checkout—and explains how to stitch them together into a seamless player experience.

### Quick Start

1. **Get your credentials**: In the TeamGames dashboard create (or open) your game server and copy its API key.
2. **Fetch the catalog**: Call the catalog endpoint with the Base64-encoded API key to list active products, images, and sale metadata.
3. **Build the cart payload**: Capture the player’s selections and assemble the `cartItems` array described in `docs/store-add-to-cart-api.md`.
4. **Complete checkout**: Send the cart to the checkout endpoint. Redirect the player based on the response, then deliver items with the v3 claim API.

### Endpoint Overview

| Step | Purpose                        | Method & Path                                  | Auth                                      |
| ---- | ------------------------------ | ---------------------------------------------- | ----------------------------------------- |
| 1    | Fetch active catalog           | `POST /api/v2/client/global/products`          | `Authorization: <base64-encoded API key>` |
| 2    | Submit cart & initiate payment | `POST /api/v2/client/global/checkout/complete` | `Authorization: <base64-encoded API key>` |

***

### REST Endpoint Workflow

#### 1. Fetch the Product Catalog

Use the catalog endpoint to populate your product grid or in-game shop before building a cart.

#### Request

```http
POST /api/v2/client/global/products HTTP/1.1
Host: <your-domain>
Content-Type: application/json
Authorization: BASE64_ENCODED_API_KEY

{}
```

* Encode the raw API key with Base64 (Node: `Buffer.from(apiKey).toString('base64')`, bash: `echo -n "API_KEY" | base64`).
* The request body may be empty, but some clients require `{}`.
* The endpoint filters out disabled products and those outside their `startDate`/`endDate` window.

#### Response

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

* `id`: Numeric identifier required by cart/checkout calls.
* `productId`: Your own stable identifier; use it in game logic.
* `sales`: Active promotions already filtered to valid date ranges.

Cache the catalog response and refresh when you publish updates to avoid excessive polling. A missing/invalid key returns:

```json
{
  "products": [],
  "message": "GAME_SERVER_NOT_FOUND",
  "extendedMessage": "The game server does not exist. Create one in the TeamGames dashboard."
}
```

***

#### 2. Build the Cart Payload

API clients create and maintain the cart locally. Assemble an array of `{id, quantity}` objects using the numeric `id` values from the catalog response.

```json
[
  { "id": 123, "quantity": 2 },
  { "id": 456, "quantity": 1 }
]
```

Before submitting the cart, convert the array to a JSON string. The checkout endpoint expects the payload in a string field named `cartItems`.

```js
const cartItems = [
  { id: 123, quantity: 2 },
  { id: 456, quantity: 1 },
  { id: 789, quantity: 3 },
];

const cartPayload = JSON.stringify(cartItems);
```

Keep a fresh copy of the catalog handy so you can validate product availability and pricing before checkout. See `docs/store-add-to-cart-api.md` for deeper guidance on handling stock changes, coupon errors, and other edge cases.

#### 3. Complete Checkout

Send the encoded cart and username to the checkout endpoint. TeamGames derives the correct game server from your API key and responds with a redirect URL.

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

On success you receive:

```json
{
  "statusCode": "200",
  "status": "SUCCESS",
  "redirect": "https://www.paypal.com/cgi-bin/webscr?token=EC-123...",
  "transactionId": "c5f3b8c4-4f74-4ac3-ae53-63a90e4cf50f",
  "isFree": false
}
```

Redirect the player immediately and persist `transactionId` for reconciliation. After payment settles, call the v3 claim endpoint (`docs/store-server-integration.md`) to deliver purchases in-game.

The endpoint returns HTTP 200 even when validation fails. Inspect the payload:

| Status                 | Meaning                                                                                                     | Recommended Action                                                     |
| ---------------------- | ----------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| `INVALID_STOCK`        | Product stock changed during checkout. Payload includes `cartItems` and `productsAffected`.                 | Refresh cart, show updated quantities, and ask the shopper to confirm. |
| `ERROR` with `message` | Gateway prerequisites not met (for example, minimum charge requirements or an unconfigured PayPal account). | Display the message and let the user adjust.                           |
| `error` property       | Coupon validation failed.                                                                                   | Surface the error text and keep the shopper on the checkout page.      |
| `statusCode: "400"`    | Basket violates gateway limits or integration misconfigured.                                                | Offer a different payment method or retry later.                       |

If you receive an HTTP 401, the API key was missing or invalid. A 404 usually means the cart payload could not be parsed—log the response, rebuild the array from the latest catalog snapshot, and retry.

***

### Java SDK Quickstart

Prefer the Java helper library? Follow these steps:

1. **Set environment variables**
   * `TEAMGAMES_API_KEY` — required. The sample code reads this value.
   * `TEAMGAMES_RUN_CHECKOUT=true` — optional. When set, the demo submits the checkout; otherwise it only prints the payload.
2.  **Run the demo**

    ```bash
    javac com/teamgames/tests/store/StoreApiDemo.java
    java com.teamgames.tests.store.StoreApiDemo
    ```
3. **Review the output** to inspect catalog results, the generated `cartItems` JSON, and (if enabled) the redirect details returned by checkout.

#### Java Helpers

Use the same building blocks in your own project:

```java
import com.teamgames.endpoints.store.StoreCatalog;
import com.teamgames.endpoints.store.StoreCheckout;
import com.teamgames.endpoints.store.StoreCheckout.CheckoutResponse;

String apiKey = System.getenv("TEAMGAMES_API_KEY");

StoreCatalog catalogClient = new StoreCatalog().setApiKey(apiKey);
StoreCatalog.CatalogResponse catalog = catalogClient.fetch();

StoreCheckout checkout = new StoreCheckout()
    .setApiKey(apiKey)
    .setUsername("Player123");

long[][] selections = {
    {catalog.products[0].id, 1},
    {catalog.products[1].id, 2},
    {catalog.products[2].id, 3}
};

for (long[] selection : selections) {
    checkout.addItem(selection[0], (int) selection[1]);
}

CheckoutResponse response = checkout.submit();

if ("SUCCESS".equals(response.status)) {
    System.out.println("Redirect: " + response.redirect);
}
```

The demo class lives at `com.teamgames.tests.store.StoreApiDemo` inside the repository for quick experimentation.

***

### Bringing It All Together

1. **Catalog sync**: Cache catalog responses and refresh when you change products to avoid rate limits.
2. **Cart persistence**: Store pending carts on your side (database, in-memory cache, etc.) so you can recreate the payload if the shopper reconnects or if checkout validation fails.
3. **Payment handoff**: Redirect immediately when checkout returns `redirect`. Listen for gateway callbacks (PayPal IPN, Stripe webhooks, etc.) on your server or rely on TeamGames’ built-in handlers.
4. **Fulfillment**: Poll the v3 claim endpoint (`POST /api/v3/store/transaction/update`) from your game server to deliver items once the order is marked delivered.

### Related Resources

* `docs/store-products-api.md` — Deep dive into the catalog response.
* `docs/store-add-to-cart-api.md` — Detailed cart management scenarios.
* `docs/store-checkout-api.md` — Full checkout flow reference.
* `docs/store-server-integration.md` — Claim purchases inside your game server.
