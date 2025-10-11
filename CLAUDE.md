# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Dawn is Shopify's reference theme representing an HTML-first, JavaScript-only-as-needed approach to theme development. It's a Liquid-based theme built on Shopify's Online Store 2.0 architecture with performance and flexibility as core priorities.

## Development Commands

### Local Development

```bash
# Start local development server with live preview
shopify theme dev
# or
npm run theme:dev
```

### Code Quality

```bash
# Run Theme Check linter
shopify theme check
# or
npm run theme:check

# Format code with Prettier
npm run format

# Check code formatting
npm run format:check
```

### Deployment

```bash
# Push theme as unpublished
shopify theme push --unpublished
# or
npm run theme:push
```

## Architecture

### Core Theme Principles

1. **Web-native first**: Uses vanilla JavaScript (ES6+), no frameworks or libraries. Custom elements (Web Components) are used extensively for interactive functionality.

2. **Server-rendered**: All HTML is rendered server-side using Liquid. JavaScript enhances progressively but never replaces server rendering.

3. **Performance-focused**: Zero Cumulative Layout Shift (CLS), no render-blocking JavaScript, no DOM manipulation before user input, no long tasks.

4. **Progressive enhancement**: Functionality degrades gracefully for older browsers without polyfills.

### Directory Structure

```
dawn/
├── assets/           # CSS, JavaScript, and static assets
│   ├── *.css        # Component and section styles
│   ├── *.js         # Web Components and interactive scripts
│   └── *.svg        # Icons and graphics
├── config/          # Theme settings and configuration
│   ├── settings_schema.json  # Theme customization options
│   └── settings_data.json    # Current settings values
├── layout/          # Theme layouts (wrapper templates)
│   ├── theme.liquid        # Main layout
│   └── password.liquid     # Password protection layout
├── locales/         # Translation files (JSON)
│   ├── en.default.json     # English translations
│   └── *.json              # Other language translations
├── sections/        # Reusable sections (Liquid + schema)
│   ├── header.liquid
│   ├── footer.liquid
│   ├── main-*.liquid       # Main content sections
│   └── *.liquid            # Feature sections
├── snippets/        # Reusable Liquid components
│   ├── card-*.liquid       # Card components
│   ├── product-*.liquid    # Product-related snippets
│   └── *.liquid            # Utility snippets
└── templates/       # Page templates (JSON)
    ├── *.json              # Template configurations
    └── customers/          # Customer account templates
```

### JavaScript Architecture

- **Web Components**: All interactive features use custom elements (`customElements.define()`)
- **No build step**: JavaScript is written in modern ES6+ but must work natively in browsers
- **Event-driven**: Uses pub/sub pattern (`pubsub.js`) for inter-component communication
- **Deferred loading**: Scripts use `defer` attribute to avoid blocking page render
- **Common utilities** in `global.js`:
  - `HTMLUpdateUtility`: For updating DOM with view transitions
  - `SectionId`: For parsing section identifiers
  - `debounce()` / `throttle()`: Performance utilities
  - `fetchConfig()`: Fetch request configuration
  - `trapFocus()` / `removeTrapFocus()`: Accessibility helpers

### Key Web Components

- `<quantity-input>`: Product quantity selectors
- `<variant-selects>`: Product variant pickers
- `<menu-drawer>` / `<header-drawer>`: Navigation menus
- `<modal-dialog>`: Modal overlays
- `<slider-component>`: Image/content sliders
- `<product-recommendations>`: Related products
- `<deferred-media>`: Lazy-loaded videos/embeds

### Liquid Template System

- **Sections**: Self-contained modules with settings defined in `{% schema %}` blocks
- **Snippets**: Reusable Liquid partials included via `{% render 'snippet-name' %}`
- **Templates**: JSON files that compose sections together
- **Localization**: All user-facing text uses `{{ 'path.to.string' | t }}` filter
- **Settings**: Access theme settings via `{{ settings.setting_name }}`
- **Sections groups**: `{% sections 'header-group' %}` and `{% sections 'footer-group' %}` for header/footer

