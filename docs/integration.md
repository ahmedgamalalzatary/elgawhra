# Middleware Integration Plan

## 📋 OVERVIEW

A Next.js middleware deployed on Vercel (free tier) that sits between Shopify and the client's ERP system + WhatsApp messaging. It receives Shopify webhook events, transforms the data, forwards it to the ERP, and triggers WhatsApp notifications.

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
┌──────────┐     Webhook      ┌──────────────────┐     API Call     ┌──────────┐
│          │ ───────────────▶  │                  │ ──────────────▶  │          │
│  Shopify │                  │  Next.js (Vercel) │                  │   ERP    │
│          │ ◀───────────────  │                  │ ◀──────────────  │          │
└──────────┘     200 OK       │                  │     Response     └──────────┘
                              │                  │
                              │                  │     API Call     ┌──────────┐
                              │                  │ ──────────────▶  │ WhatsApp │
                              │                  │                  │   API    │
                              └──────────────────┘                  └──────────┘
```

### Flow:
1. **Shopify** fires a webhook event (e.g., order created)
2. **Next.js API route** receives it, verifies HMAC signature
3. **Transforms** Shopify data → ERP format
4. **Forwards** to ERP endpoint
5. **Sends** WhatsApp notification (order confirmation, status update, etc.)
6. **Returns** 200 to Shopify

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
│           └── health/
│               └── route.ts            # Health check endpoint
├── lib/
│   ├── erp.ts                          # ERP API client
│   ├── whatsapp.ts                     # WhatsApp API client
│   ├── verify.ts                       # Shopify HMAC webhook verification
│   ├── transform.ts                    # Data mapping (Shopify → ERP format)
│   └── config.ts                       # Environment variables
├── .env.local                          # Secrets (not committed)
├── package.json
└── tsconfig.json
```

---

## 🔗 SHOPIFY WEBHOOK EVENTS

Events to subscribe to (configured in Shopify admin or via API):

| Event | Trigger | Action |
|-------|---------|--------|
| `orders/create` | New order placed | Forward to ERP + send WhatsApp confirmation |
| `orders/updated` | Order modified | Forward update to ERP |
| `orders/cancelled` | Order cancelled | Forward to ERP + send WhatsApp notification |

> **Note:** Additional events (inventory, fulfillment, refunds, etc.) can be added later as needed. Start with orders — that's the core flow.

---

## ⏱️ TIMEOUT STRATEGY

Vercel free tier has a **10-second function timeout**. This is handled as follows:

### Why 10s is enough:
- The ERP endpoint should respond well within 10 seconds for any normal operation
- If the ERP takes >10 seconds, something is **seriously wrong** — this is an incident, not a retry scenario
- The 10s limit acts as a natural health check

### Failure Handling:
If the ERP does not respond (timeout or error):
1. **WhatsApp notification is sent** with the order data — so the client always knows about the order
2. **ERP notification is sent** (if possible) — so the ERP dev knows something failed
3. **At least one of the two channels will always work** — if not both

This dual-notification approach means no order is ever lost, even if the ERP is completely down.

### Why we don't need a queue/database:
- The WhatsApp message serves as the fallback "store" for failed orders
- The client can manually enter the order into the ERP from the WhatsApp message if needed
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

### Environment Variables (`.env.local`):
```
SHOPIFY_WEBHOOK_SECRET=xxx          # From Shopify webhook settings
ERP_API_URL=xxx                     # ERP endpoint (from ERP dev)
ERP_API_KEY=xxx                     # ERP authentication (from ERP dev)
WHATSAPP_API_URL=xxx                # WhatsApp API endpoint
WHATSAPP_API_KEY=xxx                # WhatsApp API authentication
WHATSAPP_RECIPIENT=xxx              # Client's WhatsApp number
```

---

## 📱 WHATSAPP INTEGRATION

> **Decision pending:** Which WhatsApp API provider to use (Twilio, WhatsApp Business API, Wati, etc.). To be decided based on client's existing setup and budget.

### Message Types:
| Trigger | Message |
|---------|---------|
| New order | "🛒 New order #{id} — {customer_name} — {total} EGP — {items_count} items" |
| Order cancelled | "❌ Order #{id} cancelled — {customer_name}" |
| ERP failure | "⚠️ ERP unreachable — Order #{id} data: {order_details}" |

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

### Initial Setup:
1. Create a dedicated Vercel account for this project
2. Connect the GitHub repository
3. Set environment variables in Vercel dashboard
4. Deploy

### Shopify Webhook Registration:
1. Shopify Admin → Settings → Notifications → Webhooks
2. Add webhook for each event (orders/create, orders/updated, orders/cancelled)
3. Point to: `https://your-middleware.vercel.app/api/webhooks/orders`
4. Format: JSON

### Testing:
1. Use Shopify's "Send test notification" to verify webhook delivery
2. Check Vercel function logs for execution
3. Verify ERP receives the data
4. Verify WhatsApp messages are sent

---

## ⏳ DEPENDENCIES (WAITING ON)

| Item | From | Status |
|------|------|--------|
| ERP API endpoint URL | ERP Dev | ⬜ Pending (meeting done, ETA: few days) |
| ERP API authentication method | ERP Dev | ⬜ Pending |
| ERP data schema (expected format) | ERP Dev | ⬜ Pending |
| WhatsApp API provider decision | Client | ⬜ Pending |
| Client's WhatsApp number | Client | ⬜ Pending |
| Products & content (persona) | Client | ⬜ Pending |

---

## 📝 NOTES

- The middleware is **API-only** for now — no frontend/dashboard pages
- Next.js was chosen over Express/Hono specifically for: free Vercel hosting, potential future dashboard, and minimal overhead for this use case
- Start with `orders/create` webhook only — add more events as needed
- The ERP dev is working on making the ERP database accessible via an API endpoint
- Dual notification (WhatsApp + ERP) ensures no order is ever lost
- Keep the middleware stateless — no database needed at this scale
- If a dashboard is needed later, add pages to the same Next.js project — no new deployment needed
