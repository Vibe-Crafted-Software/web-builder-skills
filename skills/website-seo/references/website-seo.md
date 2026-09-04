# Website marketing & SEO playbook

How to get a website found on Google, Bing, and AI answer engines — a
one-off technical/content foundation, then an ongoing maintenance
cadence. This doc is for whoever owns a site's launch and its ongoing
upkeep (a developer, an AI agent, a marketer with some technical access).
It assumes a live site where you control the HTML `<head>`, DNS, and the
deploy process, but no CMS, ad budget, or SEO tooling beyond what's free.

**Scope**: organic/free discoverability only — technical SEO, search
engine setup, content strategy, backlinks, analytics. Paid search
(Google Ads, Microsoft Ads), broader brand/social-media strategy, and PR
beyond the backlink-adjacent mentions in Part 3 are out of scope; see
the closing note.

Everything below is `{{PLACEHOLDER}}`-tokenized so it can be dropped
into any project. A few points correct commonly-repeated SEO advice that
went stale in 2026 — those are called out explicitly as **Correction:**
rather than stated as plain fact, so you don't have to take it on faith.

---

## Part 1 — One-off technical foundation

Work through this in order once, at launch (or now, if the site's
already live and this wasn't done at launch). If this is a rebuild of
an existing site that already has real rankings/traffic, go to
**Part 7** instead — it replaces this Part's assumptions with a
migration-specific process; don't run this checklist blind on top of an
already-indexed site.

### 1. DNS / hosting sanity

- Serve **HTTPS only** — redirect any HTTP request to HTTPS.
- Pick **one canonical host** — `{{DOMAIN}}` or `www.{{DOMAIN}}`, not
  both — and 301-redirect the other to it. Two hosts both serving 200s
  for the same content splits ranking signals and duplicates the site in
  search engines' eyes. (A CDN-level redirect function, e.g. a CloudFront
  Function on the non-canonical host, is a clean way to do this without
  touching the app itself.)
- Confirm your CDN/host is actually serving compression (Brotli or
  gzip) and HTTP/2 — usually on by default, worth a one-time check with
  browser devtools' Network tab (Content-Encoding, Protocol columns).

### 2. `robots.txt` and `sitemap.xml`

`robots.txt` at `{{SITE_URL}}/robots.txt`:

```
User-agent: *
Allow: /

Sitemap: {{SITE_URL}}/sitemap.xml
```

`sitemap.xml` at `{{SITE_URL}}/sitemap.xml` — list every canonical,
indexable page (skip error pages like a 404 page):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>{{SITE_URL}}/</loc>
  </url>
  <url>
    <loc>{{SITE_URL}}/{{PAGE}}.html</loc>
  </url>
