---
name: scbd-dev-agent
description: Work on SCBD development tasks with a human — the entry point for ad-hoc or open-ended work that doesn't fit one specific command yet ("let's work on DEV-123 together", "help me with this refactor, no ticket"). Also loaded by every scbd-dev-* command for its shared ground rules.
---

# scbd-dev-agent

Ground rules for SCBD development work, and the starting point when a request doesn't yet match one
specific `scbd-dev-*` command.

**Usage:** load this directly for open-ended work, or let a `scbd-dev-*` command load it for its
shared rules.

## Context resolution

Resolve the work item in this order:

1. An explicit key argument.
2. The current branch name, `feature/<KEY>-*`.
3. A key mentioned in the conversation.
4. None: ticketless work.

With no ticket:

- Skip every Jira step without comment.
- Name plan and screenshot files, and the branch, after a short slug (`feature/<slug>`).

Only `scbd-dev-start` and `scbd-dev-next` treat an epic key as a queue. Everywhere else, a key means
one issue, or there's no ticket at all.

## Action policy

| Tier               | Examples                                                                                                   | Rule                                                          |
| ------------------ | ---------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| Local              | Read, edit, test, create/switch a branch                                                                   | Go ahead                                                      |
| Local history      | Commit on a feature branch                                                                                 | Only once the human has handed over autonomy, or asks         |
| Leaves the machine | Push, create/edit PR, PR comments/replies, any Jira change                                                 | List the exact actions, wait for OK, then run them and verify |
| Never              | Push to the default branch, mark a PR ready, force-push, `reset --hard`, discard or stash unexplained work | Refuse and explain                                            |

- A command whose whole purpose is publishing still shows the itemised actions first. One
  confirmation covers the whole list, and the human can edit it.
- Stage explicit paths only. Never `git add -A` or `git add .`, and never stage `.scratch/`.
- A dirty or foreign workspace means stop and ask. Never guess who owns the work.

## `.scratch/` layout

```text
.scratch/
  plans/<key-or-slug>.md
  screenshots/<key-or-slug>/NN-<scenario>.png
```

Before first writing to `.scratch/`, run `git check-ignore -q .scratch`. If the directory is not
ignored, say so once and offer the one-time global ignore step: `mkdir -p ~/.config/git && echo
'.scratch/' >> ~/.config/git/ignore`. Never edit `.gitignore` without being asked. Explicit-path
staging is the real safeguard; the ignore only removes noise.

## Delegation

- Default to the current conversation.
- When a plan has two or more independent steps, offer to hand them to sub-agents. Name the steps
  and why, then let the human choose.
- A delegated sub-agent gets a self-contained brief:

  ```text
  Step: <the step text from the plan>
  Files: <files involved>
  Checks: <what to run>
  Rules: local edits only, no commits, report back
  ```

## Learning

At the end of a task, if the human corrected how it ran, offer once to save the correction:

- **Personal preference** (autonomy level, commit granularity, delegation habits, verbosity): save
  it to your persistent memory, if you keep one.
- **Project fact** (component, test commands, how to run the app and log in for screenshots): offer
  a diff to the project's `AGENTS.md`, so teammates benefit too.

Read both sources first for preferences that already apply. Don't nag: offer once per correction,
and never offer after an uncorrected run.

## Open-ended work

When the request matches one `scbd-dev-*` command, name it and offer to run it instead of
improvising. When it doesn't — exploratory work, a mix of steps, a status question — work through it
directly in this conversation, using the rules above.
