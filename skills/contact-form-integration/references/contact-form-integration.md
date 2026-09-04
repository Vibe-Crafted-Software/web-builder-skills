# Website contact form — integration guide

How to add a working contact form to a website, using an already-running
shared relay backend. This doc is for whoever builds the *frontend* of a
site's contact form. It assumes a relay already exists and someone else
operates it — you don't need any AWS account, AWS credentials, AWS CLI,
or backend code to follow this. Everything here is plain HTML, CSS, and
JavaScript.

## How it works

```
Your site's frontend  --POST fetch()-->  the relay's fixed URL
                                              |
                                    (the relay operator's backend)
                                              |
                                   Arrives at the recipient email
                                   address YOU specify in the request
```

One relay endpoint serves many different sites. Each site tells the
relay, per submission, who should receive it (`recipient`) and what to
call itself in the email (`site`) — there's no per-site registration or
setup on the relay side. Adding a contact form to a new site is purely
this frontend change.

## What you need before starting

Ask whoever operates the relay for:

- **The relay's API URL** — one fixed endpoint, the same for every site
- **There is no API key, auth header, or per-site credential of any
  kind.** By design, this relay is an intentionally credential-free,
  open endpoint — the request body is the entire contract (`site`,
  `recipient`, `name`, `email`, `message`, `turnstileToken`, and an
  optional `company`). If you find yourself wondering where to get an
  API key or what header to send it in, stop — there isn't one, and
  adding an `Authorization`/`x-api-key` header does nothing (the relay
  never reads it).
- **Confirmation the relay can actually deliver to your intended
  recipient address.** The usual underlying cause is **AWS SES sandbox
  mode**: AWS's default restriction on new SES setups that only allows
  sending to individually-verified addresses, to prevent abuse from
  brand-new accounts. While a relay's SES setup is still in sandbox:
  - Only addresses the operator has manually verified in SES can
    receive mail — anything else silently fails to deliver (the relay
    itself may still return `200`, since the honeypot-style
    "acknowledge but don't guarantee delivery" pattern and genuine SES
    failures can look similar from the frontend — see Troubleshooting).
  - The real, durable fix is **requesting SES production access for the
    whole AWS account** — a one-time process (usually a short AWS
    support case) that removes the sandbox restriction entirely. This
    is an account-level task the operator does once, not something to
    repeat for every new site/recipient.
  - Verifying one recipient address at a time is a valid short-term
    workaround but doesn't scale past the first couple of sites — push
    for production access if this relay is meant to serve many clients.
- **A Cloudflare Turnstile site key**, if the relay has anti-abuse
  verification enabled (this is a *public* key, safe to put directly in
  your HTML — ask the operator whether this applies to your setup).
  Separately, confirm the relay's Turnstile **secret** key is actually
  configured server-side. Turnstile relays typically use one shared
  secret (and one shared widget) covering every site on the relay, not a
  secret per site — so this is a single yes/no fact to confirm with the
  operator once, not something to re-check per integration. If the
  secret isn't set, the server-side check silently passes every
  submission regardless of the token's validity: the widget renders
  normally, a token is generated and sent, but nothing on the backend
  actually verifies it. There is no way to detect this from the
  frontend — a network trace shows the same request either way. Ask
  directly.
- Whether anything is needed from you for **CORS** or **rate limiting**.
  In the common configuration: CORS is wide open (no per-site origin
  allow-list to get added to), and rate limiting is a single shared
  throttle across every site using the relay (a burst from your testing,
  or from a completely different site, can trip the same limit) rather
  than a per-site quota. Confirm both explicitly rather than assuming,
  since a different relay deployment could configure either one
  differently.

## The code

**HTML** — add this wherever the contact form goes on the page. The
honeypot field's odd name and label are deliberate — see the gotcha
below before changing them.

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

If the relay you're using doesn't have Turnstile enabled, you can drop
the `<script>` tag and the `.cf-turnstile` div entirely — the JavaScript
below already handles Turnstile being absent gracefully.

**JavaScript** (`contact-form.js`):

