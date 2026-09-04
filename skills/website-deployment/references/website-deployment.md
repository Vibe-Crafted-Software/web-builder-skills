# Website Deployment — Complete Playbook

Full IAM setup, provisioning, deploy, and DNS-migration detail for the
`website-deployment` skill. Replace `{{AWS_ACCOUNT_ID}}`, `{{DOMAIN}}`
(e.g. `example.com`), and other `{{PLACEHOLDER}}` tokens throughout.

## 1. IAM setup

### 1.1 Naming convention

- **Role name**: `website-deploy`
- **Customer-managed policy name**: `WebsiteDeployPolicy`
- **Bucket naming convention**: `site-<domain-with-dots-replaced-by-dashes>`,
  e.g. `example.com` → `site-example-com`. This lets the IAM policy use a
  single stable wildcard (`arn:aws:s3:::site-*`) that already covers every
  future client bucket, so onboarding a new client never requires editing
  this policy.
- Check name availability before provisioning (S3 bucket names are unique
  across *all* AWS accounts globally, not just this one):

```bash
aws s3api head-bucket --bucket site-example-com
# 404 Not Found  -> available
# 200 / 403      -> taken (by you or someone else) - pick a different prefix,
#                    e.g. qes-site-example-com, and use that prefix
#                    consistently in the IAM policy's wildcard below
```

