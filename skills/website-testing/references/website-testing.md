# Website Testing — Complete Playbook

Full setup and code for the automated test harness described in
`SKILL.md`. Everything here is dev-only tooling — none of it is deployed
to the static host.

## 1. Dev-only tooling setup

### `package.json`

```json
{
  "name": "site-tests",
  "private": true,
  "version": "1.0.0",
  "scripts": {
    "test": "playwright test",
    "test:a11y": "playwright test tests/accessibility.spec.js",
    "test:links": "playwright test tests/links.spec.js",
    "test:ui": "playwright test --ui"
  },
  "devDependencies": {
    "@playwright/test": "^1.48.0",
    "@axe-core/playwright": "^4.10.0",
    "serve": "^14.2.0"
  }
}
```

Install with `npm install`, then `npx playwright install --with-deps` once
to fetch the browser binaries.

### `.gitignore` additions

```
node_modules/
test-results/
playwright-report/
```

### Deploy-exclusion patterns

Whatever ships the site to its static host must exclude the test harness.
Check the project's actual deploy mechanism and apply the matching
pattern — don't assume one:

| Deploy mechanism         | Exclude via                                                        |
| ------------------------ | ------------------------------------------------------------------- |
| `rsync`                  | `rsync -a --exclude={node_modules,tests,package*.json,playwright.config.js} ./ user@host:/path` |
| FTP/SFTP client          | An ignore list / sync profile excluding the same paths              |
| Static host build setting | A "publish directory" set to the site root only (e.g. `public/`, or the folder-per-page root), never the repo root |
| Git-based deploy (host builds from a branch) | A `.deployignore`/host-specific ignore file, or keep the test harness in a separate branch/folder the host doesn't build from |

## 2. `playwright.config.js`

```js
// playwright.config.js
const { defineConfig, devices } = require('@playwright/test');

module.exports = defineConfig({
  testDir: './tests',
  fullyParallel: true,
  retries: process.env.CI ? 1 : 0,
  reporter: 'list',
  use: {
    baseURL: 'http://localhost:4173',
    trace: 'retain-on-failure',
  },
  webServer: {
    command: 'npx serve . -l 4173',
    url: 'http://localhost:4173',
    reuseExistingServer: !process.env.CI,
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
  ],
});
```

Add `{ name: 'firefox', use: { ...devices['Desktop Firefox'] } }` and a
WebKit project if cross-browser coverage is worth the extra CI time for
this project; chromium-only is a reasonable default for a smoke suite.

## 3. `tests/page-health.spec.js`

```js
// tests/page-health.spec.js
const { test, expect } = require('@playwright/test');
const { XMLParser } = require('fast-xml-parser'); // add as a devDependency
const fs = require('fs');
const path = require('path');

function pagesFromSitemap() {
  const xml = fs.readFileSync(path.join(__dirname, '../sitemap.xml'), 'utf8');
  const parsed = new XMLParser().parse(xml);
  const entries = parsed.urlset.url;
  return (Array.isArray(entries) ? entries : [entries]).map((e) => new URL(e.loc).pathname);
}

for (const pagePath of pagesFromSitemap()) {
  test(`page health: ${pagePath}`, async ({ page, baseURL }) => {
    await page.goto(pagePath);

    // Title
    await expect(page).toHaveTitle(/.+/);

    // Meta description
    const description = page.locator('meta[name="description"]');
    await expect(description).toHaveCount(1);
    const content = await description.getAttribute('content');
    expect(content && content.length).toBeGreaterThan(0);
    expect(content.length).toBeLessThanOrEqual(160);

    // Self-referencing canonical
    const canonical = page.locator('link[rel="canonical"]');
    await expect(canonical).toHaveCount(1);
    const href = await canonical.getAttribute('href');
    expect(new URL(href).pathname).toBe(pagePath);

    // Open Graph set
    for (const prop of ['og:title', 'og:description', 'og:type', 'og:url', 'og:image']) {
      await expect(page.locator(`meta[property="${prop}"]`)).toHaveCount(1);
    }

    // Twitter Card
    await expect(page.locator('meta[name="twitter:card"]')).toHaveCount(1);

    // JSON-LD present and parses (where expected — adjust per page type)
    const jsonLdBlocks = await page.locator('script[type="application/ld+json"]').allTextContents();
    expect(jsonLdBlocks.length).toBeGreaterThan(0);
    for (const block of jsonLdBlocks) {
      expect(() => JSON.parse(block)).not.toThrow();
    }
  });
}
```

This mirrors `website-seo`'s assertion list exactly — extend the JSON-LD
check per page type (Organization on the homepage, Article on blog posts)
rather than inventing new requirements here.

