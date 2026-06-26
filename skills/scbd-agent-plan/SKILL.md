---
name: scbd-agent-plan
description: Writes one uncommitted ticket plan using read-only Jira, git, and GitHub context. Use for delegated or direct Jira ticket planning.
---

# scbd-agent-plan

Plan one ticket and leave the plan file uncommitted for the caller.

**Usage:** `/scbd-agent-plan ticket=<key> [plan=<path>]`

## Inputs

Require `ticket=<key>` or a work order. Supplied context, criteria, constraints, lifecycle, path, and
stop conditions are authoritative. Resolve gaps from Jira, local files, read-only git, and linked PRs;
ask only for undiscoverable essentials.

Use `plan=<path>`, else `AGENTS.md`'s `scbd_plan_dir`, else `docs/plans/<ticket-key>.md`.

## Workflow

1. Read instructions, ticket context, relevant architecture, tests, and conventions.
2. Resolve discoverable questions; surface assumptions needing caller input.
3. Write and review a decision-complete plan: intent, approach, behavior, tests, risks, and criteria.
4. Do not implement or edit unrelated files.

## Boundaries

- Jira, git, and GitHub are read-only. Do not fetch, pull, switch, stage, commit, push, stash, reset,
  clean, mutate Jira, or mutate PRs.
- Do not begin implementation.
- Plan exactly one ticket per invocation.

## Handoff

```text
Outcome: completed | blocked
Summary: <planning result>
Files: <plan path and any other changed paths>
Decisions: <key decisions and assumptions>
Verification: <sources inspected and plan review>
User-facing impact: <expected impact or none>
Proposed external replies: <usually none>
Blockers: <none or details>
Next action: <recommended caller action>
```
