---
name: scbd-dev-finish
description: Wrap up after a pull request is merged — mark the Jira ticket Done and delete the local feature branch. Use for "wrap this up", "close out DEV-123", or "clean up after the merge".
---

# scbd-dev-finish

Close out a ticket once its PR has landed.

**Usage:** `/scbd-dev-finish [<jira-key>]`

## Before you start

Load `scbd-dev-agent` for the shared ground rules. Regardless:

- Nothing leaves the machine (push, PR, comments, Jira) without listing the actions and getting OK.
- Never push to the default branch, mark a PR ready, force-push, or discard unexplained work.
- Stage explicit paths only; never stage `.scratch/`.

## Check

Confirm the PR is merged and matches the ticket (`gh pr view --json state,mergedAt`). Stop without
changes if it was closed without merging, or doesn't match the ticket.

## After OK

- Transition Jira to `Done` and add a comment with the PR URL (`scbd-jira`).
- Switch to the default branch and pull.
- Delete the local branch with `git branch -d`, never `-D`.
