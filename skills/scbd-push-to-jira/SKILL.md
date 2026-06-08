---
name: scbd-push-to-jira
description: Upload local markdown issues from .scratch/<feature-slug>/ to Jira. Creates one Epic (from PRD.md), one Story per issue file, wires blocked-by links, and writes Jira keys back to every markdown file. Manually triggered only — not auto-triggered.
---

# scbd-push-to-jira

Push a local markdown feature (PRD + issues) to Jira in one pass.

**Usage:** `/scbd-push-to-jira <feature-slug> [project=<KEY>] [component=<name>] [label=<label>] [--force]`

---

## Phase 0 — Parse Arguments

Arguments received: `$ARGUMENTS`

Parse `$ARGUMENTS` accepting either positional order or `key=value` pairs:

| Parameter      | Position | Key           | Default                                                | Required |
|----------------|----------|---------------|--------------------------------------------------------|----------|
| `feature-slug` | 1st      | `slug=`       | —                                                      | **yes**  |
| `project`      | 2nd      | `project=`    | `scbd_jira_project:` from AGENTS.md → ask user         | **yes**  |
| `component`    | 3rd      | `component=`  | `scbd_component:` from AGENTS.md → ask user            | **yes**  |
| `label`        | 4th      | `label=`      | each issue's own `Status:` value (resolved per-ticket) | no       |
| `--force`      | flag     | —             | false                                                  | no       |

**If `feature-slug` is missing**, stop immediately and print — do nothing else:

```
Missing required parameter: feature-slug

Usage: /scbd-push-to-jira <feature-slug> [project=<KEY>] [component=<name>] [label=<label>] [--force]

  feature-slug   .scratch/ subdirectory name, e.g. meeting-documents-nestjs-migration   (required)
  project        Jira project key, e.g. DEV                                             (required)
                 Set a default in AGENTS.md:  scbd_jira_project: DEV
  component      Jira component name, e.g. "Gaia/Km"                                   (optional)
                 Falls back to scbd_component in AGENTS.md
  label          Jira label applied to all tickets                                      (optional)
                 Default: each issue's Status: line value
  --force        Re-upload tickets that already have a ## Jira section                 (optional)

Examples:
  /scbd-push-to-jira meeting-documents-nestjs-migration
  /scbd-push-to-jira meeting-documents-nestjs-migration project=DEV
  /scbd-push-to-jira meeting-documents-nestjs-migration project=DEV component="Gaia/Km"
  /scbd-push-to-jira slug=meeting-documents-nestjs-migration project=DEV label=ready-for-agent --force
```

**Resolve `project`:**
1. Use the `project=` argument if provided.
2. Otherwise read `AGENTS.md` at the project root and look for `scbd_jira_project: <KEY>`.
3. If still unresolved, ask the user: *"Which Jira project key should I upload to (e.g. DEV)?"* — wait for the answer before continuing.

**Resolve `component`:**
1. Use the `component=` argument if provided.
2. Otherwise read `scbd_component:` from `AGENTS.md`.
3. If neither source provides a value, proceed without a component — do not stop.

**Resolve `label`:**
If the `label=` argument was provided, use it for every ticket. If not, each ticket will use its own `Status:` value as its label (resolved in Phase 4).

Display resolved values and stop for confirmation before touching Jira:

```
FEATURE:    .scratch/<feature-slug>/
PROJECT:    <KEY>
COMPONENT:  <value or "(none)">
LABEL:      <value or "(per-issue Status:)">
FORCE:      <yes/no>
```

---

## Phase 1 — Verify Setup

1. Check that `docs/agents/issue-tracker.md` exists. If it does not, stop and print:

   ```
   docs/agents/issue-tracker.md not found.
   This repo is not configured as a local markdown issue tracker.
   Run /setup-matt-pocock-skills to set it up.
   ```

2. Read `docs/agents/issue-tracker.md` to confirm the `.scratch/` convention is in effect and understand where issue files live.

3. Read `docs/agents/triage-labels.md` and load the label mapping table. You will use it in Phase 4 to convert each issue's `Status:` value to the correct Jira label string.

