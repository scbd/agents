---
name: scbd-agent-review
description: Address peer-review comments for the next Jira ticket in PEER REVIEW status, including optional interactive phased mode with sub-agents for review triage, feedback handling, and verification. Uses scbd-agent-pr-screenshot for screenshot or visual-proof updates when review feedback affects UI, closes out merged PRs, and handles change-request cycles following karpathy-guidelines.
---

# scbd-agent-review

Address peer review comments for the next Jira ticket in review.

**Usage:** `/scbd-agent-review <epic> [component] [label]`

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

Usage: /scbd-agent-review <epic> [component] [label]

  epic        Jira epic key, e.g. DEV-20                          (required)
  component   Jira component filter, e.g. "Gaia/KM"               (required)
              Can be set as a project-level default in AGENTS.md:
                scbd_component: Gaia/KM
  label       Jira label filter                                    (optional)
              Default: ready-for-agent

Examples:
  /scbd-agent-review DEV-20
  /scbd-agent-review DEV-20 Gaia/KM
  /scbd-agent-review DEV-20 Gaia/KM my-label
  /scbd-agent-review epic=DEV-20 component=Gaia/KM label=ready-for-agent
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
   (and component **COMPONENT** if set) where status is `PEER REVIEW`.
2. For each ticket, locate its corresponding PR (via branch name `feature/<ticket-key>-*`
   or PR description referencing the ticket key).
3. Select a ticket using this priority order:
   - **First:** the ticket whose PR is **merged** (needs closing out).
   - **Second:** the ticket whose PR has **unaddressed `changes requested`** comments
     (i.e. at least one review comment with no `#done` marker in a reply from the agent).
   - If no ticket matches either condition, stop and report:

```
No actionable PEER REVIEW ticket found.
  Epic:      <EPIC>
  Component: <COMPONENT or "(none)">
  Label:     <LABEL>
```

4. Output the selected ticket key + PR link for confirmation, then proceed.

---

## Phase 2 — Handle Merged PR

*Execute this phase only if the selected ticket's PR is merged.*

5. Transition the Jira ticket status to **`Completed`**, remove the label `ready-for-agent` and add label `ready-for-human`
6. Stop — run is done.

---

## Phase 3 — Handle Closed (Not Merged) PR

*Execute this phase only if the selected ticket's PR is closed without being merged.*

7. Alert the user:
   > ⚠️ Ticket `<ticket-key>` — PR was closed without being merged. Manual intervention required.
8. Stop — do not change ticket status.

---

## Phase 4 — Address Review Comments

*Execute this phase only if the selected ticket's PR has unaddressed review comments.*

9. Load all review comments on the PR that do not yet have a reply containing `#done`.
10. For each unaddressed comment, analyse it:
    - If the comment is **asking about the code, requiring an explanation, wanting clarification or challenging the approach:** then determine if an answer can be provided. DON'T BE TOO EAGER TO IMPLEMENT A CHANGE!
    - **If the change is feasible and relevant:** implement it following the `/karpathy-guidelines` skill. Run the full test suite — do not proceed if tests fail. Commit with:
```
      fix: address review comment — <short description>
```
      Then reply on the PR comment:
      
    - If the comment was a **question/confirmation/clarification and an answer is sufficient** reply on the PR comment:
      > ✔️ Response — <one sentence response>. #done
      >
      > ---
      > 🤖 *Posted by AFK Agent on behalf of @<GitHub username>*

    - If the **change is not feasible or not relevant:** do not implement it. Reply on the PR comment:
      > ❌ Not implemented — <clear explanation of why this change is not appropriate or feasible>. #done
      >
      > ---
      > 🤖 *Posted by AFK Agent on behalf of @<GitHub username>*

    - Otherwise:
      > ✅ Implemented — <one sentence explanation of what was done>. #done
      >
      > ---
      > 🤖 *Posted by AFK Agent on behalf of @<GitHub username>*


11. Push all commits once all comments are addressed.
12. Ensure the test suite is green after the full pass.
13. If review feedback requests screenshots, visual proof, or changes user-facing UI, use the `/scbd-agent-pr-screenshot` skill to update the PR's `## User-Facing Changes` section or add a prose fallback when screenshots are not practical.

---

## Constraints & Reminders

- Never push directly to `main`.
- Make sure the local branch is the correct one and is up to date with origin (see phase 1 step 2).
- One ticket per agent run.
- Always reply to every unaddressed comment with a `#done` marker — this is what prevents the review loop from re-processing the same comment on the next run.
- Do not change ticket status during Phase 4 — status remains `PEER REVIEW` until the PR is merged (handled on a future run via Phase 2).
- Keep Jira ticket transitions and PR state in sync at every phase boundary.
