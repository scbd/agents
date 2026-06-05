---
name: scbd-agent-implement
description: Pick and implement the next unblocked Jira ticket end-to-end. Transitions the ticket through IN PROGRESS → PEER REVIEW, opens a draft PR, and follows karpathy-guidelines throughout.
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
12. Transition the Jira ticket status to **`PEER REVIEW`**
13. Remove the label `ready-for-agent` and add label `ready-for-human`

---

## Constraints & Reminders

- Never start implementation without first confirming the selected ticket in Phase 1.
- Never push directly to `main`.
- One ticket per agent run.
- Keep Jira ticket transitions and PR state in sync at every phase boundary.
