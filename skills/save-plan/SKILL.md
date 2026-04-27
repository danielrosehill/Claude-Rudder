---
name: save-plan
description: Save a Claude Code plan (from ExitPlanMode, design discussion, or multi-step task) to persistent storage. ONLY invoke when the user explicitly asks to save/stash/store a plan (e.g. "save this plan", "stash the plan", "store the plan in the planning repo"). Do NOT auto-invoke after planning. Routes to the active project's `planning/` folder when one exists, else to the user's configured central planning repo.
---

# Save Plan

Persist a written plan so it survives the session.

## Trigger discipline

**ONLY** run when the user explicitly asks. Phrases that trigger:
- "save this plan", "save the plan"
- "stash the plan", "store the plan"
- "put this in the planning repo", "park this plan"

Do **not** invoke automatically after writing or exiting a plan. Plans live in conversation context by default; persistence is opt-in.

## Routing

1. Check the current working directory's repo (walk up to the git root if needed).
2. If a `planning/` folder exists in that repo → save there.
3. Otherwise → save to the user's configured **central planning repo** (see config below).

If the user explicitly says "save to the planning repo" or "central planning repo", skip step 2 and go straight to the central repo.

## Configuration

Read central-repo path from `claude-rudder` plugin config per `claude-rudder:plugin-data-storage`:

1. Resolve config dir in this priority: `$CLAUDE_USER_DATA` → `$XDG_DATA_HOME/claude-plugins` → `~/.local/share/claude-plugins`.
2. Read `<config-dir>/claude-rudder/config.json`, key `planning.central_repo` (an absolute path or `~`-prefixed path).
3. If unset, prompt the user once for the path, write it back to the config, then proceed.

Example config:
```json
{ "planning": { "central_repo": "~/repos/Planning-Files" } }
```

## File naming

`DD-MM-YY--<slug>.md`

- `DD-MM-YY` — today's date, hyphens not slashes.
- `<slug>` — short kebab-case description of the plan subject.
- Example: `19-04-26--refactor-snapcast-skills.md`

## File body

```markdown
---
title: <one-line plan title>
created: DD-MM-YY
status: pending
source: <repo or context the plan came from, or "ad-hoc">
---

# <Title>

<full plan content — preserve structure, code blocks, decision points>
```

## After writing

- **Central repo**: `git add`, commit (`Add plan: <title>`), `git push` immediately so the plan is durable across machines.
- **In-repo `planning/`**: do NOT auto-commit. The plan is part of the project's working tree and the user will commit it with related work.

## Marking a plan done

When the user says a plan is complete:
- Move file from `pending/` → `done/` (central repo) or leave in place (in-repo).
- Update frontmatter `status: done`.
- Commit and push if in the central repo.
