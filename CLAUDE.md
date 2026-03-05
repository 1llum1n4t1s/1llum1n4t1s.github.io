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

Build uses Vite + Sass + autoprefixer. Output is **unminified** (`cssMinify: false`).

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
| `_variable.scss` | SCSS variables: breakpoints, max-widths, border-radius, transitions |
| `_theme.scss` | CSS Custom Properties for light/dark mode (auto-switches via `prefers-color-scheme`) |
| `_animations.scss` | `@keyframes` — float (glow movement), fadeUp (scroll reveal), shimmer |
| `_core.scss` | Structural layout, typography, entry cards, comments, pager, sidebar, footer (~970 lines) |
| `_components.scss` | Decorative/visual: category tags, background glows, buttons, Hatena-specific UI (~170 lines) |

Import order in `boilerplate.scss`: normalize.css → _variable → _theme → _animations → _core → _components

**Always edit SCSS sources**, not `build/boilerplate.css` (it's overwritten on build).

### Layout Strategy

`#content-inner` uses CSS Grid. `#box2` and `#box2-inner` (sidebar containers) use `display: contents` to flatten their children into the grid. `order` properties control visual sequence: search box (`-2`) → article (`-1`) → sidebar modules (`0`). Breakpoint at 768px switches from 1-column to 2-column.

### Key CSS Patterns

- **Background glows**: `#container::before/::after` with `position: fixed`, `z-index: -1`, radial gradients. Requires `#container { z-index: 0 }` to create a stacking context (otherwise glows render behind `body` background).
- **Grid overflow prevention**: `#wrapper` needs `min-width: 0` to prevent wide content (tables) from expanding beyond the grid column on mobile.
- **`display: contents` caveat**: `#box2` and `#box2-inner` use `display: contents`, so styles applied directly to them (background, padding, border) have no visible effect. Style their children instead.
- **Noise overlay**: `body::before` with SVG fractal noise, `position: fixed`, `z-index: 9999`.

### CSS Custom Properties

Theme colors are defined as `--` variables in `_theme.scss` on `:root`, with dark mode overrides via `@media (prefers-color-scheme: dark)`. When changing colors, always update both light and dark values. Key variables: `--bg-deep`, `--bg-card`, `--accent-cyan`, `--accent-blue`, `--text-primary`, `--glow-cyan`.

### Portfolio Site

`index.html` and `about.html` have **self-contained inline CSS** in `<style>` tags. They share the same design system (colors, fonts, glows) as the Hatena theme but are independent files.

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

## Temporary Files

`_server.js` and `_test_hatena_bg.html` are development aids for Claude Preview, not part of the core project.
