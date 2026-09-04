---
name: project-discovery
description: This skill should be used when the user asks to "start a new client project", "gather requirements for a new site", "research the client's competitors", "write a project brief", "run discovery for a new website", "what do I need to ask a new client", or otherwise needs a new site's business, audience, and competitive/market context gathered and researched into a standardized brief before build, SEO, sales-copy, or deployment work begins.
version: 1.0.0
---

# Project Discovery & Client Intake Playbook

Gather what's needed to start a new client project — a checklist of
questions to ask the client, plus live competitor/market research — into
a single standardized `PROJECT_BRIEF.md` that every other skill in this
plugin reads from instead of re-asking the same ground. Run this before
`website-build-standards`, before drafting any copy, before SEO planning,
before deployment.

This skill does **not** cover: the copywriting-specific discovery
questions (problem framing, pricing, objections, voice-for-copy, CTAs)
used to draft a homepage — that's `website-sales-tool`, asked *after*
this brief exists; legal-entity facts (registration number, entity type,
VAT status, registered address) — those must come from an actual
incorporation document, never a conversation, and stay owned by
`terms-of-use-website`; and the contact-form relay's own setup
requirements — `contact-form-integration`.

## Scope

- Copywriting-specific discovery (problem/solution/pricing/objections/
  voice-for-copy/CTAs) → `website-sales-tool`.
- Legal-entity facts sourced from incorporation documents → `terms-of-use-website`.
- Contact-form relay setup → `contact-form-integration`.
- Stack, folder layout, build-step rules → `website-build-standards` (this
  skill only decides *what pages* it needs to build, not how).

## Master intake checklist

Full question wording for every category is in the reference file. In
brief:

- **A. Business & product basics** — what the business does, the
  public-facing trading name, industry/maturity, new build vs. rebuild
  vs. CMS migration (and the current URL if one exists), the single most
  important outcome the site must produce.
- **B. Audience** — who the primary visitor is, whether there are
  multiple segments, where they currently find businesses like this.
- **C. Competitors named by the client** — 2-5 real names/URLs (seeds
  the research step below), which ones the client wants to actively win
  business from, any admired-but-not-competitor sites (kept separate, as
  inspiration).
- **D. Brand assets available** — logo files, existing palette/style
  guide, fonts, owned/licensed imagery vs. needing stock, existing
  collateral.
- **E. Pages & features needed** — the actual nav list (this becomes
  `website-build-standards`'s folder tree directly), any page outside
  the main nav, any feature beyond static content (a contact form →
  `contact-form-integration`; a calculator; a gated download; e-commerce
  — flag explicitly, since the current stack is brochure-only). If more
  than one language is needed, the full language/translation
  sub-checklist (which languages, who provides translation, per-language
  URL structure, RTL, legal-page coverage) is in the reference file — get
  a real answer to each rather than a one-word "yes."
- **F. Existing content/copy status** — what already exists vs. needs
  drafting, anything ranking that needs a preserved redirect on rebuild,
  anything that must migrate verbatim.
- **G. Domain & current hosting status** — does a domain already exist,
  where is it registered/DNS-hosted, is email running on it, is there a
  live site that must keep working during transition. Hands directly to
  `website-deployment`.
- **H. Budget & timeline** — launch date, fixed deadlines, budget band,
  any phasing.
- **I. Examples & inspiration** — 2-3 liked sites and specifically what
  about them, plus anything specifically disliked and why.
- **J. Voice & tone (baseline)** — formal/casual, technical/plain,
  words to avoid or insist on. This is a shallow baseline register only
  — not `website-sales-tool`'s narrower "existing copy whose tone should
  be matched" question, which only applies during an actual homepage
  rewrite.

## Live research workflow

Given the named competitors (C) and business description (A):

1. For each named competitor: search to confirm the current homepage/
   pricing URL, then fetch and read the homepage (and pricing page, if
   public). Record neutral, factual observations only — the value
   proposition the hero leads with, page/section order, whether pricing
   is public or gated, the apparent primary CTA, any differentiator
   claimed, tone/register — with the URL and today's access date next to
   each note, since competitor sites change.
2. A broader category scan (1-2 generic searches for the category) to
   note what's now table-stakes across it, beyond the client's own named
   list.
3. A rough 5-10 term seed-keyword pass, handed to `website-seo`'s
   existing content-strategy step as a starting point — not a duplicate
   of that skill's ongoing keyword-research work.
4. Write all of it into the brief's "Research notes" as neutral,
   factual observations. Never draft persuasive copy or make
   recommendations about what the client's own homepage should say at
   this stage — that stays `website-sales-tool`'s job, later.

Naming real competitors in this internal planning document is fine — it
is not a mandate for `website-sales-tool` to name them on the live page.
That skill's own rule is unchanged: position published copy against a
category, not a named competitor. This skill's research is context for
planning, not a license for what gets published.

## The Project Brief document

Create `PROJECT_BRIEF.md` in the **client site's own repo** (not this
plugin repo, which has no knowledge of individual clients — same
precedent as `website-deployment`'s `deploy.config.json`). Sections
mirror the checklist (A-J) plus:

- **Research notes** — the competitor/category/keyword findings above.
- **Open questions** — anything not yet answered. Log it here; never
  fabricate a placeholder answer.
- **Consumed by** — a short note on which sibling skill reads which
  section (E → `website-build-standards`, G → `website-deployment`,
  Research notes' seed-keyword list → `website-seo`, whole brief →
  `website-sales-tool` as context before its own deeper questions).

## Gotchas

- Never invent an answer for an unanswered checklist item — log it under
  "Open questions" instead.
- Research notes must stay factual observations, never drift into draft
  copy or persuasive framing.
- Verify domain/hosting answers (e.g. a registrar/`whois` check) rather
  than taking the client's recollection on faith — this feeds
  `website-deployment`'s DNS migration decision directly.
- Competitor research is internal planning material — it doesn't grant
  `website-sales-tool` permission to name competitors in published copy.

## Verification checklist

- Every checklist category is answered, or explicitly logged under Open
  Questions.
- Every competitor research note has a source URL and an access date.
- Section E's nav list is unambiguous enough to build a folder structure
  from directly, with no follow-up questions needed.
- Section G has a definite yes/no on whether a domain already exists.

## Additional resources

For the complete playbook — every checklist question spelled out in
full, the full research workflow with example search queries, the
complete `PROJECT_BRIEF.md` template, and a worked example — consult:

- **`references/project-discovery.md`** — the complete playbook
