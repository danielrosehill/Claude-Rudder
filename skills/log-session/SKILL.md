---
name: log-session
description: Write a comprehensive point-in-time session log into `<repo>/planning/session-logs/YYYY-MM/YYYY-MM-DD-<slug>.md` for archival rather than immediate continuation. Captures everything achieved, the current overall state, decisions, surprises, and follow-ups for the distant future. Use when the user says "log this session", "wrap up with a record", "write today's session out", "record what we did", "we may not come back to this for a while — capture it", "session log", or otherwise signals they want a journal entry rather than a handover.
---

# Log Session

A point-in-time archival record of a working session. Distinct from `handover` and `context-gate` — those are for **immediate continuation** ("next session picks this up soon"). This skill is for **eventual return** ("I might not be back to this for weeks; capture everything").

## When to use this vs handover

| You're about to... | Use |
|---|---|
| Start a fresh Claude session in the next few hours/days to continue | `context-gate` / `handover` / `handover-with-tasks` |
| Step away from this work and may not revisit for weeks/months | **`log-session`** |
| Wrap up a milestone that's "done for now" — proud of progress, no immediate next step | **`log-session`** |
| Park work indefinitely but want a complete record | **`log-session`** |

If the user is uncertain ("I'm not sure when I'll come back"), default to `log-session` — it's a superset of a handover (more context, written for the cold reader). They can layer on `context-gate` if a near-term session is also expected.

## Output location

```
<repo-root>/planning/session-logs/YYYY-MM/YYYY-MM-DD-<slug>.md
```

- Year-month subfolder (e.g. `2026-04/`) for navigability — sessions group naturally by month
- Filename starts with the full date so a `ls` reads chronologically
- `<slug>` is a short kebab-case summary of the session (e.g. `loose-skills-migration`, `auth-refactor`, `viz-dashboard-prototype`)

If the user is outside a git repo, ask where to save. Default offer:

```
${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/claude-rudder/session-logs/YYYY-MM/YYYY-MM-DD-<slug>.md
```

If `<repo-root>/planning/` is missing, run `scaffold-planning` first (silently).

## Document format

Use the canonical structure below. Frontmatter is YAML; body is markdown.

```markdown
---
schema: claude-rudder/session-log/v1
session_started_at: <ISO-8601 with timezone, if known — else null>
session_ended_at: <ISO-8601 with timezone — when this log was written>
agent_model: <e.g. claude-opus-4-7>
agent_harness: claude-code
repo: <absolute path>
branch: <branch name at session end>
final_commit: <short sha at session end>
followups_count: <count of items written to leftover-tasks.md during this session, or 0>
followups_file: planning/leftover-tasks.md  # only if followups_count > 0
related_handover: planning/handovers/handover-<ts>.md  # only if a handover was also written
slug: <slug used in filename>
title: <human-readable title>
---

# Session Log — <YYYY-MM-DD> — <Title>

## TL;DR

<2–3 sentences. What was the session about, what was achieved, where it ended. Written for someone reading this 6 months from now with no other context.>

## What we did today

<Chronological-ish list of accomplishments. Be specific — include file paths, commit hashes, plugin names, version bumps. This is the meat of the record.>

- <bullet — completed work item with concrete reference>
- <bullet — another, with file path or sha>
- <...>

## Where we got to overall

<Not just "today's work" but the broader state of the project / initiative / migration. The reader needs to understand the cumulative position, not just incremental progress. Multiple paragraphs is fine.>

## What's left for the future

<Tasks deferred to later, with enough context to pick up cold. If items were already captured in `leftover-tasks.md`, summarise them here AND link.>

- <task — with brief why-it-matters and enough context to start cold>
- <task — same>

If anything urgent must happen before resuming this work (decisions, external dependencies, prerequisite changes), surface it here.

## Decisions and rationale

<Key choices made during the session. Explain why one path over another — the cold reader needs to understand the reasoning, not just the outcome. Include rejected options.>

- **<decision>:** <why we chose it / what we rejected>
- <...>

## What we learned / surprises

<Things worth remembering even if we never come back. Gotchas, dead ends, "this thing turns out to work differently than expected", references that were unexpectedly useful.>

## Pointers

- **Repo:** <absolute path>
- **Branch:** <name>
- **Last commit:** <sha + first line of message>
- **Files touched this session:** <list, condensed if long>
- **External references:** <links, docs consulted, related issues>
- **Related session logs:** <link any prior logs in this thread, by relative path>

## (Optional) Resume guidance

<Only include if there's a non-obvious starting point for resuming. e.g., "Start by re-reading `migration-plan-v2.md`; the next batch is the new public mgmt plugins." Skip this section if the leftover tasks are self-explanatory.>
```

