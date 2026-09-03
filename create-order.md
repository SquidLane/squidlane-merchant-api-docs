# Create an Order

```
POST /api/v1/orders
```

Creates a new order and returns a `checkout_access_url` — a signed link (valid 7 days) to redirect your customer to the checkout page.

> **Checkout URL parameters for static offers**
>
> For static offers, you can append parameters to the checkout URL (available on the product page) to pre-fill information for your customers.
> These parameters are client-controlled and can be easily altered.
> **They must never be trusted as-is.**
> Always validate, sanitize, and enforce all critical values on your server before processing them.

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `offer` | string | **Yes** | UUID of the offer to purchase (e.g. `"offr_fve_FHfOP6"`). |
| `email` | string | No | Customer's email address. Pre-fills the email field at checkout. |
| `billing_email_locked` | boolean | No | If `true`, the customer cannot change the billing email. Requires `email` to be set. |
| `processSavedPaymentMethod` | boolean | No | If `true`, the hosted checkout first tries the saved payment method from a previous successful order in the same browser session and account. It falls back to the normal checkout if unavailable or declined. |
| `external_id` | string | No | Custom identifier for this order. Not unique — can be reused across orders. Searchable in your order history. |
| `metadata` | object | No | Custom key-value metadata for this order. |
| `success_url` | string | No | URL to redirect the customer after a successful payment. |
| `cancel_url` | string | No | URL to redirect the customer after a canceled payment. |
| `type` | string | No | Billing address type (`personal` or `business`). |
| `firstname` | string | No | Billing address first name. |
| `lastname` | string | No | Billing address last name. |
| `company_name` | string | No | Billing address company name. |
| `phone` | string | No | Billing address phone number. |
| `line1` | string | No | Billing address line 1. |
| `line2` | string | No | Billing address line 2. |
| `postal_code` | string | No | Billing address postal code. |
| `city` | string | No | Billing address city. |
| `state` | string | No | Billing address state. |
| `country` | string | No | Billing address country (ISO 3166-2 alpha-2, e.g. `"FR"`). |
| `registration_number` | string | No | Billing address registration number (business). |
| `vat_number` | string | No | Billing address VAT number (business). |

For a static offer link, use for example:

```text
https://www.squidlane.com/buy/offr_sta_XXXXXX?email=customer%40example.com&processSavedPaymentMethod=true
```

## Additional parameters for flexible offers

For **flexible offers**, the following fields are required at the **top level** of the request body, alongside `offer`.

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `amount` | number | **Yes** | Amount of the order item. Min: `3`, max: `50000`. |
| `currency` | string | **Yes** | Currency of the order item. Accepted values: `EUR`, `USD`, `GBP`. |
| `is_tax_incl` | boolean | **Yes** | Whether the amount includes tax. |
| `mode` | string | **Yes** | `one_off` or `subscription`. |
| `interval` | string | **Yes if `mode` is `subscription`** | Billing interval. Accepted values: `month`, `trimester`, `semester`, `year`. |
| `title` | string | No | Title of the order item. |
| `description` | string | No | Description of the order item. |
| `quantity` | integer | No | Quantity (default: `1`). |
| `first_amount` | number | No | First payment amount, if different from the recurring `amount`. Min: `3`, max: `50000`. |
| `trial_days` | integer | No | Free trial duration for a subscription. Min: `1`, max: `365`. A trial subscription must be the only item in the order. |
| `allowed_installment_count` | integer | No | Number of installments for `one_off` mode. Min: `2`, max: `12`. |

## Response

| Name | Type | Always present | Description |
|------|------|----------------|-------------|
| `success` | boolean | **Yes** | `true` if the order was created. |
| `order` | object | No | Present if `success` is `true`. |
| `order.uuid` | string | No | UUID of the created order. |
| `order.checkout_access_url` | string | No | Signed URL to redirect your customer to checkout. Valid for 7 days. |
| `order.embed_url` | string | No | Signed URL for the [embedded checkout](embed.md) (render the card form inside your own app). Valid for 24 hours. |
| `error` | string | No | Error message if the request failed. |
| `error_code` | string | No | Machine-readable error code. |

## Examples

### Static offer

```bash
curl -X POST https://www.squidlane.com/api/v1/orders \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "offer": "offr_sta_XXXXXX",
    "email": "customer@example.com",
    "billing_email_locked": true,
    "processSavedPaymentMethod": true,
    "external_id": "my-order-123",
    "success_url": "https://yourapp.com/success",
    "cancel_url": "https://yourapp.com/cancel"
  }'
```

### Flexible offer — one-time payment

```bash
curl -X POST https://www.squidlane.com/api/v1/orders \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "offer": "offr_fve_XXXXXX",
    "amount": 99.00,
    "currency": "EUR",
    "is_tax_incl": true,
    "mode": "one_off",
    "email": "customer@example.com",
    "billing_email_locked": true,
    "processSavedPaymentMethod": true,
    "external_id": "my-order-123",
    "metadata": { "workspace_id": 1 },
    "success_url": "https://yourapp.com/success",
    "cancel_url": "https://yourapp.com/cancel"
  }'
```

### Flexible offer — subscription

```bash
curl -X POST https://www.squidlane.com/api/v1/orders \
  -H "Authorization: Bearer <your-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "offer": "offr_fve_XXXXXX",
    "amount": 29.00,
    "currency": "EUR",
    "is_tax_incl": true,
    "mode": "subscription",
    "interval": "month",
    "email": "customer@example.com",
    "billing_email_locked": true,
    "external_id": "my-order-123",
    "metadata": { "workspace_id": 1 },
    "success_url": "https://yourapp.com/success",
    "cancel_url": "https://yourapp.com/cancel"
  }'
```

### Success response

```json
{
  "success": true,
  "order": {
    "uuid": "ord_XXXXXX",
    "checkout_access_url": "https://www.squidlane.com/checkout/...",
    "embed_url": "https://www.squidlane.com/api/v1/embed/ord_XXXXXX?expires=...&signature=..."
  }
}
```

### Error response

```json
{
  "success": false,
  "error": "Offer validation failed: The amount field is required.",
  "error_code": "offer_validation_failed"
}
```
