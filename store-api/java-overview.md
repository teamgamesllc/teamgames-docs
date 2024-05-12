# Java Overview

## Transaction Class

### Overview

The `Transaction` class facilitates interactions with the TeamGames store API, enabling the retrieval and management of transaction data.

### Example Usage: Fetching Transactions

To fetch transactions for a player "Nelson" using the `Transaction` class, you initialize the object, set the API key and player name, and call the `getTransactions` method. This endpoint will also set the transaction to claim to prevent further claiming. Here's how you do it:

```java
Transaction transaction = new Transaction()
.setApiKey("your_api_key_here").setPlayerName("Nelson");

try {
    Transaction[] transactions = transaction.getTransactions();
    if (transactions.length > 0) {
        for (Transaction trans : transactions) {
            System.out.println("Player: " + trans.player_name + ", Product: " + trans.product_name + ", Amount: " + trans.product_amount);
        }
    } else {
        System.out.println("No transactions found for the player.");
    }
} catch (Exception e) {
    System.err.println("Error fetching transactions: " + e.getMessage());
}
```

#### Constructor

* `Transaction()`: Initializes a new instance of the `Transaction` class.

#### Methods

* `setApiKey(String apiKey)`: Sets the API key required for authenticating requests. Returns the `Transaction` instance for method chaining.
* `setPlayerName(String playerName)`: Sets the player name for whom the transactions are queried. Returns the `Transaction` instance for method chaining.
* `getTransactions()`: Retrieves an array of `Transaction` objects representing the transactions made by the specified player. Returns empty if no transactions are found or if items have already been claimed.

#### Fields

* `player_name`: The name of the player involved in the transaction.
* `product_id`: The ID of the product purchased.
* `product_amount`: The quantity of the product purchased.
* `amount_purchased`: The total amount spent on the purchase.
* `product_name`: The name of the product.
* `product_price`: The price of the product per unit.
* `message`: A message associated with the transaction.

#### Exception

* Throws `Exception` if the API call fails.

***

## Post Class

### Overview

The `Post` class handles HTTP POST requests to the server.

#### Methods

* `sendPostData(Map<String, Object> params, String location, String apiKey)`: Sends a POST request with the specified parameters to the given location, using the provided API key. Returns the server response as a string.

#### Helper Methods

* `setRequestProperties(HttpURLConnection conn, byte[] postDataBytes)`: Sets the required properties for the outgoing HTTP POST request.

#### Exceptions

* Throws `Exception` if there are issues setting up or executing the HTTP request.

***
