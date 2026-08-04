# `site/` — the public web presence (dialcoffee.site)

**This directory is in use. Do not delete it.** It is the source of truth for
everything served at **https://dialcoffee.site**:

- `index.html`, `privacy.html`, `terms.html`, `support.html` — the marketing +
  legal pages (App Store review links to the privacy/terms/support URLs).
- `favicon*`, `apple-touch-icon.png`, `icon-192/512.png`, `site.webmanifest`,
  `favicon.ico` — browser/PWA assets requested from the site root.
- `assets/` — screenshots used by the marketing page.
- `CNAME` — the custom domain (`dialcoffee.site`) for GitHub Pages.
- `triage/index.html` — the **operator triage page** (moderator-only tool for
  approving auto-seeded catalog rows). See below.

## How it deploys — GitHub Pages, NOT Supabase

The private app repo's `site/` is the source. `tools/publish-site.sh` mirrors
this whole directory into **`tjgrist/dialcoffee-site`**, and GitHub Pages serves
that repo at `dialcoffee.site`. Nothing here is served from Supabase Storage or
from an Edge Function (a Supabase function can't return HTML — it's rewritten to
`text/plain` under a sandbox CSP).

```sh
# publish everything in site/ to the live domain
tools/publish-site.sh
```

If `git push` inside that script fails from a sandboxed environment (a known
issue, documented in the script), publish via the GitHub API instead — or, from
a Claude Code session, `add_repo tjgrist/dialcoffee-site`, clone it, copy the
changed files in, and push (the added repo gets its own working credentials).
**Always verify against the live site afterwards** (`curl -s https://dialcoffee.site …`);
publishes have silently no-op'd before.

## The operator triage page (`triage/index.html`)

It is **generated**, not hand-edited. Source: `supabase/functions/_shared/triage-html.ts`.
Regenerate + publish:

```sh
# 1. Bake the function URL into site/triage/index.html
deno run --allow-write --allow-read tools/triage/render_operator_page.ts \
  https://tddwrnpeeerbwyymkujy.supabase.co/functions/v1/operator-triage \
  site/triage/index.html
# 2. Publish site/ (above)
tools/publish-site.sh
```

The page is a self-contained SPA that calls the **`operator-triage` Edge
Function** (the JSON API: login/refresh, queue snapshots, approve/exclude
actions) cross-origin. Full runbook: `tools/triage/README.md` and
`docs/CATALOG-REVIEW.md`.
