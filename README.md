<p align="center">
  <a href="https://github.com/TheKathan/verbump-check/releases"><img alt="latest release" src="https://img.shields.io/github/v/release/TheKathan/verbump-check"></a>
  <img alt="license" src="https://img.shields.io/github/license/TheKathan/verbump-check">
  <img alt="runners: Linux, macOS" src="https://img.shields.io/badge/runners-Linux%20%7C%20macOS-555">
</p>

# verbump check

Shows on a pull request what merging it will release with [verbump](https://github.com/TheKathan/verbump): which packages, at which versions, with which release notes. It fails now if that release would fail at merge: version files that disagree, a tag that already exists, or a `release/<package>/X.Y.Z` branch that isn't newer than the last release.

```yaml
on: pull_request

jobs:
  release-preview:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v7
        with:
          fetch-depth: 0
      - uses: TheKathan/verbump-check@v1
```

The job summary then reads, for a PR from `feat/checkout` that changed `apps/web`:

> ## verbump check
>
> Bump: **minor**, from branch `feat/checkout`
>
> Merging this PR releases:
>
> | Package | Previous | Version | Tag | Why |
> |---|---|---|---|---|
> | web | 0.3.0 | 0.4.0 | `web/v0.4.0` | minor |
>
> ▸ Release notes: web 0.4.0

The notes are worked out in a throwaway git worktree; the PR's checkout is left as it was.

## Inputs

| Input | Default | Meaning |
|---|---|---|
| `branch` | the PR's source branch | Picks the bump as on merge: `feature/*` or `feat/*` minor, anything else patch, `release/<package>/X.Y.Z` exact. |
| `bump` | `""` | `major`, `minor` or `patch` instead of the branch rule. |
| `config` | `verbump.toml` | Relative to `working-directory`. |
| `working-directory` | `.` | The repo root. |
| `verbump-version` | `1` | verbump to install with [setup-verbump](https://github.com/TheKathan/setup-verbump): `1`, an exact version, or a pip requirement. |
| `install` | `"true"` | `"false"` uses a `verbump` already on the PATH. |

## Outputs

| Output | Meaning |
|---|---|
| `releases` | How many packages merging the PR would release. |
| `tags` | JSON array of the tags it would create. |
| `plan` | Path to the plan JSON. |
| `report-file` | Path to the Markdown report, the same as the job summary. |

To post the report as a PR comment, add a step with `pull-requests: write`, e.g. `gh pr comment ${{ github.event.number }} --body-file "${{ steps.check.outputs.report-file }}" --edit-last --create-if-none`.

To release on merge, use [verbump-release](https://github.com/TheKathan/verbump-release).

Runs on Linux and macOS runners. MIT licensed.
