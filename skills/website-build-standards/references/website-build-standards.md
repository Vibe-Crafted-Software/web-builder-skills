# Website Build Standards — Complete Playbook

Full detail behind `SKILL.md`: the exact WordPress-signature removal
table, a worked folder-tree example, a copy-pasteable HTML boilerplate,
and a full `main.css` skeleton.

## 1. WordPress/CMS signature removal — full table

Work through every row. "Where it appears" tells you what to search the
exported HTML/assets for; "Fix" tells you what to do once found.

| Signature | Where it appears | Search for | Fix |
|---|---|---|---|
| Generator meta tag | `<head>` | `name="generator"` | Delete the tag |
| Core asset paths | `<link>`/`<script src>` | `wp-content/`, `wp-includes/`, `wp-json/` | Re-host the actual asset under `/assets/` and rewrite the path, or delete if unused |
| Theme/plugin CSS bundles | `<link rel="stylesheet">` | filenames under `wp-content/themes/` or `wp-content/plugins/` | Fold any styles actually in use into `main.css`; delete the rest |
| Elementor wrapper markup | body HTML | `elementor-`, `data-elementor-*` | Rewrite the markup as plain semantic HTML with your own classes |
| WooCommerce markup | body HTML | `woocommerce-`, `wc-` | Remove if the site isn't keeping e-commerce; otherwise rebuild as static product markup |
| Gutenberg block wrappers | body HTML | `wp-block-`, `class="wp-block"` | Strip wrapper `<div>`s that add no styling once plugin CSS is gone; keep only the real content |
| WPForms/Contact Form 7 leftovers | body HTML | `wpforms-`, `wpcf7`, literal `[contact-form-7 ...]` text | Remove; replace with the site's real form markup (see `contact-form-integration` skill) |
| Yoast/RankMath injected meta | `<head>` | `class="yoast-schema-graph"`, comments like `<!-- This site is optimized with the Yoast SEO plugin -->` | Delete; rebuild meta/structured data per the `website-seo` skill |
| Windows Live Writer manifest | `<head>` | `wlwmanifest.xml` | Delete the `<link>` |
| XML-RPC pingback | `<head>` | `rel="pingback"` | Delete the `<link>` |
| REST API discovery | `<head>` | `rel="https://api.w.org/"` | Delete the `<link>` |
| oEmbed discovery + injector | `<head>` / inline `<script>` | `rel="alternate" type="application/json+oembed"`, `wp-embed.min.js`, inline script containing `window.wp.receiveEmbedMessage` | Delete both the `<link>` tags and the inline script |
| Emoji detection | inline `<script>`/`<style>` in `<head>` | `wp-emoji-release.min.js`, inline script containing `wpemojiSettings` | Delete the script block and the accompanying inline `<style>` |
| jQuery / jQuery Migrate | `<script src>` | `jquery.js`, `jquery-migrate.min.js` | Delete unless something in the surviving vanilla JS actually calls `jQuery`/`$` — check before removing |
| Unrendered shortcodes | visible page body text | literal bracket text: `[gallery]`, `[caption]`, `[embed]`, `[contact-form-7 ...]` | Replace with the real intended content or delete the line |
| Comment system remnants | body HTML | `id="comments"`, `class="comment-`, Disqus embed `<div id="disqus_thread">` | Remove entirely unless a comment system is intentionally being kept |
| Admin-ajax/REST calls in inline JS | inline `<script>` | `admin-ajax.php`, `/wp-json/` | Remove or rebuild against the site's real backend (see `contact-form-integration` for form submission patterns) |
| Favicon/manifest pointing at wp uploads | `<head>` | `wp-content/uploads/.../favicon` | Re-host the favicon under `/assets/img/` and rewrite the path |

**Other CMS/builder exports**: the specific fingerprints differ (Wix
injects `wixstatic.com` asset URLs and `data-testid` attributes;
Squarespace injects `squarespace-cdn.com` URLs and `sqs-block-*` classes)
but the principle is identical — grep the exported HTML for the
platform's own domain/asset-path pattern and its component wrapper
classes, and remove or rewrite every hit. WordPress is covered exhaustively
here because it's the most common source; treat the table above as a
template for auditing any other CMS export.

