---
name: website-sales-tool
description: This skill should be used when the user asks to "rewrite a homepage as a sales pitch", "turn a homepage into a sales funnel", "write homepage copy", "improve landing page conversion", or needs a business's problem/solution/pricing/objections turned into homepage structure and copy.
version: 1.0.0
---

# Homepage Sales-Pitch Playbook

Turn a homepage from a brochure page ("here's what we do") into a
deliberate sales funnel ("here's the problem you have, here's how we
solve it, here's why it's low-risk to try"). Covers the homepage's copy
and section structure only — not technical SEO (see the `website-seo`
skill), not a Terms of Use (see the `terms-of-use-website` /
`terms-of-use-software` skills), not a full brand/visual redesign.

## How to use this skill

1. **Ask the discovery questions below first.** Don't draft copy from
   assumed answers — a sales pitch built on invented answers reads as
   generic, which is exactly what this rewrite is trying to fix.
2. **Map the answers into the page structure** below. Skip sections that
   don't apply rather than padding to fill a template.
3. **Apply the conversion-copy conventions** while drafting.
4. **Relocate existing homepage content that no longer fits — never
   just delete it.**
5. **Verify** before shipping.

## Discovery questions to ask before writing anything

If this project has a `PROJECT_BRIEF.md` (see the `project-discovery`
skill), read it first — it already has the business basics, audience,
and named competitors; don't re-ask that ground. The questions below are
copywriting-specific (problem framing, pricing/objections, voice, CTAs)
and go deeper than the brief covers.

Ask directly, in the business's own words — don't paraphrase an assumed
answer and ask for confirmation; an open question gets better copy than
anything invented.

- **The problem**: what does a typical prospect's situation look like
  *before* they come to this business? Get 2-4 distinct, concrete
  situations (not one vague pain point).
- **The solution**: for each problem, what's the one-sentence answer?
  What does the obvious alternative (a bigger/generic competitor, or
  doing it manually) do badly? Frame against a *category*
  ("bloated ERPs," "point solutions"), not a named competitor.
- **What they're actually selling**: buyers often think they're buying a
  feature list — what are they actually paying for (ongoing management,
  support, updates)? Where does AI/technology change a *named, concrete*
  outcome — never lead with "AI-powered" alone.
- **What's already documented elsewhere** (so the homepage teases and
  links instead of re-writing it).
- **Pricing and risk**: the actual commercial model in plain language,
  and 3-5 objections a prospect raises right before committing, each
  with an honest one-line answer.
- **Market/vertical scope**: one market or several? Name them
  specifically (if specificity is itself a credibility signal) or frame
  a general capability (if it should read as global, or naming 2-3
  markets would undersell broader ambition).
- **Proof**: what's real (named clients with permission, a review, a
  stat)? If there's genuinely none yet, that's a valid answer — ship
  without a fabricated proof section.
- **Voice**: existing copy whose tone should be matched; anything
  accurate on the *current* homepage worth keeping even if it won't fit
  the new structure — list it now, before cutting.
- **CTAs**: the one primary action, and an optional lower-commitment
  secondary action.

## Page structure

Attention → agitate the problem → relieve it → prove what's included →
scope/positioning → risk objections → close. Reuse the site's *existing*
design-system components (cards, grids, buttons) — don't invent a new
visual language for this rewrite.

- **Hero**: eyebrow naming the audience/situation, a benefit-driven H1
  (a reader should know the problem solved within five seconds), a lede
  expanding it into the mechanism, primary + optional secondary CTA.
- **The problem**: one card per problem situation, titled as a direct
  address to the reader, in the customer's own language.
- **The solution**: one card per solution point, same order/count as the
  problem cards so the pairing reads as deliberate.
- **What you're actually paying for**: management/support/reliability,
  the pricing model in plain language, the AI/technology point
  (outcome-specific), a short bespoke-work exception if relevant.
- **What's included** (only if a fuller feature list exists elsewhere):
  a condensed teaser reusing exact existing wording, linking through.
- **Market/vertical positioning** (only if relevant): specific named
  markets, or a general capability statement, per the discovery answer.
- **Low risk to get started**: one item per objection identified in
  discovery, each with its honest answer — place directly before the
  final CTA, not mid-page.
- **Closing trust + CTA**: a short honest note on who's behind the
  product (no fabricated scale claims) and a final clear CTA.

## Conversion-copy conventions (2026 research-backed)

- **Five-second clarity rule** — problem/product/audience must be clear
  before the visitor reads any body copy.
- **Position against a category, not a named competitor** — easier
  universal comparison, no legal-review overhead.
- **Don't lead AI messaging with the word "AI"** — if removing "AI"
  entirely would still describe something wanted, keep it; if not,
  rewrite around the concrete outcome instead.
- **Objection-handling copy goes near the CTA**, not mid-page.
- **Never fabricate social proof** — ship without a proof section rather
  than invent one.
- **One primary CTA + at most one lower-commitment secondary** — not a
  wall of equally-weighted buttons.
- **Market-scope tradeoff**: name specific markets when the specificity
  itself is the credibility signal; frame a general capability when the
  claim is really about a methodology (adapting to local norms) rather
  than current footprint — naming only 2-3 markets can undersell a
  globally-ambitious business.

## Handling existing homepage content: relocate, don't delete

Before cutting anything from the current homepage, check: is it already
duplicated elsewhere on the site in equal or richer detail (safe to drop,
cite the duplicate), or is it accurate/specific and not duplicated
anywhere else (must move to a named destination page — get that
confirmed before writing, the same way a new section's copy gets
confirmed)? A rewrite that quietly deletes real content reads as work
thrown away even when the new homepage is better.

## Verification checklist

- Every placeholder replaced with a real discovery answer — none left as
  a guess.
- Every problem card has a traceable matching solution card.
- No fabricated numbers, client names, testimonials, or logos.
- Every piece of removed content is either cited as already-duplicated,
  or has a named destination it was actually moved to — checked file by
  file, not from memory.
- Run the site's automated test suite after the rewrite (see the
  `website-testing` skill if one doesn't exist yet) — meta tags, broken
  links, and structured data are exactly what a content rewrite can
  silently break.
- Load the new homepage and every page that received relocated content,
  at a few viewport widths, and read it as a first-time visitor would.
- If the site alternates section background colors for visual rhythm,
  confirm inserting/removing sections didn't leave two matching sections
  back to back.

## Additional resources

For the full playbook with worked examples and deeper rationale on every
point above, consult:

- **`references/website-sales-tool.md`** — the complete playbook
