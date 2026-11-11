# site/src/components

## Overview

The Astro presentation components that build the landing page. The top-level
components are page sections (hero, model explainer, loop, skills grid, install,
nav, footer) plus shared brand primitives (the EGM mark and per-skill icons). Two
subdirectories hold lower-level building blocks: `gb/` (gBreeze element wrappers)
and `docs/` (Starlight head override).

## Interface

- `Hero.astro`, `Model.astro`, `Loop.astro`, `Skills.astro`, `Install.astro`,
  `Nav.astro`, `Footer.astro` — page-section components composed by
  `../pages/index.astro`. They take no required props.
- `Mark.astro` — the EGM logo SVG.
  - `Props`: `variant?: 'split'|'elephant'|'goldfish'` (default `'split'`),
    `size?`, `width?`, `height?`, `class?`, `aria-hidden?`. The `variant` selects
    the SVG `viewBox` (`Mark.astro:19`).
- `SkillIcon.astro` — renders one of the seven skill icons.
  - `Props`: `skill: 'bootstrap'|'brainstorm'|'draft'|'resolve'|'drift_report'|'review'|'review_loop'`
    (keys of the `icons` map), imported from `../../../skills/*/assets/icon.svg`.
- Subdirectory contracts:
  - [gb](./gb/EGM.md) — gBreeze button, icon button, and Material Symbol wrappers.
  - [docs](./docs/EGM.md) — Starlight `Head` override for font preloading.

## Invariants

- `Mark.astro` defaults width/height to 28 for `split` and 56 otherwise, and marks
  the SVG `aria-hidden` by default (`Mark.astro:10`).
- `SkillIcon.astro`'s `skill` prop is typed as `keyof typeof icons`, so only the
  seven known skill names are valid (`SkillIcon.astro:11`).
- Section components are presentational: they import brand/gBreeze primitives
  (`Mark`, `SkillIcon`, `Button`, `IconButton`) and render static markup.

## Side Effects

- Build-time only: `SkillIcon.astro` imports the skill icon SVGs as URLs from the
  repository `skills/` tree (`SkillIcon.astro:1`). No runtime side effects.

## Verification

- Run: `npm run check` (from `site/`, `astro check`) and `npm test`
