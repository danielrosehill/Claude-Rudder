---
name: extract-skill
description: Synthesize a new SKILL.md from the procedure that was just followed in this session, so the next session can replay it without being briefed from scratch. Use when the user says "extract this as a skill", "save this process", "make a skill from what we just did", "this should be a skill", "let's not have to explain this again next time", or after a long worked example whose procedure has clear reuse value.
---

# Extract Skill

Backwards-compile a reusable SKILL.md from a worked example. Distinct from `new-skill` (which creates a skill forward from a description) — this one **synthesizes from observed conversation**: what we actually did, why we made the calls we made, what edge cases we hit, what the hard rules turned out to be.

The product is a SKILL.md following Daniel's conventions (frontmatter with name + description-with-trigger-phrases, then a body with When-to-use, Steps, Gotchas, Hard rules, Counterparts).

## When to use

- The user just walked through a non-trivial procedure and says "let's extract this as a skill"
- Late in a long session where the user wants to bottle the process before clearing context
- After a worked example surfaces enough decision points and edge cases that re-deriving them next time would be wasteful
- When `log-session` is being run and the procedural meat is worth promoting from a one-off log entry to a reusable skill

If the procedure is fully captured by an existing skill (e.g., the user is just walking through `migrate-loose-skill` and there's nothing new), don't extract — point at the existing skill instead.

## Steps

### 1. Identify the procedure

Scan the conversation. Look for:

- A clear **goal** the user articulated (often phrased as "let's…", "I want to…", "we need to…")
- A sequence of **steps** that were taken to reach it (tool calls, file edits, decisions confirmed)
- **Decision points** where the user picked between options (these become "ask before X" rules in the skill)
- **Edge cases** that emerged (these become Gotchas)
- **Hard rules** that crystallized (things the user said "don't do that" or "always do that")

If the conversation has multiple distinct procedures interleaved, ask:

> I see [N] candidate procedures: [A], [B], [C]. Which one to extract? (or all, as separate skills)

### 2. Draft the SKILL.md

Use this skeleton. Fill from observed conversation, not from imagination:

```markdown
---
name: <kebab-case-name>
description: <one paragraph: what the skill does, when to invoke it, with the ACTUAL trigger phrases the user uses or would naturally use — drawn from the conversation, not invented>
---

# <Title Case Name>

<One-paragraph framing: what the skill does and why it exists. Distinguish from any neighboring skills.>

## When to use

- <bullet — concrete trigger condition from the conversation>
- <bullet — another>
- <skip / negation rule if relevant: "don't use this when X — use Y instead">

## Steps

### 1. <action verb + object>

<concrete what-to-do, with code blocks where applicable>

### 2. <next>

…

(As many steps as the procedure had. Don't pad — but don't compress past the point where the next session could re-derive what to do.)

## Gotchas

<bugs / surprises / dead-ends from the actual session — each as a numbered item with the symptom and the fix or workaround>

1. **<symptom>** — <what causes it, what to do instead>

## Hard rules

<crystallized "always do X" / "never do Y" — drawn from user corrections or explicit statements during the session>

- **<rule>**
- **<rule>**

## Counterparts

<related skills / files / tools the procedure leans on. Both within this plugin and across plugins.>

- `<plugin>:<skill>` — <how it relates>
```

### 3. Decide where it goes

Ask the user:

> Which plugin should this skill live in?
>
> - **`claude-rudder@danielrosehill`** — if the procedure is about session UX, planning, or Claude Code itself
> - **<domain plugin>** — if the procedure is domain-specific (e.g., transcription work → `claude-transcription`, op-vault work → `op-vault`, etc.)
> - **In the project repo** at `<repo>/.claude/skills/<name>/SKILL.md` — if the procedure is specific to one repo and shouldn't be globally available

Suggest a default based on the procedure's domain, but **don't auto-pick**. The user often has a preference about scope.

### 4. Show the draft

Print the full draft SKILL.md to chat. Don't write it yet — let the user review.

> Draft of `<name>` SKILL.md (target: `<plugin>` or `<repo path>`):
>
> ```markdown
> [full draft]
> ```
>
> Ready to write? (yes / edit / wrong target / cancel)

### 5. Write and (optionally) push

If approved:

- **For a plugin target:**
  1. Create `<plugin-source-repo>/skills/<name>/SKILL.md`
  2. Bump plugin.json version (patch-bump unless user says otherwise)
  3. Commit with message: `Add <name> skill (extracted from session)`
  4. Push
  5. Update marketplace.json version if applicable
  6. Refresh marketplace cache and update the plugin

- **For a repo-local target:**
  1. Create `<repo>/.claude/skills/<name>/SKILL.md`
  2. Commit + push if the repo wants it tracked (ask once)

### 6. Confirm

```
✓ Skill written: <path>
  Target: <plugin name>@<marketplace> or <repo>
  Version bumped: <old> → <new>  (only for plugin targets)
  Commit: <sha>
```

## What makes a good extract

- **Trigger phrases come from the conversation**, not generic templates. If the user said "park this for later", that becomes a trigger phrase. If you've never heard them say "deferred follow-up capture", don't put that in the description.
- **Steps reference real tools/commands.** If the procedure involved `jq` and `gh`, name them. If it involved a specific MCP server or another skill, reference it.
- **Gotchas are real**, not anticipated. Only list problems that actually occurred or were near-misses in the conversation. Don't invent edge cases.
- **Hard rules are observed**, not boilerplate. "Always sanitize before push" is a hard rule because we hit a near-miss with it. "Never include emojis" isn't relevant unless it came up.
- **Counterparts are concrete.** Name actual skills and files, not vague categories.

## What NOT to do

- **Don't fabricate procedure.** If a step wasn't actually taken in the conversation, don't add it. The point of extracting is fidelity to the worked example.
- **Don't generalize away the specifics.** "Run the relevant command" is useless. "Run `jq '.version=\"X\"' file > tmp && mv tmp file` to update the version" is useful. Specific examples > abstract guidance.
- **Don't extract trivial procedures.** A two-step thing like "read a file and report what's in it" doesn't merit a skill. If you can't write at least 4-5 distinct steps with decisions and gotchas, the procedure isn't substantial enough yet.
- **Don't auto-write without showing the draft.** The user has taste about how their skills read; they need to review.
- **Don't write to disk before the version bump and commit are queued up too.** Doing the file write separately from the rest leads to half-state if interrupted.
- **Don't pick the target plugin without asking.** Domain routing is judgment, not inference.

## Self-application

This skill can be applied to itself: if you find yourself walking through how to extract a skill (e.g., handling a corner case the SKILL.md above doesn't cover), invoke `extract-skill` on that conversation to update *this* file. The loop terminates because eventually the procedure stabilises.

## Counterparts

- `new-skill` — forward-design from a description (different input)
- `log-session` — captures the whole session as a journal entry; this skill captures one *procedure* from it as a reusable thing
- `claude-rudder:plugin-data-storage` — convention reference for any extracted skill that needs to persist user data
- `references/handover-template.md` — same general "single source of truth" pattern at the schema level
