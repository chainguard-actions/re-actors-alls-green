<!-- markdownlint-disable -->

# Hardening Report: re-actors--alls-green/v1.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **re-actors--alls-green/v1.2.2** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): the step 'Decide whether the input jobs succeeded or failed' puts GitHub Actions expressions straight into the run: script. The lines are `${{ inputs.allowed-failures }}`, `${{ inputs.allowed-skips }}` and `${{ inputs.jobs }}`. Each one sits inside a `$(cat << EOM ... EOM)` heredoc. Because the EOM delimiter is not quoted, the shell still runs any $(...) or backticks in the value. A value containing a line equal to EOM also ends the heredoc early, so any text after it runs as shell code. Fix: pass the values through env vars (e.g. ALLOWED_FAILURES: ${{ inputs.allowed-failures }}) and give them to the script as quoted arguments ("$ALLOWED_FAILURES").

Locations:

- `action.yml:55`
- `action.yml:59`
- `action.yml:63`

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

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index fc67aa9..7350946 100644
--- a/action.yml
+++ b/action.yml
@@ -1,36 +1,28 @@
----
 name: alls-green
 description: >-
-  GitHub Action for checking that the test matrix jobs are all
-  successful before proceeding. To be used in branch protection
+  GitHub Action for checking that the test matrix jobs are all successful before proceeding. To be used in branch protection
 author: >-
   Sviatoslav Sydorenko <wk+~github.com/re-actors/alls-green@sydorenko.org.ua>
-
 branding:
   icon: check-circle
   color: green
-
 inputs:
   jobs:
     description: >-
-      A list of jobs from the `needs` context, serialized as a JSON
-      string via `toJSON(needs)`
+      A list of jobs from the `needs` context, serialized as a JSON string via `toJSON(needs)`
     required: true
   allowed-failures:
     default: >-
       []
     description: >-
-      Job names that are allowed to fail and not affect the outcome,
-      as a comma-separated list or serialized as a JSON string
+      Job names that are allowed to fail and not affect the outcome, as a comma-separated list or serialized as a JSON string
     required: false
   allowed-skips:
     default: >-
       []
     description: >-
-      Job names that are allowed to be skipped and not affect the
-      outcome, as a comma-separated list or serialized as a JSON string
+      Job names that are allowed to be skipped and not affect the outcome, as a comma-separated list or serialized as a JSON string
     required: false
-
 outputs:
   failure:
     description: Whether this check decided that the job matrix failed.
@@ -41,27 +33,36 @@ outputs:
   success:
     description: Whether this check decided that the job matrix succeeded.
     value: ${{ steps.outcome.outputs.success }}
-
 runs:
   using: composite
   steps:
-  - name: Decide whether the input jobs succeeded or failed
-    id: outcome
-    env:
-      PYTHONPATH: ${{ github.action_path }}/src
-    run: |
-      python -m normalize_needed_jobs_status \
-      "$(cat << EOM
-        ${{ inputs.allowed-failures }}
-      EOM
-      )" \
-      "$(cat << EOM
-        ${{ inputs.allowed-skips }}
-      EOM
-      )" \
-      "$(cat << EOM
-        ${{ inputs.jobs }}
-      EOM
-      )"
-    shell: bash
-...
+    - name: Decide whether the input jobs succeeded or failed
+      id: outcome
+      env:
+        PYTHONPATH: ${{ github.action_path }}/src
+        INPUT_ALLOWED_FAILURES: ${{ inputs.allowed-failures }}
+        INPUT_ALLOWED_SKIPS: ${{ inputs.allowed-skips }}
+        INPUT_JOBS: ${{ inputs.jobs }}
+      run: |
+        python -m normalize_needed_jobs_status \
+        "$INPUT_ALLOWED_FAILURES" \
+        "$INPUT_ALLOWED_SKIPS" \
+        "$INPUT_JOBS"
+      shell: bash

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, prior-fix

**Notes:**

Moved inputs.allowed-failures, inputs.allowed-skips and inputs.jobs out of the unquoted heredocs in the run: script. They now go into the step's env: as INPUT_ALLOWED_FAILURES, INPUT_ALLOWED_SKIPS and INPUT_JOBS and are passed to normalize_needed_jobs_status as double-quoted arguments. Behaviour should be the same: the Python script trims whitespace itself (json.loads, or a comma split followed by strip), so dropping the leading spaces the heredocs added makes no difference. This re-applies the security part of the earlier harden.

