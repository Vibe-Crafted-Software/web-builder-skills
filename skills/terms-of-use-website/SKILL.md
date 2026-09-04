---
name: terms-of-use-website
description: This skill should be used when the user asks to "write a website terms of use", "draft a terms of use page", "add legal terms for a website", or needs a South Africa-first (adaptable) Terms of Use covering ECTA disclosures, acceptable use, and liability.
version: 1.0.0
---

# Website Terms of Use — Drafting Guide

Draft a reusable Website Terms of Use for a company-run marketing/
informational site (a brochure site with a contact/lead form, not an
online storefront with checkout). Written South Africa-first (ECTA, the
CPA, POPIA, the Copyright Act, and SA case law on contract fairness);
see the reference file's "Adapting to another jurisdiction" section to
re-derive for elsewhere.

**This is a drafting aid, not legal advice.** Every instantiated Terms
of Use should still get a read-through from an admitted attorney in the
relevant jurisdiction before being relied on — especially the liability,
indemnity, and consumer-protection-applicability clauses.

## Scope

Covers the *website* only — browsing it, its contact/lead form, and its
content. Does **not** cover:
- Software products or custom development delivered to customers — see
  the `terms-of-use-software` skill.
- A POPIA-compliant Privacy Policy or PAIA manual — POPIA requires these
  as separate documents and requires registering an Information Officer
  with the Information Regulator before that person acts. A Terms of Use
  can reference a Privacy Policy; it isn't one — flag Privacy
  Policy/Information Officer registration as separate required work.

## Before drafting — gather these facts (never guess or fabricate)

This is a separate, document-sourced legal fact-gathering step —
distinct from the general business/audience discovery in
`project-discovery`, which can't supply a registration number, entity
type, or VAT status; those must come from an actual incorporation
certificate, not a conversation.

- The legal entity name, registration number, entity type, and country
  of incorporation (from an actual incorporation certificate, e.g. CIPC
  in South Africa — never invent these).
- The registered office address (or a decision to omit it from public
  display and rely on an email-only notices clause instead).
- Whether a VAT number is actually held — a company registration
  certificate carries an income-tax reference number, which is **not**
  the same as a VAT vendor number (SARS issues that separately, only
  once VAT-registered). Never state a VAT number that doesn't exist yet.
- Preferred governing-law division/forum, and whether disputes go
  straight to courts or through arbitration first.
- Whether a Privacy Policy already exists to link to (if not, say so
  explicitly in the clause rather than link to nothing).

## Required content, in order

1. **Supplier/operator disclosures** (ECTA s43): legal name, registration
   number, entity type, country of incorporation, registered office (or
   its deliberate omission), contact email, site URL — as a distinct
   "Company information" block on the page itself.
2. **Acceptance of terms** — browsewrap ("continued use = acceptance") is
   fine for a browse-only site; upgrade to active clickwrap the moment
   the site adds paid checkout or account signup.
3. **Acceptable use** — no unlawful use, no unauthorized access, no
   scraping beyond what `robots.txt` permits, no malware, no
   interference; reserve a suspension/termination right.
4. **Intellectual property in the website** — content/design/code owned
   by or licensed to the operator; visitors get a limited personal-use
   licence only; trade marks reserved.
5. **Third-party links disclaimer**.
6. **Forms/submissions and personal information** — point to the Privacy
   Policy (or flag its absence explicitly).
7. **Disclaimers** — "as is," no warranty of accuracy/availability, not
   professional advice.
8. **Limitation of liability** — draft to the *stricter* consumer-facing
   standard even for a page every visitor type reads (see gotchas below).
9. **Indemnity** from the user for breach/misuse.
10. **Changes to terms** — passive notice (a "Last updated" date) is fine
    for this low-stakes document; escalate to direct notice for any
    material, rights-affecting change.
11. **Governing law and jurisdiction** — as a *preference*, not an
    absolute exclusive-forum guarantee (see gotcha below).
12. **General** — severability, entire agreement (with the Privacy
    Policy), no-waiver.

## Gotchas — apply these, don't skip them

- **Don't assert a VAT number the business doesn't have** (see above).
- **The Consumer Protection Act splits the audience in two**: individuals
  and small businesses below the Minister's threshold (currently R2
  million asset value/turnover) get full CPA protection — unfair terms
  are void, and liability-limiting/indemnity clauses must be drawn to
  their attention in plain, conspicuous language (CPA s49). Larger
  corporate visitors aren't CPA-protected for that transaction. Since one
  public page serves both audiences, draft the liability clause to the
  *stricter* standard.
- **A liability cap isn't bulletproof even against a CPA-exempt
  counterparty** — SA courts can refuse to enforce a term, even between
  sophisticated commercial parties, if enforcing it would be contrary to
  public policy (*Barkhuizen v Napier* 2007 (5) SA 323 (CC)). Treat a cap
  as a strong deterrent, not an absolute shield.
- **Can't contract a claim out of the Magistrate's Court** — draft
  jurisdiction as a preferred forum, not an exclusive-forum guarantee,
  unless an attorney has specifically confirmed the relationship
  supports one.

## Verification before publishing

- Every placeholder replaced with a real, verified value — grep the
  published page for unfilled tokens before shipping.
- Registration number and registered office match the actual
  incorporation certificate, not a guess.
- No VAT number claimed unless actually VAT-registered.
- A Privacy Policy exists, or its absence is explicitly flagged, before
  the forms/submissions clause links to one.
- The page is reachable from the site footer and included in the site's
  sitemap/page inventory.
- Read by someone other than the drafter, ideally an admitted attorney,
  before treating it as load-bearing.

## Additional resources

For the complete clause-by-clause template with exact draft language,
and the full "adapting to another jurisdiction" guidance, consult:

- **`references/terms-of-use-website.md`** — the complete template
