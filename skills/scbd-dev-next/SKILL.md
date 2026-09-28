---
name: scbd-dev-next
description: Show where a ticket, epic, or the current branch stands in the development workflow, and recommend the next command. Use for "what's next on DEV-123", "what's next in this epic", or checking status before deciding what to do.
---

# scbd-dev-next

Report status and recommend one next command. Changes nothing itself.

**Usage:** `/scbd-dev-next [<jira-key>]`

## Before you start

Load `scbd-dev-agent` for the shared ground rules. Regardless:

- Nothing leaves the machine (push, PR, comments, Jira) without listing the actions and getting OK.
- Never push to the default branch, mark a PR ready, force-push, or discard unexplained work.
- Stage explicit paths only; never stage `.scratch/`.

## Inspect

The key (epic or issue), or the current branch and its PR if no key is given, plus workspace state.

## Rules (first match wins)

1. PR merged, Jira not `Done` → `/scbd-dev-finish`
2. Open PR with unresolved threads where no reply contains `#done` → `/scbd-dev-feedback`
3. Feature branch with a plan and no implementation → `/scbd-dev-implement`
4. Feature branch with implementation and no PR → `/scbd-dev-screenshot` if the UI changed, then
   `/scbd-dev-pr`
5. Issue in `To Do`, or an epic key → `/scbd-dev-start`
6. Otherwise, report the state and say nothing is obviously next

## Output

A short status, the recommended command, and why.

## Changes nothing itself

If the human says go, run that one command. Never chain further commands without asking again.
