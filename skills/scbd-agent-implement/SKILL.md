---
name: scbd-agent-implement
description: Implements one approved ticket plan entirely in the local filesystem without using git or external services. Use when scbd-agent-epic delegates an implementation or when a human wants local implementation work with an uncommitted handoff.
---

# scbd-agent-implement

Implement one approved plan and leave the result uncommitted for the caller.

**Usage:** `/scbd-agent-implement [plan=<path>]`

## Inputs

When delegated, require a work order containing the ticket, task brief, acceptance criteria,
approved plan, constraints, expected verification, and stop conditions. The plan may be inline
or supplied with `plan=<path>`.

When invoked directly, derive those fields from the user's prompt and local files. Ask the human
for essential missing context. Do not fetch it from Jira, GitHub, or git.

## Workflow

1. Read the approved plan and relevant project instructions.
2. Inspect only the local filesystem and clarify any material conflict before editing.
3. Follow `/karpathy-guidelines` while making the smallest complete implementation.
4. Add or update tests required by the change.
5. Run the most relevant verification available locally. Fix failures caused by the work; report
   unrelated or unresolved failures clearly.
6. Remove the temporary plan file after the implementation and verification are complete.
7. Stop without preparing screenshots, staging files, committing, or handing work to another phase.

## Boundaries

- Do not run any git command, including read-only commands.
- Do not access Jira or GitHub and do not use their CLIs or APIs.
- Do not create commits, switch branches, push, or edit PRs.
- Do not capture, host, or prepare PR evidence. Report user-facing impact so the caller can decide
  whether to invoke `scbd-agent-pr-screenshot`.
- Work on exactly one implementation per invocation.

## Handoff

Return these fields to the epic agent or human:

```text
Outcome: completed | blocked
Summary: <what changed>
Files: <created, changed, and deleted paths>
Decisions: <technical decisions and assumptions>
Verification: <commands and results>
User-facing impact: <none or a concise description>
Proposed external replies: <usually none>
Blockers: <none or details>
Next action: <recommended caller action>
```
