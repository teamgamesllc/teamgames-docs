# Endpoints

## Leaderboard API Documentation

This endpoint is used to update the leaderboard with the player's latest skill data. It requires several pieces of information about the player and the game mode they are playing in.

### Endpoint URL

<mark style="color:green;">`POST`</mark> `https://api.teamgames.io/v3/leaderboard/metrics/update`

### Headers

<table><thead><tr><th width="192">Header</th><th width="249">Type</th><th>Description</th></tr></thead><tbody><tr><td>Authorization</td><td>string</td><td>Bearer token for API key authentication</td></tr></tbody></table>

#### Request Body

| Parameter  | Type   | Description                                          | Required |
| ---------- | ------ | ---------------------------------------------------- | -------- |
| gameMode   | string | Game mode identifier                                 | Yes      |
| playerName | string | Player name                                          | Yes      |
| ipAddress  | string | Player's IP address (optional)                       | No       |
| userRole   | string | Player's role (optional)                             | No       |
| metrics    | array  | Array of player's skill or activity data (see below) | Yes      |

Each object in the `skills` array should have the following structure:

| Parameter | Type    | Description     |
| --------- | ------- | --------------- |
| name      | string  | Metric name     |
| value     | integer | Metric value    |
| progress  | integer | Metric progress |

#### Response

The endpoint will return a response indicating the status of the leaderboard update.

#### Example Request

```
POST /api/v3/leaderboard/metrics/update
Authorization: Bearer <apiKey>
Content-Type: application/json

{
  "gameMode": "Normal Mode",
  "playerName": "Nelson",
  "metrics": [
    {
      "name": "Attack",
      "value": 99,
      "progress": 200000000
    },
    {
      "name": "Wins",
      "value": 1
    },
    ...
  ]
}
```

### CURL Example

Here is an example of how to make a request to the Vote API using CURL:

```bash
curl -X POST "https://api.teamgames.io/v3/leaderboard/metrics/update" \
     -H "Authorization: Bearer your_api_key" \
     -H "Content-Type: application/x-www-form-urlencoded; charset=UTF-8" \
     --data-urlencode "gameMode=Normal Mode" \
     --data-urlencode "playerName=player123" \
     --data-urlencode "metrics=[
         {
             \"name\": \"Cooking\",
             \"value\": 99,
             \"progress\": 200000000
         },
         {
             \"name\": \"Wins\",
             \"value\": 10
         }
     ]"

```
