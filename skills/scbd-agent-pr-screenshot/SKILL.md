---
name: scbd-agent-pr-screenshot
description: Captures and verifies local screenshot evidence without using git or external services. Use when scbd-agent-epic requests visual proof after implementation or review work, or when a human wants local screenshot artifacts and suggested PR prose.
---

# scbd-agent-pr-screenshot

Create local visual evidence and leave publication to the caller.

**Usage:** `/scbd-agent-pr-screenshot [output=<directory>]`

## Inputs

Require a description of the user-facing change, the relevant local state, capture scenarios, and
an output directory. When invoked directly, derive these from the prompt and local project. Ask the
human for essential missing context rather than accessing Jira, GitHub, or git.

## Workflow

1. Decide whether screenshots are practical. If not, return concise `User-Facing Changes` prose
   and explain why capture was unavailable.
2. Prefer deterministic fixtures, mock data, and the project's existing browser-test setup. Never
   expose secrets, tokens, cookies, or live payloads.
3. Capture the changed surface with a temporary script or test when useful. Keep artifacts in the
   supplied output directory and remove temporary project files after capture.
4. Crop around the changed surface. Use test-only setup or injected screenshot CSS to fix clipping;
   never change product CSS solely for evidence.
5. Inspect every image visually and rerun the capture if it is blank, clipped, misleading, or stale.
6. Stop without hosting artifacts or editing a PR.

## Boundaries

- Do not run any git command, including read-only commands.
- Do not access Jira or GitHub and do not use their CLIs or APIs.
- Do not host images, create gists, edit PRs, or commit artifacts.
- Capture evidence for exactly one implementation or review iteration.

## Handoff

```text
Outcome: captured | prose-only | blocked
Artifacts: <local paths and dimensions>
Scenarios: <what each artifact proves>
Verification: <how each artifact was inspected>
Suggested PR prose: <User-Facing Changes and Testing text>
Temporary files: <removed paths or none>
Blockers: <none or details>
```
