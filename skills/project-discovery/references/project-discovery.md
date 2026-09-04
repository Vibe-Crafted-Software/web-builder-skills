# Project Discovery — Complete Playbook

Full checklist wording, research workflow, brief template, and a worked
example for the `project-discovery` skill.

## Part 1 — The full intake checklist

### A. Business & product basics

- What does the business do, in one or two plain sentences?
- What public-facing trading name should appear on the site? (Not
  registration paperwork — that's `terms-of-use-website`'s job if/when a
  Terms page is drafted.)
- What industry/vertical, and how mature is the business (new startup,
  established, rebrand)?
- Is this a brand-new site, a rebuild, or a CMS migration (WordPress,
  Wix, Squarespace)? If existing, what's the current URL?
- What's the single most important outcome the site must produce (leads,
  sales, credibility, recruiting, information)?

### B. Audience

- Who is the primary visitor (role/company type if B2B, demographic if
  B2C)?
- Are there multiple audience segments needing different paths through
  the site?
- Where does this audience currently find businesses like this (referral,
  search, social, marketplace)?

### C. Competitors named by the client

- Which specific businesses does the client consider direct competitors?
  Get 2-5 real names/URLs — this seeds the research workflow in Part 2.
- Which, if any, does the client actively want to win business from vs.
  simply track?
- Any sites admired for craft/marketing even if not a competitor? Keep
  these separate from competitor research — they feed "Examples &
  inspiration" (I) instead.

### D. Brand assets available

- Actual logo files (vector if available — not a description of one).
- Existing color palette/style guide, if any.
- Fonts already in use, or license availability for new ones.
- Photography/imagery already owned/licensed, vs. needing stock or
  AI-generated images.
- Existing marketing collateral (deck, brochure) worth mining for copy
  or facts later.

### E. Pages & features needed

- What top-level nav items does the site need? Get the actual list —
  this becomes the folder structure directly.
- For any nav item with children, what are they?
- Any page outside the main nav (privacy policy, thank-you page,
  careers, a campaign landing page)?
- Any feature beyond static content: a contact form (→
  `contact-form-integration`), a calculator/tool, a gated download,
  multi-language support?
- Is i18n needed at launch or later?
- Any e-commerce/checkout requirement? Flag this explicitly and early —
  the current stack (`website-build-standards`) is brochure-only, so
  this changes the technical approach rather than being discovered
  mid-build.

### F. Existing content/copy status

- Does copy already exist per planned page, or does it need drafting?
- Any existing indexed content ranking for real keywords that must be
  preserved via a redirect on a rebuild?
- Any content that must migrate verbatim (legal-sensitive copy,
  published case studies)?

### G. Domain & current hosting status

- Does the client already own a domain? What is it?
- Where is it currently registered, and where is DNS currently hosted
  (the same place, or different)?
- Is email currently running on that domain (Google Workspace, Microsoft
  365, other)?
- If no domain yet, is a name already chosen?
- Is there a live site at that domain today that must keep working
  during the transition?

Verify these rather than relying on the client's recollection (a quick
`whois`/registrar-login check) — `website-deployment`'s DNS migration
playbook acts directly on this answer.

### H. Budget & timeline

- Target launch date, and any fixed external deadline.
- Approximate budget band, if shared — this affects scope decisions
  (custom photography vs. stock, i18n now vs. later).
- Any phasing planned (an MVP page set now, more later)?

### I. Examples & inspiration

- 2-3 sites (competitor or not) whose look/structure the client likes —
  and *specifically* what about them (a layout choice, the tone, a
  feature). "Make it look modern" isn't an answer; push for specifics.
- Anything they've seen and specifically dislike, and why.

### J. Voice & tone (baseline)

- Formal/casual? Technical/plain? Playful/serious?
- Words or phrases to avoid, or to insist on?

