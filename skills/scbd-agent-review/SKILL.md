---
name: scbd-agent-review
description: Resolves one ticket review cycle, returning uncommitted changes and proposed replies without external mutations. Use for delegated or direct PR feedback work.
---

# scbd-agent-review

Address one ticket's feedback and leave changes uncommitted.

**Usage:** `/scbd-agent-review ticket=<key>`

## Inputs

Require `ticket=<key>` or a work order. Supplied context, plan, comments, criteria, constraints, and
stop conditions are authoritative. Resolve gaps from Jira, local files, read-only git, and GitHub.
If comments are absent, find the linked PR's unresolved comments. Ask only about ambiguous mapping or
undiscoverable essentials.

## Workflow

1. Inspect comments, ticket, PR, diff, and relevant files. Stop if the workspace is not safely tied
   to the ticket.
2. Classify each comment as a code change, sufficient clarification, or inappropriate/infeasible
   request. Do not implement a change merely because a reviewer asked a question.
3. For appropriate changes, follow `/karpathy-guidelines`, edit files and tests. For plan feedback,
   update the plan only.
4. Run relevant local verification for all code changes.
5. Draft one concise response for every supplied comment:
   - `Implemented - <summary>. #done`
   - `Response - <answer>. #done`
   - `Not implemented - <reason>. #done`
6. Stop without committing, publishing replies, or proceeding to another phase.

## Boundaries

- Jira, git, and GitHub are read-only. Do not fetch, pull, switch, stage, commit, push, stash, reset,
  clean, mutate Jira, or mutate PRs.
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
