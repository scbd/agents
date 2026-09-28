---
name: scbd-dev-feedback
description: Address review feedback on your own pull request — triage comments, make fixes, and draft and post replies. Use for "address the review feedback" or "handle the comments on my PR". Do not use this for reviewing someone else's PR.
---

# scbd-dev-feedback

Work through one review cycle on a PR you opened: triage, fix, verify, and reply.

**Usage:** `/scbd-dev-feedback [<jira-key> | <pr-number>]`

## Before you start

Load `scbd-dev-agent` for the shared ground rules. Regardless:

- Nothing leaves the machine (push, PR, comments, Jira) without listing the actions and getting OK.
- Never push to the default branch, mark a PR ready, force-push, or discard unexplained work.
- Stage explicit paths only; never stage `.scratch/`.

## Gather

Fetch unresolved review threads through `gh api graphql` (`reviewThreads`, `isResolved`), plus PR
conversation comments (`scbd-github`). Skip any thread where a reply already contains `#done`.

## Triage

Show a numbered list, each item classified as a code change, an answer, or a decline. The human can
drop or redirect any item before work starts — the human decides, there's no "addressed to the
agent" rule.

## Work

- Fix and verify.
- Commit under `scbd-dev-agent`'s action policy.
- Feedback on a plan updates the plan file only, not code.

## Replies

One reply per handled comment:

```text
Implemented - <summary>. #done
Response - <answer>. #done
Not implemented - <reason>. #done
```

## Publish

Show the push and the replies as one itemised list. After OK, post them and verify. Jira status is
left unchanged.
