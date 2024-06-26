# Endpoints

## Vote API Endpoint

The Vote API endpoint provides the necessary functionality to claim rewards based on voting by players. This section details how to make requests to this endpoint and handle the responses.

### Endpoint URL

<mark style="color:green;">`POST`</mark> `https://api.teamgames.io/v3/vote/reward/claim`

### Headers

<table><thead><tr><th width="177">Header</th><th width="249">Type</th><th>Description</th></tr></thead><tbody><tr><td>Authorization</td><td>string</td><td>Bearer token for API key authentication</td></tr></tbody></table>

#### Request Body

### Request Parameters

| Parameter | Description               |
| --------- | ------------------------- |
| `player`  | The name of the player.   |
| `reward`  | The reward identifier.    |
| `amount`  | The amount of the reward. |

### CURL Example

Here is an example of how to make a request to the Vote API using CURL:

<pre class="language-bash"><code class="lang-bash">curl -X POST https://api.teamgames.io/v3/vote/reward/claim \
-H "X-API-Key: your_api_key" \
<strong>-H "Content-Type: application/x-www-form-urlencoded; charset=UTF-8" \
</strong>-d "player=player123&#x26;reward=reward_id&#x26;amount=100"
</code></pre>
