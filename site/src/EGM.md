# site/src

## Overview

The Astro application source for the EGM landing site: the content-collection
configuration, route pages, the base layout, presentation components, and shared
build-time library modules. `styles/` and `assets/` hold CSS and static media
(not source-code contracts).

## Interface

- `content.config.ts` — defines the `docs` content collection, loaded via glob
  from `../docs/site/**/*.md` with the Starlight `docsSchema`
  (`content.config.ts:6`).
- Subdirectory contracts:
  - [pages](./pages/EGM.md) — `/` landing route and the per-doc Markdown export route.
  - [layouts](./layouts/EGM.md) — the `Base` HTML shell and gated analytics.
  - [components](./components/EGM.md) — page sections, brand marks, and gBreeze/docs primitives.
  - [lib](./lib/EGM.md) — repository link rewriting and the GitHub star fetch.
- Non-contract directories: `styles/` (CSS), `assets/` (fonts and Material Symbol
  SVGs).

## Invariants

- The `docs` collection sources Markdown from the repository `docs/site/` tree and
  strips the `docs/site/` prefix to generate entry ids (`content.config.ts:10`).
- Collection ids feed the dynamic Markdown route in `pages/[...slug].md.ts`.

## Side Effects

- Build-time: the content loader reads `../docs/site/**/*.md`; components and lib
  modules read SVGs, fonts, and (via `lib/github-stars.ts`) the GitHub API.
- No direct runtime side effects beyond the analytics gated in `layouts/Base.astro`.

## Verification

- Run: `npm run check` (from `site/`, `astro check`) and `npm test`
