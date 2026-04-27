---
name: repo-from-thread
description: Create a new GitHub repo seeded with context from the current conversation. Use when the user says "repo from thread", "repo from chat", "let's turn this conversation into a repo", or "create a repo to manage this project" mid-conversation.
---

# Repo From Thread

Mid-conversation, the user realizes the work being discussed deserves its own GitHub repo. Spin one up and seed it with the context accumulated so far, so a fresh Claude session in that repo can pick up where this thread left off.

## Steps

1. Ask the user:
   - Repo name (propose one based on the conversation topic).
   - Public or private.
   - Parent directory (default: `~/repos/github/my-repos/`).
2. Synthesize the conversation so far into seed context. Do NOT dump the raw transcript — distill it. Cover:
   - What the project is and why it exists (the goal the user articulated).
   - Decisions already made in the thread.
   - Open questions / next steps.
   - Any file paths, commands, URLs, or constraints mentioned.
3. Create and initialize the repo:
   ```bash
   mkdir -p <parent>/<name> && cd <parent>/<name>
   git init -b main
   ```
4. Seed files:
   - `README.md` — short project description.
   - `CLAUDE.md` — the distilled context from step 2, framed as instructions for a fresh Claude session ("This repo was spun out of a conversation on <date>. Context so far: …").
   - `context/thread-origin.md` — longer-form notes on decisions, open questions, next steps.
5. Initial commit, create GitHub repo, push:
   ```bash
   gh repo create danielrosehill/<name> --<public|private> --source . --remote origin --push
   ```
6. Report: local path, GitHub URL, and suggest the user run `/session-transfer:new-claude-at <path>` to continue the work in a fresh session with the seeded context.

## Notes

- The value of this skill is the distillation in step 2 — a raw transcript dump is useless. Write the CLAUDE.md as if briefing a colleague who just walked in.
- If the conversation is very short or context-thin, ask the user to add anything important you might have missed before committing.
- Default to private unless the user says otherwise.
