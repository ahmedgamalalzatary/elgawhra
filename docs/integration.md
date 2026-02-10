# Middleware Integration Plan

## 📋 OVERVIEW

A Next.js middleware deployed on Vercel (free tier) that sits between Shopify and the client's ERP system + WhatsApp messaging. It receives Shopify webhook events, **verifies orders via WhatsApp confirmation links**, transforms the data, forwards confirmed orders to the ERP, and triggers WhatsApp notifications.

**Core flow:** Shopify → Next.js → WhatsApp confirmation → Customer clicks link → Next.js → ERP

| Setting | Value |
|---------|-------|
| **Framework** | Next.js (API routes only) |
| **Hosting** | Vercel (free tier — dedicated account for this project) |
| **Request Limit** | 100k requests/month (more than enough) |
| **Function Timeout** | 10 seconds (sufficient — see Timeout Strategy below) |
| **Dashboard** | Not planned now, but Next.js allows adding one later if needed |

---

## 🏗️ ARCHITECTURE

```
┌──────────┐    Webhook     ┌──────────────────┐   Tag "unconfirmed"  ┌──────────────┐
│          │ ─────────────▶ │                  │ ───────────────────▶ │   Shopify    │
│  Shopify │                │                  │                      │  Admin API   │
│          │ ◀───────────── │                  │                      └──────────────┘
└──────────┘    200 OK      │                  │
                            │  Next.js (Vercel)│   Confirmation msg   ┌──────────────┐
                            │                  │ ───────────────────▶ │  WhatsApp    │
                            │                  │                      │  Cloud API   │
                            │                  │                      └──────┬───────┘
                            │                  │                             │
                            │   /verify?       │    Customer clicks link     │
                            │   order_id&token │ ◀──────────────────────────┘
                            │                  │
                            │                  │── Remove "unconfirmed" ──▶ Shopify
                            │                  │── Forward order ─────────▶ ERP
                            │                  │
                            │  Cron (24h)      │
                            │  Stale cleanup   │── Cancel unconfirmed ───▶ Shopify
                            └──────────────────┘
```

### Order Confirmation Flow:
1. **Shopify** fires `orders/create` webhook
2. **Next.js** receives it, verifies HMAC signature
3. **Extracts customer phone number** from webhook order data (`order.phone` or `order.shipping_address.phone`)
4. **Tags the order** as `unconfirmed` in Shopify via Admin API
5. **Generates a secure confirmation link** with hashed token: `/verify?order_id={id}&token={hash}`
6. **Sends WhatsApp message** to the customer's phone (via WhatsApp Cloud API) with the confirmation link
7. **Returns** 200 to Shopify

### When Customer Clicks the Confirmation Link:
8. **Next.js `/verify` route** receives the request
9. **Validates the hashed token** (token must match `HMAC(order_id, secret)` — prevents guessing)
10. **Removes the `unconfirmed` tag** from the order in Shopify via Admin API
11. **Transforms** Shopify order data → ERP format
12. **Forwards** the confirmed order to the ERP
13. **Sends** WhatsApp notification to the client (store owner) about the new confirmed order

### If Customer Does NOT Confirm (Fake/Spam Order):
14. **Vercel Cron Job** runs every 24 hours
15. **Fetches all orders** tagged `unconfirmed` from Shopify via Admin API
16. **Cancels/removes** orders that have been unconfirmed for >24 hours
17. Nothing is sent to the ERP — fake orders never reach the system

---

## 📁 PROJECT STRUCTURE

```
middleware/
├── src/
│   └── app/
│       └── api/
│           ├── webhooks/
│           │   ├── orders/
│           │   │   └── route.ts        # Order events (created, updated, cancelled)
│           │   └── products/
│           │       └── route.ts        # Product events (if needed)
│           ├── verify/
│           │   └── route.ts            # Order confirmation endpoint (GET /verify?order_id&token)
│           ├── cron/
│           │   └── cleanup/
│           │       └── route.ts        # Daily cron: cancel stale unconfirmed orders
│           └── health/
│               └── route.ts            # Health check endpoint
├── lib/
│   ├── erp.ts                          # ERP API client
│   ├── whatsapp.ts                     # WhatsApp Cloud API client
│   ├── shopify.ts                      # Shopify Admin API client (tagging, order management)
│   ├── hmac.ts                         # Shopify webhook HMAC verification
│   ├── token.ts                        # Confirmation link token generation & validation
│   ├── transform.ts                    # Data mapping (Shopify → ERP format)
│   └── config.ts                       # Environment variables
├── vercel.json                         # Cron job configuration
├── .env.local                          # Secrets (not committed)
├── package.json
└── tsconfig.json
```

