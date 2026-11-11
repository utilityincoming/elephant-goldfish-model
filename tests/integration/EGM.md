# tests/integration

## Overview

Opt-in integration probes that exercise native reviewer isolation and delegation
against real host CLIs (Claude, Codex, OpenCode, Copilot, Cursor). They are
excluded from the default `pytest` run (`testpaths` is `tests/unit` only) and are
gated behind `EGM_*` environment variables and `pytest.mark.skipif`, so they never
run — or touch a host — unless explicitly enabled.

## Interface

- `test_claude.py` — opt-in Claude plugin-path and native reviewer isolation probe.
- `test_codex.py` — opt-in native Codex tests; no API use or config changes unless
  enabled.
- `copilot_controlled_probe.py` — native reviewer isolation via a scripted local
  provider.
- `opencode_native_probe.py` — native OpenCode delegation with a scripted local
  model.
- `cursor_child_reads.py` — correlates native Cursor `ReadFile` calls/successes in
  one child store.
- `opencode_trace_verifier.py` — checks required reads in an exported OpenCode
  1.18.29 child session.
- Non-contract fixture: `opencode-ci/` holds only `package.json` pinning
  `opencode-linux-x64@1.18.29` (plus its lockfile); no source code.

## Invariants

- Live/native execution is opt-in: guarded by `EGM_CLAUDE_LIVE`, `EGM_CODEX_LIVE`,
  `EGM_CHILD_CONTROLLED`, and related flags via `pytest.mark.skipif`; absent the
  flag, the test skips.
- Probes assert reviewer isolation using sentinel/secret markers
  (`EGM_PARENT_ONLY_SENTINEL`, `EGM_SYNTHETIC_SECRET`) that must not leak into a
  child reviewer session.
- Host configuration is respected, not mutated, unless the opt-in flag is set
  (e.g. `CLAUDE_CONFIG_DIR`, `CODEX_HOME`, `EGM_CODEX_RELEASE_ZIP`).

## Side Effects

- Subprocess: spawns real host CLIs when enabled (`test_claude.py`,
  `test_codex.py`, `copilot_controlled_probe.py`, `opencode_native_probe.py`).
- Network: only when a live flag is set (`urllib` in `test_codex.py`; `requests`
  in `opencode_native_probe.py`); the default state makes no calls.
- Filesystem: writes to `tmp_path`; reads exported host session stores.

## Verification

- Run (opt-in): `pytest tests/integration` with the relevant `EGM_*_LIVE`
  environment variable(s) set; skipped by default
