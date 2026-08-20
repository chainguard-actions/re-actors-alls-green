<!-- markdownlint-disable -->

# Hardening Report: re-actors--alls-green/v1.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **re-actors--alls-green/v1.2.2** was hardened automatically. 7 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The composite action's run: block directly interpolates user-controlled inputs into shell command strings via ${{ }} expressions (sub-rule a). Specifically, ${{ inputs.allowed-failures }}, ${{ inputs.allowed-skips }}, and ${{ inputs.jobs }} are embedded inside shell heredocs passed as positional arguments to Python. An attacker-controlled caller can inject arbitrary shell metacharacters through these inputs. The offending lines are inside the heredoc arguments: `${{ inputs.allowed-failures }}`, `${{ inputs.allowed-skips }}`, and `${{ inputs.jobs }}`.

Locations:

- `action.yml:51`
- `action.yml:55`
- `action.yml:59`

### script-injection (severity: high)

The workflow's run: block directly interpolates ${{ matrix.rc }} into a shell command string (sub-rule a): `run: exit ${{ matrix.rc }}`. Any expression interpolated via ${{ }} inside a run: block is a script-injection risk because the value is substituted into the shell command before the shell parses it.

Locations:

- `.github/workflows/self-smoke-test-action.yml:35`

### unpinned-uses (severity: high)

The workflow uses `actions/checkout@v3` (a mutable tag reference, not a pinned 40-character SHA commit hash) in 9 separate steps across multiple jobs. This exposes the workflow to supply-chain attacks if the tag is moved or the upstream repository is compromised. All occurrences should be pinned to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v3`.

Locations:

- `.github/workflows/self-smoke-test-action.yml:40`
- `.github/workflows/self-smoke-test-action.yml:55`
- `.github/workflows/self-smoke-test-action.yml:70`
- `.github/workflows/self-smoke-test-action.yml:86`
- `.github/workflows/self-smoke-test-action.yml:103`
- `.github/workflows/self-smoke-test-action.yml:120`
- `.github/workflows/self-smoke-test-action.yml:137`
- `.github/workflows/self-smoke-test-action.yml:154`
- `.github/workflows/self-smoke-test-action.yml:172`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g. write access to contents). A minimal permissions block such as `permissions: {}` or specific scopes should be added.

Locations:

- `.github/workflows/self-smoke-test-action.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.allowed-failures }}" appears directly in run: block of step "Decide whether the input jobs succeeded or failed"; move to env: map

Locations:

- `action.yml:55`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.allowed-skips }}" appears directly in run: block of step "Decide whether the input jobs succeeded or failed"; move to env: map

Locations:

- `action.yml:59`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.jobs }}" appears directly in run: block of step "Decide whether the input jobs succeeded or failed"; move to env: map

Locations:

- `action.yml:63`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all findings: (1) action.yml: moved ${{ inputs.allowed-failures }}, ${{ inputs.allowed-skips }}, and ${{ inputs.jobs }} from the run: heredoc into the env: block as INPUT_ALLOWED_FAILURES, INPUT_ALLOWED_SKIPS, INPUT_JOBS, referenced as plain env vars in the shell script; (2) workflow: fixed 'exit ${{ matrix.rc }}' by moving matrix.rc into env: as MATRIX_RC; (3) pinned all 9 actions/checkout@v3 references to SHA a37ce9120846195fa4ece8f58b268e6043cb2f26 # v3; (4) added 'permissions: {}' at the top level of the workflow.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed 16 script injection instances across 8 jobs in .github/workflows/self-smoke-test-action.yml. In each job, the two `run:` steps that directly interpolated `${{ toJSON(needs.*.result) }}` and `${{ toJSON(steps.check.outputs) }}` into shell strings were updated to use `env:` blocks. The expressions are now assigned to environment variables RESULTS and OUTPUTS respectively, and referenced as `$RESULTS` and `$OUTPUTS` in the shell commands. The single-quoted `echo 'results=...'` form was also changed to double-quoted `echo "results=$RESULTS"` to properly expand the variable. All other parts of the workflow (pinned action SHAs, permissions block, etc.) were preserved unchanged.

