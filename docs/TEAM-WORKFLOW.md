# Team AI Development Workflow

A seed workflow for teams using AI coding agents (Claude Code + skills) without losing control of what goes into production. The core principle: **AI accelerates every step; a human approves every gate.**

This is intentionally minimal. Expand it as your team builds stronger documentation, a shared knowledge base, and more skills.

---

## Guiding principles

| Principle                   | Why it matters                                                                                    |
| --------------------------- | ------------------------------------------------------------------------------------------------- |
| Human In The Loop (HITL)    | AI generates code fast; humans own correctness, intent, and consequences                          |
| No execution before review  | Plans and tickets are written to disk first — nothing runs, nothing is pushed, until a human OKs  |
| Leverage community patterns | Karpathy guidelines for coding discipline; mattpocock skills for planning and PRD generation      |
| Tribe knowledge first       | If the concept is new to your tribe/team, slow down — explore, question, and document before code |

---

## The workflow

### Phase 1 — Exploration and alignment

These steps exist to make sure the team understands what they are building and why before any code is written.

#### Step 1 — Initial plan (no execution)

Use Claude Code to draft an initial plan and save it to disk. Do not implement anything yet.

```
/grill-me
```

The `/grill-me` skill turns the session into an interview: Claude asks questions to surface assumptions, missing context, and decision branches. Answer them. The goal is a shared mental model, not a polished document.

**Gate:** A human (you, or a colleague) reads the plan and confirms the direction makes sense before moving on.

---

#### Step 2 — Deepen against the domain model

```
/grill-with-docs
```

This goes further: Claude challenges your plan against the existing CONTEXT.md and ADRs in the project. It sharpens terminology, surfaces conflicts with prior decisions, and updates documentation inline as choices crystallise.

**Gate:** Any updated docs or ADR entries are reviewed by the team. New concepts that touch tribe knowledge require a wider review (see Step 3).

---

#### Step 3 — Tribe review (for major changes)

If the feature:
- introduces a concept that is new to your team or tribe
- crosses service or team boundaries
- could affect how other squads build against your system

then present the plan to the wider group before writing a single line of code. A short async write-up or a brief sync session is enough. The goal is to prevent surprises later when it is expensive to reverse course.

**Gate:** Affected team leads or architects acknowledge the approach. Record the outcome as an ADR entry if it establishes a new convention.

---

### Phase 2 — Product definition

#### Step 4 — Generate the PRD

```
/to-prd
```

Produces a human-readable Product Requirements Document with user stories. Share it with the product owner to confirm the planned behaviour matches business intent.

**Gate:** Product owner (or equivalent) signs off on the PRD before any tickets are created. This is the last cheap moment to change scope.

---

#### Step 5 — Generate issue files (to disk)

```
/to-issues
```

Dumps one Markdown file per ticket into `.scratch/<feature-slug>/`. Do not push to Jira yet.

**Gate:** A human reviews and edits the files directly. Rewrite acceptance criteria, split or merge tickets, add context. Nothing reaches the issue tracker until this review is complete.

---

#### Step 6 — Push to Jira

```
/scbd-push-to-jira <feature-slug>
```

Creates one Epic for the feature and one Story per issue file. Wires `Blocks` links automatically based on dependencies declared in the Markdown. Writes Jira keys back into every local file.

Optional arguments:

```
/scbd-push-to-jira <feature-slug> project=DEV component="Gaia/KM" label=ready-for-agent
```

**Gate:** Confirm in Jira that the Epic structure and dependency graph look correct before kicking off implementation.

---

### Phase 3 — Implementation loop

The following steps repeat, per ticket, until the Epic is done. Agent commands and human checkpoints interleave — they are not two separate sequential phases.

#### Step 7 — One ticket, start to merge

- `/scbd-agent-implement <project> <epic>` — agent picks the next unblocked ticket, implements it, and opens a **draft** PR. Jira ticket moves to `IN PROGRESS → PEER REVIEW`.
- **Human reviews the draft diff** — you are the last line of defence before the code is visible to the team. Leave comments directly on the draft PR.
- `/scbd-agent-review <project> <epic>` — agent reads all open comments, addresses each one (implements, explains, or marks resolved with `#done`), and frees any tickets that were blocked by this one.
- The two steps above (human comments → agent review) repeat until the diff is acceptable.
- **Human marks the PR ready** and assigns a peer reviewer.
- Peer reviewer leaves comments → run `/scbd-agent-review` again, or resolve manually. Resolved comments get `#done`.
- **Human merges** once all comments are resolved and CI is green.

**Gate:** No PR merges without a human reviewer approving it.

---

### Loop exit condition

Step 7 repeats for each ticket in the Epic. The Epic is complete when:

- every Story ticket is in `Completed` status
- every PR is merged
- no open draft PRs remain

---

## Quick reference

```
# Phase 1 — Explore
/grill-me                          Draft + interview session
/grill-with-docs                   Deepen against existing docs
                                   (tribe review if needed)

# Phase 2 — Plan
/to-prd                            Generate PRD → product owner review
/to-issues                         Dump tickets to disk → human edit
/scbd-push-to-jira <slug>          Create Epic + Stories in Jira

# Phase 3 — Build (repeat per ticket until Epic done)
/scbd-agent-implement <proj> <epic>   Pick next unblocked ticket → implement → draft PR
                                      ↳ human reviews draft, leaves comments
/scbd-agent-review <proj> <epic>      Address comments → free blocked tickets
                                      ↳ repeat until diff is acceptable
                                      ↳ human marks ready → peer review → merge
```

---

## Skill dependencies

Install these globally on every developer machine:

```bash
# This repo's skills
npx skills add scbd/agents -g

# Karpathy guidelines (required by scbd-agent-* skills)
npx skills add multica-ai/andrej-karpathy-skills --skill karpathy-guidelines -g

# mattpocock planning skills (required for /grill-me, /grill-with-docs, /to-prd, /to-issues)
npx skills add mattpocock/skills -g
```

Then run once per project to initialise local files:

```
/setup-matt-pocock-skills
```

Keep skills up to date:

```bash
npx skills update -g
```
