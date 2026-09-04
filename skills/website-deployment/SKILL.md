---
name: website-deployment
description: This skill should be used when the user asks to "deploy the site", "set up hosting on AWS", "provision AWS resources for a client site", "point the domain at AWS", "set up CloudFront", "deploy to S3", "invalidate the CloudFront cache", "move a domain's DNS to Route 53", "migrate a domain to AWS", or otherwise needs a static site's AWS hosting (S3 + CloudFront + Route 53 + ACM) provisioned, its routine deploy run, or its domain's DNS moved to Route 53.
version: 1.0.0
---

# Website Deployment Playbook

Host a site built per `website-build-standards` on AWS: one S3 bucket +
one CloudFront distribution + one ACM certificate + one Route 53 hosted
zone per client domain, all in a single shared AWS account. Covers
one-time provisioning of a new site's AWS resources, the routine
repeatable deploy, and moving a domain's DNS to Route 53.

## Scope

- Build output, folder layout, and the zero-build-step rule this skill
  assumes → `website-build-standards`.
- Running the test suite before a deploy → `website-testing` — that
  skill's own convention is "run `npm test` before deploy"; this skill
  reuses that, it doesn't restate it.
- `sitemap.xml`/`robots.txt` content → `website-seo`. They deploy like
  any other static file — no special handling here.
- IAM, EC2, Lambda, API Gateway, and any other AWS service — never
  touched by this skill's IAM role. In particular, the contact-form
  relay backend (`contact-form-integration`) runs in this same AWS
  account but is managed entirely separately; this role must never need
  or be granted access to it.
- Whether a domain already exists, and where it's currently registered/
  DNS-hosted → `project-discovery`'s intake checklist. This skill's
  Domain DNS migration section assumes that's already answered — it
  isn't a discovery step in itself.

## Architecture

Per client domain: a private S3 bucket (Block Public Access fully on,
read only via CloudFront), a CloudFront distribution using an Origin
Access Control (OAC) — not the legacy public S3 static-website-hosting
endpoint, which can't do HTTPS on a custom domain — an ACM certificate
(must be requested in `us-east-1`, regardless of which region the bucket
lives in, because that's a hard CloudFront requirement), and a Route 53
public hosted zone with alias records pointing at the distribution.

## IAM setup

Bucket naming convention: `site-<domain-with-dots-replaced-by-dashes>`
(e.g. `example.com` → `site-example-com`). This lets one IAM policy use a
single stable wildcard (`arn:aws:s3:::site-*`) that already covers every
future client bucket — onboarding a new site never requires editing the
policy. S3 bucket names are unique across every AWS account globally, so
check availability (`aws s3api head-bucket`) before assuming the name is
free; fall back to a prefixed variant and keep the policy's wildcard
consistent with whatever prefix is actually used.

Create an IAM role (not a user with static keys) that's assumed manually
from an existing admin identity in the same account:

- A trust policy allowing the account's own principals to assume the
  role (`sts:AssumeRole`), optionally gated on `aws:MultiFactorAuthPresent`
  for a role that's manually assumed for infrequent, high-trust actions.
- A customer-managed permissions policy scoped to exactly: S3 bucket and
  object management (scoped to the `site-*` wildcard), CloudFront Origin
  Access Control / distribution / function management and cache
  invalidation, ACM certificate request and management (locked to
  `us-east-1`), and Route 53 hosted-zone creation plus record changes.
  Some actions structurally require `Resource: "*"` because they create
  or list resources that don't have an ARN yet (`CreateHostedZone`,
  `CreateDistribution`, `CreateOriginAccessControl`, `ListAllMyBuckets`)
  — that's not a weaker grant, it's the only form those specific actions
  support; every action that manages an *existing* resource is ARN-scoped.
  A trailing explicit `Deny` on `iam:*`/`ec2:*`/`lambda:*`/`apigateway:*`
  (and other out-of-scope services) is included as defense-in-depth, so
  this role can't be widened by policy sprawl later without a deliberate
  edit to this policy itself.
- CLI commands to create the role, create the customer-managed policy,
  and attach it (`aws iam create-role` / `create-policy` /
  `attach-role-policy`).
- A local `~/.aws/config` profile using `role_arn` + `source_profile` so
  every command in this skill runs with `--profile site-deploy` (or
  `AWS_PROFILE=site-deploy`) rather than long-lived access keys.

Full JSON for both policies and the exact CLI commands are in the
reference file — this is the highest-stakes part of the skill, transcribe
it exactly rather than approximating from memory.

## One-time site provisioning

Ordered steps for a brand-new client site:

1. Create the S3 bucket with Block Public Access fully on and ownership
   set to `BucketOwnerEnforced` (disables ACLs — the correct pairing
   with an OAC-only access model).
2. Create a CloudFront Origin Access Control.
3. Create (once — reusable unmodified across every client distribution)
   a small CloudFront Function that appends `index.html` to clean-URL
   folder requests. S3-via-OAC does **not** do this automatically the
   way the legacy static-website endpoint did, so every distribution
   needs it attached as a `viewer-request` association or `/about/`-style
   URLs will 403/404.
4. Request an ACM certificate in `us-east-1` for the domain (+ `www`)
   via DNS validation.
5. Create (or confirm) the Route 53 hosted zone for the domain.
6. Add the ACM DNS validation CNAME record(s) to that zone, then wait for
   the certificate to reach `ISSUED`.
7. Create the CloudFront distribution: S3 origin via the OAC, the ACM
   cert, aliases for the domain (+ `www`), the AWS-managed
   `CachingOptimized` cache policy, and the URL-rewrite function from
   step 3 attached on `viewer-request`. Tag it with the domain name (and
   set its `Comment` to the domain) so it can be looked up later.
