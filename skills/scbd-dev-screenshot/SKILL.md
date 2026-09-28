---
name: scbd-dev-screenshot
description: Capture screenshots that show a UI change, save them to .scratch/, and write PR-ready markdown for the human to finish by uploading the images. Use for "take a screenshot of this" or "get evidence for the PR" once a UI change is in place.
---

# scbd-dev-screenshot

Capture publication-ready screenshots locally. Read-only git is fine to understand the change; no
GitHub or Jira access.

**Usage:** `/scbd-dev-screenshot [<jira-key>] [<what to show>]`

## Before you start

Load `scbd-dev-agent` for the shared ground rules. Regardless:

- Nothing leaves the machine (push, PR, comments, Jira) without listing the actions and getting OK.
- Never push to the default branch, mark a PR ready, force-push, or discard unexplained work.
- Stage explicit paths only; never stage `.scratch/`.

## Recipe

Look for a screenshot recipe in the project's `AGENTS.md`: how to start the app, log in, and reach
the state to capture. If none exists, work it out, then offer to save it (`scbd-dev-agent`'s
Learning section).

## Capture

- Prefer deterministic fixtures and mock data over live data.
- Never capture secrets, tokens, or live personal data.
- Temporary capture scripts go in `.scratch/` and are removed once the run is done.

## Files

Crop to the changed area. Inspect every image, and recapture any that are blank, clipped, or stale.

## Output

- Files: `.scratch/screenshots/<key-or-slug>/NN-<scenario>.png`.
- A `## User-Facing Changes` snippet with one placeholder per image:
  `<!-- drop 01-<scenario>.png here: <what it shows> -->`.
- The instruction: open the PR in the browser, edit the description, and drag each file onto its
  placeholder. GitHub then hosts it under the repo's access rules.