## 2. Worked folder-tree example

Given this nav:

```
Home
About
Services
  Web Design
  Hosting
Contact
(footer only) Privacy Policy
```

The folder tree:

```
/
  index.html                          ← Home
  about/
    index.html                        ← About
  services/
    index.html                        ← Services landing/overview
    web-design/
      index.html                      ← Services > Web Design
    hosting/
      index.html                      ← Services > Hosting
  contact/
    index.html                        ← Contact
  privacy-policy/
    index.html                        ← footer-only, still gets a folder
  assets/
    css/
      main.css
    js/
      main.js
    img/
      logo.svg
      ...
```

Every link in the rendered nav points at a clean URL matching this tree
exactly (`/services/web-design/`, not `/services-web-design.html` or
`/services/web-design.html`), so the folder structure and the site's
information architecture are the same tree.

## 3. Semantic HTML boilerplate

Starting point for every `index.html`:

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Page Title — Site Name</title>
  <meta name="description" content="Page-specific description.">
  <link rel="stylesheet" href="/assets/css/main.css">
</head>
<body>
  <a class="skip-link" href="#main-content">Skip to main content</a>

  <header class="site-header">
    <div class="container site-header__inner">
      <a class="site-header__logo" href="/">
        <img src="/assets/img/logo.svg" alt="Site Name logo" width="140" height="32">
      </a>

      <button type="button" class="nav-toggle" aria-expanded="false" aria-controls="primary-nav">
        <span class="visually-hidden">Menu</span>
        <span class="nav-toggle__icon" aria-hidden="true"></span>
      </button>

      <nav class="nav" id="primary-nav" aria-label="Primary">
        <ul class="nav__list">
          <li class="nav__item">
            <a class="nav__link" href="/" aria-current="page">Home</a>
          </li>
          <li class="nav__item">
            <a class="nav__link" href="/about/">About</a>
          </li>
          <li class="nav__item nav__item--has-submenu">
            <a class="nav__link" href="/services/">Services</a>
            <button type="button" class="nav__toggle" aria-expanded="false" aria-controls="services-submenu">
              <span class="visually-hidden">Show submenu for Services</span>
              <span class="nav__toggle-icon" aria-hidden="true"></span>
            </button>
            <ul class="nav__submenu" id="services-submenu">
              <li><a class="nav__link" href="/services/web-design/">Web Design</a></li>
              <li><a class="nav__link" href="/services/hosting/">Hosting</a></li>
            </ul>
          </li>
          <li class="nav__item">
            <a class="nav__link" href="/contact/">Contact</a>
          </li>
        </ul>
      </nav>
    </div>
  </header>

  <main id="main-content">
    <h1>Page Heading</h1>
    <!-- page content -->
  </main>

  <footer class="site-footer">
    <div class="container site-footer__inner">
      <div class="site-footer__col">
        <p class="site-footer__heading">Site Name</p>
        <p>One-line description or tagline.</p>
      </div>

      <nav class="site-footer__col" aria-label="Footer">
        <p class="site-footer__heading">Sitemap</p>
        <ul class="site-footer__list">
          <li><a href="/">Home</a></li>
          <li><a href="/about/">About</a></li>
          <li><a href="/services/">Services</a></li>
          <li><a href="/services/web-design/">Web Design</a></li>
          <li><a href="/services/hosting/">Hosting</a></li>
          <li><a href="/contact/">Contact</a></li>
        </ul>
      </nav>

      <div class="site-footer__col">
        <p class="site-footer__heading">Contact</p>
        <p><a href="mailto:hello@example.com">hello@example.com</a></p>
      </div>

      <div class="site-footer__col">
        <p class="site-footer__heading">Legal</p>
        <ul class="site-footer__list">
          <li><a href="/terms-of-use/">Terms of Use</a></li>
          <li><a href="/privacy-policy/">Privacy Policy</a></li>
        </ul>
      </div>
    </div>

    <div class="container site-footer__bottom">
      <p>&copy; 2026 Site Name. All rights reserved.</p>
    </div>
  </footer>

  <script src="/assets/js/main.js"></script>
