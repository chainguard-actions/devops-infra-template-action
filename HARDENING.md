<!-- markdownlint-disable -->

# Hardening Report: devops-infra--template-action/v1.0.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **devops-infra--template-action/v1.0.7** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image referenced by a mutable version tag (`docker://devopsinfra/template-action:v1.0.7`) rather than an immutable SHA digest. This means the image could be replaced with a malicious version without changing the action configuration. It should be pinned to a full SHA256 digest, e.g. `docker://devopsinfra/template-action@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:21`

### github-env-injection (severity: high)

In entrypoint.sh, the caller-controlled input `INPUT_FOOBAR` is assigned to `FOOBAR` and then written to `$GITHUB_OUTPUT` twice via `write_output "foobar=${FOOBAR}"` and `write_output "barfoo=${FOOBAR}"`. The `write_output` function uses `printf "%s\n" "${kv}"` which does not strip embedded newlines from the value. An attacker can supply a `foobar` input containing newline characters to inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting other outputs or poisoning the output file. The required sanitization step (`printf '%s' "$FOOBAR" | tr -d '\n\r'`) is missing before each write.

Locations:

- `entrypoint.sh:18`
- `entrypoint.sh:51`
- `entrypoint.sh:52`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

1. action.yml line 21: Pinned Docker image from mutable tag `devopsinfra/template-action:v1.0.7` to immutable digest `devopsinfra/template-action@sha256:3115f32441f146dd4bf3d4784d1fe3ec86a24593c87cc9db0420b0717c76610a # v1.0.7`. 2. entrypoint.sh lines 51-52: Added sanitization step `FOOBAR_SAFE="$(printf '%s' "${FOOBAR}" | tr -d '\n\r')"` before writing to GITHUB_OUTPUT, stripping embedded newlines/carriage-returns from the caller-controlled `INPUT_FOOBAR` value to prevent output injection attacks.