### 1.2 Trust policy — who can assume the role

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowAccountPrincipalsToAssume",
      "Effect": "Allow",
      "Principal": { "AWS": "arn:aws:iam::{{AWS_ACCOUNT_ID}}:root" },
      "Action": "sts:AssumeRole",
      "Condition": {
        "Bool": { "aws:MultiFactorAuthPresent": "true" }
      }
    }
  ]
}
```

Naming `:root` as `Principal` does **not** mean the AWS account root login
has to be used — it means "any IAM principal in this account whose own
identity-based policy grants it `sts:AssumeRole` on this role's ARN" may
assume it. The real access gate is on the assuming side (the admin
user's own policy), not this trust policy. The `MultiFactorAuthPresent`
condition is recommended hardening for a role that's manually assumed for
infrequent, high-trust deploy actions — it requires either an
MFA-authenticated `source_profile` session or passing `--serial-number`/
`--token-code` to `aws sts assume-role` directly. If MFA enforcement isn't
wanted, drop the entire `Condition` block (not an empty object).

### 1.3 Permissions policy — complete, least-privilege

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3BucketManagement",
      "Effect": "Allow",
      "Action": [
        "s3:CreateBucket",
        "s3:PutBucketOwnershipControls",
        "s3:GetBucketOwnershipControls",
        "s3:PutBucketPublicAccessBlock",
        "s3:GetBucketPublicAccessBlock",
        "s3:PutBucketPolicy",
        "s3:GetBucketPolicy",
        "s3:PutBucketTagging",
        "s3:GetBucketTagging",
        "s3:GetBucketLocation",
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::site-*"
    },
    {
      "Sid": "S3ObjectManagement",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject",
        "s3:AbortMultipartUpload",
        "s3:ListMultipartUploadParts"
      ],
      "Resource": "arn:aws:s3:::site-*/*"
    },
    {
      "Sid": "S3ListBucketsAccountWide",
      "Effect": "Allow",
      "Action": "s3:ListAllMyBuckets",
      "Resource": "*"
    },
    {
      "Sid": "CloudFrontOACCreateAndList",
      "Effect": "Allow",
      "Action": [
        "cloudfront:CreateOriginAccessControl",
        "cloudfront:ListOriginAccessControls"
      ],
      "Resource": "*"
    },
    {
      "Sid": "CloudFrontOACManage",
      "Effect": "Allow",
      "Action": [
        "cloudfront:GetOriginAccessControl",
        "cloudfront:UpdateOriginAccessControl",
        "cloudfront:DeleteOriginAccessControl"
      ],
      "Resource": "arn:aws:cloudfront::{{AWS_ACCOUNT_ID}}:origin-access-control/*"
    },
    {
      "Sid": "CloudFrontDistributionCreateAndList",
      "Effect": "Allow",
      "Action": [
        "cloudfront:CreateDistribution",
        "cloudfront:ListDistributions"
      ],
      "Resource": "*"
    },
    {
      "Sid": "CloudFrontDistributionManage",
      "Effect": "Allow",
      "Action": [
        "cloudfront:GetDistribution",
        "cloudfront:GetDistributionConfig",
        "cloudfront:UpdateDistribution",
        "cloudfront:DeleteDistribution",
        "cloudfront:CreateInvalidation",
        "cloudfront:GetInvalidation",
        "cloudfront:ListInvalidations"
      ],
      "Resource": "arn:aws:cloudfront::{{AWS_ACCOUNT_ID}}:distribution/*"
    },
    {
      "Sid": "CloudFrontFunctionCreateAndList",
      "Effect": "Allow",
      "Action": [
        "cloudfront:CreateFunction",
        "cloudfront:ListFunctions"
      ],
      "Resource": "*"
    },
    {
      "Sid": "CloudFrontFunctionManage",
      "Effect": "Allow",
      "Action": [
        "cloudfront:DescribeFunction",
        "cloudfront:GetFunction",
        "cloudfront:UpdateFunction",
        "cloudfront:PublishFunction",
        "cloudfront:TestFunction",
        "cloudfront:DeleteFunction"
      ],
      "Resource": "arn:aws:cloudfront::{{AWS_ACCOUNT_ID}}:function/*"
    },
    {
      "Sid": "CloudFrontKVSCreateAndList",
      "Effect": "Allow",
      "Action": [
        "cloudfront:CreateKeyValueStore",
        "cloudfront:ListKeyValueStores"
      ],
      "Resource": "*"
    },
    {
      "Sid": "CloudFrontKVSManage",
      "Effect": "Allow",
      "Action": [
        "cloudfront:DescribeKeyValueStore",
        "cloudfront:DeleteKeyValueStore",
        "cloudfront:UpdateKeyValueStore"
      ],
      "Resource": "arn:aws:cloudfront::{{AWS_ACCOUNT_ID}}:key-value-store/*"
    },
    {
      "Sid": "CloudFrontKVSDataPlane",
      "Effect": "Allow",
      "Action": [
        "cloudfront-keyvaluestore:DescribeKeyValueStore",
        "cloudfront-keyvaluestore:ListKeys",
        "cloudfront-keyvaluestore:GetKey",
        "cloudfront-keyvaluestore:PutKey",
        "cloudfront-keyvaluestore:DeleteKey",
        "cloudfront-keyvaluestore:UpdateKeys"
      ],
      "Resource": "arn:aws:cloudfront::{{AWS_ACCOUNT_ID}}:key-value-store/*"
    },
    {
      "Sid": "CloudFrontTagging",
      "Effect": "Allow",
      "Action": [
        "cloudfront:TagResource",
        "cloudfront:ListTagsForResource"
      ],
      "Resource": [
        "arn:aws:cloudfront::{{AWS_ACCOUNT_ID}}:distribution/*",
        "arn:aws:cloudfront::{{AWS_ACCOUNT_ID}}:function/*"
      ]
    },
    {
      "Sid": "ACMRequestAndListUsEast1Only",
      "Effect": "Allow",
      "Action": [
        "acm:RequestCertificate",
        "acm:ListCertificates"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": { "aws:RequestedRegion": "us-east-1" }
      }
    },
    {
      "Sid": "ACMManageUsEast1Only",
      "Effect": "Allow",
      "Action": [
        "acm:DescribeCertificate",
        "acm:GetCertificate",
        "acm:AddTagsToCertificate",
        "acm:DeleteCertificate",
        "acm:RenewCertificate"
      ],
      "Resource": "arn:aws:acm:us-east-1:{{AWS_ACCOUNT_ID}}:certificate/*"
    },
    {
      "Sid": "Route53ZoneCreateAndList",
      "Effect": "Allow",
      "Action": [
        "route53:CreateHostedZone",
        "route53:ListHostedZones",
        "route53:ListHostedZonesByName"
      ],
      "Resource": "*"
    },
    {
      "Sid": "Route53ZoneManage",
      "Effect": "Allow",
      "Action": [
        "route53:GetHostedZone",
        "route53:ListResourceRecordSets",
        "route53:ChangeResourceRecordSets",
        "route53:ChangeTagsForResource",
        "route53:ListTagsForResource"
      ],
      "Resource": "arn:aws:route53:::hostedzone/*"
    },
    {
      "Sid": "Route53ChangeStatus",
      "Effect": "Allow",
      "Action": "route53:GetChange",
      "Resource": "arn:aws:route53:::change/*"
    },
    {
      "Sid": "DenyOutOfScopeServicesBeltAndSuspenders",
      "Effect": "Deny",
      "Action": [
        "iam:*",
        "ec2:*",
        "lambda:*",
        "apigateway:*",
        "organizations:*",
        "account:*",
        "aws-portal:*",
        "ce:*"
      ],
      "Resource": "*"
    }
  ]
}
```

**Why some statements use `Resource: "*"`**: `CreateHostedZone`,
`ListHostedZones(ByName)`, `CreateDistribution`, `ListDistributions`,
`CreateOriginAccessControl`, `ListOriginAccessControls`, `CreateFunction`,
`ListFunctions`, `CreateKeyValueStore`, `ListKeyValueStores`, and
`ListAllMyBuckets` all either create a resource that has no ARN yet, or
list resources account-wide by definition — there is no tighter
resource form the AWS API supports for these specific actions. Every
action in this policy that manages an *already-existing* resource
(`Get*`/`Update*`/`Delete*`/`CreateInvalidation`/tagging) is scoped to
that resource type's ARN pattern instead.