</body>
</html>
```

`aria-current="page"` moves to whichever link matches the current page —
on `about/index.html` it sits on the About link instead of Home, and so
on for every page. Adjust the relative depth of `/assets/...` and nav
links as needed for nested pages (`/services/web-design/index.html`
still uses root-relative paths like `/assets/css/main.css`, so depth
never matters).

### Mobile menu and submenu JS

Add to `assets/js/main.js` — this is global, every-page behavior (every
page has this header), not the stack rule's vendored-utility exception
for optional bolt-ons, so it belongs in the site's own global script:

```javascript
/* ============================================
   Navigation: mobile menu toggle + dropdown disclosures
   ============================================ */
(function () {
  var navToggle = document.querySelector('.nav-toggle');
  var nav = document.getElementById('primary-nav');

  if (navToggle && nav) {
    navToggle.addEventListener('click', function () {
      var isOpen = navToggle.getAttribute('aria-expanded') === 'true';
      navToggle.setAttribute('aria-expanded', String(!isOpen));
      nav.classList.toggle('nav--open', !isOpen);
    });
  }

  document.querySelectorAll('.nav__toggle').forEach(function (toggle) {
    var submenu = document.getElementById(toggle.getAttribute('aria-controls'));

    toggle.addEventListener('click', function () {
      var isOpen = toggle.getAttribute('aria-expanded') === 'true';
      toggle.setAttribute('aria-expanded', String(!isOpen));
      if (submenu) submenu.classList.toggle('nav__submenu--open', !isOpen);
    });

    toggle.addEventListener('keydown', function (event) {
      if (event.key === 'Escape') {
        toggle.setAttribute('aria-expanded', 'false');
        if (submenu) submenu.classList.remove('nav__submenu--open');
        toggle.focus();
      }
    });
  });

  document.addEventListener('focusout', function (event) {
    document.querySelectorAll('.nav__toggle[aria-expanded="true"]').forEach(function (toggle) {
      var item = toggle.closest('.nav__item');
      if (item && !item.contains(event.relatedTarget)) {
        toggle.setAttribute('aria-expanded', 'false');
        var submenu = document.getElementById(toggle.getAttribute('aria-controls'));
        if (submenu) submenu.classList.remove('nav__submenu--open');
      }
    });
  });
})();
```

## 4. `main.css` skeleton

One file, five sections in this order, using CSS custom properties as
design tokens and a light BEM convention for components:

```css
/* ============================================
   1. TOKENS
   ============================================ */
:root {
  --color-primary: #1a5fb4;
  --color-text: #1a1a1a;
  --color-bg: #ffffff;
  --color-border: #dddddd;
  --color-bg-alt: #f5f5f5;
  --font-body: "Roboto", system-ui, -apple-system, "Segoe UI", Arial, sans-serif;
  --space-sm: 0.5rem;
  --space-md: 1rem;
  --space-lg: 2rem;
  --breakpoint-md: 768px;
  --header-height: 4rem;
}

/* ============================================
   2. RESET / BASE
   ============================================ */
*, *::before, *::after { box-sizing: border-box; }
body { margin: 0; font-family: var(--font-body); color: var(--color-text); background: var(--color-bg); }
img { max-width: 100%; display: block; }
h1, h2, h3, p { margin: 0 0 var(--space-md); }
a { color: var(--color-primary); }

/* ============================================
   3. LAYOUT
   ============================================ */
.container {
  max-width: 1100px;
  margin-inline: auto;
  padding-inline: var(--space-md);
}

/* ============================================
   4. COMPONENTS
   ============================================ */
.card {
  padding: var(--space-md);
  border: 1px solid #ddd;
  border-radius: 4px;
}
.card__title {
  font-size: 1.25rem;
  margin-bottom: var(--space-sm);
}
.card--featured {
  border-color: var(--color-primary);
}

.site-header {
  border-bottom: 1px solid var(--color-border);
  background: var(--color-bg);
}
.site-header__inner {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--space-md);
  padding-block: var(--space-sm);
}
.site-header__logo img {
  display: block;
  height: 2rem;
  width: auto;
}

