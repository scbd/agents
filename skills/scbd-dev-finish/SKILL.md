---
name: scbd-dev-finish
description: Wrap up after a pull request is merged — mark the Jira ticket Done and delete the local feature branch. Use for "wrap this up", "close out DEV-123", or "clean up after the merge".
---

# scbd-dev-finish

Close out a ticket once its PR has landed.

**Usage:** `/scbd-dev-finish [<jira-key>] [yes]`

## Before you start

Load `scbd-dev-agent` for the shared ground rules. Regardless:

- Commits, pushes, PRs, replies and Jira changes pass a checkpoint (`scbd-github`, `scbd-jira`).
  With no preference set, ask before anything leaves the machine.
- Never push to the default branch, mark a PR ready, force-push, or discard unexplained work.
- Stage explicit paths only; never stage `.scratch/`.

## Check

Confirm the PR is merged and matches the ticket (`gh pr view --json state,mergedAt`). Stop without
changes if it was closed without merging, or doesn't match the ticket.

## Close out

The Jira steps pass the `jira` checkpoint (`scbd-jira`). Running this command is a direct request,
so `invoked` goes ahead.

- Transition Jira to `Done` and add a comment with the PR URL (`scbd-jira`).
- Switch to the default branch and pull.
- Delete the local branch with `git branch -d`, never `-D`.
