<!-- markdownlint-disable -->

# Hardening Report: devops-infra--template-action/v1.0.9

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **devops-infra--template-action/v1.0.9** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml references a Docker image using a mutable version tag (`docker://devopsinfra/template-action:v1.0.9`) instead of an immutable SHA digest. If the tag is overwritten on the registry, the action will silently pull different (potentially malicious) code. The image reference should be pinned to a full SHA256 digest, e.g. `docker://devopsinfra/template-action@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:21`

### github-env-injection (severity: high)

In entrypoint.sh, the variable `FOOBAR` is populated from the user-controlled input `INPUT_FOOBAR` and then written directly to `$GITHUB_OUTPUT` via `write_output "foobar=${FOOBAR}"` and `write_output "barfoo=${FOOBAR}"` without first stripping newline characters. An attacker who controls the `foobar` input can embed newline sequences to inject arbitrary additional key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting outputs consumed by downstream steps. The fix is to sanitize the value before writing: `safe=$(printf '%s' "$FOOBAR" | tr -d '\n\r')` and then use `$safe` in the write.

Locations:

- `entrypoint.sh:43`
- `entrypoint.sh:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

1. action.yml line 21: Pinned Docker image from mutable tag `devopsinfra/template-action:v1.0.9` to immutable digest `devopsinfra/template-action@sha256:d2a0871c2cbb0c8513de5e466c15b8bc3c0bba2dd455cfc9d7c4b37a1d83055c` with the original tag preserved as a comment. 2. entrypoint.sh lines 43-44: Added `safe_foobar=$(printf '%s' "${FOOBAR}" | tr -d '\n\r')` before the write_output calls, and replaced `${FOOBAR}` with `${safe_foobar}` in both write_output calls to prevent newline injection into GITHUB_OUTPUT.

