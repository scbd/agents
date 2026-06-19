---
name: scbd-agent-epic
description: Orchestrates exactly one unit of work from a Jira epic, owning Jira, git, GitHub, workspace preparation, sub-agent dispatch, verification, PR evidence, and handoff. Use when performing interactive step-by-step epic work or one unattended AFK iteration that an external process may invoke repeatedly.
---

# scbd-agent-epic

Assess an epic and complete exactly one iteration of its highest-priority actionable work.

**Usage:** `/scbd-agent-epic epic=<key> [component=<name>] [label=<label>] [mode=interactive|afk]`

Read [REFERENCE.md](REFERENCE.md) before operating. It defines the lifecycle state machine,
work-order contract, external logging, recovery rules, and handoff format.

## Arguments

| Parameter | Default | Requirement |
|-----------|---------|-------------|
| `epic` | none | Required Jira epic key |
| `component` | target project's `scbd_component` | Required after fallback |
| `label` | `ready-for-agent` | Intake filter for new `TO DO` tickets only |
| `mode` | `interactive` | `interactive` or `afk` |

Accept positional arguments for backward compatibility, but prefer key-value arguments. Read
`scbd_plan_dir` from the target project's `AGENTS.md`, defaulting to `docs/plans`.

## One-iteration Contract

One invocation selects and completes no more than one of these actions:

1. Recover interrupted local work.
2. Close out one merged PR.
3. Address one coherent review cycle, including plan feedback.
4. Implement one approved plan.
5. Plan one new unblocked ticket.

Never loop to another action or ticket. Repeated invocation belongs to a human or external process.

## Modes

In `interactive` mode, present findings and wait for instructions:

1. After selecting the proposed action.
2. After preparing or recovering the workspace.
3. After reviewing the focused worker's handoff and local output.
4. Before commits, pushes, Jira transitions, PR replies, evidence publication, or other external
   mutations.

In `afk` mode, cross those boundaries without routine confirmation. Stop in either mode when the
reference identifies a human-intervention condition.

## Ownership

- This skill alone may access Jira, run git commands, or interact with GitHub and PRs.
- Dispatch one primary fresh focused sub-agent with `scbd-agent-plan`, `scbd-agent-implement`, or
  `scbd-agent-review` when the selected action needs local work. A correction retry and an auxiliary
  `scbd-agent-pr-screenshot` capture remain part of that same lifecycle action, not new iterations.
- Focused agents edit local files and run local verification, but never use git or external services.
- Review focused-agent output before publishing it. Redispatch one fresh correction agent if needed;
  if the corrected result is still unsatisfactory, preserve state and hand over to the human.
- Follow `/karpathy-guidelines` when reviewing plans and code.

## Completion

Run final verification, perform the selected action's Jira/git/GitHub bookkeeping, and print the
structured handoff from the reference. Then stop, even when more epic work is available.
