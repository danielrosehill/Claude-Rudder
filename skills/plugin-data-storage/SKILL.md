---
name: plugin-data-storage
description: Canonical reference for where Claude Code plugins should persist user data. Defines the CLAUDE_USER_DATA convention, resolution order, and forbidden locations. Use this skill whenever designing or updating a plugin's onboarding flow, preferences storage, or persistent-state handling — and whenever Claude is tempted to write under ~/.claude/.
---

# Plugin User-Data Storage — Canonical Reference

This skill is the authoritative rule for where Daniel-authored Claude Code plugins persist user data. Every plugin's onboarding, preferences, and state-storage logic should follow the convention defined here.

## TL;DR

**Never write plugin data under `~/.claude/`.** That directory is Claude Code's own config surface; `~/.claude/plugins/<plugin>/` in particular is the plugin install directory and is overwritten on every plugin update.

Resolve the base data directory like this:

```
${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/<plugin-name>/
```

## The `CLAUDE_USER_DATA` convention

`CLAUDE_USER_DATA` is the single env var that controls where plugin-owned persistent data lives. If the user sets it, everything relocates there (useful for backups, Dropbox sync, external drives). If unset, the resolution falls through to XDG, then to `~/.local/share/claude-plugins/`.

Resolution order, in the exact form plugins should use:

1. `$CLAUDE_USER_DATA` if set → `$CLAUDE_USER_DATA/<plugin-name>/`
2. Else `$XDG_DATA_HOME` if set → `$XDG_DATA_HOME/claude-plugins/<plugin-name>/`
3. Else default → `$HOME/.local/share/claude-plugins/<plugin-name>/`

On macOS and Windows, the XDG fallback is conventional (most tools honour it on macOS; on Windows, `%APPDATA%\claude-plugins\<plugin>\` is acceptable if XDG isn't set).

### Shell snippet (for commands that need to resolve the path)

```bash
PLUGIN_NAME="<plugin-name>"
PLUGIN_DATA_DIR="${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/$PLUGIN_NAME"
mkdir -p "$PLUGIN_DATA_DIR"
```

### Prose form (for skills and commands written in markdown)

> Resolve the plugin's data directory as `$CLAUDE_USER_DATA/<plugin-name>/` if `CLAUDE_USER_DATA` is set; otherwise `$XDG_DATA_HOME/claude-plugins/<plugin-name>/` if `XDG_DATA_HOME` is set; otherwise `~/.local/share/claude-plugins/<plugin-name>/`. Create the directory if it doesn't exist.

## Standard subdirectory layout

Inside `<plugin-data-dir>/`:

| Subpath | Purpose |
|---|---|
| `config.json` | Small pointers and settings (e.g., path to user-owned data repo) |
| `data/` | Plugin-owned user data, if the plugin is the owner |
| `cache/` | Regenerable — safe to delete |
| `state/` | Runtime state that should persist across sessions |

Not every plugin needs all four — use only what applies.

## When the user owns the data in their own location

Many plugins (Resume-Typesetter, Budgeting, Purchasing workspaces) operate on data that lives in a user-chosen location — a GitHub repo, `~/Documents/...`, or `~/repos/...`. The rule:

- **Ask the user during onboarding** where the data should live.
- Default suggestions should be under `~/Documents/` or `~/repos/` — **never** under `~/.claude/` and **never** under the plugin's data dir.
- The plugin's `config.json` (in the plugin data dir) stores the pointer to the user's chosen path.

## Legitimate reads of `~/.claude/` (allowed)

Plugins MAY read (never write) from:

- `~/.claude/CLAUDE.md` — user's global context, useful for personalising generated workspaces.
- `~/.claude/settings.json` — Claude Code settings, for MCP config inspection.
- `~/.claude/context/` — chunked global context files.

The `user-context-file` plugin is the only exception that *writes* to `~/.claude/CLAUDE.md` — that's its entire purpose.

## Forbidden patterns

| Pattern | Why forbidden |
|---|---|
| `~/.claude/plugins/<plugin>/config.json` | Install directory — overwritten on plugin update. Data loss. |
| `~/.claude/plugins/<plugin>/data/...` | Same as above. |
| `~/.claude/<plugin>-preferences.md` | Clutters Claude Code config surface; inconsistent; hard to back up. |
| `~/.claude/<plugin>/...` | Same as above. |
| `~/.claude/data/...` | Same as above. |

## Migration from legacy paths

Plugins previously wrote to forbidden locations. Onboarding flows should include a migration block:

1. Resolve the new path per the convention above.
2. Check each legacy path (`~/.claude/plugins/<plugin>/config.json`, `~/.claude/<plugin>-preferences.md`, etc.).
3. If a legacy file exists AND the new path is empty, move the file to the new location.
4. After successful move, delete the legacy file (do not leave stubs — the legacy path was never a stable home).
5. Log the migration to the user: "Moved <file> from <old> to <new>."

## Backup guidance for users

Because all plugin data lives under one root (`$CLAUDE_USER_DATA` or `~/.local/share/claude-plugins/`), users can back up every plugin's state with a single directory target. Documentation should mention this when relevant.

## Upstream context

This convention exists to work around Anthropic bug [#51342](https://github.com/anthropics/claude-code/issues/51342) — Claude Code itself suggests inconsistent paths under `~/.claude/` for plugin data, creating update-clobber risk. If Anthropic later blesses an official env var or path, this reference is the single place to update.
