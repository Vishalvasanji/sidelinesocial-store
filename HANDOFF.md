# Sideline Social — Developer Handoff

_Last updated: 2026-07-27 · repo tip: `main` @ `dc5ee19`_

This document is the single entry point for a developer picking up this project. It covers what the product is, how the repo is organized, the marketing homepage (the main artifact built so far), the Shopify theme (the eventual store), the current state, known issues, and the recommended roadmap.

---

## 1. What this is

**Sideline Social** is a **custom team-store service for youth sports**. The pitch: a team/coach signs up, Sideline Social designs the gear and builds/runs an online store, and players + parents + fans buy premium team merch that ships direct — no inventory, no order-collecting, no forms for the team. Tagline: _"Team Pride. Better Design."_

**Two audiences:**
- **Teams / coaches** — the buyer of the service ("Start a Team Store").
- **Players / parents / fans** — shoppers of a team's store.

**Business model:** B2B2C service (build-your-team-store), **not** a generic apparel retailer. This distinction matters for design/positioning decisions (the homepage leads with the _offer_, not with content or a product catalog).

---

## 2. Repo structure — two parallel layers

The repository contains **two separate things** that are currently decoupled:

```
sidelinesocial-store/
├── docs/                     # ← LAYER 1: static marketing homepage (GitHub Pages). Most work lives here.
│   ├── index.html            #   the whole homepage (single file, ~560 lines)
│   ├── styles.css            #   all styles + design tokens (~454 lines)
│   ├── fonts/                #   archivo-variable.woff2, inter-variable.woff2 (self-hosted)
│   └── img/                  #   logos, hero.png, comparison.png, 8× tile-*.png
│
├── (Shopify "Horizon" theme) # ← LAYER 2: the eventual real store (near-complete, EMPTY of products)
│   ├── layout/               #   theme.liquid, password.liquid
│   ├── templates/            #   product/collection/cart/search/… (JSON) + gift_card/password (liquid)
│   ├── sections/             #   47 sections: 6 custom ss-*.liquid + stock Horizon
│   ├── snippets/             #   96 snippets (stock Horizon)
│   ├── blocks/, assets/, config/, locales/
│   └── theme_export__…zip    #   theme backup export
└── HANDOFF.md                # ← this file
```

> **Key mental model:** `docs/` is a **hand-coded prototype homepage** deployed on GitHub Pages. The Shopify theme is a **separate, production-ready store skeleton** with **no products in it and no bridge to the marketing site**. Reconciling these two is the biggest open architectural decision (see §7).

---

## 3. LAYER 1 — The marketing homepage (`docs/`)

### 3.1 Tech stack
- **Plain HTML + CSS + vanilla JS. No build step, no framework, no dependencies.** Edit the files directly.
- **Deployment:** GitHub Pages, served from the `docs/` directory. Live under `vishalvasanji.github.io` (the screenshot users see). Merging to `main` deploys.
- **Fonts:** self-hosted variable woff2 (Archivo for headings, Inter for body), `preload`ed in `<head>`.
- **Cache-busting:** the stylesheet is linked as `styles.css?v=NN`. **Bump `NN` on every CSS change** (currently `?v=42`, `index.html` line 11) or GitHub Pages will serve stale CSS.

### 3.2 Homepage sections (top → bottom)
All sections live in `docs/index.html`, each marked with a numbered `<!-- N. … -->` comment.

| # | Section | Class / notes |
|---|---------|---------------|
| 1 | Announcement bar | `.ss-announce` — "Free shipping on $75+" |
| 2 | Header (sticky) | `.ss-header` — logo (icon + wordmark that **collapses on scroll**), nav, search/account/cart icons, hamburger + mobile nav |
| 3 | Hero | `.ss-hero scheme-dark` — "Team Pride. Better Design." + CTAs + trust stars + product image; faint "S" watermark |
| 4 | Category tiles | `.ss-staples` — **full-bleed** filmstrip (Youth / Women / Adult / Hoodies & Crew / Tees & Tanks), lifestyle photos |
| 5 | "More than just a logo" | `.ss-proof scheme-light` — eyebrow + headline + **full-bleed comparison image** + 4 benefit pillars |
| 6 | Mid-page CTA strip | `.ss-ctastrip scheme-dark` — compact conversion nudge |
| 7 | How It Works | `.ss-hiw scheme-light-2` — 5 numbered step cards |
| 7b | Why Sideline Social | `.ss-why scheme-dark` — **2-column sticky accordion** (left: clickable items; right: crossfading panel). Mobile: inline accordion. |
| 8 | Featured Team Stores | `.ss-stores scheme-light-2` — **light "gallery"** with a featured card + horizontally-scrolling team cards (arrows) |
| 9 | Final CTA | `.ss-cta scheme-dark` — "Your team. Your store. Your pride." |
| 10 | Trust strip | `.ss-trust scheme-dark` — 4 trust items |
| 11 | Footer | `.ss-footer scheme-dark` — nav columns, newsletter, socials |

### 3.3 Design system (all in `docs/styles.css` `:root`)
The site is a **dark athletic** brand with a **two-accent** system (coral + electric blue). Everything is token-driven; **use the tokens, don't hardcode values.**

