# Epic Iteration Reference

## Resolve Inputs

1. Require `epic`. If absent, show usage and stop without side effects.
2. Resolve `component` from arguments, then `scbd_component` in the target project's `AGENTS.md`.
   If absent, explain both configuration options and stop.
3. Use `label=ready-for-agent` unless supplied. Apply it only when admitting new `TO DO` work;
   never use it to hide already-started tickets.
4. Validate `mode` as `interactive` or `afk`.
5. Resolve the plan directory from `scbd_plan_dir`, falling back to `docs/plans`.

## Assess State

Inspect the local workspace before changing it. Read repository instructions, `git status`, current
branch, local and remote branch state, recent commits, and linked PR state. Query Jira tickets in the
epic and component, including dependencies and status. Query matching PRs by ticket reference and
`feature/<ticket-key>-*` branch.

Do not discard, overwrite, stash, or rewrite unexplained local work. Treat relevant dirty state as
the first candidate for recovery. If it cannot be confidently associated with one epic ticket, stop
for human intervention.

Recover first when relevant uncommitted work or an interrupted phase can be safely identified. Keep
the existing branch and files intact, reconstruct the interrupted action and phase from Jira, PR,
plan, and local state, then resume that action using its matrix row. Recovery is not a separate
lifecycle outcome. Describe the recovery in the technical iteration log when publishing.

Otherwise choose the first matching row in the lifecycle matrix, from top to bottom.

A dependency blocks a ticket when an `is blocked by` target is not `Done` or `Completed`. A closed,
unmerged PR, ambiguous ticket/PR mapping, unsafe workspace, or missing required access requires a
human handoff. Do not select another ticket.

## Lifecycle Matrix

| Action        | Select when                                                                                                                                                                                                                                      | Prepare                                                                                                                                                                              | Worker                                                                      | Publish                                                                                                                                                                                                                                                                                                                                       | End state                                                               |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| **Close out** | A linked PR is merged but its Jira ticket is not `Completed`.                                                                                                                                                                                    | Confirm a clean workspace and the exact merged PR/ticket pair.                                                                                                                       | None.                                                                       | Transition Jira to `Completed`, add a concise completion note linking the merged PR, and perform safe local branch cleanup.                                                                                                                                                                                                                   | Jira `Completed`; iteration ends.                                       |
| **Review**    | A linked open PR has feedback without an agent reply containing `#done`.                                                                                                                                                                         | Fetch safely, switch to the PR feature branch, and ensure local state matches its remote without overwriting local changes.                                                          | `scbd-agent-review` for one coherent review cycle, including plan feedback. | Commit accepted changes, if any, using a focused Conventional Commit; push once; publish evidence when needed; reply to every supplied comment with the reviewed response containing `#done`; append a technical iteration log to the PR.                                                                                                     | PR remains open; Jira remains `PEER REVIEW`.                            |
| **Implement** | An `IN PROGRESS` ticket has a completed plan and no unresolved plan feedback. In interactive mode, the human's instruction to proceed approves the plan; in AFK mode, the epic agent's accepted plan review is approval for the next invocation. | Fetch safely, switch to the PR feature branch, and ensure local state matches its remote without overwriting local changes.                                                          | `scbd-agent-implement` with the approved plan.                              | Remove the temporary plan; commit the accepted implementation in logical Conventional Commits; push; update the draft PR summary and testing; publish evidence; append a technical iteration log; mark the PR ready; transition Jira to `PEER REVIEW`; replace `ready-for-agent` with `ready-for-human`; add a Jira milestone linking the PR. | PR ready for review; Jira `PEER REVIEW`.                                |
| **Plan**      | A label-matching `TO DO` ticket is unblocked. Order candidates by Jira priority, then creation date.                                                                                                                                             | Transition Jira to `IN PROGRESS`, assign it to the current Jira user, create `feature/<ticket-key>-<short-slug>` from an up-to-date `main`, and choose `<plan-dir>/<ticket-key>.md`. | `scbd-agent-plan` for one decision-complete plan.                           | Commit only the plan using `docs(<ticket-key>): add implementation plan`; push; open a draft PR against `main` whose body links Jira and summarizes the plan and state; add a Jira milestone linking the PR; in AFK mode, record that the accepted epic-agent review approved the plan for a later iteration.                                 | Draft PR open; Jira remains `IN PROGRESS`; no feature code implemented. |

