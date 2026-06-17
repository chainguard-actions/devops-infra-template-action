<!-- markdownlint-disable -->

# Hardening Report: devops-infra--template-action/v1.0.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **devops-infra--template-action/v1.0.8** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image reference with a mutable version tag instead of an immutable SHA digest. `image: docker://devopsinfra/template-action:v1.0.8` uses the tag `v1.0.8`, which can be silently overwritten on the registry, enabling a supply-chain attack. It should be pinned to a full SHA256 digest, e.g. `image: docker://devopsinfra/template-action@sha256:<64-hex-char-digest> # v1.0.8`.

Locations:

- `action.yml:20`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced the mutable Docker image tag `docker://devopsinfra/template-action:v1.0.8` with the immutable SHA256 digest `docker://devopsinfra/template-action@sha256:46a3b381e36380529ee518a68020577adfc8397308fe0c8d9174aba621872055 # v1.0.8` in action.yml line 20. The tag is preserved as a comment for readability.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the GITHUB_OUTPUT injection vulnerability in entrypoint.sh by adding newline sanitization inside the write_output helper function. The kv argument is now passed through `printf '%s' "${kv}" | tr -d '\n\r'` before being written to $GITHUB_OUTPUT, stripping any embedded newlines or carriage returns that an attacker could use to inject additional key=value pairs. This centralized fix covers both write_output call sites (foobar=${FOOBAR} and barfoo=${FOOBAR}).

