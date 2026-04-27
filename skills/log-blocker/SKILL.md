---
name: log-blocker
description: Append a known blocker to `<repo>/planning/blockers.md` (auto-scaffolds if missing). Use when something is preventing progress and shouldn't be lost — waiting on a code review, an external decision, an upstream bug, a credential not yet provisioned, a flaky test that needs investigation. Triggers on "log a blocker", "we're blocked on X", "note this blocker", "add to blockers".
---

# Log Blocker

Capture an obstacle to progress. Lighter-weight than a leftover task: blockers are things you can't move past without external action, not things you chose to defer.

## Difference vs `capture-leftover`

- **leftover** = a follow-up the user / Claude *chose* not to do now, but could
- **blocker** = something the user / Claude *can't* do now without a dependency clearing

If the user says "let's circle back to that," it's a leftover. If they say "we're stuck on that until X happens," it's a blocker.

## Steps

### 1. Resolve target

```bash
repo_root=$(git rev-parse --show-toplevel 2>/dev/null) || repo_root="$PWD"
target="$repo_root/planning/blockers.md"
```

Auto-run `scaffold-planning` if missing.

### 2. Gather inputs

Ask (or infer):

- **Summary** — one line, what's blocked
- **Urgency** — `critical` (blocks shipping), `high` (blocks today's work), `normal` (blocks this sprint), `low` (annoyance, not urgent)
- **Affects** — what's blocked: a feature, a release, a specific task, the whole repo?
- **Owner** — who needs to act? `self` (waiting on something only the user can do), `external` (waiting on someone else — name them if known)
- **Notes** — what's been tried, what the next options are, any links/refs

Default urgency = `normal` if not specified.

### 3. Format and append

```markdown
## high — Vercel build failing on edge runtime since dependency bump
- **Logged at:** 2026-04-27 18:55
- **Affects:** new-blog-post deploy pipeline; can't ship until resolved
- **Owner:** self (need to investigate the @vercel/edge change)
- **Notes:** suspect bumping @vercel/og from 0.5 → 0.6 broke edge import. Tried pinning back, didn't help. Next: check Vercel changelog, open issue if confirmed bug.
- **Status:** open
```

Append after the `<!-- new entries appended below -->` marker. Group by urgency only as a presentation concern when reading; the file stays in append order.

### 4. Confirm briefly

```
✓ Logged blocker: "Vercel build failing on edge runtime..." (high) → planning/blockers.md (2 open total)
```

## Closing a blocker

When a blocker is resolved, change `**Status:** open` → `**Status:** resolved` and append:

```
- **Resolved:** 2026-04-28 10:15 — Vercel pushed a fix in @vercel/og 0.6.1
```

This skill writes new blockers; resolution is typically a manual edit or done by whichever skill confirms the unblock.

## Hard rules

- **Append-only for new blockers.** Don't reorder existing entries.
- **Don't auto-resolve.** Even if the user says "I think that's fixed," confirm before flipping status.
- **Don't escalate urgency without asking.** If the user says "blocker" without naming urgency, default `normal` rather than guessing `critical`.
- **No git commits.** Same as the other planning skills.

## Counterparts

- `scaffold-planning` — creates `blockers.md`
- `capture-leftover` — for deferred-by-choice follow-ups (different concept)