---

## 🔗 SHOPIFY WEBHOOK EVENTS

Events to subscribe to (configured in Shopify admin or via API):

| Event | Trigger | Action |
|-------|---------|--------|
| `orders/create` | New order placed | Tag as `unconfirmed` + send WhatsApp confirmation link to customer |
| `orders/updated` | Order modified | Forward update to ERP (only if order is confirmed) |
| `orders/cancelled` | Order cancelled | Forward to ERP + send WhatsApp notification to client |

> **Note:** The `orders/create` event does NOT immediately forward to ERP. Orders are held until the customer confirms via the WhatsApp link. Additional events (inventory, fulfillment, refunds, etc.) can be added later as needed.

---

## ⏱️ TIMEOUT STRATEGY

Vercel free tier has a **10-second function timeout**. This is handled as follows:

### Why 10s is enough:
- The ERP endpoint should respond well within 10 seconds for any normal operation
- If the ERP takes >10 seconds, something is **seriously wrong** — this is an incident, not a retry scenario
- The 10s limit acts as a natural health check

### Failure Handling:

**On order creation (webhook):**
- If Shopify tagging fails → still send WhatsApp confirmation (order can be confirmed manually)
- If WhatsApp API fails → log error; order stays `unconfirmed` and will be cleaned up by cron

**On order confirmation (customer clicks link):**
- If ERP does not respond (timeout or error):
  1. **WhatsApp notification is sent** to the client with the order data — so they always know about the order
  2. The `unconfirmed` tag is still removed — the order is confirmed regardless of ERP status
  3. **At least one channel will always work** — if not both

This dual-notification approach means no confirmed order is ever lost, even if the ERP is completely down.

### Why we don't need a queue/database:
- The WhatsApp message serves as the fallback "store" for failed ERP forwards
- The client can manually enter the order into the ERP from the WhatsApp message if needed
- Shopify itself retains all order data — the middleware is stateless
- For a mid-large scale business, this is a practical and cost-free safety net
- If this ever becomes insufficient, a simple database/queue can be added later

---

## 🔐 SECURITY

### Shopify HMAC Verification:
Every incoming webhook must be verified using Shopify's HMAC signature to prevent spoofing:

```typescript
import crypto from 'crypto';

function verifyShopifyWebhook(body: string, hmacHeader: string): boolean {
  const secret = process.env.SHOPIFY_WEBHOOK_SECRET!;
  const hash = crypto.createHmac('sha256', secret).update(body).digest('base64');
  return crypto.timingSafeEqual(Buffer.from(hash), Buffer.from(hmacHeader));
}
```

### Confirmation Link Token (Hashing):
The `/verify` link includes a **hashed token** to prevent anyone from guessing order IDs and confirming random orders:

```typescript
import crypto from 'crypto';

const CONFIRM_SECRET = process.env.CONFIRM_TOKEN_SECRET!;

// Generate token when creating the confirmation link
function generateToken(orderId: string): string {
  return crypto.createHmac('sha256', CONFIRM_SECRET).update(orderId).digest('hex');
}

// Validate token when customer clicks the link
function validateToken(orderId: string, token: string): boolean {
  const expected = generateToken(orderId);
  return crypto.timingSafeEqual(Buffer.from(expected), Buffer.from(token));
}

// Confirmation link format:
// https://your-middleware.vercel.app/api/verify?order_id=123456&token=a8f3c9...
```

> **Why hashing?** Without the token, anyone could visit `/verify?order_id=123` and confirm orders they didn't place. The HMAC hash ensures only someone who received the WhatsApp message (i.e., the actual phone owner) can confirm the order.

