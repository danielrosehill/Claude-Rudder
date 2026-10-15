---
name: install-from-github
description: Use when the user points at a GitHub repo (an agent skill, MCP server, or "Claude plugin"-ish project) and wants to install or register it. Routes between native plugin install, MCP server registration, and adding to the user's third-party marketplace. Triggers on phrases like "install this from GitHub", "add this skill to my setup", "register this as a third-party plugin", "is this available as a Claude plugin".
---

The user has found something on GitHub — a skill, an MCP server, a "Claude plugin" — and wants it integrated into their Claude Code setup **without leaving loose, untracked artefacts on disk**. The goal of this skill is to be a router: figure out what the project actually is, and pick the cleanest install path so the result is reproducible across machines.

Guiding principle: **everything the user installs should be reachable through a server-controlled surface** (the Skill-Substrate, when available; otherwise a marketplace they control), so a fresh machine can be reconstituted by reconnecting to the substrate or re-adding marketplaces. Loose skills dropped into `~/.claude/skills/` or one-off MCP entries scattered across configs are the failure mode this skill exists to prevent.

## Routing precedence (highest to lowest)

When a repo classifies as `claude-plugin` or `claude-marketplace`:

1. **Skill-Substrate** — if the user runs the substrate (i.e. has the `Skill-Substrate-Admin` plugin installed and the substrate is reachable), register the repo as a marketplace there. Subscriptions are server-side, so the install is reproducible across all the user's hosts without per-host bookkeeping. This is the default for Daniel.
2. **User's primary marketplace** — if the upstream is the user's own work and a primary marketplace exists.
3. **User's third-party marketplace** — for upstream by others, fork-and-wrap.
4. **Direct GitHub install** — last resort, only for one-off testing.

Detect substrate availability by checking for `~/.claude/plugins/Skill-Substrate-Admin/` or by attempting `ssh residencehome "docker ps --filter name=skill-substrate --format '{{.Names}}'"` (returns non-empty on substrate hosts). Skip the substrate path silently if neither indicator is present.

## Step 0: Onboarding — third-party marketplace pointer

This skill assumes the user has a dedicated **third-party marketplace** repo: a marketplace whose entries are wrappers around upstream projects authored by other people, kept distinct from the user's own original plugins.

Read config from:

```
${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/claude-rudder/config.json
```

Expected shape:

```json
{
  "third_party_marketplace": {
    "repo": "<owner>/<repo-name>",
    "local_path": "<absolute path to local clone>"
  },
  "primary_marketplace": {
    "repo": "<owner>/<repo-name>",
    "local_path": "<absolute path to local clone>"
  }
}
```

If `third_party_marketplace` is missing:

1. Tell the user this skill needs a designated third-party marketplace and explain why (portability — wrapped third-party skills live in one place, can be re-added on another machine via `claude plugins marketplace add`).
2. Ask whether they want to (a) point at an existing repo, or (b) bootstrap a new one.
3. If (b), scaffold a minimal marketplace repo: `.claude-plugin/marketplace.json` with empty `plugins: []`, a README explaining "this marketplace contains wrappers around third-party Claude Code skills and MCP servers", MIT license. Push to GitHub. Suggested name: `Claude-Third-Party-Plugins`.
4. Write the path back to `config.json`. Create the file (and its parent dir) if missing.

Never proceed past this step without a populated `third_party_marketplace.repo`.

## Step 1: Inspect the input

The user will give you a GitHub URL or `owner/repo`. Fetch enough to classify:

```bash
gh repo view <owner>/<repo> --json name,description,defaultBranchRef,url
gh api repos/<owner>/<repo>/contents -q '.[].name'
```

Then pull the files that disambiguate:

- `.claude-plugin/plugin.json` — it's a Claude Code plugin
- `.claude-plugin/marketplace.json` — it's a marketplace (multiple plugins inside)
- `SKILL.md` at root, or `skills/*/SKILL.md` without a `plugin.json` — loose skill(s)
- `package.json` with a `bin` field, or README mentioning `npx`, `mcpServers`, `claude_desktop_config.json` — MCP server
- `pyproject.toml` / `setup.py` exposing a CLI that speaks MCP — Python MCP server
- None of the above — out of scope; tell the user and stop

Use `gh api repos/<owner>/<repo>/contents/<path>` or `gh api repos/<owner>/<repo>/readme` to read files without cloning.

