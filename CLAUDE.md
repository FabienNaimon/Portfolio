# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A personal portfolio site for Fabien Naimon (Game / Tech Designer). It is a **static, hand-authored site with no build system, no dependencies, and no tooling** — just two HTML files at the repo root, each with all CSS and JavaScript inlined in a single `<style>` and `<script>` block.

- `index.html` — the main portfolio (hero, projects, about, CV, contact).
- `freelance.html` — a standalone freelance landing page that links back to `index.html` and shares the same design system.

Everything else at the root is a static asset: images in `images/`, plus a few loose `.jpg`/`.webp`/`.pdf` files (CV PDFs, project docs). Binary assets are committed directly to git (there is no `.gitattributes` / Git LFS setup, despite a residual `[lfs]` block in `.git/config`).

## Running & deploying

There is nothing to build. To preview, open `index.html` in a browser directly, or serve the folder:

```bash
python -m http.server 8000   # then visit http://localhost:8000
```

Deployment is via the GitHub repo `FabienNaimon/Portfolio` (GitHub Pages serves `index.html` from `main`). Committing to `main` publishes; there is no CI, test, or lint step.

## Architecture

### Single-page navigation inside index.html
The site is a faux-SPA built without a router. All content lives in one document:

- `#main-page` holds the scrolling sections (`#projects`, `#about`, `#cv`, `#contact`).
- Each project detail view is a separate full-screen `<div class="project-page" id="project-<id>">` that is hidden by default and shown with the `.open` class.

Navigation is driven by a handful of global functions in the `<script>` at the bottom of `index.html`:

- `openProject(id)` — hides `#main-page`, shows `#project-<id>`, and calls `history.pushState({project:id}, ...)`. Project cards invoke it inline: `onclick="openProject('way'); return false;"`.
- `closeProject()` / `showMain()` — return to the main page and scroll back to `#projects`.
- The `popstate` handler makes browser back/forward and the Escape key work by reading `history.state`. Escape calls `history.back()` rather than closing directly, so state stays consistent.
- `stopVideos()` resets embedded iframe `src` values to halt playback when leaving a project page.

**Adding a project** requires two coordinated edits in `index.html`: (1) a `.project-card` anchor in the `#projects` grid whose `onclick` calls `openProject('<newid>')`, and (2) a matching `<div class="project-page" id="project-<newid>">` block. The `id` suffix must match on both sides.

### Design system (duplicated per file)
There is no shared stylesheet. Both HTML files redeclare the same CSS custom properties in `:root` (dark theme; gold accent `--accent:#c8a96e`; text/border tokens). Fonts are **Syne** + **Outfit** loaded from Google Fonts. If you change a design token, colour, or nav style, apply the same edit in **both** `index.html` and `freelance.html` to keep them consistent.

### Other shared behaviours
- **Lightbox**: `openLightbox(src, alt)` / `closeLightbox()`. On `DOMContentLoaded`, images inside `.pp-screenshots` are wired to open in the lightbox.
- **Contact form**: both pages POST to the same Formspree endpoint (`https://formspree.io/f/mzdygwjg`) via `fetch`, with inline success/error messaging — no page reload.

## Conventions

- Keep CSS and JS inline within each HTML file; do not introduce a bundler, framework, or external JS/CSS files unless explicitly asked.
- Prefer editing the existing inline blocks over adding new `<link>`/`<script src>` references.
- Match the existing terse, minified-ish CSS style (multiple selectors per line) already present in the files.
