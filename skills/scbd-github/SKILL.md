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

## Action policy

Branch creation and local commits (once the human has handed over autonomy, or asks) go ahead.
Pushing, creating or editing a PR, and posting comments or replies: list the exact actions, wait for
OK, then run them and verify the result. For dev work, this is `scbd-dev-agent`'s action policy;
this skill also stands alone for ad-hoc requests.
