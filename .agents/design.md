# default-branch-alert Design

## Overview

This contract covers generated issue text, comments, logs, YAML inputs, and usage examples.
The surface type is `developer-ui`.

Use `action.yml` as the behavior source.
Use `README.md` as the setup and versioning source.

Keep failure history durable, deduplicated, and easy to scan.

## Colors

The only defined color is GitHub Action branding color `red` in `action.yml`.
It accompanies the `alert-triangle` icon on GitHub-owned surfaces.

GitHub owns issue, comment, and log colors.
Do not assign additional status meanings to those colors.

Never use color as the only failure or recovery signal.

## Typography

GitHub owns the interface fonts.
Workflow files and logs use their host's monospace presentation.

Bold the workflow name in a new issue.
Use code formatting for the branch name.

Keep input names, statuses, labels, and URLs exact.
Do not use capitalization alone to communicate state.

## Layout

Start a new issue with the workflow and branch failure sentence.
Place `Run: <url>` on its own line after a blank line.

Follow with the repeat-failure and automatic-recovery explanation.
Keep the deduplication marker last and hidden.

Format repeat comments as `Failed again: <url>`.
Format recovery comments as `Resolved by <url>`.

Keep workflow examples in fenced YAML blocks.
Show the separate reporting job before the optional single-job alternative.

## Elevation & Depth

Not applicable.
The repository defines no shadows, overlays, or stacking rules.

Represent hierarchy through issue order, Markdown emphasis, and YAML nesting.

## Shapes

Use the `alert-triangle` icon only through GitHub Action branding metadata.
Do not add a separate copied icon asset.

Use the hidden marker format `<!-- default-branch-alert:<workflow>:<branch> -->`.
Treat it as a deduplication key, not visible copy.

## Components

Required inputs are `github_token` and `status`.
Optional inputs are `workflow_name`, `branch`, `label`, and `run_url`.

The tracking label scopes direct open-issue listing.
The hidden marker distinguishes workflow and branch pairs.

On failure, create an issue or comment on the existing issue.
On success, comment and close an existing issue.

For cancelled, skipped, or other values, log the no-op and stop.
When no recovery issue exists, log that nothing needs closing.

## Do's and Don'ts

- Do require `issues: write` in caller examples.
- Don't imply the action grants its own permissions.
- Do gate pull requests and non-default branches in the caller.
- Don't claim the action detects those events itself.
- Do preserve the workflow and branch marker.
- Don't replace direct issue listing with search.
- Do show the run URL in issues and comments.
- Don't create a new issue for each repeat failure.
- Do document immutable release tags.
- Don't recommend `main` or moving aliases to consumers.
