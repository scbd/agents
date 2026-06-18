---
name: scbd-agent-implement
description: Pick and implement the next unblocked Jira ticket end-to-end, including optional interactive phased mode with sub-agents for plan PR, feedback, and implementation. Transitions the ticket through IN PROGRESS → PEER REVIEW, opens a draft PR, uses scbd-agent-pr-screenshot for user-facing evidence, and follows karpathy-guidelines throughout.
---

# scbd-agent-implement

Implement the next ready Jira ticket end-to-end.

**Usage:** `/scbd-agent-implement <epic> [component] [label]`

## Step 0 — Parse Arguments

Arguments received: `$ARGUMENTS`

Parse `$ARGUMENTS` accepting either positional order or `key=value` pairs:

| Parameter   | Position | Key          | Default                                     | Required |
|-------------|----------|--------------|---------------------------------------------|----------|
| `epic`      | 1st      | `epic=`      | —                                           | **yes**  |
| `component` | 2nd      | `component=` | `scbd_component:` value from `AGENTS.md`    | **yes**  |
| `label`     | 3rd      | `label=`     | `ready-for-agent`                           | no       |

**If `epic` is missing**, stop immediately and print the following — do nothing else:

```
Missing required parameter: epic

Usage: /scbd-agent-implement <epic> [component] [label]

  epic        Jira epic key, e.g. DEV-20                          (required)
  component   Jira component filter, e.g. "Gaia/KM"               (required)
              Can be set as a project-level default in AGENTS.md:
                scbd_component: Gaia/KM
  label       Jira label filter                                    (optional)
              Default: ready-for-agent

Examples:
  /scbd-agent-implement DEV-20
  /scbd-agent-implement DEV-20 Gaia/KM
  /scbd-agent-implement DEV-20 Gaia/KM my-label
  /scbd-agent-implement epic=DEV-20 component=Gaia/KM label=ready-for-agent
```

**If `component` is not provided in arguments**, read the current project's `AGENTS.md`
and look for a line of the form `scbd_component: <value>`. Use that value if found.
If still not found, stop immediately and print:

```
Missing required parameter: component

Pass it as an argument or add a default to your project's AGENTS.md:
  scbd_component: Gaia/KM
```

Resolve and display the final values before proceeding:

```
EPIC:      <value>
COMPONENT: <value or "(none)">
LABEL:     <value>
```

---

## Phase 1 — Ticket Selection

1. Query Jira for all issues under epic **EPIC** with label **LABEL**
   (and component **COMPONENT** if set).
2. From those, keep only tickets where:
   - Status is `TO DO`
   - No `is blocked by` dependency pointing to a ticket whose status is not `Done` / `Completed`
3. Among the remaining candidates, select the **highest-priority unblocked ticket**
   (use Jira priority field, then creation date as tiebreaker).
4. Output the selected ticket key + summary for confirmation, then proceed.

If no eligible ticket exists, stop and report:

```
No eligible ticket found.
  Epic:      <EPIC>
  Component: <COMPONENT or "(none)">
  Label:     <LABEL>
```

---

## Phase 2 — Claim the Ticket

5. Transition the selected ticket status to **`IN PROGRESS`** via the Jira API.
6. Assign the ticket to yourself if not already assigned.

---

## Phase 3 — Implementation

7. Follow the `/karpathy-guidelines` skill for the entire implementation.
8. Work in a feature branch named `feature/<ticket-key>-<short-slug>`.
9. Commit in logical, atomic steps — each commit message must follow Conventional Commits:
```
   <type>(<scope>): <short description>

   [optional body referencing ticket key]
```
   Preferred types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`.

10. Before considering implementation done, run the full test suite and confirm all tests pass.
    - If tests fail, fix them before proceeding — do not move to Phase 4 with a red test suite.
    - If the feature requires new tests, write them as part of the implementation (not as an afterthought).

### Interactive Phase 3 Option

When the user asks to work interactively, phase implementation, or use sub-agents for each part, split Phase 3 and stop for instructions after each part:

- **Phase 3.1 — Branch + Plan PR**: create/switch to the feature branch, inspect enough code to write a concrete implementation plan, save the plan in the project, commit it, push, and open/update a draft PR that links the plan. Add a comment to the Jira ticket that includes a link to the PR. Do not implement feature code.
- **Phase 3.2 — Plan Feedback**: address user or reviewer feedback on the plan. Update the plan/PR, commit and push if files changed. Do not start implementation until the user approves moving on.
- **Phase 3.3 — Implementation**: implement the approved plan, delete the plan, commit logical changes, run verification, update the draft PR body and screenshots, then report back. Do not transition Jira or mark the PR ready until the user confirms.

For each interactive part:

- Spawn a fresh sub-agent only when the user explicitly asks for sub-agents. Give it one bounded part, explicit stop conditions, and the expected final report fields.
- Keep the main context as coordinator: verify branch state, review the sub-agent handoff, summarize results to the user, and wait for the next instruction.
- Do not let a sub-agent proceed into the next sub-phase without explicit user approval.

---

## Phase 4 — Draft PR

11. Once implementation is complete, push the branch and open a **draft PR** using the `gh` CLI:
```bash
    gh pr create \
      --draft \
      --title "<ticket-key>: <ticket summary>" \
      --body "$(cat <<'EOF'
    ## Summary
    Closes (<ticket-key>)[<url-to-the-ticket>]

    <!-- describe what was done -->

    ## Testing
    <!-- describe how to test -->

    ---
    🤖 *Posted by AFK Agent on behalf of @<GitHub username>*
    EOF
    )" \
      --base main
```
12. Transition the Jira ticket status to **`PEER REVIEW`** and add a comment linking to the PR
13. If there were user-facing impacts, add a PR section named `## User-Facing Changes`.
    - Use the `/scbd-agent-pr-screenshot` skill to capture, crop, verify, host, and link screenshots when the project supports them.
    - If the project does not have a practical screenshot path, use `/scbd-agent-pr-screenshot` for the prose fallback and testing note.
14. Remove the label `ready-for-agent` and add label `ready-for-human`

---

## Constraints & Reminders

- Never start implementation without first confirming the selected ticket in Phase 1.
- Never push directly to `main`.
- One ticket per agent run.
- Keep Jira ticket transitions and PR state in sync at every phase boundary.
