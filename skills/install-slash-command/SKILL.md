---
name: install-slash-command
description: Use when the user wants to install a new user-level slash command at ~/.claude/commands/. Triggers on phrases like "install a slash command", "add a /command", "create a user-level slash command", "save this as a slash command".
---

The user wants to install a new slash command at the user level so it's available across all Claude Code sessions on this machine.

User-level commands live in `~/.claude/commands/` as `.md` files. The filename (without `.md`) becomes the invocation, e.g. `~/.claude/commands/foo.md` → `/foo`. Subfolders namespace commands, e.g. `~/.claude/commands/git/sync.md` → `/git:sync`.

## Step 1: Gather inputs

You need:
1. **Name** — kebab-case, no spaces, no leading slash. Optionally namespaced as `namespace/name`.
2. **Description** — one short line. Shown in the `/` picker and used to decide relevance.
3. **Prompt body** — the instructions Claude will follow when the command fires.
4. *(optional)* **argument-hint** — placeholder shown after the command name, e.g. `[file]` or `[pr-number]`.
5. *(optional)* **allowed-tools** — restrict tool access, e.g. `Read, Grep, Bash`.
6. *(optional)* **model** — override default, e.g. `haiku` for cheap/fast commands.

If the user gave you source material (a message, file, URL, existing prompt), convert it into a clear prompt body. If anything essential is missing (especially name or description), ask once before writing.

## Step 2: Validate

- Name: `^[a-z0-9-]+(/[a-z0-9-]+)?$` — kebab-case, optional one level of namespacing.
- Check for collision: `ls ~/.claude/commands/<name>.md` — if it exists, confirm with the user before overwriting.
- If namespaced, ensure the subfolder exists (create if not).

## Step 3: Write the file

Format:

```markdown
---
description: {{one-line description}}
argument-hint: {{optional — omit field if not used}}
allowed-tools: {{optional — omit field if not used}}
model: {{optional — omit field if not used}}
---

{{prompt body}}

{{reference $ARGUMENTS if the command takes input}}
```

Only include optional frontmatter fields the user actually wants — don't emit empty ones.

The prompt body should be written in second person addressing Claude ("You are...", "Read the file at..."), same style as a system prompt. If the command takes runtime input, reference it via `$ARGUMENTS` (or `$1`, `$2`, ... for positional args).

## Step 4: Verify

- `ls -la ~/.claude/commands/<name>.md` to confirm the file exists.
- Report the invocation string to the user (e.g. "Installed — use `/foo` or `/foo some-arg`").
- Mention they may need to restart the Claude Code session or run `/reload` for the command to appear in the picker (new sessions pick it up automatically).

## Notes

- Do NOT register the command anywhere else — dropping the file in `~/.claude/commands/` is the entire install.
- Do NOT confuse this with **skills** (which live in `~/.claude/skills/<name>/SKILL.md` and have richer metadata). If the user's request sounds like it wants autonomous triggering on certain phrases, suggest a skill instead — slash commands only fire when explicitly invoked with `/`.
- Do NOT add this to any plugin marketplace or index — it's a local user-level file.
