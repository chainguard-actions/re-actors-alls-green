<!-- markdownlint-disable -->

# Hardening Report: re-actors--alls-green/v1.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **re-actors--alls-green/v1.3.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 9c25073..0c1de71 100644
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
@@ -41,21 +33,36 @@ outputs:
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
-      INPUT_ALLOWED_FAILURES: ${{ inputs.allowed-failures }}
-      INPUT_ALLOWED_SKIPS: ${{ inputs.allowed-skips }}
-      INPUT_JOBS: ${{ inputs.jobs }}
-      PYTHONPATH: ${{ github.action_path }}/src
-    run: |
-      python -m normalize_needed_jobs_status \
-        "$INPUT_ALLOWED_FAILURES" \
-        "$INPUT_ALLOWED_SKIPS" \
-        "$INPUT_JOBS"
-    shell: bash
-...
+    - name: Decide whether the input jobs succeeded or failed
+      id: outcome
+      env:
+        INPUT_ALLOWED_FAILURES: ${{ inputs.allowed-failures }}
+        INPUT_ALLOWED_SKIPS: ${{ inputs.allowed-skips }}
+        INPUT_JOBS: ${{ inputs.jobs }}
+        PYTHONPATH: ${{ github.action_path }}/src
+      run: |
+        python -m normalize_needed_jobs_status \
+          "$INPUT_ALLOWED_FAILURES" \
+          "$INPUT_ALLOWED_SKIPS" \
+          "$INPUT_JOBS"
+      shell: bash

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** prior-fix

**Notes:**

I made no changes, because the earlier patch only changed formatting. It re-indented the steps, removed the '---' and '...' markers and rejoined the folded description lines. Nothing in it affected security. The current action.yml already does the safe thing: all three inputs go into the step's env: block and are used as quoted "$VAR" arguments in the script. The only ${{ }} expressions left in the file are the env: values and the output declarations, and neither is injectable. The Python script writes only fixed values to GITHUB_OUTPUT. There are no uses: references and no container images to pin, so the action file is unchanged.

