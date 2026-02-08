# Product Requirements Document (PRD)

## 📋 PROJECT OVERVIEW

| Field | Value |
|-------|-------|
| **Project Name** | El Gawhra — E-Commerce Store |
| **Industry** | Food / Snacks / Roastery |
| **Platform** | Shopify (Dawn theme v15.4.1) |
| **Market** | Egypt only |
| **Currency** | EGP (Egyptian Pound) |
| **Languages** | Arabic (primary, source) + English (secondary, translated) |
| **Direction** | RTL (Arabic) / LTR (English) |
| **Reference Store** | [Snacks Roastery](https://snacks-roastery.com/en) — clone layout/structure, replace branding and products |
| **Target Audience** | Egyptian consumers, primarily mobile users |

---

## 🎯 GOALS

1. Launch a bilingual (AR/EN) e-commerce store for food/snacks products
2. Clone the layout and structure of the reference store (Snacks Roastery)
3. Integrate with client's existing ERP system via webhook middleware
4. Provide WhatsApp notifications for order events
5. Full RTL support for Arabic-first experience

---

## 📄 PAGES

### Unique Pages (3)

| Page | Route | Description |
|------|-------|-------------|
| **Home** | `/` (AR) · `/en` (EN) | Landing page — hero banner, featured collections, featured products, promotional sections. Layout cloned from reference store. |
| **About Us** | `/pages/about-us` | Brand story, mission, team info, images. Content pending from client. |
| **Contact Us** | `/pages/contact-us` | Contact form, phone number, email, address, social links, working hours, embedded map (if applicable). Content pending from client. |

### Collection Pages (7)

| Page | Route | Description |
|------|-------|-------------|
| **All Products** | `/collections/all` | Full product catalog with sorting, filtering, and search |
| **Category 1–6** | `/collections/[category-handle]` | Category-specific product listings. Category names/count pending from client. |

> **Note:** Category names, count, and hierarchy are pending from the client. The 6 category pages are an estimate based on the reference store — actual number may vary.

### System Pages (Built into Dawn/Shopify)

| Page | Route | Description |
|------|-------|-------------|
| **Product Detail** | `/products/[handle]` | Individual product page — images, title, description, price, variants, add to cart |
| **Cart** | `/cart` | Cart page with item list, quantities, totals |
| **Search Results** | `/search` | Search results page with filtering |
| **404** | `/404` | Custom not-found page |

### Customer Account Pages (TBD)

| Page | Route | Description |
|------|-------|-------------|
| **Login** | `/account/login` | Customer login |
| **Register** | `/account/register` | Customer registration |
| **Account** | `/account` | Order history, profile |
| **Addresses** | `/account/addresses` | Saved addresses |
| **Order Detail** | `/account/orders/[id]` | Individual order details |

> **Decision pending:** Whether customer accounts are needed or guest checkout only. These pages are built into Dawn and can be enabled/disabled via Shopify admin.

---

## 🛍️ PRODUCT STRUCTURE

| Field | Details |
|-------|---------|
| **Type** | Food / Snacks / Roastery products |
| **Variants** | Weight/size options expected (e.g., 250g, 500g, 1kg) — pending from client |
| **Images** | Product photos — pending from client |
| **Pricing** | EGP only — prices pending from client |
| **Inventory** | Managed via Shopify + synced to ERP |
| **Categories** | Pending from client (estimated 6 categories) |

> **All product data (titles, descriptions, images, prices, categories, variants) is pending from the client.**

---

## ✨ FEATURES

### Core E-Commerce

| Feature | Status | Details |
|---------|--------|---------|
| Product catalog | ✅ Built into Dawn | Grid/list view, product cards with images, prices, quick add |
| Product detail page | ✅ Built into Dawn | Image gallery, description, variants, add to cart, related products |
| Shopping cart | ✅ Built into Dawn | Cart page + cart drawer (slide-out) |
| Checkout | ✅ Shopify native | Handled by Shopify checkout |
| Customer accounts | ⬜ TBD | Login, register, order history — decision pending |

### Search, Sorting & Filtering

| Feature | Status | Details |
|---------|--------|---------|
| **Search** | ✅ Built into Dawn | Predictive search with live results in header |
| **Sorting** | ✅ Built into Dawn | Sort by: featured, price (low→high, high→low), best selling, A-Z, Z-A, date (new→old, old→new) |
| **Filtering** | ✅ Built into Dawn | Filter by: availability, price range, product type, vendor, tags — filters depend on product data |

> Sorting and filtering are Dawn's native `facets.liquid` system. Filter options will auto-populate based on product data once products are added.

### Language & RTL Support

| Feature | Status | Details |
|---------|--------|---------|
| **Arabic (RTL)** | ⬜ Planned | Primary language at `/`, full RTL layout |
| **English (LTR)** | ⬜ Planned | Secondary language at `/en`, standard LTR layout |
| **Language switcher** | ⬜ Planned | Dawn's built-in switcher in header, mobile menu, and footer |
| **Translation** | ⬜ Planned | Shopify Translate & Adapt app, Arabic is source |
| **`dir="rtl"` attribute** | ⬜ Planned | Dynamic direction on `<html>` tag based on locale |

> See `docs/localization.md` for the full RTL implementation plan.

### Payment

| Feature | Details |
|---------|---------|
| **Gateway** | Paymob |
| **Currency** | EGP only |
| **Methods** | As supported by Paymob (credit/debit cards, mobile wallets, etc.) |

### Shipping

| Feature | Status |
|---------|--------|
| **Shipping method** | ⬜ Not decided yet |
| **Shipping zones** | Egypt only |
| **Rates** | Pending decision (flat rate / free / conditional free shipping) |

### Discounts & Promotions

| Feature | Status |
|---------|--------|
| **Discount codes** | ⬜ Probably yes — decision pending |
| **Sale prices** | ⬜ Probably yes — decision pending |
| **Promotional banners** | ⬜ TBD |

> Shopify supports all of these natively. No custom development needed — just configuration.

### WhatsApp Integration

| Feature | Type | Details |
|---------|------|---------|
| **Chat widget** | Frontend | Floating WhatsApp button on all pages for customers to message the store |
| **Order notifications** | Backend | Automated WhatsApp messages on order events (via middleware) |

### Middleware / ERP Integration

| Feature | Details |
|---------|---------|
| **Architecture** | Next.js API routes on Vercel (free tier) |
| **Shopify → ERP** | Webhook events forwarded to client's ERP system |
| **Notifications** | Dual notification (ERP + WhatsApp) — no order is ever lost |
| **Events** | `orders/create`, `orders/updated`, `orders/cancelled` (more can be added) |

> See `docs/integration.md` for the full middleware plan.

---

## 🧩 STORE COMPONENTS

### Header
- Logo (left in LTR, right in RTL — auto-handled by `dir`)
- Navigation menu (links to collections + pages)
- Search icon (predictive search)
- Cart icon with item count bubble
- Language switcher (AR/EN toggle)
- Mobile: hamburger menu with all of the above

### Footer
- Navigation links
- Social media icons
- Payment method icons
- Language switcher
- Copyright text
- Newsletter signup (if desired — TBD)

### Home Page Sections (cloned from reference store)
- Hero banner / slideshow
- Featured collections
- Featured products grid
- Promotional banners / image with text
- Possibly: testimonials, about snippet, newsletter signup

> Exact sections to be finalized when cloning the reference store layout.

### Product Card (across all collection/grid pages)
- Product image
- Product title
- Price (with compare-at price for sales)
- Quick add to cart
- Variant selector (if applicable)

### Cart Drawer
- Slide-out cart panel
- Item list with quantity controls
- Subtotal / totals
- Checkout button
- Continue shopping link
- In RTL: slides from left side

---

## 📱 RESPONSIVE DESIGN

| Breakpoint | Priority | Notes |
|------------|----------|-------|
| **Mobile** | 🔴 Highest | Most Egypt traffic is mobile. Test everything mobile-first. |
| **Tablet** | 🟡 Medium | Standard responsive behavior |
| **Desktop** | 🟢 Standard | Full layout with all features |

---

## 🔗 THIRD-PARTY INTEGRATIONS

| Service | Purpose | Status |
|---------|---------|--------|
| **Paymob** | Payment gateway | ⬜ To be configured |
| **Shopify Translate & Adapt** | Translation management | ⬜ To be installed |
| **WhatsApp Business API** | Chat widget + order notifications | ⬜ Provider TBD |
| **Client's ERP** | Order/inventory sync | ⬜ Waiting on ERP dev for API endpoint |
| **Google Analytics** | Traffic analytics | ⬜ TBD |
| **Facebook Pixel** | Ad tracking | ⬜ TBD |

---

## ⏳ DEPENDENCIES & BLOCKERS

| Item | From | Status | Blocks |
|------|------|--------|--------|
| Brand persona (name, logo, colors, fonts) | Client | ⬜ Pending | All visual work |
| Product data (titles, descriptions, images, prices) | Client | ⬜ Pending | All product/collection pages |
| Product categories | Client | ⬜ Pending | Navigation menu, collection pages |
| About Us content | Client | ⬜ Pending | About Us page |
| Contact info (phone, email, address, socials) | Client | ⬜ Pending | Contact Us page, footer |
| WhatsApp number | Client | ⬜ Pending | Chat widget, notifications |
| ERP API endpoint + schema | ERP Dev | ⬜ Pending (ETA: few days) | Middleware integration |
| Shipping method decision | Client | ⬜ Pending | Checkout configuration |
| Customer accounts decision | Client | ⬜ Pending | Account pages |
| Discount/promotions decision | Client | ⬜ Pending | Discount configuration |

---

## ✅ WHAT'S DONE

| Item | Status |
|------|--------|
| Dawn theme setup (v15.4.1) | ✅ |
| Header section | ✅ |
| Footer section | ✅ |
| RTL implementation plan | ✅ (`docs/localization.md`) |
| Middleware architecture plan | ✅ (`docs/integration.md`) |
| PRD documentation | ✅ (this file) |
| RTL code audit (all custom code is direction-neutral) | ✅ |

---

## 📝 NOTES

- The store is a **layout clone** of [Snacks Roastery](https://snacks-roastery.com/en) — same structure and UX, different branding and products
- Dawn theme provides most features out of the box (search, sorting, filtering, cart, product pages) — minimal custom development needed
- Arabic is the **source language** — all content is authored in Arabic, English is translated
- Most features are **configuration, not code** — Shopify admin handles payment, shipping, discounts, accounts
- The project is currently at a **natural pause point** — all foundational work is done, remaining work is blocked on client deliverables
- Mobile-first testing is critical — Egyptian market is predominantly mobile
- See `docs/localization.md` for RTL plan, `docs/integration.md` for middleware plan
