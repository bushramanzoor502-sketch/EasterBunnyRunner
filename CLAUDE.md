# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A **static, no-build marketing website** for the *Easter Bunny Runner* Android game.
Three hand-written HTML pages (`index.html`, `privacy.html`, `contact.html`), one shared
stylesheet (`assets/css/styles.css`), one shared script (`assets/js/main.js`), and ~34 game-art
images in `assets/img/`. There is **no package.json, no framework, no bundler, no tests, and no lint step.**

## Run / deploy

```bash
# from the repo root — any static server works
python -m http.server 8080      # then open http://localhost:8080
```

Opening `index.html` directly via `file://` also works. Deploy by uploading the whole folder to
any static host (GitHub Pages, Netlify, Vercel, Firebase Hosting).

## Architecture (the parts that span files)

**Progressive enhancement is a hard contract, not a nicety.** Every page must render all its content
and stay usable with JavaScript disabled or if the CDN scripts (GSAP, ScrollTrigger, Lenis, all loaded
from CDN at the bottom of each page) fail to load. Animations are purely additive. When adding content,
never gate meaning behind JS.

**The reveal pattern is the main trap.** `main.js` adds `class="js"` to `<html>` on load. CSS hides
`.reveal` / `[data-stagger]` elements *only* when `html.js` is present, and JS reveals them via
ScrollTrigger — or, if ScrollTrigger/GSAP is missing or the user prefers reduced motion, via
`revealAllImmediately()`. If you add a hidden-then-revealed element, it must carry `.reveal` (or live
inside a `[data-stagger]` group) so one of those code paths un-hides it; otherwise it vanishes for
no-JS users. Don't invent a new hide-by-default mechanism.

**JS behavior is wired entirely through data attributes** in `main.js` (one IIFE, feature-detected,
never throws if a lib is absent):
- `[data-hero]` / `[data-hero-el]` — hero intro timeline
- `[data-parallax="0.5"]` — scroll parallax depth
- `[data-stagger]` — its direct children fade/rise in sequence
- `[data-count]` (+ optional `[data-suffix]`) — animated number counter
- `[data-mailto-form]` + `data-mailto="addr"` — the contact form (see below)
- `[data-lenis-prevent]` — opt a scroll container out of Lenis smooth-scroll

**No templating — nav and footer are copy-pasted into all three HTML files.** Any change to the navbar
or footer must be replicated across `index.html`, `privacy.html`, and `contact.html`. The `.active`
nav link differs per page. The `© <span id="year">` is filled by an inline script at the bottom of
each page.

**Contact form has no backend.** `initMailtoForm()` validates fields, builds a `mailto:` URL from the
form's `data-mailto` attribute, and opens the visitor's email client. There is no server round-trip.

## Design system

CSS custom properties in `:root` at the top of `assets/css/styles.css` are the single source of truth
for color, radius, shadow, and layout width — deliberately mirrored from the game's `Color.kt` tokens.
Use the existing vars (`--mint`, `--peach-deep`, `--ink`, `--radius`, `--maxw`, etc.) rather than
hardcoding values. Font is Fredoka (Google Fonts). Responsive breakpoints live at the bottom of the
file: `980px`, `680px`, `480px`, plus a `prefers-reduced-motion` block.

## Before shipping — placeholders to replace

- **Google Play link:** `#REPLACE_WITH_PLAY_STORE_URL` appears 5 times (3× `index.html`, 1× each in
  `privacy.html` / `contact.html`). Replace every occurrence with the real listing URL.
- **Contact email:** `bushramanzoor502@gmail.com` is hardcoded in `contact.html` (form `data-mailto`
  + visible links) and in every page footer.

Images in `assets/img/` are independent copies of the game's drawable art — editing them here does not
affect the game.
