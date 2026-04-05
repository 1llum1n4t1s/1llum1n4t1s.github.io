# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **GitHub Pages** static site (`https://1llum1n4t1s.github.io/`) with two roles:

1. **Portfolio site** — `index.html` (free software showcase), `about.html` (profile)
2. **Hatena Blog theme "Illuminatis"** — `Hatena-Blog-Theme-Boilerplate/` builds a CSS file applied to `https://1llum1n4t1.org/`

Detailed specs and past debugging notes are in `doc/PROJECT_OVERVIEW.md` and `doc/DEBUGGING_GUIDE.md` (both written in Japanese).

## Build Commands

```bash
cd Hatena-Blog-Theme-Boilerplate
npm install       # first time only
npm run build     # SCSS → build/boilerplate.css
```

Build uses Vite 8 + Sass (with `@use` module system) + autoprefixer. Output is **unminified** (`cssMinify: false`). Requires Node `^20.19.0 || >=22.12.0`.

## Local Preview

A Claude Preview dev server is configured in `.claude/launch.json`:
- Server name: `static-server` (runs `node _server.js` on port 3000)
- Default page: `_test_hatena_bg.html` (mock Hatena Blog DOM structure)

Alternatively, the Vite dev server supports live-reloading against a real Hatena Blog (requires `<script>`/`<link>` tags injected into the blog's `<head>` settings — see `Hatena-Blog-Theme-Boilerplate/README.md` for setup):
```bash
cd Hatena-Blog-Theme-Boilerplate
npm start -- 1llum1n4t1.org
```

## Architecture

### Hatena Blog Theme (main development target)

**Critical constraint**: Hatena Blog has a **fixed HTML structure**. Only CSS customization is possible — no HTML or class names can be added. See `doc/PROJECT_OVERVIEW.md` for the full DOM tree.

**SCSS modules** (`Hatena-Blog-Theme-Boilerplate/scss/lib/`):

| File | Purpose |
|------|---------|
| `_variable.scss` | SCSS variables: breakpoints, max-widths, border-radius (`$radius-sm/md/lg/pill`), transitions |
| `_theme.scss` | CSS Custom Properties for light/dark mode (auto-switches via `prefers-color-scheme`) |
| `_animations.scss` | `@keyframes` — float (glow movement), fadeUp (scroll reveal), shimmer |
| `_core.scss` | Structural layout, typography, entry cards, comments, pager, sidebar, footer (~970 lines) |
| `_components.scss` | Decorative/visual: category tags, background glows, buttons, Hatena-specific UI (~170 lines) |

Load order in `boilerplate.scss` (uses `@use`, not `@import`): normalize.css → _variable → _theme → _animations → _core → _components. Files that need SCSS variables use `@use 'variable' as *` at the top.

**Always edit SCSS sources**, not `build/boilerplate.css` (it's overwritten on build).

### Layout Strategy

`#content-inner` uses CSS Grid. `#box2` and `#box2-inner` (sidebar containers) use `display: contents` to flatten their children into the grid. `order` properties control visual sequence: search box (`-2`) → article (`-1`) → sidebar modules (`0`). Breakpoint at 768px switches from 1-column to 2-column.

### Key CSS Patterns

- **Background glows**: `#container::before/::after` with `position: fixed`, `z-index: -1`, radial gradients, `will-change: transform` for GPU layer promotion. Requires `#container { z-index: 0 }` to create a stacking context (otherwise glows render behind `body` background).
- **Grid overflow prevention**: `#wrapper` needs `min-width: 0` to prevent wide content (tables) from expanding beyond the grid column on mobile.
- **`display: contents` caveat**: `#box2` and `#box2-inner` use `display: contents`, so styles applied directly to them (background, padding, border) have no visible effect. Style their children instead.
- **Noise overlay**: `body::before` with SVG fractal noise, `position: fixed`, `z-index: 9999`, `transform: translateZ(0)` for GPU layer promotion.
- **Search form DRY pattern**: `_core.scss` uses `%search-form-base`, `%search-input-base`, `%search-button-base` placeholders shared by `.search-form`/`.search-module-*` and `.search-result-form`/`.search-result-*` via `@extend`.
- **Gradient text (Safari)**: `#title a` needs both `-webkit-text-fill-color: transparent` and `color: transparent` for cross-browser gradient text. Removing the `-webkit-` prefixed version breaks Safari/iOS.

### CSS Custom Properties

Theme colors are defined as `--` variables in `_theme.scss` on `:root`, with dark mode overrides via `@media (prefers-color-scheme: dark)`. When changing colors, always update both light and dark values. Key variables: `--bg-deep`, `--bg-card`, `--accent-cyan`, `--accent-blue`, `--text-primary`, `--glow-cyan`.

### Portfolio Site

`index.html` and `about.html` have **self-contained inline CSS** in `<style>` tags. They share the same design system (colors, fonts, glows) as the Hatena theme but are independent files.

## Adding a New App/Library to the Portfolio

When adding a new product to `index.html`, update **all four** locations:

1. **Card HTML** — Add an `<div class="app-card" data-accent="...">` inside the appropriate `tab-panel` (`#tab-chrome`, `#tab-windows`, `#tab-nuget`, or `#tab-websites`). Each card has: `.card-icon`, `.card-name`, `.card-desc`, `.card-tags`, and `.card-link`.
2. **Structured data** — Add a `SoftwareApplication` entry to the JSON-LD `ItemList` in `<head>`, incrementing `position`.
3. **Counts** — Update three numbers: `numberOfItems` in JSON-LD, `.stat-number` in the hero section, and the `<meta name="description">` product count.
4. **Section description** — If the new product changes the scope of a section (e.g., adding non-fork libraries to the NuGet section), update the `<p class="section-desc">` text.

Accent colors available: `cyan`, `blue`, `purple`, `emerald`.

## Deployment

1. Run `npm run build` in `Hatena-Blog-Theme-Boilerplate/`
2. Copy contents of `build/boilerplate.css`
3. Paste into Hatena Blog admin → Design → Customize → Design CSS
4. The portfolio pages (`index.html`, `about.html`) deploy automatically via GitHub Pages

## Breakpoints

- `480px` (max): extra-small mobile
- `768px` (min): tablet — triggers 2-column grid
- `992px` (min): desktop
- `1200px` (min): wide screen

## Hatena Blog DOM Pitfalls

Hatena Blog's actual DOM class names sometimes differ from what you'd expect. **Always verify class names against the live site** (`https://1llum1n4t1.org/`) using Chrome DevTools MCP before writing CSS selectors. Known examples:
- Hatena Star container: actual class is `.star-container` (not `.hatena-star-container`)
- Category links: actual class is `.archive-category-link` (not `.entry-category-link`)

## Temporary Files

`_server.js` and `_test_hatena_bg.html` are development aids for Claude Preview, not part of the core project.
