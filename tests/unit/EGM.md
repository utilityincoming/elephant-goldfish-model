# tests/unit

## Overview

The offline unit test suite — the suite `pytest` runs by default
(`testpaths = ["tests/unit"]`). These tests assert packaging, adapter, trace,
frontmatter, link, and policy contracts without contacting any host model or
network, so they are deterministic and safe to run in CI.

## Interface

Each module is a `test_*.py` file of `test_*` functions (pytest). By subject:

- Packaging & releases: `test_packaging.py`, `test_releases.py`,
  `test_harnesses.py`, `test_ci_workflow.py`.
- Host adapter contracts: `test_copilot_adapter.py`, `test_cursor.py`,
  `test_cursor_child_reads.py`, `test_opencode_adapter.py`.
- Trace / evidence verifiers: `test_claude_traces.py`, `test_codex_traces.py`,
  `test_opencode_trace_verifier.py`.
- Content & authoring contracts: `test_frontmatter.py`, `test_link_integrity.py`,
  `test_markdown_tables.py`, `test_skill_load_limits.py`,
  `test_scenario_coverage.py`, `test_style_and_boundaries.py`.
- CI / PR tooling: `test_pr_triage_review.py` (covers `.github/scripts/review_pr.py`,
  workflow hardening, CODEOWNERS), `test_scope_advisory.py`.

## Invariants

- The suite is offline: modules documented as "offline" make no model or live
  network calls; `requests`/`urllib` usage is routed through `monkeypatch` or local
  harnesses (e.g. `test_scope_advisory.py` runs the scope workflow against an
  offline GitHub API harness).
- Several modules "fail closed": incomplete evidence is treated as failure
  (`test_cursor_child_reads.py`, `test_opencode_trace_verifier.py`).
- Tests discover skills through the shared `discovered_skills` fixture (see
  `../EGM.md`), so adding a skill directory extends coverage automatically.

## Side Effects

- Filesystem: many modules write to pytest `tmp_path` only.
- Subprocess: `test_ci_workflow.py`, `test_copilot_adapter.py`, `test_cursor.py`,
  `test_cursor_child_reads.py`, `test_scope_advisory.py`, and
  `test_pr_triage_review.py` spawn local scripts/processes (no host CLIs).
- No persistent state outside the temp dirs; no outbound network.

## Verification

- Run: `pytest` (from the repository root)