### CSS Architecture

- **Component-scoped**: Each component has its own CSS file (e.g., `component-cart.css`)
- **CSS Custom Properties**: Extensive use of CSS variables defined in `layout/theme.liquid`
- **Base styles**: `base.css` provides foundational styles
- **Responsive**: Mobile-first with `@media screen and (min-width: 750px)` breakpoint
- **Progressive loading**: Non-critical CSS uses `media="print" onload="this.media='all'"`

### Performance Considerations

- **Cart performance tracking**: `CartPerformance` class measures cart operations with Performance API
- **Intersection Observer**: Used for lazy loading (`ProductRecommendations`, `BulkModal`)
- **Resize Observer**: Used for responsive components (`SliderComponent`)
- **Debounced operations**: Cart updates, search, and filters use debouncing

## Coding Standards

### Liquid

- Use `{%- -%}` (with hyphens) to strip whitespace where appropriate
- Prefer `{% liquid %}` tag for multiple sequential statements
- Always use translation filters for user-facing text
- Include `{{ block.shopify_attributes }}` on block elements for theme editor support

### JavaScript

- Write vanilla ES6+, no transpilation
- Use custom elements for interactive components
- Defer script loading when possible
- No DOM queries before user interaction
- Use `const` by default, `let` when mutation needed
- Private class fields use `#` syntax

### CSS

- Use CSS custom properties for themeable values
- Scope styles to components
- Mobile-first responsive design
- Leverage CSS Grid and Flexbox
- Avoid `!important` unless absolutely necessary

### Accessibility

- Include ARIA attributes (`aria-label`, `aria-expanded`, `aria-hidden`)
- Manage focus appropriately with `trapFocus()` utility
- Support keyboard navigation
- Use semantic HTML elements
- Provide screen reader text where needed

## Theme Check Configuration

Theme Check runs automatically in CI and can be run locally. Configuration in `.theme-check.yml`:

- `MatchingTranslations`: Disabled (translations managed separately)
- `TemplateLength`: Disabled (some templates legitimately long)

## Testing and CI

GitHub Actions workflow (`.github/workflows/ci.yml`) runs:

1. **Lighthouse CI**: Performance audits on home, product, and collection pages
2. **Theme Check**: Liquid linting and best practices

## Important Files

- `assets/global.js`: Core utilities and web components
- `layout/theme.liquid`: Main HTML structure, CSS variables, global scripts
- `config/settings_schema.json`: Theme customization UI definition
- `snippets/product-variant-picker.liquid`: Complex product variant logic
- `sections/main-product.liquid`: Product page template logic

## Working with Translations

- Default locale: `locales/en.default.json`
- Schema translations: `*.schema.json` files alongside locale files
- Access in Liquid: `{{ 'general.404.title' | t }}`
- Structure translations by feature area (e.g., `sections.cart.*`, `products.product.*`)

## Common Patterns

### Adding a new section

1. Create `sections/your-section.liquid`
2. Include `{% schema %}` block with section name and settings
3. Add corresponding CSS in `assets/section-your-section.css`
4. Add translations in locale files
5. Include section in template JSON files as needed

### Creating a web component

1. Define in `assets/` or within section
2. Use `customElements.define('your-element', YourClass)`
3. Extend `HTMLElement`
4. Use lifecycle methods: `constructor()`, `connectedCallback()`, `disconnectedCallback()`
5. Clean up event listeners in `disconnectedCallback()`

### Updating cart

- Use fetch with `routes.cart_add_url`, `routes.cart_change_url`, or `routes.cart_update_url`
- Leverage `fetchConfig()` helper for POST requests
- Publish events via pub/sub for cart updates
- Update UI with `HTMLUpdateUtility.viewTransition()`

## Staying Up to Date

