# site/src/components/gb

## Overview

Native-element wrappers for gBreeze (`@material/web@2.5.0` labs) UI primitives.
Each component renders a plain HTML element carrying the exact class names the
corresponding `<md-gb-*>` element emits inside its shadow root, so styling comes
from the package stylesheets imported in `../../layouts/Base.astro` without
shipping the custom elements.

## Interface

- `Button.astro` — gBreeze button as a `<button>` or `<a>` (when `href` is set).
  - `Props`: `color?: 'filled'|'elevated'|'tonal'|'outlined'|'text'` (default
    `'text'`), `size?: 'xs'|'sm'|'md'|'lg'` (default `'sm'`), `href?`, `target?`,
    `class?`, `label?`, `popovertarget?`, `'aria-haspopup'?: 'menu'`, and
    `data-*` attributes.
- `IconButton.astro` — gBreeze icon button as a `<button>` or `<a>`.
  - `Props`: `color?: 'filled'|'tonal'|'outlined'|'standard'` (default
    `'standard'`), `size?: 'xs'|'sm'|'md'` (default `'sm'`), `label: string`
    (required), `href?`, `target?`, `class?`, `popovertarget?`, `aria-haspopup?`,
    `aria-expanded?`, `aria-controls?`, and `data-*` attributes. Expects an inline
    `<svg>` in the default slot.
- `Symbol.astro` — inlines a Material Symbol SVG by name.
  - `Props`: `{ name: string; class?: string }`.

## Invariants

- `Button`/`IconButton` compose class names from a fixed ripple/focus-ring/variant
  pattern and set `rel="noopener"` whenever `target === '_blank'`
  (`Button.astro:21`, `IconButton.astro:21`).
- `IconButton` requires a `label` (it is a required prop).
- `Symbol.astro` throws `Unknown Material Symbol: <name>` if
  `../../assets/symbols/<name>.svg` is not present (`Symbol.astro:4`), and rewrites
  all `fill` attributes to `currentColor`.

## Side Effects

- `Symbol.astro` eagerly imports every `../../assets/symbols/*.svg` as raw text at
  build time (`import.meta.glob`, `Symbol.astro:3`). No runtime side effects.

## Verification

- Run: `npm run check` (from `site/`, `astro check`) and `npm test`
