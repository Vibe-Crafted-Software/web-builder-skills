---
name: terms-of-use-software
description: This skill should be used when the user asks to "write software terms of use", "draft a SaaS or software licence agreement", "write terms for custom development work", or needs IP-ownership, licensing, and liability terms for a software product or development engagement, South Africa-first (adaptable).
version: 1.0.0
---

# Software Terms of Use — Drafting Guide

Draft a master Terms of Use for a company that (a) licenses pre-built
"Standard" software products to multiple customers, and/or (b) delivers
bespoke/custom software development under individually negotiated
engagements — as one combined document, since both relationships
typically run through the same company, support desk, and legal entity.
Written South Africa-first (the Copyright Act's treatment of computer
programs, the CPA, POPIA, the Prescribed Rate of Interest Act); see the
reference file's "Adapting to another jurisdiction" section for
elsewhere.

**This is a drafting aid, not legal advice.** Get an admitted attorney to
review before relying on it, particularly the IP assignment clause and
the liability cap — both depend on facts only the business can confirm.

## Scope

Covers the commercial relationship around **software** — licensing a
standard product, or building something bespoke. Does **not** cover:
- General use of the company's marketing/informational website — see
  the `terms-of-use-website` skill.
- A POPIA-compliant Privacy Policy, Data Processing Addendum, or PAIA
  manual — this doc's data-protection clause references POPIA's
  operator/responsible-party framework but doesn't itself satisfy it.

## Before drafting — gather these facts (never guess or fabricate)

- The legal entity name (matches the `terms-of-use-website` facts if the
  same company).
- Whether the business does bespoke custom development, Standard App
  licensing, or both.
- Actual commercial policy for: payment terms (days), warranty period,
  liability-cap lookback period, acceptance period for deliverables,
  cure period for breach — the template's example numbers (30 days
  payment, 90-day warranty, 12-month liability lookback, 10-business-day
  acceptance, 14-day cure) are reasonable industry defaults, **not**
  necessarily this business's actual policy — confirm before treating
  them as final.
- Who actually retains ownership of reusable frameworks/components
  ("Background IP") vs. what gets assigned to the client per engagement
  — a real business decision, not a default to accept blindly.

## The critical gotcha: IP ownership (get this exactly right)

South African copyright law's usual "the client who commissions and pays
owns it" rule does **not** apply to computer programs. The Copyright Act
98 of 1978 s21(1)(c) vests ownership of certain commissioned works
(photographs, portraits, gramophone records, films) in the person who
commissioned and paid for them — but that list explicitly **excludes**
computer programs. Copyright in a computer program instead vests in
whoever "exercised control over the making" of it — usually the
developer, not the client who paid and waited. Without an explicit
written assignment, a client who simply pays for bespoke software can
end up **not owning the copyright at all**, regardless of what everyone
assumed. The IP clause below removes that ambiguity by contract:

- **Background IP** (pre-existing frameworks, Standard App codebases,
  reusable tools) stays owned by the software company, licensed to
  clients only as embedded in what's delivered.
- **Custom IP** (built specifically for one client under an SOW) is
  assigned to the client **upon full and final payment** — conditional
  assignment both resolves the control-test ambiguity and gives the
  company real leverage against non-payment (ownership doesn't transfer
  until the invoice clears).

## Required content, in order

1. **Definitions** — Standard App, Custom Development/Bespoke Software,
   Order/SOW, Deliverables, Background IP, Custom IP, Client Materials,
   Fees.
2. **How this master document works with an Order/SOW** — every
   Order/SOW incorporates these terms by reference; an Order/SOW's
   explicit term prevails for that engagement only.
3. **Intellectual property** — as above (Background IP retained, Custom
   IP assigned on full payment, Client Materials retained by client).
4. **Standard Apps: what's included** — point at "the applicable Order"
   as the source of truth for included features; don't hard-code a fixed
   feature list into the master terms (a specific Order may not include
   everything yet).
5. **Custom Development engagements** — scope/Fees/timeline governed by
   the SOW; change orders require written agreement; a defined
   acceptance period with deemed-acceptance if the client doesn't object
   in writing.
6. **Fees and payment** — exclusive of VAT/taxes, a defined payment term,
   interest on overdue amounts under the Prescribed Rate of Interest Act
   55 of 1975 (the standard SA statutory basis, not an invented rate),
   and a right to suspend for non-payment past a grace period.
7. **Support, maintenance, no guaranteed SLA by default** — reasonable
   efforts unless a separate SLA is purchased and agreed in writing.
8. **Confidentiality and data protection** — mutual confidentiality; a
   POPIA operator/responsible-party clause (the software company is the
   "operator" processing on the client's instructions when handling the
   client's own end-user data) with a pointer to a separate data
   processing addendum for anything more specific.
9. **Warranties and disclaimers** — a limited warranty (materially
   conforms to Documentation for a defined period, sole remedy = repair)
   plus a broad disclaimer — **but see the CPA gotcha below**.
10. **Limitation of liability** — capped at Fees paid in a defined
    lookback period, excluding indirect/consequential loss, with
    carve-outs for death/personal injury/fraud/gross negligence.
11. **Indemnity** — mutual, for breach/gross negligence/wilful
    misconduct.
12. **Term and termination** — for uncured material breach (with a cure
    period), for non-payment, and the effect of termination (pay for
    work done, licence ends, return/destroy confidential info).
13. **Force majeure** — explicitly name load-shedding/utility failures if
    the business operates in a jurisdiction where that's the realistic
    recurring event, not just generic boilerplate.
14. **Compliance** — anti-bribery/anti-corruption law, trade
    control/sanctions law for cross-border delivery.
15. **Governing law and dispute resolution**.
16. **General** — severability, order of precedence (Order/SOW → this
    document → marketing material), assignment, independent-contractor
    relationship, no-waiver.

## Gotchas — apply these, don't skip them

- **The warranty disclaimer doesn't apply to every client.** If the
  client is a CPA "consumer" (a natural person, or a juristic person
  below the Minister's threshold, currently R2 million asset
  value/turnover), the CPA's implied warranty of good quality (ss55-56)
  applies for 6 months after delivery regardless of the disclaimer — the
  disclaimer is only fully effective against clients above that
  threshold. Don't present it to a small-business client as if it
  eliminates all recourse.
- **A liability cap isn't bulletproof even against a sophisticated
  counterparty** — SA courts can decline to enforce a limitation, even
  between two CPA-exempt commercial parties, if enforcing it would be
  contrary to public policy (*Barkhuizen v Napier* 2007 (5) SA 323 (CC)).

## Verification before publishing or signing

- Every placeholder replaced with a real value.
- The IP clause matches the business's actual commercial intent (some
  businesses deliberately keep more Background IP than others — confirm
  this reflects a real decision, not just the template default).
- Payment terms, warranty period, liability lookback, and acceptance
  period reflect actual commercial policy, not just the template's
  example numbers.
- A data processing addendum exists (or is flagged as a known gap)
  before an engagement genuinely processing end-user personal data at
  scale relies on the data-protection clause alone.
- Read by someone other than the drafter, ideally an admitted attorney,
  before treating it as load-bearing — especially the IP and liability
  clauses.

## Additional resources

For the complete clause-by-clause template with exact draft language,
and the full "adapting to another jurisdiction" guidance, consult:

- **`references/terms-of-use-software.md`** — the complete template