To pull latest Dawn changes into a forked repository:

```bash
git remote add upstream https://github.com/Shopify/dawn.git
git fetch upstream
git pull upstream main
```

## Zakres i priorytety

1. Dodawaj funkcje jako **nowe sekcje/snippety** zamiast modyfikować core Dawn, kiedy to możliwe.
2. UI „calm/wellness”: dużo „powietrza”, spokojne kolory, dobra hierarchia typograficzna, bez ciężkich bibliotek.
3. Kluczowe komponenty:
   - **Hero + USP + CTA**
   - **Kafle kategorii (cele zdrowotne)**
   - **Zestawy/bundles** z wyraźną oszczędnością
   - **Badges + price per serving** na PLP/PDP
   - **FAQ/Disclaimer** (PL) dla suplementów
4. Strukturalne dane przez **metafields/metaobjects** (poniżej mapa), by UI było „mądre”.

## Katalogi i pliki, które wolno zmieniać

- `sections/` — nowe sekcje
- `snippets/` — małe, współdzielone fragmenty (badges, ikony, price-per-serving)
- `templates/*.json` — układy (tylko przy dodawaniu nowych układów)
- `assets/` — lekki CSS/JS, SVG
- `locales/` — tłumaczenia i etykiety UI

**Nie zmieniaj bez potrzeby:** `config/`, plików core Dawn i workflowów CI. Nie publikuj live z automatu.

## Standardy jakości (must-have)

- **Theme Check:** zero błędów (`shopify theme check`).
- **Prettier Liquid:** automatyczne formatowanie (plugin `@shopify/prettier-plugin-liquid`).
- **Dostępność:** sensowne `alt`, focus states, role/aria, kontrast; semantyczny HTML.
- **I18N:** teksty w `locales/*.json` (nie hardkoduj UI-copy w kodzie).
- **Wydajność:**
  - Obrazy przez `image_url` + `image_tag`, poprawne `widths/sizes`, lazy-load (domyślne).
  - Tylko 1 hero obraz z `fetchpriority="high"` (jeśli potrzebne).
  - Minimalny JS (bez frameworków; używaj dawnowych utili).
- **Bezpieczeństwo:** brak zewnętrznych skryptów bez zgody; nie dotykaj sekretów/kluczy.

## Metafields (produkty) — konwencja

Ustalony namespace: `nutrition`, `flags`, `usage`, `warnings`, `certs`, `origin`.

- `product.metafields.nutrition.serving_size` (string, np. „2 kaps.”)
- `product.metafields.nutrition.servings_per_container` (integer)
- `product.metafields.nutrition.price_per_serving` (money)
- `product.metafields.flags.vegan` (boolean)
- `product.metafields.flags.gluten_free` (boolean)
- `product.metafields.flags.bio` (boolean)
- `product.metafields.usage.how_to_use` (multi-line text)
- `product.metafields.warnings.contraindications` (multi-line text)
- `product.metafields.certs.list` (list of single-line text / metaobject)
- `product.metafields.origin.country` (single-line text)

**Zasada:** sekcje/snippety korzystają z powyższych pól, a przy braku — degradują się elegancko (ukryj element / placeholder).

## Workflow i komendy

Zanim zaproponujesz commit:

1. Zaplanuj kroki → zaproponuj **unified diff**.
2. Lokalnie zweryfikuj (symulacyjnie):
   ```bash
   npm run format
   shopify theme check
   ```

## Recenzja zmian — checklista

- Theme Check: 0 błędów / 0 ostrzeżeń istotnych
- Prettier Liquid: sformatowane
- Brak twardych tekstów — wszystko w locales/
- Dostępność (alt/aria/focus)
- Brak ciężkich skryptów; rozmiar i liczba assetów pod kontrolą
- Sekcje działają bez metafields (degradacja)
- Instrukcje testu w Customizerze są jasne
- Żadnych zmian w live theme