Record what you found in a short summary you'll show the user before acting.

## Step 2: Classify

Pick exactly one of:

| Class | Signal |
|---|---|
| `claude-plugin` | Has `.claude-plugin/plugin.json` |
| `claude-marketplace` | Has `.claude-plugin/marketplace.json` |
| `loose-skill` | Has `SKILL.md` or `skills/*/SKILL.md` but no `plugin.json` |
| `mcp-server` | NPX/Python package documented for MCP, no Claude plugin manifest |
| `other` | None of the above (CLI tool, library, demo) |

If the repo is **mixed** (e.g. an MCP server that also ships an example skill), prefer the most installable surface — usually `mcp-server` — and mention the secondary surface to the user.

## Step 3: Route

Show the user the classification and the proposed action, then act on confirmation.

### 3a. `claude-plugin`

Best case — it's already structured for native install.

1. Check whether the repo is already known to any of the user's surfaces:
   - **Substrate** (if available): `ssh residencehome "docker exec skill-substrate-skills-ingester-1 skills marketplace list"` — match on `source_repo`.
   - **Local marketplaces**: `claude plugins marketplace list`, then for each, check its `marketplace.json` for an entry whose `source.repo` matches.

2. **If found on the substrate** as marketplace `M`, the install command is:
   ```
   ssh residencehome "docker exec skill-substrate-skills-ingester-1 \
     skills install <M>:<plugin-name>"
   ```
   (Or call the `Skill-Substrate-Admin:admin-subscribe` skill.) Subscription is server-side; no further action needed.

3. **If found in a local marketplace** `M` (legacy / external host), the install command is:
   ```bash
   claude plugins install <plugin-name>@<M>
   ```
   Show the command. **Do not auto-install at user scope.** Mention `--scope project` for ad-hoc use.

4. **If not found anywhere**, offer routes in precedence order:
   - **Substrate route (preferred when substrate is reachable)** — register the repo as a one-plugin marketplace on the substrate, then ingest and subscribe:
     ```
     ssh residencehome "docker exec skill-substrate-skills-ingester-1 \
       skills marketplace add <owner>-<repo> --repo https://github.com/<owner>/<repo>.git \
         --visibility private --authorship third_party"
     ssh residencehome "docker exec skill-substrate-skills-ingester-1 \
       skills marketplace ingest <owner>-<repo>"
     ssh residencehome "docker exec skill-substrate-skills-ingester-1 \
       skills install <owner>-<repo>:<plugin-name>"
     ```
     Use `--authorship third_party` for upstream-by-others (engages the trust gate for review-on-update). Use `--authorship user` only if the upstream is also Daniel's. Prefer calling the `Skill-Substrate-Admin:admin-marketplace-add` and `admin-marketplace-ingest` and `admin-subscribe` skills directly when available.
   - **Third-party marketplace route (substrate not reachable, or multi-host portability without substrate)** — jump to Step 4 with the existing `plugin.json` as the source of truth.
   - **Direct install from GitHub** (one-off, not tracked anywhere):
     ```bash
     claude plugins install <owner>/<repo>
     ```
     Warn that this is the loose-install case the skill is meant to avoid. Only suggest if the user is testing.

### 3b. `claude-marketplace`

The repo is itself a marketplace.

**Substrate route (preferred when available):**

```
ssh residencehome "docker exec skill-substrate-skills-ingester-1 \
  skills marketplace add <owner>-<repo> --repo https://github.com/<owner>/<repo>.git \
    --visibility private --authorship third_party"
ssh residencehome "docker exec skill-substrate-skills-ingester-1 \
  skills marketplace ingest <owner>-<repo>"
```

Then `skills marketplace show <owner>-<repo>` to list its plugins, and `skills install <owner>-<repo>:<plugin-name>` for the ones the user wants. Prefer the corresponding admin skills when available.

**Local-marketplace route (fallback for non-substrate hosts):**

```bash
claude plugins marketplace add <owner>/<repo>
```

List its plugins and ask the user which one(s) to install. Same scoping caveat as 3a — show the command, don't auto-install at user scope.

### 3c. `mcp-server`

Delegate to `rudder:add-mcp-server` — it already knows how to generate a valid `claude mcp add` invocation. Pass through the repo URL and what you learned (npx invocation, env vars from the README, transport type).