This is a shallow baseline register for general site copy — it is not
`website-sales-tool`'s narrower "existing copy whose tone should be
matched" question, which only matters during an actual homepage rewrite
and goes deeper than this baseline needs to.

## Part 2 — Live research workflow

### Step 1: Per named competitor

For each competitor named in checklist item C:

1. Search to confirm the current homepage and pricing-page URL (don't
   assume a URL the client gave is still current).
2. Fetch and read the homepage, and the pricing page if public.
3. Record neutral, factual observations:
   - What value proposition leads the hero?
   - What's the page/section order (problem-first? feature-first?
     pricing visible immediately?)?
   - Is pricing public or gated behind a form/call?
   - What's the apparent primary call-to-action?
   - What differentiator do they explicitly claim?
   - What's the tone/register (formal, casual, technical)?
4. Cite the URL and today's access date next to each note — competitor
   sites change, and a stale note without a date is misleading later.

Example search queries: `"<competitor name>" pricing`, `"<competitor
name>" vs`, `site:<competitor domain>` (to find their pricing/about
pages if not linked from the homepage).

### Step 2: Category scan

Run 1-2 broader searches for the category itself (e.g. "best
<category> for <audience>") to surface competitors beyond the client's
own list, and note *conventions* common across the category — do most
show pricing, lead with a demo request, share a similar page structure?
This is about what's now table-stakes in the category, recorded for
planning — not written as anyone's copy.

### Step 3: Seed-keyword pass

Produce 5-10 phrases the audience (from checklist item B) would
plausibly search. Hand this list to `website-seo`'s existing
content-strategy step as a starting point — that skill owns the ongoing
keyword research, free-tool expansion, and cadence; this is just the
one-time seed.

### Step 4: Write it up

Everything from steps 1-3 goes into the brief's "Research notes" section
as neutral, factual observations — never draft copy, never a
recommendation for what the client's own homepage should say. That
drafting work is `website-sales-tool`'s job, done later, using these
notes plus its own deeper discovery questions.

**Boundary to hold firmly**: naming real competitors in this internal,
unpublished planning document is fine and expected. It does not license
`website-sales-tool` to name competitors on the published page — that
skill's rule (position against a category, not a named competitor) is
unchanged and remains the authority on what actually ships.

## Part 3 — `PROJECT_BRIEF.md` template

Create this file in the client site's own repository (not this plugin
repo).

```markdown
# Project Brief — {{CLIENT_NAME}}

Prepared: {{DATE}}

## A. Business & product basics
- What the business does: {{BUSINESS_SUMMARY}}
- Trading name for the site: {{TRADING_NAME}}
- Industry/maturity: {{INDUSTRY_MATURITY}}
- New build / rebuild / migration (current URL if any): {{BUILD_TYPE}}
- Primary outcome the site must produce: {{PRIMARY_OUTCOME}}

## B. Audience
- Primary visitor: {{PRIMARY_AUDIENCE}}
- Additional segments: {{OTHER_SEGMENTS}}
- Where they currently find businesses like this: {{DISCOVERY_CHANNEL}}

## C. Competitors (named by client)
- {{COMPETITOR_1}} — {{URL}}
- {{COMPETITOR_2}} — {{URL}}
- Sites admired but not competitors: {{INSPIRATION_ONLY}}

## D. Brand assets available
- Logo: {{LOGO_STATUS}}
- Palette/style guide: {{PALETTE_STATUS}}
- Fonts: {{FONT_STATUS}}
- Imagery: {{IMAGERY_STATUS}}
- Existing collateral: {{COLLATERAL}}

## E. Pages & features needed
- {{NAV_ITEM_1}}
  - {{CHILD_PAGE}}
- {{NAV_ITEM_2}}
- Pages outside main nav: {{OTHER_PAGES}}
- Features beyond static content: {{FEATURES}}
- i18n: {{I18N_STATUS}}
- E-commerce/checkout: {{ECOMMERCE_STATUS}}

## F. Existing content/copy status
- {{CONTENT_STATUS}}
- Content requiring preserved redirects: {{REDIRECT_CANDIDATES}}
- Content that must migrate verbatim: {{VERBATIM_CONTENT}}

## G. Domain & current hosting status
- Existing domain: {{DOMAIN}}
- Current registrar/DNS host: {{CURRENT_DNS_HOST}}
- Email on this domain: {{EMAIL_STATUS}}
- Live site to keep working during transition: {{TRANSITION_NOTES}}

## H. Budget & timeline
- Launch date/deadline: {{LAUNCH_DATE}}
- Budget band: {{BUDGET_BAND}}
- Phasing: {{PHASING}}

## I. Examples & inspiration
- Liked: {{LIKED_SITES}} — specifically: {{WHAT_THEY_LIKE}}
- Disliked: {{DISLIKED_SITES}} — specifically: {{WHY}}

## J. Voice & tone (baseline)
- Register: {{TONE}}
- Words to avoid/insist on: {{WORD_PREFERENCES}}

## Research notes
- {{COMPETITOR_1}} ({{URL}}, accessed {{DATE}}): {{OBSERVATIONS}}
- Category conventions: {{CATEGORY_NOTES}}
- Seed keywords: {{KEYWORD_LIST}}

## Open questions
- {{UNANSWERED_ITEM}}

## Consumed by
- `website-build-standards` → Section E
- `website-sales-tool` → whole brief as context, before its own deeper
  discovery questions
- `website-seo` → Research notes' seed-keyword list
- `website-deployment` → Section G
```

