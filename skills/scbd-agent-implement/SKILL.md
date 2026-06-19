---
name: scbd-agent-implement
description: Implements a Jira ticket from a supplied or discovered plan, or directly from ticket context when no plan exists, using read-only Jira, git, and GitHub access and leaving local changes uncommitted. Use when scbd-agent-epic delegates implementation or a human directly requests implementation of one ticket.
---

# scbd-agent-implement

Implement one ticket from an explicit plan, a confirmed discovered plan, or ticket context, and leave
the result uncommitted for the caller.

**Usage:** `/scbd-agent-implement ticket=<key> [plan=<path>]`

## Inputs

When delegated, use the work order's ticket, task brief, acceptance criteria, approved plan,
constraints, expected verification, and stop conditions. The plan may be inline or supplied with
`plan=<path>`. Read external context when needed to resolve or verify the work order, but do not
mutate it.

When invoked directly, require `ticket=<key>`. Read the Jira ticket and use read-only git and GitHub
inspection to understand its current branch, linked PR, prior work, and review state. If
`plan=<path>` is supplied, validate and follow it. Otherwise search the local filesystem for a plan
matching the ticket key, starting with `scbd_plan_dir` from `AGENTS.md` and `docs/plans`. Validate
matches against the ticket. If exactly one plan matches, summarize it and obtain explicit human
confirmation before using it. If several match, ask the human to choose. Only when no matching plan
exists should the skill derive a decision-complete implementation approach from the ticket, project
instructions, codebase, and external context. Ask the human when a material product or technical
decision cannot be resolved from those sources.

## Workflow

1. Read the ticket, relevant project instructions, and linked PR context. Use the supplied plan when
   present. Otherwise search for a matching local plan and pause for explicit confirmation before
   using it. If none exists, inspect the relevant code and tests and settle the implementation
   approach before editing.
2. Inspect the current workspace with read-only git commands. If it is not safely associated with
   the ticket, stop rather than switching branches or changing git state.
3. Follow `/karpathy-guidelines` while making the smallest complete implementation.
4. Add or update tests required by the change.
5. Run the most relevant verification available locally. Fix failures caused by the work; report
   unrelated or unresolved failures clearly.
6. Stop without removing plan files, preparing screenshots, staging files, committing, or handing
   work to another phase.

## Boundaries

- Jira, git, and GitHub access is read-only. Do not transition, assign, label, or comment on Jira;
  mutate git state or the worktree through git; or create, edit, review, comment on, or merge a PR.
- Do not fetch, pull, switch branches, stage, commit, push, stash, reset, or clean.
- Preserve every supplied or discovered plan file. Identify the plan used in the handoff so the epic
  agent or human can decide whether to remove it.
- Do not capture, host, or prepare PR evidence. Report user-facing impact so the caller can decide
  whether to invoke `scbd-agent-pr-screenshot`.
- Work on exactly one implementation per invocation.

## Handoff

Return these fields to the epic agent or human:

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
