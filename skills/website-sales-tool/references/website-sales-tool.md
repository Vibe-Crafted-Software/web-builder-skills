# Homepage sales-pitch playbook — template

How to turn a homepage from a brochure page ("here's what we do") into a
deliberate sales funnel ("here's the problem you have, here's how we solve
it, here's why it's low-risk to try"). This doc is for whoever owns a
homepage rewrite — a developer, an AI agent, or a founder without a
copywriter on staff — and needs both the *questions to ask first* and the
*structure to write into*, not just a stack of marketing theory.

**This is a drafting aid, not a substitute for a real conversation with
the business.** Every placeholder below exists because the answer has to
come from the business itself — don't guess at a competitor's pricing
model, invent a customer pain point, or assume a market-positioning
decision on their behalf. If a placeholder can't be filled from something
the business has actually told you, ask before writing.

**Scope**: this covers the homepage's copy and section structure only.
It assumes a site that already exists — pages, a design system, a product
worth pitching. It does not cover technical SEO/discoverability (see the
`website-seo` skill), a legal Terms of Use (see the `terms-of-use-website`
/ `terms-of-use-software` skills), or a full brand identity/visual
redesign — those are separate, deliberately out of scope here.

---

## How to use this doc

1. **Ask the Part 1 discovery questions first.** Don't start writing
   copy before you have real answers — a sales pitch built on assumed
   answers reads as generic, and generic is exactly what a homepage
   rewrite is trying to fix.
2. **Map the answers into the Part 2 structure.** Not every section will
   apply to every business — skip what doesn't fit rather than forcing it.
3. **Apply the Part 3 conventions** (research-backed corrections to
   common copywriting instinct) while drafting.
4. **Handle existing homepage content per Part 4** — relocate what's
   still useful, don't just delete it.
5. **Verify per Part 5** before shipping.

---

## Part 1 — Discovery questions to ask before writing anything

Ask these directly, in the business's own words. Don't paraphrase an
assumed answer and ask them to confirm it — an open question gets you
their actual language, which is usually better copy than anything you'd
invent.

### The problem (the "before" state)
- What does a typical prospect's situation look like *before* they come
  to you? List 2-4 distinct, concrete situations, not one vague pain
  point — e.g. "running on spreadsheets," "using three disconnected
  tools," "outgrown a generic off-the-shelf product" are three different,
  nameable situations, not one blur.
- Complete the sentence: `Our best-fit customer is currently doing
  {{X}}, and it's not working because {{Y}}.` — get one of these per
  problem situation identified above.

### The solution (why you, specifically)
- For each problem above: what's your answer to it, in one sentence?
- What do you do that the obvious alternative (a bigger/more generic
  competitor, or doing it manually) *can't* do, or does badly? Frame this
  against a **category** ("bloated ERPs," "point solutions," "manual
  spreadsheets"), not a named competitor — see Part 3's note on why.
- Is there a flexibility angle — e.g. "works alongside what you already
  have," "no rip-and-replace required"? That's often a bigger relief to a
  buyer than a pure feature comparison.

### What you're actually selling
- Buyers often think they're buying a feature list. What are they
  *actually* paying for — ongoing management, support, updates, someone
  accountable when something breaks? Name it explicitly; don't assume
  it's obvious from the feature list alone.
- Where does AI (or any other differentiating technology) actually
  change the outcome for the customer — not "AI-powered," but the
  specific, named task that shrinks or disappears? If you can't name a
  concrete task, don't lead with the technology (see Part 3's gotcha).
- Is there a point past which you do fully custom/bespoke work outside
  your standard offering? Note it — it's usually a short closing
  aside, not its own section.

### What's already documented (don't duplicate it)
- Does another page already describe your full feature set, pricing
  tiers, or product details in depth? Name that page — the homepage
  should tease and link to it, not re-write it from scratch.

### Pricing and risk (the objections a buyer has right before saying yes)
- What's your actual commercial model (subscription, project-based,
  freemium, usage-based)? Describe it in plain language, no invented
  numbers if none exist yet.
- What objections does a prospect typically raise right before
  committing? List 3-5, and a one-line honest answer to each — e.g. "Is
  this going to be a huge upfront cost?" / "Can I get out if it doesn't
  work?" / "How long until I see value?" / "What happens as we grow?"
  This becomes the closing "low risk" section — see Part 2.

