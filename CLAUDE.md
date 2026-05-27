# CLAUDE.md

This file is the canonical agent guide for this repository. Claude Code should follow this file when working with code here.

## Project Overview

This is a **GitHub Pages** static site (`https://1llum1n4t1s.github.io/`) serving as ゆろち's **portfolio site**:

- `index.html` — Free software showcase (Windows apps, Chrome extensions, NuGet libraries)
- `about.html` — Profile page

Both pages have **self-contained inline CSS** in `<style>` tags and are independent files. They share the same design system (colors, fonts, soft pastel palette + OneDark dark mode) by convention.

> **Note**: The Hatena Blog theme "Illuminatis" was previously developed in this repo under `Hatena-Blog-Theme-Boilerplate/`. As of 2026-05, it has been moved to the dedicated [`1llum1n4t1s/Blog`](https://github.com/1llum1n4t1s/Blog) repository (`theme/` subdirectory). For theme work, switch to that repository.

## Adding a New App/Library to the Portfolio

When adding a new product to `index.html`, update **all four** locations:

1. **Card HTML** — Add a `<div class="app-card" data-accent="...">` inside the appropriate `tab-panel` (`#tab-chrome`, `#tab-windows`, `#tab-nuget`, or `#tab-websites`). Each card has: `.card-icon`, `.card-name`, `.card-desc`, `.card-tags`, and `.card-link` (or `<div class="card-links">` containing multiple `.card-link` children when an app ships on multiple stores — e.g. Chrome Web Store + Firefox AMO + GitHub).
2. **Structured data** — Add a `SoftwareApplication` entry to the JSON-LD `ItemList` in `<head>`, incrementing `position`. Set `operatingSystem` accurately (e.g. `"Google Chrome, Mozilla Firefox"` for cross-browser extensions).
3. **Counts** — Update three numbers: `numberOfItems` in JSON-LD, `.stat-number` in the hero section, and the `<meta name="description">` product count.
4. **Section description** — If the new product changes the scope of a section, update the `<p class="section-desc">` text.

Accent colors available: `cyan`, `blue`, `purple`, `emerald`.

## Deployment

`index.html` / `about.html` and other static assets deploy automatically via GitHub Pages on push to `main`.

## Local-only dev aids

`_server.js` and `_test_hatena_bg.html` were Hatena theme preview helpers (gitignored). They remain in the working tree for legacy local use but are no longer referenced by this repository's workflow — see the Blog repository for the current preview path.
