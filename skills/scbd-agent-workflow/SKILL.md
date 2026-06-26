---
name: scbd-agent-workflow
description: Runs one Jira-backed workflow iteration, including workspace, agents, verification, external state, evidence, and handoff. Use for interactive or unattended SCBD agent workflow work.
---

# scbd-agent-workflow

Complete one iteration for a Jira key, treating epics as queues and issues as targeted lifecycle
work.

**Usage:** `/scbd-agent-workflow <jira-key> [component=<name>] [label=<label>] [mode=interactive|afk]`

Read [REFERENCE.md](REFERENCE.md) before operating.

## Arguments

| Parameter    | Default                    | Requirement                                      |
| ------------ | -------------------------- | ------------------------------------------------ |
| `<jira-key>` | none                       | Required Jira epic or issue key                  |
| `component`  | project's `scbd_component` | Required after fallback for epic queue selection |
| `label`      | `ready-for-agent`          | Intake filter for epic-mode new `TO DO` tickets  |
| `mode`       | `interactive`              | `interactive` or `afk`                           |

Prefer the Jira key as the first positional argument. Accept legacy `epic=<key>` and `ticket=<key>`
aliases, but normalize them to the same Jira key input. Accept `mode=<mode>` and `mode: <mode>`.
Read `scbd_plan_dir` from the project's `AGENTS.md`; default to `docs/plans`.

Inspect the Jira key before selecting work. If it is an epic, select the next actionable issue from
that epic/component queue. If it is an issue, do not scan for other work; resume that issue's
lifecycle and perform the next matching action.

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

- This skill coordinates the workflow bookkeeping across Jira, git, and GitHub. Companion skills may
  read or mutate those systems when explicitly invoked for their responsibility.
- Load `scbd-agent-jira` before Jira state changes and `scbd-agent-github` before git/GitHub
  branch, PR, review reply, evidence hosting, or close-out work.
- For local work, dispatch one fresh `scbd-agent-plan`, `scbd-agent-implement`, or
  `scbd-agent-review` agent. One correction retry belongs to the same iteration. Apply
  `scbd-agent-screenshot` in the workflow agent's context; never delegate screenshot capture. Host
  accepted artifacts and update the PR through `scbd-agent-github`.
- Review output before publishing. After one failed correction, preserve state and stop for a human.
- Follow `/karpathy-guidelines` when reviewing plans and code.

## Completion

Verify, publish the selected action, print the reference handoff, and stop.
