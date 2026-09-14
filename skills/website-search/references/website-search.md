# Website Search (Pagefind) — Reference Implementation

Complete, copy-pasteable code for the pattern summarized in
`SKILL.md`, adapted from a working production implementation. Anything
in `{{DOUBLE_BRACES}}` is site-specific — fill it in from the site's own
data (its category list, page count, etc.) rather than hand-typing it,
since it must stay in sync with the actual content tree.

## 1. Header markup (every page)

Drop this into the `.site-header__actions` flex group from
`website-build-standards` — logo stays the other, only, sibling of this
`<div>`:

```html
<div class="site-header__actions">
  <form class="site-search" action="/search/" role="search">
    <label class="visually-hidden" for="site-search-input">Search {{SITE_NAME}}</label>
    <input class="site-search__input" id="site-search-input" name="q" type="search" placeholder="Search...">
  </form>

  <!-- any other header buttons (feedback, contact, etc.) go here,
       between search and the theme toggle -->

  <button type="button" class="theme-toggle" aria-label="Toggle color theme">
    <svg class="theme-toggle__icon--dark" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path></svg>
    <svg class="theme-toggle__icon--light" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><circle cx="12" cy="12" r="4"></circle><path d="M12 2v2M12 20v2M4.93 4.93l1.41 1.41M17.66 17.66l1.41 1.41M2 12h2M20 12h2M6.34 17.66l-1.41 1.41M19.07 4.93l-1.41 1.41"></path></svg>
  </button>
</div>
```

This is generated once per page in whatever templating the site's own
build uses (or hand-copied identically onto every page, if the site has
no build step of its own beyond the Pagefind reindex). Every page needs
identical header markup — the JS below finds elements by id/class, not
by page.

## 2. Theme toggle + `/` shortcut + search-input wiring (`main.js`, every page)

The theme-toggle *behavior* is `website-build-standards`'s own snippet
(reproduced here for completeness since it now pairs with real button
markup); the `/` shortcut is new, specific to having a search input in
the header:

```js
/* Theme toggle */
(function () {
  var toggle = document.querySelector(".theme-toggle");
  if (!toggle) return;

  toggle.addEventListener("click", function () {
    var current =
      document.documentElement.getAttribute("data-theme") ||
      (matchMedia("(prefers-color-scheme: dark)").matches ? "dark" : "light");
    var next = current === "dark" ? "light" : "dark";
    document.documentElement.setAttribute("data-theme", next);
    try {
      localStorage.setItem("theme", next);
    } catch (e) {
      /* private browsing / storage blocked - toggle still works this load */
    }
  });
})();

/* Header search box: plain <form action="/search/">, so Enter/submit
   navigates to the dedicated search results page natively (no JS
   required for that part). "/" focuses it from anywhere on the site,
   unless focus is already in a text input/textarea/select or a
   contenteditable region. */
(function () {
  var input = document.getElementById("site-search-input");
  if (!input) return;

  document.addEventListener("keydown", function (event) {
    if (event.key !== "/" || event.metaKey || event.ctrlKey || event.altKey) return;
    var active = document.activeElement;
    var tag = active && active.tagName;
    if (tag === "INPUT" || tag === "TEXTAREA" || tag === "SELECT" || (active && active.isContentEditable)) return;
    event.preventDefault();
    input.focus();
  });
})();
```

Also needs the no-flash theme script in `<head>` before the stylesheet
(already documented in `website-build-standards`'s monochrome section —
reproduced here only because the toggle button depends on it):

```html
<script>
  (function () {
    var stored = localStorage.getItem("theme");
    if (stored) document.documentElement.setAttribute("data-theme", stored);
  })();
</script>
```

## 3. Content pages: index scoping + category tag

Every indexable page's content wrapper:

```html
<article class="doc-content" data-pagefind-body>
  <span hidden data-pagefind-filter="category">{{PAGE_CATEGORY}}</span>
  {{PAGE_CONTENT_HTML}}
</article>
```

- `data-pagefind-body` must wrap **only** this — never the header, nav,
  sidebar, or footer.
