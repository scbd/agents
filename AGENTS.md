# AGENTS.md

This file provides guidance to AI Coding Agent when working with code in this repository.

## Purpose

This repository is a collection of reusable AI agent skill definitions — plain-Markdown instruction files that Claude Code (and other supported agents) can invoke as slash commands. Skills are installed globally to a developer's machine, not bundled into any application.

## Skill structure

Each skill lives at `skills/<name>/SKILL.md` with mandatory YAML frontmatter:

```yaml
---
name: <name>
description: One-sentence description of what the skill does and when to use it.
---
```

The `name` must match the directory name. The `description` is what the harness uses to decide when to auto-trigger the skill.

## Available skills

| Skill                      | Description                                                    | Dependencies          |
| -------------------------- | -------------------------------------------------------------- | --------------------- |
| `scbd-agent-epic`          | Orchestrate one Jira epic iteration and all external state     | `karpathy-guidelines` |
| `scbd-agent-plan`          | Create one local implementation plan                           | —                     |
| `scbd-agent-implement`     | Implement one approved plan locally                            | `karpathy-guidelines` |
| `scbd-agent-review`        | Address one supplied review cycle locally                      | `karpathy-guidelines` |
| `scbd-agent-pr-screenshot` | Capture and verify local screenshot evidence                   | —                     |

Only `scbd-agent-epic` interacts with Jira, git, or GitHub. It accepts `epic=`, `component=`, `label=`, and `mode=` arguments. It reads `scbd_component:` and optional `scbd_plan_dir:` defaults from the **target project's** `AGENTS.md`.

## Installing dependencies

```bash
# External skill used by the epic, implementation, and review skills
npx skills add multica-ai/andrej-karpathy-skills --skill karpathy-guidelines -g

# Update all installed skills
npx skills update -g
```

## Invoking skills

```bash
/scbd-agent-epic epic=DEV-20 mode=interactive
/scbd-agent-epic epic=DEV-20 component=Gaia/KM mode=afk

/scbd-agent-plan ticket=DEV-123 plan=docs/plans/DEV-123.md
/scbd-agent-implement plan=docs/plans/DEV-123.md
/scbd-agent-review
```

## Key conventions enforced by the skills

- **Branch naming:** `feature/<ticket-key>-<short-slug>`
- **Commits:** Conventional Commits — `feat`, `fix`, `refactor`, `test`, `docs`, `chore`
- **Review loop:** Every addressed PR comment must include `#done` in the reply to prevent re-processing on the next agent run
- **Iteration boundary:** Every epic invocation performs exactly one iteration; an external caller owns repetition
- **Mode behavior:** Interactive pauses at phase boundaries; AFK completes the same iteration without routine pauses
- **External ownership:** Only the epic skill may access Jira, git, GitHub, commits, pushes, PRs, or evidence hosting
- **PR state:** Create plan PRs as drafts, mark them ready after implementation, and never push to `main`
- **Jira sync:** Ticket status transitions (`IN PROGRESS` → `PEER REVIEW` → `Completed`) must stay in sync with PR state at every phase boundary

## Setting up a new project to use these skills

Add this line to the target project's `AGENTS.md`:

```
scbd_component: Your/Component
scbd_plan_dir: docs/plans
```

## Adding a new skill

1. Create `skills/<name>/SKILL.md` with the YAML frontmatter above
2. Add a row to the skills table in `skills/README.md`
3. List any external skill dependencies in the `skills/README.md` Dependencies table

## Markdown rules

- Keep table columns aligned in raw text mode.
