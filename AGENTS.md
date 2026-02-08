# AGENTS.md

Guidelines for AI agents working on this Shopify Dawn theme (v15.4.1) for El Gawhra, an Egyptian e-commerce store with Arabic/English bilingual support.

## Build & Development Commands

```bash
shopify theme dev                              # Start local dev server with hot reload
shopify theme dev --store=store.myshopify.com  # Connect to specific store
shopify theme check                            # Lint all files for Liquid/JSON errors
shopify theme check sections/header.liquid     # Lint a single file
shopify theme push                             # Push theme to store
shopify theme push --theme=THEME_ID            # Push to specific theme (avoids overwriting live)
shopify theme pull                             # Pull latest theme from store
shopify theme open                             # Open theme preview in browser
shopify theme package                          # Package as zip
```

> **Note:** No unit tests exist. Use `shopify theme check` for validation.

## Project Structure

| Directory | Purpose |
|-----------|---------|
| `layout/` | Base HTML wrappers (`theme.liquid` is main layout) |
| `sections/` | Modular page sections with schema at bottom; prefix `main-` for page content |
| `snippets/` | Reusable Liquid partials rendered via `{% render %}` |
| `templates/` | JSON files defining which sections appear on each page type |
| `assets/` | CSS (`component-*.css`), JS (Web Components), SVGs, images |
| `config/` | Theme settings schema and data |
| `locales/` | Translation JSON files (30+ languages) |

## Code Style Guidelines

### Liquid Templates

**File Naming:** `kebab-case.liquid` (e.g., `main-product.liquid`, `card-product.liquid`)

**Syntax Rules:**
- Use `{% render 'snippet-name' %}` — never `{% include %}` (deprecated)
- Use `{%- -%}` (with hyphens) to strip whitespace when formatting matters
- Use `{% liquid %}` tag for multi-line logic blocks
- Apply `| escape` filter for user-generated content
- Translations: `{{ 'key.path' | t }}` for UI strings
- Schema labels: use `t:` prefix (e.g., `"label": "t:sections.header.name"`)

**Snippet Documentation (Required):**
```liquid
{% comment %}
  Renders a product card

  Accepts:
  - card_product: {Object} Product Liquid object (optional)
  - show_vendor: {Boolean} Show the product vendor. Default: false
  - lazy_load: {Boolean} Image should be lazy loaded. Default: true

  Usage:
  {% render 'card-product', show_vendor: section.settings.show_vendor %}
{% endcomment %}
```

**Error Handling:**
- Check for `blank` or `empty` before accessing properties
- Use `| default:` filter for fallback values
- Wrap optional features in `{%- if setting -%}` blocks


### JavaScript (Web Components)

**Naming Conventions:**
- Classes: `PascalCase` (e.g., `CartDrawer`, `HTMLUpdateUtility`, `SectionId`)
- Functions/methods: `camelCase` (e.g., `getFocusableElements`, `trapFocus`)
- Private fields: `#` prefix (e.g., `static #separator = '__'`)
- Custom element tags: `kebab-case` (e.g., `<cart-drawer>`, `<product-info>`)


**Best Practices:**
- Use ES6+: arrow functions, template literals, destructuring, optional chaining (`?.`), nullish coalescing (`??`)
- Prefer `const` over `let`; never use `var`
- DOM queries: `querySelector`/`querySelectorAll` over `getElementById`
- Guard clauses: `if (!element) return;`

**Accessibility (Required):**
- Set ARIA attributes: `role`, `aria-expanded`, `aria-controls`, `aria-label`, `aria-haspopup`
- Implement keyboard navigation (Escape to close, Tab trapping in modals)
- Use `focus-inset` class for focus indicators
- Use `trapFocus(container, element)` and `removeTrapFocus(element)` utilities from `global.js`

### CSS

**Organization:**
- Component styles: `assets/component-*.css`
- Section-specific styles: inline `<style>` within section file
- Conditional CSS loading with media print trick for async loading

**RTL Rules:**
- Use CSS logical properties: `margin-inline-start` instead of `margin-left`
- Use `[dir="rtl"]` selector for RTL-specific overrides
- Flip directional icons: `transform: scaleX(-1)` in RTL

### JSON Templates & Schemas

**Template Structure** (`templates/*.json`):
```json
{
  "sections": {
    "main": { "type": "main-product", "settings": {} }
  },
  "order": ["main"]
}
```

**Section Schema** (at bottom of `.liquid` file):
```liquid
{% schema %}
{
  "name": "t:sections.header.name",
  "settings": [
    {
      "type": "select",
      "id": "menu_type_desktop",
      "options": [{ "value": "dropdown", "label": "t:sections.header.settings.dropdown.label" }],
      "default": "dropdown",
      "label": "t:sections.header.settings.menu_type_desktop.label"
    }
  ]
}
{% endschema %}
```

## RTL/Localization

| Language | URL | Direction |
|----------|-----|-----------|
| Arabic | `/` (root) | RTL |
| English | `/en` | LTR |

**Detection:** `request.locale.iso_code == 'ar'`

## Git Conventions

- **DO NOT** commit or push changes unless explicitly instructed
- Follow existing commit message styles when requested

## RTL Implementation Status

**Adding `dir="rtl"` to `theme.liquid` is sufficient for a functional RTL layout.** A git audit of all custom code confirmed that no edited/created files have physical CSS properties that cause UI problems in RTL. The browser's native `dir="rtl"` handling (flexbox/grid flipping, text direction, inline flow) covers all custom components.

Key points for future sessions:
- All custom CSS (footer, header drawer, dropdown menus, etc.) is already direction-neutral
- `sections/header.liquid` dropdown uses centering logic (`left: 50%` + `translateX(-50%)`) which works in both directions
- `sections/top-button.liquid` uses `right: 5%` which is acceptable in both RTL and LTR
- **Dawn's original CSS** (`base.css`, `component-*.css`) still has physical properties that need logical property conversion — but that's a polish task, not a blocker
- See `docs/localization.md` for the full RTL implementation plan and detailed audit results

## Key Utilities (global.js)

- `getFocusableElements(container)` - Returns array of focusable elements
- `trapFocus(container, element)` - Traps focus within container
- `removeTrapFocus(element)` - Removes focus trap and returns focus
- `SectionId` class - Parses/builds qualified section IDs
- `HTMLUpdateUtility` class - Safe HTML swapping with script reinjection