/* Opt-in only - see the sticky-header guidance in SKILL.md before enabling.
   If applied, also set `html { scroll-padding-top: var(--header-height); }`
   so anchor links and the skip-link target land below the fixed bar. */
.site-header--sticky {
  position: sticky;
  top: 0;
  z-index: 10;
}

.nav-toggle {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 2.5rem;
  height: 2.5rem;
  border: 0;
  background: transparent;
  cursor: pointer;
}
.nav-toggle__icon,
.nav-toggle__icon::before,
.nav-toggle__icon::after {
  display: block;
  width: 1.5rem;
  height: 2px;
  background: var(--color-text);
}
.nav-toggle__icon::before,
.nav-toggle__icon::after {
  content: "";
  margin-block: 5px;
}

.nav { display: none; }
.nav--open { display: block; }
.nav__list {
  list-style: none;
  margin: 0;
  padding: 0;
}
.nav__link {
  display: block;
  padding: var(--space-sm) 0;
  text-decoration: none;
  color: var(--color-text);
}
.nav__link[aria-current="page"] {
  color: var(--color-primary);
  font-weight: 600;
}
.nav__item--has-submenu { position: relative; }
.nav__toggle {
  border: 0;
  background: transparent;
  cursor: pointer;
}
.nav__toggle-icon {
  display: inline-block;
  width: 0.4rem;
  height: 0.4rem;
  border-right: 2px solid var(--color-text);
  border-bottom: 2px solid var(--color-text);
  transform: rotate(45deg);
  margin-left: var(--space-sm);
}
.nav__submenu {
  list-style: none;
  margin: 0;
  padding: 0 0 0 var(--space-md);
  display: none;
}
.nav__submenu--open { display: block; }

.site-footer {
  background: var(--color-bg-alt);
  margin-top: var(--space-lg);
}
.site-footer__inner {
  display: grid;
  gap: var(--space-lg);
  padding-block: var(--space-lg);
  grid-template-columns: 1fr;
}
.site-footer__col { min-width: 0; }
.site-footer__heading {
  font-weight: 600;
  margin-bottom: var(--space-sm);
}
.site-footer__list {
  list-style: none;
  margin: 0;
  padding: 0;
}
.site-footer__bottom {
  border-top: 1px solid var(--color-border);
  padding-block: var(--space-sm);
  font-size: 0.875rem;
}

/* ============================================
   5. UTILITIES
   ============================================ */
.visually-hidden {
  position: absolute;
  width: 1px; height: 1px;
  overflow: hidden;
  clip: rect(0 0 0 0);
}
.text-center { text-align: center; }

.skip-link {
  position: absolute;
  top: -3rem;
  left: var(--space-sm);
  background: var(--color-primary);
  color: #fff;
  padding: var(--space-sm) var(--space-md);
  z-index: 100;
}
.skip-link:focus { top: var(--space-sm); }

