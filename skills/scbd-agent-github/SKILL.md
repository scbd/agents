---
name: scbd-agent-github
description: Handles git branches, commits, GitHub pull requests, review replies, and hosted evidence for SCBD agent workflows. Use when creating, updating, verifying, or closing PR-linked SCBD work.
---

# scbd-agent-github

Operate on local git and GitHub for SCBD agent workflows. Read-only use is allowed for focused
agents; mutations belong to the epic agent or a human explicitly operating this skill.

**Usage:** `/scbd-agent-github ticket=<key> action=<prepare-branch|open-pr|update-pr|reply-review|host-evidence|close-out>`

## Branches And Commits

- Match branches and PRs by ticket reference and `feature/<ticket-key>-*`.
- Create feature branches from current `main` as `feature/<ticket-key>-<short-slug>`.
- Never push to `main`.
- Never discard, overwrite, stash, or rewrite unexplained work.
- Commit only coherent changes. Use Conventional Commits: `feat`, `fix`, `refactor`, `test`,
  `docs`, or `chore`.
- Planning commits contain only the plan file and use `docs(<ticket-key>): add implementation plan`.

## Pull Requests

New PRs target `main`, remain draft, and link the Jira ticket. Never change draft status; only a
human marks a PR ready.

Keep the PR body current:

```markdown
## Summary
Closes [<ticket-key>](<jira-url>)

## Plan or Implementation
<current description>

## Testing
<verification>

## User-Facing Changes
<evidence or prose fallback, if applicable>
```

Add one PR comment per published iteration with action, files, technical decisions, verification,
evidence, and next state. Reply to original review comments with the focused agent's proposed
`#done` responses after verifying the changes.

## Evidence

Keep screenshots out of the project repository unless it explicitly stores PR assets.

To host screenshots:

1. Wrap each PNG in a text SVG containing an embedded `data:image/png;base64,...`.
2. Create a secret gist with the SVG files. `gh gist create` creates secret gists by default; use
   `--public` only when explicitly requested.
3. Fetch exact raw URLs with
   `gh api gists/<gist-id> --jq '.files | to_entries[] | [.key, .value.raw_url] | @tsv'`.
4. Embed raw SVG URLs under `## User-Facing Changes`, link the gist, mention capture in
   `## Testing`, and verify the rendered PR body.
5. Remove temporary local artifacts after verifying the gist and PR update.

Create an SVG wrapper with actual PNG dimensions:

```bash
node -e "const fs=require('fs'); const [src,out,w,h]=process.argv.slice(1); const b64=fs.readFileSync(src).toString('base64'); fs.writeFileSync(out, '<svg xmlns=\"http://www.w3.org/2000/svg\" width=\"'+w+'\" height=\"'+h+'\" viewBox=\"0 0 '+w+' '+h+'\"><image width=\"'+w+'\" height=\"'+h+'\" href=\"data:image/png;base64,'+b64+'\"/></svg>')" screenshot.png screenshot.svg 1200 800
```

Use `file screenshot.png` to obtain dimensions when image tooling is unavailable.

## Stop

Stop without further mutation for unsafe workspace state, branch/PR mismatch, closed-unmerged PRs,
unavailable credentials, red verification, failed publication, or human judgment.

## Handoff

```text
Outcome: completed | blocked | stopped
Ticket: <key and summary>
Git/GitHub: <branch, commits, PR, replies, evidence>
Verification: <checks and rendered PR validation>
Blockers: <none or details>
Next action: <recommended caller action>
```
