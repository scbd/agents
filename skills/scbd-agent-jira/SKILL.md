---
name: scbd-agent-jira
description: Handles Jira state, labels, links, blockers, and audit comments for SCBD agent workflows through Jira Cloud REST.
---

# scbd-agent-jira

Operate on Jira for SCBD agent workflows using the Jira Cloud REST API. Read-only use is allowed
for focused agents; mutations belong to the workflow agent or a human explicitly operating this
skill.

**Usage:** `/scbd-agent-jira ticket=<key> [epic=<key>] action=<assess|start|peer-review|complete|log|link-pr>`

## Inputs

Require a ticket or epic key and the intended action. Resolve component from arguments, then project
`AGENTS.md`'s `scbd_component`. Default intake label is `ready-for-agent`.

Use Jira REST as the normal path. Before any Jira read or mutation, check that these environment
variables are available:

```bash
JIRA_SITE      # example: https://scbd.atlassian.net
JIRA_EMAIL     # Atlassian account email
JIRA_API_TOKEN # Atlassian API token
```

If any are missing, ask the user to define them in their shell startup/private secrets file and stop
before Jira work. Do not fall back to connector/MCP Jira tools unless the user explicitly asks for
that fallback.

Never print or write `JIRA_API_TOKEN`, bearer tokens, cookies, or full auth headers. If a token
appears in a repo file, command output, comment, PR, or handoff, tell the user to rotate/revoke it
and remove it immediately.

## REST Usage

Use Jira Cloud REST endpoints directly:

- Search: `GET $JIRA_SITE/rest/api/3/search/jql`
- Long search: `POST $JIRA_SITE/rest/api/3/search/jql`
- Issue: `GET $JIRA_SITE/rest/api/3/issue/<key>`
- Edit issue or labels: `PUT $JIRA_SITE/rest/api/3/issue/<key>`
- Current user: `GET $JIRA_SITE/rest/api/3/myself`
- Assign issue: `PUT $JIRA_SITE/rest/api/3/issue/<key>/assignee`
- Transitions: `GET $JIRA_SITE/rest/api/3/issue/<key>/transitions`
- Transition: `POST $JIRA_SITE/rest/api/3/issue/<key>/transitions`
- Add comment: `POST $JIRA_SITE/rest/api/3/issue/<key>/comment`
- Update comment: `PUT $JIRA_SITE/rest/api/3/issue/<key>/comment/<comment-id>`
- Link issue: `POST $JIRA_SITE/rest/api/3/issueLink`

Authenticate with basic auth using `$JIRA_EMAIL:$JIRA_API_TOKEN`. Prefer `curl --get` with
`--data-urlencode` for JQL queries so quoting remains predictable.

## Read Strategy

Use narrow Jira reads by default. Avoid `*all`, descriptions, comments, changelog, rendered fields,
avatars, or full payloads unless the current action requires them.

Recommended field sets:

- Epic existence: `summary,status`
- Epic child inventory: `summary,status`
- Workflow candidate assessment: `summary,status,labels,components,assignee,resolution,issuelinks`
- Selected-ticket planning/review context: add `description` and `comment` only after a ticket is
  selected or when a mutation preflight requires them.

For epic-level inventory, first list children with only `summary,status`. Then run focused candidate
queries for active or intake tickets using labels, components, assignee, resolution, and issue links.
Fetch comments/descriptions only for the selected ticket or mutation preflight.

When saving or displaying Jira search results, compact them into rows or small JSON objects. Never
paste full Jira payloads into the conversation or handoff.

Suggested row format:

```text
key | summary | status | labels | component | assignee | blockers
```

## Blockers

For blocker checks, inspect `issuelinks`. A ticket is blocked when a link has
`type.inward == "is blocked by"` and an `inwardIssue` whose status is not `Done` or `Completed`.
Report blockers compactly as `<key>: <status>` unless more detail is required.

## State Rules

- `TO DO` plus intake label is eligible for planning when unblocked.
- `IN PROGRESS` means planned or implementing work is active.
- `PEER REVIEW` means an open PR is the source of truth.
- `Completed` means the matching PR is merged or the work was explicitly closed.
- An `is blocked by` target blocks unless its status is `Done` or `Completed`.
- Replace `ready-for-agent` with `ready-for-human` after implementation publication.
- Keep Jira coherent with PR state at phase boundaries.

## Mutations

Before mutating, inspect the ticket, epic relationship, status, labels, assignee, dependencies,
linked PRs, recent Jira comments if relevant, available transitions, and the matching local/PR state
supplied by the caller.

- Start planning: transition to `IN PROGRESS`, assign the current Jira user, and log the selected
  plan path/branch if known.
- Publish implementation: transition to `PEER REVIEW`, update labels, link or note the PR, and log
  verification/evidence summary.
- Close out: transition to `Completed` only after verifying the exact merged PR-ticket pair.
- Link PR milestones using the relationship or link convention available in the project.
- If preparation fails after a Jira mutation, add a concise failure/blocker comment before stopping.

For workflow audit logs, post comments with the dedicated comment endpoint before or after the
transition, not inside the transition payload. Jira transition calls return `204 No Content` and may
not create bundled comments consistently across workflow screens. Verify the transition state and
latest audit comment with separate readbacks.

## Comments

Use Jira for milestones, state changes, links, blockers, and major product decisions. Avoid
implementation detail that belongs in commits or PR comments.

When posting on behalf of a GitHub user, append the established attribution signature. Use
`AFK Agent` in `mode=afk`; use `HITL Agent` in `mode=interactive`.

```text
🤖 *Posted by <AFK Agent | HITL Agent> on behalf of @<GitHub username>*
```

Comment format:

```text
Agent update: <action>
State: <previous> -> <current>
PR: <url or none>
Verification: <summary>
Blockers: <none or details>
Next: <human or workflow action>
🤖 *Posted by <AFK Agent | HITL Agent> on behalf of @<GitHub username>*
```

When sending comments through Jira REST API v3, convert the comment body to Atlassian Document
Format JSON. Do not send the Markdown/text block above as a raw string body.

## Stop

Stop without further mutation for unavailable credentials/transitions, ambiguous ticket ownership,
blocked work, mismatched PR or branch, missing component, red verification, human judgment, or any
risk of exposing credentials.

## Handoff

```text
Outcome: completed | blocked | stopped
Ticket: <key and summary>
Jira: <status, labels, assignee, links, comments, transitions>
Decisions: <important assumptions>
Blockers: <none or details>
Next action: <recommended caller action>
```
