---
name: scbd-github
description: Work with git and GitHub the SCBD way — branch naming, commits, draft pull requests, the PR body template, fetching and replying to review threads. Use for ad-hoc git/GitHub requests, and as the shared reference for every scbd-dev-* skill's git and GitHub steps.
---

# scbd-github

Team conventions for git and GitHub. Other `scbd-dev-*` skills load this for their branch, commit,
PR, and review-reply steps; it also runs directly for one-off requests.

**Usage:** `/scbd-github <request>`, for example "open a draft PR for this branch" or "what are the
unresolved review threads on PR 42".

## Default branch

Never hardcode `main` or `master`. Discover it each time:

```bash
git symbolic-ref --short refs/remotes/origin/HEAD   # strip the "origin/" prefix
# if that fails:
gh repo view --json defaultBranchRef -q .defaultBranchRef.name
```

Skill text always says "the default branch".

## Branches

- `feature/<jira-key>-<short-slug>` for ticketed work, `feature/<slug>` for ticketless work.
- Create branches from an up-to-date default branch.
- Never push to the default branch.
- Never discard, overwrite, stash, or rewrite work you didn't create without asking first.

## Commits

- Conventional Commits: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`.
- One logical change per commit.
- Stage explicit paths. Never `git add -A` or `git add .`, and never stage `.scratch/`.
- A plan-only commit contains only the plan file: `docs(<jira-key>): add implementation plan`.

## Pull requests

Open PRs as drafts against the default branch, and link the Jira ticket in the body. Only a human
marks a PR ready for review — never change draft status yourself.

```bash
git push -u origin <branch>
gh pr create --draft --base <default-branch> --title "<title>" --body-file <path>
gh pr edit <number> --body-file <path>          # to update an existing PR
gh pr view --json url,number,state,isDraft      # to check current state
```

PR body template:

```markdown
## Details
<description>

## Testing
<verification>

## User-Facing Changes
<evidence or "N/A" if there is none>

## Summary
Closes [<jira-key>](<jira-url>) <if part of an epic: from epic [<epic-key>](<jira-url>)>
```

With no ticket, replace the `## Summary` line with one sentence on why there is none.

Drop the `## User-Facing Changes` section entirely when the change has no user-facing effect. If
`/scbd-dev-screenshot` produced placeholders, include them under that heading — see its skill for
the placeholder format. Files are never uploaded by an agent; the human drags them into the PR
description in the browser.

## Review threads

Fetch unresolved review threads and existing replies with `gh api graphql`. Include each thread's
`id` — replying to it needs that, not the PR number:

```bash
gh api graphql -f query='
query($owner:String!, $repo:String!, $pr:Int!) {
  repository(owner:$owner, name:$repo) {
    pullRequest(number:$pr) {
      reviewThreads(first: 50) {
        nodes {
          id
          isResolved
          comments(first: 20) {
            nodes { id author { login } body url }
          }
        }
      }
    }
  }
}' -f owner=<org> -f repo=<repo> -F pr=<number>
```

A thread is already handled when any of its replies contains `#done`, anywhere in the body — the
attribution signature comes after it, so don't match on "ends with". Skip those threads.

For the rest, post one reply per handled thread on its own thread `id`:

```bash
gh api graphql -f query='
mutation($threadId:ID!, $body:String!) {
  addPullRequestReviewThreadReply(input: { pullRequestReviewThreadId: $threadId, body: $body }) {
    comment { id url }
  }
}' -f threadId=<thread-id> -f body="<reply text>"
```

Use `gh pr comment` instead for top-level conversation comments that aren't part of a review thread.

Reply formats:

```text
Implemented - <summary>. #done
Response - <answer>. #done
Not implemented - <reason>. #done
```

## Attribution

Every PR comment or review reply posted on a human's behalf ends with:

```text
🤖 *Posted by <agent name> on behalf of @<username>*
```

`<agent name>` is the running agent's product name (`Claude Code`, `Codex`, …). `<username>` is the
GitHub login for the authenticated account:

```bash
gh api user -q .login
```

## Checkpoints

Reads, and creating or switching branches, go ahead. Every other git or GitHub change passes a
checkpoint, whether a `scbd-dev-*` command or an ad-hoc request triggers it.

| Key            | Covers                                        | Default   |
| -------------- | --------------------------------------------- | --------- |
| `git.commit`   | Local commits on a feature branch             | `invoked` |
| `github.push`  | Pushing a feature branch                      | `ask`     |
| `github.pr`    | Creating or editing a draft PR, and its push  | `ask`     |
| `github.reply` | PR comments and review replies                | `ask`     |

`git: <mode>` or `github: <mode>` sets every checkpoint of that system. A full key overrides it.

Modes:

- `ask`: list the exact actions, wait for OK, then run them.
- `invoked`: go ahead when the human asked for this action directly: by command (`/scbd-dev-pr`), in
  words ("open a PR"), or by handing over a task that includes it ("work towards X, commit as you
  go"). Otherwise ask.
- `auto`: go ahead, even when you decide on the action yourself.

In every mode, verify the result and report exactly what ran.

Hard limits, whatever any preference says: never push to the default branch, mark a PR ready,
force-push, `reset --hard`, or discard or stash unexplained work.

## Preferences

Resolve each checkpoint's mode:

1. An instruction from the human in this conversation wins, including a `yes` argument on a
   command. It lasts for the session.
2. Otherwise take the human's preferences file, `~/.config/scbd-agents/preferences.md`, or the
   default above.
3. Then apply the project's `AGENTS.md` (`scbd_checkpoints:`) as a floor: where it is stricter
   (`auto` → `invoked` → `ask`), use it.

```markdown
# SCBD agent preferences

- github.pr: invoked
- git.commit: auto
- jira: ask
- Free-text preferences are fine too, e.g. "run the full test suite before any push".
```

If your persistent memory holds a checkpoint preference the file lacks, follow it and offer to add it
to the file. If memory and the file disagree, follow the file and mention the mismatch once.

When the human says not to ask again (or to always ask), offer to write the matching line to the
file. Show the line, and create the file if it's missing.
