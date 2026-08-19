<!-- markdownlint-disable -->

# Hardening Report: jpetrucciani--black-check/26.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **jpetrucciani--black-check/26.5.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files are pinned to mutable tags or version strings rather than full 40-character SHA commit hashes. This exposes the action to supply-chain attacks if the referenced tag is moved or the upstream repository is compromised. Failing references: `actions/checkout@v3` (tag.yml line 10, update.yml line 8), `actions/setup-python@v2` (update.yml line 10), `peter-evans/create-pull-request@v3.10.0` (update.yml line 22).

Locations:

- `.github/workflows/tag.yml:10`
- `.github/workflows/update.yml:8`
- `.github/workflows/update.yml:10`
- `.github/workflows/update.yml:22`

### missing-permissions (severity: medium)

Neither `.github/workflows/tag.yml` nor `.github/workflows/update.yml` defines a top-level `permissions:` key, and no job in either file defines job-level `permissions:`. Without explicit permissions, workflows run with the default (often broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/tag.yml:1`
- `.github/workflows/update.yml:1`

### script-injection (severity: high)

Sub-rule (a): A `run:` block in `update.yml` directly interpolates a `${{ ... }}` expression into a shell command string. The offending line is: `run: gh pr merge --auto --squash ${{ steps.new_pull_request.outputs.pull-request-url }}`. The expression `${{ steps.new_pull_request.outputs.pull-request-url }}` is substituted into the shell command before the shell parses it, allowing shell metacharacters in the value to be interpreted. The value should be passed via an `env:` variable and then referenced as a double-quoted shell variable (e.g., `"$PR_URL"`).

Locations:

- `.github/workflows/update.yml:30`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

tag.yml: pinned actions/checkout@v3 to SHA f43a0e5ff2bd294095638e18286ca9a3d1956744, added top-level permissions: {} and job-level permissions: {contents: write}. update.yml: pinned actions/checkout@v3 to SHA f43a0e5ff2bd294095638e18286ca9a3d1956744, actions/setup-python@v2 to SHA e9aba2c848f5ebd159c070c61ea2c4e2b122355e, peter-evans/create-pull-request@v3.10.0 to SHA 9825ae65b1cb54b543b938503728b432a0176d29; added top-level permissions: {} and job-level permissions: {contents: write, pull-requests: write}; fixed script injection by moving ${{ steps.new_pull_request.outputs.pull-request-url }} into an env var PR_URL and referencing it as "$PR_URL" in the shell command.