/* Mobile-first: base styles above are the small-screen defaults */
@media (min-width: 768px) {
  .card { padding: var(--space-lg); }

  .nav-toggle { display: none; }
  .nav { display: block; }
  .nav__list { display: flex; gap: var(--space-lg); }
  .nav__toggle { display: none; }
  .nav__submenu {
    position: absolute;
    top: 100%;
    left: 0;
    min-width: 10rem;
    background: var(--color-bg);
    border: 1px solid var(--color-border);
    padding: var(--space-sm);
  }
  .nav__item--has-submenu:hover .nav__submenu,
  .nav__item--has-submenu:focus-within .nav__submenu { display: block; }

  .site-footer__inner { grid-template-columns: repeat(4, 1fr); }
}
```

`.skip-link` sits in Utilities rather than Components — it's a one-off
accessibility helper, the same category as `.visually-hidden`, not a
reusable named UI piece.

**Optional modern alternative**: teams comfortable with newer CSS can wrap
the same five sections in `@layer` (`@layer tokens, base, layout,
components, utilities;` declared once, then each section body wrapped in
its matching `@layer name { ... }`). This makes the cascade order
explicit independent of source order, but it's optional — the plain
top-to-bottom ordering above is sufficient and simpler for a small site.

**When (and only when) to split a second stylesheet**: if one page has a
large, genuinely page-specific block of styles (e.g. a pricing table used
nowhere else) and `main.css` has grown to the point a human can no longer
scan it comfortably, add a second file (`/assets/css/pricing.css`) linked
only on that page — and say so explicitly, since the default for this
skill is one stylesheet.

## 5. Typography: choosing and loading fonts

**Default rule**: Roboto, sans-serif, self-hosted. Sans-serif renders
more reliably at small UI sizes and low DPI, reads as neutral/modern,
and Roboto specifically is Android's system font — one of the most
battle-tested sans-serifs on the web. Serif is a legitimate choice for
editorial, legal, heritage, or luxury-positioned brands, but it's a
*deliberate* call, never the silent default — don't let a build drift
into "elegant serif" just because it looks more designed.

**When to deviate**: the client's brief/style guide already specifies a
brand font — use it (checking license/webfont availability first) — or
the brief specifically wants a serif/editorial feel. Either way, call
the deviation out explicitly rather than silently swapping the default.

**Self-hosting Roboto (or any chosen font)**: download the WOFF2 weights
actually needed (typically 400 and 600/700) from Google Fonts' "Download
family" or a self-hosting generator, place them under `/assets/fonts/`,
and declare them in the Tokens section of `main.css`, before `:root`:

```css
@font-face {
  font-family: "Roboto";
  src: url("/assets/fonts/roboto-400.woff2") format("woff2");
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}
@font-face {
  font-family: "Roboto";
  src: url("/assets/fonts/roboto-700.woff2") format("woff2");
  font-weight: 700;
  font-style: normal;
  font-display: swap;
}
```

Self-hosting beats a Google Fonts CDN `<link>` here: cross-site font
caching was removed from every major browser years ago, so the CDN has
no shared-cache advantage left, self-hosted WOFF2 on the same origin
(any static host here runs HTTP/2) matches or beats it with no extra
DNS/TLS round trip, and it avoids sending every visitor's IP to Google
at page load without consent — the same issue behind GDPR fines in the
EU, worth avoiding given this plugin's South-Africa-first legal skills
(`terms-of-use-website`, POPIA). Self-hosting is also just static files,
so it fits the stack rule with zero build tooling.

`--font-body` always carries a full fallback stack, never a bare font
name:

```css
--font-body: "Roboto", system-ui, -apple-system, "Segoe UI", Arial, sans-serif;
```

`system-ui, sans-serif` alone (no web font at all) remains an acceptable
zero-webfont option for a very simple/low-budget build — call that
choice out explicitly if taken, same as any other deviation from the
default.

**Loading performance**: `font-display: swap` on every `@font-face`
(shown above) so text is visible in a fallback font immediately, never
invisible while the web font loads. No `<link>` to
`fonts.googleapis.com`/`fonts.gstatic.com` at all when self-hosting —
there's nothing to preconnect to. If a client-supplied brand font can
only be loaded from a third-party CDN, add
`<link rel="preconnect" href="...">` for that origin in `<head>`.

**Weight/file discipline**: two weights (400 regular + 600/700 bold)
covers a typical marketing site — build hierarchy with size and weight,
not extra font files. Don't pull in italic/light/black cuts unless the
design genuinely uses them; each extra weight is another render-blocking
file.

**Accessibility**: keep body text at a `1rem` (≈16px) base with
line-height around 1.5 for body copy, and don't rely on font-weight
alone to convey meaning that also needs color/underline to register for
users who can't perceive weight differences.

## 6. Monochrome style: locked light/dark token system

**When this applies**: only on a confirmed strict-monochrome choice from
`project-discovery` (category D). This section **replaces**, not
supplements, the example `--color-*` tokens in the `main.css` skeleton's
Tokens section above — don't blend the two.

**The locked token set** (`:root`, light values):

```css
:root {
  --gray-950: #050505;
  --gray-900: #0c0c0c;
  --gray-800: #262626;
  --gray-700: #404040;
  --gray-600: #525252;
  --gray-500: #6b6b6b;
  --gray-400: #a3a3a3;
  --gray-300: #d4d4d4;
  --gray-200: #e5e5e5;
  --gray-100: #f0f0f0;
  --gray-50:  #f7f7f7;

  --background: #ffffff;
  --background-alt: var(--gray-50);
  --foreground: var(--gray-900);
  --muted: var(--gray-50);
  --muted-foreground: var(--gray-500);
  --border: var(--gray-200);
  --border-strong: var(--gray-300);
  --primary: var(--gray-900);
  --primary-dark: var(--gray-950);
  --primary-foreground: #ffffff;

  --destructive: #b91c1c;
  --destructive-bg: rgba(220, 38, 38, 0.1);
  --destructive-foreground: #ffffff;
}
```

**Dark-mode override**, keyed off a `data-theme="dark"` attribute on
`<html>` (an attribute, not a `.dark` class, to avoid colliding with
this skill's own BEM component-class convention):

```css
:root[data-theme="dark"] {
  --dark-elevated: #1c1c1c;
  --background: var(--gray-950);
  --background-alt: var(--dark-elevated);
  --foreground: var(--gray-50);
  --muted: var(--dark-elevated);
  --muted-foreground: var(--gray-400);
  --border: var(--gray-800);
  --border-strong: var(--gray-700);
  --primary: var(--gray-50);
  --primary-dark: #ffffff;
  --primary-foreground: var(--gray-950);

  --destructive: #f87171;
  --destructive-bg: rgba(248, 113, 113, 0.14);
  --destructive-foreground: var(--gray-950);
}

