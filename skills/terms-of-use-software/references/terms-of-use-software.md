# Software Terms of Use — template

A reusable master Terms of Use for a company that (a) licenses
pre-built "standard" software products to multiple customers, and (b)
also delivers bespoke/custom software development under individually
negotiated engagements. It's written as one combined document because
that's how a small software business actually operates: the same
company, the same support desk, the same legal entity — just two
different commercial relationships layered on the same foundation. This
doc is for whoever needs a solid first draft of that foundation fast: a
developer, an AI agent, or a founder without in-house counsel.

**Jurisdiction: South Africa first.** Every clause below is written
against South African law — most importantly the Copyright Act 98 of
1978's treatment of computer programs (which is *not* what most
developers assume — see Part 3's gotcha), the Consumer Protection Act
68 of 2008, POPIA, and the Prescribed Rate of Interest Act 55 of 1975.
See "Adapting to another jurisdiction" at the end for what to re-derive
when reusing this for a different jurisdiction.

**This is a drafting aid, not legal advice.** Get an admitted attorney
to review the instantiated document before relying on it, particularly
the IP assignment clause (Part 3) and the liability cap (Part 9) — both
depend on facts (who actually controls development, what the business
can actually deliver on) that only the business itself can confirm.

**Scope of this document**: this template covers the commercial
relationship around **software** — licensing a standard product, or
building something bespoke. It does **not** cover:
- **General use of the company's marketing/informational website** —
  see the companion `terms-of-use-website` skill.
- **A POPIA-compliant Privacy Policy, Data Processing Addendum, or PAIA
  manual** — this doc's Part 8 references POPIA's operator/responsible-
  party framework but does not itself satisfy it. Those remain separate,
  required documents.

Every `{{PLACEHOLDER}}` below needs a real value before this is
published or signed. An unfilled placeholder in a document a customer
is meant to rely on is worse than no document.

---

## Part 1 — Definitions

> - **"Standard App"** means a pre-built software application offered
>   by {{LEGAL_ENTITY_NAME}} to more than one customer under
>   substantially the same terms, as described in the applicable Order.
> - **"Custom Development"** or **"Bespoke Software"** means software
>   designed and built specifically for one Client under a Statement of
>   Work ("**SOW**").
> - **"Order"** and **"SOW"** mean the specific proposal, quote, or
>   statement of work agreed in writing (including by email) between
>   {{LEGAL_ENTITY_NAME}} and the Client, describing the Standard App
>   licensed or the Custom Development to be performed, the fees, and
>   the timeline.
> - **"Deliverables"** means the Standard App and/or the software,
>   documentation, and other materials delivered under an SOW.
> - **"Background IP"** means intellectual property that exists before,
>   or is developed independently of, a particular Order or SOW —
>   including {{LEGAL_ENTITY_NAME}}'s Standard App codebases, internal
>   frameworks, tools, and know-how.
> - **"Custom IP"** means intellectual property created specifically
>   for the Client under an SOW, excluding any Background IP embedded
>   in it.
> - **"Client Materials"** means data, content, branding, and other
>   materials the Client provides to {{LEGAL_ENTITY_NAME}} for use in
>   performing an Order or SOW.
> - **"Fees"** means the amounts payable under an Order or SOW.

## Part 2 — How this document works with an Order/SOW

> This document is {{LEGAL_ENTITY_NAME}}'s master Terms of Use for
> software licensing and development. Every Order and every SOW
> incorporates it by reference. If an Order or SOW expressly states a
> different term for that specific engagement, the Order/SOW term
> prevails for that engagement only — this document still governs
> everything the Order/SOW doesn't address.

**Note:** this "master terms + engagement-specific SOW" structure is
what lets you sign a five-line SOW for each new client instead of
re-negotiating a full contract every time — the SOW only needs to state
what's actually engagement-specific (scope, price, timeline), and this
document carries everything else.

## Part 3 — Intellectual property

This is the section most likely to be copied wrong, because the
intuitive assumption — "the client paid for it, so the client owns
it" — is not how South African copyright law actually treats software.

**Gotcha: South African copyright law's usual "the client who
commissions and pays owns it" rule does *not* apply to computer
programs.** The Copyright Act 98 of 1978 s21(1)(c) does vest ownership
of certain *commissioned* works (photographs, portraits, gramophone
records, cinematograph films) in the person who commissioned and paid
for them — but that list explicitly excludes computer programs.
Copyright in a computer program instead vests in "the person who
exercised control over the making of the computer program" — which
usually means the developer who actually wrote and directed the build,
not the client who paid for it and waited for delivery. Practically:
without an explicit written assignment, a client who simply pays for
bespoke software can end up **not owning the copyright in it at all**,
regardless of what everyone assumed. The clauses below exist
specifically to remove that ambiguity by contract instead of leaving it
to the "who controlled the build" test.

