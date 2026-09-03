# SquidLane Merchant API

Welcome to the SquidLane Merchant API documentation.

**Base URL:**

```
https://www.squidlane.com/api/v1
```

## Token scopes

Personal access tokens are created from **Settings → API Tokens**, with one of two scopes:

| Scope | What it can do |
|-------|----------------|
| **Read only** | Every `GET` endpoint: accounts, balances, payments, orders. |
| **Full access** | The same, plus every endpoint that changes data: [Create an Order](create-order.md), [Update an Order Item](update-order-item.md). |

Pick **Read only** for reporting and monitoring integrations. A read-only token calling a write endpoint gets a `403`.

Tokens created before scopes existed keep full access.

## Table of contents

- [Authentication](authentication.md)
- [Retrieve Order](retrieve-order.md) — `GET /api/v1/order/{uuid}`
- [Create an Order](create-order.md) — `POST /api/v1/orders`
- [Update an Order Item](update-order-item.md) — `PATCH /api/v1/order-item/{id}` — change a subscription price
- [List Accounts](accounts.md) — `GET /api/v1/accounts`
- [Account Balances](balances.md) — `GET /api/v1/account/{uuid}/balances`
- [List Payments](payments.md) — `GET /api/v1/account/{uuid}/payments`
- [Embedded Checkout](embed.md) — render the card form inside your own app (no redirect)
- [Webhooks](webhooks.md)