### Market / vertical scope
- Do you serve one market, or several? If several: do you want to name
  them specifically (works well if the specific markets are themselves a
  credibility signal — deep local expertise, an established local
  partner) or frame the capability generally (works better if you want to
  read as a global operation, or if naming only 2-3 markets undersells a
  broader ambition)? See Part 3 for the tradeoff.

### Proof (and its absence)
- What real, verifiable proof do you have today — named clients (with
  permission to use their name), a case study, a review, a usage
  statistic? List only what's real.
- **If there's genuinely no proof yet, say so explicitly rather than
  leaving the question unanswered** — the answer "we don't have
  public case studies yet" is a valid, actionable answer. It means the
  homepage ships without a social-proof section instead of a fabricated
  one. See Part 3's gotcha on this.

### Voice and existing content
- Is there existing copy (any page) whose tone should be matched? Quote
  a line or two as the reference.
- Is there anything on the *current* homepage that's accurate and worth
  keeping, even if it won't fit the new structure? List it now, before
  you start cutting — see Part 4.

### Calls to action
- What's the one primary action you want a visitor to take (contact
  form, demo request, signup)? What's a lower-commitment secondary
  action (see a features page, see pricing)?

---

## Part 2 — The page structure

A generic section order that maps the Part 1 answers into a funnel: grab
attention → agitate the problem → relieve it → prove what's included →
handle scope/positioning → handle risk objections → close. Not every
business needs all seven body sections — cut what doesn't apply rather
than padding to hit a count.

Reuse your site's *existing* design-system components (cards, grids,
buttons, section-alternating backgrounds) for every section below — a
homepage rewrite is not the moment to invent a new visual language. If
your project has a components inventory or style guide, read it before
drafting a single section.

**Hero**
- Eyebrow: names the audience/situation in one line: `{{AUDIENCE_OR_SITUATION_LINE}}`
- H1: a benefit-driven headline that states the promise, not a feature —
  a reader should be able to tell what problem you solve within five
  seconds of landing, before reading a word of body copy.
- Lede: one sentence expanding the H1 into the concrete mechanism —
  `{{HOW_IT_WORKS_ONE_SENTENCE}}`
- Primary CTA → `{{PRIMARY_CTA_DESTINATION}}`; optional secondary CTA
  (lower commitment) → `{{SECONDARY_CTA_DESTINATION}}`, often an in-page
  anchor to the "what's included" section below.

**The problem**
- One card per problem situation from Part 1, in the customer's own
  language, not yours. Title the section as a direct address to the
  reader (`You're probably {{DOING_X}}`), not an abstract label.

**The solution**
- One card per solution point from Part 1, ideally in the same order and
  count as the problem cards so the pairing reads as deliberate, not
  coincidental.

**What you're actually paying for**
- The value-reframe section: management/support/reliability, the pricing
  model in plain language, and the AI/technology point (outcome-specific,
  per Part 3). A short closing line naming the bespoke/custom-work
  exception, if one exists, belongs here rather than as its own section.

**What's included** *(only if there's an existing feature list elsewhere to tease)*
- A condensed teaser of already-written content — reuse the exact wording
  from wherever it's documented in full, don't paraphrase it a second
  time — with a link through to the full page.

**Market / vertical positioning** *(only if relevant to the business)*
- Either specific named markets with what's distinct about each, or a
  general capability statement, per the Part 1 decision — see Part 3's
  tradeoff note before picking.

**Low risk to get started**
- One item per objection identified in Part 1, each with its honest
  one-line answer. This section works best placed directly before the
  final CTA, not buried mid-page — see Part 3.

**Closing trust + CTA**
- A short, honest note about who's behind the product (no fabricated
  scale claims) and a final, clear CTA. If the current homepage already
  has a closing section like this that's still accurate, keep it here
  largely as-is rather than rewriting it for its own sake.

---

## Part 3 — Conversion-copy conventions

Corrections to common copywriting instinct, grounded in current
(2026) B2B/SaaS conversion research — called out explicitly rather than
stated as plain fact, so you can weigh them against your own case.

- **Five-second clarity rule.** A visitor should be able to answer "what
  problem, what product, who for" within five seconds of landing — a
  clever or abstract headline that requires reading three sections to
  understand costs more conversions than it gains in personality.
- **Correction: position against a category, not a named competitor.**
  "Spreadsheets," "point solutions," "bloated ERPs" are easier, more
  universal comparisons for a reader to make against their own situation
  than a head-to-head against one named competitor — and they don't
  require legal review the way naming a competitor can.