**Color**
- Base darks: `--ss-bg-dark #0a0b0d`, `--ss-bg-deep #0e0f12`, `--ss-bg-panel #17181c`; lights `--ss-bg-light #f7f8fa`, `--ss-bg-white #fff`.
- Accents: `--coral #fb6a45` (CTAs, key headline words), `--blue #3b82f6` (eyebrows, numbers, icons, borders). RGB-channel tokens `--coral-rgb / --blue-rgb / --ink-rgb` exist for `rgba()` glows.
- Text on light: `--ss-text-dark #101318`, `--ss-text-muted #6f7680`.

**Scheme system (the important pattern).** Each section gets a scheme class that sets _local_ vars (`--bg / --panel / --fg / --muted / --heading / --border`, plus `--fg-strong / --faint` on dark). Style components through these local vars so a section ports between light/dark by swapping one class:
- `.scheme-dark` (near-black), `.scheme-deep` (slightly deeper), `.scheme-light` (white), `.scheme-light-2` (light gray).
- Dark text tiers: `--fg-strong` 0.85 · `--fg` 0.74 · `--muted` 0.5 · `--faint` 0.4 (white opacities).

**Type**
- `--font-heading: Archivo` (condensed, uppercase, `font-stretch: 110%` for titles), `--font-body: Inter`.
- Section-title tiers: "statement" titles ~3.8rem (proof / Why / final CTA); "functional" titles 2.7rem via `.ss-sec-title` (How It Works / Featured). Eyebrows `.ss-eyebrow` (blue, 0.74rem, uppercase, tracked).

**Shape / elevation / motion**
- Radii: `--radius 16px` (cards/panels), `--radius-sm 10px` (buttons/chips/inputs), `50%` circles.
- Shadows: `--shadow-card` (light-bg card elevation), `--shadow-btn` (coral glow on primary-button hover). Dark cards use inset borders/blue glows, not drop shadows.
- Motion tokens: `--dur-fast 0.15s` · `--dur-med 0.3s` · `--dur-slow 0.4s` · `--dur-reveal 0.6s`. Easing is `ease`.

**Layout**
- `--container-max 1300px`, `--page-margin 20px` (mobile) / `40px` (≥760px). `.container` centers content to that width — **every section shares this content spine** except the two intentional full-bleed moments (category tiles, comparison image).
- `--ss-header-h 61px` (sticky-header offset).
- **Breakpoints:** 640 (phone nav), 760 (grids / margin), 900 (final-CTA cols), 960 (hero + How-It-Works + Why + stores desktop), 1080 (desktop nav).

### 3.4 JavaScript (all inline at the bottom of `index.html`, ~line 466)
Five small IIFEs, progressively enhanced (gated on an `html.js` class added in `<head>`):
1. **Scroll reveal** — `IntersectionObserver` adds `.is-visible` to `.reveal` elements (staggered via inline `--reveal-delay`).
2. **Featured-stores carousel** — arrow buttons scroll the `#storesTrack`.
3. **Mobile nav toggle** — hamburger toggles `.is-open`.
4. **Header scroll-collapse** — adds `.is-scrolled` past 8px (collapses the logo wordmark).
5. **Why Sideline accordion** — clones each item's detail into a sticky right-panel "stage" and crossfades on click (desktop); inline expand/collapse (mobile). Respects `prefers-reduced-motion`.

### 3.5 Conventions
- **BEM-ish naming:** `ss-block__element` (e.g. `ss-why__item`, `ss-store-card__name`).
- State classes: `.is-active`, `.is-open`, `.is-visible`, `.is-scrolled`.
- Buttons: `.btn` + `.btn--primary` (coral fill) / `.btn--secondary` (outline).
- **Always bump the `?v=` cache-bust when CSS changes.**

---

## 4. LAYER 2 — The Shopify "Horizon" theme (the eventual store)

A near-complete **Shopify Horizon** theme lives at the repo root (~87% stock Horizon + 6 custom sections). It is a **production-ready store skeleton with no products loaded**.

**Templates present:** `index`, `product`, `collection`, `cart`, `search`, `list-collections`, `blog`, `article`, `page`, `page.contact`, `404`, `password`, `gift_card`. These wire up PDP, PLP, cart, search, blog, and content pages out of the box.

**Templates MISSING:** `templates/customers/*` — **no account, login, register, order-history, or address pages.** These must be added for a real store.

**Custom sections** (`sections/ss-*.liquid`, mirror the homepage prototype): `ss-hero`, `ss-categories`, `ss-differentiator`, `ss-benefits`, `ss-how-it-works`, `ss-final-cta`. The live homepage (`templates/index.json`) is built from these.

**Config:** `config/settings_data.json` + `settings_schema.json` carry the brand (color schemes, fonts Archivo/Inter, button/card settings). Cart is drawer-mode.

**Porting strategy (agreed earlier — hybrid):** bake the coral/blue accents + Archivo into Horizon's admin-editable **color schemes** and **font settings** so merchants can edit them in the theme editor; keep the signature **split logo + scroll-collapse** as a small isolated custom snippet. The 6 `ss-*` sections already consume `color_scheme`, so they inherit brand tokens automatically.

