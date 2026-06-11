<!-- markdownlint-disable -->

# Hardening Report: re-actors--alls-green/v1.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **re-actors--alls-green/v1.2.2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a) violation: The run: block in action.yml directly interpolates GitHub Actions expressions ${{ inputs.allowed-failures }}, ${{ inputs.allowed-skips }}, and ${{ inputs.jobs }} inside the shell script. Although these values are placed inside heredocs (cat << EOM ... EOM), the GitHub Actions template engine expands all ${{ ... }} expressions before the shell ever parses the script. An attacker-controlled input containing shell metacharacters or a premature heredoc terminator (e.g., a value like 'EOM\n) && malicious_command #') can break out of the heredoc context and inject arbitrary shell commands. All three inputs must be passed via env: variables and referenced as quoted shell variables (e.g., "$ALLOWED_FAILURES") instead of being interpolated directly.

Locations:

- `action.yml:52`
- `action.yml:57`
- `action.yml:62`

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

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Moved all three ${{ inputs.* }} expressions (${{ inputs.allowed-failures }}, ${{ inputs.allowed-skips }}, ${{ inputs.jobs }}) from the run: block into the env: block as ALLOWED_FAILURES, ALLOWED_SKIPS, and JOBS environment variables. The shell script now references these as quoted shell variables ("$ALLOWED_FAILURES", "$ALLOWED_SKIPS", "$JOBS") instead of directly interpolating GitHub Actions expressions inside heredocs. The heredoc construct was removed as it is no longer needed. This eliminates the risk of shell injection via attacker-controlled input values containing shell metacharacters or heredoc terminators.

