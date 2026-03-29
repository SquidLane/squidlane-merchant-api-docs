# Retrieve Order

```
GET /api/v1/order/<order-uuid>
```

## Parameters

This endpoint does not require any parameters. The order to retrieve is determined by the order UUID in the URL.

## Response

| Name | Type | Description |
|------|------|-------------|
| `success` | boolean | Indicates if the request was successful. |
| `order` | object | |
| `order.uuid` | string | |
| `order.intent_status` | string | Status of the order intent. Can be `started`, `processing`, `succeeded`, `failed` or `canceled`. |
| `order.subscription_status` | string | Status of the subscription. Can be `active`, `canceling`, `past_due` or `expired`. |
| `order.installment_status` | string | Status of the installment. Can be `active`, `defaulted` or `completed`. |
| `order.current_period_ends_at` | date | |
| `order.locale` | string | Locale of the order (ISO 639-1 language code). |
| `order.country` | string | Country of the order (ISO 3166-2 alpha-2 code). |
| `order.currency` | string | Currency of the order (ISO 4217 currency code). |
| `order.is_tax_incl` | boolean | |
| `order.tax_rate` | float | |
| `order.amount_excl_tax` | float | |
| `order.amount_tax` | float | |
| `order.amount_incl_tax` | float | |
| `order.first_amount_excl_tax` | float | |
| `order.first_amount_tax` | float | |
| `order.first_amount_incl_tax` | float | |
| `order.interval` | string | Interval of the order. Can be `month`, `trimester`, `semester` or `year`. |
| `order.mode` | string | Mode of the order. Can be `one_off` or `subscription`. |
| `order.external_id` | string | Custom identifier provided. This field is not unique and can be used for multiple orders. Never trust this data as it could be injected by the client in a checkout link. |
| `order.success_url` | string | URL to redirect the customer after a successful payment. |
| `order.cancel_url` | string | URL to redirect the customer after a canceled payment. |
| `order.metadata` | array | Custom metadata provided. Never trust this data as it could be injected by the client in a checkout link. |
| `order.billing_email_locked` | boolean | Indicates if the billing email is locked and cannot be changed by the customer. |
| `order.allowed_installment_count` | int | Allowed installment count for the order. (min:2, max:12) |
| `order.customer_order_url` | string | URL to access the customer's order management page. |
| **order.account** | | |
| `order.account.uuid` | string | |
| `order.account.name` | string | |
| **order.billingAddress** | | |
| `order.billingAddress.type` | string | Billing address type (`personal` or `business`). |
| `order.billingAddress.firstname` | string | |
| `order.billingAddress.lastname` | string | |
| `order.billingAddress.email` | string | |
| `order.billingAddress.phone` | string | |
| `order.billingAddress.company_name` | string | |
| `order.billingAddress.line1` | string | |
| `order.billingAddress.line2` | string | |
| `order.billingAddress.postal_code` | string | |
| `order.billingAddress.city` | string | |
| `order.billingAddress.state` | string | |
| `order.billingAddress.country` | string | |
| `order.billingAddress.registration_number` | string | |
| `order.billingAddress.vat_number` | string | |
| **order.lane** | | |
| `order.lane.id` | int | |
| `order.lane.name` | string | |
| **order.items[]** | | |
| `order.items[].id` | int | |
| `order.items[].product_uuid` | string | |
| `order.items[].offer_uuid` | string | |
| `order.items[].external_id` | string | Custom identifier provided for the order item (product variant for Shopify orders). |
| `order.items[].affiliate_bonzai_id` | string | Affiliate Bonzai ID |
| `order.items[].title` | string | |
| `order.items[].description` | string | |
| `order.items[].is_fulfillable` | boolean | |
| `order.items[].is_fulfilled` | boolean | |
| `order.items[].amount` | float | |
| `order.items[].currency` | string | |
| `order.items[].is_tax_incl` | boolean | |
| `order.items[].first_amount` | float | |
| `order.items[].interval` | string | Interval of the order. Can be `month`, `trimester`, `semester` or `year`. |
| `order.items[].mode` | string | Mode of the order. Can be `one_off` or `subscription`. |
| `order.items[].allowed_installment_count` | int | Allowed installment count for the order. (min:2, max:12) |
| `order.items[].affiliate_commission` | int | Affiliate commission percentage for the order item (min:0, max:100). |
| **order.payments[]** | | |
| `order.payments[].uuid` | string | |
| `order.payments[].status` | string | |
| `order.payments[].paid_at` | datetime | |
| `order.payments[].invoiced_at` | datetime | |
| `order.payments[].amount_excl_tax` | float | |
| `order.payments[].amount_tax` | float | |
| `order.payments[].amount_incl_tax` | float | |
| `order.payments[].currency` | string | |
| **Timestamps** | | |
| `order.created_at` | datetime | |
| `order.updated_at` | datetime | |
