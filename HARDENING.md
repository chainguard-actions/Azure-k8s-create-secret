<!-- markdownlint-disable -->

# Hardening Report: Azure--k8s-create-secret/v6.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Azure--k8s-create-secret/v6.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite action uses `medyagh/setup-minikube@latest`, which is pinned to a mutable tag (`latest`) rather than a full 40-character commit SHA. This means the referenced action can be silently changed by its maintainer (or a supply-chain attacker) at any time, potentially injecting malicious code into the workflow. It should be pinned to a specific commit SHA, e.g. `medyagh/setup-minikube@<40-char-sha> # latest`.

Locations:

- `.github/actions/setup-test-environment/action.yml:23`

### script-injection (severity: high)

Sub-rule (a): Two `${{ inputs.pr-base-ref }}` expressions are interpolated directly inside a `run:` shell script. Because GitHub Actions performs template substitution before the shell ever sees the string, an attacker who controls the `pr-base-ref` input can inject arbitrary shell commands. Line 31: `echo ${{ inputs.pr-base-ref }}` — the value is also unquoted, compounding the risk. Line 32: `if [[ "${{ inputs.pr-base-ref }}" != releases/* ]]` — even though the outer double-quotes are present, the template substitution happens before quoting, so a value containing `" ]; do_evil; if [[` can break out of the condition. Both occurrences must be replaced by routing the input through an `env:` variable and referencing it as a quoted shell variable (e.g. `"$PR_BASE_REF"`).

Locations:

- `.github/actions/setup-test-environment/action.yml:31`
- `.github/actions/setup-test-environment/action.yml:32`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed two findings in .github/actions/setup-test-environment/action.yml: (1) Pinned medyagh/setup-minikube@latest to full commit SHA e9e035a86bbc3caea26a450bd4dbf9d0c453682e with '# latest' comment for readability. (2) Moved both ${{ inputs.pr-base-ref }} expressions out of the run: shell script into an env: block as PR_BASE_REF, then referenced it as "$PR_BASE_REF" in the shell script (also added quotes to the echo line to prevent word-splitting).