### Environment Variables (`.env.local`):
```
SHOPIFY_WEBHOOK_SECRET=xxx          # From Shopify webhook settings
SHOPIFY_ADMIN_API_TOKEN=xxx         # Shopify Admin API access token (for tagging orders)
SHOPIFY_STORE_DOMAIN=xxx            # e.g., store.myshopify.com
CONFIRM_TOKEN_SECRET=xxx            # Secret for hashing confirmation link tokens
ERP_API_URL=xxx                     # ERP endpoint (from ERP dev)
ERP_API_KEY=xxx                     # ERP authentication (from ERP dev)
WHATSAPP_PHONE_NUMBER_ID=xxx        # Meta WhatsApp phone number ID
WHATSAPP_ACCESS_TOKEN=xxx           # Meta WhatsApp Cloud API access token
WHATSAPP_CLIENT_NUMBER=xxx          # Client's (store owner) WhatsApp number for notifications
```

---

## 📱 WHATSAPP INTEGRATION

### Provider Decision:

Two options were evaluated:

| Option | Cost | Effort | Notes |
|--------|------|--------|-------|
| **WhatsApp Cloud API (Meta direct)** ✅ | ~$0.0541/msg (~2.5 EGP) — Meta conversation fee only | Medium (API integration) | No middleman, full control, fits existing Next.js architecture |
| **CartSaver (Shopify Add-on)** | $10/month (170 confirmations) or $0.04/msg | None (plug & play) | Vendor lock-in, USD pricing, less control over Arabic messages |

**Decision: WhatsApp Cloud API (Meta direct)** — no BSP middleware needed. The Next.js middleware calls Meta's Graph API directly.

### Setup:
1. Create a **Meta Business Account** at [developers.facebook.com](https://developers.facebook.com)
2. Add the **WhatsApp** product to the Meta app
3. Register a business phone number
4. Create **message templates** (must be approved by Meta before use)

### How the Middleware Sends Messages:

The customer's phone number is **extracted from the Shopify webhook** order data (`order.phone` or `order.shipping_address.phone`). No hardcoded recipient — each message targets the buyer who placed the order.

```typescript
// POST https://graph.facebook.com/v18.0/{PHONE_NUMBER_ID}/messages
const response = await fetch(
  `https://graph.facebook.com/v18.0/${process.env.WHATSAPP_PHONE_NUMBER_ID}/messages`,
  {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.WHATSAPP_ACCESS_TOKEN}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      messaging_product: 'whatsapp',
      to: customerPhone,  // From Shopify webhook order data
      type: 'template',
      template: {
        name: 'order_confirmation_ar',
        language: { code: 'ar' },
        components: [{
          type: 'body',
          parameters: [
            { type: 'text', text: customerName },
            { type: 'text', text: orderId },
            { type: 'text', text: confirmationLink },
          ],
        }],
      },
    }),
  }
);
```

### Message Templates (to be submitted to Meta for approval):

| Template | Recipient | Purpose | Example |
|----------|-----------|---------|---------|
| `order_confirmation_ar` | Customer (buyer) | Confirmation link | "مرحباً {{1}}! 🛒 طلبك رقم #{{2}} محتاج تأكيد. اضغط هنا: {{3}}" |
| `order_confirmed_notify` | Client (store owner) | New confirmed order | "🛒 طلب جديد مؤكد #{id} — {customer_name} — {total} EGP" |
| `order_cancelled_notify` | Client (store owner) | Order cancelled | "❌ طلب #{id} اتلغى — {customer_name}" |
| `erp_failure_notify` | Client (store owner) | ERP down fallback | "⚠️ ERP مش شغال — طلب #{id}: {order_details}" |

> **Note:** The confirmation message goes to the **customer**. All other notifications go to the **client (store owner)**.

---

## 🔄 DATA TRANSFORMATION

Shopify order data needs to be mapped to the ERP's expected format. The `transform.ts` file handles this.

**Example mapping (to be finalized when ERP dev provides schema):**

```typescript
function transformOrder(shopifyOrder: ShopifyOrder): ERPOrder {
  return {
    // Map Shopify fields → ERP fields
    order_id: shopifyOrder.id,
    customer_name: shopifyOrder.customer?.first_name + ' ' + shopifyOrder.customer?.last_name,
    customer_phone: shopifyOrder.customer?.phone,
    total: shopifyOrder.total_price,
    currency: shopifyOrder.currency,            // Always EGP
    items: shopifyOrder.line_items.map(item => ({
      name: item.title,
      quantity: item.quantity,
      price: item.price,
      sku: item.sku,
    })),
    shipping_address: shopifyOrder.shipping_address,
    created_at: shopifyOrder.created_at,
  };
}
```

---

## 🚀 DEPLOYMENT

### Shopify Webhook Registration:
1. Shopify Admin → Settings → Notifications → Webhooks
2. Add webhook for each event (orders/create, orders/updated, orders/cancelled)
3. Point to: `https://your-middleware.vercel.app/api/webhooks/orders`
4. Format: JSON