Keep Jira and PR state coherent at completed phase boundaries. If preparation fails after a Jira
mutation, record the failure on Jira before handing over. Never push directly to `main`. Verify every
Jira, git, and GitHub mutation after performing it.

## Interactive Boundaries

At each boundary, summarize completed observations, the next mutation, and material risks, then wait
for explicit instructions. A human may adjust scope, edit files, or stop the iteration. Reassess local
state after any human intervention before continuing.

The boundaries are assessment, workspace preparation, worker review, and external publication. A
close-out iteration has no worker, so use assessment and external-publication boundaries only.

## Dispatch Work

Give the focused sub-agent a structured prompt with:

```text
Skill: <scbd-agent-plan | scbd-agent-implement | scbd-agent-review>
Ticket: <key and summary>
Task brief: <description>
Acceptance criteria: <criteria>
Lifecycle state: <current state and selected action>
Plan: <path or inline approved plan>
Review comments: <exact text and stable identifiers, when applicable>
Constraints: <project and ticket constraints>
Expected artifacts: <paths or none>
Verification: <required checks>
Allowed operations: local filesystem edits, local test commands, and read-only Jira/git/GitHub inspection; no Jira, git, or GitHub mutations
Stop conditions: <explicit boundaries>
Required handoff: outcome, summary, plan used, files, decisions, verification, user-facing impact,
proposed external replies, blockers, next action
```

Workers resolve supplied fields into one work order and discover missing context within their
read-only constraints. When dispatching, provide a complete work order so discovery is unnecessary.

## Review Worker Output

Inspect every changed file and compare it with the work order, project conventions, and acceptance
criteria. Run appropriate final verification independently. Do not accept unrelated edits, missing
tests, unexplained deletions, red checks, or a plan that leaves implementation decisions unresolved.

If correction is needed, dispatch one fresh agent using the same focused skill with the original work
order plus precise findings. Review again. On a second failure, preserve local state, record any safe
blocker note, and stop for the human.

For implementation or review changes with user-facing impact, dispatch
`scbd-agent-pr-screenshot` with local scenarios and a temporary output directory. Review each image.
The screenshot worker only captures; this skill hosts accepted artifacts, updates PR prose, and removes
temporary artifacts after publication. If capture is impractical, publish its prose fallback.

## Durable Logs

Use Jira for lifecycle milestones, status changes, links, blockers, and significant product decisions.
Keep entries concise and avoid duplicating implementation detail.

Use the PR body for current state:

```markdown
## Summary
Closes [<ticket-key>](<jira-url>)

## Plan or Implementation
<current concise description>

## Testing
<verification>

## User-Facing Changes
<evidence or prose fallback when applicable>
```

Append one PR comment per published technical iteration containing the action, changed files, technical
decisions, verification, evidence, and next state. Review replies remain attached to their original
comments and must contain `#done`. Add the established agent attribution when posting on behalf of a
user.

## Stop Conditions

Stop safely instead of selecting other work when:

- Workspace ownership or recovery is ambiguous.
- A PR is closed without merge.
- Required credentials, transitions, branches, or external state are unavailable.
- Verification remains red or the correction retry fails.
- Review feedback or a product decision requires human judgment.
- An external mutation only partially succeeds and cannot be safely reconciled.

Record the blocker on Jira or the PR only when the correct destination is certain. Preserve local
files and branch state for recovery.

## Final Handoff

Always end with:

```text
Mode: interactive | afk
Iteration: recover | close-out | review | implement | plan
Ticket: <key and summary>
Outcome: completed | blocked | stopped
Local state: <branch and clean/dirty summary>
Changes: <commits or uncommitted files>
Verification: <checks and results>
Jira: <transitions, comments, and current state>
GitHub: <PR, replies, evidence, and current state>
Decisions: <important decisions and assumptions>
Blockers: <none or details>
Next iteration: <recommended action for the human or external process>
```

After printing the handoff, stop even if another action is immediately available.
