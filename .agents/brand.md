# default-branch-alert Brand

## Identity

Write the action name as `default-branch-alert`.
Describe it as a composite GitHub Action for default-branch workflow reporting.

## Audience

Address maintainers who own GitHub Actions workflows and repository issues.
Assume they can edit workflow conditions and permissions.

## Product promise

Turn a reported default-branch failure into a persistent tracking issue.
Reuse that issue for repeat failures and close it after a reported recovery.

Keep the promise limited to statuses passed by the caller.

## Message order

1. Explain the separate reporting job and its `needs` dependency.
2. Show the caller's default-branch condition.
3. Grant `issues: write`.
4. Pass `github_token` and `status`.
5. Explain repeat failure and recovery behavior.
6. State the immutable tag policy.

## Voice

Use operational, exact language.
Name the caller's responsibility before describing action behavior.

Use “failure,” “repeat failure,” and “recovery” consistently.
Avoid alarmist copy and vague notification claims.

## Claims

The action handles only `failure` and `success` as active states.
It treats other status values as no-ops.

The caller must exclude pull requests and non-default branches.
Do not claim that the action performs that gating.

Deduplication uses one open issue per workflow and branch marker.
It lists labeled open issues directly and does not use search.

The action creates the tracking label when it is missing.
It needs `issues: write` through the supplied GitHub token.

## Naming and versions

Write input names exactly as defined in `action.yml`.
Write the default label as `default-branch-failure`.

Consumers pin immutable release tags.
Use `v1.0.0` for the current documented release.
Do not recommend `main` or a moving `v1` alias.

## Assets

GitHub renders the `alert-triangle` icon with its `red` action color.
Both values come from `action.yml`.

The repository defines no custom logo, font, palette, website, or deployed interface.
