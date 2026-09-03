# Account Balances

```
GET /api/v1/account/<account-uuid>/balances
```

Returns the **live** Bonzai balances of the account, one entry per active Bonzai lane.

Nothing is cached: every call queries Bonzai directly. If Bonzai is unreachable the endpoint answers `503` rather than reporting a zero balance.

Only the account owner and the users who belong to the account team can call this endpoint. Any other token gets a `403`.

## Parameters

This endpoint does not take any parameter. The account is determined by the account UUID in the URL.

## Response

| Name | Type | Description |
|------|------|-------------|
| `success` | boolean | Indicates if the request was successful. |
| `retrieved_at` | datetime | Moment the balances were read from Bonzai. |
| **balances.lanes[]** | | One entry per active lane connected to Bonzai. Empty if the account has no Bonzai lane. |
| `balances.lanes[].lane_id` | int | |
| `balances.lanes[].lane_name` | string | |
| `balances.lanes[].bonzai_team_id` | int | Identifier of the lane on the Bonzai side. |
| `balances.lanes[].balances` | object | Keyed by currency (ISO 4217). |
| `balances.lanes[].balances.<CUR>.balance` | float | Total balance held in that currency. |
| `balances.lanes[].balances.<CUR>.available_to_withdraw` | float | Part of the balance that can be withdrawn right now. |
| **balances.totals** | object | Sum of every lane, keyed by currency. Same `balance` / `available_to_withdraw` fields. |

Currency-keyed objects are serialized as an empty array (`[]`) when they contain no currency.

## Example

```json
{
  "balances": {
    "lanes": [
      {
        "lane_id": 12,
        "lane_name": "Bonzai (Acme)",
        "bonzai_team_id": 123,
        "balances": {
          "EUR": { "balance": 4210.55, "available_to_withdraw": 3980.00 },
          "USD": { "balance": 120.00, "available_to_withdraw": 120.00 }
        }
      }
    ],
    "totals": {
      "EUR": { "balance": 4210.55, "available_to_withdraw": 3980.00 },
      "USD": { "balance": 120.00, "available_to_withdraw": 120.00 }
    }
  },
  "retrieved_at": "2026-08-19T10:32:07+00:00",
  "success": true
}
```

## Errors

| Status | Description |
|--------|-------------|
| `403` | The authenticated user does not have access to this account. |
| `404` | Unknown account UUID. |
| `503` | Bonzai is unreachable or returned an error. Retry later; no balance is returned. |
