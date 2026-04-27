---
name: setup-agent-logs
description: First-run setup for the session log archive. Creates a private Agent-Logs repo on GitHub, clones it locally, and saves the path to memory so handovers auto-archive there. Triggers on phrases like "set up agent logs", "configure handover logging", "where should handovers go", "setup session logs".
---

# Setup Agent Logs

One-time setup that creates the private repo where session handovers are archived, and persists the location so the handover-writer agent can find it in future sessions.

## When to use

- First time using the session-transfer plugin on a new machine
- User says "set up agent logs", "configure handover logging", "setup session logs"
- The handover-writer can't find the archive repo and suggests running this

## Migration from legacy path (one-time)

Before resolving the data directory, check for legacy data at these paths:
- `~/.claude/projects/-home-*/memory/reference_agent_logs.md`

If a legacy file exists AND `<plugin-data-dir>/references/agent_logs.md` does NOT exist, move it to the new location and delete the legacy file. Tell the user: "Migrated reference_agent_logs.md from ~/.claude/projects/.../memory/ to <new>."

## Path resolution

Resolve the plugin's data directory as `$CLAUDE_USER_DATA/dev-tools/` if `CLAUDE_USER_DATA` is set; otherwise `$XDG_DATA_HOME/claude-plugins/dev-tools/` if `XDG_DATA_HOME` is set; otherwise `~/.local/share/claude-plugins/dev-tools/`. Create the directory (and a `references/` subdirectory) if it doesn't exist. See the canonical convention in the `claude-rudder:plugin-data-storage` skill.

## Procedure

### 1. Check if already configured

Check `<plugin-data-dir>/references/agent_logs.md` for a saved Agent-Logs archive path. If it exists, read it and confirm the path is still valid:

```bash
ls <saved-path>/README.md
```

If the path exists and is a git repo, tell the user it's already set up and show the path. Done.

### 2. Ask where to store logs

Ask the user where they want the Agent-Logs repo. Suggest a default based on their filesystem:

- If `~/repos/github/my-repos/` exists: suggest `~/repos/github/my-repos/Agent-Logs/`
- Otherwise: suggest `~/Agent-Logs/`

The user can accept the default or provide a custom path.

### 3. Check if the repo already exists on GitHub

```bash
gh repo view danielrosehill/Agent-Logs --json name 2>/dev/null
```

If it exists, clone it to the chosen path and skip to step 5.

### 4. Create the repo

Create the directory, initialize it, and push to GitHub as a **private** repo:

```bash
mkdir -p <path>
cd <path>
git init
git branch -m main
```

Write the repo by cloning the template from the existing `danielrosehill/Agent-Logs` repo on GitHub if it exists. If not, scaffold manually with these files:

- `CLAUDE.md` -- **required** -- defines the repo's purpose, structure, JSON schema, and how to query. Seed it with:

```markdown
# Agent-Logs

Private archive of Claude Code session handovers and agent logs.

## Purpose

This repo stores structured JSON session logs produced by the handover-writer agent every time a session handover is created. It serves as a queryable history of all Claude Code sessions -- what was worked on, what's pending, what's blocked.

## Structure

\`\`\`
DDMMYY/
  HHMMSS_model-name.json     Structured session log (one per handover)
sessions.db                   SQLite database (derived, gitignored)
scripts/
  ingest.py                   Scans JSON files → inserts into SQLite
\`\`\`

## How logs arrive

The session-transfer plugin's handover-writer agent automatically writes a JSON file here after every handover. The agent then runs python3 scripts/ingest.py to update the SQLite index.

## JSON schema

Each log file contains: created, model, agent_type, working_directory, repository, branch, last_commit, blocked, user_input_needed, debugging_required, pending_tasks, goal, completed, in_progress, not_started, blockers, resumption_summary.

## Working in this repo

- Don't edit JSON files -- they are append-only records written by agents
- Regenerate the DB at any time: python3 scripts/ingest.py
- Query with SQLite: sqlite3 sessions.db "SELECT created, model, goal FROM sessions ORDER BY created DESC LIMIT 10"
- The DB is gitignored -- it's a derived artifact rebuilt from JSON files
```

- `README.md` -- documenting the JSON + SQLite structure
- `.gitignore` with `.DS_Store`, `.env`, `sessions.db`
- `scripts/ingest.py` -- the ingestion script that scans `DDMMYY/*.json` and inserts into `sessions.db`

The ingestion script, CLAUDE.md, and README are maintained in the canonical `danielrosehill/Agent-Logs` repo -- prefer cloning over scaffolding from scratch.

Commit and push:

```bash
git add -A
git commit -m "Initial scaffold for agent session logs"
gh repo create danielrosehill/Agent-Logs --private --source=. --push
```

### 5. Save to memory

Write a reference file so future sessions know where the archive lives:

**File**: `<plugin-data-dir>/references/agent_logs.md`

```markdown
---
name: Agent Logs archive location
description: Path to the private Agent-Logs repo where session handovers are archived by the handover-writer agent
type: reference
---

Agent session logs are archived at `<absolute-path>`.

GitHub repo: `danielrosehill/Agent-Logs` (private).

Structure: `DDMMYY/HHMMSS_model-name.md` folders organized by date.
```

(The reference is read directly from this path; no global memory index is needed.)

### 6. Confirm

Tell the user:
- The repo is created at `<path>`
- GitHub URL (private)
- Handovers will now auto-archive there as `DDMMYY/HHMMSS_model.json` files
- The ingestion script (`scripts/ingest.py`) builds a queryable SQLite DB from the JSON logs
- They can run this skill again on another machine to set up the same archive
