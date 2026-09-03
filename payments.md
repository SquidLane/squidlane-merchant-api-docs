# List Payments

```
GET /api/v1/account/<account-uuid>/payments
```

Returns the payments received on the account, most recent first.

Only the account owner and the users who belong to the account team can call this endpoint. Any other token gets a `403`.

## Parameters

All parameters are optional. Unknown values are rejected with a `422`.

| Name | Type | Description |
|------|------|-------------|
| `status` | string | Filter on the payment status: `started`, `processing`, `succeeded`, `failed`, `canceled`, `refunded` or `chargeback`. All statuses are returned when omitted. |
| `mode` | string | Filter on the payment intent mode: `one_off` or `subscription`. |
| `currency` | string | Filter on the currency (ISO 4217, case insensitive). |
| `order_uuid` | string | Only the payments of that order. |
| `from` | date | Payments created at or after this date. |
| `to` | date | Payments created at or before this date. |
| `page` | int | Page number. Defaults to `1`. |
| `per_page` | int | Items per page. Defaults to `50`, capped at `200`. |

## Response

| Name | Type | Description |
|------|------|-------------|
| `success` | boolean | Indicates if the request was successful. |
| **payments[]** | | |
| `payments[].uuid` | string | Identifier of the payment. |
| `payments[].order_uuid` | string | Order the payment belongs to. |
| `payments[].order_external_id` | string|null | Custom identifier provided on the order. Never trust this data as it could be injected by the client in a checkout link. |
| `payments[].status` | string | `started`, `processing`, `succeeded`, `failed`, `canceled`, `refunded` or `chargeback`. |
| `payments[].paid_at` | datetime|null | Moment the payment was captured. `null` until it succeeds. |
| `payments[].invoiced_at` | datetime|null | |
| `payments[].amount_excl_tax` | float | |
| `payments[].amount_tax` | float | |
| `payments[].amount_incl_tax` | float | Amount actually charged, tax included. |
| `payments[].currency` | string | ISO 4217 currency code. |
| `payments[].lane.id` | int | |
| `payments[].lane.name` | string | |
| **payments[].recurrence** | | Describes whether this charge is part of a repeating series. |
| `payments[].recurrence.is_recurring` | boolean | `true` for a subscription charge or an installment charge. |
| `payments[].recurrence.type` | string | `one_off`, `subscription` or `installment`. |
| `payments[].recurrence.interval` | string|null | Billing interval of the subscription: `month`, `trimester`, `semester` or `year`. `null` outside of a subscription. |
| `payments[].recurrence.installment_count` | int|null | Total number of installments. `null` outside of an installment plan. |
| `payments[].recurrence.sequence` | int | Rank of this charge in its series, starting at `1`. |
| `payments[].recurrence.is_first_payment` | boolean | `true` for the initial charge of the series. |
| `payments[].recurrence.subscription_status` | string|null | `active`, `canceling`, `past_due` or `expired`. |
| `payments[].recurrence.current_period_ends_at` | date|null | End of the current subscription period. |
| `payments[].created_at` | datetime | |
| `payments[].updated_at` | datetime | |
| **pagination** | | |
| `pagination.current_page` | int | |
| `pagination.per_page` | int | |
| `pagination.last_page` | int | |
| `pagination.total` | int | Total number of payments matching the filters. |

## Example

```json
{
  "payments": [
    {
      "uuid": "pay_m4x2_a1b2c3",
      "order_uuid": "ord_m4x2_z9y8x7",
      "order_external_id": null,
      "status": "succeeded",
      "paid_at": "2026-08-18T09:14:22+00:00",
      "invoiced_at": "2026-08-18T09:14:25+00:00",
      "amount_excl_tax": "41.66",
      "amount_tax": "8.33",
      "amount_incl_tax": "49.99",
      "currency": "EUR",
      "lane": { "id": 12, "name": "Bonzai (Acme)" },
      "recurrence": {
        "is_recurring": true,
        "type": "subscription",
        "interval": "month",
        "installment_count": null,
        "sequence": 3,
        "is_first_payment": false,
        "subscription_status": "active",
        "current_period_ends_at": "2026-09-18"
      },
      "created_at": "2026-08-18T09:14:20+00:00",
      "updated_at": "2026-08-18T09:14:25+00:00"
    }
  ],
  "pagination": {
    "current_page": 1,
    "per_page": 50,
    "last_page": 4,
    "total": 187
  },
  "success": true
}
```

## Errors

| Status | Description |
|--------|-------------|
| `403` | The authenticated user does not have access to this account. |
| `404` | Unknown account UUID. |
| `422` | Invalid filter value. |
