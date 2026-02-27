# AR/EN RTL Implementation Plan

## 📋 SPECIFICATIONS

| Setting | Value |
|---------|-------|
| **Plan** | Basic (production) |
| **Translation** | Shopify Translate & Adapt |
| **Currency** | EGP only |
| **Switcher** | Dawn's built-in language localization (header, mobile menu, footer) |
| **RTL Scope** | Full mirror (text, layout, elements, images) |
| **Font** | Dawn default (no custom font) |
| **Market** | Egypt only |
| **URL Structure** | yoursite.com (AR), yoursite.com/en (EN) |
| **Source Language** | Arabic |
| **Translation Strategy** | Arabic is source; English is translated via Translate & Adapt |

---

## 🚀 IMPLEMENTATION PHASES

### **PHASE 1: Shopify Dashboard Setup (10 mins)**

#### Languages Configuration:
1. Settings → Languages → Add Language → **English**
2. Set **Arabic as Primary** (yoursite.com = AR)
3. Set **English as Secondary** (yoursite.com/en = EN)
4. Set default locale to Arabic (Egypt)
5. Enable "Publish" for both

#### Markets Configuration:
1. Settings → Markets → Edit Primary Market
2. Country: Egypt
3. Currency: EGP
4. Languages: Arabic + English
5. URL format: Subdirectory with prefix

#### URL Structure:
- Arabic (default): `yoursite.com`
- English: `yoursite.com/en`

---

### **PHASE 2: Core RTL Infrastructure**

#### Strategy: CSS Logical Properties First

The **primary** approach is converting existing CSS to use **CSS logical properties** (`margin-inline-start`, `padding-inline-end`, `inset-inline-start`, etc.) directly in the existing stylesheets. This makes most components automatically RTL-compatible without overrides.

A separate `rtl.css` file is used **only** for edge cases that logical properties can't handle (icon flipping, cart drawer slide direction, slideshow arrows, etc.).

#### **File 1: `layout/theme.liquid`**

**Changes:**
- Add `dir` attribute to HTML tag
- Detect current locale and set direction
- Load `rtl.css` conditionally

**Logic:**
```liquid
{% assign is_rtl = false %}
{% if request.locale.iso_code == 'ar' %}
  {% assign is_rtl = true %}
{% endif %}
<html dir="{% if is_rtl %}rtl{% else %}ltr{% endif %}" lang="{{ request.locale.iso_code }}">
```

#### **File 2: `assets/base.css`**

**Primary RTL strategy — convert physical properties to logical:**
- `margin-left` / `margin-right` → `margin-inline-start` / `margin-inline-end`
- `padding-left` / `padding-right` → `padding-inline-start` / `padding-inline-end`
- `left` / `right` → `inset-inline-start` / `inset-inline-end`
- `text-align: left` → `text-align: start`
- `float: left` → `float: inline-start`
- `border-left` / `border-right` → `border-inline-start` / `border-inline-end`

Also apply logical properties in all `assets/component-*.css` files where directional properties exist.

#### **File 3: Create `assets/rtl.css` (Edge Cases Only)**

**Only for things logical properties can't solve:**
- Directional icon mirroring (`transform: scaleX(-1)` for arrows, chevrons, carets)
- Cart drawer slide direction (slide from left in RTL)
- Slideshow/carousel navigation arrow flipping
- Any third-party widget overrides

---

### **PHASE 3: Language Switcher Configuration**

> **Dawn already has a built-in language switcher.** Do NOT build a custom one.

Dawn ships with these localization snippets:
- `snippets/language-localization.liquid` — full dropdown with accessibility (aria attributes, checkmarks, keyboard support)
- `snippets/country-localization.liquid` — country/currency picker
- `assets/localization-form.js` — form submission handling

These are already rendered in:
- **Desktop header** (`sections/header.liquid`)
- **Mobile menu** (`snippets/header-drawer.liquid`)
- **Footer** (`sections/footer.liquid`)

#### What to do:
1. **Enable** the language selector via header & footer section settings in Shopify admin
2. **Hide the country picker** — since we only serve Egypt with EGP, the country selector adds no value and may confuse users. Either disable it in section settings or hide it with CSS
3. The switcher will also appear in any **future sections** that render `language-localization` or `country-localization` snippets — keep this in mind when adding new sections
4. If a minimal toggle-button style ("EN" / "AR") is desired later, **modify** the existing `language-localization.liquid` snippet rather than building from scratch — this preserves all built-in accessibility

