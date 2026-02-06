# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Shopify Dawn theme (v15.4.1)** customized for El Gawhra, an Egyptian e-commerce store. The theme is being adapted for Arabic/English bilingual support with RTL (right-to-left) layout capabilities.

## Development Commands

```bash
# Start local development server (connects to Shopify store)
shopify theme dev

# Push theme to store
shopify theme push

# Pull latest theme from store
shopify theme pull

# Check theme for errors
shopify theme check

# Open theme in browser
shopify theme open
```

## Architecture

### Shopify Theme Structure

- **layout/** - Base HTML wrappers (`theme.liquid` is the main layout)
- **sections/** - Modular page sections configurable via Shopify admin
  - `main-*.liquid` - Primary content sections for each page type
  - `header.liquid`, `footer.liquid` - Global sections
- **snippets/** - Reusable Liquid partials (rendered with `{% render 'snippet-name' %}`)
  - `header-drawer.liquid`, `header-mega-menu.liquid`, `header-dropdown-menu.liquid` - Header navigation components
  - `card-product.liquid`, `card-collection.liquid` - Card components
- **templates/** - JSON files defining which sections appear on each page type
- **assets/** - CSS, JS, SVG, and image files
  - `global.js` - Core JavaScript utilities and Web Components
  - `component-*.css` - Component-specific styles
- **config/** - Theme settings schema and data
- **locales/** - Translation files (supports 30+ languages)

### Key Patterns

- **Web Components**: JavaScript uses custom elements (e.g., `<header-drawer>`, `<cart-drawer>`)
- **CSS Variables**: Colors defined via `--color-*` variables in theme settings
- **Liquid Translations**: Use `{{ 'key' | t }}` for translatable strings; schema labels use `t:` prefix
- **Section Rendering**: Sections can be rendered via `{% section 'section-name' %}` or JSON templates

## RTL/Localization

The theme is being configured for Arabic (primary) + English bilingual support:
- Arabic: `yoursite.com` (default, RTL)
- English: `yoursite.com/en` (LTR)
- RTL detection: Check `request.locale.iso_code == 'ar'`
- See `docs/localization.md` for full implementation plan

### RTL Considerations
- Add `dir="rtl"` to HTML tag for Arabic
- Cart drawer should slide from left in RTL mode
- Icons with direction (arrows, chevrons) need CSS `transform: scaleX(-1)`
- Use CSS logical properties where possible (`margin-inline-start` vs `margin-left`)

## Custom Modifications

Recent customizations include:
- Collections dropdown menu in header with hover behavior
- Custom mega menu and dropdown menu snippets
- Back to Top button section
- Header drawer with Collections submenu
