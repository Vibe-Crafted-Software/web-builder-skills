---
name: website-build-standards
description: This skill should be used when the user asks to "build a website", "start a new client site", "convert a WordPress site to static HTML", "strip WordPress out of a site", "set up the folder structure for a site", or otherwise needs the foundational build standard (stack, folder layout, CMS-stripping) applied before other work like SEO, sales copy, or forms begins.
version: 1.0.0
---

# Website Build Standards

Establish the foundation a client site is built on: the stack, the folder
layout, and — when the source is a WordPress (or other CMS) export — the
removal of every trace of that CMS. This is the upstream skill: apply it
first, before `website-seo`, `website-sales-tool`,
`contact-form-integration`, or the `terms-of-use-*` skills come into play.

## Scope

Covers stack choice, file/folder layout, and CMS-stripping only. Does
**not** cover:
- Technical/content SEO setup → `website-seo`
- Homepage copy/sales-funnel structure → `website-sales-tool`
- Wiring a contact form to a backend → `contact-form-integration`
- Legal terms pages → `terms-of-use-website` / `terms-of-use-software`

## The stack rule

Plain HTML/CSS/JS. No CMS, no framework (React/Vue/etc.), no bundler, no
build step, no preprocessor. Pages must run correctly served flat off any
static host with zero build tooling.

**One exception**: a small, dependency-free JS utility (e.g. a lightbox or
carousel with no npm dependency) may be vendored in as a single file when
genuinely needed. Never pull in a package manager or framework to get it.

If migrating off a CMS: no inline `<script>`/`<style>` blocks injected by
former plugins may survive, even if they render invisibly today.

## Folder structure standard

Folder-per-page, mirroring the nav menu, with clean URLs (no `.html` in
links). The folder tree should be derivable just by reading the nav — no
digging through code to find where a page lives.

```
/
  index.html
  about/
    index.html
  services/
    index.html
    web-design/
      index.html
    hosting/
      index.html
  contact/
    index.html
  privacy-policy/
    index.html          ← footer-only page still gets a folder matching its URL
  assets/
    css/
      main.css
    js/
      main.js
    img/
```

Rules:
- Every nav item (top-level or nested) gets its own folder containing an
  `index.html`. Sub-menu items nest inside their parent's folder.
- Pages not in the main nav (privacy policy, thank-you pages) still follow
  the same folder-per-URL-path pattern.
- One shared `/assets` at the root with `css/`, `js/`, `img/`
  subfolders — never scattered per-page asset folders.

## WordPress/CMS stripping checklist

When converting a WordPress export to static HTML, remove every platform
fingerprint, not just the visible plugin UI:

- `<meta name="generator" content="WordPress ...">`
- `wp-content/`, `wp-includes/`, `wp-json/` paths — and any asset that
  still points at them
- Plugin-injected CSS/JS and their leftover wrapper classes/data
  attributes (`elementor-*`, `wp-block-*`, `woocommerce-*`)
- `wlwmanifest.xml` link, XML-RPC pingback `<link rel="pingback">`, REST
  API discovery `<link rel="https://api.w.org/...">`
- oEmbed discovery `<link>` tags and the injected oEmbed inline script
- WordPress emoji-detection inline script/style
  (`wp-emoji-release.min.js` and its accompanying inline `<script>`)
- jQuery / jQuery Migrate — remove unless something in the surviving
  vanilla JS genuinely still needs it (WP auto-enqueues it regardless of
  use)
- Leftover shortcode text that never rendered, visible in the page body
  (`[gallery]`, `[contact-form-7 ...]`)
- Comment-system remnants (native WP comments markup, Disqus embed,
  `#comments` anchors with nothing behind them)
- Gutenberg block wrapper `<div>`s/classes that add no styling once the
  block-library CSS is gone
- Any `admin-ajax.php` or REST endpoint referenced by leftover inline JS

See `references/website-build-standards.md` for the full table (exact
strings/selectors to grep for and what to do about each) and notes on
other CMS/builder exports (Wix, Squarespace).

## Baseline HTML/CSS/JS conventions

**HTML**: semantic HTML5 landmarks (`<header>`, `<nav>`, `<main>`,
`<footer>`), one `<meta name="viewport">`, no inline `style=`/`onclick=`
attributes.

**CSS — one central stylesheet, `/assets/css/main.css`.** For a site this
size, one cached stylesheet loaded on every page beats splitting per page
or per component — fewer requests, simpler maintenance, and the
duplication/complexity tradeoff that justifies per-page splitting only
shows up at large multi-team scale (e.g. gov.uk), not a client marketing
site. Structure the single file top-to-bottom so later sections can
safely override earlier ones without fighting specificity:

1. **Tokens** — CSS custom properties on `:root` (colors, font stack,
   spacing scale, breakpoints). Every rule below references a token, never
   a raw hex/px value.
2. **Reset/base** — a small modern reset (box-sizing, margin reset,
   `img{max-width:100%}`) plus bare element defaults.
3. **Layout** — page-level structural rules (containers, shared
   grid/flex scaffolding).
4. **Components** — one block per reusable UI piece (nav, card, button,
   footer), named with a light BEM convention (`.card`, `.card__title`,
   `.card--featured`) so class names stay collision-free as the file
   grows.
5. **Utilities** — small single-purpose helpers, used sparingly
   (`.visually-hidden`, `.text-center`).

Mobile-first media queries (base styles unprefixed, `min-width` queries
layer up). No `!important` — fix source order/selector specificity
instead.

**Escape hatch, not the default**: only add a second stylesheet once
`main.css` demonstrably becomes unwieldy for a human to navigate, and call
that out to the user explicitly rather than splitting proactively.

**JS**: split by concern (`main.js` global + optional page-specific
files). Never let JS depend on `main.css`'s internal structure.

## Verification checklist

Before calling a build or migration done:

- `grep -ri wordpress`, `grep -ri wp-content`, `grep -ri wp-json` across
  the entire output return nothing
- No 404s for stripped asset paths (check the network tab)
- Every nav link resolves to a real folder/`index.html` per the folder
  standard above
- Site runs correctly served flat, with no server-side includes or build
  step required
- No leftover CMS admin links, generator meta tags, or plugin references
  in `<head>`

## Additional resources

- **`references/website-build-standards.md`** — the complete playbook:
  full WordPress-signature removal table with exact grep patterns, a
  worked multi-level nav → folder-tree example, a copy-pasteable semantic
  HTML boilerplate, and a full `main.css` skeleton with real example
  rules for each of the five sections above.