> **3.1 Background IP.** {{LEGAL_ENTITY_NAME}} owns, and retains, all
> right, title, and interest in its Background IP, including the
> codebase of any Standard App and any reusable frameworks, components,
> or tools used to build Deliverables. Nothing in this document or any
> SOW transfers ownership of Background IP.
>
> **3.2 Standard App licence.** Subject to full payment of the
> applicable Fees, {{LEGAL_ENTITY_NAME}} grants the Client a
> non-exclusive, non-transferable, non-sublicensable licence to use the
> Standard App for the Client's own internal business purposes, for the
> {{LICENSE_TERM, e.g. duration of the applicable subscription/support
> period}}, as further described in the applicable Order. The Client
> may not, and may not permit any third party to: reverse-engineer,
> decompile, or disassemble the Standard App except to the extent
> applicable law makes this restriction unenforceable; resell,
> sublicense, or provide the Standard App to any third party as a
> standalone product; or use the Standard App to build a competing
> product.
>
> **3.3 Custom IP assignment.** Upon {{LEGAL_ENTITY_NAME}}'s receipt of
> full and final payment of all Fees due under the applicable SOW,
> {{LEGAL_ENTITY_NAME}} assigns to the Client all right, title, and
> interest in the copyright and other intellectual property rights in
> the Custom IP created specifically for the Client under that SOW,
> excluding any Background IP embedded in it, which remains licensed to
> the Client under clause 3.2 (applied to that SOW's Deliverables) for
> as long as the Client continues to use the Deliverables in accordance
> with this document. Until such payment is received in full, all
> Custom IP remains the exclusive property of {{LEGAL_ENTITY_NAME}}.
>
> **3.4 Client Materials.** The Client retains all rights in Client
> Materials. The Client grants {{LEGAL_ENTITY_NAME}} a licence to use
> Client Materials solely to perform the applicable Order or SOW.

**Note:** clause 3.3's "assignment conditional on full payment" pattern
does double duty — it resolves the control-test ambiguity above *and*
gives {{LEGAL_ENTITY_NAME}} real leverage against non-payment, since
ownership doesn't transfer until the invoice clears.

## Part 4 — Standard Apps: what's included

> Each Standard App licence includes the features and support described
> in the applicable Order (for example: a dedicated app website,
> in-app support ticketing, documentation, and AI-assisted setup, where
> stated). {{LEGAL_ENTITY_NAME}} may update or improve a Standard App
> from time to time; material reductions in functionality will be
> communicated in advance where reasonably possible.

**Gotcha: don't let this section promise more than the Order actually
describes.** It's tempting to hard-code "every Standard App includes
X, Y, Z" directly into the master terms — but then every future Order
is locked into that bundle even if a particular app genuinely doesn't
include one of those things yet. Point at "the applicable Order" as the
source of truth instead, and keep the master terms describing the
*mechanism* (support ticketing exists), not a fixed feature list.

## Part 5 — Custom Development engagements

> **5.1 Scope.** Each Custom Development engagement is governed by its
> SOW, which describes the scope, Fees, milestones, and timeline.
>
> **5.2 Change orders.** Either party may propose a change to scope,
> timeline, or Fees. No change is binding until agreed in writing
> (including by email) by both parties.
>
> **5.3 Acceptance.** {{LEGAL_ENTITY_NAME}} will notify the Client when
> a Deliverable is ready for review. The Client has {{ACCEPTANCE_PERIOD,
> e.g. 10 business days}} from that notice to identify, in writing, any
> material respect in which the Deliverable fails to conform to the
> applicable SOW. If the Client doesn't do so within that period, the
> Deliverable is deemed accepted. If the Client identifies a valid
> non-conformity, {{LEGAL_ENTITY_NAME}} will remedy it within a
> reasonable time and resubmit it for acceptance.

## Part 6 — Fees and payment

> Fees are stated in the applicable Order/SOW, exclusive of VAT and any
> other applicable taxes, which are payable in addition where
> applicable. Invoices are payable within {{PAYMENT_TERMS_DAYS, e.g.
> 30}} days of the invoice date. {{LEGAL_ENTITY_NAME}} may charge
> interest on overdue amounts at the rate prescribed from time to time
> under the Prescribed Rate of Interest Act 55 of 1975, and may suspend
> access to a Standard App or pause Custom Development work while an
> invoice remains unpaid beyond {{SUSPENSION_GRACE_DAYS, e.g. 14}} days
> past due, without that suspension constituting a breach by
> {{LEGAL_ENTITY_NAME}}.

**Note:** the Prescribed Rate of Interest Act is the standard South
African legal basis for charging interest on an unpaid commercial debt
absent a specific contractual rate — citing it here means the interest
clause has a statutory anchor instead of an arbitrary invented
percentage.

## Part 7 — Support, maintenance, and no guaranteed SLA by default

> {{LEGAL_ENTITY_NAME}} will use reasonable efforts to respond to
> support requests raised through the channel described in the
> applicable Order. Unless a separate Service Level Agreement has been
> purchased and expressly agreed in writing, no specific response time
> or uptime is guaranteed. Support does not cover issues caused by
> Client misuse, unauthorised modification, or third-party integrations
> not built or approved by {{LEGAL_ENTITY_NAME}}.

## Part 8 — Confidentiality and data protection

> **8.1 Confidentiality.** Each party will keep the other's
> confidential information (information disclosed under this
> relationship that a reasonable person would understand to be
> confidential) confidential, and use it only to perform its
> obligations here, except information that is public, was already
> known, is independently developed, or must be disclosed by law.
>
> **8.2 Personal information.** Where {{LEGAL_ENTITY_NAME}} processes
> personal information on the Client's behalf in connection with a
> Standard App or Custom Development (for example, the Client's own
> end-user data flowing through a system {{LEGAL_ENTITY_NAME}} built),
> {{LEGAL_ENTITY_NAME}} acts as an operator as defined in the Protection
> of Personal Information Act 4 of 2013 ("**POPIA**"), and the Client
> remains the responsible party. {{LEGAL_ENTITY_NAME}} will process
> that information only on the Client's documented instructions,
> maintain the confidentiality of it, and implement appropriate
> security safeguards, consistent with POPIA s19-s21. The parties will
> agree a separate data processing addendum where the engagement
> requires more specific terms than this clause provides.

**Note:** clause 8.2 is a pointer, not a substitute for an actual data
processing addendum once an engagement genuinely involves processing
someone else's personal information at scale (e.g. a Standard App that
stores the Client's own customers' data). Flag that as separate,
still-required work rather than treating this paragraph as sufficient
on its own.

