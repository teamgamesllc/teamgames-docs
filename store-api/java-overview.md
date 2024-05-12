# Java Overview

## Transaction Class

### Overview

The `Transaction` class facilitates interactions with the TeamGames store API, enabling the retrieval and management of transaction data.

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