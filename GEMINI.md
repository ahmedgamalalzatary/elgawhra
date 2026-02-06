# Gemini CLI Project Context: El Gawhra Shopify Theme

This project is a customized **Shopify Dawn theme (v15.4.1)** for **El Gawhra**, an Egyptian e-commerce store. It is specifically architected for Arabic (primary, RTL) and English (secondary, LTR) bilingual support.

## Project Overview

- **Core Technologies**: Liquid, JavaScript (Web Components), CSS (BEM, Custom Properties), JSON.
- **Primary Market**: Egypt (EGP currency).
- **Localization**:
  - **Arabic**: Default locale, root URL (`/`), RTL (Right-to-Left) layout.
  - **English**: Secondary locale, subdirectory URL (`/en`), LTR (Left-to-Right) layout.
- **Architecture**: Standard Shopify theme structure optimized for modular sections and reusable snippets.

## Development Workflow

### Key Commands (Shopify CLI)
Ensure you have the Shopify CLI installed and authenticated.

- `shopify theme dev`: Start the local development server with hot-reloading.
- `shopify theme pull`: Pull the latest theme changes from the store.
- `shopify theme push`: Push local changes to the theme on the store.
- `shopify theme check`: Run a linter for Liquid and JSON files to ensure best practices.
- `shopify theme open`: Open the theme preview in your default browser.

### Git Conventions
- **DO NOT** commit or push changes unless explicitly instructed.
- Follow existing styles for commit messages if requested.

## Project Structure

- **`layout/`**: Contains `theme.liquid`, the main wrapper. Direction (`dir="rtl"` or `ltr`) is dynamically set here.
- **`sections/`**: Modular components configurable via the Shopify Theme Editor. Prefix `main-` for page-specific content.
- **`snippets/`**: Reusable Liquid partials rendered via `{% render %}`.
- **`assets/`**:
  - `global.js`: Core JavaScript implementing Web Components.
  - `component-*.css`: Component-specific styling.
  - `rtl.css`: (If implemented/referenced) specifically for RTL overrides.
- **`locales/`**: JSON translation files for the storefront interface.
- **`config/`**: Theme settings and schema definitions.

## Coding Standards & Patterns

### Liquid & HTML
- **Directionality**: Use `{% if request.locale.iso_code == 'ar' %}rtl{% else %}ltr{% endif %}` to handle layout logic.
- **Snippets**: Always use `{% render 'snippet-name' %}` instead of the deprecated `{% include %}`.
- **Clean Logic**: Use `{%- -%}` for whitespace control and `{% liquid %}` for multi-line logic blocks.

### JavaScript (Web Components)
- Custom elements are used extensively (e.g., `<cart-drawer>`, `<header-drawer>`).
- Follow the ES6+ class pattern for custom elements as seen in `assets/*.js`.
- Accessibility (ARIA) is a priority; ensure all interactive elements have proper roles and states.

### CSS
- **BEM Naming**: `block__element--modifier`.
- **CSS Variables**: Use theme-defined variables like `rgb(var(--color-foreground))`.
- **RTL Support**: Prefer CSS logical properties (`margin-inline-start`, `padding-inline-end`) where possible, or use `[dir="rtl"]` selectors for explicit overrides.

## Implementation Priorities (RTL/Bilingual)
- **Header**: Language switcher at the end of the icon row.
- **Cart Drawer**: Must slide from the **left** in RTL mode.
- **Icon Mirroring**: Arrows and chevrons must be flipped (`transform: scaleX(-1)`) in RTL.
- **Footer**: Column order should be reversed in RTL.

## Reference Files
- `AGENTS.md`: Detailed guidelines for AI agents working on this specific codebase.
- `CLAUDE.md`: Overview and architecture guidance.
- `docs/localization.md`: The roadmap for the AR/EN RTL implementation.
