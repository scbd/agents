# Shared AI Agent Tools

Reusable tools for AI coding agents — skills, prompts, configurations, and conventions — installable to any developer machine with a single command.

Everything in this repository is agent-portable and distributed globally, not bundled into any single project.

## Install

Install the skills from this repository globally:

```bash
npx skills add scbd/agents -g
```

## Prerequisites

- [`acli`](https://developer.atlassian.com/cloud/acli/) installed and authenticated against your
  Jira site (`acli jira auth status`). Jira skills use `acli` only — no REST calls, no tokens in
  `~/.netrc`.
- [`gh`](https://cli.github.com/) installed and authenticated (`gh auth status`).
- The external skills that the bundled skills depend on:

  ```bash
  npx skills add multica-ai/andrej-karpathy-skills --skill karpathy-guidelines -g
  ```

- A one-time, per-machine git ignore for `.scratch/`, the directory skills use for plans,
  screenshots, and other working files. This covers every project without touching any repo's own
  `.gitignore`:

  ```bash
  mkdir -p ~/.config/git && echo '.scratch/' >> ~/.config/git/ignore
  ```

  A project that wants to commit `.scratch/` content can add a negation to its own `.gitignore`.

## Personal preferences

Each person tunes how often agents ask before committing, pushing, opening PRs, replying, or
changing Jira in `~/.config/scbd-agents/preferences.md`. See
[Checkpoints and preferences](skills/README.md#checkpoints-and-preferences).

## Update

Keep all installed skills up to date:

```bash
npx skills update -g
```

## Migrating from `scbd-agent-*`

The `scbd-agent-*` skills were replaced by task-shaped `scbd-dev-*` commands, plus the shared
`scbd-jira` and `scbd-github` skills. `npx skills update -g` does not remove renamed skills, so
existing installs need one manual cleanup step:

```bash
npx skills remove scbd-agent-workflow scbd-agent-plan scbd-agent-implement scbd-agent-review \
  scbd-agent-screenshot scbd-agent-jira scbd-agent-github -g -y
npx skills add scbd/agents -g
```

This also removes any hand-installed REST-based `scbd-agent-jira` copy — Jira access now goes
through `acli` only.

## Skills catalog

See [skills/README.md](skills/README.md) for the full catalog, invocation examples, workflow
conventions, and target-project setup.

## Recommended third-party skills

The [mattpocock/skills](https://github.com/mattpocock/skills) collection is considered part of the standard AI-assisted development setup and should also be installed:

```bash
npx skills add mattpocock/skills
```

## Supported agents

See the [Vercel Labs skills README](https://github.com/vercel-labs/skills#readme) for the list of agents that can run these skills.
