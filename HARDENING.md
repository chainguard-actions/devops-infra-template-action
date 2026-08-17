<!-- markdownlint-disable -->

# Hardening Report: devops-infra--template-action/v1.0.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **devops-infra--template-action/v1.0.8** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files reference reusable workflows using mutable tag or branch refs instead of pinned 40-character SHA commits. This exposes the action to supply-chain attacks if the referenced repository is compromised or the tag is moved. Affected references:
- auto-pull-request-create.yml: `devops-infra/.github/.github/workflows/reusable-auto-pull-request-create.yml@v1`
- auto-release-create.yml: `devops-infra/.github/.github/workflows/reusable-auto-release-create.yml@v1`
- cron-dependency-update.yml: `devops-infra/.github/.github/workflows/reusable-cron-dependency-update.yml@v1`
- manual-e2e-validate.yml: `devops-infra/triglav/.github/workflows/e2e-action-template-action.yml@master`
- manual-release-branch-prepare.yml: `devops-infra/.github/.github/workflows/reusable-manual-release-branch-prepare.yml@v1`

Additionally, action.yml uses a mutable Docker image tag `docker://devopsinfra/template-action:v1.0.8` instead of a SHA digest (e.g. `docker://devopsinfra/template-action@sha256:<64-hex-char-digest>`).

Locations:

- `.github/workflows/auto-pull-request-create.yml:15`
- `.github/workflows/auto-release-create.yml:26`
- `.github/workflows/cron-dependency-update.yml:14`
- `.github/workflows/manual-e2e-validate.yml:24`
- `.github/workflows/manual-release-branch-prepare.yml:28`
- `action.yml:22`

### github-env-injection (severity: high)

entrypoint.sh writes the caller-controlled input value `${FOOBAR}` (sourced from `INPUT_FOOBAR`, which maps to the action's `foobar` input) to `$GITHUB_OUTPUT` via the `write_output` helper without first sanitizing newline characters. The calls `write_output "foobar=${FOOBAR}"` and `write_output "barfoo=${FOOBAR}"` pass the raw value directly to `printf "%s\n" "${kv}" >> "${GITHUB_OUTPUT}"`. An attacker-controlled value containing newline characters (e.g. `foo\nsome_key=injected_value`) can inject arbitrary key-value pairs into the GITHUB_OUTPUT file, potentially overwriting other outputs or injecting environment variables consumed by downstream steps. The required sanitization step — `safe=$(printf '%s' "$FOOBAR" | tr -d '\n\r')` — must be applied before writing to $GITHUB_OUTPUT.

Locations:

- `entrypoint.sh:47`
- `entrypoint.sh:48`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

Fixed all 6 unpinned workflow/image references: pinned devops-infra/.github reusable workflows (auto-pull-request-create, auto-release-create, cron-dependency-update, manual-release-branch-prepare) to SHA 88b425df3a0f0415f12a0678b84d065562363bdc (v1); pinned devops-infra/triglav e2e workflow to SHA 6b7cf10d8042334e484bdc14ea1c9b4b335cea3e (master); pinned Docker image devopsinfra/template-action:v1.0.8 to sha256:46a3b381e36380529ee518a68020577adfc8397308fe0c8d9174aba621872055 preserving the docker:// scheme and tag inline. Fixed github-env-injection in entrypoint.sh by sanitizing FOOBAR with `printf '%s' "${FOOBAR}" | tr -d '\n\r'` into SAFE_FOOBAR before both write_output calls.

