---
name: scbd-dev-screenshot
description: Capture screenshots that show a UI change, save them to .scratch/, and write PR-ready markdown for the human to finish by uploading the images. Use for "take a screenshot of this" or "get evidence for the PR" once a UI change is in place.
---

# scbd-dev-screenshot

Capture publication-ready screenshots locally. Read-only git is fine to understand the change; no
GitHub or Jira access.

**Usage:** `/scbd-dev-screenshot [<jira-key>] [<what to show>]`

## Recipe

Look for a screenshot recipe in the project's `AGENTS.md`: how to start the app, log in, and reach
the state to capture. If none exists, work it out, then offer to save it (see Learning, below).

## Capture

- Prefer deterministic fixtures and mock data over live data.
- Never capture secrets, tokens, or live personal data.
- Temporary capture scripts go in `.scratch/` and are removed once the run is done.

## Files

Crop to the changed area. Inspect every image, and recapture any that are blank, clipped, or stale.

## Output

- Files: `.scratch/screenshots/<key-or-slug>/NN-<scenario>.png`.
- A `## User-Facing Changes` snippet with one placeholder per image:
  `<!-- drop 01-<scenario>.png here: <what it shows> -->`.
- The instruction: open the PR in the browser, edit the description, and drag each file onto its
  placeholder. GitHub then hosts it under the repo's access rules.

## Ground rules

Every `scbd-dev-*` skill carries this block. It holds even when the skill is the only one loaded.

### Context resolution

Resolve the work item in this order:

1. An explicit key argument.
2. The current branch name, `feature/<KEY>-*`.
3. A key mentioned in the conversation.
4. None: ticketless work.

With no ticket:

- Skip every Jira step without comment.
- Name plan and screenshot files, and the branch, after a short slug (`feature/<slug>`).

Only `scbd-dev-start` and `scbd-dev-next` treat an epic key as a queue. Every other skill needs an
issue, or no ticket at all.

### Action policy

| Tier               | Examples                                                                                                   | Rule                                                          |
| ------------------ | ---------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| Local              | Read, edit, test, create/switch a branch                                                                   | Go ahead                                                      |
| Local history      | Commit on a feature branch                                                                                 | Only once the human has handed over autonomy, or asks         |
| Leaves the machine | Push, create/edit PR, PR comments/replies, any Jira change                                                 | List the exact actions, wait for OK, then run them and verify |
| Never              | Push to the default branch, mark a PR ready, force-push, `reset --hard`, discard or stash unexplained work | Refuse and explain                                            |

- Running a command whose whole purpose is publishing (`/scbd-dev-pr`) still shows the itemised
  actions first. One confirmation covers the whole list, and the human can edit it.
- Stage explicit paths only. Never `git add -A` or `git add .`, and never stage `.scratch/`.
- A dirty or foreign workspace means stop and ask. Never guess who owns the work.

### Default branch

Discover it with `git symbolic-ref --short refs/remotes/origin/HEAD` (strip `origin/`). If that
fails, use `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`. Skill text always says
"the default branch". It never says `main` or `master`.

### `.scratch/` layout

```text
.scratch/
  plans/<key-or-slug>.md
  screenshots/<key-or-slug>/NN-<scenario>.png
```

Before first writing to `.scratch/`, run `git check-ignore -q .scratch`. If the directory is not
ignored, say so once and point to `skills/README.md`'s global-ignore install step. Never edit `.gitignore`
without being asked. Explicit-path staging is the real safeguard; the ignore only removes noise.

### Delegation

- Default to the current conversation.
- When a plan has two or more independent steps, `scbd-dev-implement` may suggest handing them to
  sub-agents. It names the steps and why, then lets the human choose.
- A delegated sub-agent gets a self-contained brief: the step text from the plan, the files
  involved, the checks to run, and the rules "local edits only, no commits, report back".
- The brief format lives in `scbd-dev-plan`, because plan steps are written in that format.

### Learning

At the end of a skill, if the human corrected how the command ran, offer once to save the
correction:

- **Personal preference** (autonomy level, commit granularity, delegation habits, verbosity): save
  to the agent's persistent memory, if the agent has one.
- **Project fact** (component, test commands, how to run the app and log in for screenshots): offer
  a diff to the project's `AGENTS.md`, so teammates benefit too.

Before starting, read both sources for preferences that apply. Wording stays agent-neutral ("if your
agent keeps persistent memory"). Don't nag: offer once per correction, and never offer after an
uncorrected run.

### Attribution

Text posted to GitHub or Jira ends with one signature:

```text
🤖 *Posted by <agent name> on behalf of @<username>*
```

- `<agent name>` is the running agent's product name: `Claude Code`, `Codex`, and so on.
- `<username>` is the posting account on that system:
  - on GitHub, the GitHub login (`gh api user -q .login`);
  - on Jira, the Jira display name (from `scbd-jira`'s current-user lookup, or the ticket's
    assignee once `@me` is assigned).
- The canonical text lives in `scbd-github` and `scbd-jira`.

