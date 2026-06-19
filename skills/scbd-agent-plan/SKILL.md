---
name: scbd-agent-plan
description: Creates one implementation plan from a supplied ticket brief using only the local filesystem and leaves it uncommitted. Use when scbd-agent-epic delegates planning or when a human wants a concrete local plan without Jira, GitHub, or git interaction.
---

# scbd-agent-plan

Plan one ticket and leave the plan file uncommitted for the caller.

**Usage:** `/scbd-agent-plan ticket=<key> [plan=<path>]`

## Inputs

When delegated, require a work order containing the ticket identity, task brief, acceptance
criteria, constraints, current lifecycle state, target plan path, and stop conditions.

When invoked directly, derive those fields from the user's prompt and local files. Ask the human
for essential missing ticket context. Do not fetch it from Jira, GitHub, or git.

Use the supplied `plan=<path>`. Otherwise use the target project's `scbd_plan_dir` setting from
`AGENTS.md`, falling back to `docs/plans/<ticket-key>.md`.

## Workflow

1. Read project instructions and inspect the relevant local architecture, tests, and conventions.
2. Resolve discoverable questions from local files; surface assumptions that require caller input.
3. Write a decision-complete plan covering intent, implementation approach, affected behavior,
   tests, risks, and acceptance criteria.
4. Do not implement feature code or modify unrelated files.
5. Stop after writing and reviewing the plan.

## Boundaries

- Do not run any git command, including read-only commands.
- Do not access Jira or GitHub and do not use their CLIs or APIs.
- Do not stage, commit, push, open a PR, or begin implementation.
- Plan exactly one ticket per invocation.

## Handoff

```text
Outcome: completed | blocked
Summary: <planning result>
Files: <plan path and any other changed paths>
Decisions: <key decisions and assumptions>
Verification: <local sources inspected and plan review>
User-facing impact: <expected impact or none>
Proposed external replies: <usually none>
Blockers: <none or details>
Next action: <recommended caller action>
```
