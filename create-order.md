# Create an Order

```
POST /api/v1/orders
```

> **Checkout URL parameters for static offers**
>
> For static offers, you can append parameters to the checkout URL (available on the product page) to pre-fill information for your customers.
> These parameters are client-controlled and can be easily altered.
> **They must never be trusted as-is.**
> Always validate, sanitize, and enforce all critical values on your server before processing them.

## Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `offer` | array | **Yes** | An offer UUID to purchase. |
| `email` | string | No | Customer's email address. |
| `billing_email_locked` | boolean | No | Indicates if the billing email is locked and cannot be changed by the customer. Email is required if billing email is locked. |
| `external_id` | string | No | Any custom identifier you want to provide for this order. This field is not unique and can be used for multiple orders. This value is searchable in your order history. |
| `metadata` | array | No | Any custom metadata you want to provide for this order. |
| `type` | string | No | Billing address type (`personal` or `business`). |
| `company_name` | string | No | Billing address company name. |
| `firstname` | string | No | Billing address first name. |
| `lastname` | string | No | Billing address last name. |
| `phone` | string | No | Billing address phone number. |
| `line1` | string | No | Billing address line 1. |
| `line2` | string | No | Billing address line 2. |
| `postal_code` | string | No | Billing address postal code. |
| `city` | string | No | Billing address city. |
| `state` | string | No | Billing address state. |
| `country` | string | No | Billing address country (ISO 3166-2 alpha-2 code). |
| `registration_number` | string | No | Billing address registration number. |
| `vat_number` | string | No | Billing address VAT number. |
| `external_id` | string | No | Custom identifier provided for the order item. |
| `success_url` | string | No | URL to redirect the customer after a successful payment. |
| `cancel_url` | string | No | URL to redirect the customer after a canceled payment. |

## Parameters for flexible offers

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `title` | string | No | Title of the order item. |
| `description` | string | No | Description of the order item. |
| `amount` | string | **Yes** | Amount of the order item. |
| `currency` | string | **Yes** | Currency of the order item. |
| `is_tax_incl` | boolean | **Yes** | Indicates if the amount includes tax. |
| `quantity` | integer | No | Quantity of the order item (1 by default). |
| `mode` | string | **Yes** | Mode of the order item (`one_off` or `subscription`). |
| `allowed_installment_count` | string | No | Allowed installment count for the order item. (min:2, max:12) |
| `first_amount` | string | No | First amount of the order item. |
| `interval` | string | No | Interval of the order item. Required if `mode` is `subscription`. |

## Response

| Name | Type | Always Present | Description |
|------|------|----------------|-------------|
| `success` | boolean | **Yes** | Indicates if the request was successful. |
| `order` | object | No | Order details if the request was successful. |
| `order.uuid` | string | No | UUID of the order. |
| `order.checkout_access_url` | string | No | URL to access the checkout for the order. |
| `error` | string | No | Error message if the request failed. |
| `error_code` | string | No | Error code if the request failed. |