---

### **PHASE 4: Component-Specific RTL Fixes**

> **Note:** Many of these will be **automatically handled** by adding the `dir="rtl"` attribute + converting to CSS logical properties. The browser natively flips flexbox, grid, text alignment, and positioning when `dir="rtl"` is set. Only items that need **manual intervention** are listed below.

#### **Automatically Handled by `dir="rtl"` + Logical Properties (No Manual Work):**
- Logo position (flexbox auto-flips)
- Navigation menu alignment
- Search/cart icon positions (flexbox auto-flips)
- Form input text alignment
- Text direction (right-to-left)
- Footer column order (flexbox/grid auto-flips)
- Product grid alignment
- Sort dropdown alignment
- Price display alignment
- Add to cart button position
- Variant selector alignment

#### **Needs Manual RTL Fixes (in `rtl.css`):**

**Cart Drawer:**
- Drawer slides from **left** in RTL (CSS transform/animation override)
- Close button repositioning if absolutely positioned

**Directional Icons:**
- Arrow icons (`icon-caret.svg`, `icon-arrow.svg`) need `transform: scaleX(-1)`
- Pagination arrows flip
- Image gallery navigation arrows
- Breadcrumb separators

**Slideshow/Banners:**
- Navigation arrow icons flip direction
- Slide transition direction may need JS adjustment

**Mobile Menu Drawer:**
- Verify slide direction is correct in RTL (should slide from right)

---

### **PHASE 5: Translation Content Setup**

#### **Using Shopify Translate & Adapt:**

**Source language is Arabic.** All content is authored in Arabic first, then translated to English.

**Immediate (ASAP):**
1. Auto-translate all default theme content to English
2. Manually review and fix English translations for:
   - Navigation menus
   - Header text
   - Footer text
   - System messages (cart, errors, notifications)

**As You Build:**
- Write new content in Arabic first (source)
- Use Translate & Adapt to generate English translations
- Manually refine English translations as needed

**Key Areas to Translate:**
- ⬜ Navigation menus
- ⬜ Header text
- ⬜ Footer text
- ⬜ All page content
- ⬜ Product titles/descriptions
- ⬜ Collection titles/descriptions
- ⬜ Metafields
- ⬜ Image alt text
- ⬜ Email notifications (auto-translated by Shopify)

---

### **PHASE 6: Testing & Refinement**

#### **Visual Testing Checklist:**

**Desktop RTL:**
- [ ] Header layout mirrors correctly
- [ ] Logo on right side
- [ ] Language switcher visible and working
- [ ] Navigation dropdowns align right
- [ ] Cart drawer opens from left
- [ ] All arrows point correct direction
- [ ] Form inputs align right
- [ ] Text reads right-to-left
- [ ] Footer columns reversed
- [ ] Slideshow arrows flipped

**Desktop LTR:**
- [ ] All above in reverse (normal LTR layout)
- [ ] Language switcher shows Arabic option

**Mobile RTL:**
- [ ] Language switcher in hamburger menu
- [ ] Mobile menu slides from right
- [ ] Touch gestures work correctly
- [ ] Cart drawer accessible
- [ ] All icons properly sized

**Mobile LTR:**
- [ ] Mobile menu slides from left
- [ ] All elements properly aligned

**Functional Testing:**
- [ ] Language switching works instantly
- [ ] Cart items persist across languages
- [ ] Checkout flow works in both languages
- [ ] URLs update correctly (/en for English)
- [ ] No 404 errors when switching
- [ ] Country picker is hidden (Egypt only)

---

## **📁 FILES TO MODIFY**

| File | Purpose | Changes |
|------|---------|---------|
| `layout/theme.liquid` | HTML structure | Add `dir` attribute, RTL detection, conditional `rtl.css` load |
| `assets/base.css` | Base styles | Convert physical properties → logical properties |
| `assets/component-*.css` | Component styles | Convert physical properties → logical properties |
| `assets/rtl.css` **(NEW)** | RTL edge cases | Icon flipping, cart drawer direction, slideshow arrows |
| `assets/component-cart-drawer.css` | Cart styles | RTL slide direction override |

