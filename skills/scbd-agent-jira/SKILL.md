---
name: scbd-agent-jira
description: Handles Jira state, labels, links, blockers, and audit comments for SCBD agent workflows. Use when mutating or validating Jira state for SCBD tickets, epics, and PR-linked work.
---

# scbd-agent-jira

Operate on Jira for SCBD agent workflows. Read-only use is allowed for focused agents; mutations
belong to the workflow agent or a human explicitly operating this skill.

**Usage:** `/scbd-agent-jira ticket=<key> [epic=<key>] action=<assess|start|peer-review|complete|log|link-pr>`

## Inputs

Require a ticket or epic key and the intended action. Resolve component from arguments, then project
`AGENTS.md`'s `scbd_component`. Default intake label is `ready-for-agent`.

Before mutating, inspect the ticket, epic relationship, status, labels, assignee, dependencies,
linked PRs, recent Jira comments, and the matching local/PR state supplied by the caller.

## State Rules

- `TO DO` plus intake label is eligible for planning when unblocked.
- `IN PROGRESS` means planned or implementing work is active.
- `PEER REVIEW` means an open PR is the source of truth.
- `Completed` means the matching PR is merged or the work was explicitly closed.
- An `is blocked by` target blocks unless its status is `Done` or `Completed`.
- Replace `ready-for-agent` with `ready-for-human` after implementation publication.
- Keep Jira coherent with PR state at phase boundaries.

## Mutations

- Start planning: transition to `IN PROGRESS`, assign the current Jira user, and log the selected
  plan path/branch if known.
- Publish implementation: transition to `PEER REVIEW`, update labels, link or note the PR, and log
  verification/evidence summary.
- Close out: transition to `Completed` only after verifying the exact merged PR-ticket pair.
- Link PR milestones using the relationship or link convention available in the project.
- If preparation fails after a Jira mutation, add a concise failure/blocker comment before stopping.

## Comments

Use Jira for milestones, state changes, links, blockers, and major product decisions. Avoid
implementation detail that belongs in commits or PR comments.

When posting on behalf of a GitHub user, append the established attribution signature. Use
`AFK Agent` in `mode=afk`; use `HITL Agent` in `mode=interactive`.

```text
🤖 *Posted by <AFK Agent | HITL Agent> on behalf of @<GitHub username>*
```

Comment format:

```text
Agent update: <action>
State: <previous> -> <current>
PR: <url or none>
Verification: <summary>
Blockers: <none or details>
Next: <human or workflow action>
Attribution: 🤖 *Posted by <AFK Agent | HITL Agent> on behalf of @<GitHub username>*
```

## Stop

Stop without further mutation for unavailable credentials/transitions, ambiguous ticket ownership,
blocked work, mismatched PR or branch, missing component, red verification, or human judgment.

## Handoff

```text
Outcome: completed | blocked | stopped
Ticket: <key and summary>
Jira: <status, labels, assignee, links, comments, transitions>
Decisions: <important assumptions>
Blockers: <none or details>
Next action: <recommended caller action>
```
