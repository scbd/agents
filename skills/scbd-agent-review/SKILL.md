---
name: scbd-agent-review
description: Resolves one coherent review cycle for a Jira ticket using read-only Jira, git, and GitHub context, returning uncommitted changes and proposed replies. Use when scbd-agent-epic delegates PR feedback or a human directly requests review work for one ticket.
---

# scbd-agent-review

Address one ticket's review feedback and leave any changes uncommitted for the caller.

**Usage:** `/scbd-agent-review ticket=<key>`

## Inputs

Resolve one work order. Require a ticket key from `ticket=<key>` or a supplied work order, and treat
supplied ticket context, task brief, plan or current approach, review comments, acceptance criteria,
constraints, and stop conditions as authoritative. Fill missing facts from the Jira ticket, local
project, and read-only git and GitHub context. When review comments are not supplied, identify the
linked PR and collect its unresolved comments. Ask the human only when the ticket-to-PR mapping is
ambiguous or essential context cannot be discovered.

## Workflow

1. Read the supplied or discovered comments and inspect the ticket, PR, diff, and relevant local files.
   If the current workspace is not safely associated with the ticket, stop rather than switching
   branches or changing git state.
2. Classify each comment as a code change, sufficient clarification, or inappropriate/infeasible
   request. Do not implement a change merely because a reviewer asked a question.
3. For appropriate changes, follow `/karpathy-guidelines`, edit the local files, and add or update
   tests. For plan feedback, update the plan rather than implementing feature code.
4. Run relevant local verification for all code changes.
5. Draft one concise response for every supplied comment:
   - `Implemented - <summary>. #done`
   - `Response - <answer>. #done`
   - `Not implemented - <reason>. #done`
6. Stop without committing, publishing replies, or proceeding to another phase.

## Boundaries

- Jira, git, and GitHub access is read-only. Do not transition, assign, label, or comment on Jira;
  mutate git state or the worktree through git; or create, edit, review, comment on, or merge a PR.
- Do not fetch, pull, switch branches, stage, commit, push, stash, reset, or clean.
- Do not capture or prepare PR evidence. Report user-facing impact to the caller.
- Handle exactly one coherent review cycle per invocation.

## Handoff

```text
Outcome: completed | blocked
Summary: <what changed or was concluded>
Files: <created, changed, and deleted paths>
Decisions: <per-comment classification and assumptions>
Verification: <commands and results>
User-facing impact: <none or a concise description>
Proposed external replies: <comment identifier and #done response>
Blockers: <none or details>
Next action: <recommended caller action>
```
