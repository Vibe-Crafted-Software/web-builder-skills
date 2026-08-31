# Website Terms of Use — template

A reusable Website Terms of Use for a company-run marketing/informational
site: a brochure site with a contact form or lead-capture form, not an
online storefront with checkout. This doc is for whoever stands up a new
site's legal footer link — a developer, an AI agent, or a founder without
in-house counsel — and needs a solid first draft fast.

**Jurisdiction: South Africa first.** Every clause below is written
against South African law (ECTA, the CPA, POPIA, the Copyright Act, and
South African case law on contract fairness). See "Adapting to another
jurisdiction" at the end for what to change when reusing this template
for a site operating under different law.

**This is a drafting aid, not legal advice.** It's built from real
statutes and current legal-practice guidance, but every instantiated
Terms of Use should still get a read-through from an admitted attorney in
the relevant jurisdiction before you rely on it — especially the
liability, indemnity, and CPA-applicability clauses, which are exactly
the kind of clause a court will refuse to enforce if it's copied in
blindly without matching the business's actual facts (see the
Barkhuizen gotcha in Part 5).

**Scope of this document**: this template covers the *website* only —
browsing it, using its contact/lead form, and the content on it. It does
**not** cover:
- **Software products or custom development delivered to customers** —
  that's a different relationship with different risk (payment,
  IP ownership, warranties, support). See the companion
  `terms-of-use-software` skill.
