---
name: contact-form-integration
description: This skill should be used when the user asks to "add a contact form", "wire up a contact form to a backend", "integrate a website's contact form with a relay or API", or mentions a shared contact-form relay endpoint.
version: 1.0.0
---

# Contact Form Integration Guide

Add a working contact form to a website's *frontend*, using an
already-running shared relay backend. Assumes the relay already exists
and someone else operates it — no AWS account, credentials, CLI, or
backend code needed. Plain HTML, CSS, and JavaScript only.

## How it works

The site's frontend does a `POST fetch()` to the relay's fixed URL. The
relay's backend delivers it to the recipient email address specified in
the request. One relay endpoint serves many different sites — each site
tells the relay, per submission, who should receive it (`recipient`) and
what to call itself in the email (`site`). Adding a contact form to a new
site is purely a frontend change; there's no per-site registration on
the relay side.

## What to get from the relay operator before starting

- The relay's fixed API URL (the same for every site).
- **There is no API key or auth header.** By design, this is an
  intentionally credential-free, open relay — the request body (`site`,
  `recipient`, `name`, `email`, `message`, `turnstileToken`) is all it
  ever checks. If you're looking for where to get an API key, there
  isn't one; don't add an `Authorization`/`x-api-key` header, the relay
  never reads one.
- Confirmation the relay can actually deliver to the intended recipient
  address. In practice this usually means: **is the relay's AWS SES
  account still in sandbox mode?** While it is, only individually
  SES-verified addresses can receive mail — fixing this for a new
  recipient means either the operator verifies that address specifically,
  or (the real fix) requests SES production access for the account once.
  That's a one-time, account-level task, not something redone per site.
- A Cloudflare Turnstile site key, if the relay has anti-abuse
  verification enabled (a public key, safe to embed directly in HTML).
  Separately, **confirm the relay's Turnstile *secret* key is actually
  configured**, not just that you have a site key. If the operator
  hasn't set the secret, the relay's server-side check silently no-ops —
  the widget still renders and a token still gets sent, but nothing
  actually verifies it. There's no way to tell this from the frontend;
  ask the operator directly.
- Whether the relay's CORS/rate-limit setup needs anything from you — in
  the common configuration, CORS is wide open (no per-site origin
  allow-listing needed) and rate limiting is one shared, global throttle
  across every site using the relay, not a per-site limit. Confirm both
  with the operator rather than assuming, since a different deployment
  could configure this differently.

## The code

HTML (add wherever the form goes on the page — keep the honeypot field's
odd name/label exactly as shown, see the gotcha below):

```html
<script src="https://challenges.cloudflare.com/turnstile/v0/api.js" async defer></script>

<form id="contact-form" novalidate>
  <div class="form-honeypot" aria-hidden="true">
    <label for="hp-field">Leave this field blank</label>
    <input type="text" id="hp-field" name="hp_field" tabindex="-1" autocomplete="off" />
  </div>

  <div class="form-group">
    <label for="name">Name</label>
    <input type="text" id="name" name="name" required />
  </div>

  <div class="form-group">
    <label for="email">Email</label>
    <input type="email" id="email" name="email" required />
  </div>

  <div class="form-group">
    <label for="message">Message</label>
    <textarea id="message" name="message" required></textarea>
  </div>

  <div class="cf-turnstile" data-sitekey="{{TURNSTILE_SITE_KEY}}"></div>

  <button type="submit">Send message</button>
  <div id="form-status" role="status" aria-live="polite"></div>
</form>
```

If Turnstile isn't enabled on the relay being used, drop the `<script>`
tag and the `.cf-turnstile` div — the JavaScript handles Turnstile being
absent gracefully.

JavaScript (`contact-form.js`) posts `{site, recipient, name, email,
message, turnstileToken}` (plus an optional `company` field if the form
has one) to the relay URL, validates required fields client-side,
disables the submit button while sending, and shows a success/error
status message. Fill in `{{RELAY_API_URL}}`, `{{SITE_NAME}}`,
`{{RECIPIENT_EMAIL}}`, and `{{TURNSTILE_SITE_KEY}}` (if applicable) — see
`references/contact-form-integration.md` for the complete, ready-to-copy
script.

## Testing

Test in a real browser, not just a scripted API call — a direct
`fetch`/`curl` test can succeed while the real page silently fails (see
the gotcha below). Submit as a real user would and confirm both the
success message appears *and* the email actually arrives.

## Troubleshooting

The relay has a small, fixed set of outcomes — match what you observe
against this list rather than guessing:

- **`400`, invalid request body** — the POST body wasn't valid JSON.
  Check `Content-Type: application/json` is set and the body is actually
  `JSON.stringify`'d.
- **`400`, "provide a valid name, email, message, and recipient"** — one
  of those fields was missing or failed basic validation (e.g. `email`/
  `recipient` isn't a syntactically valid address).
- **`400`, verification failed** — Turnstile rejected the token. This is
  only ever checked if the operator has configured a secret key (see
  above) — if you get this, Turnstile enforcement is active.
- **`502`** — the relay's own email send failed on its side (commonly an
  SES-side issue). This isn't something to fix in the frontend; report it
  to the operator.
- **`200` with a generic success message, but no email arrives** — check
  the honeypot first (see the gotcha below), then SES sandbox/recipient
  verification (see above). A filled honeypot is designed to look
  identical to a real success from the frontend's perspective — that's
  intentional, not a bug to chase.
- **No `401`/`403` is possible** — there's no auth to fail (see above).
  If you're seeing one, something other than this relay is intercepting
  the request (a CDN rule, a proxy, browser extension, etc.).
- **`429`** — API Gateway's throttle was hit. In the common
  configuration this is one shared limit across *every* site using the
  relay, not a per-site limit — a burst from testing, or from another
  site entirely, can trigger it. Not necessarily a bug in this
  integration.

## Gotcha: honeypot fields and browser autofill

If the form does nothing on submit (no network request, no console
error, no visible reaction), the likely cause is **browser autofill
silently filling the honeypot field**. Some browsers (Edge especially)
autofill fields based on name/label matching a known category (like
"company" or "website"), even with `autocomplete="off"`. Since the
honeypot's job is "any value present = silently block the submission,"
an autofilled honeypot makes every real submission from that browser
vanish with zero feedback — not a network bug, not a backend bug, just a
field name that matched an autofill heuristic. This is why the honeypot
is named `hp_field` and labeled "Leave this field blank" — deliberately
nothing that matches a real autofill category. Test carefully in a real
browser with real saved autofill data before renaming it to anything
more "natural."

## If there's no relay to point this at yet

Setting up the relay backend itself is a separate, infrastructure-
specific task for whoever manages hosting — out of scope for this
skill, which only covers integrating a frontend with a relay that
already exists.

## Additional resources

- **`references/contact-form-integration.md`** — the complete integration
  guide, including the full, ready-to-copy JavaScript file.