- **Correction: don't lead AI messaging with the word "AI."** 2026 buyers
  have absorbed several years of generic "AI-powered" vendor claims and
  evaluate on named, concrete outcomes instead. Test: if you removed the
  word "AI" entirely, would the sentence still describe something a
  customer wants? If not, it's describing the technology instead of the
  outcome — rewrite it around the outcome.
- **Objection-handling copy belongs near the CTA, not mid-page.** Risk
  reducers ("cancel anytime," "no big upfront cost," "start small")
  convert best placed directly before the action you want taken, not
  buried in a features section three scrolls earlier.
- **Never fabricate social proof.** No invented client counts, logos, or
  testimonials. If there's no real proof yet, ship without a proof
  section rather than inventing one — a business that's caught with a
  fake testimonial loses far more trust than one that simply hasn't
  published a case study yet.
- **Multiple CTAs by commitment level, not multiple CTAs by theme.** One
  clear primary action, and at most one lower-commitment secondary
  action (e.g. "see what's included" as an in-page anchor) — a wall of
  equally-weighted buttons makes the reader do the prioritizing you
  should have done for them.

### The market-scope tradeoff (specific vs. global)

There's no universally correct answer here — it depends on what the
named markets are actually doing for the pitch:
- **Name specific markets** when the specificity itself is the
  credibility signal — a named local partner, a stated track record, deep
  domain knowledge a competitor can't claim. Naming exactly 2-3 markets
  can also *undersell* a business that's positioning itself as a global
  or globally-ambitious operation — the specificity reads as a current
  limitation rather than a strength.
- **Frame it as a general capability** ("we build to how your market
  actually works, wherever that is") when the underlying claim is really
  about a *methodology* — you adapt to local tax/compliance/process norms
  — rather than about specific, current market presence. This keeps the
  door open to any market without needing a homepage edit every time a
  new one is added.

Whichever way this goes, keep the *specific* version of the story (real
market names, real local domain knowledge) somewhere on the site if it
exists — see Part 4. A homepage pitched at "global capability" and a
product/pricing page listing exactly three real markets aren't in
conflict; they're serving different readers at different points in the
funnel.

---

## Part 4 — Handling existing homepage content: relocate, don't delete

**Never delete existing homepage content just because it doesn't fit the
new structure.** Before cutting anything, check:

1. **Is it already duplicated elsewhere on the site, in equal or richer
   detail?** If yes — cite the duplicate page/section as evidence, and
   it's safe to drop from the homepage without relocating it.
2. **Is it accurate, specific, and not duplicated anywhere else?** If
   yes — it must move to wherever it now fits best, not disappear. Name
   the destination page explicitly as part of the plan, and get it
   confirmed before writing, the same way you'd confirm a new homepage
   section's copy.

A rewrite that quietly deletes real, accurate content (an engineering
philosophy explainer, a specific market's domain-knowledge blurb,
anything a previous stakeholder wrote deliberately) reads to that
stakeholder as work being thrown away, even when the homepage itself
reads better afterward. Relocating it costs one extra decision per piece
of content and avoids that entirely.

---

## Part 5 — Verification checklist

- [ ] Every `{{PLACEHOLDER}}` replaced with a real answer from Part 1 —
  none left as a guess.
- [ ] Every problem card has a matching solution card addressing it
  (even if not 1:1 in count, the pairing should be traceable).
- [ ] No fabricated numbers, client names, testimonials, or logos
  anywhere in the new copy.
- [ ] Every piece of content removed from the homepage is either cited
  as already-duplicated-elsewhere, or has a named destination page it
  was actually moved to (Part 4) — check this file by file, not by
  memory.
- [ ] Run the site's test suite after the rewrite — see the
  `website-testing` skill for how to set one up if the project doesn't
  have one yet. Meta tags, broken links, and structured data are exactly
  the kind of thing a content rewrite can silently break.
- [ ] Load the new homepage and every page that received relocated
  content, at a few viewport widths, and read it as a first-time visitor
  would — not just section by section in isolation.
- [ ] If the site alternates section background colors (or any other
  repeating visual rhythm) for rhythm/scanability, confirm inserting or
  removing sections didn't leave two matching sections back to back.

## Closing scope note

This covers homepage sales-pitch copy and structure only. Technical
SEO/discoverability work (structured data, sitemaps, meta tags,
analytics) is a separate concern — see the `website-seo` skill. Legal
terms governing the site or the product being sold are a separate
concern — see the `terms-of-use-website` and `terms-of-use-software`
skills. A full visual/brand identity redesign is out of scope here —
this playbook assumes the existing design system stays as-is and only
the copy and section structure change.