## Part 4 — Worked example (excerpt)

```markdown
# Project Brief — Acme Facilities Co.

Prepared: 2026-09-04

## A. Business & product basics
- What the business does: Commercial facilities management (cleaning,
  maintenance, security staffing) for mid-size office buildings in
  Gauteng.
- Trading name for the site: Acme Facilities
- Industry/maturity: B2B services, established 8 years, first proper
  website (currently a single Wix landing page).
- New build / rebuild / migration: Migration off Wix,
  currently at acmefacilities.co.za
- Primary outcome the site must produce: Qualified quote requests from
  building/property managers.

## C. Competitors (named by client)
- CleanCorp SA — cleancorp.co.za
- Metro FM Services — metrofmservices.co.za

## E. Pages & features needed
- Home
- Services
  - Cleaning
  - Maintenance
  - Security Staffing
- About
- Contact
- Pages outside main nav: Privacy Policy
- Features beyond static content: contact form with quote-request
  fields (see `contact-form-integration`)

## Research notes
- CleanCorp SA (cleancorp.co.za, accessed 2026-09-04): hero leads with
  "24/7 facilities support," pricing not public (quote-request only),
  primary CTA is "Request a Quote," tone is formal/corporate, page order
  is services → case studies → contact.
- Category conventions: none of the 3 competitors surveyed publish
  pricing; all lead with a quote-request CTA rather than self-serve
  signup; case studies/logos are common trust signals in this category.
- Seed keywords: "facilities management Gauteng," "office cleaning
  contract Johannesburg," "commercial security staffing," "building
  maintenance company."
```

## Part 5 — Full verification checklist

- [ ] Every checklist category (A-J) is answered, or explicitly logged
  under Open Questions — no fabricated placeholder answers.
- [ ] Every competitor research note has a source URL and an access
  date.
- [ ] Section E's nav list is unambiguous enough to build a folder
  structure from directly (per `website-build-standards`'s folder-per-
  page convention), with no follow-up questions needed.
- [ ] Section G has a definite yes/no on whether a domain already
  exists, and — if yes — its current registrar/DNS host is confirmed,
  not assumed.
- [ ] Research notes contain only factual observations — no draft copy,
  no persuasive language, no recommendations for the client's own
  homepage content.
- [ ] `PROJECT_BRIEF.md` is saved in the client site's own repository,
  not this plugin repository.
