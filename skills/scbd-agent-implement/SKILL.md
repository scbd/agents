---
name: scbd-agent-implement
description: Implements one Jira ticket, with or without a plan, leaving changes uncommitted and external systems untouched. Use for delegated or direct ticket implementation.
---

# scbd-agent-implement

Implement one ticket and leave changes uncommitted.

**Usage:** `/scbd-agent-implement ticket=<key> [plan=<path>]`

## Inputs

Require `ticket=<key>` or a work order. Supplied context, criteria, constraints, verification, stop
conditions, and approved plan are authoritative. Resolve gaps from Jira, local files, read-only git,
and GitHub; ask only for undiscoverable essentials.

Validate an inline or supplied plan. Otherwise search `scbd_plan_dir`, then `docs/plans`, for the
ticket key. Validate matches against Jira. Confirm the sole match; ask the human to choose among
several. If none exists, derive a decision-complete approach. Ask about unresolved material decisions.

## Workflow

1. Read the ticket, instructions, linked PR, plan or approach, code, and tests.
2. Inspect the workspace read-only. Stop if it is not safely associated with the ticket.
3. Follow `/karpathy-guidelines` while making the smallest complete implementation.
4. Add or update tests required by the change.
5. Run relevant local verification. Fix caused failures; report others.
6. Stop without removing plans, capturing evidence, staging, committing, or starting another phase.

## Boundaries

- Jira, git, and GitHub are read-only. Do not fetch, pull, switch, stage, commit, push, stash, reset,
  clean, mutate Jira, or mutate PRs.
- Preserve plans and identify the one used.
- Do not prepare PR evidence; report user-facing impact.
- Work on exactly one implementation per invocation.

## Handoff

Return these fields to the workflow agent or human:

```text
Outcome: completed | blocked
Summary: <what changed>
Plan: <path or none; preserved for caller>
Files: <created, changed, and deleted paths>
Decisions: <technical decisions and assumptions>
Verification: <commands and results>
User-facing impact: <none or a concise description>
Proposed external replies: <usually none>
Blockers: <none or details>
Next action: <recommended caller action>
```