**KVS data-plane actions live under a separate IAM namespace**:
`cloudfront-keyvaluestore:*` (reading/writing redirect-map entries) is a
distinct IAM action namespace from `cloudfront:*` (managing the KVS
resource itself — create/describe/delete), even though both sets of
actions apply to the same `key-value-store` ARN path. Both are needed:
the `cloudfront:*` statements let this role create and manage the store
as an AWS resource; the `cloudfront-keyvaluestore:*` statement lets it
actually read/write the redirect entries inside it (used by §4.6's
`aws cloudfront-keyvaluestore update-keys`, which is itself a different
CLI service namespace than plain `aws cloudfront`).

**ACM's region lock uses two mechanisms at once**: `RequestCertificate`/
`ListCertificates` can't be pre-scoped to an ARN (the cert doesn't exist
yet, and listing is account-wide), so they're locked to `us-east-1` via
the `aws:RequestedRegion` condition key. The other four ACM actions *can*
be ARN-scoped, and because ACM certificate ARNs embed the region,
scoping the resource to `arn:aws:acm:us-east-1:...` **is** the region
lock for those — belt-and-suspenders, not redundant: even if a mistake
tried to request a cert elsewhere, both the condition (on request) and
the ARN scope (on every subsequent describe/get/tag/delete/renew) block
it.

**Route 53 record-change scoping stops at `hostedzone/*`, not per-zone
IDs**: `ChangeResourceRecordSets`/`GetHostedZone`/`ListResourceRecordSets`
do support a specific hosted-zone ARN, but naming individual zone IDs
here would mean editing and redeploying this policy every time a new
client is onboarded — defeating the "provision once, reuse forever"
design. `hostedzone/*` already means "every hosted zone this account
owns," which matches this role's entire purpose (managing this account's
client DNS). A stricter per-client variant (literal zone ID in the
resource) is the right move only if a specific narrower principal should
ever be limited to one client's zone.

**The trailing explicit `Deny` is defense-in-depth, not the primary
control** — the primary control is simply never granting `iam`/`ec2`/
`lambda`/`apigateway`/etc. actions (the implicit deny already covers
that). The explicit deny guarantees that if this policy is ever combined
with a broader one on the same principal later, IAM's "explicit deny
always wins" rule keeps this role from reaching those services anyway —
a real technical backstop for the "never widen this role's scope
temporarily" gotcha, not just a written rule.

### 1.4 Create the role and a local CLI profile

```bash
# 1. Create the role with its trust policy
aws iam create-role \
  --role-name website-deploy \
  --assume-role-policy-document file://trust-policy.json \
  --description "Least-privilege role for manual S3+CloudFront+ACM+Route53 client site hosting and deploys"

# 2. Create the customer-managed permissions policy
aws iam create-policy \
  --policy-name WebsiteDeployPolicy \
  --policy-document file://website-deploy-policy.json

# 3. Attach it to the role
aws iam attach-role-policy \
  --role-name website-deploy \
  --policy-arn arn:aws:iam::{{AWS_ACCOUNT_ID}}:policy/WebsiteDeployPolicy
```

Use `create-policy` + `attach-role-policy` (a reusable, versioned
customer-managed policy) rather than `put-role-policy` (an inline policy
welded to the role) — future updates become a clean
`create-policy-version --set-as-default` instead of editing an inline
blob. IAM keeps at most 5 versions of a customer-managed policy; prune
old non-default versions with `delete-policy-version` when updating.

`~/.aws/config`:

```ini
[profile admin]
region = us-east-1
# the existing admin identity already used for this account

[profile site-deploy]
role_arn = arn:aws:iam::{{AWS_ACCOUNT_ID}}:role/website-deploy
source_profile = admin
region = us-east-1
mfa_serial = arn:aws:iam::{{AWS_ACCOUNT_ID}}:mfa/{{IAM_USERNAME}}
```

Drop the `mfa_serial` line if the trust policy's MFA condition was
skipped. Every command below is run with `--profile site-deploy` (or
`export AWS_PROFILE=site-deploy` for the session). A one-off equivalent
without a saved profile: `aws sts assume-role --role-arn ... --role-session-name deploy --profile admin`, exporting the three returned
credentials as environment variables.

## 2. One-time site provisioning

Bucket region is a free choice (only the ACM cert must be `us-east-1`);
default to `us-east-1` for the bucket too unless there's a specific
reason (e.g. data residency) to pick another — CloudFront's edge network
makes bucket region irrelevant to end-user latency.

### 2.1 Clean-URL fix (CloudFront Function)

