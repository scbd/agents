# Epic Iteration Reference

## Inputs

1. Require `epic`; otherwise show usage and stop unchanged.
2. Resolve `component` from arguments, then project `AGENTS.md`'s `scbd_component`; otherwise explain
   both options and stop.
3. Default `label` to `ready-for-agent`. Filter only new `TO DO` intake, never started work.
4. Require `mode=interactive|afk`.
5. Resolve `scbd_plan_dir`; default to `docs/plans`.

## Select

Before mutations, read project instructions, workspace and branch state, recent commits, remotes,
Jira epic/component tickets and dependencies, and linked PRs. Load `scbd-agent-jira` for Jira state
rules and `scbd-agent-github` for branch/PR matching and publication rules.

Never discard, overwrite, stash, or rewrite unexplained work. Recover relevant dirty or interrupted
work first: preserve branch/files, infer its ticket/action from Jira, PR, plan, and local state, then
resume that matrix row. Log the recovery. If ownership is ambiguous, stop for a human.

Otherwise select the first matching action below. A closed-unmerged PR, ambiguous mapping, unsafe
workspace, blocked ticket, or missing access requires handoff; do not choose another ticket.

## Lifecycle (first match wins)

1. **Close out:** Merged PR with Jira not `Completed`. Verify a clean workspace and exact pair.
   Transition Jira to `Completed`, note the PR, and safely clean the local branch. End: Jira
   `Completed`.
2. **Review:** Open PR feedback lacks an agent `#done` reply. Safely fetch/switch to its feature
   branch and match the remote without overwriting work. Run `scbd-agent-review` for one cycle.
   Commit accepted changes if any, push once, publish needed evidence, reply to every comment with
   `#done`, and log. End: PR open; Jira `PEER REVIEW`.
3. **Implement:** Jira `IN PROGRESS`; plan completed; no unresolved plan feedback; human approval in
   interactive mode or accepted epic review in AFK. Safely fetch/switch to the feature branch and
   match its remote without overwriting work. Run `scbd-agent-implement` with the plan. Remove the
   plan; create logical Conventional Commits; push; update draft PR summary/testing; publish evidence
   and log; keep draft state; transition Jira to `PEER REVIEW`; replace `ready-for-agent` with
   `ready-for-human`; link the PR milestone. End: draft unchanged; Jira `PEER REVIEW`.
4. **Plan:** Unblocked, label-matching `TO DO`; order by priority then creation. Transition to
   `IN PROGRESS`; assign the current Jira user; create `feature/<ticket-key>-<short-slug>` from current
   `main`; choose `<plan-dir>/<ticket-key>.md`; run `scbd-agent-plan`. Commit only the plan as
   `docs(<ticket-key>): add implementation plan`; push; open a draft PR to `main` linking Jira and
   summarizing plan/state; link the PR milestone. In AFK, record approval for later implementation.
   End: draft PR; Jira `IN PROGRESS`; no feature code.

Keep Jira and PR coherent at phase boundaries. Use `scbd-agent-jira` and `scbd-agent-github` for
external mutations, logs, comments, labels, PR bodies, review replies, branch cleanup, and evidence
hosting. Verify every external mutation.

## Interactive Mode

At assessment, preparation, worker review, and publication boundaries, summarize observations, next
mutation, and risks; await instructions. Close-out omits worker boundaries. After human intervention,
reassess local state.

## Work Order

Dispatch a focused agent with:

```text
Skill: <scbd-agent-plan | scbd-agent-implement | scbd-agent-review>
Ticket: <key and summary>
Task brief: <description>
Acceptance criteria: <criteria>
Lifecycle state: <state and action>
Plan: <path or approved inline plan>
Review comments: <exact text and stable IDs, if applicable>
Constraints: <project and ticket constraints>
Expected artifacts: <paths or none>
Verification: <checks>
Allowed operations: local edits/tests and read-only Jira/git/GitHub; no external mutations
Stop conditions: <boundaries>
Required handoff: outcome, summary, plan used, files, decisions, verification, user-facing impact,
proposed replies, blockers, next action
```

Supply a complete order; workers may discover gaps only within read-only boundaries.

## Review Output

Inspect every changed file against the order, conventions, and criteria; independently verify. Reject
unrelated edits, missing tests, unexplained deletions, red checks, or undecided plans. If needed,
dispatch one fresh correction agent with the original order and precise findings. After a second
failure, preserve state, log any safe blocker, and stop.

For user-facing implementation/review changes, apply `scbd-agent-screenshot` in this context;
never dispatch a screenshot agent. Give it deterministic local scenarios and a temporary output
directory. Review its handoff and inspect every image before publication. If capture is impractical,
publish its prose fallback.

The epic agent owns publication. Use `scbd-agent-github` for PR evidence hosting and publication
format.

## Logs

Use `scbd-agent-jira` for milestone, state, link, blocker, and decision logs. Use
`scbd-agent-github` for PR body updates, PR comments, review replies, and hosted evidence.

## Stop

Stop without selecting other work for ambiguous ownership/recovery, closed-unmerged PRs, unavailable
credentials/transitions/branches/state, red verification, failed correction, human judgment, or an
irreconcilable partial mutation. Log only to a certain destination. Preserve files and branch.

Always finish with:

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
Next iteration: <human or external-process recommendation>
```

Then stop, even if more work is actionable.
