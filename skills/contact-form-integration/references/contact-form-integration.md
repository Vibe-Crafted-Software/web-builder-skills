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
- **Confirmation the relay can actually deliver to your intended
  recipient address** — depending on how the relay's email sending is
  configured, an address may need to be pre-approved before it can
  receive mail; check with the operator rather than assuming
- **A Cloudflare Turnstile site key**, if the relay has anti-abuse
  verification enabled (this is a *public* key, safe to put directly in
  your HTML — ask the operator whether this applies to your setup)

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