S3-via-OAC only auto-serves `index.html` for the literal distribution
root — it does not do so for `/about/`-style folder requests the way the
legacy S3 static-website-hosting endpoint did (which isn't usable here
since it can't do HTTPS/OAC). Fix it with one small CloudFront Function,
written once and reused unmodified across every client distribution:

```javascript
function handler(event) {
    var request = event.request;
    var uri = request.uri;

    if (uri.endsWith('/')) {
        request.uri += 'index.html';
    } else if (!uri.substring(uri.lastIndexOf('/') + 1).includes('.')) {
        request.uri += '/index.html';
    }

    return request;
}
```

```bash
aws cloudfront create-function \
  --name clean-url-rewrite \
  --function-config Comment="Append index.html for clean-URL folder requests",Runtime=cloudfront-js-2.0 \
  --function-code fileb://clean-url-rewrite.js
  # capture ETag from the response

aws cloudfront publish-function \
  --name clean-url-rewrite \
  --if-match {{ETAG_FROM_CREATE}}
  # capture the FunctionARN - this is {{FUNCTION_ARN}} below, reused for every site
```

### 2.2 Ordered provisioning steps

```bash
# 1. Create the bucket
aws s3api create-bucket --bucket site-example-com --region us-east-1

# 2. Block all public access
aws s3api put-public-access-block --bucket site-example-com \
  --public-access-block-configuration \
  BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true

# 3. Disable ACLs entirely - correct pairing with an OAC-only access model
aws s3api put-bucket-ownership-controls --bucket site-example-com \
  --ownership-controls Rules=[{ObjectOwnership=BucketOwnerEnforced}]

# 4. Create the Origin Access Control
aws cloudfront create-origin-access-control \
  --origin-access-control-config \
  Name="site-example-com-oac",SigningProtocol=sigv4,SigningBehavior=always,OriginAccessControlOriginType=s3
  # capture the Id -> {{OAC_ID}}

# 5. Request the ACM certificate in us-east-1
aws acm request-certificate --region us-east-1 \
  --domain-name example.com \
  --subject-alternative-names www.example.com \
  --validation-method DNS
  # capture the CertificateArn -> {{CERT_ARN}}

# 6. Create the hosted zone (skip if it already exists from a DNS migration, see section 4)
aws route53 create-hosted-zone \
  --name example.com \
  --caller-reference "example-com-$(date +%s)"
  # capture the Id -> {{HOSTED_ZONE_ID}}

# 7. Read the DNS validation record(s) ACM wants
aws acm describe-certificate --region us-east-1 --certificate-arn {{CERT_ARN}} \
  --query "Certificate.DomainValidationOptions[].ResourceRecord"

# 8. Create each validation CNAME in Route 53 (one change-batch per record, or combine)
aws route53 change-resource-record-sets --hosted-zone-id {{HOSTED_ZONE_ID}} \
  --change-batch file://acm-validation-record.json

# 9. Wait for issuance (can take minutes, occasionally longer)
aws acm wait certificate-validated --region us-east-1 --certificate-arn {{CERT_ARN}}

# 10. Create the CloudFront distribution
aws cloudfront create-distribution --distribution-config file://distribution-config.json
  # capture Id -> {{DISTRIBUTION_ID}}, DomainName -> {{DISTRIBUTION_DOMAIN_NAME}}, ARN

# 11. Tag it for lookup
aws cloudfront tag-resource --resource arn:aws:cloudfront::{{AWS_ACCOUNT_ID}}:distribution/{{DISTRIBUTION_ID}} \
  --tags Items=[{Key=domain,Value=example.com}]

# 12. Attach the bucket policy scoped to this specific distribution
aws s3api put-bucket-policy --bucket site-example-com --policy file://bucket-policy.json

# 13. Create the A/AAAA alias records
aws route53 change-resource-record-sets --hosted-zone-id {{HOSTED_ZONE_ID}} \
  --change-batch file://alias-records.json

# 14. Verify
curl -I https://example.com/
curl -I https://example.com/about/
curl -I https://www.example.com/
```

### 2.3 `distribution-config.json`

```json
{
  "CallerReference": "example-com-2026-09-04",
  "Comment": "example.com",
  "Enabled": true,
  "Aliases": { "Quantity": 2, "Items": ["example.com", "www.example.com"] },
  "DefaultRootObject": "index.html",
  "Origins": {
    "Quantity": 1,
    "Items": [
      {
        "Id": "s3-site-example-com",
        "DomainName": "site-example-com.s3.us-east-1.amazonaws.com",
        "OriginAccessControlId": "{{OAC_ID}}",
        "S3OriginConfig": { "OriginAccessIdentity": "" },
        "ConnectionAttempts": 3,
        "ConnectionTimeout": 10,
        "OriginShield": { "Enabled": false }
      }
    ]
  },
  "DefaultCacheBehavior": {
    "TargetOriginId": "s3-site-example-com",
    "ViewerProtocolPolicy": "redirect-to-https",
    "AllowedMethods": {
      "Quantity": 2,
      "Items": ["GET", "HEAD"],
      "CachedMethods": { "Quantity": 2, "Items": ["GET", "HEAD"] }
    },
    "Compress": true,
    "CachePolicyId": "658327ea-f89d-4fab-a63d-7e88639e58f6",
    "FunctionAssociations": {
      "Quantity": 1,
      "Items": [{ "EventType": "viewer-request", "FunctionARN": "{{FUNCTION_ARN}}" }]
    }
  },
  "CacheBehaviors": { "Quantity": 0 },
  "CustomErrorResponses": { "Quantity": 0 },
  "PriceClass": "PriceClass_100",
  "ViewerCertificate": {
    "ACMCertificateArn": "{{CERT_ARN}}",
    "SSLSupportMethod": "sni-only",
    "MinimumProtocolVersion": "TLSv1.2_2021"
  },
  "Restrictions": { "GeoRestriction": { "RestrictionType": "none", "Quantity": 0 } },
  "HttpVersion": "http2and3",
  "IsIPV6Enabled": true
}
```

`CachePolicyId` `658327ea-f89d-4fab-a63d-7e88639e58f6` is the AWS-managed
`CachingOptimized` policy (1s min / 86400s default / 31536000s max TTL,
no cookies or headers in the cache key) — a sensible default for a
static site with no per-viewer variation. If an older CLI/API rejects
this payload for missing fields, add empty `"Logging"` and `"WebACLId": ""`
blocks; current API versions treat both as optional when omitted.

### 2.4 `bucket-policy.json`

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontServicePrincipalReadOnly",
      "Effect": "Allow",
      "Principal": { "Service": "cloudfront.amazonaws.com" },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::site-example-com/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::{{AWS_ACCOUNT_ID}}:distribution/{{DISTRIBUTION_ID}}"
        }
      }
    }
  ]
}
```

The `AWS:SourceArn` condition is what makes this least-privilege at the
bucket level — a different CloudFront distribution, even one in the same
account, cannot read this bucket.

### 2.5 `alias-records.json`

```json
{
  "Changes": [
    {
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "example.com",
        "Type": "A",
        "AliasTarget": {
          "HostedZoneId": "Z2FDTNDATAQYW2",
          "DNSName": "{{DISTRIBUTION_DOMAIN_NAME}}",
          "EvaluateTargetHealth": false
        }
      }
    },
    {
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "www.example.com",
        "Type": "A",
        "AliasTarget": {
          "HostedZoneId": "Z2FDTNDATAQYW2",
          "DNSName": "{{DISTRIBUTION_DOMAIN_NAME}}",
          "EvaluateTargetHealth": false
        }
      }
    }
  ]
}
```

`Z2FDTNDATAQYW2` is a fixed constant — the same alias hosted-zone ID for
*every* CloudFront distribution, in every account, everywhere. It's not
looked up per-distribution. Add matching `AAAA` entries the same way if
IPv6 alias records are wanted (optional, since `IsIPV6Enabled` on the
distribution already serves IPv6 viewers over the CloudFront domain
either way).

## 3. Routine deploy

```bash
# 1. Long-cache: everything under /assets/
aws s3 sync ./dist/assets s3://site-example-com/assets \
  --delete \
  --cache-control "public, max-age=31536000, immutable" \
  --profile site-deploy

