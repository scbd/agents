---
name: scbd-dev-pr
description: Create or update a draft pull request for the current branch — pushes, writes the PR description, and links the Jira ticket. Use for "open a PR", "push this up", or "update the PR" once a change is ready to share.
---

# scbd-dev-pr

Push the branch and create or update a draft PR. Never marks a PR ready.

**Usage:** `/scbd-dev-pr [<jira-key>] [ask|invoked|auto]`

## Before you start

Load `scbd-dev-agent` for the shared ground rules. Regardless:

- Commits, pushes, PRs, replies and Jira changes pass a checkpoint (`scbd-github`, `scbd-jira`).
  With no preference set, ask before anything leaves the machine.
- Never push to the default branch, mark a PR ready, force-push, or discard unexplained work.
- Stage explicit paths only; never stage `.scratch/`.

## Steps

1. Resolve the context, the default branch, and any existing PR (`gh pr view`, `scbd-github`).
2. If there are uncommitted changes, offer to commit them, or stop.
3. Draft the title and body from the template in `scbd-github`. Include screenshot placeholders
   from `.scratch/screenshots/<key>/` if any exist.
4. List the actions:
   - push
   - `gh pr create --draft --base <default-branch>` or `gh pr edit`
   - optional Jira steps: transition to `Peer Review`, swap the label `ready-for-agent` for
     `ready-for-human`, and add a comment with the PR URL (`scbd-jira`)
5. Pass the checkpoints: `github.pr` for the push and PR, `jira` for the Jira steps. Running this
   command is a direct request for the PR, so `invoked` goes ahead; the Jira steps are offered, not
   requested, so they go ahead only under `auto`. Where a checkpoint needs OK, show the list and
   wait.
6. Run the actions and verify: the PR renders correctly, and the Jira status and comment are there.
   Report the PR URL and exactly what ran.

## Jira `Peer Review`

Offer the Jira transition only when the PR contains implementation, not for a plan-only PR.

## Never mark the PR ready

That's a human decision, always.

## Screenshot reminder

If the body has screenshot placeholders, remind the human to drop the files in before sharing the
PR link.
