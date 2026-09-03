# Update an Order Item

```
PATCH /api/v1/order-item/<order-item-id>
```

Sets the price charged on the **next** billing cycle of an active subscription — up or down.

Nothing is charged or refunded at the moment of the call. The customer keeps the cycle they
already paid for, and the new price takes effect on the following renewal. Because of that,
an increase and a decrease behave identically: no proration, no card challenge, no customer
in front of a screen.

Requires a **Full access** token. See [Authentication](authentication.md).

## Finding the order item id

Call [Retrieve Order](retrieve-order.md) and pick the line you want to reprice from
`order.items[]`. The `id` field of that line is what goes in the URL.

An order can hold several subscription lines. Repricing one recomputes the order total and
pushes that new total to the payment provider, so subscribe-and-forget integrations only
need to touch the line whose price changed.

## Parameters

| Name | Type | Description |
|------|------|-------------|
| `new_unit_amount` | float | **Required.** New unit price for the line. Between `0.01` and `999999.99`. Tax-inclusive or tax-exclusive following the line's own `is_tax_incl`, exactly like the price you set when the order was created. |

The line total becomes `new_unit_amount × quantity`, and the order total is recomputed from
all its lines before being sent to the payment provider.

The resulting **subscription total, excluding tax**, must be at least **0.50** in the order's
currency. A single line can go below that as long as the other lines bring the total back up.
Note that the floor applies to the tax-exclusive total: with 20% VAT, a tax-inclusive total of
`0.59` sits at `0.49` excluding tax and is refused.

## When the new price applies

At the end of the current billing period. The response's `order.current_period_ends_at` is
the date the customer will first be charged the new amount — read it back if you need to
tell the customer when the change takes effect.

## Response

`200` returns the full order in the same shape as [Retrieve Order](retrieve-order.md):

```json
{
  "success": true,
  "order": { "...": "see Retrieve Order" }
}
```

## Errors

| Status | `error_code` | Meaning |
|--------|--------------|---------|
| `422` | `invalid_amount` | `new_unit_amount` is missing, not a number, or outside `0.01`–`999999.99`. |
| `422` | `not_a_subscription_item` | The line is a one-off purchase, not a subscription. |
| `422` | `subscription_not_active` | The subscription is not active (cancelled, expired, past due, or never started). |
| `422` | `not_a_subscription_intent` | The order is an installment plan, not a subscription. Installment schedules cannot be repriced. |
| `422` | `no_payment_intent` | The order was never paid. |
| `422` | `gateway_unsupported` | The payment provider on this order does not support repricing. |
| `422` | `gateway_subscription_missing` | No provider-side subscription is attached to this order. |
| `422` | `gateway_subscription_not_active` | The subscription is in a state the provider will not reprice — typically a free trial, or a plan change already in flight. |
| `422` | `amount_below_gateway_minimum` | The resulting subscription total is under the 0.50 minimum. |
| `502` | `gateway_update_failed` | The payment provider refused the new amount. **The order is left exactly as it was** — retrying is safe. |
| `403` | — | The token is read-only, or the order belongs to another account. |
| `404` | — | No order item with this id. |

## Example

```bash
curl -X PATCH https://www.squidlane.com/api/v1/order-item/8412 \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{"new_unit_amount": 149.90}'
```

## Notes

- **Promo codes are dropped.** `new_unit_amount` is the final price. If the line still
  carried a promo code, the code is detached so the amount you send is the amount charged.
- **Entry fees are untouched.** A subscription's first-cycle amount describes a cycle that
  is already billed; only the recurring amount moves.
- **Every change is recorded** against the order item, with the previous amount, the new
  amount, and who made the change — whether it came from the API or the dashboard.
- **A pending change can be replaced.** A subscription carries at most one pending plan
  change, but sending a new one before the effective date simply replaces it — you do not
  need to cancel anything first. Useful to correct a wrong amount before it is charged.
- **Your webhook fires.** If the account has a webhook URL, an order update is sent once the
  change is confirmed by the payment provider. See [Webhooks](webhooks.md).
