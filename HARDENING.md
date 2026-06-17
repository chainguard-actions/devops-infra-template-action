<!-- markdownlint-disable -->

# Hardening Report: devops-infra--template-action/v1.0.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **devops-infra--template-action/v1.0.10** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action uses a Docker image referenced by a mutable version tag (`docker://devopsinfra/template-action:v1.0.10`) instead of an immutable SHA digest. This means the image could be replaced with a malicious version without changing the action configuration. It should be pinned to a SHA digest, e.g. `docker://devopsinfra/template-action@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:20`

### github-env-injection (severity: high)

In `entrypoint.sh`, the user-controlled input `INPUT_FOOBAR` is stored in `FOOBAR` and then written to `$GITHUB_OUTPUT` via `write_output "foobar=${FOOBAR}"` and `write_output "barfoo=${FOOBAR}"`. The `write_output` function uses `printf "%s\n" "${kv}" >> "${GITHUB_OUTPUT}"` without first stripping newline characters (`tr -d '\n\r'`). An attacker can supply a value containing newline characters to inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting outputs consumed by downstream steps.

Locations:

- `entrypoint.sh:44`
- `entrypoint.sh:45`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

1. action.yml line 20: Pinned Docker image from mutable tag `devopsinfra/template-action:v1.0.10` to immutable digest `devopsinfra/template-action@sha256:dd618c7fccfeb8758480b129ada64867a856ceeae22e91c1a5fac71a681373c8 # v1.0.10`. 2. entrypoint.sh write_output function: Added newline sanitization using `printf '%s' "${kv}" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT, preventing injection of arbitrary key=value pairs via newline characters in user-controlled input.

