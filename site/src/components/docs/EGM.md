# site/src/components/docs

## Overview

Documentation-site integration component. Overrides the Starlight `<Head>` so the
docs pages preload the same custom fonts used by the rest of the site.

## Interface

- `Head.astro` — renders Starlight's default `Head` (via `<slot name="..." />`
  passthrough) and adds `<Font>` preload links for the display and mono fonts.
  Takes no component props of its own.

## Invariants

- Wraps, rather than replaces, `@astrojs/starlight/components/Head.astro`, so
  Starlight's default head metadata is preserved (`Head.astro:1`).

## Side Effects

- Build-time font registration through `astro:assets`. No runtime side effects.

## Verification

- Run: `npm run check` (from `site/`, `astro check`) and `npm test`