4. Verify that `.scratch/<feature-slug>/` exists. If it does not, stop and print:

   ```
   Feature directory not found: .scratch/<feature-slug>/
   Check the slug and try again.
   ```

---

## Phase 2 — Read Feature Files

5. Read `.scratch/<feature-slug>/PRD.md`. Extract:
   - **Epic summary:** text of the first `# ` heading (strip the `# ` prefix).
   - **Full PRD body:** entire file content — this becomes the Epic description.

6. List all files matching `.scratch/<feature-slug>/issues/NN-*.md`, sorted numerically by the `NN` prefix. This order is used in Phases 4 and 5.

7. For each issue file, read it and check for an existing `## Jira` section (a line starting with `## Jira`).
   - If `--force` was **not** passed and a `## Jira` section is present, mark that file **skip**.
   - If `--force` was passed, clear all skip marks.

8. If ALL issue files are marked skip and `--force` was not passed, stop and print:

   ```
   All issues in .scratch/<feature-slug>/issues/ already have Jira keys.
   Pass --force to re-upload and overwrite existing keys.
   ```

9. Also check PRD.md for an existing `## Jira` section. If found and `--force` was **not** passed,
   mark the Epic **skip** — extract the existing key from the link (e.g. `[DEV-994](...)` → `DEV-994`)
   and store it as `EPIC_KEY`. Phase 3 will be skipped; `EPIC_KEY` is still used as the parent in Phase 4.

10. Print an upload summary and ask for confirmation before proceeding:

    ```
    Ready to upload:
      Epic:   <PRD title>
      Issues: <N> of <total> (<skipped> skipped — already have Jira keys)

    Proceed? [y/N]
    ```

---

## Phase 3 — Create Epic

**If the Epic was marked skip in step 9, skip this entire phase** — go directly to Phase 4 using the existing `EPIC_KEY`.

11. Discover the fields needed to create an Epic in the target project:
    - Call `mcp__claude_ai_Atlassian__getJiraProjectIssueTypesMetadata` with the resolved project key to get the `Epic` issue type `id`.
    - Call `mcp__claude_ai_Atlassian__getJiraIssueTypeMetaWithFields` for that type to find the field ID for "Epic Name" (commonly `customfield_10011` but varies per instance). You need this field to set the epic name alongside `summary`.

12. Call `mcp__claude_ai_Atlassian__createJiraIssue` to create the Epic:

    | Field       | Value                                             |
    |-------------|---------------------------------------------------|
    | `project`   | resolved project key                              |
    | `issuetype` | `Epic`                                            |
    | `summary`   | PRD title (from step 5)                           |
    | `description` | full PRD.md content (from step 5)               |
    | Epic Name   | PRD title (using the field ID discovered in step 11) |
    | `components`| resolved component (omit the field entirely if none) |
    | `labels`    | resolved label if the `label=` argument was provided; otherwise omit — do not apply any label to the Epic |

    Store the returned key as `EPIC_KEY` and the browse URL as `EPIC_URL`.

13. Write the Epic key back to PRD.md by appending (or replacing if `--force`) a `## Jira` section at the end of the file:

    ```markdown

    ## Jira
    Epic: [<EPIC_KEY>](<EPIC_URL>)
    ```

---

## Phase 4 — Create Child Issues (Stories)

Process issue files in the sorted numerical order from step 6. For each file **not marked skip**:

14. Parse the issue file. Extract:
    - **Summary:** the `# NN — ...` heading text (strip the leading `# `).
    - **Description:** the "## What to build" section and the "## Acceptance criteria" section concatenated (preserve full text of both, with a blank line between them). This becomes the Jira description.
    - **Label:** if the `label=` argument was provided, use it. Otherwise read the `Status:` line near the top of the file (e.g. `Status: ready-for-agent`) and look up the right-hand "Label in our tracker" column from `triage-labels.md`.