## 4. `tests/links.spec.js`

```js
// tests/links.spec.js
const { test, expect } = require('@playwright/test');

async function crawl(page, baseURL, startPath = '/') {
  const visited = new Set();
  const queue = [startPath];
  const broken = [];

  while (queue.length) {
    const current = queue.shift();
    if (visited.has(current)) continue;
    visited.add(current);

    const response = await page.goto(current);
    if (!response || response.status() >= 400) {
      broken.push({ page: current, status: response ? response.status() : 'no response' });
      continue;
    }

    const hrefs = await page.$$eval('a[href]', (as) => as.map((a) => a.getAttribute('href')));
    for (const href of hrefs) {
      if (!href || href.startsWith('http') || href.startsWith('mailto:') || href.startsWith('tel:')) continue;
      const [pathPart] = href.split('#');
      if (pathPart && !visited.has(pathPart)) queue.push(pathPart);
    }
  }
  return broken;
}

test('no broken internal links', async ({ page, baseURL }) => {
  const broken = await crawl(page, baseURL);
  expect(broken, JSON.stringify(broken, null, 2)).toEqual([]);
});

test('in-page anchors resolve', async ({ page }) => {
  await page.goto('/');
  const anchorHrefs = await page.$$eval('a[href^="#"]', (as) => as.map((a) => a.getAttribute('href')));
  for (const href of anchorHrefs) {
    const id = href.slice(1);
    if (!id) continue;
    await expect(page.locator(`#${id}`)).toHaveCount(1);
  }
});

test('page assets return 200', async ({ page, request }) => {
  await page.goto('/');
  const assetUrls = await page.$$eval('link[rel="stylesheet"], script[src], img[src]', (els) =>
    els.map((el) => el.getAttribute('href') || el.getAttribute('src')).filter(Boolean)
  );
  for (const url of assetUrls.filter((u) => !u.startsWith('http'))) {
    const res = await request.get(url);
    expect(res.status(), url).toBeLessThan(400);
  }
});
```

### `scripts/check-external-links.js` (manual, non-blocking)

```js
// scripts/check-external-links.js
// Run manually: node scripts/check-external-links.js
// Not wired into `npm test` — external-link failures are advisory, not a
// defect in this site (rate limiting, transient outages, etc.).
const { chromium } = require('@playwright/test');

(async () => {
  const browser = await chromium.launch();
  const page = await browser.newPage();
  await page.goto('http://localhost:4173/');
  const externalHrefs = await page.$$eval('a[href^="http"]', (as) => as.map((a) => a.href));

  for (const url of [...new Set(externalHrefs)]) {
    try {
      const res = await fetch(url, { method: 'HEAD' });
      console.log(`${res.status} ${url}`);
    } catch (e) {
      console.log(`FAILED ${url} (${e.message})`);
    }
  }
  await browser.close();
})();
```

Sample output:

```
200 https://www.linkedin.com/company/example
999 https://www.linkedin.com/company/example  <- LinkedIn blocks HEAD from bots; expected, not a real break
404 https://old-partner-site.example.com/page  <- worth investigating
```

## 5. `tests/contact-form.spec.js`

```js
// tests/contact-form.spec.js
const { test, expect } = require('@playwright/test');

test('contact form: real submission', async ({ page }) => {
  await page.goto('/contact/');

  await page.getByLabel('Name').fill('Test User');
  await page.getByLabel('Email').fill('test@example.com');
  await page.getByLabel('Message').fill('Automated test submission.');
  // Honeypot field intentionally left untouched — filling it would trip
  // the anti-spam check the same way a bot would.

  await page.getByRole('button', { name: /send|submit/i }).click();
  await expect(page.getByRole('status')).toContainText(/thank you|message sent/i);
});

test('contact form: CI variant with mocked relay', async ({ page }) => {
  // Fast and network-independent for CI, but this alone does NOT catch
  // the honeypot/autofill failure mode described in the
  // contact-form-integration skill — keep the real-submission test above
  // running periodically as well.
  await page.route('**/relay-endpoint', (route) =>
    route.fulfill({ status: 200, body: JSON.stringify({ ok: true }) })
  );

  await page.goto('/contact/');
  await page.getByLabel('Name').fill('Test User');
  await page.getByLabel('Email').fill('test@example.com');
  await page.getByLabel('Message').fill('Automated test submission.');
  await page.getByRole('button', { name: /send|submit/i }).click();
  await expect(page.getByRole('status')).toContainText(/thank you|message sent/i);
});
```

## 6. `tests/accessibility.spec.js`

```js
// tests/accessibility.spec.js
const { test, expect } = require('@playwright/test');
const AxeBuilder = require('@axe-core/playwright').default;
const fs = require('fs');
const path = require('path');

