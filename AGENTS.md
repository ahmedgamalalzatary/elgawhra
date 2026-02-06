# AGENTS.md

Guidelines for AI agents working on this Shopify Dawn theme (v15.4.1) for El Gawhra, an Egyptian e-commerce store with Arabic/English bilingual support.

## Build & Development Commands

```bash
# Start local development server with hot reload
shopify theme dev

# Connect to specific store
shopify theme dev --store=your-store.myshopify.com

# Push theme to store (use --theme=THEME_ID to avoid overwriting live)
shopify theme push
shopify theme push --theme=THEME_ID

# Pull latest theme from store
shopify theme pull

# Lint theme for Liquid/JSON errors and best practices
shopify theme check

# Lint a single file
shopify theme check sections/header.liquid

# Open theme preview in browser
shopify theme open

# List all themes / Package as zip
shopify theme list
shopify theme package
```

> **Note:** There are no unit tests in this project. Use `shopify theme check` for validation.

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

**File Naming:**
- Sections: `kebab-case.liquid` (e.g., `main-product.liquid`, `featured-collection.liquid`)
- Snippets: `kebab-case.liquid` (e.g., `card-product.liquid`)

**Syntax Rules:**
- Always use `{% render 'snippet-name' %}` — never `{% include %}` (deprecated)
- Use `{%- -%}` (with hyphens) to strip whitespace when formatting matters
- Use `{% liquid %}` tag for multi-line logic blocks
- Apply `| escape` filter for user-generated content
- Translations: `{{ 'key.path' | t }}` for UI strings
- Schema labels: use `t:` prefix (e.g., `"label": "t:sections.header.name"`)

**Required Snippet Documentation:**
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

**Naming Conventions:**
- Classes: `PascalCase` (e.g., `CartDrawer`, `HTMLUpdateUtility`)
- Functions/methods: `camelCase` (e.g., `getFocusableElements`, `trapFocus`)
- Private fields: `#` prefix (e.g., `static #separator = '__'`)
- Custom element tags: `kebab-case` (e.g., `<cart-drawer>`, `<header-drawer>`)

**Best Practices:**
- Use ES6+: arrow functions, template literals, destructuring, optional chaining (`?.`), nullish coalescing (`??`)
- Prefer `const` over `let`; never use `var`
- DOM queries: use `querySelector`/`querySelectorAll` over `getElementById`
- Guard clauses for early returns: `if (!element) return;`

**Accessibility (Required):**
- Set ARIA attributes: `role`, `aria-expanded`, `aria-controls`, `aria-label`
- Implement keyboard navigation (Escape to close, Tab trapping in modals)
- Use `focus-inset` class for focus indicators

**Organization:**
- Component styles: `assets/component-*.css`
- Section-specific styles: inline `<style>` within section file
- Conditional CSS loading:
```html
<link rel="stylesheet" href="{{ 'component.css' | asset_url }}" media="print" onload="this.media='all'">
```

**IDs:** PascalCase with hyphens for dynamic parts (e.g., `Details-HeaderMenu-{{ handle }}`)

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

This theme supports Arabic (primary, RTL) and English (secondary, LTR):

| Language | URL | Direction |
|----------|-----|-----------|
| Arabic | `/` (root) | RTL |
| English | `/en` | LTR |

**Detection:** `request.locale.iso_code == 'ar'`

**RTL CSS Rules:**
- Use CSS logical properties: `margin-inline-start` instead of `margin-left`
- Use `[dir="rtl"]` selector for RTL-specific overrides
- Flip directional icons: `transform: scaleX(-1)` in RTL
- Cart drawer slides from left in RTL mode

**Implementation:** See `docs/localization.md` for full roadmap.

## Git Conventions

- **DO NOT** commit or push changes unless explicitly instructed
- Follow existing commit message styles when requested
