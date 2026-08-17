<!-- markdownlint-disable -->

# Hardening Report: devops-infra--template-action/v1.0.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **devops-infra--template-action/v1.0.7** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference reusable workflows using a mutable `@v1` tag instead of a full 40-character commit SHA. Additionally, action.yml references a Docker image using a mutable version tag (`docker://devopsinfra/template-action:v1.0.7`) instead of a SHA digest. Mutable refs can be silently updated to point to malicious code, enabling supply-chain attacks.

Failing references:
- `.github/workflows/auto-pull-request-create.yml`: `devops-infra/.github/.github/workflows/reusable-auto-pull-request-create.yml@v1`
- `.github/workflows/auto-release-create.yml`: `devops-infra/.github/.github/workflows/reusable-auto-release-create.yml@v1`
- `.github/workflows/cron-dependency-update.yml`: `devops-infra/.github/.github/workflows/reusable-cron-dependency-update.yml@v1`
- `.github/workflows/manual-release-branch-prepare.yml`: `devops-infra/.github/.github/workflows/reusable-manual-release-branch-prepare.yml@v1`
- `action.yml`: `image: docker://devopsinfra/template-action:v1.0.7` (tag, not a SHA digest)

Locations:

- `.github/workflows/auto-pull-request-create.yml:15`
- `.github/workflows/auto-release-create.yml:27`
- `.github/workflows/cron-dependency-update.yml:14`
- `.github/workflows/manual-release-branch-prepare.yml:27`
- `action.yml:20`

### github-env-injection (severity: high)

In `entrypoint.sh`, the `write_output` function writes caller-controlled input values to `$GITHUB_OUTPUT` without first sanitizing newline characters. Specifically, `FOOBAR` is derived from `INPUT_FOOBAR` (the inherited process env var for the `foobar` action input, which is set by the calling workflow and is therefore attacker-controllable). The calls `write_output "foobar=${FOOBAR}"` and `write_output "barfoo=${FOOBAR}"` pass this value directly to `printf "%s\n" "${kv}" >> "${GITHUB_OUTPUT}"` with no `tr -d '\n\r'` sanitization step. An attacker-controlled newline in `INPUT_FOOBAR` can inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting other outputs or injecting environment variables into downstream steps.

Locations:

- `entrypoint.sh:18`
- `entrypoint.sh:51`
- `entrypoint.sh:52`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

Fixed all findings:

1. unpinned-uses: Pinned Docker image in action.yml to immutable SHA digest (docker://devopsinfra/template-action:v1.0.7@sha256:3115f32441f146dd4bf3d4784d1fe3ec86a24593c87cc9db0420b0717c76610a). Pinned all four reusable workflow references in .github/workflows/ from @v1 to @88b425df3a0f0415f12a0678b84d065562363bdc # v1.

2. github-env-injection: Refactored write_output() in entrypoint.sh to take separate key and value arguments, sanitizing the value with `tr -d '\n\r'` before writing to $GITHUB_OUTPUT. Updated both call sites (foobar and barfoo outputs) to use the new two-argument form.

