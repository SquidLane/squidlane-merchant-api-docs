# Embedded Checkout

Render the SquidLane card payment form **directly inside your own application** — no redirect, no popup. The customer pays without ever leaving your page.

```
<script src="https://www.squidlane.com/embed.js"></script>
```

> The embed supports **one-off**, **subscription**, and **pay-in-installments** payments. Subscriptions with an entry fee or free trial, and embedded **upsells** (one-click post-purchase offers using the saved card) are still in progress — for those, use the redirect [`checkout_access_url`](create-order.md) in the meantime.

---

## How it works

```
1. Your backend creates an order (server-to-server)        ── POST /api/v1/orders
                                                              └─► returns order.embed_url (signed, 24h)
2. Your frontend loads embed.js and calls SquidLane.mount({ embedUrl, container, ... })
3. SquidLane opens the signed embed_url, detects the customer (IP → country/tax),
   starts an Inflow payment session, and returns the keys the card form needs.
4. The PCI-compliant card form (Inflow iframe) is mounted in your container.
5. The customer pays. 3D Secure is handled automatically.
6. onSuccess / onError fire — and the order.updated webhook is sent to your backend.
```

Card data is entered inside **Inflow's PCI-compliant iframe** and never touches your page nor SquidLane. The script only ever receives the Inflow **public** key and a payment id — never any secret.

---

## Prerequisites

- An [API token](authentication.md).
- An **offer** to sell (the same offer UUID you use for [Create an Order](create-order.md)).
- The customer's **email** (required to open a payment session). Country is optional — SquidLane detects it from the customer's IP if you don't provide it.

---

## Step 1 — Create the order (backend)

Create the order server-to-server, exactly like the redirect flow. The response now includes **`embed_url`**.

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
    "firstname": "Jean",
    "lastname": "Dupont",
    "postal_code": "75001",
    "success_url": "https://yourapp.com/success",
    "cancel_url": "https://yourapp.com/cancel",
    "external_id": "order-123"
  }'
```

```json
{
  "success": true,
  "order": {
    "uuid": "ordr_XXXXXX",
    "checkout_access_url": "https://www.squidlane.com/checkout/...",
    "embed_url": "https://www.squidlane.com/api/v1/embed/ordr_XXXXXX?expires=...&signature=..."
  }
}
```

`embed_url` is an **unguessable signed URL bound to that single order**, valid for **24 hours**. Pass it to your frontend.

> **Never** expose your API token to the browser. Always create the order from your backend and hand only the resulting `embed_url` to the frontend.

---

## Step 2 — Mount the checkout (frontend)

```html
<div id="squidlane-checkout"></div>

<script src="https://www.squidlane.com/embed.js"></script>
<script>
  SquidLane.mount({
    embedUrl: "<order.embed_url from your backend>",
    container: "#squidlane-checkout",
    locale: "fr",
    onSuccess: function (result) {
      // Payment completed. UX signal only — see "Fulfillment" below.
      window.location.href = "/thank-you";
    },
    onError: function (error) {
      console.error(error.code, error.message);
    },
  });
</script>
```

That's it. The card form renders inside `#squidlane-checkout`.

---

## `SquidLane.mount(options)`

Returns a handle: `{ destroy() }`. Call `destroy()` when you remove the checkout from the DOM (e.g. in a SPA effect cleanup) to avoid stale iframes.