## Part 9 — Warranties and disclaimers

> **9.1 Limited warranty.** {{LEGAL_ENTITY_NAME}} warrants that, for
> {{WARRANTY_PERIOD, e.g. 90 days}} from delivery, Deliverables will
> materially conform to their Documentation. The Client's sole remedy
> for breach of this warranty is that {{LEGAL_ENTITY_NAME}} will repair
> the non-conformity at no additional charge.
>
> **9.2 Disclaimer.** Except as expressly stated in clause 9.1, and to
> the maximum extent permitted by law, Deliverables are provided
> without any other warranty, express or implied, including any
> warranty of merchantability or fitness for a particular purpose.

**Gotcha: this disclaimer doesn't apply to every Client.** If the
Client is a "consumer" under the Consumer Protection Act 68 of 2008 —
broadly, a natural person, or a juristic person whose asset value and
annual turnover are both below the Minister's threshold (currently
**R2 million**, GG 34181 of 1 April 2011) — the CPA's implied warranty
that goods (which includes software supplied under the CPA's wide
definition) are of good quality and reasonably suited to their intended
purpose (CPA ss55-56) applies regardless of what clause 9.2 says, for
6 months after delivery. Clause 9.2 is only fully effective against
Clients above that threshold. Don't present it to a small-business or
individual client as if it eliminates all recourse — it doesn't.

## Part 10 — Limitation of liability

> To the maximum extent permitted by law, {{LEGAL_ENTITY_NAME}}'s total
> liability arising out of or in connection with an Order, an SOW, or
> this document, whether in contract, delict, or otherwise, is limited
> to the total Fees paid by the Client under the applicable Order/SOW
> in the {{LIABILITY_LOOKBACK_PERIOD, e.g. 12 months}} preceding the
> event giving rise to the claim. Neither party is liable for any
> indirect, special, or consequential loss, including loss of profit,
> revenue, or data.
>
> Nothing in this clause limits liability for death or personal injury
> caused by negligence, fraud, gross negligence, wilful misconduct, or
> anything else that cannot lawfully be excluded or limited under South
> African law.

**Gotcha: don't treat this cap as bulletproof against a sophisticated
counterparty either.** Following *Barkhuizen v Napier* 2007 (5) SA 323
(CC), a South African court can decline to enforce a contractual
limitation — even between two CPA-exempt commercial parties — if
enforcing it would be contrary to public policy (for example, because
it was never drawn to the other party's attention, or because
enforcing it in the circumstances would be unconscionable). A liability
cap is a strong deterrent and a real ceiling in the ordinary case; it
is not a guarantee that a court will enforce it in every case.

