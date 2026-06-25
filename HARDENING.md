<!-- markdownlint-disable -->

# Hardening Report: Azure--k8s-create-secret/v6.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **Azure--k8s-create-secret/v6.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The composite action run: block directly interpolates ${{ inputs.pr-base-ref }} into shell commands without routing through an env: variable. This violates rule (a): any ${{ ... }} expression inside a run: script is a script injection risk. Offending lines: `echo ${{ inputs.pr-base-ref }}` (line 31) and `if [[ "${{ inputs.pr-base-ref }}" != releases/* ]]; then` (line 32). An attacker who controls the pr-base-ref input can inject arbitrary shell commands.

Locations:

- `.github/actions/setup-test-environment/action.yml:31`
- `.github/actions/setup-test-environment/action.yml:32`

### unpinned-uses (severity: high)

The composite action uses medyagh/setup-minikube@latest, which is a mutable tag reference rather than a pinned 40-character commit SHA. This exposes the action to supply-chain attacks if the upstream repository is compromised or the tag is moved.

Locations:

- `.github/actions/setup-test-environment/action.yml:23`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed two findings in .github/actions/setup-test-environment/action.yml: (1) Pinned medyagh/setup-minikube@latest to full commit SHA e9e035a86bbc3caea26a450bd4dbf9d0c453682e with the tag preserved as a comment. (2) Moved both uses of ${{ inputs.pr-base-ref }} in the run: block into an env: variable PR_BASE_REF, then referenced it safely as "$PR_BASE_REF" in the shell script to prevent script injection attacks.

