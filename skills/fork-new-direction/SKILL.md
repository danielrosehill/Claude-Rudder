---
name: fork-new-direction
description: Create a new repo that starts from the current repo's state but takes a new direction — copy the code, seed CLAUDE.md with the revised approach, and push. Use when the user says "fork this in a new direction", "new repo from here with a new approach", or "branch this into a new project".
---

# Fork Repo Into New Direction

Spin the current repo off into a brand-new repository that inherits the current code state but is briefed for a different direction. The new repo's `CLAUDE.md` captures the revised approach so the next Claude session can pick up cleanly.

This is a session-transfer pattern: the handover is encoded in the new repo's CLAUDE.md rather than a HANDOVER.md.

## Steps

1. Capture current repo info:
   ```bash
   root=$(git rev-parse --show-toplevel)
   name=$(basename "$root")
   parent=$(dirname "$root")
   ```
2. Ask the user for:
   - New repo name (suggest `<name>-v2` or similar).
   - A short description of the new direction — what's changing, what's being abandoned, what should the next agent focus on. This becomes the heart of the new `CLAUDE.md`.
   - Public or private (default private).
3. Create the new directory and copy the working tree (no `.git`), so history stays with the original:
   ```bash
   mkdir "$parent/<new-name>"
   rsync -a --exclude='.git' "$root/" "$parent/<new-name>/"
   cd "$parent/<new-name>"
   git init -b main
   ```
4. Rewrite `CLAUDE.md` in the new repo. Structure:
   - **Origin**: forked from `<original-name>` at commit `<sha>` on `<date>`.
   - **Original direction (abandoned)**: one-paragraph summary of what the old repo was doing.
   - **New direction**: the user's revised approach, verbatim where possible.
   - **Carried over**: what code/assets are reused from the original.
   - **Next steps**: concrete starting tasks for the next session.
   Preserve any existing project-specific CLAUDE.md content that is still relevant; replace the parts that no longer apply. Confirm with the user before overwriting.
5. Update `README.md` with a brief note that this repo is a new-direction fork of `<original>`.
6. Initial commit, create GitHub repo, push:
   ```bash
   git add -A
   git commit -m "Initial commit: fork from <original-name>@<short-sha> with new direction"
   gh repo create danielrosehill/<new-name> --private --source . --remote origin --push
   ```
   (Use `--public` if the user asked for public.)
7. Report: new local path, GitHub URL, commit sha the fork started from, and a reminder that the original repo is untouched.

## Notes

- Do not modify the original repo — this is a one-way fork.
- Copy the working tree, not `.git`, so the new repo starts with a clean history and a single origin commit.
- The revised-approach CLAUDE.md is the whole point — do not skip or stub it. If the user hasn't given enough detail, ask before committing.
