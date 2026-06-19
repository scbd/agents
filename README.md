# Shared AI Agent Tools

Reusable tools for AI coding agents — skills, prompts, configurations, and conventions — installable to any developer machine with a single command.

Everything in this repository is agent-portable and distributed globally, not bundled into any single project.

## Install

Install the skills from this repository globally:

```bash
npx skills add scbd/agents -g
```

## Prerequisites

Install the external skills that the bundled skills depend on:

```bash
npx skills add multica-ai/andrej-karpathy-skills --skill karpathy-guidelines -g
```

## Update

Keep all installed skills up to date:

```bash
npx skills update -g
```

## Skills catalog

See [skills/README.md](skills/README.md) for the full list of available skills, invocation examples, and instructions for adding new ones.

The `scbd-agent-epic` skill orchestrates one Jira epic iteration and owns Jira, git, GitHub, and PR evidence. Its focused plan, implementation, review, and screenshot skills work only with local files and return uncommitted handoffs.

## Recommended third-party skills

The [mattpocock/skills](https://github.com/mattpocock/skills) collection is considered part of the standard AI-assisted development setup and should also be installed:

```bash
npx skills add mattpocock/skills
```

## Supported agents

See the [Vercel Labs skills README](https://github.com/vercel-labs/skills#readme) for the list of agents that can run these skills.
