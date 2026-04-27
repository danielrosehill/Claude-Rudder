---
name: mirror-to-private
description: Create a private GitHub repo that mirrors the current public repo. Use when the user says "mirror this to a private repo", "create a private mirror", or "private copy of this repo".
---

# Mirror Public Repo To Private

Create a private GitHub repository that mirrors the current repo's contents, so Daniel can keep a private working copy alongside the public one.

## Steps

1. Confirm the current directory is a git repo and capture its name and remote:
   ```bash
   git rev-parse --show-toplevel
   git remote get-url origin
   basename "$(git rev-parse --show-toplevel)"
   ```
2. Ask the user for the private repo name (default: `<current-name>-private`). Confirm before creating.
3. Create an empty private repo on GitHub under `danielrosehill`:
   ```bash
   gh repo create danielrosehill/<NAME> --private --description "Private mirror of <original>"
   ```
4. Clone the current repo as a mirror into a temp dir and push to the new private remote, so all branches and tags carry over:
   ```bash
   tmp=$(mktemp -d)
   git clone --mirror "$(git rev-parse --show-toplevel)" "$tmp/mirror.git"
   cd "$tmp/mirror.git"
   git push --mirror "git@github.com:danielrosehill/<NAME>.git"
   ```
5. Report both URLs (public original + new private mirror) to the user. Do not alter the original repo's remotes unless asked.

## Notes

- Use `--mirror` (not a plain clone) so all refs transfer.
- Never change the original repo's `origin` remote without explicit confirmation.
- If the user wants ongoing sync, suggest they run this skill again or set up a scheduled sync — do not silently add automation.
