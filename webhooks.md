# Webhooks

Receive notifications for order updates via webhooks.

## Configuration

Set your webhook URL in the **Account Settings > Webhooks** section of your SquidLane dashboard.

## How it works

Each webhook is sent as a JSON payload in a `POST` request to your configured webhook URL.

The JSON payload includes two keys:

| Key | Type | Description |
|-----|------|-------------|
| `event` | string | The event type. Only one event is available: `order.updated`. |
| `order` | object | The full order object. See the [Retrieve Order](retrieve-order.md) response for the complete schema. |

### Example payload

```json
{
  "event": "order.updated",
  "order": {
    "uuid": "...",
    "intent_status": "succeeded",
    "subscription_status": null,
    "installment_status": null,
    "current_period_ends_at": null,
    "locale": "fr",
    "currency": "EUR",
    "is_tax_incl": true,
    "tax_rate": 20,
    "amount_excl_tax": 100.00,
    "amount_tax": 20.00,
    "amount_incl_tax": 120.00,
    "..."
  }
}
```

## Signature verification

If you configure a **webhook secret** in your account settings, each webhook request will include a `X-Signature-256` header containing an HMAC-SHA256 signature of the JSON payload.

To verify the signature on your server:

1. Read the raw JSON body of the incoming request.
2. Compute the HMAC-SHA256 hash of that body using your webhook secret as the key.
3. Compare the computed hash with the value of the `X-Signature-256` header.

### Example (PHP)

```php
$payload = file_get_contents('php://input');
$signature = hash_hmac('sha256', $payload, $webhookSecret);

if (!hash_equals($signature, $_SERVER['HTTP_X_SIGNATURE_256'] ?? '')) {
    http_response_code(403);
    exit('Invalid signature');
}
```

### Example (Node.js)

```js
const crypto = require('crypto');

const signature = crypto
  .createHmac('sha256', webhookSecret)
  .update(rawBody)
  .digest('hex');

if (signature !== req.headers['x-signature-256']) {
  return res.status(403).send('Invalid signature');
}
```

> **Important:** Always use a constant-time comparison function (like `hash_equals` in PHP or `crypto.timingSafeEqual` in Node.js) to prevent timing attacks.

## Events

| Event | Description |
|-------|-------------|
| `order.updated` | Triggered when an order is updated (status change, payment received, etc.). |

## Delivery status

Each webhook delivery is logged and can be viewed in the **Delivery history** section of the Webhooks settings page.

| Status | Description |
|--------|-------------|
| `succeeded` | The webhook was delivered successfully (HTTP 2xx response). |
| `failed` | The webhook delivery failed (non-2xx response or network error). |

## Testing

You can trigger a test webhook from any order detail page in your dashboard.
