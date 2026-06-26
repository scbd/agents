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

## Workflow documentation

Use [`skills/README.md`](skills/README.md) as the canonical catalog for skill inventory, invocation,
target-project setup, dependencies, and workflow conventions. Update it whenever a skill's public
interface or workflow policy changes; do not duplicate those details here.

## Adding a new skill

1. Create `skills/<name>/SKILL.md` with the YAML frontmatter above
2. Add a row to the skills table in `skills/README.md`
3. List any external skill dependencies in the `skills/README.md` Dependencies table

## Markdown rules

- Keep table columns aligned in raw text mode.
