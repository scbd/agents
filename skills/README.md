# Skills catalog

## Design philosophy — externally driven iterations

`scbd-agent-epic` performs one epic unit and exits. A human or scheduler owns repetition.

Both modes stop after one iteration. Interactive pauses at phase boundaries; AFK does not.

Contract:

- Durable state lives in Jira, git history, PRs, and committed plan artifacts
- One plan, implementation, review, recovery, or close-out per run
- A handoff ends every run
- Focused skills may read Jira, git, and GitHub; only the epic skill mutates them

## Available skills

| Skill                      | Description                                                    | Dependencies          |
| -------------------------- | -------------------------------------------------------------- | --------------------- |
| `scbd-agent-epic`          | Orchestrate one Jira epic iteration and all external state     | `karpathy-guidelines` |
| `scbd-agent-plan`          | Plan one Jira ticket with read-only external context           | —                     |
| `scbd-agent-implement`     | Implement one Jira ticket locally, with or without a plan      | `karpathy-guidelines` |
| `scbd-agent-review`        | Address one Jira ticket's review cycle locally                 | `karpathy-guidelines` |
| `scbd-agent-pr-screenshot` | Capture and verify local screenshot evidence                   | —                     |

The epic skill dispatches and reviews focused agents, then handles external state. Focused skills also
run directly and return uncommitted work.

## Invoking skills

```bash
/scbd-agent-epic epic=DEV-20 mode=interactive    # component from AGENTS.md
/scbd-agent-epic epic=DEV-20 component=Gaia/KM mode=afk

/scbd-agent-plan ticket=DEV-123 plan=docs/plans/DEV-123.md
/scbd-agent-implement ticket=DEV-123             # discover plan, else implement directly
/scbd-agent-implement ticket=DEV-123 plan=docs/plans/DEV-123.md
/scbd-agent-review ticket=DEV-123
/scbd-agent-pr-screenshot output=/tmp/DEV-123-evidence
```

## Setting up a new project

Add to the project's `AGENTS.md`:

```
scbd_component: Your/Component
scbd_plan_dir: docs/plans
```

`scbd_plan_dir` is optional and defaults to `docs/plans`.

## Workflow conventions

- **Branch naming:** `feature/<ticket-key>-<short-slug>`
- **Commits:** Conventional Commits — `feat`, `fix`, `refactor`, `test`, `docs`, `chore`
- **Review:** Addressed comments get `#done` replies
- **Iteration:** One epic invocation; callers own repetition
- **Modes:** Interactive pauses at boundaries; AFK does not
- **Ownership:** Focused skills read external systems; only epic mutates or hosts evidence
- **PR state:** Agents keep new PRs draft and never push to `main`; only humans mark PRs ready
- **Jira sync:** Ticket status (`IN PROGRESS` → `PEER REVIEW` → `Completed`) stays synchronized with PR state

## External dependencies

Install these once, globally:

| Skill                 | Source                              | Required by                                                         |
| --------------------- | ----------------------------------- | ------------------------------------------------------------------- |
| `karpathy-guidelines` | `multica-ai/andrej-karpathy-skills` | `scbd-agent-epic`, `scbd-agent-implement`, `scbd-agent-review`      |

```bash
npx skills add multica-ai/andrej-karpathy-skills --skill karpathy-guidelines -g
```
