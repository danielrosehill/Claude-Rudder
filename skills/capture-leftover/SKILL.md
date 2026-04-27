---
name: capture-leftover
description: Capture a follow-up task that surfaced mid-flow but shouldn't distract from the current focus. Appends to `<repo>/planning/leftover-tasks.md` (auto-scaffolds if missing). Use when the user says "park this", "leftover task", "note this for later", "follow-up but not now", "don't let me forget about X", or when Claude and the user mutually agree to defer something rather than context-switch.
---

# Capture Leftover Task

Append a deferred follow-up to the lightweight planning structure without leaving the current task. Designed to take seconds — minimal interruption, maximum recall later.

## When to use

- The user explicitly defers something: "let's not do that now, but remember it"
- Claude proposes something tangential and the user says "noted, but later"
- Mid-debug, a separate issue surfaces — capture it before forgetting, but stay on the original
- Mutual "let's park this" moments

If the follow-up is **in-scope of the current task**, just add it to the working TODO. This skill is for things that would otherwise derail the session.

## Steps

### 1. Resolve target file

```bash
repo_root=$(git rev-parse --show-toplevel 2>/dev/null) || repo_root="$PWD"
target="$repo_root/planning/leftover-tasks.md"
```

If `target` doesn't exist, run `scaffold-planning` first (silently — don't pause to ask). After scaffolding, `target` will exist with a header and template.

If `cwd` is not in a git repo and the user didn't specify a target, ask:

> Capture this as a leftover task in: (1) cwd's `planning/leftover-tasks.md`, (2) global `$CLAUDE_USER_DATA/claude-rudder/leftover-tasks.md`, or (3) somewhere else?

Default to (1) but confirm.

### 2. Gather inputs

In one short prompt or by inferring from context, get:

- **Summary** — one line, ≤80 chars (e.g. "Refactor auth middleware to remove the legacy session-token path")
- **Surfaced during** — what the current session is doing (Claude usually knows this from context — propose it, let user override)
- **Why deferred** — short reason. Default: "would distract from current focus" if user doesn't elaborate
- **Context** — the minimum a future session needs to pick this up. Often a sentence or two; sometimes a code path, file name, or external link
- **Estimated scope** — `small`, `medium`, `large`, or `unknown`. Default `unknown` if user doesn't say

Don't over-interrogate. If the user has only given a one-liner, fill the rest with reasonable inferences and a `Context: <propose>` that they can confirm or revise.

### 3. Format and append

Build the entry:

```markdown
## 2026-04-27 18:53 — Refactor auth middleware to drop legacy session-token path
- **Surfaced during:** debugging the new login flow in src/auth/login.ts
- **Why deferred:** would derail the current bug fix — pure tech-debt
- **Context:** see `src/auth/legacy-session.ts`; new flow already in place but old code still referenced. ~200 LOC removal.
- **Estimated scope:** medium
- **Status:** open
```

Append to `target` after the existing `<!-- new entries appended below -->` marker (or just at end of file). Each entry is separated from the next by a single blank line.

Use ISO-8601 minute resolution for the timestamp, in the local timezone. Example: `2026-04-27 18:53` (no seconds, no timezone — keep it readable; the marker is the absent timezone).

### 4. Confirm briefly

Print one line:

```
✓ Captured: "Refactor auth middleware..." → planning/leftover-tasks.md (3 open total)
```

The "(N open total)" tally is helpful — it lets the user see the parking lot is filling up and consider running `start-leftover` soon.

### 5. Return to the original task

Don't elaborate. Don't summarise the captured task back at length. The whole point is **minimum interruption** — get back to what we were doing.

## Bulk mode

If the user provides multiple follow-ups in one go ("park these three: …, …, …"), handle as a loop:

- Confirm the list once before writing
- Append all entries in order, same timestamp ± 1 minute apart, or just same timestamp
- Print a summary: "✓ Captured 3 leftovers → planning/leftover-tasks.md (5 open total)"

## Hard rules

- **Don't switch tasks.** This skill is non-disruptive by design. Capture and return.
- **Don't ask more than 1-2 follow-up questions.** If context is missing, fill with reasonable inferences and let the user revise the file later.
- **Append-only.** Never reorder or rewrite existing entries.
- **No git commits.** Let the user decide when to commit the planning folder.

## Counterparts

- `start-leftover` — picks up an entry from this file in a fresh session
- `scaffold-planning` — creates the file and surrounding structure
- `log-blocker` — for things blocking progress (different category)
- `leftover-aggregator` agent — proactively scans conversation for missed defer-moments