### Testing:
1. Use Shopify's "Send test notification" to verify webhook delivery
2. Check Vercel function logs for execution
3. Verify order gets tagged `unconfirmed` in Shopify
4. Verify WhatsApp confirmation message is sent to customer's phone
5. Click the confirmation link → verify tag is removed and order is forwarded to ERP
6. Test with an invalid/tampered token → verify it is rejected
7. Wait for cron job (or trigger manually) → verify stale unconfirmed orders are cancelled

---

## ⏰ CRON JOB (STALE ORDER CLEANUP)

A Vercel Cron Job runs every 24 hours to clean up unconfirmed orders.

### `vercel.json` Configuration:
```json
{
  "crons": [
    {
      "path": "/api/cron/cleanup",
      "schedule": "0 2 * * *"
    }
  ]
}
```
> Runs daily at 2:00 AM (UTC). Vercel free tier supports **one cron job per day** — which is exactly what we need.

### Cleanup Logic:
1. Fetch all orders tagged `unconfirmed` from Shopify Admin API
2. For each order: check `created_at` timestamp
3. If order is older than 1 minute and still `unconfirmed` → cancel it in Shopify

### Security:
The cron endpoint is protected by Vercel's `CRON_SECRET` header — only Vercel's scheduler can trigger it.

---

## ⏳ DEPENDENCIES (WAITING ON)

| Item | From | Status |
|------|------|--------|
| ERP API endpoint URL | ERP Dev | ⬜ Pending (meeting done, ETA: few days) |
| ERP API authentication method | ERP Dev | ⬜ Pending |
| ERP data schema (expected format) | ERP Dev | ⬜ Pending |
| WhatsApp API provider decision | add-ons/whatsapp cloud api | ⬜ Pending  |
| Meta Business Account setup | Client | ⬜ Pending |
| WhatsApp message template approval | Meta | ⬜ Pending (submit after Meta account setup) |
| Client's WhatsApp number (store owner) | Client | ⬜ Pending |
| Products & content (persona) | Client | ⬜ Pending |

---

## 📝 NOTES

- The middleware is **API-only** for now — no frontend/dashboard pages
- Next.js was chosen over Express/Hono specifically for: free Vercel hosting, and minimal overhead for this use case
- Start with `orders/create` webhook only — add more events as needed
- The ERP dev is working on making the ERP database accessible via an API endpoint
- Dual notification (WhatsApp + ERP) ensures no order is ever lost
- Keep the middleware stateless — no database needed at this scale
- If a dashboard is needed later, add pages to the same Next.js project — no new deployment needed
- **Order confirmation is mandatory** — no order reaches the ERP without customer verification via WhatsApp link
- **Shopify has no built-in phone/OTP verification** — this middleware solves that gap at near-zero cost
- The confirmation token uses **HMAC-SHA256 hashing** — not plain order IDs — to prevent URL guessing
- WhatsApp Cloud API is called **directly via Meta's Graph API** — no BSP (Business Solution Provider) middleware needed
- Customer phone number is extracted from Shopify webhook data — no manual input or hardcoding needed
