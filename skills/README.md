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
check with me before publishing", is covered by **checkpoints** (below): local work goes ahead, and
each kind of commit, push, PR, reply or Jira change asks first by default. Each person can loosen or
tighten any checkpoint in their own preferences file.

## Development skills

| Skill                 | Description                                                            |
| --------------------- | ---------------------------------------------------------------------- |
| `scbd-dev-agent`      | Shared ground rules, and the entry point for open-ended dev work       |
| `scbd-dev-start`      | Pick up a ticket, the next ready ticket in an epic, or ticketless work |
| `scbd-dev-plan`       | Think a task through and write an implementation plan                  |
| `scbd-dev-implement`  | Build from a plan or the conversation, committing chunks when asked    |
| `scbd-dev-screenshot` | Capture screenshots into `.scratch/`, plus PR text with placeholders   |
| `scbd-dev-pr`         | Push and create or update a draft pull request                         |
| `scbd-dev-feedback`   | Triage PR review comments, fix, and draft replies                      |
| `scbd-dev-finish`     | After merge: mark Jira `Done`, delete the local branch                 |
| `scbd-dev-next`       | Report where work stands and recommend the next command                |

`scbd-dev-agent` does double duty: every other `scbd-dev-*` skill loads it for its shared ground
rules (context resolution, the action policy, `.scratch/`, delegation, learning), and it also runs
directly when a request doesn't match one specific command yet — open-ended or exploratory work, a
mix of steps, "let's work on DEV-123 together". It hands off to the matching command once the shape
of the work becomes clear.

## Planning skills

Reserved namespace: `scbd-planning-*`. Nothing is published under it yet.

## Shared skills

| Skill         | Description                                                                       |
| ------------- | --------------------------------------------------------------------------------- |
| `scbd-jira`   | `acli` cheat sheet and team conventions for Jira: status, labels, comments, links |
| `scbd-github` | Branch, commit, and PR conventions; fetching and replying to review threads       |

These two are unprefixed because they're shared by both development and planning work, and also run
directly for ad-hoc requests ("move DEV-12 to Peer Review", "what's the unresolved review feedback
on PR 42").

## Naming rule

- `scbd-dev-*` — development skills, one per task, plus `scbd-dev-agent` for shared rules and
  open-ended work.
- `scbd-planning-*` — planning skills, reserved.
- `scbd-<system>` — shared, unprefixed skills for one external system (`scbd-jira`, `scbd-github`).

## Ground rules

The shared rules every `scbd-dev-*` command follows — context resolution, the action policy,
`.scratch/`, delegation, learning — live in `scbd-dev-agent`, not duplicated here. Checkpoints,
attribution and default-branch discovery live in `scbd-github` and `scbd-jira`, so planning skills
get them too.

## Checkpoints and preferences

Every change to git history, GitHub or Jira passes a checkpoint:

| Key            | Covers                                            | Default   |
| -------------- | ------------------------------------------------- | --------- |
| `git.commit`   | Local commits on a feature branch                 | `invoked` |
| `github.push`  | Pushing a feature branch                          | `ask`     |
| `github.pr`    | Creating or editing a draft PR, and its push      | `ask`     |
| `github.reply` | PR comments and review replies                    | `ask`     |
| `jira`         | Transitions, assignments, labels, comments, links | `ask`     |

| Mode      | Behaviour                                                                          |
| --------- | ---------------------------------------------------------------------------------- |
| `ask`     | List the exact actions and wait for OK                                             |
| `invoked` | Go ahead when the human asked for the action directly (command or words); else ask |
| `auto`    | Go ahead, even when the agent decides on the action itself                         |

`git`, `github` or `jira` alone sets every checkpoint of that system; a full key overrides it. In
every mode the agent verifies the result and reports what ran. The hard limits never change: no push
to the default branch, no marking a PR ready, no force-push, no discarding unexplained work.

Set personal modes in `~/.config/scbd-agents/preferences.md`. The file works with any agent, and
free-text preferences are fine too:

```markdown
# SCBD agent preferences

- github.pr: invoked
- git.commit: auto
- Run the full test suite before any push.
```

You don't have to write it by hand: when you tell an agent "don't ask me again for this", it offers
the matching line.

How the mode is resolved:

1. An instruction in the conversation wins for that session, including a `yes` argument
   (`/scbd-dev-pr yes`).
2. Otherwise the preferences file, else the default.
3. A project's `AGENTS.md` can set a floor with `scbd_checkpoints:`, e.g. `github.push: ask` in a
   repo where every push must be confirmed. The stricter of the two applies.

Checkpoints control the skill's own questions. Your agent's permission system is a separate layer:
Claude Code, for example, may still prompt before `git push` or `gh pr create` unless its
`settings.json` allows those commands.

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
/scbd-dev-agent "let's work through DEV-123 together"
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
let's work on DEV-123 together
help me with this refactor, no ticket
```

## Setting up a new project

Add to the project's `AGENTS.md`:

```text
scbd_component: Your/Component
scbd_plan_dir: docs/plans
```

Optionally, require confirmation for some checkpoints in this project, whatever personal
preferences say:

```text
scbd_checkpoints:
  github.push: ask
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
- **Review replies:** addressed comments get a reply containing `#done`.
- **Jira sync:** ticket status (`To Do` → `In Progress` → `Peer Review` → `Done`) tracks PR state.

## Dependencies

Install these once, globally:

| Skill                 | Source                              | Required by                                                |
| --------------------- | ----------------------------------- | ---------------------------------------------------------- |
| `karpathy-guidelines` | `multica-ai/andrej-karpathy-skills` | `scbd-dev-plan`, `scbd-dev-implement`, `scbd-dev-feedback` |

```bash
npx skills add multica-ai/andrej-karpathy-skills --skill karpathy-guidelines -g
```
