# prepare-release-action

A GitHub Action that creates a release-triggering PR summarizing dependency changes since the last release tag.

Designed for projects using **release-please** + **Renovate**, where dependency updates use `chore(deps):` commits that don't trigger releases. This action bridges that gap with a manual workflow trigger.

## Usage

```yaml
name: Prepare Release

on:
    workflow_dispatch:

permissions:
    contents: write
    pull-requests: write

jobs:
    prepare-release:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v6
              with:
                  fetch-depth: 0

            - uses: genu/prepare-release-action@v1
              id: prepare
```

## Requirements

- `jq` must be available on the runner (included on GitHub-hosted runners)
- A `package.json` must exist in the repository root
- `fetch-depth: 0` is required in the checkout step for full git history
- Auto-merge requires branch protection with required status checks enabled

## What it does

1. Checks if there are unreleased commits since the last git tag
2. Diffs `package.json` dependencies between the last tag and HEAD (updated, added, and removed)
3. Creates a branch (`prepare-release/<timestamp>`), commits the change, and opens a PR with a summary like:
   ```
   updated @zenstackhq/orm ^3.4.2 → ^3.4.4, added new-pkg ^1.0.0, removed old-pkg
   ```
4. Enables auto-merge (squash) on the PR by default, so it merges once status checks pass

## Inputs

| Input | Default | Description |
|---|---|---|
| `commit-type` | `fix` | Conventional commit type for the trigger commit |
| `commit-scope` | `deps` | Conventional commit scope |
| `commit-message` | `update dependencies` | Commit message (without type/scope prefix) |
| `auto-merge` | `true` | Enable auto-merge on the created PR |
| `pr-labels` | | Comma-separated labels to apply to the PR |

## Outputs

| Output | Description |
|---|---|
| `committed` | `true` if a PR was created, `false` if there was nothing to release |
| `pr-url` | URL of the created pull request |
| `pr-number` | Number of the created pull request |
