# site/src/layouts

## Overview

The base HTML document shell for every page. It imports the gBreeze
(`@material/web` labs) and site stylesheets, sets document metadata (SEO, Open
Graph, Twitter cards, theme colors, fonts), and conditionally bootstraps
privacy-gated Google Analytics.

## Interface

- `Base.astro` — layout component.
  - `Props`: `{ title: string; description: string }`.
  - Renders `<html>`/`<head>`/`<body>`, a skip link, and a default `<slot />`.

## Invariants

- Analytics is emitted only when `PUBLIC_GA_MEASUREMENT_ID` is set (`Base.astro:55`)
  AND, at runtime, `window.location.hostname === 'egm.dev'` (`Base.astro:60`), so
  forks and local/preview builds never load Google scripts.
- GA starts disabled: `window['ga-disable-<id>'] = true` and the `gtag` shim drops
  `event`/`config` calls until consent is `ACCEPTED` (`Base.astro:62`,
  `Base.astro:65`).
- Consent defaults deny all storage; `allow_google_signals` and
  `allow_ad_personalization_signals` are set false (`Base.astro:71`).
- Only an allow-listed set of query params (UTM, `gclid`, `dclid`, `gclsrc`) is
  kept in the reported `page_location`; the hash is stripped (`Base.astro:82`).
- The canonical URL and OG image are derived from `Astro.site` (`Base.astro:23`).

## Side Effects

- Browser-only (inside the gated inline script): mutates `window.dataLayer`,
  injects the Google Cookie Notification Bar CSS/JS from `www.gstatic.com` and the
  GA4 tag from `www.googletagmanager.com`, and sends measurement requests after an
  affirmative consent.
- Build-time: imports stylesheets and preloads display/mono fonts via
  `astro:assets`.

## Verification

- Run: `npm test` (from `site/`); the consent-gating behavior is asserted by
  `../../scripts/verify-analytics.mjs` (`npm run test:browser`)
