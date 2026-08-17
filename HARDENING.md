<!-- markdownlint-disable -->

# Hardening Report: devops-infra--template-action/v1.0.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **devops-infra--template-action/v1.0.10** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In entrypoint.sh, the action input `foobar` is read from the caller-controlled environment variable `INPUT_FOOBAR` into `FOOBAR`, then written unsanitized to `$GITHUB_OUTPUT` via `write_output "foobar=${FOOBAR}"` and `write_output "barfoo=${FOOBAR}"`. The `write_output` helper uses `printf "%s\n" "${kv}" >> "${GITHUB_OUTPUT}"` with no newline stripping (`tr -d '\n\r'`) applied before the write. A caller supplying a newline-containing value for `foobar` can inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially poisoning downstream steps. Fix: sanitize the value before writing, e.g. `safe=$(printf '%s' "$FOOBAR" | tr -d '\n\r')` then `write_output "foobar=${safe}"`

Locations:

- `entrypoint.sh:17`
- `entrypoint.sh:51`
- `entrypoint.sh:52`

### unpinned-uses (severity: high)

The Docker action in action.yml references the image `docker://devopsinfra/template-action:v1.0.10` using a mutable version tag (`v1.0.10`) instead of an immutable SHA digest. A tag can be silently overwritten to point to a different (potentially malicious) image, enabling a supply-chain attack. Fix: pin to a specific SHA digest, e.g. `image: docker://devopsinfra/template-action@sha256:<64-hex-char-digest> # v1.0.10`

Locations:

- `action.yml:21`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, unpinned-uses

**Notes:**

1. entrypoint.sh: Added newline sanitization before writing FOOBAR to GITHUB_OUTPUT. Introduced `FOOBAR_SAFE=$(printf '%s' "${FOOBAR}" | tr -d '\n\r')` and replaced both `write_output "foobar=${FOOBAR}"` and `write_output "barfoo=${FOOBAR}"` with `write_output "foobar=${FOOBAR_SAFE}"` and `write_output "barfoo=${FOOBAR_SAFE}"` respectively. 2. action.yml: Pinned the Docker image reference from the mutable tag `docker://devopsinfra/template-action:v1.0.10` to the immutable digest `docker://devopsinfra/template-action:v1.0.10@sha256:dd618c7fccfeb8758480b129ada64867a856ceeae22e91c1a5fac71a681373c8`, preserving the `docker://` scheme and the version tag inline.

