# default-branch-alert

A composite GitHub Action that turns a failed default-branch workflow run into
a **persistent, visible artifact** instead of a GitHub notification email
nobody reads.

- One open issue per `(workflow, branch)`. A repeat failure comments on the
  existing issue instead of opening a new one.
- The issue closes itself (with a comment linking the recovering run) the
  next time that workflow succeeds on that branch.
- It never fires on a PR run, a cancelled run, or a branch other than the
  repository's actual default branch — that gating lives in the caller's
  `if:`, using `github.event.repository.default_branch` rather than a
  hardcoded `main`, because several repos in this fleet default to
  `production`, `master`, or something mid-rename.

## Usage

Add one step to the end of a job that deploys, migrates, or otherwise gates
the default branch. It needs `issues: write` on that job only:

```yaml
jobs:
  deploy:
    permissions:
      contents: read
      issues: write   # required for default-branch-alert
    steps:
      - # ...your existing deploy steps...

      - name: Report default-branch status
        if: always() && (success() || failure()) && github.ref == format('refs/heads/{0}', github.event.repository.default_branch)
        uses: pooriaarab/default-branch-alert@main
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          status: ${{ job.status }}
```

`always() && (success() || failure())` is deliberate: it runs the step on
both outcomes (so it can create/update on failure *and* close on recovery)
while excluding `cancelled` and `skipped`, which are neither `success()` nor
`failure()`.

## Inputs

| Input           | Required | Default                                    | Notes                                   |
| --------------- | -------- | ------------------------------------------- | ---------------------------------------- |
| `github_token`  | yes      | —                                            | Needs `issues: write` on the calling job |
| `status`        | yes      | —                                            | Pass `${{ job.status }}`                 |
| `workflow_name` | no       | `${{ github.workflow }}`                     | Part of the dedup key                    |
| `branch`        | no       | `${{ github.ref_name }}`                     | Part of the dedup key                    |
| `label`         | no       | `default-branch-failure`                     | Created on first use if missing          |
| `run_url`       | no       | link to the current run                      | Included in the issue/comment            |

## How dedup works

The dedup key is `(workflow_name, branch)`, embedded as a hidden HTML comment
marker in the issue body. Lookup lists **open** issues carrying the tracking
label and filters by that marker — a direct list call, not the eventually
consistent search API — so a burst of failures across a busy pipeline can't
race into duplicate issues.

## Versioning

Consumers should pin a tag (e.g. `@v1`) once one exists rather than tracking
`@main` directly, the same convention used by
[`pooriaarab/vibecodereview`](https://github.com/pooriaarab/vibecodereview).