@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) {
    /* same custom properties as the [data-theme="dark"] block above */
    --dark-elevated: #1c1c1c;
    --background: var(--gray-950);
    --background-alt: var(--dark-elevated);
    --foreground: var(--gray-50);
    --muted: var(--dark-elevated);
    --muted-foreground: var(--gray-400);
    --border: var(--gray-800);
    --border-strong: var(--gray-700);
    --primary: var(--gray-50);
    --primary-dark: #ffffff;
    --primary-foreground: var(--gray-950);
    --destructive: #f87171;
    --destructive-bg: rgba(248, 113, 113, 0.14);
    --destructive-foreground: var(--gray-950);
  }
}
```

The pattern: automatic dark mode comes from the `prefers-color-scheme`
media query; an explicit `data-theme="light"` or `data-theme="dark"`
attribute (set by the optional toggle below) always wins over the media
query in both directions, since `:not([data-theme="light"])` only
suppresses the media block when light mode was explicitly forced, and
the plain `[data-theme="dark"]` rule already beats the media query on
specificity when dark was explicitly forced.

**No-flash theme script**: a small inline `<script>` in `<head>`,
*before* `main.css` loads, so there's no flash of the wrong theme on
load — vanilla JS, no framework required:

```html
<script>
  (function () {
    var stored = localStorage.getItem('theme');
    if (stored) document.documentElement.setAttribute('data-theme', stored);
  })();
