# Endpoints

## Leaderboard API Documentation

This endpoint is used to update the leaderboard with the player's latest skill data. It requires several pieces of information about the player and the game mode they are playing in.

### Endpoint URL

<mark style="color:green;">`POST`</mark> `https://api.teamgames.io/v3/leaderboard/update`

### Headers

<table><thead><tr><th width="192">Header</th><th width="249">Type</th><th>Description</th></tr></thead><tbody><tr><td>Authorization</td><td>string</td><td>Bearer token for API key authentication</td></tr></tbody></table>

#### Request Body

| Parameter  | Type   | Description                              | Required |
| ---------- | ------ | ---------------------------------------- | -------- |
| gameMode   | string | Game mode identifier                     | Yes      |
| playerName | string | Player name                              | Yes      |
| ipAddress  | string | Player's IP address (optional)           | No       |
| userRole   | string | Player's role (optional)                 | No       |
| skills     | array  | Array of player's skill data (see below) | Yes      |

Each object in the `skills` array should have the following structure:

| Parameter  | Type    | Description      |
| ---------- | ------- | ---------------- |
| name       | string  | Skill name       |
| level      | integer | Skill level      |
| experience | integer | Skill experience |

#### Response

The endpoint will return a response indicating the status of the leaderboard update.

#### Example Request

```
POST /api/v3/leaderboard/update
Authorization: Bearer <apiKey>
Content-Type: application/json

{
  "secret": "<apiKey>",
  "gameMode": "Normal Mode",
  "playerName": "Nelson",
  "skills": [
    {
      "name": "Attack",
      "level": 99,
      "experience": 1000000
    },
    {
      "name": "Defense",
      "level": 99,
      "experience": 1000000
    },
    ...
  ]
}
```

**Example Response**\
\
`{ "message": "Leaderboard updated successfully" }`