- **A POPIA-compliant Privacy Policy or PAIA manual** — POPIA requires
  these as separate documents, and requires the business to register an
  Information Officer with the Information Regulator before that person
  acts (POPIA s55(1)-(2), via the Regulator's eServices portal). A
  Terms of Use can reference a Privacy Policy; it isn't one. Don't treat
  "we wrote a Terms of Use" as "we're POPIA compliant" — flag the
  Privacy Policy + Information Officer registration as separate,
  still-required work.

Everything below is `{{PLACEHOLDER}}`-tokenized. Fill in every
placeholder before publishing — an unfilled `{{PLACEHOLDER}}` in a live
legal document is worse than not having the document at all, since it
signals the whole thing was never actually reviewed.

---

## Part 1 — Required supplier/operator disclosures

South Africa's Electronic Communications and Transactions Act 25 of 2002
("**ECTA**") s43 requires a business transacting electronically with the
public to disclose its identity, legal status, registration details, and
contact information on the site. Even a lead-generation site with no
online checkout should carry this block — it's the least-disputable
"who are we, how do you serve us with legal notice" information a
visitor or regulator can ask for, and it's needed the moment the site
*does* grow an online checkout.

Publish this as a distinct, easy-to-find block (a "Company information"
or "Who we are" section) inside the Terms of Use page itself:

> **{{LEGAL_ENTITY_NAME}}** (registration number **{{REGISTRATION_NUMBER}}**),
> a **{{ENTITY_TYPE, e.g. private company}}** incorporated in
> **{{COUNTRY_OF_INCORPORATION}}**.
>
> Registered office: **{{REGISTERED_OFFICE_ADDRESS}}**
>
> Contact: **{{CONTACT_EMAIL}}**
>
> This website is operated at **{{SITE_URL}}**.

**Gotcha: don't assert a VAT number you don't have.** A CIPC company
registration certificate carries an income tax reference number, issued
automatically on incorporation — that is *not* the same as a VAT vendor
number, which SARS issues separately only once the business registers
for VAT (mandatory above a turnover threshold, optional below it).
Stating a VAT number the business doesn't hold is a compliance problem
on every invoice that follows. Leave it out, or label it explicitly as
"Income Tax Reference Number" until VAT registration actually happens.

## Part 2 — Acceptance of these terms

> By accessing or using this website, you agree to be bound by these
> Terms of Use. If you do not agree, do not use this website.

**Note:** for a browse-only informational site, this "continued use =
acceptance" (browsewrap) model is standard and enforceable under ECTA
s22 (which validates agreements concluded wholly or partly through data
messages, with no formality requirement beyond what the parties choose).
**The moment the site adds any paid checkout or account signup**,
upgrade this to active clickwrap — an unticked "I agree" checkbox the
user must actively select before the transaction completes proves
consensus far more robustly than a footer link, and ECTA's own
consumer-transaction disclosure duties (s43) start applying in full at
that point too (see Part 1).

## Part 3 — Acceptable use of the website

> You agree not to:
> - use this website for any unlawful purpose, or in any way that
>   infringes the rights of any person;
> - attempt to gain unauthorised access to this website, its servers,
>   or any system or network connected to it;
> - use any automated means (scraping, crawling bots outside those we
>   permit via `robots.txt`, or similar) to extract content from this
>   website at scale without our prior written consent;
> - introduce any virus, malware, or other harmful code to this website;
> - interfere with or disrupt this website or servers/networks
>   connected to it.
>
> We may suspend or terminate your access to this website, without
> notice, if we reasonably believe you have breached this clause.

**Note:** unauthorised access and interference with a computer system
are also criminal offences in South Africa under the Cybercrimes Act 19
of 2020 — this clause doesn't create that liability, it just reserves
the website operator's own civil remedy (blocking access) separately
from any criminal process.

## Part 4 — Intellectual property in the website

> Unless otherwise indicated, all content on this website — including
> text, graphics, logos, images, and the underlying design and code —
> is owned by or licensed to {{LEGAL_ENTITY_NAME}} and is protected by
> South African copyright and trade mark law. You may view and print
> pages from this website for your own personal, non-commercial use.
> You may not reproduce, republish, distribute, or create derivative
> works from any part of this website for any other purpose without our
> prior written consent.
>
> {{BRAND_NAME}} and our logo are trade marks of {{LEGAL_ENTITY_NAME}}.
> Nothing in these terms grants you any right to use them.

## Part 5 — Third-party links

> This website may link to third-party websites. We don't control and
> aren't responsible for the content, accuracy, or availability of any
> linked third-party website. A link doesn't imply our endorsement of
> that site. You access third-party websites at your own risk and
> subject to their own terms.

## Part 6 — Forms, submissions, and your information

> Any personal information you submit through this website (for example,
> via a contact or enquiry form) is handled in accordance with our
> Privacy Policy, available at {{PRIVACY_POLICY_URL}}. {{^IF_NO_PRIVACY_POLICY_YET}}[**Gap**: publish a POPIA-compliant Privacy Policy before relying on this
> clause — see the scope note at the top of this document.]{{/IF_NO_PRIVACY_POLICY_YET}}
>
> Don't submit any content through this website that is unlawful,
> defamatory, or that you don't have the right to share.

## Part 7 — Disclaimers

> This website and its content are provided "as is." To the maximum
> extent permitted by law, we make no representations or warranties of
> any kind, express or implied, about the completeness, accuracy,
> reliability, or availability of this website or its content. Nothing
> on this website constitutes professional, legal, or financial advice.

## Part 8 — Limitation of liability

> To the maximum extent permitted by law, {{LEGAL_ENTITY_NAME}} will not
> be liable for any indirect, incidental, special, or consequential loss
> or damage arising from or in connection with your use of, or inability
> to use, this website, even if advised of the possibility of such loss.
>
> Nothing in these terms limits liability for death or personal injury
> caused by our negligence, for fraud, or for anything else that cannot
> lawfully be excluded or limited under South African law.

**Gotcha: the Consumer Protection Act splits your audience in two.** The
Consumer Protection Act 68 of 2008 ("**CPA**") doesn't apply to a
transaction where the other party is a juristic person (a company, CC,
trust, etc.) whose asset value or annual turnover, at the time of the
transaction, equals or exceeds a threshold set by the Minister — currently
**R2 million** (Government Gazette 34181 of 1 April 2011). That means:
- **Individual/consumer visitors, and small-business (sub-R2m) visitors**
  get the full protection of CPA ss48-52: unfair, unreasonable, or
  unjust contract terms are void, and any clause that limits the
  operator's liability, imposes risk on the user, or requires an
  indemnity must be drawn to that person's attention "in plain,
  intelligible language" and in a way "likely to attract the attention
  of an ordinarily alert consumer" (CPA s49) — small dense boilerplate
  buried in a wall of text will not hold up against these visitors.
- **Larger corporate visitors (at/above R2m)** aren't protected by the
  CPA at all for that transaction, so the clause above can be drafted
  more broadly for them.

A single public Terms of Use page is read by both audiences at once, so
draft the liability clause to the *stricter* (consumer-facing) standard
— it's what actually gets enforced against the audience the CPA
protects, and a broader carve-out for large corporate counterparties
doesn't help a page every visitor sees.

**Gotcha: a liability cap isn't bulletproof even against a CPA-exempt
counterparty.** South African courts can refuse to enforce a contract
term — even between two sophisticated commercial parties where the CPA
doesn't apply at all — if enforcing it would be contrary to public
policy, following the Constitutional Court's test in *Barkhuizen v
Napier* 2007 (5) SA 323 (CC). An exemption clause that's unreasonable,
unconscionable, or was never actually drawn to the other party's
attention is exactly the kind of clause that test targets. Don't treat a
liability cap as an absolute shield; it's a strong deterrent against
speculative claims, not a guarantee against every claim.

## Part 9 — Indemnity

> You agree to indemnify {{LEGAL_ENTITY_NAME}} against any claim, loss,
> or damage arising from your breach of these terms or your misuse of
> this website.

## Part 10 — Changes to these terms

> We may update these terms from time to time. The "Last updated" date
> below reflects the current version. Continued use of this website
> after an update means you accept the revised terms.

**Note:** this passive-notice model matches the low-stakes,
browse-only nature of this document. The moment a material,
rights-affecting change happens (e.g. adding a paid product, or
changing how personal data is used), give affected users more direct
notice than a silent date change — an email, a banner, or (for a
material Privacy Policy change) whatever POPIA's own notice
expectations require.

## Part 11 — Governing law and jurisdiction

> These terms are governed by the laws of the Republic of South Africa.
> You agree to submit to the non-exclusive jurisdiction of the South
> African courts, with the {{PREFERRED_DIVISION, e.g. Gauteng Division
> of the High Court (Johannesburg)}} as the parties' preferred forum.

**Gotcha: you can't contract a claim out of the Magistrate's Court.** A
"High Court only" jurisdiction clause is common but can't override a
litigant's statutory right to bring a claim within the Magistrate's
Court's jurisdictional limit in that court instead. Draft the clause as
a *preference*, not an exclusive-forum guarantee, unless an attorney has
specifically confirmed the higher-value relationship this template is
being used for can support an exclusive clause.

## Part 12 — General

> If any provision of these terms is found unenforceable, the rest
> remains in full force. These terms, together with our Privacy Policy,
> constitute the entire agreement between you and {{LEGAL_ENTITY_NAME}}
> regarding your use of this website. Failure to enforce any provision
> is not a waiver of it.
>
> **Last updated: {{LAST_UPDATED_DATE}}**

---

## Verification checklist before publishing

- [ ] Every `{{PLACEHOLDER}}` replaced with the real value — grep the
  published page for `{{` before shipping.
- [ ] Registration number and registered office match the actual
  incorporation certificate (CIPC in South Africa), not a guess.
- [ ] No VAT number claimed unless the business is actually VAT-registered.
- [ ] A Privacy Policy exists (or is explicitly flagged as a known gap)
  before the "Forms, submissions, and your information" clause links to
  one.
- [ ] The page is reachable from the site footer, and included in
  `sitemap.xml`/the site's page-inventory test fixture if one exists.
- [ ] Read by someone who isn't the document's author, ideally an
  admitted attorney, before treating this as load-bearing.

## Adapting to another jurisdiction

To reuse this template for a site operating under a different country's
law, at minimum re-derive:
- **Part 1** — the mandatory supplier-disclosure regime (ECTA s43 here;
  most jurisdictions have an equivalent e-commerce/consumer-protection
  disclosure duty, e.g. the EU's e-Commerce Directive Art. 5).
- **Part 8** — the consumer-protection statute and its thresholds (the
  CPA and its R2m juristic-person threshold here) and the local
  unfair-contract-terms doctrine (here, *Barkhuizen*'s public-policy
  test) — both are jurisdiction-specific and don't transplant directly.
- **Part 6** — the data-protection law referenced (POPIA here; GDPR,
  CCPA, etc. elsewhere have different notice and rights requirements).
- **Part 11** — governing law and forum.

Don't assume any clause "probably transfers" — each of the above has
already been wrong once in this document's own history (see the VAT and
Magistrate's Court gotchas above), precisely because it looked like
boilerplate that any jurisdiction would accept.

## Closing scope note

This document is the **Website** Terms of Use only. Software licensing,
custom development, support, fees, and product-specific IP ownership
belong in the `terms-of-use-software` skill, not here. A Privacy Policy
and (in South Africa) a PAIA manual are separate, still-required
documents this template does not attempt to substitute for.
