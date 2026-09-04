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
  --font-body: system-ui, sans-serif;
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

## 5. Full verification checklist

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
