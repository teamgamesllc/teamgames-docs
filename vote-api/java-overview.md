# Java Overview

## Vote API

The Vote API in the TeamGames framework enables developers to integrate voting functionalities, specifically for claiming vote rewards within games. This documentation provides a comprehensive guide on using the `VoteEndpoint` class to interface with the TeamGames system to validate and claim rewards for players based on their voting actions.

### Classes and Methods

#### `VoteEndpoint` Class

`com.teamgames.endpoints.vote.VoteEndpoint`

**Constructors**

* `public VoteEndpoint()`
  * Initializes a new instance of the `VoteEndpoint` class.

**Methods**

* `public VoteEndpoint setApiKey(String apiKey)`
  * Sets the API key required for authentication.
  * Returns `this` for chaining method calls.
* `public VoteEndpoint setPlayerName(String playerName)`
  * Sets the player's name for whom the reward is being claimed.
  * Returns `this` for chaining method calls.
* `public VoteEndpoint setRewardId(String rewardId)`
  * Sets the identifier for the reward to be claimed.
  * Returns `this` for chaining method calls.
* `public VoteEndpoint setAmount(String amount)`
  * Sets the amount of the reward to be claimed.
  * Returns `this` for chaining method calls.
* `public ClaimReward getReward() throws Exception`
  * Retrieves a `ClaimReward` object representing the reward for a player.
  * Calls the TeamGames Vote API to validate the player's vote and claim the designated reward.
  * Throws `Exception` if there is an error during the HTTP POST request or while parsing the response.

#### `Post` Class

`com.teamgames.https.Post`

**Methods**

* `public static String sendPostData(Map<String, Object> params, String location, String apiKey) throws Exception`
  * Sends a POST request to the specified location with the provided parameters and API key.
  * Constructs the request URL, encodes the parameters, sets the request properties, and handles the response.
  * Throws `Exception` for any errors encountered during the request.

**Utility Methods**

* `public static void setRequestProperties(HttpURLConnection conn, byte[] postDataBytes) throws Exception`
  * Sets the necessary HTTP headers for the POST request.
  * Throws `Exception` if unable to set the properties.

### Usage Example

Below is an example of how to use the `VoteEndpoint` to claim a reward:

```java
VoteEndpoint voteEndpoint = new VoteEndpoint();
voteEndpoint.setApiKey("your_api_key")
            .setPlayerName("player123")
            .setRewardId("reward_id")
            .setAmount("100");
try {
    ClaimReward reward = voteEndpoint.getReward();
    System.out.println("Reward claimed: " + reward);
} catch (Exception e) {
    System.err.println("Error claiming reward: " + e.getMessage());
}
```