# 2. Short/no-cache: everything else (HTML pages, robots.txt, sitemap.xml, etc.)
aws s3 sync ./dist s3://site-example-com \
  --delete \
  --exclude "assets/*" \
  --cache-control "public, max-age=0, must-revalidate" \
  --profile site-deploy

# 3. Always invalidate
aws cloudfront create-invalidation \
  --distribution-id {{DISTRIBUTION_ID}} \
  --paths "/*" \
  --profile site-deploy
```

This stack has no build step and no content-hashed filenames, so a
changed asset re-uploads under the *same* filename every deploy.
`Cache-Control: immutable, max-age=31536000` is what you want for
caching efficiency between deploys, but cache headers control *duration*,
not correctness — only the invalidation forces CloudFront to re-fetch
from origin immediately, regardless of what the headers say. Step 3 is
what makes step 1's long TTL safe to use at all with unhashed filenames;
don't skip it.

CloudFront invalidation billing is per path *string* submitted, not per
object matched — `"/*"` is one path string, consuming exactly 1 of the
1,000 free monthly invalidation paths regardless of how many files it
touches. For a hand-deployed small site, running it on every deploy is
effectively free.

### `deploy.config.json` (lives in the client site's own repo, not this plugin repo)

```json
{
  "domain": "example.com",
  "bucket": "site-example-com",
  "bucketRegion": "us-east-1",
  "distributionId": "{{DISTRIBUTION_ID}}",
  "hostedZoneId": "{{HOSTED_ZONE_ID}}",
  "certificateArn": "{{CERT_ARN}}",
  "awsProfile": "site-deploy"
}
```

A small `deploy.sh` in the same repo reads this file and runs the three
commands above. If the file is ever lost, recover the distribution ID
with `aws cloudfront list-distributions --query "DistributionList.Items[?Comment=='example.com'].[Id,DomainName]"`
(relies on §2.2 step 11 setting `Comment` to the bare domain) or by
checking tags on candidate distributions (relies on the `domain` tag from
the same step).

## 4. URL redirect map (site relaunch)

When a redesign replaces a site that already has search-engine
rankings, `website-seo`'s migration checklist produces a redirect map
(old URL → new URL). This section covers only how those 301s actually
get served from this stack — see `website-seo` for which URLs redirect
and why.

### 4.1 Why this must be merged into the existing CloudFront Function

Two platform limits make this a modification of the existing
clean-URL-rewrite function (§2.1), not a second function:

- A CloudFront cache behavior allows only **one function association
  per event type** — `viewer-request` already has the clean-URL
  function attached; there's no way to attach a second one alongside it.
- A CloudFront Function can have only **one KeyValueStore association**.

So the redirect lookup runs first, inside the same function; if the
requested URI isn't in the redirect map, execution falls through
unchanged into the existing index.html-append logic.

### 4.2 Why the map lives in a KeyValueStore, not in-code

CloudFront Functions have a **hard, non-adjustable 10 KB total code-size
quota** — AWS's own documentation for this exact quota points at
KeyValueStore as the intended solution for anything beyond trivial
in-code data: *"To store additional data for your CloudFront Functions,
create a key value store and add your key-value pairs."* A real
redirect entry (`"/2019/03/old-post-slug/":"/blog/new-post-slug/",`)
runs roughly 55-70 bytes; after the function's own handler code, that
leaves realistically **100-150 short entries** before risking the hard
limit — too small for a full legacy WordPress site's URL inventory,
with zero headroom for adding more later. A CloudFront KeyValueStore
(KVS) holds up to **5 MB per store** (tens of thousands of entries),
which is what this skill uses instead, always, rather than starting
in-code and migrating later under client pressure.

### 4.3 Merged CloudFront Function

Replaces the function from §2.1 (same name, same `viewer-request`
association — this is an update, not a new function):

```javascript
import cf from 'cloudfront';

