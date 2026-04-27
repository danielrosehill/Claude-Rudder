---
name: context-gate
description: One-shot context-checkpoint workflow — when the current session is getting cluttered, write a structured handover plan capturing what's done and what's outstanding, then offer to spawn a fresh Claude session that opens already aware of the plan. Use when the user says "context is getting heavy", "let's checkpoint here", "stop, handover, restart", "context-gate", or otherwise signals they want to wrap the current session and continue cleanly in a new one.
---

# Context Gate

A controlled checkpoint between sessions. When context gets cluttered or you've reached a natural pause, this skill orchestrates the three-step "stop → handover → resume cleanly" workflow as a single action so you don't have to remember each step or which skill to invoke.

## Why use this

Long Claude sessions accumulate noise — exploratory tool calls, abandoned approaches, reasoning that's no longer load-bearing. A fresh session reading a tight handover plan starts faster, makes fewer mistakes, and costs less per turn than letting context drift toward the limit. This skill makes the swap a one-step action.

## What it does

1. **Pause the current session.** Stop further work. Surface anything in flight that the new session needs to know.
2. **Write a handover plan** by delegating to `handover-with-tasks` (in this same plugin). The plan captures: what the user originally asked, what's been done, what's outstanding, any decisions made, any open questions, and any pitfalls discovered. Saved to a known location so the next session can find it.
3. **Offer to spawn a fresh session** that opens with the plan as context. Default form: a new terminal at the current working directory with the plan already passed as the first message. Delegate to `new-claude-here` (also in this plugin) with the plan path baked in.
4. **Confirm the handoff.** Show the plan path and the launch command. The user just approves; they're in a clean session in seconds.

The user types `/claude-rudder:context-gate` once. The orchestration happens behind that one entry point.

## Inputs

The skill takes no required arguments. It infers what's relevant from the current conversation. Optional:

- `--reason <text>` — short note explaining why the user is checkpointing (e.g. "context cluttered after long migration debug"). Goes into the plan header.
- `--no-resume` — write the plan but don't offer to spawn a new session. User can launch later manually.
- `--plan-path <path>` — override where the plan is saved. Default per the resolver below.

## Plan path resolver

Default location, in order of preference:

1. If `cwd` is inside a git repo → `<repo-root>/planning/handovers/handover-<YYYY-MM-DD-HHMM>.md`. Auto-run `scaffold-planning` first if `planning/` is missing.
2. Else → `${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/claude-rudder/handovers/handover-<YYYY-MM-DD-HHMM>.md`.

Use ISO-8601 minute resolution so multiple handovers per day sort cleanly.

## Plan structure

Delegate authorship to `handover-with-tasks` but ensure the resulting plan has these sections (the existing skill should already produce most of them):

```markdown
# Handover — <YYYY-MM-DD HHMM>

## Why this checkpoint
<one or two lines: the reason for stopping>

## Original ask
<what the user originally requested at the top of the session>

## Done
- <bullets of completed work, with file paths or commit hashes where useful>

## Outstanding
- <bullets of remaining tasks, ordered by priority>

## Decisions made
- <key choices: why this approach, what was rejected, any user calls>

## Open questions
- <things the new session should ask the user about, if any>

## Gotchas / pitfalls
- <surprises encountered, dead ends, things the new session shouldn't waste time on>

## Pointers
- Repo: <path>
- Branch: <name>
- Last commit: <sha + msg>
- Relevant files: <list>
```

## Resume command

After writing the plan, print exactly one launch command the user can copy or approve, e.g.:

```
claude --message "Read $PLAN_PATH and continue from the Outstanding section. Ask me before doing anything destructive."
```

If the user agrees, delegate to `new-claude-here` with that message pre-filled. If they decline (`--no-resume` or "not yet"), just print the path and exit — they can resume any time.

## Confirmation flow

After the plan is written, show:

```
✓ Handover plan written: <absolute path>
  Reason: <reason or "(none)">
  Outstanding items: <count>

Spawn a new Claude session pre-loaded with this plan? [Y/n]
```

If `Y`: launch via `new-claude-here`.
If `n`: print the manual resume command and exit.

## Hard rules

- **Don't summarise speculatively.** The handover should reflect what actually happened in this session, not what Claude thinks should happen next. If a decision is uncertain, mark it as an open question rather than committing to it.
- **Don't include sensitive output.** Strip tokens, API keys, raw credential values from any tool output quoted in the plan. References by env var name or `op://` are fine.
- **One plan per gate.** Don't append to an existing plan — checkpoints are point-in-time snapshots. If the user wants a multi-day journal, that's a different workflow.
- **Don't auto-launch.** Always confirm before spawning a new session. The user might want to read the plan first.
