---
name: scbd-agent-screenshot
description: Captures, crops, and verifies local screenshot evidence without git or external services. Use when an epic iteration or human needs publication-ready visual proof and suggested PR prose.
---

# scbd-agent-screenshot

Create publication-ready local visual evidence and leave hosting and PR updates to the caller.

**Usage:** `/scbd-agent-screenshot [output=<directory>]`

## Inputs

Require the user-facing change, local state, scenarios, and output directory. For direct use, derive
them locally. Ask for missing essentials; never inspect Jira, GitHub, or git.

## Workflow

1. If capture is impractical, return concise `User-Facing Changes` prose and the reason.
2. Prefer deterministic fixtures, mock data, and the project's existing browser-test setup. Never
   expose secrets, tokens, cookies, or live payloads.
3. Capture with a temporary script or test when useful. Write only to the output directory; remove
   temporary project files.
4. Crop around the changed surface. Use test-only setup or injected screenshot CSS to fix clipping;
   never change product CSS solely for evidence.
5. Inspect each image; recapture if blank, clipped, misleading, or stale.
6. Return the accepted local artifacts and suggested PR prose without hosting or publishing them.

## Boundaries

- Do not access git, Jira, or GitHub, including through CLIs or APIs.
- Do not host images, create gists, edit PRs, or commit artifacts.
- Capture evidence for exactly one implementation or review iteration.

## Handoff

```text
Outcome: captured | prose-only | blocked
Artifacts: <local paths and dimensions>
Scenarios: <what each artifact proves>
Verification: <how each artifact was inspected>
Publication readiness: <accepted artifacts safe to host, or reason none are available>
Suggested PR prose: <User-Facing Changes and Testing text>
Temporary files: <removed paths or none>
Blockers: <none or details>
```
