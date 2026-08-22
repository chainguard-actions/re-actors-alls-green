<!-- markdownlint-disable -->

# Hardening Report: re-actors--alls-green/v1.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **re-actors--alls-green/v1.3.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in the workflow directly interpolate `${{ }}` expressions, violating rule (a). (1) `run: exit ${{ matrix.rc }}` — `matrix.rc` is interpolated directly into a shell command. (2) `run: echo 'results=${{ toJSON(needs.*.result) }}'` — `needs.*.result` is interpolated directly into a shell command (repeated across 8 jobs). (3) `run: echo 'outputs=${{ toJSON(steps.check.outputs) }}'` — `steps.check.outputs` is interpolated directly into a shell command (repeated across 8 jobs). Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the template engine before the shell ever sees it, allowing metacharacters to escape quoting.

Locations:

- `.github/workflows/self-smoke-test-action.yml:42`
- `.github/workflows/self-smoke-test-action.yml:54`
- `.github/workflows/self-smoke-test-action.yml:63`

### unpinned-uses (severity: high)

The workflow uses `actions/checkout@v3` (a mutable tag, not a full 40-character SHA commit hash) in 9 separate jobs: wait-for-success, wait-for-failure, wait-for-both, allow-some-failures, allow-some-failures-str, allow-failures-n-skips, allow-skips, allow-partials, and check. A tag reference can be moved by the upstream repository owner at any time, enabling a supply-chain attack. Each reference should be pinned to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v3`.

Locations:

- `.github/workflows/self-smoke-test-action.yml:53`

### missing-permissions (severity: medium)

The workflow file `.github/workflows/self-smoke-test-action.yml` has no top-level `permissions:` key and none of the individual jobs define their own `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g. `write` access to contents, pull-requests, etc.). A minimal `permissions:` block (e.g. `permissions: {}` or specific scopes like `contents: read`) should be added at the top level or to each job.

Locations:

- `.github/workflows/self-smoke-test-action.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings in .github/workflows/self-smoke-test-action.yml: (1) Added `permissions: {}` at the top level to address missing-permissions. (2) Pinned all 9 `actions/checkout@v3` references to the full SHA `a37ce9120846195fa4ece8f58b268e6043cb2f26` with `# v3` comment. (3) Moved all `${{ }}` expressions out of `run:` blocks into `env:` blocks: `matrix.rc` → `MATRIX_RC` env var with `exit "$MATRIX_RC"`, `toJSON(needs.*.result)` → `RESULTS` env var with `echo "results=$RESULTS"` (8 jobs), and `toJSON(steps.check.outputs)` → `OUTPUTS` env var with `echo "outputs=$OUTPUTS"` (8 jobs).

