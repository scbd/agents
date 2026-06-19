---
name: scbd-agent-pr-screenshot
description: Capture, crop, verify, host, and link PR screenshots for user-facing UI changes or review feedback. Use when a PR needs visual proof, a User-Facing Changes section, screenshot updates, or a prose fallback for projects without reliable screenshot/e2e support.
---

# scbd-agent-pr-screenshot

Create PR-ready evidence for user-facing UI changes.

## Workflow

1. Decide whether screenshots are practical.
   - If the project has reliable UI automation or an easy local app path, capture screenshots.
   - If the project lacks a practical screenshot path, write a clear `## User-Facing Changes` prose summary and note in `## Testing` that screenshots were not captured because screenshot/e2e support is unavailable.

2. Prefer deterministic data.
   - Use fixture/mock data instead of live data.
   - Use the project's existing Playwright/e2e fixture auth mode when available.
   - Avoid printing secrets, tokens, cookies, or full live payloads.

3. Capture the changed UI.
   - Use a temporary screenshot-only Playwright spec or script when that is the cleanest path.
   - Remove temporary screenshot specs/scripts before the final commit unless the project wants to keep screenshot coverage.
   - Crop around the changed surface, not the whole app.
   - Verify the images visually before committing or linking them.

4. Handle clipping without product-only changes.
   - If menus, scroll containers, or overflow clip the crop, adjust the test setup or inject screenshot-only CSS in the temporary spec.
   - Do not change product CSS only to make a screenshot work.

5. Host images according to project preference.
   - Keep screenshot artifacts out of the repo unless the project explicitly stores PR assets.
   - For GitHub Gist hosting, `gh gist create` rejects binary PNGs. Wrap each PNG in a text SVG with an embedded `data:image/png;base64,...`, create a secret gist, and embed the gist raw SVG URL in the PR body.
   - `gh gist create` creates secret gists by default. Some installed versions do not support `--secret`; use `--public` only when a public gist is explicitly wanted.
   - After creating a gist, fetch the exact raw URLs with `gh api gists/<gist-id> --jq '.files | to_entries[] | [.key, .value.raw_url] | @tsv'`. Do not guess raw URLs from the web URL.
   - If repo-hosting screenshots temporarily, remove them once gist-hosted or otherwise externally hosted.

6. Update the PR.
   - Add or update `## User-Facing Changes` with concise bullets and embedded screenshots when available.
   - Include the screenshot gist/link when using external hosting.
   - Mention screenshot generation in `## Testing`, including if the screenshot spec/script was temporary and removed.
   - When generating a Markdown PR body from a shell command, avoid JavaScript template literals if the Markdown contains backticks. Build the body from an array of strings or write it to a temp file with a safer editor/script, then patch the PR from that file.
   - Verify the PR body after patching, then remove temporary specs/scripts and confirm `git status --short` is clean unless the screenshot test is intentionally kept.

## SVG Wrapper Pattern

Use this pattern when GitHub Gist needs to host PNG screenshots:

```bash
node -e "const fs=require('fs'); const [src,out,w,h]=process.argv.slice(1); const b64=fs.readFileSync(src).toString('base64'); fs.writeFileSync(out, '<svg xmlns=\"http://www.w3.org/2000/svg\" width=\"'+w+'\" height=\"'+h+'\" viewBox=\"0 0 '+w+' '+h+'\"><image width=\"'+w+'\" height=\"'+h+'\" href=\"data:image/png;base64,'+b64+'\"/></svg>')" screenshot.png screenshot.svg 1200 800
```

Get PNG dimensions with `file screenshot.png` if image tooling is unavailable.
