---
name: new-claude-here
description: Open a new Konsole window at the current working directory running a fresh Claude Code CLI session. Use when the user asks for a "new claude session here", "claude here", "jump to a new claude", or "new claude window".
---

# New Claude Session Here

Spawns a sibling Claude Code session in a new Konsole window at the current `$PWD`. Equivalent to the shell one-liner `konsole --workdir "$PWD" -e claude`.

## Why this lives in the session-transfer plugin

The common reason to jump to a fresh Claude session mid-task is that the current session can't pick up changes without a restart — most often after installing a new MCP server, editing settings, or hitting a stuck state. In those cases the user wants session continuity without losing their place. This skill is the "quick jump" path; `session-transfer:write-handover` / `session-transfer:start-from-handover` is the "pipe the full context across" path. Offer both when relevant.

## Command

```bash
setsid konsole --workdir "$PWD" -e claude >/dev/null 2>&1 &
```

`setsid` + background detaches the new Konsole so it keeps running after the current Claude Code session exits.

## Notes

- The new session is standalone — no shared memory with this one. If the user needs context carried over, suggest running `session-transfer:write-handover` first.
- Report the directory the new session was launched in.
