---
name: scbd-jira
description: Work with SCBD Jira tickets through acli — view, search, transition, assign, label, comment, link. Use for ad-hoc Jira requests such as "move DEV-12 to Peer Review" or "who's blocking DEV-40", and as the shared reference for every scbd-dev-* skill's Jira steps.
---

# scbd-jira

Team conventions and an `acli` cheat sheet for SCBD Jira work. Other `scbd-dev-*` skills load this
for their Jira steps; it also runs directly for one-off requests.

**Usage:** `/scbd-jira <request>`, for example "move DEV-12 to Peer Review" or "who does DEV-40
depend on".

Jira access is through `acli` only. Confirm setup with `acli jira auth status` before mutating
anything; if it fails, stop and tell the human to run `acli jira auth login`.

## Read strategy

Ask for only the fields the task needs. Never fetch `*all` or full payloads.

| Task                            | Fields                                                 |
| ------------------------------- | ------------------------------------------------------ |
| Does the ticket/epic exist?     | `summary,status`                                       |
| Epic child inventory            | `summary,status`                                       |
| Candidate assessment            | `summary,status,labels,components,assignee,issuelinks` |
| Selected-ticket planning/review | add `description,comment`                              |

Compact search results into rows before showing them or writing them to a handoff:

```text
key | summary | status | labels | component | assignee
```

Never paste a full Jira JSON payload into the conversation.

## `acli` cheat sheet

```bash
# Auth
acli jira auth status

# View one ticket, narrow fields
acli jira workitem view DEV-1057 --fields summary,status,labels,assignee

# Search
acli jira workitem search --jql "project = DEV AND status = 'To Do'" --fields key,summary,status --json

# Transition
acli jira workitem transition --key DEV-1057 --status "Peer Review"

# Assign
acli jira workitem assign --key DEV-1057 --assignee "@me"

# Labels — see "Labels" below before using --labels
acli jira workitem edit --key DEV-1057 --remove-labels "ready-for-agent" --labels "ready-for-human"

# Comment
acli jira workitem comment create --key DEV-1057 --body "…"

# Link (blocker or relates)
acli jira workitem link create --out DEV-1057 --in DEV-1050 --type Blocks
```

`acli jira workitem --help` and `acli jira workitem <command> --help` list every flag.

**Labels:** whether `edit --labels` replaces or adds the label set has not been confirmed. Treat it
as unverified: pass `--remove-labels` for the label being dropped and `--labels` for the one being
added in the same call, then read the ticket back with `acli jira workitem view <key> --fields
labels` to confirm the result before reporting success.

## Status meanings

| Status        | Meaning                                                                                  |
| ------------- | ---------------------------------------------------------------------------------------- |
| `To Do`       | Not started. Eligible for `/scbd-dev-start` when unblocked and carrying the intake label |
| `In Progress` | Planning or implementation is active                                                     |
| `Peer Review` | An open PR is the source of truth for the work                                           |
| `Done`        | The matching PR is merged, or the work was explicitly closed                             |

Use the status names exactly as written above. The default intake label is `ready-for-agent`; swap
it for `ready-for-human` once implementation is published (see `scbd-dev-pr`).

## Blockers

`search` does not accept `issuelinks` as a field. Check blockers with `view` instead:

```bash
acli jira workitem view DEV-1057 --fields issuelinks --json
```

A ticket is blocked when an issue link's inward description is "is blocked by" (the `Blocks` link
type, viewed from the blocked side) and the linked issue's status is not `Done`. Report blockers
compactly: `<key>: <status>`.

## Comments

Use Jira comments for milestones, state changes, links, blockers, and product decisions.
Implementation detail belongs in commit messages and PR comments, not Jira.

Every comment posted on a human's behalf ends with the attribution line:

```text
🤖 *Posted by <agent name> on behalf of @<username>*
```

`<agent name>` is the running agent's product name (`Claude Code`, `Codex`, …). `<username>` is the
Jira display name for the account `acli` is authenticated as — read it once per session:

```bash
acli jira workitem search --jql "assignee = currentUser()" --fields assignee --json --limit 1
```

(`.fields.assignee.displayName` on the first result if you're currently assigned something; if
nothing is assigned, ask the human for their display name instead of guessing.)

## Action policy

Reads run freely. Every transition, assignment, label change, comment, and link: list the exact
`acli` calls, wait for the human's OK, then run them and read the result back to confirm. For dev
work, this is `scbd-dev-agent`'s action policy; this skill also stands alone for ad-hoc requests.