</script>
```

Note it only sets the attribute when a preference was explicitly
**stored** — with no stored preference, the CSS media query alone
handles theming and the attribute stays unset, matching the "media
query is the default, attribute is the override" rule above.

**Optional manual toggle** (documented, not mandatory — default builds
skip this; only add it when a client specifically asks for manual
control): a header button that flips the stored preference and applies
it immediately.

```js
var toggle = document.querySelector('.theme-toggle');
if (toggle) {
  toggle.addEventListener('click', function () {
    var current = document.documentElement.getAttribute('data-theme')
      || (matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light');
    var next = current === 'dark' ? 'light' : 'dark';
    document.documentElement.setAttribute('data-theme', next);
    localStorage.setItem('theme', next);
  });
}
```

**Section alternation**: `.section` / `.section-alt` swap
`--background-alt` in for visual rhythm down the page:

```css
.section { padding: var(--space-lg) 0; }
.section-alt { background: var(--background-alt); }
```

In dark mode, the "alt" surface should step *up* the gray scale (e.g.
`--gray-800` rather than the default `--dark-elevated`) instead of just
swapping light/dark the way the base background does — that's the
detail that keeps dark mode from reading as a flat inversion:

```css
:root[data-theme="dark"] .section-alt {
  --background-alt: var(--gray-800);
}
@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) .section-alt {
    --background-alt: var(--gray-800);
  }
}
```

**Button component** (new — the skeleton's Components section has no
button today): built entirely from the grayscale tokens above, no
accent color on any CTA.

```css
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--space-sm);
  padding: 0.75rem 1.5rem;
  border-radius: 4px;
  font-weight: 600;
  font-family: var(--font-body);
  border: 1px solid transparent;
  cursor: pointer;
  text-decoration: none;
}
.btn-primary {
  background: var(--primary);
  color: var(--primary-foreground);
}
.btn-primary:hover { background: var(--primary-dark); }
.btn-outline {
  background: transparent;
  border-color: var(--border-strong);
  color: var(--foreground);
}
.btn-outline:hover { border-color: var(--primary); }
```

**What's deliberately excluded**: the reference implementation this
spec is drawn from also defines two accent tokens (an indigo and a
teal) but never applies them anywhere on its public pages — confirmed
by inspecting its compiled CSS. They're left out of this standard on
purpose: the locked palette stays strictly grayscale plus the one
destructive-red exception. Don't add an accent color under this spec
unless the client explicitly asks for a deviation — and if they do,
call it out explicitly as a deviation rather than quietly reintroducing
color.

**Accessibility note**: verify `--muted-foreground` on `--background`
and on `--background-alt` clears WCAG AA contrast in *both* themes
before shipping — a gray step that passes in light mode can fail in
dark mode even though the palette looks symmetric on paper.

## 7. Full verification checklist

Run all of these before calling a build or WordPress migration done:

```bash
grep -ril "wordpress"        . --include=*.html
grep -ril "wp-content"       . --include=*.html
grep -ril "wp-includes"      . --include=*.html
grep -ril "wp-json"          . --include=*.html
grep -ril "wp-emoji"         . --include=*.html
grep -ril "elementor-"       . --include=*.html
grep -ril "wp-block-"        . --include=*.html
grep -ril "woocommerce-"     . --include=*.html
grep -ril "shortcode\|\[gallery\]\|\[contact-form-7" . --include=*.html
```

All of the above should return no matches. Then:

- Open the site with dev tools' Network tab and browse every nav path —
  zero 404s.
- Confirm every link in the rendered nav resolves to a real
  `folder/index.html` matching the folder-structure standard.
- Confirm the site works when served as plain static files (no server
  config beyond a static file server — no PHP, no server-side includes).
- View-source every page's `<head>` and confirm no generator meta tag,
  pingback link, REST API discovery link, or oEmbed link remains.

Then, for the header/nav/footer:

- Tab through every page from a blank focus state: the skip link is the
  first thing focused and, when activated, moves focus into `<main>`;
  the mobile toggle is reachable and its `aria-expanded` value is
  announced correctly (check the browser dev tools' accessibility tree
  if unsure); the Services submenu toggle opens with Enter/Space, closes
  with Escape, and returns focus to the toggle.
- Resize to a mobile width and confirm the Services submenu behaves as
  an in-place accordion inside the open mobile panel, not a hover flyout
  (there's no hover on touch).
- Confirm exactly one nav link per page carries `aria-current="page"`,
  and it matches that page.

Then, for typography:

- Font files are self-hosted WOFF2 under `/assets/fonts/` — the Network
  tab shows no request to `fonts.googleapis.com`/`fonts.gstatic.com`,
  unless a documented brand-font exception applies.
- Every `@font-face` rule sets `font-display: swap`.
- `--font-body` carries a full fallback stack, never a bare font name.
- No more than two font weights are loaded unless the brief specifically
  calls for more.
- If a serif or other non-default font was used, it's because the brief
  called for it — not a silent default.

If monochrome was chosen, also check:

- Tokens match the locked scale/semantic set exactly — no ad hoc hex
  values, no accent color beyond the destructive red.
- Reloading with a previously-saved theme preference shows no
  flash-of-wrong-theme (the inline script runs before first paint).
- Both `[data-theme="dark"]` and the `prefers-color-scheme: dark` media
  fallback produce the same result when no explicit choice is stored.
- `--muted-foreground` passes AA contrast against `--background` and
  `--background-alt` in both themes.