8. Attach a bucket policy granting `s3:GetObject` only to the CloudFront
   service principal, scoped via `AWS:SourceArn` to *this specific
   distribution's* ARN — not a wildcard, so no other distribution in the
   account can read this bucket.
9. Create Route 53 A and AAAA alias records for the domain (+ `www`)
   pointing at the distribution, using CloudFront's fixed alias
   hosted-zone-id constant.
10. Verify: `curl -I` against the domain root and a real folder-style
    page (confirms the rewrite function works), and check the browser
    padlock. CloudFront changes take roughly 5-15+ minutes to propagate
    — don't test immediately and conclude something's broken.

## Routine deploy

Every time a site's files change:

1. `aws s3 sync --delete` the built `/assets/` folder with a long,
   immutable `Cache-Control` header.
2. `aws s3 sync --delete` the rest of the site (HTML pages, etc.) with a
   short/no-cache `Cache-Control` header.
3. `aws cloudfront create-invalidation --paths "/*"` — always, every
   deploy. This stack has no build step and no content-hashed filenames,
   so a changed file re-uploads under the *same* name each time; the
   long asset cache TTL from step 1 is only safe because this step
   forces CloudFront to re-fetch from origin regardless of what the
   cache headers say. Invalidation is billed per path string submitted,
   not per object matched, so `"/*"` on every deploy is effectively free
   at this scale.

Keep the bucket name, distribution ID, hosted zone ID, and cert ARN for a
site in a `deploy.config.json` in that **client site's own repo** (not
this plugin repo, which has no knowledge of individual clients). If that
file is ever lost, look the distribution up by its `Comment`/tag from
step 7 above.

## Domain DNS migration

When a domain needs to move to Route 53 (DNS-only — this is sufficient
for hosting; see the note on registrar transfer below):

1. Inventory every existing DNS record directly from the domain's
   *current* authoritative nameservers — MX, TXT (SPF/DKIM/DMARC), A,
   AAAA, CNAME (including `www` and anything mail-related like
   `autodiscover`), and any other subdomains in use. Prefer a zone-file
   export from the current host if one's available.
2. Create the Route 53 hosted zone.
3. Recreate every record in it, **MX and TXT (SPF/DKIM/DMARC) first**,
   then A/AAAA/CNAME, then everything else. Email is the single biggest
   cutover risk — a missed mail record breaks deliverability silently,
   with no obvious error at cutover time.
4. Verify against the *new* zone's own nameservers directly (`dig
   @<route53-ns> ...`) and confirm every value matches the inventory —
   **before touching the registrar at all**.
5. Only once that's fully clean, update the domain's nameservers at its
   existing registrar to the 4 NS records Route 53 assigned.
6. TTL/propagation delay at this point only affects *when* a given
   resolver picks up the new (already-verified-correct) answer, never
   whether it's right once it does — you no longer control the old DNS
   host's TTL by the time you're mid-migration, so there's nothing to
   tune here; just don't mistake propagation delay for a broken record.
7. Leave the old DNS zone/provider active and untouched for a 48-72h
   safety window after cutover before deleting anything there.

Full registrar transfer into Route 53 Domains (making AWS the registrar
of record) is a separate, slower, optional follow-up — it needs an
unlock + auth code from the current registrar and a multi-day transfer
window — and isn't needed just to make hosting work. Only pursue it if
the domain should also change registrars.

## Gotchas

- ACM for CloudFront must be requested in `us-east-1` regardless of
  bucket region — a hard requirement, not a preference.
- S3 bucket names are globally unique across all AWS accounts — check
  availability before assuming the naming convention is free.
- CloudFront distribution creation/updates take 5-15+ minutes to
  propagate to edge locations.
- The clean-URL rewrite function (provisioning step 3) must be attached
  on every new distribution — it's easy to forget after the first one
  since it's not one of the "obvious" distribution fields.
- Block Public Access must stay fully on; the bucket has no other access
  path than the OAC-scoped policy, so turning any of the four settings
  off "to test something" defeats the entire model.
- Skipping the invalidation step after a sync can leave stale content
  live for up to a year, since nothing else forces a re-fetch.
- The biggest DNS-migration risk is a silent email outage — verify
  MX/SPF/DKIM/DMARC first and hardest, not just that the site resolves.
- Never widen this role's permissions "temporarily" for convenience —
  the explicit `Deny` in the policy exists specifically to make that
  harder to do by accident.

## Verification checklist

**Provisioning**: bucket has Block Public Access fully on and
`BucketOwnerEnforced`; the bucket policy's `AWS:SourceArn` names this
specific distribution, not a wildcard; ACM cert status is `ISSUED` and
was requested in `us-east-1`; distribution status is `Deployed`; the
CloudFront Function is `PUBLISHED` and attached on `viewer-request`;
Route 53 has both the ACM validation record(s) and the final alias
records; `https://<domain>/`, a real folder-style page, and
`https://www.<domain>/` all return `200` with a valid cert.

**Routine deploy**: both sync passes completed cleanly; spot-checked
`Cache-Control` headers on one asset and one HTML page; the invalidation
reached `Completed`; the live change is confirmed in a real browser, not
just via `curl`.

**DNS migration**: every record from the pre-migration inventory exists
in the new zone with matching values, MX/SPF/DKIM/DMARC checked
explicitly; verified against the new zone's nameservers before the
registrar was touched; old zone kept live through the safety window.

## Additional resources

For the complete playbook — full IAM policy JSON with the per-statement
scoping rationale, every CLI command with every flag, the complete
CloudFront `DistributionConfig` and bucket policy JSON, the CloudFront
Function source, an example `deploy.config.json`/`deploy.sh`, and the
full DNS-migration command sequence — consult:

- **`references/website-deployment.md`** — the complete playbook