If the user explicitly wants the MCP server **also** trackable through their marketplace setup (so a fresh machine reconstitutes it), additionally register a stub entry in the third-party marketplace whose `description` documents the MCP add command. This is a workaround until Claude Code marketplaces natively carry MCP entries — be honest about that in the entry's description.

### 3d. `loose-skill`

This is the case the user described. The upstream repo has skill content but isn't packaged as a Claude Code plugin. Wrap it.

Ask the user which wrapping strategy:

1. **Fork + thin manifest (preferred for active upstreams).**
   - `gh repo fork <owner>/<repo> --clone --remote`
   - Add `.claude-plugin/plugin.json` at the root naming it `<upstream-name>-wrapped` (or similar — confirm name with user). Keep the upstream's existing `skills/`, `commands/`, `agents/` layout if it already matches the plugin convention; if not, reshape minimally and document what changed.
   - Commit on a `claude-plugin-wrap` branch so `main` stays mergeable from upstream.
   - Push the fork. The fork's `origin` is the user's account; `upstream` points at the original — record that so updates can be pulled.

2. **Per-skill wrapper repo (for tiny single-skill cases).**
   - Create `~/repos/github/my-repos/Third-Party-<Name>-Plugin/` with a fresh `.claude-plugin/plugin.json` and a `skills/<name>/` that `git subtree` or vendors the upstream skill content.
   - Add a `.upstream` file at repo root recording `repo`, `commit`, `path` so a companion update flow can refresh later.

Default to (1). Only fall back to (2) if forking the upstream is awkward (monorepo, license issues, the skill is one file in a giant repo).

After wrapping, proceed to Step 4 to register the wrapper in the third-party marketplace.

### 3e. `other`

Not installable as a Claude Code surface. Tell the user what the repo actually is and stop. Don't try to force-fit it.

## Step 4: Register in the third-party marketplace

For every path that produced a Claude-installable plugin (3a-register, 3d-wrap), append an entry to `<third_party_marketplace.local_path>/.claude-plugin/marketplace.json`:

```json
{
  "name": "<plugin-name>",
  "source": {
    "source": "github",
    "repo": "<owner-of-fork-or-wrapper>/<repo>"
  },
  "description": "<one line — note 'wrapper around <upstream owner/repo>' if applicable>",
  "version": "<version from plugin.json>",
  "author": {
    "name": "<upstream author if known, else Daniel Rosehill>"
  },
  "license": "<upstream license>",
  "tags": ["third-party", "<topical tag>"]
}
```

Also append to the marketplace's README under a "Third-Party Wrappers" section noting the upstream link and the wrapping strategy used.

Validate JSON, commit, push:

```bash
cd <third_party_marketplace.local_path>
python3 -c "import json; json.load(open('.claude-plugin/marketplace.json'))"
git add -A
git commit -m "Add <plugin-name> (wraps <upstream owner/repo>)"
git push
```

Refresh the local marketplace cache so the new entry is discoverable:

```bash
claude plugins marketplace update <owner>
```

## Step 5: Report

Show the user:

1. What the input repo was classified as.
2. Which routing path was chosen (substrate / primary marketplace / third-party marketplace / direct GitHub install) and why.
3. What action was taken (substrate marketplace registered + ingested + subscribed / install command shown / marketplace entry added / MCP add-command emitted).
4. For substrate routes: the substrate marketplace name, ingest report record count, and any `rejected[]` entries.
5. For local-marketplace routes: the exact `claude plugins install <plugin-name>@<marketplace>` command they can run if/when they want it active. **Don't run it for them** unless they explicitly ask.
6. For wrapped third-party skills, the upstream tracking metadata (fork remote `upstream`, or `.upstream` file path) so a future update flow can pull changes.

## Notes on upstream tracking

Wrapped third-party skills will eventually drift from upstream. This skill does **not** implement the update sweep — that's a future companion skill (`update-third-party-skills`). What this skill must do is leave enough metadata that the update sweep is mechanical:

- Forks: `git remote -v` shows `upstream`. The wrapping branch is named `claude-plugin-wrap` so `main` can fast-forward from upstream.
- Wrapper repos: `.upstream` file at repo root, format:
  ```
  repo=<owner>/<repo>
  commit=<sha at wrap time>
  path=<path within upstream that was vendored>
  strategy=<subtree|vendor>
  ```

Don't skip these — without them, the third-party marketplace becomes its own untrackable mess, defeating the purpose.