| Option | Type | Required | Description |
|--------|------|----------|-------------|
| `embedUrl` | string | **Yes** | The signed `embed_url` from the create-order response. |
| `container` | string \| HTMLElement | **Yes** | CSS selector or DOM element where the card form mounts. |
| `locale` | string | No | `en`, `fr`, `de`, `es`, `it`, `nl`, `pl`, `pt`. Overrides the auto-detected locale. |
| `style` | object | No | Visual customization. See [Styling](#styling). |
| `buttonText` | string | No | Custom pay-button label. |
| `placeholders` | object | No | `{ cardNumber, expiry, cvc }` custom input placeholders. |
| `showDefaultSuccessUI` | boolean | No | Show the SDK's built-in success screen (default `true`). Set `false` to render your own. |
| `installments` | boolean | No | When the order allows installments (`allowed_installment_count ≥ 2`), the embed shows a built-in plan chooser before the card form. Set `false` to always charge in full and skip it. Default `true`. See [Pay in installments](#pay-in-installments). |
| `showInstallmentChange` | boolean | No | Show a "change payment plan" link above the card so the customer can go back and re-pick. **Off by default** — read the caveat in [Pay in installments](#pay-in-installments) before enabling. |
| `installmentText` | object | No | Copy overrides for the installment chooser: `{ title, continueLabel, change }`. |
| `onSuccess` | function | No | Called on a completed payment. See below. |
| `onError` | function | No | Called on any failure (load, validation, payment). |
| `onReady` | function | No | Called once the card form is mounted and interactive. |
| `onIncomplete` | function | No | Called when the order is missing required billing info. |

### `onSuccess(result)`

```js
{
  status: "succeeded",
  confirmed: true,          // true once the SquidLane server confirmed "succeeded"
  payment_id: "pay_...",    // the Inflow payment id
  success_url: "https://yourapp.com/success" // your configured success URL, if any
}
```

> **`onSuccess` is a UX signal**, not a fulfillment signal. `confirmed: false` means the gateway reported success but the SquidLane server hadn't finished reconciling yet. **Always fulfill from the signed [`order.updated` webhook](webhooks.md)**, never from this callback alone.

### `onError(error)`

```js
{ code: "PAYMENT_FAILED", message: "Payment failed.", retryable: true }
```

| Code | Meaning |
|------|---------|
| `MISSING_EMBED_URL` | `options.embedUrl` was not provided. |
| `CONTAINER_NOT_FOUND` | `options.container` did not resolve to an element. |
| `BOOTSTRAP_FAILED` | The embed session request failed (e.g. expired/invalid signed URL). |
| `INCOMPLETE` | The order lacks required billing info — see `onIncomplete`. |
| `UNAVAILABLE` | The embed is not available for this order (`error.status` carries the raw status). |
| `PAYMENT_FAILED` | The card payment failed (declined, 3DS failure). `error.retryable` indicates if the customer can retry. |
| `SDK_LOAD_FAILED` / `SDK_ORIGIN_UNKNOWN` | The SDK could not be loaded — make sure `embed.js` is included with a normal `<script src>` tag. |
| `UNEXPECTED_ERROR` | Any other unexpected failure. |

### `onIncomplete(error)`

```js
{ code: "INCOMPLETE", message: "...", missing: ["email"] }
```

Fired when the order can't open a payment session because billing info is insufficient (typically a missing `email`). Collect the missing fields and create a new order, or set them at creation time.

---

## Pay in installments

The customer can split a **one-off** payment into several monthly installments ("pay in N×") — handled entirely inside the embed, no redirect.

**Enable it at order creation** by setting [`allowed_installment_count`](create-order.md) (the maximum number of payments, `2`–`12`) on the order. When it's `≥ 2`, `embed.js` shows a **built-in plan chooser** *before* the card form:

```
Choose how to pay
 ( )  Pay in full                          200.00 € today
 (•)  Pay in 4×                             50.00 € today
      then 3 × 50.00 €
              [ Continue to payment ]
```

The customer picks a plan, clicks **Continue**, and the Inflow card form opens for that plan. The plan is chosen **before** the payment session opens — it defines the installment schedule and is fixed once the card is being paid.

- **Amounts are computed server-side**, so what's shown is exactly what gets charged (the first payment absorbs any rounding remainder).
- The chooser is **localized** (`en`/`fr`, English fallback) and takes its accent from your `style.button` color by default — override via [`style.installments`](#styling).
- Set **`installments: false`** to skip the chooser and always charge in full.
- **Nothing changes for non-installment orders**: if `allowed_installment_count` isn't set, the card form opens directly, exactly as before.

```js
SquidLane.mount({
  embedUrl: "...",
  container: "#squidlane-checkout",
  // installments: false,            // optional — force pay-in-full, skip the chooser
  installmentText: {                 // optional — override the copy
    title: "Choisissez comment payer",
    continueLabel: "Continuer vers le paiement",
  },
  style: { installments: { accentColor: "#C9A84C" } },  // see Styling
});
```

### Letting the customer change plan (opt-in)

`showInstallmentChange: true` adds a "change payment plan" link above the card so the customer can go back and re-pick.

> **Off by default — enable with care.** Once the card is mounted, the Inflow SDK exposes no "payment started" event, so `embed.js` can't reliably hide the link the instant the customer pays. If a customer goes back **while a payment is in flight**, they could be charged on the old plan while the order switches to a new one. Only enable this if your flow guarantees the customer cannot go back after submitting payment.

---

## Loading state

Mounting isn't instant: after you call `mount()`, SquidLane opens the payment session (a few server round-trips) before the card form appears — slightly longer for subscriptions.

**You don't need to build a loader.** `embed.js` shows a **built-in loading indicator** inside your container for the whole wait, then replaces it with the card form. It's **localized** to the same `locale` as the card form (falling back to English), so the customer sees a consistent language throughout. `onReady` fires the moment the loader is replaced by the interactive form, if you want to coordinate surrounding UI.

To keep the wait short:

- Include `embed.js` **early** (e.g. on page load, not on button click) so the script download isn't on the click→form critical path.
- Create the order on your backend as soon as you have the customer's email, so `embed_url` is ready when you mount.

---

## Styling

Pass a `style` object to match your brand. All keys are optional.

```js
SquidLane.mount({
  embedUrl: "...",
  container: "#squidlane-checkout",
  style: {
    fontFamily: "Inter", // 'Inter' | 'Poppins' | 'Nunito' | 'DM Sans' | 'Work Sans' | 'Manrope' | 'Rubik' | 'Figtree' | 'Outfit' | ...
    fillParent: true,    // the card form fills its container's width
    inputContainer: { backgroundColor: "#FFFFFF", borderColor: "#E8E4DC", borderEnabled: true, borderRadius: "14px" },
    input:          { backgroundColor: "#F5F3EF", textColor: "#1A1612", placeholderColor: "#C0B8AE", borderColor: "#DDD8CD", borderEnabled: true, borderRadius: "10px" },
    button:         { backgroundColor: "#C9A84C", textColor: "#FFFFFF", borderRadius: "12px", fontSize: "15px", fontWeight: 600, hover: { backgroundColor: "#A8873B" } },
    disclaimerColor: "#9B9389",
    fieldErrorColor: "#DC2626",
    // dark: { ... }      // dark-mode overrides (same shape)

    // Pay-in-installments plan chooser (shown when allowed_installment_count ≥ 2).
    // All keys optional; defaults derive from `button` so it matches the card out of the box.
    installments: {
      accentColor: "#C9A84C",      // selected highlight, radio dot, Continue button (default: button.backgroundColor)
      accentTextColor: "#FFFFFF",  // Continue button text (default: button.textColor)
      textColor: "#1A1612",
      mutedColor: "#9B9389",
      borderColor: "#E8E4DC",
      borderRadius: "12px",
      fontFamily: "system-ui",
    },
  },
});
```

To **center** the form, give your container a max width and center it (the iframe fills the container when `fillParent: true`):

```html
<div style="display:flex; justify-content:center;">
  <div id="squidlane-checkout" style="width:100%; max-width:24rem;"></div>
</div>
```

---

## Fulfillment & source of truth

The browser callbacks are for UX. **Fulfillment must rely on the [webhook](webhooks.md).**

- SquidLane sends a signed `order.updated` webhook to your configured URL whenever the order state changes (including `intent_status: "succeeded"`).
- The webhook is **HMAC-SHA256 signed** and cannot be forged — unlike a browser callback.
- Webhooks are **at-least-once**: key on `order.uuid` + `intent_status` and make your handler idempotent.

You can also poll [`GET /api/v1/order/{uuid}`](retrieve-order.md) from your backend to read the authoritative status.

---

## Security model

- **Stateless & cross-origin.** The embed endpoints are authorized **only** by the signed `embed_url` — never by cookies/session. They work from any merchant domain.
- **No secrets in the browser.** Only the Inflow **public** key and a payment id are returned. Your API token and Inflow's private key are never exposed.
- **PCI.** Card data is entered in Inflow's PCI-compliant iframe; it never touches your page nor SquidLane.
- **Bound to one order, time-limited.** The `embed_url` is an HMAC-signed URL for a single order, valid 24h. Don't log it.
- **3D Secure** is handled automatically by the SDK when the bank requires it.

---

## Full example

**Backend (Node/Express):**

```js
app.post("/api/start-checkout", async (req, res) => {
  const r = await fetch("https://www.squidlane.com/api/v1/orders", {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${process.env.SQUIDLANE_API_TOKEN}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      offer: process.env.SQUIDLANE_OFFER_UUID,
      amount: 99.0,
      currency: "EUR",
      is_tax_incl: true,
      mode: "one_off",
      email: req.body.email,
      firstname: req.body.firstname,
      lastname: req.body.lastname,
      postal_code: req.body.postal_code,
      success_url: "https://yourapp.com/success",
    }),
  });
  const data = await r.json();
  res.json({ embed_url: data.order.embed_url });
});
```

**Frontend:**

```html
<div id="squidlane-checkout"></div>
<script src="https://www.squidlane.com/embed.js"></script>
<script>
  async function startCheckout(customer) {
    const res = await fetch("/api/start-checkout", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(customer),
    });
    const { embed_url } = await res.json();

    const checkout = SquidLane.mount({
      embedUrl: embed_url,
      container: "#squidlane-checkout",
      locale: "fr",
      onSuccess: () => (window.location.href = "/thank-you"),
      onError: (e) => alert(e.message),
    });

    // checkout.destroy() when you remove the form from the DOM
  }
</script>
```

---

## Notes

- The embed supports **one-off**, **subscription**, and **pay-in-installments** orders in **EUR / USD / GBP**. For installments, set [`allowed_installment_count`](create-order.md) on the order — see [Pay in installments](#pay-in-installments).
- Subscriptions with an **entry fee** or a **free trial** are not supported in the embed yet (pending Inflow SDK support) — use the redirect [`checkout_access_url`](create-order.md) for those.
- Embedded **upsells are coming soon** — use [`checkout_access_url`](create-order.md) for those today.
- The Inflow SDK is served from SquidLane's own origin; include `embed.js` with a standard `<script src="https://www.squidlane.com/embed.js">` tag (not bundled/inlined) so the script can locate it.
