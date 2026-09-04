---
name: website-testing
description: This skill should be used when the user asks to "add tests", "test the website", "write a Playwright test", "check for broken links", "test the contact form", "run an accessibility check", or otherwise needs automated page-health, link, form, accessibility, visual, or performance checks set up or run against a site built with the website-build-standards stack.
version: 1.0.0
---

# Website Testing Playbook

Add or extend an automated, dev-only verification layer — Playwright plus
`@axe-core/playwright` — on top of a site built per `website-build-standards`.
This skill doesn't decide what a page's meta tags, copy, or structured data
should say, and it doesn't build the contact form; it only tests those
things. Point here whenever another skill says to "run the test suite."

## Scope

- Deciding meta description/title/OG/JSON-LD *content* or SEO strategy →
  `website-seo`. This skill only asserts those tags exist and are
  well-formed, per the spec `website-seo` defines.
- Building or wiring the contact form to a relay → `contact-form-integration`.
  This skill only automates testing an already-integrated form, and reuses
  that skill's real-browser gotcha rather than re-deriving it.
- Homepage copy/conversion structure → `website-sales-tool`.
- Stack, folder layout, and the zero-build-step rule → `website-build-standards`
  — this skill's own tooling must respect that rule (see Setup below).
- Legal terms content → `terms-of-use-website` / `terms-of-use-software`.
- Which old URLs redirect where, and the SEO strategy behind a site
  relaunch → `website-seo`; the CloudFront/KVS mechanism that actually
  serves those 301s → `website-deployment`. This skill only verifies the
  redirect map behaves as configured, once it exists.

## Setup: dev-only tooling, never deployed

- Playwright (`@playwright/test`) and `@axe-core/playwright` are
  `devDependencies` installed via `npm`. A `package.json`, `node_modules/`,
  `playwright.config.js`, and a `tests/` folder live at the project root,
  sibling to `assets/` — but none of it ships to the static host.
- This is not a build step: nothing here bundles, transpiles, or minifies
  the HTML/CSS/JS that gets deployed. It's a parallel verification harness
  that runs on a developer machine or in CI and produces no output that
  ships.
- Before relying on this suite, confirm — by reading the actual deploy
  mechanism, not by assuming — that it excludes `tests/`, `package.json`,
  `package-lock.json`, `playwright.config.js`, and `node_modules/`.
- `playwright.config.js` basics: `testDir: './tests'`, a `webServer` block
  that serves the static folder locally (e.g. `npx serve .`), and a
  `baseURL` pointing at it.

## Page-health checks

One spec crawls every page, driven from `sitemap.xml` so new pages are
covered automatically rather than hardcoded into the test. Per page, assert:
a title and meta description are present (description within a reasonable
length range), the canonical tag is present and self-referencing, the full
Open Graph set (title/description/type/url/image) is present, a Twitter
Card is present, and valid JSON-LD is present where expected.

`website-seo` owns what these tags should *say*; this only verifies they
exist and are well-formed. Don't re-derive the content guidance here.

