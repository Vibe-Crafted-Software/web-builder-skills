---
name: website-seo
description: This skill should be used when the user asks to "improve SEO", "get a website found on Google", "set up Search Console or Bing Webmaster Tools", "write a sitemap or robots.txt", "add structured data / JSON-LD", "set up IndexNow", "plan a content or keyword strategy", or otherwise needs a website's technical or content SEO handled, one-off setup or ongoing maintenance.
version: 1.0.0
---

# Website SEO Playbook

Provide the one-off technical/content SEO foundation for a website launch,
then the ongoing maintenance cadence to keep it working. Covers organic,
free discoverability only — not paid search (Google Ads, Microsoft Ads),
not broader brand/social strategy, not PR.

## When this applies

Use when helping launch a new site's SEO foundation, auditing an existing
site's SEO gaps, or handling the recurring weekly/monthly/quarterly SEO
maintenance for a site already live.

## Core one-off checklist (do in order, once)

1. DNS/hosting sanity: HTTPS-only, one canonical host (redirect the other),
   confirm compression/HTTP2.
2. `robots.txt` + `sitemap.xml` at the site root, sitemap listing every
   canonical indexable page.
3. Per-page on-page basics: unique title (~50-60 chars) and meta
   description (~140-160 chars), one `<h1>`, a self-referencing canonical
   tag on *every* page, descriptive image alt text, the full Open Graph
   set (title/description/type/url/image) and Twitter Card tags on every
   page — not just the homepage.
4. JSON-LD structured data (not Microdata): Organization + WebSite on the
   homepage, BreadcrumbList site-wide, Article/BlogPosting per post. The
   nav hierarchy `website-build-standards` defines (e.g. Services > Web
   Design/Hosting) is the same structure a BreadcrumbList encodes, and
   the nav/footer link set should match `sitemap.xml`'s page set.
5. Google Search Console: verify via DNS TXT record (most durable),
   submit the sitemap.
6. Bing Webmaster Tools: import from an already-verified GSC property, or
   verify separately; submit the same sitemap.
7. IndexNow: a key file + a post-deploy POST of changed URLs to
   `api.indexnow.org` — covers Bing/Yandex/Naver/Seznam, **not Google**.
8. Core Web Vitals hygiene (modern image formats, explicit image
   dimensions, deferred third-party scripts, font-display swap).
9. Analytics: default to a privacy-first tool (no consent banner needed);
   upgrade to GA4 only once deep funnel/ads attribution is genuinely
   needed.
10. AI/answer-engine visibility rides on the same fundamentals as 3-4
    above — no separate workstream needed.

## Corrections to stale/common advice — apply these, don't skip them

- **FAQPage schema** no longer earns a Google rich result (deprecated in
  Search in 2026) — don't add it chasing that benefit.
- **Google's Indexing API** is officially only for `JobPosting`/
  `BroadcastEvent` content — never build it for ordinary marketing/blog
  pages; use sitemap + manual "Request Indexing" instead.
- **IndexNow does not reach Google** — it covers Bing/Yandex/Naver/Seznam
  only.
- **`llms.txt` is not an adopted standard** — Google has said directly it
  doesn't use it. Fine as a cheap, optional hedge; never present it as
  required infrastructure like `robots.txt`.
- **Google Business Profile generally doesn't apply** to a purely online
  B2B business with no physical/in-person customer interaction — don't
  set one up; signal region/vertical instead via plain-text content and
  `areaServed`/`knowsAbout` in the Organization schema.

## Content strategy essentials

If a `PROJECT_BRIEF.md` exists (see the `project-discovery` skill),
start from its Research notes' seed-keyword list rather than re-deriving
one from scratch; everything below is what happens after that seed
list, not a duplicate of gathering it.

Seed keyword research from the business itself (not a paid tool), expand
with free tools (Google Keyword Planner, Google Trends, Bing Webmaster's
Keyword Research tool), segment by funnel intent, and prioritize
long-tail/low-competition terms while the domain is new. Structure
content as pillar pages + linked cluster pages. A sustainable cadence
(2-4 posts/month) beats a burst-then-silence pattern.

## Off-page / authority building

Realistic tactics for a brand-new domain: relevant directories (G2,
Capterra, Clutch, etc. depending on business type), a populated LinkedIn
company page, genuine guest content on relevant editorial publications,
broken-link building. Hard avoid: paid links without `rel=sponsored`/
`nofollow`, mass guest-posting, PBNs, link farms. If the business has a
reseller/partner channel, partner directory listings and natural
cross-linking with resellers are a legitimate, disclosed advantage —
distinct from generic backlink tactics.

## Ongoing cadence

- **Weekly** (~15-30 min): scan Search Console's Coverage report for new
  errors, check Core Web Vitals flags, glance at tracked keyword
  positions for sudden drops.
- **Monthly**: full GSC performance review, broken-link crawl, re-submit
  sitemap after new pages, refresh 1-2 aging pages.
- **Quarterly**: deep technical audit, manual-action check, competitor
  gap analysis, backlink profile review, refresh the keyword list from
  real GSC query data.

## Honest expectations

A brand-new domain: months 1-3 is foundation/minimal traffic, 3-6 first
long-tail movement, 6-12 consistent growth (typical ROI break-even),
9-12+ for competitive terms. Set this timeline explicitly rather than
overpromise.

## Additional resources

For the complete playbook — full statutory/practice detail, exact code
snippets (robots.txt, sitemap.xml, JSON-LD blocks, IndexNow POST body),
the full off-page-tactics list, and the full verification checklist —
consult:

- **`references/website-seo.md`** — the complete playbook

Automated verification of these tags (meta description length, canonical
self-reference, full OG/Twitter set, JSON-LD validity) is covered by the
`website-testing` skill's page-health checks — use that skill to add or
run the Playwright suite rather than writing test assertions from scratch
here.
