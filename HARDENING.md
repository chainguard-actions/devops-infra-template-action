<!-- markdownlint-disable -->

# Hardening Report: devops-infra--template-action/v1.0.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **devops-infra--template-action/v1.0.6** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In entrypoint.sh, the write_output function writes the value of FOOBAR (sourced from INPUT_FOOBAR, an inherited process env var set by the calling workflow) directly to $GITHUB_OUTPUT without sanitization. Specifically, lines 51-52 call write_output "foobar=${FOOBAR}" and write_output "barfoo=${FOOBAR}", and the write_output function (line 19) does printf "%s\n" "${kv}" >> "${GITHUB_OUTPUT}" with no newline stripping (tr -d '\n\r'). A calling workflow that sets the foobar input to a value containing a newline could inject arbitrary additional key=value pairs into GITHUB_OUTPUT, potentially overwriting other outputs or poisoning downstream steps.

Locations:

- `entrypoint.sh:19`
- `entrypoint.sh:51`
- `entrypoint.sh:52`

### unpinned-uses (severity: high)

Multiple workflow files reference reusable workflows using mutable version tags (@v1) instead of pinned 40-character commit SHAs. Additionally, action.yml references a Docker image using a mutable version tag (v1.0.6) instead of a SHA digest. Mutable tags can be silently updated to point to different (potentially malicious) code, enabling supply-chain attacks.

Failing references:
- .github/workflows/auto-pull-request-create.yml: uses: devops-infra/.github/.github/workflows/reusable-auto-pull-request-create.yml@v1
- .github/workflows/auto-release-create.yml: uses: devops-infra/.github/.github/workflows/reusable-auto-release-create.yml@v1
- .github/workflows/cron-dependency-update.yml: uses: devops-infra/.github/.github/workflows/reusable-cron-dependency-update.yml@v1
- .github/workflows/manual-release-branch-prepare.yml: uses: devops-infra/.github/.github/workflows/reusable-manual-release-branch-prepare.yml@v1
- action.yml: image: docker://devopsinfra/template-action:v1.0.6 (tag instead of SHA digest)

Locations:

- `.github/workflows/auto-pull-request-create.yml:16`
- `.github/workflows/auto-release-create.yml:26`
- `.github/workflows/cron-dependency-update.yml:13`
- `.github/workflows/manual-release-branch-prepare.yml:27`
- `action.yml:21`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, unpinned-uses

**Notes:**

Fixed two security findings:

1. **github-env-injection** (entrypoint.sh, line 19): Modified the `write_output` function to sanitize the key=value string before writing to $GITHUB_OUTPUT. Added `safe_kv=$(printf '%s' "${kv}" | tr -d '\n\r')` to strip newlines/carriage returns, preventing a malicious `foobar` input value containing newlines from injecting additional key=value pairs into GITHUB_OUTPUT.

2. **unpinned-uses** (5 locations):
   - `action.yml`: Pinned Docker image `docker://devopsinfra/template-action:v1.0.6` to immutable digest `docker://devopsinfra/template-action:v1.0.6@sha256:ae813db4d6146511313ac871c05e90035bab8fdbd0835681e4b03cb8f97d14d2` (preserving docker:// scheme and tag inline).
   - All four `.github/workflows/*.yml` files: Replaced `@v1` with `@88b425df3a0f0415f12a0678b84d065562363bdc # v1` for the devops-infra/.github reusable workflow references.

