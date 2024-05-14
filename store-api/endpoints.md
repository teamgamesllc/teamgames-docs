# Endpoints

## Store Transaction Endpoint

### Endpoint URL

<mark style="color:green;">`POST`</mark> `https://api.teamgames.io/v3/store/transaction/claim`

### Headers

<table><thead><tr><th width="177">Header</th><th width="249">Type</th><th>Description</th></tr></thead><tbody><tr><td>Authorization</td><td>string</td><td>Bearer token for API key authentication</td></tr></tbody></table>

#### Request Body

### Request Parameters

| Parameter    | Type   | Description            |
| ------------ | ------ | ---------------------- |
| `playerName` | string | The name of the player |

### Response Format

The response is in JSON format and contains an array of `Transaction` objects.

#### Transaction Object

| Field              | Type   | Description                                        |
| ------------------ | ------ | -------------------------------------------------- |
| `player_name`      | string | The name of the player                             |
| `product_id`       | int    | The ID of the purchased product                    |
| `product_amount`   | int    | The amount of the product purchased                |
| `amount_purchased` | int    | The total amount of products purchased             |
| `product_name`     | string | The name of the purchased product                  |
| `product_price`    | double | The price of the purchased product                 |
| `message`          | string | A message indicating the status of the transaction |

### Example Request

```bash
curl -X POST \
  'https://teamgames.io/v3/store/transaction/claim' \
  -H "Authorization: Bearer your_api_key" \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'playerName=player123'
```