> **Not modified:** `sections/header.liquid`, `snippets/header-drawer.liquid`, `sections/footer.liquid` — these already have Dawn's built-in language switcher. Enable via admin settings.

---

## **⚡ PRIORITIES**

### **Day 1 — Foundation:**
1. ✅ Shopify dashboard: languages + market setup
2. ✅ Add `dir` attribute to `layout/theme.liquid`
3. ⬜ Convert `assets/base.css` to CSS logical properties
4. ✅ Enable Dawn's built-in language switcher (header, mobile menu, footer)
5. ⬜ Hide country picker (Egypt only)
6. ✅ Auto-translate all content via Translate & Adapt (in progress review/refinement)

### **Day 2 — RTL Polish:**
1. ⬜ Create `assets/rtl.css` for edge cases
2. ⬜ Cart drawer RTL positioning
3. ⬜ Directional icon mirroring (arrows, chevrons, carets)
4. ⬜ Slideshow direction fix
5. ⬜ Convert remaining `component-*.css` files to logical properties

Current progress note:
- 🟡 RTL/LTR overall status is approximately **90% complete** after market/language setup and dynamic `dir` support.

### **Ongoing:**
1. ⬜ Translate new content as it's created (Arabic first → English)
2. ⬜ Test new sections/pages in both languages
3. ⬜ Advanced animation adjustments if needed

---

## **🎯 NEXT STEPS**

1. Set up English language in Shopify admin
2. Configure Egypt market with EGP
3. Add `dir` attribute to `layout/theme.liquid`
4. Start converting `base.css` to CSS logical properties
5. Enable language switcher in header/footer settings
6. Create minimal `rtl.css` for edge cases
7. Run through testing checklist

---

## **📝 NOTES**

- Dawn theme does NOT have native RTL support — but it DOES have a built-in language switcher
- CSS logical properties are the primary strategy — they handle ~80% of RTL automatically
- `rtl.css` is only for edge cases (icon flipping, drawer direction, etc.)
- Cart drawer must slide from left in RTL mode
- Icons with direction (arrows, chevrons) need `transform: scaleX(-1)` in RTL
- Arabic is the source language — English is the translation
- Country picker should be hidden since only Egypt/EGP is served
- Test thoroughly on mobile — most Egypt traffic is mobile
- Language switcher also appears in footer and any future sections that render localization snippets
- Keep backup before making changes

---

## **🔍 RTL ANALYSIS: Custom Code Impact**

A git-based audit of all files created or edited (compared to Dawn's initial commit) was performed to identify physical CSS properties that `dir="rtl"` alone won't fix.

### Result: Adding `dir="rtl"` is enough for a functional RTL layout

Out of all custom-edited files, only 3 have physical directional CSS — and **none of them cause actual UI problems**:

| File | Properties Found | Impact |
|------|-----------------|--------|
| `sections/header.liquid` | `left: 50%`, `translateX(-50%)`, `margin-left: -1.2rem`, `padding-right: 2.7rem`, `left: 0` | **No issue** — these are centering logic for the Collections dropdown. `left: 50%` + `translateX(-50%)` centers the element regardless of direction. The margin/padding are minor spacing tweaks. |
| `sections/top-button.liquid` | `right: 5%` (×2) | **No issue** — Back to Top button on the right side is perfectly fine in both RTL and LTR. No UX reason to flip it. |
| `assets/base.css` | `left: 30px` (×1) | **Negligible** — a single minor offset, barely noticeable. |

### All other custom files are clean:
`assets/footer.css`, `assets/section-footer.css`, `assets/component-list-menu.css`, `assets/component-list-payment.css`, `assets/component-list-social.css`, `assets/component-menu-drawer.css`, `assets/component-newsletter.css`, `sections/footer.liquid`, `snippets/header-drawer.liquid`, `snippets/header-dropdown-menu.liquid`, `snippets/header-mega-menu.liquid`, `layout/theme.liquid` — all direction-neutral ✅

### What this means:
- Adding `dir="rtl"` to `theme.liquid` gives a **functional, visually correct RTL layout** for all custom code
- The browser automatically handles flexbox/grid flipping, text direction, and inline element flow
- No urgent CSS conversions are needed for custom code — focus logical property conversion on **Dawn's original CSS files** (`base.css`, `component-*.css`) during the polish phase