async function handler(event) {
    var request = event.request;
    var uri = request.uri;

    // 1. Redirect map lookup (KVS) - runs first, before any URL rewriting
    try {
        var kvsHandle = cf.kvs();
        var newPath = await kvsHandle.get(uri);
        return {
            statusCode: 301,
            statusDescription: 'Moved Permanently',
            headers: { "location": { "value": newPath } }
        };
    } catch (err) {
        // no match in the redirect map - fall through to clean-URL handling
    }

    // 2. Existing clean-URL fix (S3-via-OAC doesn't auto-append index.html)
    if (uri.endsWith('/')) {
        request.uri += 'index.html';
    } else if (!uri.substring(uri.lastIndexOf('/') + 1).includes('.')) {
        request.uri += '/index.html';
    }

    return request;
}
```

Keys must match the *exact* incoming `request.uri`, including a
trailing slash if the old URL had one — the KVS lookup happens before
any slash-normalization, so `/old-page` and `/old-page/` are different
keys if the old site served both.

### 4.4 `redirects.json` (the redirect map, one-time authoring format)

```json
{
  "/old-blog/2019/03/old-post-slug/": "/blog/new-post-slug/",
  "/old-services.html": "/services/",
  "/old-services/web-design.html": "/services/web-design/"
}
```

### 4.5 Create the KVS and wire it into the function

```bash
# 1. Upload redirects.json to a scratch S3 location the KVS import can read
aws s3 cp redirects.json s3://{{IMPORT_BUCKET}}/redirects-kvs-import.json

# 2. Create the KVS, importing the initial map (import only works at creation time)
aws cloudfront create-key-value-store \
  --name redirect-map-example-com \
  --comment "301 redirect map for example.com relaunch" \
  --import-source SourceType=S3,SourceARN=arn:aws:s3:::{{IMPORT_BUCKET}}/redirects-kvs-import.json
  # capture Id/ARN -> {{KVS_ARN}}

# 3. Update the existing clean-url-rewrite function: bump runtime if needed,
#    replace its code (section 4.3 above), and associate the KVS
aws cloudfront update-function \
  --name clean-url-rewrite \
  --if-match {{FUNCTION_ETAG}} \
  --function-config Comment="Clean-URL rewrite + redirect map",Runtime=cloudfront-js-2.0,KeyValueStoreAssociations={Quantity=1,Items=[{KeyValueStoreARN={{KVS_ARN}}}]} \
  --function-code fileb://clean-url-rewrite.js

