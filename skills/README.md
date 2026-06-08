# Skills catalog

## Design philosophy — the Ralph Wiggum loop

These skills are built with the **Ralph Wiggum loop** concept in mind (from Andrej Karpathy's work on agentic systems): an agent runs once inside a clean context window, does exactly one unit of work, then exits. The loop is external — a scheduler or human invokes the skill again for the next unit of work.

> **Note:** the Ralph Wiggum loop itself is not implemented here. These skills are designed to be called *from* a Ralph loop — they are the payload, not the loop. Each skill assumes it starts with a clean context and exits when its single task is done.

Each skill follows this contract:
- Stateless — no memory of previous runs
- One ticket, one review cycle, one PR per run
- Clean exit so the caller can reset context and run again

## Available skills

| Skill                  | Description                                                                                                    | Dependencies          |
| ---------------------- | -------------------------------------------------------------------------------------------------------------- | --------------------- |
| `scbd-agent-implement` | Pick and implement the next unblocked Jira ticket end-to-end                                                   | `karpathy-guidelines` |
| `scbd-agent-review`    | Address peer-review comments on in-progress PRs                                                                | `karpathy-guidelines` |
| `scbd-push-to-jira`    | Upload local markdown issues from `.scratch/<feature-slug>/` to Jira (Epic + Stories + blocked-by links)      | —                     |

`scbd-agent-implement` and `scbd-agent-review` accept `<epic> [component] [label]` arguments and read a project-level `scbd_component:` default from the target project's `AGENTS.md` when no component argument is passed.

### End-to-end planning → implementation workflow

`scbd-push-to-jira` bridges the [mattpocock/skills](https://github.com/mattpocock/skills) planning tools and the `scbd-agent-*` implementation tools:

1. **`/to-prd`** (mattpocock) — generates a PRD and individual issue files and saves them to `.scratch/<feature-slug>/` on local disk.
2. **Human review** — the developer reviews and edits `.scratch/<feature-slug>/` directly, optionally with agent assistance. The local-markdown save exists specifically for this gate: nothing reaches Jira until a human has signed off. This step is required.
3. **`/scbd-push-to-jira <feature-slug>`** — reads those local files and creates one Epic (from `PRD.md`) and one Story per issue file in Jira, wires `Blocks` links, and writes the resulting Jira keys back into every markdown file.
4. **`/scbd-agent-implement`** / **`/scbd-agent-review`** — pick up the Jira tickets created in step 3 and implement them.

Any issue file that already contains a `## Jira` section is skipped automatically; pass `--force` to re-upload.

## Invoking skills

```bash
/scbd-agent-implement DEV-20                        # component from AGENTS.md
/scbd-agent-implement DEV-20 Gaia/KM
/scbd-agent-implement DEV-20 Gaia/KM my-label

/scbd-agent-review DEV-20
/scbd-agent-review DEV-20 Gaia/KM

/scbd-push-to-jira meeting-documents-nestjs-migration
/scbd-push-to-jira meeting-documents-nestjs-migration project=DEV
/scbd-push-to-jira meeting-documents-nestjs-migration project=DEV component="Gaia/Km" label=ready-for-agent
/scbd-push-to-jira meeting-documents-nestjs-migration --force    # re-upload tickets that already have Jira keys
```

## Setting up a new project

Add these lines to the project's `AGENTS.md` so skills can pick up defaults:

```
scbd_component:     Your/Component    # used by all scbd-agent-* and scbd-push-to-jira
scbd_jira_project:  DEV               # used by scbd-push-to-jira
```

## External dependencies

Install these once, globally:

| Skill                 | Source                                | Required by                                   |
| --------------------- | ------------------------------------- | --------------------------------------------- |
| `karpathy-guidelines` | `multica-ai/andrej-karpathy-skills`   | `scbd-agent-implement`, `scbd-agent-review`   |

```bash
npx skills add multica-ai/andrej-karpathy-skills --skill karpathy-guidelines -g
```

## Adding a new skill

1. Create `skills/<name>/SKILL.md` with YAML frontmatter:
   ```yaml
   ---
   name: <name>
   description: One-sentence description of what the skill does and when to use it.
   ---
   ```
2. Add a row to the skills table above.
3. List any external dependencies in the Dependencies table above.
