---
name: scbd-agent-epic
description: Runs one Jira epic iteration, including workspace, agents, verification, external state, evidence, and handoff. Use for interactive or unattended epic work.
---

# scbd-agent-epic

Complete one iteration of an epic's highest-priority actionable work.

**Usage:** `/scbd-agent-epic epic=<key> [component=<name>] [label=<label>] [mode=interactive|afk]`

Read [REFERENCE.md](REFERENCE.md) before operating.

## Arguments

| Parameter   | Default                    | Requirement                                |
| ----------- | -------------------------- | ------------------------------------------ |
| `epic`      | none                       | Required Jira epic key                     |
| `component` | project's `scbd_component` | Required after fallback                    |
| `label`     | `ready-for-agent`          | Intake filter for new `TO DO` tickets only |
| `mode`      | `interactive`              | `interactive` or `afk`                     |

Accept legacy positional arguments; prefer key-value arguments. Read `scbd_plan_dir` from the
project's `AGENTS.md`; default to `docs/plans`.

## One-iteration Contract

Resume interrupted work or complete one lifecycle action. Never continue to another action or ticket.

## Modes

In `interactive` mode, pause:

1. After selecting the proposed action.
2. After preparing or recovering the workspace.
3. After reviewing the focused worker's handoff and local output.
4. Before any external mutation.

In `afk` mode, cross these boundaries without routine confirmation. Always obey reference stop
conditions.

## Ownership

- Only this skill mutates Jira, git, or GitHub. Focused agents may edit files, test, and inspect those
  systems read-only.
- For local work, dispatch one fresh `scbd-agent-plan`, `scbd-agent-implement`, or
  `scbd-agent-review` agent. One correction retry belongs to the same iteration. Apply
  `scbd-agent-screenshot` in the epic agent's context; never delegate screenshot capture. The
  epic agent alone hosts accepted artifacts and updates the PR.
- Review output before publishing. After one failed correction, preserve state and stop for a human.
- Follow `/karpathy-guidelines` when reviewing plans and code.

## Completion

Verify, publish the selected action, print the reference handoff, and stop.
