---
name: context-workflow-reference
description: Reference and worked example for the CONTEXT.md / CLAUDE.md two-file workflow. Use when the user asks "what is CONTEXT.md", how the human-context vs agent-briefing split works, or wants a copyable example pair to seed a new repo. Pairs with the operational skills (context-to-claude, claude-to-context, add-to-context, remove-from-context, manual-context-update, chunk-repo-context) — this skill explains the model; those skills do the work.
---

# CONTEXT.md / CLAUDE.md Workflow Reference

Two-file pattern for managing project context in Claude Code. This skill is reference material — it documents the model and ships a worked example. For operations on these files, use the dedicated skills (listed at the end).

## The Two-File Model

Humans express intent best in conversational, narrative form. Agents work best on structured, concise briefings. LLMs are very good at converting between them. The pattern leans into that asymmetry.

**`CONTEXT.md`** — human-authored scratchpad
- Conversational, exploratory, voice-friendly
- Includes the *why* behind decisions
- Captures open questions, alternatives considered, gut feelings
- Often grows from voice dictation with minimal cleanup
- Not read by Claude Code automatically; referenced from `CLAUDE.md`

**`CLAUDE.md`** — agent-ready briefing
- Structured sections: overview, requirements, constraints, implementation guidance
- Concise, actionable, decision-extracted
- Auto-loaded by Claude Code at session start
- Should reference `CONTEXT.md` for deeper background

The two files are kept in sync by deliberate sync points, not by trying to write a single document that serves both audiences.

## When to Use This Pattern

Good fit:
- Solo projects where the user thinks out loud (voice transcription, late-night idea dumps)
- Pre-formal projects where requirements are still being discovered
- Projects where future-you needs to recall *why* a decision was made, not just what it was
- Any repo where the user prefers to write naturally and have Claude reshape it

Skip this pattern when:
- The repo already has a strong README or design doc that fills the briefing role
- Project context is genuinely simple — a 30-line `CLAUDE.md` is fine alone
- A team workflow uses formal RFCs / ADRs instead

## Optional: Chunked Context

When `CONTEXT.md` grows past ~500–1000 lines, split it into `context-data/` with topic files and a README index. `CLAUDE.md` then references `context-data/` instead of (or alongside) `CONTEXT.md`. The `chunk-repo-context` skill handles the split.

## Worked Example

`examples/web-app/` in this skill ships a paired `CONTEXT.md` + `CLAUDE.md` for a hypothetical reading-tracker web app, plus a `README.md` annotating what to notice in each file.

To seed a new repo from it:

```bash
SKILL_DIR=$(dirname "$0")  # or wherever this skill is mounted
cp "$SKILL_DIR/examples/web-app/CONTEXT.md" ./CONTEXT.md
cp "$SKILL_DIR/examples/web-app/CLAUDE.md" ./CLAUDE.md
# then edit both to match the actual project
```

Read both files side-by-side first — the point is to internalize the difference in voice and structure between the two, not to keep the reading-tracker content.

## Hooks That Support This Workflow

`rudder` ships two hooks (in `hooks/hooks.json`) that reinforce the pattern automatically:

- **SessionStart**: detects `CONTEXT.md` or `context-data/` and reminds the agent they exist
- **PostToolUse (Write|Edit)**: when a context file is touched, prompts a reminder to keep the pair in sync

No setup required — they activate when the plugin is installed.

## Companion Skills (Operations)

| Need | Skill |
|---|---|
| Generate `CLAUDE.md` from `CONTEXT.md` | `context-to-claude` |
| Pull info from `CLAUDE.md` back into `CONTEXT.md` | `claude-to-context` |
| Add a new piece of context cleanly | `add-to-context` |
| Remove outdated/wrong context | `remove-from-context` |
| Guided coordinated manual edit of both files | `manual-context-update` |
| Split a large `CONTEXT.md` into `context-data/` | `chunk-repo-context` |
| Run a structured audit on existing context files | `context-health-check` |
| Prune a bloated `CLAUDE.md` to essentials | `chunk-claude` / `chunk-claude-md` |

## Best Practices

**Writing `CONTEXT.md`:**
- Write the way you'd talk through the project with a friend
- Keep the *why* — it's the part that's hardest to reconstruct later
- Don't self-edit aggressively; let the conversion step do that
- Voice transcription with light cleanup is a first-class input

**Maintaining `CLAUDE.md`:**
- Treat as derived, not authoritative; the truth lives in `CONTEXT.md`
- Keep sections short and action-oriented
- Always include a pointer back to `CONTEXT.md` so the agent knows deeper context exists
- Re-derive (don't hand-edit) when `CONTEXT.md` shifts substantially

**Sync cadence:**
- After a brain-dump session in `CONTEXT.md` → run `/context-to-claude`
- After Claude refines requirements during work → run `/claude-to-context` to capture them in human form
- When something is wrong/stale → `/remove-from-context` rather than letting it rot

## Provenance

This skill folds in the conceptual content of the standalone `Claude-Code-Context-Toolkit` repo. The operational commands from that repo were already present in `rudder`; what's preserved here is the workflow articulation and the worked example.
