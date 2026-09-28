---
name: scbd-dev-plan
description: Think a task through and write an implementation plan, for a Jira ticket or ad-hoc work, before touching code. Use for "plan DEV-123" or "plan a refactor of the X service". Optionally shares the plan for review.
---

# scbd-dev-plan

Think a task through and write a plan. No code changes, and no external changes unless the human
picks a sharing option at the end.

**Usage:** `/scbd-dev-plan [<jira-key>]`

## Before you start

Load `scbd-dev-agent` for the shared ground rules. Regardless:

- Nothing leaves the machine (push, PR, comments, Jira) without listing the actions and getting OK.
- Never push to the default branch, mark a PR ready, force-push, or discard unexplained work.
- Stage explicit paths only; never stage `.scratch/`.

## Steps

1. Read the ticket (if any), the project's `AGENTS.md`, and the relevant code and tests.
2. Settle what the code and ticket can already answer.
3. List material open questions in chat rather than guessing at them.
4. Write `.scratch/plans/<key-or-slug>.md`. Size it to the task — a small fix gets a short plan.

## Plan shape

- **Intent** — what this accomplishes and why.
- **Approach** — the chosen design, briefly.
- **Steps** — each one self-contained: the files it touches, the change, the checks to run, and
  what "done" looks like. This doubles as `scbd-dev-agent`'s delegation brief.
- **Risks** — what could go wrong or needs care.
- **Open questions** — anything left for the human.

## At the end

Offer three options:

1. **Keep it local** (the default) — nothing more happens.
2. **Share for review** — copy the plan to `scbd_plan_dir`, commit it as
   `docs(<KEY>): add implementation plan`, then hand off to `/scbd-dev-pr`.
3. **Post a summary as a Jira comment** (`scbd-jira`).
