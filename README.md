# web-builder-skills

Internal Claude Code plugin bundling seven reusable "web builder"
playbooks as Skills — vanilla HTML/CSS/JS build standards, SEO, homepage
sales copy, contact-form integration, automated testing, and South
Africa-first Terms of Use templates (website + software). Originally
drafted in the `VCS_Website` project; this repo is the maintained,
cross-project source going forward.

**This is a private, internal tool.** It is never published to
Anthropic's public plugin marketplace and is not intended for anyone
outside this project's own use.

## What's in it

Seven skills, one plugin (`skills/<name>/SKILL.md` + a
`references/<name>.md` with the full playbook):

- **`website-build-standards`** — the foundational stack/folder-structure
  standard (vanilla HTML/CSS/JS, folder-per-page mirroring the nav, one
  central stylesheet) and the checklist for stripping WordPress/CMS
  fingerprints out of a migrated site.
- **`website-seo`** — technical/content SEO setup and ongoing cadence.
- **`website-sales-tool`** — turning a homepage into a sales-pitch funnel.
- **`contact-form-integration`** — wiring a site's contact form to a
  shared relay backend.
- **`website-testing`** — automated Playwright/axe-core testing (page
  health, broken links, contact-form, accessibility, responsive smoke
  checks) as dev-only tooling that never ships to the static host.
- **`terms-of-use-website`** — a South Africa-first Website Terms of Use
  template.
- **`terms-of-use-software`** — a South Africa-first Software Terms of
  Use template (Standard App licensing + bespoke development).

Skills auto-activate when a request matches their `description`
trigger phrases; any skill can also be invoked explicitly by name (e.g.
`/website-seo`).

## Install (once per machine)

```
/plugin marketplace add <owner>/<repo>
/plugin install web-builder-skills@web-builder-skills
```

Installed plugins are available across every project on that machine —
no per-project setup needed after the first install.

## Updating

1. Edit the relevant `skills/<name>/references/*.md` (or `SKILL.md`).
2. Commit and push.
3. Bump `version` in `.claude-plugin/plugin.json` (semver).
4. Optionally tag the release: `git tag vX.Y.Z && git push --tags`.
5. On any machine with it installed: `/plugin marketplace update
   web-builder-skills` then `/plugin update web-builder-skills`.

## Structure

```
.claude-plugin/
  plugin.json        - plugin manifest
  marketplace.json    - marketplace manifest (this repo is its own marketplace)
skills/
  website-build-standards/
  website-seo/
  website-sales-tool/
  contact-form-integration/
  website-testing/
  terms-of-use-website/
  terms-of-use-software/
```
