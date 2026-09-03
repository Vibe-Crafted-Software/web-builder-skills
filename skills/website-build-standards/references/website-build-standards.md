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
  <header>
    <nav>
      <a href="/">Home</a>
      <a href="/about/">About</a>
      <a href="/services/">Services</a>
      <a href="/contact/">Contact</a>
    </nav>
  </header>

  <main>
    <h1>Page Heading</h1>
    <!-- page content -->
  </main>

  <footer>
    <a href="/privacy-policy/">Privacy Policy</a>
  </footer>

  <script src="/assets/js/main.js"></script>
</body>
</html>
```

Adjust the relative depth of `/assets/...` and nav links as needed for
nested pages (`/services/web-design/index.html` still uses root-relative
paths like `/assets/css/main.css`, so depth never matters).

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
  --font-body: system-ui, sans-serif;
  --space-sm: 0.5rem;
  --space-md: 1rem;
  --space-lg: 2rem;
  --breakpoint-md: 768px;
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

/* Mobile-first: base styles above are the small-screen defaults */
@media (min-width: 768px) {
  .card { padding: var(--space-lg); }
}
```

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
