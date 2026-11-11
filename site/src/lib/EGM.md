# site/src/lib

## Overview

Shared build-time library modules for the site. They rewrite repository-relative
Markdown links into absolute GitHub URLs and fetch the repository star count at
build time. These run during Astro prerendering, not in the browser.

## Interface

- `repository-markdown.mjs`
  - `repositoryLink(url, file, image = false) -> string` — resolves a
    repository-relative `url` (referenced from source `file`) to an absolute
    `github.com/.../blob/main/` URL, a `tree/main/` URL for directories, or a
    `raw.githubusercontent.com/.../main/` URL when `image` is true. Pass-throughs,
    absolute URLs, anchors, and bracketed text are returned unchanged.
  - `transformRepositoryMarkdown(tree, file) -> void` — mutates an mdast tree in
    place: drops a leading `# ` heading for `SKILL.md` sources and rewrites
    `link`/`image`/`definition` node URLs via `repositoryLink`.
  - `remarkRepositoryLinks()` — remark plugin wrapper applying
    `transformRepositoryMarkdown` using `file.path`.
- `github-stars.ts`
  - `githubStars` — an exported `Promise<number | null>` resolving to the repo's
    `stargazers_count`, or `null` on any failure.

## Invariants

- `repositoryLink` throws if the resolved target escapes the repository root, has a
  non-`github.com` origin, or leaves the `/blob/main/` path prefix
  (`repository-markdown.mjs:15`).
- URLs matching a scheme, a leading `/`, a leading `#`, or containing `[`/`]` are
  returned unchanged (`repository-markdown.mjs:9`).
- The module computes the repo root as `resolve(process.cwd(), '..')`, relying on
  npm running from `site/` (`repository-markdown.mjs:5`).
- `image` definitions are rewritten to `raw.githubusercontent.com` only when the
  definition identifier is referenced by an `imageReference` node
  (`repository-markdown.mjs:28`, `repository-markdown.mjs:36`).
- `githubStars` resolves to `null` unless `stargazers_count` is a safe,
  non-negative integer (`github-stars.ts:11`).

## Side Effects

- `repositoryLink` / `transformRepositoryMarkdown` call `statSync(local)` to detect
  directories — a filesystem read during build (`repository-markdown.mjs:18`).
- `github-stars.ts` performs one `fetch` to `https://api.github.com` per build with
  a 5-second `AbortSignal.timeout`, optionally sending a bearer `GITHUB_TOKEN`.

## Verification

- Run: `npm test` (from `site/`); link rewriting and star parsing are exercised by
  `../../scripts/check-docs.mjs`