For a multi-language site (`project-discovery`'s language checklist),
also assert every page's `hreflang` tags are present, including an
`x-default`. The reference spec checks presence only, not full
reciprocity (every language variant linking back to every other) —
extend it if the site's language setup is complex enough to warrant
that deeper check.

## Broken-link checking

Crawl internal links starting from the homepage (or iterate `sitemap.xml`),
respecting the folder-per-page/clean-URL convention from
`website-build-standards` — a link to `/services/hosting/` must resolve on
the local static server. Also check that in-page anchors resolve to a real
`id`, and that referenced assets (css/js/img) return 200.

Check external links separately and treat them as advisory, not
CI-blocking — a third party's rate limiting or a transient outage isn't a
defect in this site.

## Redirect verification (site relaunch)

When a rebuild relaunches over an existing, already-ranking site,
`website-seo`'s migration checklist requires crawling the *entire*
old-URL inventory post-cutover to confirm every redirect actually
works — spot-checking a handful by hand doesn't scale to a real
legacy-site redirect map (`website-deployment`'s `redirects.json`,
potentially hundreds of entries). Add a dedicated spec that reads that
same `redirects.json` and, for every old path, asserts a single-hop
301 to the exact expected destination — not just "redirects somewhere."
This only applies when a redirect map exists; skip it entirely for a
brand-new site with no prior URLs to protect.

## Form testing

Reuses `contact-form-integration`'s existing gotcha rather than restating
it: honeypot fields and browser autofill can silently break a real
submission even when a scripted `fetch`/`curl` call succeeds. Concrete
pattern: use a real browser context, fill only the visible labeled fields
(`getByLabel`), leave the honeypot field untouched, submit, and assert the
success message appears.

A mocked-network variant (`page.route`) is fine for cheap, fast CI runs,
but it doesn't catch the autofill failure mode itself — a periodic real,
unmocked submission stays necessary, exactly as the sibling skill already
warns.

## Accessibility testing

Run `@axe-core/playwright` against every page, targeting WCAG 2.1 AA plus
the WCAG 2.2 AA additions (target size, focus-not-obscured, accessible
authentication), and assert zero critical/serious violations. This is
genuinely new ground for
this repo, so treat it as introductory/smoke-level: it catches missing alt
text, contrast failures, unlabeled form fields, and landmark issues — it is
not a substitute for manual keyboard and screen-reader testing, and a clean
run does not mean the site is fully accessible.

`website-build-standards`'s header/nav/footer patterns give this suite
concrete, non-generic assertions beyond a blanket axe scan: the skip
link's target `id` exists and receives focus, the mobile toggle's
`aria-expanded` value flips on click, and exactly one nav link per page
carries `aria-current="page"`.

When adding this to an existing, previously-untested site, treat the first
run as an audit — triage the findings before treating any of them as hard
blockers.

## Visual / responsive smoke testing

Default to a cheap, low-maintenance check: 2-3 viewport widths per page,
asserting there's no horizontal overflow (compare `scrollWidth` to the
viewport width). Full pixel screenshot-diffing is available as an optional
upgrade, but it's high-maintenance and prone to false positives across
machines/fonts — don't make it the default.

**Exception**: for a strict-monochrome site (`project-discovery`'s VCS
style question), that false-positive risk mostly disappears — any stray
color on a grayscale-only design is an unambiguous, high-signal bug, not
a font-rendering artifact. Screenshot-diffing (or a cheaper computed-style
color-audit assertion) is a much better cost/benefit trade specifically
for that case.

For a multi-language site with an RTL language, run the same overflow
check against an RTL page too — RTL layout bugs are exactly the kind of
thing this cheap check catches almost for free.

## Performance

Lighthouse/PageSpeed Insights is `website-seo`'s tool for Core Web Vitals —
don't restate its thresholds here. Optionally, add Lighthouse CI as a
devDependency to automate that same check against the local static server.

## Running tests & CI

Run `npm test` locally before every deploy. Keep CI optional and
lightweight: add a job to an existing pipeline if one exists; otherwise,
for a static-site shop with no CI yet, "run locally before deploy" is a
perfectly reasonable stopping point — don't build CI infrastructure that
wasn't asked for.

## Gotchas

- Deploy-exclusion of the test tooling is not automatic — verify the
  actual deploy config, don't assume it.
- The contact-form gotcha belongs to `contact-form-integration`; link to
  it rather than restating it, and remember a mocked test alone won't
  catch the autofill failure mode.
- External-link checks and full-screenshot visual diffing are both
  inherently flaky — keep them advisory, never a hard CI gate.
- Automated accessibility tools (axe included) catch only a minority of
  real WCAG issues — don't oversell a clean run.
- Adding this suite to a previously-untested site will surface real,
  pre-existing failures immediately. Treat the first run as an audit, not
  a broken build.

## Verification checklist

- A clean-clone `npm install` only adds devDependencies.
- The deploy step is confirmed to exclude `tests/`, `package.json`,
  `package-lock.json`, `playwright.config.js`, and `node_modules/`.
- The full suite passes locally before pushing.
- Every page listed in `sitemap.xml` is covered by the page-health spec.
- The contact form has also been verified once by hand in a real browser,
  per `contact-form-integration`'s gotcha.
- Any suppressed accessibility rule has a documented reason.
- If this is a site relaunch with a redirect map, every entry in
  `redirects.json` is covered by the redirect-verification spec and
  passes — not just a spot-checked sample.
- CI, if present, blocks on page-health/internal-link/form/accessibility
  failures but not on flaky external-link or visual-diff checks.

## Additional resources

For the complete playbook — full `package.json`, `playwright.config.js`,
every spec file in full, an optional Lighthouse CI config, and an optional
CI workflow — consult:

- **`references/website-testing.md`** — the complete playbook
