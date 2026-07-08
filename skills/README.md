# Skills catalog

## Design philosophy — externally driven iterations

`scbd-agent-workflow` performs one Jira-backed workflow unit and exits. A human or scheduler owns
repetition.

Both modes stop after one iteration. Interactive pauses at phase boundaries; AFK does not.

Contract:

- Durable state lives in Jira, git history, PRs, and committed plan artifacts
- One plan, implementation, review, recovery, or close-out per run
- A handoff ends every run
- Workflow bookkeeping is coordinated by `scbd-agent-workflow`; companion skills may read or mutate
  Jira, git, and GitHub when explicitly invoked for their responsibility

## Available skills

| Skill                      | Description                                                    | Dependencies                                                                                          |
| -------------------------- | -------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `scbd-agent-workflow`      | Orchestrate one Jira-backed workflow and all external state    | `karpathy-guidelines`, `scbd-agent-jira`, `scbd-agent-github`, `scbd-agent-screenshot`                |
| `scbd-agent-jira`          | Mutate and validate Jira state for SCBD agent workflows        | —                                                                                                     |
| `scbd-agent-github`        | Mutate and validate git/GitHub state for SCBD agent workflows  | —                                                                                                     |
| `scbd-agent-plan`          | Plan one Jira ticket with read-only external context           | —                                                                                                     |
| `scbd-agent-implement`     | Implement one Jira ticket locally, with or without a plan      | `karpathy-guidelines`                                                                                 |
| `scbd-agent-review`        | Address one Jira ticket's review cycle locally                 | `karpathy-guidelines`                                                                                 |
| `scbd-agent-screenshot`    | Capture and verify local screenshot evidence                   | —                                                                                                     |

The workflow skill dispatches and reviews planning, implementation, and review agents, applies the
screenshot skill locally, then hosts accepted evidence and handles external state itself. Focused
skills also run directly and return uncommitted work.

## Workflow

Invoke `scbd-agent-workflow` with a Jira key. If the key is an epic, the agent should pick the next
actionable ticket from the queue, run one planning, implementation, review, recovery, or close-out
action, publish state, and stop. If the key is a specific Jira issue, the agent should resume that
issue's lifecycle, perform the next matching action, and stop without scanning for other work.

Use `mode=interactive` or `mode: interactive` when a human should confirm phase boundaries and maybe
provide instructions. Use `mode=afk` or `mode: afk` for unattended routine iterations; it still stops
for blockers, unsafe state, red verification, or ambiguity.

Use focused skills directly when the target is already known: plan or implement one Jira ticket,
address one PR review cycle, capture screenshot evidence, or perform an explicit Jira/GitHub action.
Focused skills do one job and return a handoff.

## Invoking skills

```bash
/scbd-agent-workflow DEV-20 mode=interactive # component from AGENTS.md
/scbd-agent-workflow DEV-20 component=Gaia/KM mode: afk
/scbd-agent-workflow DEV-123 mode=interactive
/scbd-agent-jira ticket=DEV-123 action=assess
/scbd-agent-github ticket=DEV-123 action=update-pr

/scbd-agent-plan ticket=DEV-123
/scbd-agent-implement ticket=DEV-123 # discover plan, else implement directly
/scbd-agent-implement ticket=DEV-123 plan=docs/plans/DEV-123.md
/scbd-agent-review ticket=DEV-123
/scbd-agent-screenshot output=/tmp/DEV-123-evidence
```

## Setting up a new project

Add to the project's `AGENTS.md`:

```
scbd_component: Your/Component
scbd_plan_dir: docs/plans
```

`scbd_plan_dir` is optional and defaults to `docs/plans`.

## Workflow conventions

- **Branch naming:** `feature/<ticket-key>-<short-slug>`
- **Commits:** Conventional Commits — `feat`, `fix`, `refactor`, `test`, `docs`, `chore`
- **Review:** Addressed comments get `#done` replies
- **Iteration:** One workflow invocation; callers own repetition
- **Modes:** Interactive pauses at boundaries; AFK does not
- **Ownership:** `scbd-agent-workflow` coordinates state; `scbd-agent-jira` and
  `scbd-agent-github` handle their external systems when explicitly invoked; screenshot captures
  local evidence without external access
- **PR state:** Agents keep new PRs draft and never push to `main`; only humans mark PRs ready
- **Jira sync:** Ticket status (`IN PROGRESS` → `PEER REVIEW` → `Completed`) stays synchronized with PR state

## External dependencies

Install these once, globally:

| Skill                 | Source                              | Required by                                                         |
| --------------------- | ----------------------------------- | ------------------------------------------------------------------- |
| `karpathy-guidelines` | `multica-ai/andrej-karpathy-skills` | `scbd-agent-workflow`, `scbd-agent-implement`, `scbd-agent-review`  |

```bash
npx skills add multica-ai/andrej-karpathy-skills --skill karpathy-guidelines -g
```

## Jira credentials

The Jira skills talk to `https://scbd.atlassian.net` over REST. Credentials live in `curl`'s
`~/.netrc` so the token never enters a command line, the agent context, or a process listing — the
skill authenticates with `curl --netrc` and never handles the token directly.

Set this up once. The `password` is an Atlassian API token from
https://id.atlassian.com/manage-profile/security/api-tokens, not an account password:

```bash
printf 'machine scbd.atlassian.net login %s password %s\n' \
  you@example.com YOUR_API_TOKEN >> ~/.netrc
chmod 600 ~/.netrc
```

On a `401`/`403`, the Jira skill stops and asks the human to check or install this entry.
