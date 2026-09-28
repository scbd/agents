---
name: scbd-dev-start
description: Start work on a Jira ticket, pick the next ready ticket from an epic, or start ticketless work — moves the ticket to In Progress, assigns it, and creates the feature branch. Use for "let's start on DEV-123", "pick up the next ticket in this epic", or beginning work with no ticket at all.
---

# scbd-dev-start

Pick up one piece of work and get a feature branch ready for it.

**Usage:** `/scbd-dev-start [<jira-key>] [<description>] [yes]`

## Before you start

Load `scbd-dev-agent` for the shared ground rules. Regardless:

- Commits, pushes, PRs, replies and Jira changes pass a checkpoint (`scbd-github`, `scbd-jira`).
  With no preference set, ask before anything leaves the machine.
- Never push to the default branch, mark a PR ready, force-push, or discard unexplained work.
- Stage explicit paths only; never stage `.scratch/`.

## Finding the work

- **Epic key:** search the epic for candidates:
  `parent = <KEY> AND status = "To Do" AND labels = ready-for-agent`, adding
  `component = <scbd_component>` when the project's `AGENTS.md` sets one. Order by priority, then
  created date. Drop blocked candidates (see `scbd-jira`'s blocker check). Show the top few with
  their blockers, and let the human pick one.
- **Issue key:** use it directly. If it's already `In Progress` and a `feature/<KEY>-*` branch
  exists, resume by switching to that branch instead of creating a new one.
- **No key:** ask for a one-line description and derive a short slug from it.

## Steps

1. Check the workspace. Stop if it's dirty or on an unrelated branch — see `scbd-dev-agent`'s
   "dirty or foreign workspace" rule.
2. Fetch the default branch (`scbd-github`).
3. Create `feature/<KEY>-<slug>` (or `feature/<slug>` with no ticket) from it.
4. Transition the ticket to `In Progress` and assign it with `@me`, under the `jira` checkpoint
   (`scbd-jira`). Running this command is a direct request, so `invoked` goes ahead.

## Report

- The branch created or resumed.
- The ticket's summary and acceptance criteria, trimmed to what matters.
- A suggested next step: `/scbd-dev-plan` for anything non-trivial, otherwise `/scbd-dev-implement`.
