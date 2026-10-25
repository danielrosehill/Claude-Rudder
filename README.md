# meta-tools

Claude Code plugin: meta-tooling workflow — context-break evaluation, CLAUDE.md chunking, MCP command generation, primitive selection, and claude-code feedback filing, with context-management / mcp-integration / feedback-filing / primitive-selection variants.

Part of the [danielrosehill Claude Code marketplace](https://github.com/danielrosehill/Claude-Code-Plugins).

This plugin is **primitive-only** — its value is the set of commands and skills it makes globally available. The `new-workspace` skill provisions a minimal scaffold only for users who want a repo to collect meta-tooling notes.

## What you get

### Commands (`/meta-tools:<name>`)

**Context management** (from `claudemd-chunker` + `new-turn-hook`):

- `chunk-claude` — prune a bloated `CLAUDE.md` and offload supplementary context to `agent-context/`.
- `should-i-start-fresh` — evaluate whether the current conversation has accumulated bloat.
- `context-health-check` — audit the current conversation's context load.

**MCP integration** (from `mcp-command-generator`):

- `add-mcp-server` — generate a valid `claude mcp add` command from plain English.
- `list-mcp-servers` — show configured MCP servers.
- `mcp-command-help` — syntax help for the MCP CLI.

### Skills (auto-invoked)

- `context-break-evaluator` — structured criteria for context-break decisions.
- `document-as-you-go` — sweep the session for durable findings and undocumented changes and file them (canonical text: `danielrosehill/Document-As-You-Go`).
- `one-handed` — the user is typing one-handed; infer intent from terse or incomplete prompts.
- `mcp-syntax-reference` — full `claude mcp add` syntax reference (HTTP / SSE / stdio, scopes, auth).
- `what-thing-pick` / `what-thing-help` / `what-thing-set-context` / `what-thing-refresh-docs` — choose the right Claude Code primitive (skill, subagent, command, plugin, MCP, hook, output style).
- `feedback-bug` / `feedback-feature-request` / `feedback-model-behavior` / `feedback-docs` / `feedback-refresh-templates` — file well-formed issues against `anthropics/claude-code` using live issue-form templates.

### Provisioning skill

- `/meta-tools:new-workspace <name> [--variant=context-management|mcp-integration|feedback-filing|primitive-selection|generic] [--local-only] [--private]`

Creates a thin note-taking scaffold. The primitives above remain global and do not require this workspace.

## Variants

Variants only flavor the generated `CLAUDE.md` header — the on-disk layout is a single shared `generic/` scaffold.

- `context-management` — emphasises chunker + context-break primitives.
- `mcp-integration` — emphasises MCP command generator + syntax reference.
- `feedback-filing` — emphasises the five feedback skills.
- `primitive-selection` — emphasises the what-thing skills.
- `generic` (default) — lists all primitives.

## Pattern

Primitives live in the plugin → globally available from any cwd.
The scaffold is **data** → no `.claude/` tree inside provisioned workspaces.
Plugin updates never touch workspace data.

See [PLAN.md in Claude-Workspace-Reshaping-190426](https://github.com/danielrosehill/Claude-Workspace-Reshaping-190426) for the full pattern spec.

## Install

Via the danielrosehill marketplace:

```
/plugin marketplace add danielrosehill/Claude-Code-Plugins
/plugin install meta-tools
```

## Absorbed plugins

This plugin consolidates five previously-standalone plugins:

- `new-turn-hook`
- `claudemd-chunker`
- `mcp-command-generator`
- `what-thing`
- `feedback-filer`

## License

MIT.
