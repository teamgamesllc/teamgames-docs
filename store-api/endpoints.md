# Endpoints

## Store Transaction Endpoint

### Endpoint URL

`POST https://api.teamgames.io/v3/store/transaction/update`

### Request Headers

* `Content-Type: application/x-www-form-urlencoded`
* `Authorization: Bearer <base64_encoded_api_key>`

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
  'https://api.teamgames.io/v3/store/transaction/update' \
  -H 'Authorization: Bearer <base64_encoded_api_key>' \  
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'playerName=Nelson'
```