- `{{PAGE_CATEGORY}}` is whatever the site calls this page's top-level
  section (its nav group, its folder's first path segment, etc.) — must
  exactly match one of the category values used in the `/search/` page's
  checkbox list below (case-sensitive).
- A page that should never be searchable (an unlisted style page, an
  error page) simply has no `data-pagefind-body` at all — once any page
  on the site uses the attribute, every page without it is automatically
  excluded from the index.

## 4. `/search/index.html` (static shell)

Generated once (by the site's own build tooling, from its actual
category list) — the category `<li>`s below are static HTML; only the
counts and results are filled in client-side:

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Search — {{SITE_NAME}}</title>
  <meta name="robots" content="noindex">
  <script>
    (function () {
      var stored = localStorage.getItem("theme");
      if (stored) document.documentElement.setAttribute("data-theme", stored);
    })();
  </script>
  <link rel="stylesheet" href="/assets/css/main.css">
</head>
<body>
  <a class="skip-link" href="#main-content">Skip to main content</a>

  <!-- site header, identical to every other page (see section 1) -->

  <div class="site-body">
    <div class="doc-layout search-layout">
      <aside class="search-filters" aria-label="Filter by category">
        <h2 class="search-filters__heading">Categories</h2>
        <ul class="search-filters__list" id="search-filters-list">
          <!-- one <li> per category, e.g.: -->
          <li class="search-filters__item">
            <label>
              <input type="checkbox" name="category" value="{{CATEGORY_NAME}}">
              <span class="search-filters__label">{{CATEGORY_NAME}}</span>
              <span class="search-filters__count" data-category="{{CATEGORY_NAME}}"></span>
            </label>
          </li>
          <!-- ...repeat per category... -->
        </ul>
      </aside>

      <main id="main-content" class="doc-content-wrapper">
        <h1 id="search-heading" class="search-heading"></h1>
        <div class="search-toolbar">
          <p id="search-status" class="search-status" aria-live="polite"></p>
          <label class="search-sort">
            Sort by
            <select id="search-sort-select" aria-label="Sort results by">
              <option value="relevance">Relevance</option>
              <option value="title">Title (A–Z)</option>
            </select>
          </label>
        </div>
        <div id="search-results-list" class="search-results-list"></div>
      </main>
    </div>
  </div>

  <!-- site footer, identical to every other page -->

  <script src="/assets/js/main.js"></script>
  <script src="/assets/js/search-page.js"></script>
</body>
</html>
```

## 5. `assets/js/search-page.js` (loaded only on `/search/`)

```js
(function () {
  var headingEl = document.getElementById("search-heading");
  var statusEl = document.getElementById("search-status");
  var resultsEl = document.getElementById("search-results-list");
  var filtersList = document.getElementById("search-filters-list");
  var headerInput = document.getElementById("site-search-input");
  var sortSelect = document.getElementById("search-sort-select");
  if (!resultsEl || !filtersList) return;

  var pagefindPromise = null;
  function ensurePagefind() {
    if (!pagefindPromise) {
      pagefindPromise = import("/pagefind/pagefind.js").catch(function () {
        return null;
      });
    }
    return pagefindPromise;
  }

  function getState() {
    var params = new URLSearchParams(location.search);
    var sort = params.get("sort");
    return {
      q: params.get("q") || "",
      categories: params.getAll("category"),
      sort: sort === "title" ? "title" : "relevance",
    };
  }

  function setUrl(state) {
    var params = new URLSearchParams();
    if (state.q) params.set("q", state.q);
    state.categories.forEach(function (c) {
      params.append("category", c);
    });
    if (state.sort && state.sort !== "relevance") params.set("sort", state.sort);
    var next = location.pathname + (params.toString() ? "?" + params.toString() : "");
    history.pushState(state, "", next);
  }

  function sortResults(results, sort) {
    if (sort !== "title") return results; // "relevance" = Pagefind's own ranked order
    var titled = results.slice();
    titled.sort(function (a, b) {
      var titleA = (a.meta && a.meta.title) || "";
      var titleB = (b.meta && b.meta.title) || "";
      return titleA.localeCompare(titleB, undefined, { sensitivity: "base" });
    });
    return titled;
  }

  function renderResults(results) {
    resultsEl.innerHTML = "";
    results.forEach(function (r) {
      var a = document.createElement("a");
      a.className = "search-result-card";
      a.href = r.url;

      var title = document.createElement("h2");
      title.className = "search-result-card__title";
      title.textContent = r.meta && r.meta.title ? r.meta.title : r.url;
      a.appendChild(title);

      var url = document.createElement("span");
      url.className = "search-result-card__url";
      url.textContent = r.url;
      a.appendChild(url);

      if (r.excerpt) {
        var excerpt = document.createElement("p");
        excerpt.className = "search-result-card__excerpt";
        // Pagefind generates this excerpt HTML itself (its own <mark>
        // highlights around our own indexed content) - not third-party
        // or user-supplied markup.
        excerpt.innerHTML = r.excerpt;
        a.appendChild(excerpt);
      }

      resultsEl.appendChild(a);
    });
  }

  function updateFacetCounts(categoryCounts) {
    var counts = categoryCounts || {};
    filtersList.querySelectorAll(".search-filters__count").forEach(function (el) {
      var name = el.getAttribute("data-category");
      var n = counts[name] || 0;
      el.textContent = n ? "(" + n + ")" : "";
      var item = el.closest(".search-filters__item");
      if (item) item.classList.toggle("search-filters__item--empty", n === 0);
    });
  }

  function syncCheckboxes(selected) {
    filtersList.querySelectorAll('input[name="category"]').forEach(function (cb) {
      cb.checked = selected.indexOf(cb.value) !== -1;
    });
  }

  function runSearch(state, pushUrl) {
    if (headerInput) headerInput.value = state.q;
    syncCheckboxes(state.categories);
    if (sortSelect) sortSelect.value = state.sort;

    if (!state.q) {
      headingEl.textContent = "Search";
      statusEl.textContent = "Enter a search term above to get started.";
      resultsEl.innerHTML = "";
      updateFacetCounts({});
      return;
    }

    headingEl.textContent = 'Search results for "' + state.q + '"';
    statusEl.textContent = "Searching…";

    ensurePagefind().then(function (pagefind) {
      if (!pagefind) {
        statusEl.textContent = "Search is unavailable right now.";
        return;
      }
      pagefind
        .search(state.q, { filters: { category: state.categories } })
        .then(function (search) {
          updateFacetCounts(search.totalFilters && search.totalFilters.category);
          return Promise.all(search.results.map(function (r) { return r.data(); })).then(function (results) {
            statusEl.textContent =
              results.length === 0
                ? "No results found."
                : results.length + " result" + (results.length === 1 ? "" : "s") + " found.";
            renderResults(sortResults(results, state.sort));
          });
        });
    });

    if (pushUrl) setUrl(state);
  }

  filtersList.addEventListener("change", function () {
    var state = getState();
    var selected = [];
    filtersList.querySelectorAll('input[name="category"]:checked').forEach(function (cb) {
      selected.push(cb.value);
    });
    runSearch({ q: state.q, categories: selected, sort: state.sort }, true);
  });

  if (sortSelect) {
    sortSelect.addEventListener("change", function () {
      var state = getState();
      runSearch({ q: state.q, categories: state.categories, sort: sortSelect.value }, true);
    });
  }

  window.addEventListener("popstate", function () {
    runSearch(getState(), false);
  });

  runSearch(getState(), false);
})();
```

**Why `filters: { category: state.categories }` is always passed
explicitly**, even as `[]`: empirically, omitting the `filters` option
entirely leaves `search.totalFilters` empty, while explicitly passing an
empty per-key array still populates `search.totalFilters.category` with
a live per-category count for the current query, without narrowing
results. That's what powers the `(21)`-style counts next to each
checkbox — don't "simplify" this to only set `filters` when something's
checked.

## 6. `tools/reindex.js` (Node, build-time)

```js
// Rebuilds /pagefind/ from whatever HTML is currently on disk. Uses
// Pagefind's Node indexing API (createIndex/addHTMLFile/writeFiles) with
// our own file walk, not the `pagefind` CLI's own directory walk - the
// CLI would also pick up anything else living under the same repo root
// (a tools/ cache, node_modules fixtures).
const fs = require("fs");
const path = require("path");

const REPO_ROOT = path.resolve(__dirname, "..");
const SKIP_DIRS = new Set(["tools", ".git", "node_modules"]);

function walk(dir, out) {
  for (const entry of fs.readdirSync(dir, { withFileTypes: true })) {
    if (entry.isDirectory()) {
      if (SKIP_DIRS.has(entry.name)) continue;
      walk(path.join(dir, entry.name), out);
    } else if (entry.name === "index.html" || entry.name === "404.html") {
      out.push(path.join(dir, entry.name));
    }
  }
  return out;
}

async function reindex() {
  const pagefind = await import("pagefind"); // ESM-only, hence dynamic import
  const files = walk(REPO_ROOT, []);

  const { index, errors: createErrors } = await pagefind.createIndex({});
  if (createErrors && createErrors.length) {
    console.error("Pagefind createIndex errors:", createErrors);
  }

  for (const file of files) {
    const rel = path.relative(REPO_ROOT, file).replace(/\\/g, "/");
    const content = fs.readFileSync(file, "utf8");
    const { errors } = await index.addHTMLFile({ sourcePath: rel, content });
    if (errors && errors.length) {
      console.error(`Pagefind indexing error for ${rel}:`, errors);
    }
  }

  const { errors: writeErrors } = await index.writeFiles({
    outputPath: path.join(REPO_ROOT, "pagefind"),
  });
  if (writeErrors && writeErrors.length) {
    console.error("Pagefind writeFiles errors:", writeErrors);
  }
  await pagefind.close();

  // Drop Pagefind's own prebuilt widget bundles - dead weight, since this
  // skill always builds a fully custom UI on the low-level API instead.
  const pagefindDir = path.join(REPO_ROOT, "pagefind");
  for (const f of fs.readdirSync(pagefindDir)) {
    if (/^pagefind-(ui|component-ui|modular-ui)\.(js|css)$/.test(f)) {
      fs.rmSync(path.join(pagefindDir, f));
    }
  }

  console.log(`Reindexed ${files.length} pages into /pagefind/.`);
  return files.length;
}

module.exports = { reindex };

if (require.main === module) {
  reindex().catch((e) => {
    console.error(e);
    process.exit(1);
  });
}
```

Wire this into the site's deploy script as its **first** step, always —
so a deploy after a hand-edited page reindexes automatically, with
nothing extra to remember. Run `node tools/reindex.js` directly any time
you want a refreshed local `/pagefind/` without doing a full deploy.

`tools/package.json` needs:

```json
{
  "devDependencies": {
    "pagefind": "^1.5.2"
  }
}
```

## 7. CSS

Written against `website-build-standards`'s token names
(`--foreground`, `--muted-foreground`, `--border`, `--border-strong`,
`--background`, `--muted`, `--destructive`, `--space-*`) — drops in
unchanged whether the site uses the default token set or the locked
monochrome one.

```css
/* --- Header actions row --- */
.site-header__actions {
  display: flex;
  align-items: center;
  gap: var(--space-sm);
  min-width: 0;
}

/* --- Search (header box - submits to /search/) --- */
.site-search { flex: 1; min-width: 0; max-width: 26rem; margin: 0; }
.site-search__input {
  width: 100%;
  padding: var(--space-xs) var(--space-sm);
  border: 1px solid var(--border-strong);
  border-radius: 4px;
  background: var(--background);
  color: var(--foreground);
  font: inherit;
}

/* --- Theme toggle ---
   Shows the icon for the mode you'd switch TO. No stored preference =
   follows the OS/browser setting; clicking sets an explicit data-theme
   and remembers it. */
.theme-toggle {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 2.5rem;
  height: 2.5rem;
  border: 0;
  background: transparent;
  cursor: pointer;
  color: var(--foreground);
  flex-shrink: 0;
}
.theme-toggle svg { width: 1.25rem; height: 1.25rem; }
.theme-toggle__icon--dark { display: block; }
.theme-toggle__icon--light { display: none; }

:root[data-theme="dark"] .theme-toggle__icon--dark { display: none; }
:root[data-theme="dark"] .theme-toggle__icon--light { display: block; }

@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) .theme-toggle__icon--dark { display: none; }
  :root:not([data-theme="light"]) .theme-toggle__icon--light { display: block; }
}