15. Call `mcp__claude_ai_Atlassian__createJiraIssue`:

    | Field        | Value                                              |
    |--------------|----------------------------------------------------|
    | `project`    | resolved project key                               |
    | `issuetype`  | `Story`                                            |
    | `summary`    | issue title from step 14                           |
    | `description`| description body from step 14                     |
    | `parent`     | `EPIC_KEY` (links the Story under the Epic)        |
    | `components` | resolved component (omit if none)                  |
    | `labels`     | label from step 14                                 |

    Store the returned key and URL. Add an entry to the tracking map:

    ```
    { "./issues/NN-<slug>.md" → "DEV-NNN" }
    ```

    The key is the relative path from `.scratch/<feature-slug>/` — matching exactly how `## Blocked by` entries are written in sibling issue files.

16. Print progress as each Story is created:

    ```
    Created DEV-NNN  |  NN — <summary>
    ```

---

## Phase 5 — Set Blocked-By Links

Run this phase **only after the full Phase 4 loop completes** — all Stories must exist before any link is created.

17. Iterate the issue files again. For each file that has a `## Blocked by` section:
    - Parse every list item under `## Blocked by`. Items are markdown links, e.g.:
      `- [04 — File & log operations](./04-file-and-log-operations.md)`
    - Extract the relative path from the link target (`./04-file-and-log-operations.md`).
    - Look up that path in the tracking map from Phase 4.
    - If not found (blocker was skipped or unmapped), print a warning and skip:
      ```
      Warning: could not resolve blocker "./04-file-and-log-operations.md" for DEV-NNN — link skipped.
      ```
    - Otherwise call `mcp__claude_ai_Atlassian__createIssueLink`:

      | Field          | Value                                              |
      |----------------|----------------------------------------------------|
      | `type`         | `"Blocks"`                                         |
      | `inwardIssue`  | blocker's Jira key (the ticket that blocks)        |
      | `outwardIssue` | current issue's Jira key (the one being blocked)   |

---

## Phase 6 — Write Back Jira Keys

18. For each issue file that was uploaded in Phase 4, append a `## Jira` section at the end of the file. If a `## Jira` section already exists (re-upload with `--force`), replace it:

    ```markdown

    ## Jira
    [<JIRA-KEY>](<JIRA-URL>)
    ```

    Do not reformat or reorder any other content in the file.

---

## Phase 7 — Report Summary

19. Print a results table:

    ```
    Jira Key   | Summary                                     | Result
    -----------|---------------------------------------------|----------------
    DEV-100    | PRD: Migrate meetingDocument Controller...  | Epic created
    DEV-101    | 01 — Baseline test suite                    | Story created
    DEV-102    | 02 — Module scaffold + read paths           | Story created
    DEV-103    | 03 — Write paths                            | Story created
    DEV-104    | 04 — File & log operations                  | Story created
    DEV-105    | 05 — Pin operations                         | Story created
    DEV-106    | 06 — Cutover                                | Story created
    -----------|---------------------------------------------|----------------
    Blocked-by links created: 4
    Files written back:        7
    Skipped (existing keys):   0
    Errors:                    0
    ```

    List any errors (failed `createJiraIssue` calls) below the table with their file names and the Jira API error message.

---

## Constraints & Reminders

- **Any markdown file that already contains a `## Jira` section with a ticket reference is skipped — never re-uploaded.** This applies equally to `PRD.md` (Epic) and every issue file. Pass `--force` only when explicitly asked to re-push a ticket that was already uploaded.
- Never create Jira tickets without the user's confirmation at the end of Phase 2.
- Phase 5 (blocked-by links) runs only after Phase 4 is fully complete — the tracking map must be complete before any `createIssueLink` call.
- If `createJiraIssue` fails for one issue, log the error, skip that file, and continue. Report all failures in Phase 7 — do not abort the whole run.
- Tracking map keys use relative paths from `.scratch/<feature-slug>/` — match them exactly as written in `## Blocked by` list items (usually `./NN-<slug>.md`).
- The Epic Name custom field ID must be discovered via `getJiraIssueTypeMetaWithFields` — do not hard-code `customfield_10011`.
- Phase 6 appends to existing files. Never reformat, reorder, or touch any content above the `## Jira` section.
