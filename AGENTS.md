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

| Skill                  | Description                                                                                                | Dependencies          |
| ---------------------- | ---------------------------------------------------------------------------------------------------------- | --------------------- |
| `scbd-agent-implement` | Pick and implement the next unblocked Jira ticket end-to-end                                               | `karpathy-guidelines` |
| `scbd-agent-review`    | Address peer-review comments on in-progress PRs                                                            | `karpathy-guidelines` |
| `scbd-push-to-jira`    | Upload local markdown issues from `.scratch/<feature-slug>/` to Jira (Epic + Stories + blocked-by links)  | —                     |

`scbd-agent-implement` and `scbd-agent-review` accept `<epic> [component] [label]` arguments. They read `scbd_component:` from the **target project's** `AGENTS.md` as a default component filter when none is passed.

`scbd-push-to-jira` bridges the mattpocock planning tools and the `scbd-agent-*` implementation tools:

1. **`/to-prd`** (mattpocock) — generates a PRD and issue files, saves them to `.scratch/<feature-slug>/` on local disk.
2. **Human review** — the developer reviews and edits `.scratch/<feature-slug>/` directly, optionally with agent assistance. The local-markdown save exists specifically for this gate: nothing reaches Jira until a human has signed off. This step is required.
3. **`/scbd-push-to-jira <feature-slug>`** — reads those local files and creates one Epic and one Story per issue in Jira, wires `Blocks` links, and writes Jira keys back into every markdown file.
4. **`/scbd-agent-implement`** / **`/scbd-agent-review`** — pick up the Jira tickets created in step 3 and implement them.

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

/scbd-push-to-jira my-feature-slug
/scbd-push-to-jira my-feature-slug project=DEV component="Gaia/KM" label=ready-for-agent
/scbd-push-to-jira my-feature-slug --force   # re-upload tickets that already have Jira keys
```

## Key conventions enforced by the skills

- **Branch naming:** `feature/<ticket-key>-<short-slug>`
- **Commits:** Conventional Commits — `feat`, `fix`, `refactor`, `test`, `docs`, `chore`
- **Review loop:** Every addressed PR comment must include `#done` in the reply to prevent re-processing on the next agent run
- **PR state:** Always draft on creation; never push to `main`
- **Jira sync:** Ticket status transitions (`IN PROGRESS` → `PEER REVIEW` → `Completed`) must stay in sync with PR state at every phase boundary

## Setting up a new project to use these skills

Add these lines to the target project's `AGENTS.md`:

```
scbd_component:     Your/Component    # used by scbd-agent-* and scbd-push-to-jira
scbd_jira_project:  DEV               # used by scbd-push-to-jira (avoids being asked each run)
```

## Adding a new skill

1. Create `skills/<name>/SKILL.md` with the YAML frontmatter above
2. Add a row to the skills table in `skills/README.md`
3. List any external skill dependencies in the `skills/README.md` Dependencies table

## Markdown rules

- Keep table columns aligned in raw text mode.
