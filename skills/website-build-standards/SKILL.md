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
`contact-form-integration`, or the `terms-of-use-*` skills come into play
— except `project-discovery`, which precedes even this: its Pages &
features checklist decides the nav this skill turns into folders.

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

If a `PROJECT_BRIEF.md` exists (see `project-discovery`), its Pages &
features section is the nav list to build this tree from directly.

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

**Typography — default to sans-serif, not serif.** AI-assisted builds
tend to reach for an "elegant" serif by default; don't. Unless the
client's existing brand/style guide says otherwise (check
`PROJECT_BRIEF.md`'s Fonts field from `project-discovery` first):

- Default font is **Roboto**, self-hosted as WOFF2 — never a bare
  `<link>` to Google Fonts, and never a bare font name with no fallback
  stack on the `--font-body` token.
- Serif is a deliberate brand decision (editorial, legal, heritage,
  luxury positioning), not a default — only use one when the brief
  calls for it, and say so explicitly.
- One font family, two weights (400 + 600/700) is enough for a typical
  client site — build hierarchy with size/weight/color, not extra
  families or cuts.

Full loading pattern (self-hosting, `@font-face`, fallback stack) →
`references/website-build-standards.md`.

**Monochrome style — opt-in, locked spec.** Applies only when
`project-discovery`'s Brand-assets question (category D) recorded a
strict-monochrome choice. Otherwise this doesn't apply — use the
generic example tokens in the `main.css` skeleton below.

- When it applies, this is a **locked spec, not a starting point**: the
  exact grayscale scale, semantic tokens, and dark/light mechanism in
  `references/website-build-standards.md` replace the skeleton's
  example color tokens verbatim — don't invent new grays or an accent
  color.
- Light/dark mode is **automatic by default**
  (`prefers-color-scheme`, no visible toggle needed). A manual toggle
  button is an optional, documented add-on for when a client
  specifically wants one, not the default expectation.
- Only one non-gray color is permitted: a red destructive/error state
  for form validation and similar. No accent/brand color is introduced
  by this spec.
- Font choice is unaffected — still Roboto per the Typography rule
  above; monochrome governs color/theming only.

Full token values, dark-mode override block, the no-flash toggle
script, section-alternation pattern, and the `.btn`/`.btn-primary`/
`.btn-outline` component → `references/website-build-standards.md`.

**JS**: split by concern (`main.js` global + optional page-specific
files). Never let JS depend on `main.css`'s internal structure.

**Header, navigation, and footer**: the boilerplate's header/nav/footer
aren't placeholder markup — build every page's copy from this exact
pattern.

- **Skip link** — the first focusable element in `<body>`, pointing to
  `#main-content` on `<main>`. Visually hidden by default (off-screen
  positioning, never `display:none`/`visibility:hidden`), revealed on
  `:focus`. Still current WCAG 2.4.1 (Bypass Blocks) guidance, not
  superseded by anything newer.
- **Mobile menu toggle** — a real `<button>` carrying `aria-expanded`
  (flipped `true`/`false` on open/close) and `aria-controls` pointing at
  the nav's `id`. Never a `<div>` with a click handler, and never the
  checkbox-hack (`<input type="checkbox">` + `<label>`) — the label isn't
  a real interactive control and loses proper focus/announcement
  semantics that a `<button>` gets for free. The icon may swap
  hamburger→X; the accessible name stays constant across states.
- **Multi-level nav** (e.g. this skill's own Services > Web Design,
  Hosting example) — use the WAI-ARIA APG's *Disclosure* pattern, not the
  full Menu/Menubar pattern (that's for app-like widgets, not site nav).
  A dedicated toggle button (`aria-expanded`/`aria-controls`) sits beside
  the parent link, so the parent keeps navigating to its own landing page
  while the button reveals the submenu list. Submenus must **not**
  auto-open when a keyboard user Tabs to the parent link — only on
  click/Enter on the toggle. `Escape` closes an open submenu and returns
  focus to its toggle button. On desktop, hover-reveal is acceptable *in
  addition to* the click toggle (pair `:hover` with `:focus-within`,
  never hover-only). On mobile, the same disclosure becomes an in-place
  accordion inside the open mobile panel — never a hover flyout, since
  touch has no hover state.
- **Active-page indication** — `aria-current="page"` on the nav link
  matching the current page. Since this stack has no includes/templating,
  each page's own hardcoded nav copy carries its own `aria-current` on its
  own link — nothing else changes between pages' nav markup.
- **Sticky header — opt-in, not the default.** A sticky header has real
  costs on a small site: it eats mobile viewport space, can visually bury
  a keyboard user's focus outline while they Tab down the page, and
  buries in-page anchor targets. Default to a static header. If sticky
  behavior is specifically wanted, apply it as an explicit modifier and
  pair it with `scroll-padding-top` (or per-target `scroll-margin-top`)
  set to the header's height, so anchor links and the skip-link target
  still land below the fixed bar.
- **Logo** — links to `/`. Alt text is `"[Site Name] logo"` when the
  image is the link's only content; empty `alt=""` when it sits beside
  visible company-name text in the same link, so a screen reader doesn't
  announce the name twice.
- **Footer content** — a sitemap-style link recap, contact info, legal
  links (Terms of Use, Privacy Policy), and a copyright line with the
  year, in a responsive multi-column layout that stacks to one column on
  mobile. This skill only guarantees the *slots* exist (a Terms of Use
  link, a contact email); the legal wording itself is
  `terms-of-use-website`'s job — don't duplicate its "Company
  information" block into the footer, just link to the page that has it.

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
- Every page's nav copy carries `aria-current="page"` on exactly the one
  link matching that page
- The mobile menu toggle is a real `<button>` with `aria-expanded` that
  flips on open/close — not a checkbox hack or a `<div>` click handler
- The skip link is the first focusable element and is visually
  hidden-until-focus, never `display:none`
- If a sticky header is used, `scroll-padding-top`/`scroll-margin-top` is
  set to its height

## Additional resources

- **`references/website-build-standards.md`** — the complete playbook:
  full WordPress-signature removal table with exact grep patterns, a
  worked multi-level nav → folder-tree example, a copy-pasteable semantic
  HTML boilerplate, a full `main.css` skeleton with real example rules
  for each of the five sections above, a typography section covering
  self-hosting Roboto (or a brand font) with `@font-face` and
  `font-display: swap`, and the locked monochrome light/dark token
  system with its no-flash theme script and button component.
