<!-- markdownlint-disable -->

# Hardening Report: devops-infra--template-action/v1.0.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **devops-infra--template-action/v1.0.6** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image referenced by a mutable version tag rather than an immutable SHA digest. `image: docker://devopsinfra/template-action:v1.0.6` can be silently replaced by a different image at the same tag, enabling supply-chain attacks. It should be pinned to a SHA digest, e.g. `image: docker://devopsinfra/template-action@sha256:<64-hex-char-digest> # v1.0.6`.

Locations:

- `action.yml:20`

### github-env-injection (severity: high)

entrypoint.sh writes the `foobar` action input (an untrusted value sourced from `INPUT_FOOBAR`) to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The `write_output` helper function does `printf "%s\n" "${kv}" >> "${GITHUB_OUTPUT}"` with no newline stripping. A caller can inject newlines into the output file to forge additional key=value pairs, potentially overwriting subsequent outputs or injecting environment variables. The two affected calls are `write_output "foobar=${FOOBAR}"` and `write_output "barfoo=${FOOBAR}"`.

Locations:

- `entrypoint.sh:17`
- `entrypoint.sh:50`
- `entrypoint.sh:51`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

1. action.yml: Pinned the Docker image from mutable tag `devopsinfra/template-action:v1.0.6` to immutable digest `devopsinfra/template-action@sha256:ae813db4d6146511313ac871c05e90035bab8fdbd0835681e4b03cb8f97d14d2 # v1.0.6`. 2. entrypoint.sh: Updated the `write_output` helper function to split the key=value argument, sanitize the value portion with `printf '%s' "${value}" | tr -d '\n\r'` to strip embedded newlines/carriage-returns, and write the sanitized `key=safe_value` pair to GITHUB_OUTPUT. This prevents newline injection attacks on both `write_output "foobar=${FOOBAR}"` and `write_output "barfoo=${FOOBAR}"` call sites.

