---
name: scaffold-planning
description: Create a lightweight `planning/` folder in the current repo with leftover-tasks.md, blockers.md, session-logs/, and handovers/. Idempotent — leaves existing files alone. Acknowledges task-queuer's heavyweight structure if present without duplicating. Use when the user says "scaffold planning", "set up planning folder", "init planning", or when another claude-rudder skill needs the structure and finds it missing.
---

# Scaffold Planning Folder

Lightweight planning structure for repos. Lives at `<repo-root>/planning/`. Designed to coexist with `task-queuer@danielrosehill`'s heavier queue if both are in use.

## What it creates

```
<repo-root>/planning/
  README.md              # index — what each file is for, which skills consume it
  leftover-tasks.md      # follow-ups deferred mid-flow (capture-leftover writes here)
  blockers.md            # known blockers ordered by urgency (log-blocker writes here)
  session-logs/          # one file per significant session, append-only
    .gitkeep
  handovers/             # context-gate / handover skills write here
    .gitkeep
```

## When to run

Run automatically (silently) when another claude-rudder skill needs to write to one of these files and finds it missing. Run explicitly when the user asks to set up the planning structure for a new repo.

## Idempotency

This skill **never overwrites** existing content. For each artifact:

- If file exists → skip silently
- If file missing → create with the seed content below
- If `planning/` doesn't exist at all → create the whole tree

So running it twice (or a hundred times) on the same repo is safe.

## Steps

### 1. Resolve repo root

```bash
repo_root=$(git rev-parse --show-toplevel 2>/dev/null) || repo_root="$PWD"
```

If `cwd` is not in a git repo, ask the user where to scaffold (default to cwd, but confirm).

### 2. Detect task-queuer

If `<repo-root>/planning/tasks/in-queue/` exists, task-queuer's queue is already set up here. **Don't touch its files.** Only add the lightweight artifacts (leftover-tasks.md, blockers.md, session-logs/, handovers/) alongside. The README.md should mention both systems coexist.

### 3. Create directories

```bash
mkdir -p "$repo_root/planning/session-logs" "$repo_root/planning/handovers"
touch "$repo_root/planning/session-logs/.gitkeep" "$repo_root/planning/handovers/.gitkeep"
```

### 4. Create files (only if absent)

#### `planning/leftover-tasks.md`

```markdown
# Leftover Tasks

Follow-up work surfaced mid-flow and deliberately deferred. Captured by `claude-rudder:capture-leftover`. Tackled in fresh sessions via `claude-rudder:start-leftover`.

Each entry follows this structure:

```
## YYYY-MM-DD HH:MM — <one-line summary>
- **Surfaced during:** <what we were doing>
- **Why deferred:** <reason>
- **Context:** <minimum the next session needs>
- **Estimated scope:** small | medium | large | unknown
- **Status:** open | in-progress | done
```

---

<!-- new entries appended below -->
```

#### `planning/blockers.md`

```markdown
# Blockers

Things preventing progress, ordered by urgency. Captured by `claude-rudder:log-blocker`. Reviewed at session start, cleared individually.

Each entry follows this structure:

```
## <urgency: critical | high | normal | low> — <one-line summary>
- **Logged at:** YYYY-MM-DD HH:MM
- **Affects:** <what's blocked by this>
- **Owner:** <who needs to act, or "self" / "external">
- **Notes:** <details, attempts, next options>
- **Status:** open | resolved
```

---

<!-- new entries appended below -->
```

#### `planning/README.md`

```markdown
# Planning Folder

Lightweight planning artifacts for this repo. Read by `claude-rudder` skills.

## Files

| File | Purpose | Writer skill | Reader skills |
|---|---|---|---|
| `leftover-tasks.md` | Follow-ups surfaced mid-flow, deferred | `capture-leftover` | `start-leftover` |
| `blockers.md` | Known blockers, ordered by urgency | `log-blocker` | (manual review) |
| `session-logs/` | Per-session summaries, append-only | (any) | `start-from-handover` may consult |
| `handovers/` | Frozen handover snapshots | `context-gate`, `handover`, `handover-with-tasks` | `start-from-handover` |

## Coexistence with task-queuer

If `task-queuer@danielrosehill` is also in use here, you'll see additional folders alongside (`tasks/in-queue/`, `tasks/holding/`, `tasks/done/`, `bug-reports/`, `project-decisions/`). Different system, different purpose:

- **task-queuer** = heavyweight, categorised, lifecycle-tracked queue (Trello-style)
- **claude-rudder lightweight** = parking lot for ideas + blockers + session continuity

They don't overlap by design. A "leftover task" surfaced during a session can be promoted into the task-queuer queue if it grows in scope.
```

### 5. Acknowledge

Print one line of confirmation:

```
✓ Scaffolded planning/ at <repo-root>/planning/  (created: <list of new files>; existing: <list>)
```

If nothing was created (everything already existed), say:

```
✓ planning/ already in place at <repo-root>/planning/  (no changes)
```

## Hard rules

- **Idempotent.** Never overwrite existing files. If you find a `leftover-tasks.md` with content, leave it alone.
- **No git commits.** Scaffolding is local — let the user decide when to commit.
- **No top-level `.gitignore` modifications.** The planning folder should be tracked by default; if the user wants to ignore parts of it, they decide.
- **Don't presume the variant.** If neither task-queuer nor any planning structure exists, the lightweight version is the default. The user can layer on task-queuer later.
