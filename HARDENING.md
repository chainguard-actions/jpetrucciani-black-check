<!-- markdownlint-disable -->

# Hardening Report: jpetrucciani--black-check/26.5.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **jpetrucciani--black-check/26.5.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tag/version refs instead of pinned 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the referenced action tag is moved or compromised.

Failing references:
- .github/workflows/tag.yml: `uses: actions/checkout@v3`
- .github/workflows/update.yml: `uses: actions/checkout@v3`, `uses: actions/setup-python@v2`, `uses: peter-evans/create-pull-request@v3.10.0`

Locations:

- `.github/workflows/tag.yml:10`
- `.github/workflows/update.yml:10`
- `.github/workflows/update.yml:12`
- `.github/workflows/update.yml:21`

### missing-permissions (severity: medium)

Neither workflow file declares a top-level `permissions:` key, and no job in either file has a job-level `permissions:` block. Without explicit permissions, workflows run with the default (potentially broad) token permissions. Minimal required permissions should be declared explicitly.

Locations:

- `.github/workflows/tag.yml:1`
- `.github/workflows/update.yml:1`

### script-injection (severity: high)

Rule (a) violation: In update.yml, the final `run:` step directly interpolates the GitHub Actions expression `${{ steps.new_pull_request.outputs.pull-request-url }}` into the shell command string. This value flows through YAML template substitution before the shell parses it, allowing an attacker who can influence the pull-request URL (e.g. via a crafted branch name or PR) to inject arbitrary shell commands. The offending line is:
  `run: gh pr merge --auto --squash ${{ steps.new_pull_request.outputs.pull-request-url }}`
Fix: move the value into an `env:` variable and reference it as a quoted shell variable, e.g. `"$PR_URL"`.

Locations:

- `.github/workflows/update.yml:28`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across .github/workflows/tag.yml and .github/workflows/update.yml:

1. **unpinned-uses**: Pinned all four action references to full commit SHAs:
   - actions/checkout@v3 → @f43a0e5ff2bd294095638e18286ca9a3d1956744 (in both files)
   - actions/setup-python@v2 → @e9aba2c848f5ebd159c070c61ea2c4e2b122355e
   - peter-evans/create-pull-request@v3.10.0 → @9825ae65b1cb54b543b938503728b432a0176d29
   Original tags preserved as inline comments for readability.

2. **missing-permissions**: Added `permissions: {}` at the top level of both workflow files (deny-all default), plus job-level permissions granting only what's needed:
   - tag.yml job: `contents: write` (to push tags)
   - update.yml job: `contents: write` + `pull-requests: write` (to create/merge PRs)

3. **script-injection**: In update.yml's final step, moved `${{ steps.new_pull_request.outputs.pull-request-url }}` out of the `run:` shell string into an `env:` block as `PR_URL`, then referenced it as the quoted shell variable `"$PR_URL"` in the command.