```javascript
document.addEventListener("DOMContentLoaded", function () {
  var RELAY_URL = "{{RELAY_API_URL}}";   // the relay's fixed endpoint, given to you by its operator
  var SITE_NAME = "{{SITE_NAME}}";       // shown in the email subject, e.g. "acmewidgets.com"
  var RECIPIENT = "{{RECIPIENT_EMAIL}}"; // where THIS site's submissions should go

  var form = document.querySelector("#contact-form");
  if (!form) return;

  var statusEl = document.querySelector("#form-status");
  var submitBtn = form.querySelector('button[type="submit"]');

  function setStatus(message, type) {
    statusEl.textContent = message;
    statusEl.className = "form-status is-visible is-" + type;
  }

  form.addEventListener("submit", function (event) {
    event.preventDefault();

    // Honeypot: real users never fill this in. If this ever silently
    // blocks real submissions, suspect browser autofill - see the
    // gotcha below before assuming the backend is broken.
    if (form.elements["hp_field"] && form.elements["hp_field"].value) {
      console.warn("Contact form: honeypot field was non-empty, submission blocked.");
      return;
    }

    var turnstileToken = window.turnstile ? window.turnstile.getResponse() : "";

    var payload = {
      site: SITE_NAME,
      recipient: RECIPIENT,
      name: form.elements["name"].value.trim(),
      email: form.elements["email"].value.trim(),
      message: form.elements["message"].value.trim(),
      turnstileToken: turnstileToken,
    };

    if (!payload.name || !payload.email || !payload.message) {
      setStatus("Please fill in your name, email, and message.", "error");
      return;
    }

    submitBtn.disabled = true;
    submitBtn.textContent = "Sending…";
    statusEl.className = "form-status";

    fetch(RELAY_URL, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(payload),
    })
      .then(function (response) {
        if (!response.ok) throw new Error("Request failed");
        return response.json().catch(function () { return {}; });
      })
      .then(function () {
        setStatus("Thanks — your message is on its way.", "success");
        form.reset();
        if (window.turnstile) window.turnstile.reset();
      })
      .catch(function () {
        setStatus("Something went wrong. Please try again, or email us directly.", "error");
      })
      .finally(function () {
        submitBtn.disabled = false;
        submitBtn.textContent = "Send message";
      });
  });
});
```

Fill in `{{RELAY_API_URL}}`, `{{SITE_NAME}}`, `{{RECIPIENT_EMAIL}}`, and
(if applicable) `{{TURNSTILE_SITE_KEY}}`.

An optional `company` field, if your form has one, is also accepted —
just add `company: form.elements["company"] ? form.elements["company"].value.trim() : ""`
to the payload; omitting it is fine too.

## Testing

**Test in a real browser, not just a scripted API call.** A direct
`fetch`/`curl` test can succeed while the real page still silently fails
— see the gotcha below. Fill out the form as a real user would, submit
it, and confirm both the success message appears *and* the email
actually arrives.

## Troubleshooting — matching a response to a cause

The relay has a small, fixed set of outcomes. Match what you actually
observe (status code and, where available, the response body's
`message`) against this table rather than guessing at a fix:

| Status | Response `message` (typical) | Cause | What to do |
|---|---|---|---|
| `400` | "Invalid request body." | The POST body wasn't valid JSON | Check `Content-Type: application/json` is set and the body is `JSON.stringify`'d, not a form-encoded body |
| `400` | "Please provide a valid name, email, message, and recipient." | One of `name`/`email`/`message`/`recipient` was missing, empty, or (for `email`/`recipient`) not a syntactically valid address | Check client-side validation is actually running before the request is sent |
| `400` | "Verification failed. Please try again." | Turnstile rejected the token | Only possible if the operator has a Turnstile secret configured — getting this response confirms enforcement is active. If it happens to real users repeatedly, check the site key matches the operator's widget |
| `502` | "Could not send message. Please try again later." | The relay's own email send failed server-side (commonly an SES-side issue — sandbox mode, a bounced/suppressed address, a misconfigured sender) | Not fixable from the frontend — report the exact time of the failed request to the operator so they can check their logs |
| `200` | A generic success message, but the email never arrives | Either (a) the honeypot field was silently filled — see the gotcha below, or (b) an SES delivery failure that the relay couldn't detect synchronously | Rule out the honeypot/autofill gotcha first; if that's clean, check recipient verification/SES sandbox status with the operator |
| *(none — not possible)* | — | There is no `401`/`403` path; the relay has no auth to fail | If you observe one, something other than the relay is intercepting the request first (a CDN/WAF rule, a corporate proxy, a browser extension) |
| `429` | (API Gateway's own throttling response, not from the relay's own code) | The relay's shared rate limit was hit | In the common configuration this is one global limit across every site on the relay, not a per-site quota — a testing burst or another site's traffic can trigger it. Space out retries; this usually isn't a bug in your integration |

## Gotcha: honeypot fields and browser autofill

If the form does nothing on submit — no network request, no console
error, no visible reaction at all — the most likely cause is **browser
autofill silently filling the honeypot field**. Some browsers (Edge
especially) autofill fields based on their name/label matching a known
category (like "company" or "website"), *even with*
`autocomplete="off"` on the field. Since the honeypot's whole job is
"if this field has any value, silently treat the submission as spam and
do nothing," an autofilled honeypot makes every real submission from
that browser vanish with zero feedback — not a bug in the network call,
not a bug in the backend, just a field name that accidentally matched
an autofill heuristic.

That's why the honeypot above is named `hp_field` and labeled "Leave
this field blank" — deliberately nothing that matches any real autofill
category. If you ever rename it to something more "natural," test
carefully in a real browser with real saved autofill data before
trusting it.

## If you don't have a relay to point this at yet

Setting up the relay backend itself is a separate, AWS-specific task for
whoever manages your infrastructure — it's out of scope for this doc,
which is only about integrating a site's frontend with one that already
exists.
