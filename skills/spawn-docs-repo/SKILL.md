---
name: spawn-docs-repo
description: Create a sibling docs repo for the current project, move documentation out of the main repo into it, seed CLAUDE.md, and push to GitHub. Use when the user says "spawn a docs repo", "split docs into its own repo", or "rip out the docs".
---

# Spawn Docs Repo

Create a dedicated documentation repository alongside the current code repo. Docs (README-adjacent content, guides, references, tutorials, architecture write-ups) are moved out of the main repo into the new one, keeping the code repo lean.

## Steps

1. Capture current repo info:
   ```bash
   root=$(git rev-parse --show-toplevel)
   name=$(basename "$root")
   parent=$(dirname "$root")
   ```
2. Propose a docs repo name (default: `<name>-docs`) and confirm with the user. Ask public or private (default private).
3. Survey the current repo for documentation — e.g. `docs/`, `documentation/`, `guides/`, `reference/`, `tutorials/`, top-level `*.md` files beyond `README.md`/`CLAUDE.md`/`LICENSE`. Present the list to the user and confirm what should move. Do not guess silently. Leave the top-level `README.md` in the main repo.
4. Create the new repo directory and initialize it:
   ```bash
   mkdir "$parent/<docs-name>"
   cd "$parent/<docs-name>"
   git init -b main
   ```
5. Seed it with:
   - A `README.md` naming the sibling code repo and explaining this is its documentation companion.
   - A `CLAUDE.md` briefly stating: this repo holds documentation for `<name>`, which lives at `<path>` / `<github url>`. Include a note that docs should be written for humans (not AI handover-style), and preserve any existing docs-writing conventions from the source repo's CLAUDE.md if relevant.
   - The doc files moved from the main repo, preserving their directory structure where it makes sense.
6. In the main repo: remove the moved files with `git rm` (or `rm` if untracked), update the main `README.md` to link to the new docs repo, and commit with a message like `chore: move docs into sibling <docs-name> repo`. Do NOT push the main repo unless the user asks.
7. In the new docs repo: `git add -A`, initial commit, then create the GitHub repo and push:
   ```bash
   gh repo create danielrosehill/<docs-name> --private --source . --remote origin --push
   ```
   (Use `--public` if the user asked for public.)
8. Report: new repo path, GitHub URL, list of files moved, and the pending (uncommitted-pushed) state of the main repo.

## Notes

- Always confirm the file list before moving anything — what counts as "docs" is subjective.
- Leave the top-level `README.md` in the main repo; it's the entry point.
- Update the main `README.md` to point at the new docs repo so discoverability isn't lost.
- Preserve subdirectory structure (`docs/guides/foo.md` → `guides/foo.md` in the new repo) unless the user wants a flatter layout.
