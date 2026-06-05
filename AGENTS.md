# AGENTS.md

This file provides guidance to AI Conding Agent when working with code in this repository.

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

| Skill                  | Description                                                  | Dependencies          |
| ---------------------- | ------------------------------------------------------------ | --------------------- |
| `scbd-agent-implement` | Pick and implement the next unblocked Jira ticket end-to-end | `karpathy-guidelines` |
| `scbd-agent-review`    | Address peer-review comments on in-progress PRs              | `karpathy-guidelines` |

Both skills accept `<epic> [component] [label]` arguments. They read `scbd_component:` from the **target project's** `AGENTS.md` as a default component filter when none is passed.

## Installing dependencies

```bash
# External skill required by both scbd-agent-* skills
npx skills add multica-ai/andrej-karpathy-skills --skill karpathy-guidelines -g

# Update all installed skills
npx skills update -g
```

## Invoking skills

```bash
/scbd-agent-implement DEV-20               # uses scbd_component from AGENTS.md
/scbd-agent-implement DEV-20 Gaia/KM
/scbd-agent-review DEV-20 Gaia/KM my-label
```

## Key conventions enforced by the skills

- **Branch naming:** `feature/<ticket-key>-<short-slug>`
- **Commits:** Conventional Commits — `feat`, `fix`, `refactor`, `test`, `docs`, `chore`
- **Review loop:** Every addressed PR comment must include `#done` in the reply to prevent re-processing on the next agent run
- **PR state:** Always draft on creation; never push to `main`
- **Jira sync:** Ticket status transitions (`IN PROGRESS` → `PEER REVIEW` → `Completed`) must stay in sync with PR state at every phase boundary

## Setting up a new project to use these skills

Add this line to the target project's `AGENTS.md`:

```
scbd_component: Your/Component
```

## Adding a new skill

1. Create `skills/<name>/SKILL.md` with the YAML frontmatter above
2. Add a row to the skills table in `skills/README.md`
3. List any external skill dependencies in the `skills/README.md` Dependencies table

## Markdown rules

- Keep table columns aligned in raw text mode.