## Part 11 — Indemnity

> Each party will indemnify the other against third-party claims
> arising from that party's breach of this document, gross negligence,
> or wilful misconduct, subject to the limitation in Part 10.

## Part 12 — Term and termination

> **12.1** Either party may terminate an Order or SOW for the other
> party's uncured material breach, on {{CURE_PERIOD, e.g. 14}} days'
> written notice describing the breach, if it remains uncured at the
> end of that period.
>
> **12.2** {{LEGAL_ENTITY_NAME}} may suspend or terminate a Standard App
> licence for non-payment as described in Part 6.
>
> **12.3** On termination: the Client pays for all work performed and
> Fees due up to the termination date; any Standard App licence ends;
> and each party returns or destroys the other's confidential
> information and Client Materials on request, except as needed to
> comply with law or for backup/archival purposes consistent with
> ordinary retention practice.

## Part 13 — Force majeure

> Neither party is liable for a failure or delay in performance caused
> by circumstances beyond its reasonable control, including
> load-shedding or other utility failures, internet or cloud-provider
> outages, natural disaster, or governmental action, for as long as
> that circumstance continues.

**Note:** naming load-shedding explicitly is deliberate, not filler —
it's the single most common real-world force majeure event a South
African software business actually invokes, and naming it removes any
argument that it wasn't contemplated.

## Part 14 — Compliance

> Each party will comply with applicable law in performing its
> obligations, including anti-bribery and anti-corruption law (in South
> Africa, the Prevention and Combating of Corrupt Activities Act 12 of
> 2004) and any applicable trade control or sanctions law, where the
> engagement involves cross-border delivery.

## Part 15 — Governing law and dispute resolution

> These terms, and any Order or SOW entered into under them, are
> governed by the laws of the Republic of South Africa. The parties
> submit to the non-exclusive jurisdiction of the South African courts,
> with the {{PREFERRED_DIVISION, e.g. Gauteng Division of the High
> Court (Johannesburg)}} as the parties' preferred forum.

## Part 16 — General

> If any provision is found unenforceable, the rest remains in force.
> The order of precedence between documents is: the applicable
> Order/SOW, then this document, then any marketing or descriptive
> material (which is not contractually binding). Neither party may
> assign this document without the other's consent, except to a
> successor of substantially all its business. The relationship between
> the parties is that of independent contractors; nothing here creates
> a partnership, agency, or employment relationship. No failure to
> enforce any provision is a waiver of it.
>
> **Last updated: {{LAST_UPDATED_DATE}}**

---

## Verification checklist before publishing or signing

- [ ] Every `{{PLACEHOLDER}}` replaced with a real value.
- [ ] Part 3's IP assignment matches the business's actual commercial
  intent (some businesses deliberately keep more Background IP than
  others — confirm this reflects a real decision, not just the default
  text).
- [ ] Payment terms, warranty period, liability lookback period, and
  acceptance period reflect the business's actual commercial policy,
  not just this template's example numbers.
- [ ] A data processing addendum exists (or is flagged as a known gap)
  before an engagement that genuinely processes end-user personal data
  at scale relies on Part 8 alone.
- [ ] Read by someone who isn't the document's author, ideally an
  admitted attorney, before treating this as load-bearing — especially
  Part 3 and Part 10.

## Adapting to another jurisdiction

To reuse this template outside South Africa, at minimum re-derive:
- **Part 3** — the local default-ownership rule for commissioned
  software. Don't assume "client pays, client owns" holds; South
  Africa is a concrete example of a jurisdiction where it doesn't for
  computer programs specifically.
- **Part 6** — the statutory basis (if any) for charging interest on
  overdue invoices, and default payment-term norms.
- **Part 8** — the local data-protection law's processor/controller
  framework (POPIA's operator/responsible-party split here; GDPR's
  processor/controller elsewhere).
- **Part 9/10** — the local consumer-protection regime's thresholds and
  unwaivable implied warranties, and the local unfair-contract-terms
  doctrine (here, the CPA and *Barkhuizen*).
- **Part 13/14** — force majeure events and compliance regimes that are
  actually relevant locally (a jurisdiction without South Africa's
  utility-supply issues doesn't need the load-shedding callout, but
  will have its own equivalent worth naming specifically).
- **Part 15** — governing law and forum.

## Closing scope note

This document is the **Software** Terms of Use — Standard App licensing
and Custom Development. General use of the company's marketing website
belongs in the `terms-of-use-website` skill, not here. A Privacy Policy,
data processing addendum, and (in South Africa) a PAIA manual are
separate, still-required documents this template does not attempt to
substitute for.