/* --- Search results page (/search/) ---
   Desktop/tablet: always a two-column grid. Add a stacked fallback under
   your own smallest supported breakpoint if the site supports phone
   widths (website-build-standards' reference site does not). */
.doc-layout.search-layout {
  grid-template-columns: 14rem minmax(0, 1fr);
}
.search-filters__heading {
  font-size: 1rem;
  font-weight: 700;
  margin: 0 0 var(--space-sm);
}
.search-filters__list {
  list-style: none;
  margin: 0;
  padding: 0;
}
.search-filters__item label {
  display: flex;
  align-items: baseline;
  gap: var(--space-xs);
  padding: var(--space-xs) 0;
  cursor: pointer;
  font-size: 0.9375rem;
}
.search-filters__label { flex: 1; }
.search-filters__count { color: var(--muted-foreground); font-size: 0.8125rem; }
.search-filters__item--empty .search-filters__label,
.search-filters__item--empty .search-filters__count {
  color: var(--muted-foreground);
  opacity: 0.6;
}

.search-heading { margin-bottom: var(--space-xs); }
.search-toolbar {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: var(--space-sm);
  margin-bottom: var(--space-lg);
}
.search-status { color: var(--muted-foreground); }
.search-sort {
  display: inline-flex;
  align-items: center;
  gap: var(--space-xs);
  font-size: 0.875rem;
  color: var(--muted-foreground);
  flex-shrink: 0;
}
.search-sort select {
  padding: var(--space-xs) var(--space-sm);
  border: 1px solid var(--border-strong);
  border-radius: 4px;
  background: var(--background);
  color: var(--foreground);
  font: inherit;
  font-size: 0.875rem;
}

