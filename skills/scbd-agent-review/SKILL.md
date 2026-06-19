---
name: scbd-agent-review
description: Resolves one coherent set of supplied review comments using only the local filesystem, returning code changes or proposed replies without using git or external services. Use when scbd-agent-epic delegates PR feedback or when a human wants review feedback handled locally.
---

# scbd-agent-review

Address supplied review feedback and leave any changes uncommitted for the caller.

**Usage:** `/scbd-agent-review`

## Inputs

When delegated, require a work order containing the ticket, task brief, relevant plan or current
approach, exact review comments, acceptance criteria, constraints, and stop conditions.

When invoked directly, derive those fields from the user's prompt and local files. Ask the human
for missing review text or other essential context. Do not fetch it from Jira, GitHub, or git.

## Workflow

1. Read the supplied comments and inspect the relevant local files.
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

- Do not run any git command, including read-only commands.
- Do not access Jira or GitHub and do not use their CLIs or APIs.
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
