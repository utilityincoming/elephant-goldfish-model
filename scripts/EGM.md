# scripts

## Overview

Repository release tooling. These Python scripts assemble the shippable EGM
runtime bundles (per host) and the versioned, checksummed artifacts for a draft
release. They are build/packaging utilities, run manually or from CI; they are
not part of the installed plugin runtime.

## Interface

- `pack_plugin.py`
  - `build(root: Path, target: str, output: Path) -> tuple[Path, Path]` — copies
    the allow-listed runtime files for `target` (`"claude"` or `"openai"`) into a
    new `egm-<target>/` folder and a reproducible `egm-<target>.zip`; returns
    `(folder, archive)`.
  - CLI: `--target {claude,openai}` (keys of `HOST`), `--output <dir>`.
  - Module constants defining the bundle contract: `SHARED`, `HOST`, `SKILLS`,
    `REQUIRED_SHARED`.
- `prepare_release.py`
  - `version_for_tag(root: Path, tag: str) -> str` — validates `vMAJOR.MINOR.PATCH`
    and that every manifest in `HOST_MANIFESTS` declares that version; returns the
    version without the leading `v`.
  - `prepare(root: Path, tag: str, output: Path, commit: str) -> None` — builds
    both host archives via `build`, writes `SHA256SUMS` and `RELEASE_NOTES.md`.
  - CLI: `--tag <vX.Y.Z>`, `--commit <40-hex-sha>`, `--output <dir>`.
  - Imports `build` from `pack_plugin`.

## Invariants

- `build` refuses to run if the output folder or ZIP already exists, or if the
  output path would contain the source tree (`pack_plugin.py:62`, `pack_plugin.py:64`).
- Every path in `REQUIRED_SHARED` plus the target's host manifest must be present,
  or `build` raises `ValueError` (`pack_plugin.py:77`).
- Runtime packages reject `hooks`, and `mcpServers`/`mcp`/`apps` metadata
  (`pack_plugin.py:83`, `pack_plugin.py:86`).
- Manifest references (`agents`, icons, OpenAI `composerIcon`/`logo`) must be
  relative, contain no `..`, and resolve to a bundled file (`pack_plugin.py:104`).
- Symlinks anywhere in a runtime path are rejected; resolved paths must stay under
  `root` (`pack_plugin.py:70`, `pack_plugin.py:73`).
- `version_for_tag` rejects tags with leading zeroes and any manifest whose
  `version` differs from the tag (`prepare_release.py:32`).
- `prepare` requires `commit` to be exactly 40 lowercase hex digits and does not
  verify it identifies the checkout (`prepare_release.py:58`).

## Side Effects

- Creates the `--output` directory tree, runtime folders, and ZIP archives on disk.
- `build` writes files with mode `0o644` and fixed ZIP timestamps (1980-01-01) for
  reproducibility; uses a temporary dir and `hardlink_to` for the final archive.
- `prepare` writes `SHA256SUMS` and `RELEASE_NOTES.md`; on any exception it
  `shutil.rmtree`s the output directory (`prepare_release.py:90`).
- Reads manifest and runtime source files from the repository root.

## Verification

- Run: `pytest` (from the repository root; unit tests live in `tests/unit/`)
