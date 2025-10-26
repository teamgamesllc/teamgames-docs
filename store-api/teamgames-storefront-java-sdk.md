---
description: >-
  Welcome! This guide walks you through the Java helpers that ship with the
  TeamGames platform. It is written for developers who are new to the SDK and
  want a practical, end‑to‑end integration path.
---

# TeamGames Storefront Java SDK

### 1. Before You Start

| Requirement                                    | Why it matters                                                                                |
| ---------------------------------------------- | --------------------------------------------------------------------------------------------- |
| Java 8 or newer                                | All helpers compile against Java 8 bytecode.                                                  |
| TeamGames API key (`TEAMGAMES_API_KEY`)        | Authenticates every request. Grab it from the TeamGames dashboard.                            |
| Internet access to `https://api.teamgames.io/` | The helpers point to this domain by default.                                                  |
| Optional: local API sandbox                    | Toggle by calling `JsonPost.setLocal(true)` / `Post.setLocal(true)` before issuing a request. |

> Tip: Export your API key once in your shell profile (`export TEAMGAMES_API_KEY=...`) so every demo and integration code pick it up automatically.

***

### 2. Quick Start (10 Minutes)

1. **Open the SDK tests directory**\
   `java-library/teamgames-api/src/com/teamgames/tests/store`
2.  **Compile the sample**

    ```bash
    cd java-library/teamgames-api/src
    javac com/teamgames/tests/store/StoreApiDemo.java
    ```
3.  **Run the full flow**

    ```bash
    export TEAMGAMES_API_KEY=your_api_key_here
    # Optional: export TEAMGAMES_RUN_CHECKOUT=true   # performs a live checkout
    java com.teamgames.tests.store.StoreApiDemo
    ```
4. **Observe the output**
   * Catalog summary
   * Optional checkout redirect
   * Ready-made snippet showing how to grant purchases

Once the demo works with your credentials, you can embed the same client classes into your own project.

***

### 3. Core Building Blocks

| Class                                            | Type            | What it does                                                                       |
| ------------------------------------------------ | --------------- | ---------------------------------------------------------------------------------- |
| `StoreCatalogClient`                             | Facade          | Fetches the product catalog (sync or async).                                       |
| `StoreCheckoutClient`                            | Facade          | Builds carts and submits checkout requests.                                        |
| `StoreClaimClient`                               | Facade          | Retrieves undelivered purchases and marks them delivered.                          |
| `StoreCatalog` / `StoreCheckout` / `Transaction` | Builders & DTOs | Provide strongly typed requests/responses underneath each facade.                  |
| `JsonPost` / `Post`                              | HTTP helpers    | Handle headers, environment switching, pooled connections, and timeouts.           |
| `Thread.executor`                                | Shared executor | Default async runner used across clients (override with your own pools if needed). |

Usage pattern:

1. Instantiate a client once per API key.
2. Call `newRequest()` (or equivalent) to get a single-use builder.
3. Configure the request (username, cart items, preview flag).
4. Execute synchronously (`execute()`, `fetch()`, `submit()`) or asynchronously (`executeAsync()`, etc.).

***

### 4. Deliver Purchases (Claim API)

```java
import com.teamgames.endpoints.store.StoreClaimClient;
import com.teamgames.endpoints.store.Transaction;

StoreClaimClient claimClient = new StoreClaimClient(apiKey);
Transaction.ClaimResponse claimResponse = claimClient.newRequest()
    .playerName("Player123")
    .useV4Endpoint()   // structured envelope (recommended)
    .preview(false)    // set to true for dry runs
    .execute();

if ("SUCCESS".equals(claimResponse.status)) {
    for (Transaction tx : claimResponse.data.claims) {
        System.out.printf("Grant product %s (id=%s) x %d%n",
            tx.product_name,
            tx.product_id_string,
            tx.product_amount);
    }
} else if ("PREVIEW".equals(claimResponse.status)) {
    System.out.println("Preview only: " + claimResponse.message);
} else {
    System.out.println("Claim error: " + claimResponse.code + " :: " + claimResponse.message);
}
```

Need audit info? Each response also contains `claimResponse.data.rawTransactions` as `JsonObject[]`:

```java
for (com.teamgames.lib.gson.JsonObject raw : claimResponse.data.rawTransactions) {
    String invoice = raw.has("invoice") ? raw.get("invoice").getAsString() : null;
    String gateway = raw.has("paymenttype") ? raw.get("paymenttype").getAsString() : null;
    System.out.printf("Invoice %s paid via %s%n", invoice, gateway);
}
```

> `ClaimResponse.code` will be `SUCCESS`, `NO_ITEMS`, `RATE_LIMIT`, `SERVER_NOT_FOUND`, etc. Show those messages to your players for friendlier feedback.