.search-results-list {
  display: flex;
  flex-direction: column;
  gap: var(--space-md);
}
.search-result-card {
  display: block;
  padding: var(--space-md);
  border: 1px solid var(--border);
  border-radius: 6px;
  text-decoration: none;
  color: inherit;
}
.search-result-card:hover,
.search-result-card:focus-visible {
  border-color: var(--border-strong);
  background: var(--muted);
}
.search-result-card__title {
  font-size: 1.0625rem;
  font-weight: 600;
  color: var(--foreground);
  margin: 0 0 var(--space-xs);
}
.search-result-card__url {
  display: block;
  font-size: 0.8125rem;
  color: var(--muted-foreground);
  margin-bottom: var(--space-xs);
}
.search-result-card__excerpt {
  font-size: 0.9375rem;
  color: var(--muted-foreground);
  line-height: 1.5;
  margin: 0;
}
.search-result-card__excerpt mark {
  background: none;
  color: var(--foreground);
  font-weight: 700;
}
```

## 8. Deploy caching addendum

Extends `website-deployment`'s two-pass sync (long-cache immutable for
`/assets/`, short/no-cache for everything else):

- Add `pagefind/fragment/`, `pagefind/index/`, `pagefind/filter/` to the
  **long-cache immutable** pass — genuinely content-hashed, same
  guarantee as the existing `/assets/` pass.
- Add `pagefind/pagefind.js`, `pagefind/pagefind-worker.js`,
  `pagefind/pagefind-entry.json`, `pagefind/*.pf_meta`,
  `pagefind/wasm.*.pagefind` to the **short-cache/must-revalidate** pass
  — same filename every rebuild, so long-caching these would keep
  serving stale search code after a reindex.
- Run `node tools/reindex.js` as the first step of the deploy script,
  before either sync pass, so the index on disk always matches the
  content about to ship.
