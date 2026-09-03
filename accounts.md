# List Accounts

```
GET /api/v1/accounts
```

Returns the accounts your token can access: the ones you own, plus the ones you were invited to as a team member.

This is the entry point when you follow several merchants with a single token — it gives you the account UUIDs expected by [Account Balances](balances.md) and [List Payments](payments.md).

## Parameters

This endpoint does not take any parameter.

## Response

| Name | Type | Description |
|------|------|-------------|
| `success` | boolean | Indicates if the request was successful. |
| **accounts[]** | | Sorted by name. Empty if the token belongs to no account. |
| `accounts[].uuid` | string | Use it in the account-scoped endpoints. |
| `accounts[].name` | string | Legal name of the account. |
| `accounts[].display_name` | string | Brand name when set, otherwise `name`. |
| `accounts[].currency` | string | Default currency of the account (ISO 4217). |
| `accounts[].image_url` | string\|null | Account avatar, `null` when none is set. |
| `accounts[].is_owner` | boolean | `true` when you own the account, `false` when you are a team member. |

## Example

```json
{
  "accounts": [
    {
      "uuid": "acct_m4x2_ab12cd",
      "name": "Acme SAS",
      "display_name": "Acme",
      "currency": "EUR",
      "image_url": "https://squidlane.b-cdn.net/...",
      "is_owner": false
    }
  ],
  "success": true
}
```

## Typical follow-up loop

```
GET /api/v1/accounts
  → for each account uuid:
      GET /api/v1/account/<uuid>/balances
      GET /api/v1/account/<uuid>/payments?status=succeeded&from=2026-08-01
```
