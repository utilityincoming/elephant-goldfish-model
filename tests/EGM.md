# tests Blueprint

## System Overview

The EGM test suite. It verifies the repository's packaging, host-adapter, trace,
and authoring/policy contracts offline, with an opt-in layer of native host
probes. `pytest` runs only the `tests/unit` tree by default; integration probes
and the Node-based CI helpers run separately.

## Local Interface & Invariants

- `conftest.py` — session-scoped fixtures shared by the suite:
  - `repo_root` — the repository root (`parents[1]` of the file).
  - `discovered_skills` — all non-hidden directories under `skills/`, sorted;
    asserts at least one skill exists.
- `pr_triage_post.cjs` — Node `node:test` harness that loads a GitHub Actions
  script body from stdin (fd 0) and exercises the PR triage comment logic in a
  temp dir.
- `scope_advisory.cjs` — Node `node:test` harness that evaluates the scope-advisory
  Actions script against a stubbed `github`/`context`/`core`, keyed by the
  `<!-- egm-scope-advisory -->` marker.
- Default test discovery is `testpaths = ["tests/unit"]`, `python_files =
  ["test_*.py"]`, `python_functions = ["test_*"]` (`pyproject.toml`).

## Subdirectories & System Boundaries

- unit: [./unit/EGM.md](./unit/EGM.md) — the offline, default-run unit suite.
- integration: [./integration/EGM.md](./integration/EGM.md) — opt-in native host
  probes, gated by `EGM_*` env flags.

## Global Invariants

- The default suite is deterministic and offline; any model/network access lives
  in `tests/integration` behind explicit opt-in flags.
- Skill-aware tests enumerate skills through `discovered_skills`, so new skills are
  covered without editing the tests.

## Global Side Effects

- Python tests write only to `tmp_path`; some unit modules and the integration
  probes spawn subprocesses (local scripts by default, host CLIs when opted in).
- The `.cjs` harnesses read a script body from stdin and run under Node's test
  runner; they create and clean up their own temp dirs.

## System Verification

- Run: `pytest` (unit, default). Opt-in: `pytest tests/integration` with
  `EGM_*_LIVE` set. Node helpers run under `node --test` with the Actions script
  piped to stdin.
