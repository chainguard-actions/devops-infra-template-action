<!-- markdownlint-disable -->

# Hardening Report: devops-infra--template-action/v1.0.9

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **devops-infra--template-action/v1.0.9** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In entrypoint.sh, the action input `foobar` (caller-controlled via `INPUT_FOOBAR`) is assigned to `FOOBAR` and then written to `$GITHUB_OUTPUT` via `write_output "foobar=${FOOBAR}"` and `write_output "barfoo=${FOOBAR}"`. The `write_output` helper uses `printf "%s\n" "${kv}" >> "${GITHUB_OUTPUT}"` with no newline-stripping sanitization (`printf '%s' ... | tr -d '\n\r'`) applied before the write. A caller-supplied value containing a newline character could inject additional key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting subsequent outputs.

Locations:

- `entrypoint.sh:47`
- `entrypoint.sh:48`

### unpinned-uses (severity: high)

The action.yml Docker action references the image `docker://devopsinfra/template-action:v1.0.9` using a mutable version tag (`v1.0.9`) rather than an immutable SHA digest (e.g. `docker://devopsinfra/template-action@sha256:<64-hex-char-digest>`). A mutable tag can be silently updated to point to a different image, enabling supply-chain attacks.

Locations:

- `action.yml:21`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

1. action.yml: Pinned the Docker image from `docker://devopsinfra/template-action:v1.0.9` to `docker://devopsinfra/template-action:v1.0.9@sha256:d2a0871c2cbb0c8513de5e466c15b8bc3c0bba2dd455cfc9d7c4b37a1d83055c`, preserving the `docker://` scheme and tag inline. 2. entrypoint.sh: Added newline sanitization for the `FOOBAR` input before writing to `$GITHUB_OUTPUT` — `FOOBAR_SAFE="$(printf '%s' "${FOOBAR}" | tr -d '\n\r')"` — and updated both `write_output` calls to use `FOOBAR_SAFE` instead of `FOOBAR`, preventing newline injection into `$GITHUB_OUTPUT`.