***

### 5. Optional: Build a Custom Storefront UI

Want to mirror the web store inside your game launcher or website? Use the catalog and checkout helpers below. Skip this section if you only need to fulfill purchases.

#### 5.1 Fetch the Product Catalog

```java
import com.teamgames.endpoints.store.StoreCatalogClient;

String apiKey = System.getenv("TEAMGAMES_API_KEY");
StoreCatalogClient catalogClient = new StoreCatalogClient(apiKey);

StoreCatalog.CatalogResponse catalog = catalogClient.fetch();
System.out.println("Products available: " + catalog.products.length);
```

Need async?

```java
catalogClient.fetchAsync()
    .thenAccept(result -> System.out.println("Fetched " + result.products.length + " products"))
    .exceptionally(ex -> { ex.printStackTrace(); return null; });
```

#### 5.2 Build and Submit Checkout

```java
import com.teamgames.endpoints.store.StoreCheckoutClient;
import com.teamgames.endpoints.store.StoreCheckout;

StoreCheckoutClient checkoutClient = new StoreCheckoutClient(apiKey);
StoreCheckoutClient.CheckoutRequest checkout = checkoutClient.newRequest()
    .username("Player123");

checkout.addItem(42, 1);   // product id, quantity
checkout.addItem(84, 3);

StoreCheckout.CheckoutResponse response = checkout.submit();
if ("SUCCESS".equals(response.status)) {
    System.out.println("Redirect player to: " + response.redirect);
} else {
    System.out.println("Checkout failed: " + response.code + " :: " + response.message);
}
```

Async submission works the same way using `submitAsync()` (optionally with your own executor).

***

### 6. Going Async

Every client exposes `*Async()` variants. By default they use `Thread.executor`, a shared thread pool that ships with the SDK. To control concurrency:

```java
ExecutorService gameExecutor = Executors.newFixedThreadPool(4);

claimClient.newRequest()
    .playerName("Player123")
    .useV4Endpoint()
    .executeAsync(gameExecutor)
    .thenAccept(response -> grantItems(response))
    .exceptionally(ex -> { log.error("Claim failed", ex); return null; });
```

Remember: each `newRequest()` builder is single-use. Create a fresh one for every call.

***

### 7. HTTP Configuration & Connection Pooling

The SDK reuses HTTP connections and sets sensible defaults:

| Setting                | Default    | How to override                               |
| ---------------------- | ---------- | --------------------------------------------- |
| Connect timeout        | 10 seconds | `-Dteamgames.http.connectTimeoutMs=5000`      |
| Read timeout           | 15 seconds | `-Dteamgames.http.readTimeoutMs=15000`        |
| Keep-alive             | Enabled    | `-Dteamgames.http.keepAlive=false` to disable |
| Max pooled connections | 50         | `-Dteamgames.http.maxConnections=100`         |

Add the flags to your JVM or application server startup command. Keep-alive dramatically improves performance on busy servers—leave it on unless a proxy requires otherwise.

***

### 8. Local Testing & Environments

Point the helpers at a development server:

```java
JsonPost.setLocal(true); // applies to the next request on the same thread
Post.setLocal(true);
```

Once the request finishes, the helper automatically clears the flag for that thread.

***

### 9. Troubleshooting

| Symptom                                              | Likely cause                                        | Quick fix                                                                                  |
| ---------------------------------------------------- | --------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| `IllegalArgumentException: API key must not be null` | Forgot to set the env variable                      | `export TEAMGAMES_API_KEY=...` (or call `.setApiKey(...)`).                                |
| `HTTP 401` or `SERVER_NOT_FOUND`                     | Wrong/rotated API key                               | Refresh the key in the dashboard and redeploy.                                             |
| `NO_ITEMS` on claim                                  | Nothing pending for that player                     | Verify the username matches checkout exactly; try preview mode first.                      |
| `RATE_LIMIT` on claim                                | Player is requesting too fast                       | Cache the last claim timestamp and wait \~10 seconds.                                      |
| `JsonSyntaxException`                                | Unexpected payload (network proxy, HTML error page) | Log the raw response body and HTTP status; fire a support ticket if TeamGames API is down. |

Still stuck? Check the SDK tests (`StoreClaimClientTest`, `TestStoreCommand`) for working examples, or open an issue with logs attached.

***

### 10. Where to Go Next

* **REST endpoints** — Need raw HTTP details or building a non-Java integration? See `docs/storefront-rest-endpoints.md`.
* **Server delivery guide** — Step-by-step instructions for handing out items in-game: `docs/store-server-integration.md`.
* **Checkout deep dive** — Advanced validation and error handling: `docs/store-checkout-api.md`.

Happy shipping! If the docs miss something you expected, please let the team know so the next developer has an even smoother journey.
