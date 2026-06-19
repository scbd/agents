# Skills catalog

## Design philosophy — externally driven iterations

`scbd-agent-epic` performs exactly one unit of epic work and exits. A scheduler or human may invoke it again, but repetition is deliberately outside the skill. This keeps each run recoverable whether one context lives for a ticket lifecycle or only one iteration.

Interactive and AFK modes have the same one-iteration boundary. Interactive mode pauses for instructions at each phase boundary; AFK mode crosses those boundaries without routine confirmation.

Each skill follows this contract:
- Durable state lives in Jira, git history, PRs, and committed plan artifacts
- One plan, implementation, review cycle, recovery, or close-out per run
- Clean handoff so the caller can inspect state or invoke the epic skill again
- Focused skills may read Jira, git, and GitHub; only the epic skill mutates them

## Available skills

| Skill                      | Description                                                    | Dependencies          |
| -------------------------- | -------------------------------------------------------------- | --------------------- |
| `scbd-agent-epic`          | Orchestrate one Jira epic iteration and all external state     | `karpathy-guidelines` |
| `scbd-agent-plan`          | Plan one Jira ticket with read-only external context           | —                     |
| `scbd-agent-implement`     | Implement one Jira ticket locally, with or without a plan      | `karpathy-guidelines` |
| `scbd-agent-review`        | Address one Jira ticket's review cycle locally                 | `karpathy-guidelines` |
| `scbd-agent-pr-screenshot` | Capture and verify local screenshot evidence                   | —                     |

The epic skill dispatches the focused skills with a structured work order, reviews their local output, and handles all external bookkeeping. Each focused skill can also be invoked directly and hands uncommitted local work back to the human.

## Invoking skills

```bash
/scbd-agent-epic epic=DEV-20 mode=interactive       # component from AGENTS.md
/scbd-agent-epic epic=DEV-20 component=Gaia/KM mode=afk

/scbd-agent-plan ticket=DEV-123 plan=docs/plans/DEV-123.md
/scbd-agent-implement ticket=DEV-123                            # discover a plan first; implement directly if none exists
/scbd-agent-implement ticket=DEV-123 plan=docs/plans/DEV-123.md
/scbd-agent-review ticket=DEV-123
/scbd-agent-pr-screenshot output=/tmp/DEV-123-evidence
```

## Setting up a new project

Add this line to the project's `AGENTS.md` so the skills can pick up a default component:

```
scbd_component: Your/Component
scbd_plan_dir: docs/plans
```

`scbd_plan_dir` is optional and defaults to `docs/plans`.

## External dependencies

Install these once, globally:

| Skill                 | Source                              | Required by                                                         |
| --------------------- | ----------------------------------- | ------------------------------------------------------------------- |
| `karpathy-guidelines` | `multica-ai/andrej-karpathy-skills` | `scbd-agent-epic`, `scbd-agent-implement`, `scbd-agent-review`      |

```bash
npx skills add multica-ai/andrej-karpathy-skills --skill karpathy-guidelines -g
```

## Adding a new skill

1. Create `skills/<name>/SKILL.md` with YAML frontmatter:
   ```yaml
   ---
   name: <name>
   description: One-sentence description of what the skill does and when to use it.
   ---
   ```
2. Add a row to the skills table above.
3. List any external dependencies in the Dependencies table above.