# 4. Publish
aws cloudfront publish-function --name clean-url-rewrite --if-match {{NEW_ETAG}}
```

### 4.6 Add or change redirects after launch (no redeploy needed)

Uses a **separate CLI service namespace**, `cloudfront-keyvaluestore` —
not `cloudfront` — even though the resource ARN is the same
`key-value-store/*` path. Writes are optimistic-locked by `ETag`:

```bash
aws cloudfront-keyvaluestore describe-key-value-store --kvs-arn {{KVS_ARN}}
# capture ETag

aws cloudfront-keyvaluestore update-keys \
  --kvs-arn {{KVS_ARN}} \
  --if-match {{ETAG}} \
  --puts '[{"Key":"/another-old-path/","Value":"/its/new/path/"}]'
```

Updates propagate to all edge locations in roughly seconds — no
function republish, no cache invalidation, no waiting for the 5-15
minute distribution-propagation window that a `create-distribution`/
`update-distribution` change requires.

### 4.7 Test before publishing, verify after

```bash
# Unit-test the function logic against a sample event, before publishing
aws cloudfront test-function \
  --name clean-url-rewrite \
  --if-match {{ETAG}} \
  --event-object fileb://event-redirect-test.json \
  --stage DEVELOPMENT
```

`event-redirect-test.json` sets `request.uri` to a known old path (see
`functions-event-structure` in AWS's docs for the full event shape);
check the returned `FunctionOutput` shows the expected 301/Location, and
also test a known pass-through URI to confirm clean-URL handling still
works. `test-function` only catches execution errors, not live-
distribution behavior — after publishing, confirm with a real request:

```bash
curl -I https://example.com/old-services.html
# expect: HTTP/2 301, location: https://example.com/services/
```

### 4.8 Cost

Both CloudFront Function invocations and KVS reads (from inside the
function) fall inside AWS's always-free monthly tier at this scale —
2,000,000 invocations/month and 2,000,000 KVS reads/month, no expiry.
KVS write-side API calls (`update-keys`, etc.) are billed per call
(a flat rate, not per-key) — a relaunch's occasional redirect-map
updates cost cents at most, never a real line item.

### 4.9 IAM permissions

The KVS create/manage and `cloudfront-keyvaluestore` data-plane
statements this section's commands need are already included in the
permissions policy in §1.3 (`CloudFrontKVSCreateAndList`,
`CloudFrontKVSManage`, `CloudFrontKVSDataPlane`) — if the role was
created before this section existed, update the policy to the current
version in §1.3 rather than granting KVS access separately.

## 5. Domain DNS migration

### 5.1 Inventory the existing DNS first

Query the domain's *current* authoritative nameservers directly (not a
public resolver, to avoid stale caches) — prefer a zone-file export from
the current host if one's available, since `dig` can only find records
for names you already know to ask about:

```bash
dig @<current-ns> example.com MX
dig @<current-ns> example.com TXT
dig @<current-ns> _dmarc.example.com TXT
dig @<current-ns> example.com A
dig @<current-ns> example.com AAAA
dig @<current-ns> www.example.com CNAME
dig @<current-ns> autodiscover.example.com CNAME   # Microsoft 365 email, if in use
dig @<current-ns> example.com NS
```

Also check for any other subdomains in active use (blog., shop., etc.)
and any DKIM selector TXT records the mail provider documents.

### 5.2 Create the zone and recreate records, in this order

```bash
aws route53 create-hosted-zone --name example.com --caller-reference "example-com-migration-$(date +%s)"
# capture Id -> {{HOSTED_ZONE_ID}}, and the 4 NS values from DelegationSet.NameServers
```

Recreate records **MX and TXT (SPF/DKIM/DMARC) first**, then A/AAAA/CNAME,
then everything else — via `route53 change-resource-record-sets` batches,
same shape as §2.2 step 8. This ordering isn't cosmetic: email is the
single biggest cutover risk, since a missed record silently breaks
inbound or outbound mail with no obvious error at cutover time.

### 5.3 Verify against the new zone before touching the registrar

```bash
aws route53 get-hosted-zone --id {{HOSTED_ZONE_ID}} --query "DelegationSet.NameServers"

dig @<route53-ns-1> example.com MX
dig @<route53-ns-1> example.com TXT
dig @<route53-ns-1> _dmarc.example.com TXT
dig @<route53-ns-1> example.com A
# ...confirm every value matches the §5.1 inventory exactly
```

### 5.4 Cut over

Only once §5.3 is fully clean: update the domain's nameservers at its
existing registrar (in that registrar's own dashboard — entirely outside
AWS and this IAM role) to the 4 Route 53 NS records.

By this point you no longer control the old DNS host, so "lower the TTL
in advance" isn't a lever you can pull retroactively — resolvers
worldwide cache the old NS delegation for however long its TTL says
(commonly 24-48h, sometimes controlled by the registry). The real
mitigation already happened in §5.3: since the new zone is provably
correct before any resolver ever queries it, propagation delay only
affects *when* a resolver picks up the right answer, never whether it's
right once it does. Use `dig +trace example.com` or a multi-location
propagation checker for reassurance if needed, but don't treat a slow
resolver as a broken record.

Keep the old DNS provider/zone active and untouched for 48-72h after
cutover before deleting anything there.

### 5.5 Optional: full registrar transfer

Transferring the domain's *registration* into Route 53 Domains (making
AWS the registrar of record) is a separate, optional, slower follow-up —
it additionally requires unlocking the domain at the current registrar,
obtaining its EPP/auth code, initiating the transfer in Route 53
Domains, and waiting out a transfer window (commonly 5-7 days). None of
that is needed for hosting — the DNS-only migration above is completely
sufficient — so only pursue it if the domain should also change
registrars.

## 6. Gotchas (full list)

- ACM for CloudFront must be requested in `us-east-1`, regardless of
  which region the S3 bucket or the client lives in — a hard CloudFront
  requirement, not a preference.
- S3 bucket names are unique across every AWS account globally — check
  availability before assuming the naming convention is free.
- CloudFront distribution creation/updates propagate to edge locations
  over roughly 5-15+ minutes (sometimes longer) — don't test immediately
  after `create-distribution`/`update-distribution` returns.
- CloudFront invalidation is billed per path string, not per object
  matched — `"/*"` is one unit; a non-issue for a small, manually
  deployed site even run on every deploy.
- The clean-URL/`index.html` fix is specific to S3-via-OAC (the legacy
  S3 static-website endpoint handled this natively, which is exactly why
  it isn't usable here) — the CloudFront Function must be attached on
  *every* new distribution's default cache behavior; easy to forget past
  the first one since it's not part of the "obvious" fields.
- The bucket must have Block Public Access fully on in all four settings,
  relying solely on the OAC-scoped bucket policy for access — turning
  any of the four off "just to test something" defeats the entire point
  of using OAC over a public bucket.
- No content-hashed filenames means long asset cache TTLs are only safe
  because every routine deploy invalidates — skipping the invalidation
  step is the easiest way to ship a change customers don't see for up to
  a year.
- DNS cutover's biggest real risk is a silent email outage from a missed
  MX/SPF/DKIM/DMARC record — verify those first and hardest, before
  anything else in the zone.
- This role must stay scoped to exactly `s3`/`cloudfront`/`acm`/`route53`
  actions as enumerated in §1.3 — never attach a broader managed policy
  "temporarily" for convenience; the explicit `Deny` statement exists
  specifically to make that mistake harder to make by accident.
- A redirect map that's grown past a "handful" of entries must live in
  a KVS, not in the CloudFront Function's own code — the 10 KB code-size
  limit is hard and non-adjustable, with no graceful failure mode when
  exceeded (the function just won't publish). Start with KVS for a real
  site relaunch rather than planning to "migrate later."
- A cache behavior allows only one function per event type and a
  function only one KVS association — the redirect lookup must be
  merged into the existing clean-URL function (§4.1), never added as a
  second `viewer-request` function.
- KVS lookup keys must match the incoming `request.uri` exactly,
  including a trailing slash — a redirect map entry for `/old-page`
  won't match a request for `/old-page/`, and vice versa.

## 7. Full verification checklists

**Provisioning**:
- [ ] Bucket exists, Block Public Access is fully `true` on all four
  settings, ownership is `BucketOwnerEnforced`.
- [ ] OAC exists and is attached to the distribution's S3 origin; the
  bucket policy's `AWS:SourceArn` names this distribution's ARN
  specifically (not a wildcard).
- [ ] ACM cert status is `ISSUED` (not `PENDING_VALIDATION`) and was
  requested in `us-east-1`.
- [ ] Distribution status is `Deployed` (not `InProgress`) before
  declaring done.
- [ ] CloudFront Function is `PUBLISHED` (not just `DEVELOPMENT` stage)
  and associated on the default cache behavior's `viewer-request` event.
- [ ] Route 53 zone has both the ACM validation CNAME(s) and the final
  A/AAAA alias records.
- [ ] `curl -I` against the domain root, a real folder-style page, and
  `www` all return `200` over HTTPS with a valid cert.

**Routine deploy**:
- [ ] Both `sync` passes completed with no errors; spot-checked one
  asset's and one HTML page's `Cache-Control` header via `curl -I`.
- [ ] Invalidation reached `Completed` status
  (`aws cloudfront get-invalidation`).
- [ ] The live site reflects the new content in a real browser, not just
  via `curl` (a browser's own cache can mask a broken deploy).

**URL redirect map**:
- [ ] `aws cloudfront test-function` passes for both a sample redirected
  URI and a sample pass-through URI before publishing.
- [ ] After publish, a real `curl -I` against several redirected URLs
  returns `301` with the correct `Location`, and a non-redirected page
  still resolves correctly (clean-URL fallback still works).
- [ ] The KVS's key count/spot-checked entries match the source
  `redirects.json` — a bad S3 import can silently truncate or malform
  entries.
- [ ] The role's permissions policy includes the `CloudFrontKVS*`
  statements from §1.3 (not just the original policy without them, if
  the role predates this section).

**DNS migration**:
- [ ] Every record from the pre-migration inventory (§5.1) exists in the
  new Route 53 zone with matching values — MX/SPF/DKIM/DMARC checked
  explicitly, not just "the site resolves."
- [ ] Verified via `dig @<route53-ns>` directly, before the registrar's
  nameservers were touched.
- [ ] Registrar nameservers updated only after the above passed clean.
- [ ] Old DNS host/zone left active and unchanged for the 48-72h safety
  window post-cutover.
