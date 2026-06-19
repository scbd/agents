---
name: scbd-agent-plan
description: Creates one implementation plan from a Jira ticket or delegated work order, using read-only Jira, git, and GitHub context, and leaves it uncommitted. Use when scbd-agent-epic delegates planning or a human wants a concrete local plan for one ticket.
---

# scbd-agent-plan

Plan one ticket and leave the plan file uncommitted for the caller.

**Usage:** `/scbd-agent-plan ticket=<key> [plan=<path>]`

## Inputs

When delegated, use the work order's ticket identity, task brief, acceptance criteria, constraints,
current lifecycle state, target plan path, and stop conditions. Read external context when it is
needed to resolve or verify the work order, but do not mutate it.

When invoked directly, require `ticket=<key>`. Read the Jira ticket, including its description,
acceptance criteria, status, dependencies, and relevant comments. Inspect read-only git history and
workspace state plus any linked GitHub PRs when they clarify prior work or constraints. Ask the human
only for essential context that these sources cannot resolve.

Use the supplied `plan=<path>`. Otherwise use the target project's `scbd_plan_dir` setting from
`AGENTS.md`, falling back to `docs/plans/<ticket-key>.md`.

## Workflow

1. Read project instructions, ticket context, and relevant local architecture, tests, and conventions.
2. Resolve discoverable questions from Jira, read-only git and GitHub context, and local files;
   surface assumptions that require caller input.
3. Write a decision-complete plan covering intent, implementation approach, affected behavior,
   tests, risks, and acceptance criteria.
4. Do not implement feature code or modify unrelated files.
5. Stop after writing and reviewing the plan.

## Boundaries

- Jira, git, and GitHub access is read-only. Do not transition, assign, label, or comment on Jira;
  mutate git state or the worktree through git; or create, edit, review, comment on, or merge a PR.
- Do not fetch, pull, switch branches, stage, commit, push, stash, reset, clean, or open a PR.
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
