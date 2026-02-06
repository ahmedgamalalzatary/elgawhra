# AR/EN RTL Implementation Plan

## 📋 SPECIFICATIONS

| Setting | Value |
|---------|-------|
| **Plan** | Basic (production) |
| **Translation** | Shopify Translate & Adapt |
| **Currency** | EGP only |
| **Switcher Position** | Header - at the end (after cart) |
| **Switcher Style** | Toggle buttons showing available options |
| **RTL Scope** | Full mirror (text, layout, elements, images) |
| **Font** | Dawn default (no custom font) |
| **Market** | Egypt only |
| **URL Structure** | yoursite.com (AR), yoursite.com/en (EN) |
| **Mobile Switcher** | In mobile menu (hamburger) |
| **Translation Strategy** | Translate as you build (header/footer done, rest default) |

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

#### **File 1: `layout/theme.liquid`**

**Changes:**
- Add `dir` attribute to HTML tag
- Detect current locale and set direction
- Load RTL CSS conditionally

**Logic:**
```liquid
{% assign is_rtl = false %}
{% if request.locale.iso_code == 'ar' %}
  {% assign is_rtl = true %}
{% endif %}
<html dir="{% if is_rtl %}rtl{% else %}ltr{% endif %}" lang="{{ request.locale.iso_code }}">
```

#### **File 2: Create `assets/rtl.css`**

**Comprehensive RTL styles covering:**
- Base RTL reset (text-align, direction)
- Layout flipping (margin-left → margin-right)
- Icon mirroring (transform: scaleX(-1))
- Flex/grid direction adjustments
- Cart drawer position (left side in RTL)
- Navigation dropdown alignment
- Form input alignment
- Slideshow/carousel direction reversal
- Header icon order reversal
- Footer column order

#### **File 3: `assets/base.css`**

**Add:**
- CSS logical properties where needed
- Support for `[dir="rtl"]` selectors
- Transition effects for direction changes

---

### **PHASE 3: Language Switcher Implementation**

#### **Desktop Header (`sections/header.liquid`)**

**Position:** End of icon row (Search → Account → Cart → **Language**)

**Implementation:**
- Toggle buttons: Shows the OTHER language
- If on Arabic → Shows "EN" button
- If on English → Shows "AR" button
- Style: Clean buttons with active/hover states
- Icons: Optional flag or just text

**Code Structure:**
```liquid
<div class="language-switcher">
  {% for locale in shop.published_locales %}
    {% unless locale.iso_code == request.locale.iso_code %}
      <a href="{{ locale.root_url }}" class="lang-btn">{{ locale.iso_code | upcase }}</a>
    {% endunless %}
  {% endfor %}
</div>
```

#### **Mobile Menu (`snippets/header-drawer.liquid`)**

**Add to mobile menu drawer:**
- Language toggle at top or bottom of menu
- Same style as desktop but adapted for mobile
- Clear visual separation from navigation items

---

### **PHASE 4: Component-Specific RTL Fixes**

#### **Header Section:**
- Logo position (left in LTR, right in RTL)
- Navigation menu alignment
- Search icon position
- Cart icon position
- Language switcher position

#### **Cart Drawer:**
- Drawer slides from **left** in RTL (not right)
- Close button position
- Item layout mirroring
- Quantity buttons alignment

#### **Product Pages:**
- Image gallery arrows flip
- Thumbnail navigation direction
- Variant selector alignment
- Price display alignment
- Add to cart button position

#### **Collection Pages:**
- Filter sidebar position (left in RTL)
- Product grid alignment
- Sort dropdown alignment
- Pagination arrows flip

#### **Footer:**
- Column order reversal (col-1 becomes last)
- Newsletter form alignment
- Social icons direction
- Payment icons alignment

#### **Slideshow/Banners:**
- Navigation arrows flip direction
- Slide direction (RTL = right to left)
- Text overlay alignment

---

### **PHASE 5: Translation Content Setup**

#### **Using Shopify Translate & Adapt:**

**Immediate (ASAP):**
1. Auto-translate all default content
2. Manually review and fix:
   - Navigation menus
   - Header text
   - Footer text
   - System messages (cart, errors)

**As You Build:**
- Translate new sections as you customize them
- Keep English as source language
- Arabic translations auto-generated, then refine

**Key Areas to Translate:**
- ✅ Header (already done - needs Arabic version)
- ✅ Footer (already done - needs Arabic version)
- ⬜ Navigation menus (create separate for each language)
- ⬜ All page content
- ⬜ Product titles/descriptions
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
- [ ] All above in reverse (normal)
- [ ] Language switcher shows "AR"

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

---

## **📁 FILES TO MODIFY**

| File | Purpose | Changes |
|------|---------|---------|
| `layout/theme.liquid` | HTML structure | Add dir attribute, RTL detection |
| `assets/rtl.css` (NEW) | RTL styles | Comprehensive RTL stylesheet |
| `assets/base.css` | Base styles | Add logical properties |
| `sections/header.liquid` | Header | Add language switcher |
| `snippets/header-drawer.liquid` | Mobile menu | Add language switcher |
| `assets/component-cart-drawer.css` | Cart styles | RTL positioning |
| `assets/component-slideshow.css` | Slideshow | Arrow directions |
| `assets/component-facets.css` | Filters | Layout adjustments |
| Various section files | Components | Minor RTL tweaks |

---

## **⚡ ASAP PRIORITIES**

### **Must Have (Day 1):**
1. ✅ Shopify languages setup
2. ✅ HTML dir attribute
3. ✅ Basic RTL CSS (layout, text, forms)
4. ✅ Language switcher in header
5. ✅ Language switcher in mobile menu
6. ✅ Cart drawer RTL positioning
7. ✅ Auto-translate all content

### **Should Have (Day 2):**
1. Icon mirroring (arrows, chevrons)
2. Slideshow direction fix
3. Navigation menu RTL
4. Footer column reversal
5. Mobile menu RTL

### **Nice to Have (Ongoing):**
1. Advanced animation adjustments
2. Custom font loading optimization
3. Image mirroring for specific graphics
4. Advanced hover states

---

## **🎯 NEXT STEPS**

**Ready to implement?**

1. Set up English language in Shopify admin
2. Start with `layout/theme.liquid` - add dir attribute
3. Create `assets/rtl.css` with comprehensive RTL styles
4. Add language switcher to header
5. Add language switcher to mobile menu
6. Test and refine

---

## **📝 NOTES**

- Dawn theme does NOT have native RTL support
- All physical CSS properties need RTL overrides
- Cart drawer must slide from left in RTL mode
- Icons with direction (arrows, chevrons) need flipping
- Consider creating mirrored versions of text-on-image graphics
- Test thoroughly on mobile - most Egypt traffic is mobile
- Keep backup before making changes
