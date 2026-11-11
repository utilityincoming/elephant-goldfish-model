# site/src/pages

## Overview

Astro route endpoints for the site. One route renders the marketing landing page;
the other emits a plain-text Markdown representation of every documentation entry
for machine/LLM consumption.

## Interface

- `index.astro` — the `/` landing page. Composes `Base` with `Nav`, `Hero`,
  `Model`, `Loop`, `Skills`, `Install`, and `Footer`, and injects WebSite
  JSON-LD structured data.
- `[...slug].md.ts` — dynamic `APIRoute` emitting `text/markdown` per docs entry.
  - `getStaticPaths()` — one path per `docs` collection entry, keyed by `entry.id`.
  - `GET` — parses the entry body, applies `transformRepositoryMarkdown`, and
    returns `# <title>\n\n<description>\n\n<body>` as `text/markdown; charset=utf-8`.

## Invariants

- `index.astro` passes a fixed `title` and `description` into `Base`
  (`index.astro:11`).
- `[...slug].md.ts` rewrites repository links against the entry's absolute
  `filePath` before stringifying (`[...slug].md.ts:18`).
- The Markdown output always begins with an H1 built from `entry.data.title`
  (`[...slug].md.ts:19`), satisfying the checks in `../../scripts/check-docs.mjs`.

## Side Effects

- Both routes are prerendered at build time into static output under `dist/`.
- `[...slug].md.ts` reads the `docs` content collection
  (`getCollection('docs')`) defined in `../content.config.ts`.

## Verification

- Run: `npm test` (from `site/`); route output is validated by
  `../../scripts/check-docs.mjs`
