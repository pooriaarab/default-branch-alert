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

Add a **separate job** that `needs:` the job you're tracking, rather than a
step inside it. That way this never touches the permissions or steps of a
job that already works — it's purely additive:

```yaml
jobs:
  deploy:
    steps:
      - # ...your existing deploy steps, unchanged...

  report-default-branch-status:
    name: Report default-branch status
    needs: [deploy]
    if: always() && github.ref == format('refs/heads/{0}', github.event.repository.default_branch)
    runs-on: ubuntu-latest
    permissions:
      issues: write
    steps:
      - uses: pooriaarab/default-branch-alert@v1.0.0
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          status: ${{ needs.deploy.result }}
```

`needs.<job>.result` is one of `success`, `failure`, `cancelled`, or
`skipped` — feed it straight through. `cancelled` and `skipped` are already
no-ops inside the action, and `if: always()` just means this reporting job
itself always gets a turn to check; it does nothing on either of those two.

If you'd rather not add a job (e.g. a single-job workflow where a separate
job is overkill), a step at the end of the existing job works too — just
make sure that job already carries (or gets) `issues: write`:

```yaml
      - name: Report default-branch status
        if: always() && (success() || failure()) && github.ref == format('refs/heads/{0}', github.event.repository.default_branch)
        uses: pooriaarab/default-branch-alert@v1.0.0
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          status: ${{ job.status }}
```

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

Consumers pin an **immutable** tag — `@v1.0.0` today — never `@main` and
never a moving alias like `@v1`. **Tags here never move.** A fix or change
ships as a new tag (`v1.0.1`, `v1.1.0`, ...); the old tag keeps pointing at
the old commit forever.

This is a deliberate departure from
[`pooriaarab/vibecodereview`](https://github.com/pooriaarab/vibecodereview),
which uses a moving `@v1` — and that's the right call *for that action*, not
a mistake to copy here. The difference is the consequence of a bad
propagation, not a general preference for pinning:

- A moving tag auto-propagates the instant it moves. The only gate is that a
  human chose to move it — not that anyone reviewed the effect on any
  specific consumer.
- For a PR reviewer bot, that's an acceptable trade: high call volume, fast
  feedback, and the worst case is a bad review comment somebody notices
  within the hour.
- For this action, it inverts. It fires only on failure, so a regression can
  sit undetected for weeks, and the failure mode is that **every consumer's
  alerting goes quietly wrong at once** — a broken alerter manufacturing
  false confidence, which is worse than no alerter at all. (This isn't
  hypothetical: two real bugs shipped here during initial testing, and both
  looked fine until they hit a live runner.)

An immutable tag costs slower propagation of a real fix — each consumer
needs its own small bump PR to pick one up. For something this
infrequent-but-high-stakes, that's a good trade: it turns an invisible,
fleet-wide blast radius into a mandatory, reviewable touchpoint per repo.

If you're adding a new shared action and wondering which convention to
follow: ask what a bad, unreviewed change would cost. High call volume and
fast feedback → a moving tag is fine. Low call volume and a failure mode
that's silent until it matters → pin immutable tags instead.