const pages = ['/', '/contact/', '/services/']; // or derive from sitemap.xml as in page-health.spec.js

for (const pagePath of pages) {
  test(`accessibility: ${pagePath}`, async ({ page }) => {
    await page.goto(pagePath);
    const results = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa', 'wcag21aa'])
      // Example documented suppression — only ever suppress with a reason:
      // .exclude('#third-party-embed') // vendor widget outside our control, tracked in TICKET-123
      .analyze();

    const seriousOrWorse = results.violations.filter((v) => ['serious', 'critical'].includes(v.impact));
    expect(seriousOrWorse, JSON.stringify(seriousOrWorse, null, 2)).toEqual([]);
  });
}
```

## 7. `tests/responsive.spec.js`

```js
// tests/responsive.spec.js
const { test, expect } = require('@playwright/test');

const viewports = [
  { name: 'mobile', width: 375, height: 812 },
  { name: 'tablet', width: 768, height: 1024 },
  { name: 'desktop', width: 1440, height: 900 },
];
const pages = ['/', '/contact/', '/services/'];

for (const pagePath of pages) {
  for (const vp of viewports) {
    test(`no horizontal overflow: ${pagePath} @ ${vp.name}`, async ({ page }) => {
      await page.setViewportSize({ width: vp.width, height: vp.height });
      await page.goto(pagePath);
      const overflow = await page.evaluate(() => document.documentElement.scrollWidth - document.documentElement.clientWidth);
      expect(overflow).toBeLessThanOrEqual(0);
    });

    // Optional, opt-in only — pixel screenshot diffing is high-maintenance
    // (fonts/rendering vary across machines) and prone to false positives.
    // test(`visual: ${pagePath} @ ${vp.name}`, async ({ page }) => {
    //   await page.setViewportSize({ width: vp.width, height: vp.height });
    //   await page.goto(pagePath);
    //   await expect(page).toHaveScreenshot(`${pagePath.replace(/\//g, '_')}-${vp.name}.png`);
    // });
  }
}
```

## 8. Optional: `lighthouserc.json`

```json
{
  "ci": {
    "collect": {
      "staticDistDir": ".",
      "url": ["http://localhost/index.html", "http://localhost/contact/index.html"]
    },
    "assert": {
      "assertions": {
        "categories:performance": ["warn", { "minScore": 0.8 }],
        "categories:accessibility": ["error", { "minScore": 0.9 }],
        "categories:seo": ["warn", { "minScore": 0.9 }]
      }
    },
    "upload": { "target": "temporary-public-storage" }
  }
}
```

Run with `npx lhci autorun` (requires `@lhci/cli` as a devDependency). For
interpreting Core Web Vitals scores and thresholds, see `website-seo`
rather than duplicating that guidance here.

## 9. Optional: `.github/workflows/test.yml`

Illustrative only — adapt to the project's actual CI provider, or omit
entirely if the project has no CI yet and "run locally before deploy" is
sufficient.

```yaml
name: Site tests
on: [pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npm test
```

## 10. Full verification checklist

- [ ] `npm install` — confirm only devDependencies are added, nothing the
      production site needs at runtime.
- [ ] Read the actual deploy config/script and confirm it excludes
      `tests/`, `package.json`, `package-lock.json`, `playwright.config.js`,
      `node_modules/` (see the exclusion table in §1).
- [ ] `npm test` passes locally.
- [ ] `npm run test:a11y` passes, or every failure has a documented,
      justified suppression.
- [ ] `npm run test:links` passes (internal links only — external links
      are checked separately via `scripts/check-external-links.js` and are
      advisory).
- [ ] Every URL in `sitemap.xml` is covered by `page-health.spec.js`.
- [ ] The contact form has also been submitted once by hand, in a real
      browser with autofill available, per `contact-form-integration`'s
      gotcha — a passing automated test alone is not sufficient sign-off.
- [ ] If CI is configured, confirm it blocks merges on
      page-health/links/form/accessibility failures but does not block on
      the external-link script or any opt-in visual-diff test.

## 11. Out of scope

- SEO tag content, strategy, structured-data types to use, and ongoing SEO
  cadence — see `website-seo`.
- Building or wiring the contact form to a relay backend — see
  `contact-form-integration`.
- Homepage/copy structure and conversion strategy — see `website-sales-tool`.
- The production stack, folder layout, and the no-build-step rule this
  test tooling must not violate — see `website-build-standards`.
- Legal terms content — see `terms-of-use-website` / `terms-of-use-software`.
