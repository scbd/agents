# Skills catalog

## Design philosophy

Skills are shaped around tasks, not around a single coordinator. Each `scbd-dev-*` skill does one
job and works whether it's run on its own or as part of a longer session — there's no required
"start the workflow" step.

Two mental models are supported at once:

- **Command users** type `/scbd-dev-` and browse by name.
- **Natural-language users** say what they want ("let's start on DEV-123", "open a draft PR"), and
  the skill descriptions are written to match.

Every level of autonomy, from steering one small change to "work towards the goal, commit chunks,
check with me before publishing", is covered by one **action policy** (below): local work goes
ahead, anything that leaves the machine is listed and confirmed first.

## Development skills

| Skill                  | Description                                                                 |
| ----------------------- | ------------------------------------------------------------------------------ |
| `scbd-dev-start`       | Pick up a ticket, the next ready ticket in an epic, or ticketless work        |
| `scbd-dev-plan`        | Think a task through and write an implementation plan                        |
| `scbd-dev-implement`   | Build from a plan or the conversation, committing chunks when asked          |
| `scbd-dev-screenshot`  | Capture screenshots into `.scratch/`, plus PR text with placeholders         |
| `scbd-dev-pr`          | Push and create or update a draft pull request                              |
| `scbd-dev-feedback`    | Triage PR review comments, fix, and draft replies                           |
| `scbd-dev-finish`      | After merge: mark Jira `Done`, delete the local branch                       |
| `scbd-dev-next`        | Report where work stands and recommend the next command                      |

## Planning skills

Reserved namespace: `scbd-planning-*`. Nothing is published under it yet.

## Shared skills

| Skill        | Description                                                                          |
| ------------- | ---------------------------------------------------------------------------------------- |
| `scbd-jira`  | `acli` cheat sheet and team conventions for Jira: status, labels, comments, links       |
| `scbd-github` | Branch, commit, and PR conventions; fetching and replying to review threads             |

These two are unprefixed because they're shared by both development and planning work, and also run
directly for ad-hoc requests ("move DEV-12 to Peer Review", "what's the unresolved review feedback
on PR 42").

## Naming rule

- `scbd-dev-*` — development skills, one per task.
- `scbd-planning-*` — planning skills, reserved.
- `scbd-<system>` — shared, unprefixed skills for one external system (`scbd-jira`, `scbd-github`).

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

| Tier                 | Examples                                                    | Rule                                                        |
| --------------------- | -------------------------------------------------------------- | ---------------------------------------------------------------- |
| Local                 | Read, edit, test, create/switch a branch                      | Go ahead                                                        |
| Local history         | Commit on a feature branch                                    | Only once the human has handed over autonomy, or asks           |
| Leaves the machine    | Push, create/edit PR, PR comments/replies, any Jira change    | List the exact actions, wait for OK, then run them and verify   |
| Never                 | Push to the default branch, mark a PR ready, force-push, `reset --hard`, discard or stash unexplained work | Refuse and explain |

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
ignored, say so once and point to this README's global-ignore install step. Never edit `.gitignore`
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

## Invocation examples

As commands:

```bash
/scbd-dev-start DEV-123
/scbd-dev-start DEV-20            # DEV-20 is an epic: pick the next ready ticket
/scbd-dev-plan DEV-123
/scbd-dev-implement DEV-123
/scbd-dev-implement plan=.scratch/plans/DEV-123.md
/scbd-dev-screenshot DEV-123 "the new settings panel"
/scbd-dev-pr DEV-123
/scbd-dev-feedback DEV-123
/scbd-dev-finish DEV-123
/scbd-dev-next DEV-123
/scbd-jira "move DEV-12 to Peer Review"
/scbd-github "open a draft PR for this branch"
```

As natural language:

```text
let's start on DEV-123
plan a refactor of the X service
implement DEV-123
address the review feedback on this PR
what's next on DEV-20?
```

## Setting up a new project

Add to the project's `AGENTS.md`:

```text
scbd_component: Your/Component
scbd_plan_dir: docs/plans
```

`scbd_component` scopes epic-queue candidate search in `scbd-dev-start` and `scbd-dev-next`.
`scbd_plan_dir` is where a shared plan gets copied when a human asks to share it for review; it
defaults to `docs/plans`. Plans are local to `.scratch/plans/` by default and are never copied there
automatically.

If screenshots need a login or a specific way to start the app, document the recipe in `AGENTS.md`
too — `scbd-dev-screenshot` looks for it there first, and offers to save one it worked out.

### `.scratch/` note

`.scratch/` holds working files a skill produces for a human to read or reuse: plans, screenshots,
temporary capture scripts. It is not automatically git-ignored by this repository's setup — see the
top-level [README.md](../README.md) for the one-time global ignore step. Skills stage explicit paths
only, so `.scratch/` content is never committed by accident even before that step is done.

## Conventions

- **Branch naming:** `feature/<jira-key>-<short-slug>`, or `feature/<slug>` with no ticket.
- **Commits:** Conventional Commits — `feat`, `fix`, `refactor`, `test`, `docs`, `chore`.
- **PRs:** always draft, always against the default branch. Only a human marks a PR ready.
- **Review replies:** addressed comments get a reply ending in `#done`.
- **Jira sync:** ticket status (`To Do` → `In Progress` → `Peer Review` → `Done`) tracks PR state.

## Dependencies

Install these once, globally:

| Skill                 | Source                              | Required by                                                          |
| ---------------------- | -------------------------------------- | ------------------------------------------------------------------------ |
| `karpathy-guidelines`  | `multica-ai/andrej-karpathy-skills`   | `scbd-dev-plan`, `scbd-dev-implement`, `scbd-dev-feedback`             |

```bash
npx skills add multica-ai/andrej-karpathy-skills --skill karpathy-guidelines -g
```
