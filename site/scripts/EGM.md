# site/scripts

## Overview

Build-time and test automation for the EGM landing site. These Node ES modules
regenerate committed images and verify the built `dist/` output (documentation
routes, Markdown exports, repository link rewriting, and privacy-preserving
analytics behavior). They are invoked through the `site/package.json` scripts and
drive a headless Chrome via `puppeteer-core`.

## Interface

- `capture.mjs` — top-level script (no exports). Serves `../dist/` over an
  ephemeral `127.0.0.1` HTTP server and screenshots the Open Graph image plus the
  hero/full/mobile views in light and dark schemes. Invoked by `npm run capture`
  (which runs `astro build` first).
- `check-docs.mjs` — top-level assertion script (no exports). Validates
  `../src/lib/repository-markdown.mjs` and `github-stars.ts`, then checks every
  documentation route in `../dist/` for a single `<h1>`, discoverable Markdown,
  clean Markdown exports, resolvable `/_astro/*` assets, `llms*.txt`, the Pagefind
  index, and redirect targets. Invoked by `npm run check:docs`.
- `verify-analytics.mjs` — top-level assertion script (no exports). Loads built
  HTML in Chrome with fully intercepted Google responses (no live network) and
  asserts GA4 consent gating. Invoked by `npm run test:browser`; requires the path
  to an analytics-disabled build as `process.argv[2]`.
- `og.html` — static HTML source rendered by `capture.mjs` into `public/og.png`
  (not executable code).

## Invariants

- `capture.mjs` binds the local server to `127.0.0.1` on an OS-assigned port
  (`listen(0, '127.0.0.1')`) and only serves files resolved under `dist/`.
- `check-docs.mjs` asserts each route has exactly one page title and that Markdown
  exports omit page chrome (`<nav>`); it fails the process on any violation.
- `verify-analytics.mjs` intercepts every request so nothing reaches Google or any
  external host; it asserts GA loads only after an explicit `ACCEPTED` consent and
  that revocation sets `window['ga-disable-<id>']` and blocks later collection.
- The browser self-reports `googleResponses: 'intercepted fixtures; no live vendor
  verification'` — these tests do not validate real Google endpoints
  (`verify-analytics.mjs:185`).

## Side Effects

- `capture.mjs` overwrites `../public/og.png` and `../screenshots/*.png`; opens a
  local HTTP server and a Chrome process (closed in `finally`).
- `check-docs.mjs` and `verify-analytics.mjs` read from `../dist/` (they require a
  prior `astro build`) and launch/close a Chrome process.
- All three launch Chrome via `puppeteer-core` using `channel: 'chrome'`, honoring
  `CHROME_PATH`; `github-stars.ts` under test issues one `fetch` to the GitHub API,
  stubbed in `check-docs.mjs`.

## Verification

- Run: `npm test` (from `site/`), with `npm run check:docs`, `npm run test:browser`,
  and `npm run capture` exercising these scripts specifically