## Steps

### 1. Confirm scope and slug

Before writing, ask the user:

> Title for this log? (a short phrase — I'll slugify for the filename)

If they're terse or want you to pick, propose one based on session content and confirm.

### 2. Resolve path

```bash
repo_root=$(git rev-parse --show-toplevel 2>/dev/null) || repo_root="$PWD"
date_part=$(date +%Y-%m-%d)
month_part=$(date +%Y-%m)
slug=<from-user-or-inferred>
target="$repo_root/planning/session-logs/$month_part/$date_part-$slug.md"
mkdir -p "$(dirname "$target")"
```

If `<repo-root>/planning/` is absent, run `scaffold-planning` first.

If `target` already exists (rare — same date, same slug), append a numeric suffix: `-2`, `-3`, etc.

### 3. Gather session content

Reconstruct from conversation memory:

- **What was achieved** — survey the actual conversation. Don't rely on a single TaskCreate list; that misses freeform work.
- **Decisions** — moments where the user picked an option (renames, architecture choices, scope decisions). These are typically multi-turn discussions, not single facts.
- **Files touched** — track from tool calls (Read/Edit/Write/Bash with file paths) plus git activity if visible.
- **Surprises** — things that pivoted the plan, dead ends abandoned, unexpected findings.
- **Followups** — count entries written to `planning/leftover-tasks.md` during this session if any. The frontmatter records the count and the file linkage.

### 4. Write the file

Write the full document. Use the user's preferred date format if you've recorded one (Daniel's is DD/MM/YY in display, but filename uses ISO YYYY-MM-DD for sortability).

### 5. Optionally pair with a handover

After writing, ask once:

> Want to also write a handover (`context-gate`) in case you do come back to this in the next few days?

- If yes → invoke `context-gate`, then update this log's frontmatter `related_handover` to point at the handover file.
- If no (default expectation) → done.

### 6. Confirm

Print:

```
✓ Session logged: <relative path from repo root>
  Title: <title>
  Achievements: <count>
  Followups captured: <count>
  Decisions: <count>
  <if related handover: "Paired handover: <path>">
```

## Difference from handover (formal)

| Aspect | `handover` | `log-session` |
|---|---|---|
| Audience | Next session, days away max | Future-self, weeks-to-months out |
| Length | Tight — Outstanding section is the focus | Comprehensive — full record is the point |
| `Done` section | Cumulative work in current session | Same, but also "Where we got to overall" frames the broader state |
| `Outstanding` | Active work to resume | "What's left for the future" — colder, more contextual |
| `Decisions` | Briefly mentioned | Explained with rationale and rejected options |
| Filename | `handover-<timestamp>.md` | `<date>-<slug>.md` (date-first for chronological browsing) |
| Folder | `planning/handovers/` | `planning/session-logs/<YYYY-MM>/` |

## Hard rules

- **Don't conflate today's work with overall state.** "What we did today" is incremental; "Where we got to overall" is cumulative. Both matter for the cold reader.
- **Don't be terse for terseness's sake.** This is the one document where verbosity earns its keep — the future-self reader has lost all the context.
- **Don't omit rejected options.** A decision without rejected alternatives is just an outcome; the reader needs to understand the choice.
- **Don't auto-pair with a handover.** Default = log only. Pair only on explicit user yes.
- **No git commits.** Let the user decide when to commit the planning folder.
- **Don't include raw credentials in any quoted output.** Same as handover rules.

## Counterparts

- `handover` / `handover-with-tasks` / `context-gate` — for immediate-continuation use
- `scaffold-planning` — creates `planning/session-logs/`
- `capture-leftover` — fine-grained mid-session capture; this skill rolls up the cumulative session
- `references/handover-template.md` — companion format spec for handovers (this skill has its own structure above; same `claude-rudder/<type>/v<n>` schema convention)
