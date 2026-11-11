# adapters/opencode

## Overview

Experimental installer for running EGM as an OpenCode V1 bundle. `install.py`
copies the portable EGM runtime into a new, self-contained installation directory
and generates an `opencode.json` that registers the reviewer subagents, the `egm`
orchestrator, and one slash command per skill — without modifying any existing
user configuration. It is a standalone CLI, not part of the installed plugin
runtime.

## Interface

- `install.py`
  - `name(path: Path) -> str` — extracts the canonical `egm-*` name from a
    persona/skill Markdown front-matter block without a YAML dependency; raises
    `ValueError` if absent.
  - `install(destination: Path) -> Path` — builds a fresh OpenCode installation at
    `destination` and returns the written `opencode.json` path.
  - CLI: `python install.py <destination>` — prints the generated config path.
  - Module constants: `ROOT` (repo root, two parents up), `HOST_VERSION`
    (`"1.18.29"`), `REVIEWERS` (the eight read-only reviewer persona names).

## Invariants

- `install` refuses to overwrite: it raises `FileExistsError` if `destination`
  already exists or is a symlink (`install.py:33`).
- The resolved `destination` must not contain template/glob characters
  (`* ? [ ] { } ! @ ` `` $ ``, backslash, newlines), must not contain `.env.`
  (OpenCode's `.env` read rules would deny bundle reads), and must not be inside
  `ROOT` (`install.py:37`, `install.py:43`, `install.py:45`).
- Reviewer agents are registered `mode: subagent` with a deny-by-default
  permission set: `read` allows files except `*.env`/`*.env.*` (keeping
  `*.env.example`), `grep` is denied (it bypasses read rules), `glob` is allowed,
  and `external_directory` is denied except the bundle tree (`install.py:69`).
- The `egm` orchestrator is `mode: primary`, may run only `egm-*` skills, and may
  dispatch only the registered reviewer subagents as tasks (`install.py:94`).
- Each skill is exposed as a command bound to the `egm` agent; its `template`
  embeds the shared bundle-root context so paths resolve against the bundle
  (`install.py:117`).

## Side Effects

- Creates `destination/`, copies `skills/`, `agents/`, `resources/`, `docs/` and
  the shared root files (`plugin.json`, `LICENSE`, `README.md`, `TESTING.md`) into
  `destination/bundle/` (`install.py:50`).
- Writes `destination/opencode.json`, `destination/installation.json` (with per-file
  SHA-256 digests and `status: "experimental"`), and symlinks under
  `destination/skills/` pointing at each bundled skill directory (`install.py:116`,
  `install.py:122`, `install.py:132`).
- Reads the repository runtime tree and `plugin.json` metadata.

## Verification

- Run: `pytest` (from the repository root; covered by
  `tests/unit/test_opencode_adapter.py`)