</urlset>
```

Keep this updated as pages are added — a page that exists but isn't in
the sitemap is easy for a crawler to miss. For a small static site, hand
maintenance is fine; there's no need to generate this dynamically.

### 3. On-page basics, per page

- **Title tag** — unique per page, ~50-60 characters (Google truncates
  by pixel width, not a hard character count; 60 is a safe proxy).
  Primary topic near the front, brand name at the end:
  `{{PAGE_TOPIC}} | {{BRAND_NAME}}`. Don't stack multiple pipe-separated
  segments.
- **Meta description** — unique per page, ~140-160 characters,
  front-loaded with the actual value proposition. Treat this as a
  click-through aid, not a ranking factor — Google frequently rewrites
  descriptions that don't match the searcher's query, so a good one
  still helps but isn't guaranteed to be shown verbatim.
- **Exactly one `<h1>` per page**, stating the page's topic, with
  logically nested `<h2>`/`<h3>` beneath it.
- **A self-referencing canonical tag on every page** —
  `<link rel="canonical" href="{{SITE_URL}}/{{THIS_PAGE}}" />` — not
  just on pages with obvious duplicates. This is cheap insurance against
  `?utm=` tracking params, trailing-slash variants, or a www/non-www
  split all being read as separate pages.
- **Descriptive image `alt` text** — not keyword-stuffed; empty
  `alt=""` for purely decorative images. This also matters for
  accessibility, not just SEO.
- **The full Open Graph tag set on every page** — not just the
  homepage:
  ```html
  <meta property="og:title" content="{{PAGE_TITLE}}" />
  <meta property="og:description" content="{{PAGE_DESCRIPTION}}" />
  <meta property="og:type" content="website" />
  <meta property="og:url" content="{{SITE_URL}}/{{THIS_PAGE}}" />
  <meta property="og:image" content="{{SITE_URL}}/{{OG_IMAGE}}" />
  ```
  A page with only `og:image` set and no `og:title`/`og:description`/
  `og:type`/`og:url` of its own will fall back to whatever a given
  crawler decides to substitute when someone shares that page — often
  the homepage's values, sometimes nothing. Every indexable page needs
  its own complete set.
- **Twitter Card tags**, alongside Open Graph:
  ```html
  <meta name="twitter:card" content="summary_large_image" />
  <meta name="twitter:title" content="{{PAGE_TITLE}}" />
  <meta name="twitter:description" content="{{PAGE_DESCRIPTION}}" />
  <meta name="twitter:image" content="{{SITE_URL}}/{{OG_IMAGE}}" />
  ```
- **Mobile-friendly, responsive design** — Google indexes mobile-first
  for effectively everything now. A single responsive layout (no
  separate mobile URLs) sidesteps a whole class of `rel=alternate`/
  mobile-pairing complexity — recommended over any device-specific URL
  scheme.

### 4. Structured data (JSON-LD)

Use JSON-LD (`<script type="application/ld+json">`), not
Microdata/RDFa — it's the format Google recommends, and it keeps
structured data separate from visible markup, which matters most on a
site with no templating engine.

Recommended, in priority order:

- **Organization** — on the homepage (or identically repeated site-wide):
  ```json
  {
    "@context": "https://schema.org",
    "@type": "Organization",
    "@id": "{{SITE_URL}}/#organization",
    "name": "{{BRAND_NAME}}",
    "url": "{{SITE_URL}}",
    "logo": "{{SITE_URL}}/{{LOGO_PATH}}",
    "sameAs": ["{{SOCIAL_PROFILE_URL}}"],
    "contactPoint": {
      "@type": "ContactPoint",
      "email": "{{CONTACT_EMAIL}}",
      "contactType": "customer support"
    }
  }
  ```
- **WebSite** — on the homepage, referencing the Organization as
  `publisher`. Add a `SearchAction` only if the site has real internal
  search.
- **BreadcrumbList** — site-wide, on every non-home page, mirroring the
  actual visible nav hierarchy.
- **Article** / **BlogPosting** — one per blog post: `headline`,
  `datePublished`, `dateModified`, `author`, `image`, `publisher`.
- **SoftwareApplication**, **Product**, or **Service** — only if the
  site markets a specific named product (with a price/download) or a
  consulting/development service; skip this if it doesn't map cleanly.

Validate every block with
[Google's Rich Results Test](https://search.google.com/test/rich-results)
and [validator.schema.org](https://validator.schema.org/) before
publishing.

**Correction: don't add FAQPage schema expecting an SEO rich-result
benefit.** Google deprecated FAQ rich results in Search in 2026 —
they no longer appear in results, and Search Console's report for them
is gone too. FAQPage markup isn't harmful to leave in place if you
already have it, but don't add it now chasing a rich-result snippet that
no longer exists; if you use it at all, treat it as a minor AI-answer-
engine hedge (see below), not a ranking tactic.

### 5. Google Search Console

- **Verify via a DNS TXT record** at the domain root — it's the most
  durable method: it survives redeploys, `<head>` template rewrites, and
  full-bucket-sync deploys that might otherwise clobber an HTML
  verification file or a meta tag. Add the TXT record Search Console
  gives you at your DNS provider.
- **HTML-file verification** is a documented fallback (upload the file
  Google provides to the site root) — but if your deploy process does a
  destructive full sync (e.g. `aws s3 sync --delete`), make sure that
  file is either excluded from deletion or re-uploaded every deploy, or
  it'll silently break verification.
- **Submit your sitemap** (`{{SITE_URL}}/sitemap.xml`) via the Sitemaps
  report. Re-submission after every deploy isn't necessary — Google
  re-crawls a known sitemap periodically — but do re-submit after adding
  a meaningful batch of new pages.
- Check the **Coverage/Indexing report** a week or two after launch (and
  periodically after — see Part 5) for crawl errors, and use
  **URL Inspection → Request Indexing** on the homepage and a couple of
  key pages if they haven't been picked up yet.

**Correction: don't build an Indexing API integration for this.**
Google's Indexing API is officially supported only for pages marked up
as `JobPosting` or `BroadcastEvent`/livestream content — not ordinary
marketing pages or blog posts. Using it for other content is something
Google has said it detects and disregards. Rely on the sitemap +
internal linking + manual Request Indexing instead.

### 6. Bing Webmaster Tools

- Fastest setup: **import directly from an already-verified Google
  Search Console property** — a one-click option that skips separate
  verification.
- Otherwise, verify via meta tag, XML file, or a DNS record (again, DNS
  is the most durable choice).
- Submit the same `sitemap.xml` used for Google. Data typically starts
  populating within about 48 hours.

### 7. IndexNow

A push protocol: notify participating search engines the instant a page
is added, changed, or removed, instead of waiting for a crawl.

**Correction, since this is commonly misunderstood: IndexNow covers
Bing, Yandex, Naver, and Seznam — not Google.** Google relies on its own
crawling plus the mechanisms in section 5 above; it doesn't participate
in IndexNow and has no plans to. IndexNow is still worth setting up for
the engines it does cover, just don't expect it to affect Google
indexing.

Implementation for a site with no backend:

1. Generate a key (Bing Webmaster Tools has a key generator, or use any
   GUID-like string).
2. Host it as a plaintext file at `{{SITE_URL}}/{{KEY}}.txt`, containing
   just the key.
3. After each deploy, `POST` the changed URLs to the shared endpoint:
   ```json
   POST https://api.indexnow.org/IndexNow
   Content-Type: application/json

   {
     "host": "{{DOMAIN}}",
     "key": "{{KEY}}",
     "keyLocation": "{{SITE_URL}}/{{KEY}}.txt",
     "urlList": ["{{SITE_URL}}/page1.html", "{{SITE_URL}}/page2.html"]
   }
   ```
   A `200`/`202` response means success; `400`/`403`/`422`/`429`
   indicate a format, key, or rate-limit problem.

This is a natural post-deploy script step — a small addition to whatever
already syncs the site to its host, sending just the sitemap's URL list
(simplest, fine for a small site) or only the URLs that actually changed.

### 8. Core Web Vitals

Current thresholds (measured at the 75th percentile of real-user field
data — Chrome UX Report — not a one-off lab score):

| Metric | Good | Poor |
|---|---|---|
| LCP (loading) | ≤ 2.5s | > 4.0s |
| INP (interactivity) | ≤ 200ms | > 500ms |
| CLS (visual stability) | ≤ 0.1 | > 0.25 |

Lab tools (Lighthouse, PageSpeed Insights) are for development-time
diagnosis; the field data is what actually feeds the ranking signal. A
CDN-hosted static site should find this section close to "free" if
basic hygiene is followed:

- Modern image formats (AVIF/WebP with a fallback), explicit
  `width`/`height` attributes so images don't shift layout while loading.
- Long-`max-age`, fingerprinted filenames for static assets (JS/CSS/
  images); short/no-cache for HTML so content updates actually propagate.
- Defer/async non-critical third-party scripts; preload key fonts and
  use `font-display: swap` to avoid font-swap layout shift.
- Confirm compression and HTTP/2 are actually being served (section 1).

### 9. Analytics

| | GA4 | Privacy-first (Plausible / Fathom / Simple Analytics / Umami) |
|---|---|---|
| Cost | Free | Small paid SaaS fee, or free if self-hosted (Umami) |
| Cookies / consent banner | Cookie-based; generally needs an EU/UK consent banner | Cookieless; no banner needed in most jurisdictions |
| Data ownership | Google's servers/terms | Vendor's servers, or fully yours if self-hosted |
| Script weight | Heavier (~45KB) | Lightweight (<5KB) |
| Best fit | Deep funnel/conversion/ads-attribution needs, or once GSC↔Analytics linking matters | Simple, privacy-safe traffic stats with zero consent-banner overhead |

**Default recommendation**: start with a privacy-first tool — no
consent banner required, GDPR-safe out of the box, a single `<script>`
tag. Upgrade to GA4 later if/when deeper conversion-funnel or ads
attribution is genuinely needed — and add a cookie-consent banner at
the same time if the site has EU/UK visitors.

### 10. AI / answer-engine optimization (GEO/AEO)

Being cited in Google AI Overviews, Bing Copilot, ChatGPT, or Perplexity
runs on the same substrate as classic SEO, not a separate ruleset: clean
semantic HTML, clear heading structure, direct factual statements near
the top of a section (not buried after narrative build-up), accurate
structured data (an LLM retrieval pipeline parses `Organization`/
`Article` JSON-LD the same way a crawler does), and genuinely specific,
original facts rather than generic rephrased content. If sections 3-4
above are done properly, there's no separate "AI SEO" workstream needed.

**Correction: `llms.txt` is not an adopted standard.** Despite a lot of
SEO-content-marketing hype, Google has stated directly that it doesn't
use `llms.txt` and has no plans to — comparing it to the old, long-
ignored `keywords` meta tag. It's cheap and harmless to add (a plain
markdown file at `{{SITE_URL}}/llms.txt` linking to key pages) if you
want to hedge for AI clients that do consult it, but don't present it as
required infrastructure the way `robots.txt` and `sitemap.xml` are.

---

## Part 2 — Content & keyword strategy

### Keyword research, without a paid tool budget

1. **Seed from the business, not a tool**: list product/service
   categories, the specific problems solved, and the roles/personas
   involved in buying.
2. **Expand** each seed with free tools: Google Keyword Planner (volume/
   CPC ranges, no ad spend required), Google Trends (relative interest
   over time, geography, comparing terms), and Bing Webmaster Tools'
   Keyword Research tool (reports **exact** volumes rather than buckets,
   filterable by country/language/device — competition on Bing is
   typically much lower than on Google, making it a good early-win
   target too).
3. **Segment by funnel intent**: informational ("what is
   {{TOPIC}}", "how to {{TASK}}") for top-of-funnel blog content;
   comparison ("{{X}} vs {{Y}}", "best {{CATEGORY}} software") for
   mid-funnel; solution/brand+category terms for bottom-of-funnel/
   product pages.
4. **Once the site is live**, treat Search Console's own query data as
   the highest-value free source — queries already generating
   impressions but sitting at a low position or low CTR are the
   cheapest wins, since you already have some visibility.
5. **Prioritize long-tail, specific, lower-competition phrases over
   head terms** while the domain is new — a brand-new site competing
   directly on a broad, contested term against established competitors
   is a poor use of early effort.

### Content structure: topic clusters

- One **pillar page** covers a broad topic comprehensively; several
  **cluster pages** each cover a narrower subtopic in depth and link
  back to the pillar with descriptive anchor text, and the pillar links
  out to each cluster page. This internal-linking mesh signals topical
  authority more strongly than isolated, disconnected pages.
- Natural pillar topics for a software company: product/feature
  categories, use-case verticals, and integration/ecosystem topics
  (e.g. one pillar per major integration target, with cluster pages
  underneath).

### Cadence

**2-4 substantive posts a month** is a realistic, sustainable target for
a small team — consistency and depth beat volume or a daily-posting
burst followed by silence. Optionally pair with one longer flagship
piece (a pillar page or major guide) per quarter. A cadence you can
actually sustain indefinitely is worth more than a faster one you can't.

### Internal linking discipline

- Every blog post links at least once, ideally 2-3 times in context, to
  the most relevant product/service page, using descriptive anchor text
  (never "click here").
- Product pages link back to their most relevant supporting posts.
- No orphan pages — every published page should be reachable within a
  couple of clicks from the homepage/nav and present in the sitemap.

---

## Part 3 — Off-page / authority building

### Realistic tactics for a brand-new domain with zero existing authority

- **Relevant directories** — for software specifically: G2, Capterra,
  GetApp, Software Advice, AlternativeTo; for services/agencies: Clutch,
  GoodFirms; a launch-moment listing on Product Hunt or similar. Pick by
  whether your actual buyer persona uses that directory, not by raw
  traffic.
- **LinkedIn company page** — not a backlink in the classic sense, but
  baseline trust/brand signal expected of any B2B company: a populated
  profile, regular posts, and a live link to the site.
- **GitHub / open-source presence** — if there's any open-source
  component, SDK, or sample integration, a GitHub org profile with
  genuine technical content and links back to the site is a legitimate,
  relevant link source.
- **Guest content**, only on genuinely relevant, editorially-run
  industry publications — written for that audience, not purely to
  plant a link. Mass/low-quality guest-post schemes are a specific
  enforcement target (see below).
- **PR / launch outreach** — HARO itself is defunct; current
  replacements include Featured, Qwoted, and monitoring journalist
  requests directly on LinkedIn/X. Treat a launch as a sequenced effort
  (launch directories → PR outreach → industry directories), not a
  single event.
- **Broken-link building** — find relevant broken links in your niche
  and offer your own page as a replacement; low-risk and ethical.

### What to explicitly avoid

Google's spam enforcement in 2026 is, if anything, sharper than before,
with specific attention on AI-generated mass guest-post schemes as a
newer enforcement category. Treat these as hard rules, not soft
suggestions:

- Paid links that pass ranking credit without `rel="sponsored"` or
  `rel="nofollow"`.
- Large-scale/mass guest posting purely for links.
- Reciprocal link-exchange schemes.
- Private blog networks (PBNs), often built on expired domains.
- Automated link generation, and widget/footer link abuse.
- Non-editorial directory spam.

Manual actions target contaminated link profiles; recovery requires
auditing, removing or disavowing the offending links, and a
reconsideration request. It's cheaper to just not do this in the first
place.

### If the business has a reseller/partner channel

This is a genuine structural advantage, distinct from the generic
tactics above, and fully legitimate because the relationship is real and
disclosed rather than manufactured for SEO:

- **Partner directory listings** — if partners run an official
  integration/partner directory, get listed there; it's editorially
  justified and typically sits on a strong domain.
- **Natural cross-linking with resellers** — each reseller linking to
  you (and a "find a partner"/"our resellers" page linking back) is
  disclosed, mutually beneficial, and provides real user value (helping
  a prospective buyer find a local implementation partner) — not a link
  scheme.
- **Co-branded content** — joint case studies, webinars, and press
  releases create natural links from both sides without cold outreach.
- Keep anchor text varied and avoid site-wide footer link farms even
  here — the relationship being real doesn't mean the links should look
  mechanical.

---

## Part 4 — Signaling region/vertical without a storefront

**Correction: Google Business Profile generally does not apply** to a
purely online B2B business. Eligibility requires either a physical
location customers can visit or genuine in-person interaction; a company
operating entirely via web/email/video calls doesn't qualify, and
creating a listing anyway risks suspension. Skip it unless there's a
real physical office relevant to a subset of customers.

Instead, signal "who we serve" through:

- **Plain-text on-page content** — an explicit "industries we serve" /
  "regions we serve" section, in real text (not just conveyed through
  images or design).
- **Structured data** — `areaServed` on the Organization schema block,
  listing served countries/regions; `knowsAbout` or similar for
  verticals.
- **hreflang** — only if the site genuinely has multiple language/
  country-targeted page variants; skip entirely for a single-language
  site.
- **Consistent regional cues** — named client industries/regions in
  case studies and testimonials, and a clear "where we work" statement
  somewhere prominent (footer, about, or contact page).

---

## Part 5 — Ongoing cadence

**Weekly (~15-30 minutes):**
- Scan Search Console's Coverage/Indexing report for new errors (404s,
  redirect chains, server errors, accidental robots.txt blocks).
- Check the Core Web Vitals / Experience report for newly-flagged
  "Poor" pages.
- Glance at your tracked keywords for a sudden drop (more than 3-5
  positions), which usually signals an algorithm update or a competitor
  move worth investigating.

**Monthly:**
- Full Search Console performance review — impressions, clicks, CTR,
  average position, by page and query, month over month.
- A broken-link crawl (a free-tier crawler is generally sufficient for
  a small marketing site).
- Re-submit the sitemap in Search Console/Bing Webmaster if pages were
  added that month.
- Check indexed-page count against total published pages — a mismatch
  means Google has dropped something worth investigating.
- Pick 1-2 aging or underperforming pages to refresh (updated stats, new
  sections, corrected/updated product references).

**Quarterly:**
- A deeper technical audit: full crawl, orphan-page check,
  duplicate-content check, structured-data validation.
- Confirm zero manual actions / security issues in Search Console.
- A competitor gap check — what topics/keywords they now rank for that
  you don't, and what new backlinks they've picked up.
- A light backlink-profile review for anything toxic worth disavowing
  (rare for a small legitimate site, but worth a glance).
- Refresh the keyword/topic list against real Search Console query data
  to inform the next quarter's content calendar.

---

## Part 6 — KPIs and honest expectations

**Track**: impressions, clicks/organic sessions, CTR, average position,
indexed-page count vs. published count, referring domains, and —
ultimately the metric that matters to the business — organic
conversions (form fills, demo requests, contact submissions attributed
to organic traffic), not raw traffic as a vanity number.

**Realistic timeline for a brand-new domain** (set this expectation
explicitly rather than overpromise):

- **Months 1-3**: technical foundation, initial indexing, first pages
  crawled — expect minimal organic traffic.
- **Months 3-6**: first movement on long-tail, low-competition terms;
  early impressions growth in Search Console.
- **Months 6-12**: consistent, visible growth for a well-executed
  content/off-page program — roughly when a small SEO effort typically
  reaches break-even ROI.
- **9-12+ months**: meaningful rankings on more competitive terms,
  further out again in genuinely competitive B2B software niches.

Low-competition long-tail terms can occasionally rank in 1-3 months;
competitive "money" terms realistically take 6-12+ months. Treat SEO as
a compounding, multi-quarter investment, not a fast-payoff campaign.

---

## Part 7 — Site migration & redesign (existing site with real rankings)

Everything in Parts 1-6 assumes a new domain building organic
visibility from zero. This Part is for the opposite case: an existing,
already-indexed site with real rankings/traffic getting a redesign,
replatform, CMS-to-static migration, or domain change. A botched
migration can crater organic traffic for months or permanently — this
checklist exists because that risk is real and well-documented, not
theoretical.

**Assumes**: `project-discovery` has already established that this is
a rebuild (not a new build) with real search visibility worth
protecting, confirmed GSC/GA access, and captured `{{OLD_SITE_URL}}` —
don't re-derive any of that here. Also assumes the *implementation*
mechanism for serving redirects (how a 301 actually gets served from a
static, AWS-hosted site) is `website-deployment`'s concern — see that
skill's "URL redirect map" section. This Part owns strategy, the
redirect map itself, and GSC tooling, not the CloudFront/KVS mechanics.

### 7.1 Inventory every currently-indexed/ranking URL

Before building anything, assemble the full list of URLs actually worth
protecting — from all of these sources, not just the pages you remember
building:

- **GSC Coverage/Indexing report** — every URL Google currently has
  indexed for `{{OLD_SITE_URL}}`.
- **GSC Performance report** — sort by clicks/impressions; these are
  the highest-priority tier, since they're already earning real search
  visibility.
- **The current `sitemap.xml`** — the site's own declared canonical set.
- **An independent full crawl** (a free-tier crawler is enough for a
  small marketing site) — catches URLs that are still linked/indexed
  but missing from the sitemap.
- **GSC Links report** — "Top linked pages," for external backlink
  equity a sitemap alone won't reveal. This report is a sample, not
  exhaustive, per Google's own documentation — supplement with a
  third-party backlink tool if the site is large enough that a perfect
  1:1 map genuinely isn't feasible, and prioritize the redirect map by
  this list first when it isn't.

Union all four/five sources into one master list before writing a
single redirect rule.

### 7.2 Build the redirect map

- **301 (permanent), never 302** — a 302 tells Google the old URL
  should stay canonical, which is wrong for a permanent migration and
  can leave both old and new URLs competing or the old one lingering in
  the index.
- **Map to the single true final destination — zero redirect hops.**
  Old URL → new URL directly, never old → intermediate → new. Google's
  own guidance tolerates up to 5 hops, recommends ≤3; the actual target
  for a planned migration is 0.
- **One-to-one where possible; deliberate many-to-one (merge) where a
  genuine content match exists** — several old pages consolidating into
  one stronger new page is a legitimate, common pattern.

**Correction: never fall back to redirecting everything without a
match to the homepage.** This is a well-documented anti-pattern, not a
matter of taste — Google (including statements from Google's own John
Mueller) treats a redirect to an irrelevant destination as a **soft
404**: it gets re-crawled repeatedly but isn't indexed, and passes
little to no ranking credit — functionally worse than just letting a
genuinely valueless page 404/410. If a retired page has no reasonable
new equivalent and no meaningful traffic/links, let it 404/410 rather
than forcing a homepage redirect.

**Correction: only a working 301 actually preserves backlink equity —
don't plan on outreach to third-party sites to update their links.**
Asking every external site linking to `{{OLD_SITE_URL}}` to update its
link is unrealistic at scale; the redirect itself is what carries that
equity forward, given enough time live (see §7.9).

### 7.3 Domain/subdomain changes: GSC Change of Address tool

Use this only if the domain or subdomain is actually changing (e.g.
`{{OLD_DOMAIN}}` → `{{NEW_DOMAIN}}`). **It does not apply** to a
same-domain URL restructure, an HTTP→HTTPS move, a www/non-www switch
on the same domain, or a design/CMS change with no URL changes — those
don't need it at all.

- Verify **both** the old and new domain in Search Console under the
  same account first.
- **301 redirects must already be live** before filing — the tool
  validates a sample of top URLs.
- If moving domains, also verify and submit Change of Address for the
  www, non-www, and any subdomain variants of the *old* domain, even
  ones not actively serving traffic — Google's 2026 guidance update
  treats each verified variant as its own property that needs its own
  signal.
- Effects (prioritized crawling of the new domain, signal forwarding)
  last 180 days from filing — after that, Google treats the old domain
  as unrelated if it's still live. That means the redirects themselves
  need to stay live well past that window (see §7.9), not just the
  tool's own signal-transfer period.

### 7.4 Timing and rollout scope

- **Small-to-medium site**: migrate everything at once — this helps
  Google recognize it as a single coordinated move rather than
  independent, unrelated changes.
- **Large site**: migrating section-by-section is acceptable and makes
  monitoring/rollback more tractable — just keep each section's own
  redirect map complete before that section cuts over.
- Schedule the cutover for a real traffic lull where possible, and
  never mid-way through an unrelated infrastructure change.

### 7.5 Don't bundle content/design/URL changes together

**Correction: a redesign that changes visual design only — same URLs,
same titles/meta/H1s, same copy, same internal links — gives Google no
reason to re-evaluate a page at all.** Risk appears specifically when a
URL/design migration *also* rewrites titles, meta descriptions, H1s, or
restructures internal links in the same pass, because if rankings move
afterward there's no way to isolate which change caused it.

- Carry over existing title tags, meta descriptions, and H1s as-is (or
  with only minimal, necessary edits) through the cutover itself.
- Run any real on-page content refresh as a **separate, later pass**,
  after the migration itself has stabilized in Search Console (§7.9's
  weekly Coverage check turning quiet again is a reasonable go-ahead
  signal).
- Preserve the existing internal-link structure and anchor text where
  the new nav/design reasonably allows — new internal links should
  point straight at final new URLs, not through the redirect map.
- **Carry every JSON-LD block over into the new templates.** JSON-LD's
  separation from markup is exactly why it *can* survive a redesign
  cleanly, but the common real-world failure is a rebuilt template
  silently dropping the `<script type="application/ld+json">` block
  entirely. Re-validate every page type with
  [Rich Results Test](https://search.google.com/test/rich-results) and
  [validator.schema.org](https://validator.schema.org/) (Part 1 §4's
  tools) as an explicit migration gate, not an afterthought.

### 7.6 Protect the staging environment correctly

The specific, well-documented failure mode: a staging environment's
`Disallow: /` (correct for staging) survives, unchanged, into the
production `robots.txt` at launch — deindexing the entire live site
within days. Two facts make this worse than it looks:

- **`robots.txt` Disallow stops crawling, not indexing** — Google can
  still list a blocked URL (with no snippet) if it's discovered via a
  link elsewhere, so Disallow alone doesn't reliably keep a URL out of
  the index.
- **A `noindex` tag only works if the crawler can fetch the page** — if
  `robots.txt` blocks the URL first, Googlebot never sees the `noindex`
  tag, so blocking staging with `robots.txt` *and* relying on `noindex`
  to keep it out of the index is actually contradictory.

**Correct layering, staging → production:**

1. **HTTP Basic Auth (or an IP allowlist)** as the primary gate —
   crawlers can't authenticate, so they never see `robots.txt` or any
   meta tag at all.
2. **`noindex` meta tag / `X-Robots-Tag` header** as a second layer, in
   case auth is ever misconfigured or bypassed.
3. **`robots.txt` `Disallow: /`** only as a third, courtesy layer for
   well-behaved crawlers — never the sole defense.

**Explicit pre-launch and immediately-post-launch checklist item**:
diff the production `robots.txt` against staging's, and confirm no
`noindex` meta/header survived into production — this is an invisible
config detail nobody notices until rankings collapse.

### 7.7 Pre-launch testing

- Test the full site (including every redirect in the map) against
  staging before cutover.
- Confirm Googlebot isn't blocked by a firewall/DoS rule using GSC's
  URL Inspection tool on a few representative new URLs.
- For a domain-level DNS cutover, lower DNS TTL at least a week ahead
  of the move.
- Remove every temporary crawl block (staging `robots.txt`/`noindex`)
  as part of the cutover itself, not as a follow-up task — see §7.6.

### 7.8 Cutover sequence

1. Remove staging blocks; deploy the new site/DNS.
2. Submit the new `sitemap.xml` in Search Console (and Bing Webmaster
   Tools).
3. If a domain/subdomain changed, file the Change of Address request
   (§7.3) once redirects are confirmed live.
4. Optionally keep the *old* sitemap live, temporarily, listing the old
   URLs, to help crawlers rediscover them and hit the redirects sooner.
   **Correction, since this gets oversold**: asked directly, Google's
   John Mueller called this "fine, but I suspect the effect would be
   minimal." Treat it as a harmless, low-confidence hedge, not a
   load-bearing step. The redirects themselves, and the new sitemap, do
   the real work.

### 7.9 Post-migration monitoring

- **Daily for the first ~1-2 weeks, then weekly** through the first
  couple of months:
  - **GSC Coverage/Indexing report** — watch for a spike in 404s,
    "Redirect error," or a sudden drop in indexed-page count; this is
    the earliest reliable signal something in the redirect map is
    wrong.
  - **A full redirect-verification crawl of the entire old-URL
    inventory from §7.1** — checking actual HTTP status codes and final
    destination for *every* old URL, not a spot-check sample. Even a
    small gap in redirect coverage on a high-traffic old URL can cause
    a disproportionate traffic drop.
  - **Core Web Vitals** — a redesign/replatform is exactly the moment
    field data (Part 1 §8's thresholds) can regress; check it, don't
    assume the new templates are fine.
  - **GSC Performance trend** against the pre-migration baseline.
- **Escalation threshold**: if organic traffic is down more than
  roughly 25-30% and hasn't stabilized within about a month, treat it
  as a real problem and start a structured investigation (redirect
  gaps, accidental noindex/robots blocks, and content-parity are the
  first three things to re-check). A normal, well-executed migration's
  dip typically resolves within 2-4 weeks; Google's own docs note full
  resettling can take a few weeks to a few months, and complete
  signal/link-credit transfer on a domain change can take up to the
  ~180-day Change of Address window or longer — don't panic at week
  one, but don't wait indefinitely past a month with no explanation
  either.
- **GSC Links report churn is expected, not necessarily bad**: a
  redirect/canonical change can visibly reorder "Top linked pages"
  simply because credit is being consolidated onto new URLs — that can
  look like backlink loss when it's actually the migration working.

### 7.10 Redirect duration

Keep every migration redirect live for **at least 180 days, and
realistically closer to a year** — this is how long Google's own
guidance says full signal transfer (including reassigning credit from
third-party links pointing at old URLs) can take. There's no penalty
for leaving 301s in place indefinitely; the practical failure mode is
removing them too early, not too late.

### 7.11 Analytics continuity

**Keep the same GA4 property/measurement ID through the migration —
even across a domain change** — so historical trend comparisons across
the cutover stay valid. A GA4 property is defined by its Measurement
ID, not rigidly tied to one domain; retarget the existing property's
data stream to the new domain/design rather than starting a new
property. Only create a genuinely new property if the migration also
represents a distinct new business entity that needs to be tracked
completely separately going forward.

---

## Verification / regression checks

See the `website-testing` skill for how to set up (or extend) an
automated Playwright page-health check. It asserts, per page:

- A meta description is present and within a reasonable length range.
- The canonical tag is present and self-referencing.
- The full Open Graph set (title/description/type/url/image) is present.
- Valid JSON-LD is present where expected (e.g. Organization on the
  homepage, Article on blog posts).

See that skill's `references/website-testing.md` for the exact spec —
don't re-derive the assertions here.

---

## Out of scope

This doc covers organic/free discoverability only. It deliberately does
not cover: paid search campaigns (Google Ads, Microsoft Ads) and their
budget/bidding/conversion-tracking setup, broader brand or social-media
marketing strategy, or PR beyond the backlink-adjacent tactics in
Part 3. Those are separate concerns with their own playbooks.

Part 7's migration checklist assumes `project-discovery` has already
established that this is a rebuild of an existing, already-live site
and captured the current live URL, GSC access, and GA access — it does
not re-derive or re-ask that here. It also does not cover the actual
redirect *implementation* mechanism — how 301s get served from a
static, AWS-hosted site (a CloudFront Function keyed off a redirect
map) is `website-deployment`'s concern; this skill owns migration
strategy, the redirect map, and GSC tooling, not the CloudFront/KVS
mechanics.
