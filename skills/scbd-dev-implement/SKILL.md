---
name: scbd-dev-implement
description: Implement a Jira ticket or task from a plan or the conversation, make tests pass, and commit logical chunks when asked. Use for "implement DEV-123" or "build this" once a plan exists or the change is clear.
---

# scbd-dev-implement

Build the change. Follow `karpathy-guidelines` and stop before pushing.

**Usage:** `/scbd-dev-implement [<jira-key>] [plan=<path>]`

## Before you start

Load `scbd-dev-agent` for the shared ground rules. Regardless:

- Commits, pushes, PRs, replies and Jira changes pass a checkpoint (`scbd-github`, `scbd-jira`).
  With no preference set, ask before anything leaves the machine.
- Never push to the default branch, mark a PR ready, force-push, or discard unexplained work.
- Stage explicit paths only; never stage `.scratch/`.

## Finding the plan

Look in order: the `plan=` argument, `.scratch/plans/<key>.md`, `<scbd_plan_dir>/<key>.md`, then a
plan already in the conversation.

- **None found:** state a brief approach in chat. Proceed if the change is small; ask first if it
  involves a material decision.
- **Several matches:** ask the human which one to use.

## Steps

1. Follow `karpathy-guidelines`.
2. Make the smallest complete change.
3. Add the tests the change needs.
4. Run the relevant checks. Fix failures the change caused; report failures that were already
   there.

## Delegation

Suggest delegating to sub-agents when the plan has independent steps (`scbd-dev-agent`'s
Delegation section). Otherwise do the work in this conversation.

## Commits

Commits pass the `git.commit` checkpoint (`scbd-github`). Conventional Commits, one logical change
per commit.

If the plan was committed to `scbd_plan_dir` (shared for review), ask whether to remove it in the
implementation commit.

## Stop before pushing

Report what changed, the checks that ran, and the user-facing impact. Suggest
`/scbd-dev-screenshot` when the UI changed, then `/scbd-dev-pr`.