---

## 5. Current state — what's real vs. not

**Done / polished**
- The `docs/` homepage is visually complete and refined (see §8), responsive, and deployed.
- The Horizon theme has all core commerce templates.

**Decorative / not wired (homepage)**
- **All nav links, category tiles, "Shop Gear", "Start a Team Store", cart, and search are `href="#"` / non-functional.** There is no browse → product → cart → checkout path in the prototype.
- Two **placeholder blocks** remain: the **Final-CTA media** (`.ph ph--dark` box, `index.html:395`) and the **Featured-store "preview"** (4 light `.ph` boxes, `index.html:346`).

**Missing entirely**
- **No product catalog** anywhere (no products/collections in Shopify — the store can't transact yet).
- No customer-account templates (§4).
- No favicon / Open Graph / Twitter card / JSON-LD in `docs/index.html <head>`.
- No analytics.

---

## 6. Known issues & TODOs

| Priority | Item | Detail |
|---|---|---|
| High | **Mobile comparison image is too small** | `comparison.png` is a wide ~2.4:1 graphic with text baked in; on phones it collapses to a ~180px band and the ✗ bullet points/labels become illegible. Fix: pull the label/bullet copy out of the image into real HTML, and/or ship a portrait/stacked mobile crop via `<picture>`. |
| High | **Wire the homepage links** | Everything is `href="#"`. Depends on the architecture decision (§7). |
| High | **No catalog** | Build collections + products (with variants, pricing, photography) in Shopify. |
| Med | **Image weight** | `docs/img/` ships **~19 MB of PNGs** (8 tiles ≈ 1.7–2.4 MB each, `comparison.png` 2.4 MB, `hero.png` 0.7 MB). Convert to WebP/AVIF + responsive `srcset` (target ~1–2 MB total). A canvas-based WebP re-encode was prototyped (~19 MB → ~0.7 MB) but **not committed**. |
| Med | **`<head>` polish** | Add favicon, OG/Twitter, canonical, JSON-LD `Organization`. |
| Med | **Customer-account templates** | Add `templates/customers/*` to the Horizon theme. |
| Med | **Replace the 2 placeholders** | Final-CTA media + Featured-store preview need real imagery. |
| Low | **Real photography** | Category tiles / hero / Featured stores use stock-style placeholders; swap in real team photos + named testimonials. |

---

## 7. Recommended roadmap

The pivotal decision is **how the marketing site and the store relate.** Recommendation: **consolidate onto Shopify** — make the Horizon store the real site and port the `docs/` homepage into the theme (the 6 `ss-*` sections already mirror it). This fixes the dead links, the empty-store problem, and the duplicated effort at once.

Sequence: **1)** decide architecture → **2)** finish + wire the homepage (kill placeholders, wire links, compress images, fix `<head>`) → **3)** populate catalog + collections + photography → **4)** add account templates + elevate PDP/PLP/cart/search → **5)** youth-sports differentiators (team-store "windows" with countdowns, group/roster ordering, player name/number, player/parent/fan segmentation) → **6)** cross-cutting (reviews, performance, a11y, SEO, analytics).

---

## 8. Design-system history (why the tokens look deliberate)

The homepage went through an evaluation + refinement pass (PRs #46–#50). Net result: unified section-title hierarchy, one card family, balanced light/dark rhythm, a card/button elevation model, and a clean token system for **color, radius, motion, and text-opacity**. If you touch styling, **work through the tokens** — they were deliberately consolidated (e.g. one `--radius-sm`, four `--dur-*` tokens, four dark-text opacity tiers). Deliberately **not** done: a formal `--space-N` spacing scale (judged not worth the churn).

---

## 9. Dev workflow

- **Branching:** feature work happens on `claude/serene-hypatia-6553ll`; PRs merge to `main`. (Adjust to your own branch naming.)
- **Preview locally:** open `docs/index.html` directly, or (recommended, so fonts load same-origin) `python3 -m http.server -d docs 8000` → `http://localhost:8000`.
- **Visual QA:** the project was verified with headless Playwright/Chromium screenshots at 1440 (desktop) and 390 (mobile), forcing `.reveal` visible and awaiting `document.fonts.ready`. There is no test suite — QA is visual.
- **On CSS change:** bump `styles.css?v=NN` in `index.html`.
- **Deploy:** merge to `main` → GitHub Pages rebuilds `docs/`.

## 10. Quick reference

| Thing | Where |
|---|---|
| Homepage markup | `docs/index.html` |
| Styles + design tokens | `docs/styles.css` (`:root`, then schemes) |
| Inline JS | bottom of `docs/index.html` (~line 466) |
| Fonts | `docs/fonts/` (Archivo, Inter) |
| Images | `docs/img/` |
| Shopify homepage | `templates/index.json` + `sections/ss-*.liquid` |
| Shopify brand config | `config/settings_data.json` |
| Cache-bust version | `index.html` line 11 (`?v=42`) |
