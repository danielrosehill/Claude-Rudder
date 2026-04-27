---
name: chunk-claude-md
description: Use when the user wants to prune and chunk a bloated CLAUDE.md into a structured context/ store with pipeline folders and subagent templates. Triggers on phrases like "chunk claude", "prune CLAUDE.md", "set up context store", "split CLAUDE.md".
---

Review the CLAUDE.md in this repository. It may be bloated. Work through these tasks in order:

## 1. Create Context Store

Create a folder called `context` at the repository root if it doesn't already exist. This is the context store — a structured directory where detailed context data lives, organized by topic.

## 2. Prune & Chunk CLAUDE.md

- Reduce CLAUDE.md to the minimum length possible for an AI agent to gain the essential details of this project. "Essential details" means enough information to gain foundational context about the project and understand its key purpose.
- Any extraneous information (detail that may be helpful to an agent but isn't immediately necessary) should be extracted into topic-based subfolders and markdown files within `context/`.

Example (paths relative to repo root):

```
context/
context/infra/
context/infra/deployments.md
context/architecture/
context/architecture/data-model.md
```

Each file should have a descriptive name. Group related files into topic subfolders.

## 3. Reference the Context Store

Add a reference in CLAUDE.md pointing agents to the context store:

> For more specific context data to assist with this project, see the files in the `context/` folder. Review relevant context files before asking the user questions.

## 4. Create the Pipeline Folder

Create `context/pipeline/` with these subdirectories, each containing a `.gitkeep`:

```
context/pipeline/.gitkeep
context/pipeline/audio/.gitkeep
context/pipeline/documents/.gitkeep
```

- `audio/` — voice notes, recordings, transcripts
- `documents/` — PDFs, text files, reference materials

Write a `CLAUDE.md` inside `context/pipeline/`:

```
# Pipeline

This folder is an ingestion pipeline for the context store.

Users may place raw data here — notes, transcripts, research, references, or any unstructured material — that they want incorporated into the project's context pool.

## Subdirectories

- `audio/` — Audio files (voice notes, recordings). Transcribe these first, then extract context.
- `documents/` — Document files (PDFs, text, reference materials). Parse and extract context.

## Processing Rules

When you find files in this folder (or its subdirectories):

1. Parse and understand the raw content (transcribe audio files first if needed)
2. Extract relevant context information
3. Create or update appropriately named markdown files in the parent `context/` directory (organized into topic subfolders)
4. Once the content has been successfully ingested into the context store, delete the source file from `pipeline/`

This folder should generally remain empty after processing. It is a staging area, not a permanent store.
```

## 5. Create Subagent Templates

Create `context/agents/` containing templates for subagents that maintain the context store.

### 5a. `context/agents/pipeline-processor.md`

```
# Pipeline Processor Agent

## Purpose

Process raw context from the pipeline into the context store.

## Instructions

1. Scan `context/pipeline/`, `context/pipeline/audio/`, and `context/pipeline/documents/` for new files.
2. For each file found:
   - **Audio files**: Transcribe the audio content, then extract key context information from the transcript.
   - **Documents**: Parse the document content and extract key context information.
   - **Other files**: Read and extract relevant context.
3. Chunk the extracted information into focused, topic-based markdown files.
4. Place each chunk into the appropriate subfolder within `context/`, creating new subfolders and files as needed. Use descriptive filenames.
5. If the information updates or extends an existing context file, merge it cleanly rather than creating duplicates.
6. After successful ingestion, **delete the source file** from the pipeline folder.
7. Commit changes with a message describing what context was ingested.

## Constraints

- Never leave partially processed files — either fully ingest or leave untouched.
- Keep individual context files focused on a single topic or concern.
- Use clear, descriptive filenames and folder names.
```

### 5b. `context/agents/context-organizer.md`

```
# Context Organizer Agent

## Purpose

Maintain a clean, well-structured, and non-redundant context store.

## Instructions

1. Review all files and subfolders within `context/` (excluding `pipeline/` and `agents/`).
2. Identify and resolve:
   - **Duplicate or overlapping content**: Merge into a single authoritative file.
   - **Misplaced files**: Move files to the correct topic subfolder.
   - **Overly large files**: Split into focused, topic-specific files.
   - **Stale or outdated information**: Flag to the user or remove if clearly obsolete.
   - **Poorly named files or folders**: Rename for clarity and consistency.
3. Ensure every subfolder has a clear topical scope.
4. After reorganization, verify that references in `CLAUDE.md` still point to valid paths. Update if needed.
5. Commit changes with a message describing the reorganization performed.

## Constraints

- Never delete context that may still be relevant — when in doubt, keep it and flag for user review.
- Preserve the meaning and detail of all content during merges or splits.
- Do not modify `pipeline/` or `agents/` directories.
```

### 5c. `context/agents/CLAUDE.md`

```
# Agent Templates

This folder contains subagent templates for maintaining the context store.

The orchestrating agent (the primary Claude Code session) should spawn these subagents as needed:

- **pipeline-processor.md** — Run when new files appear in `context/pipeline/`. Ingests raw context into the store.
- **context-organizer.md** — Run periodically or on request to clean up, deduplicate, and reorganize the context store.

To use: read the relevant template and launch a Task agent with its instructions.
```

## 6. Add Memory Boundary Guidance

Add this to the repository's root `CLAUDE.md` after the context store reference:

```
## Context Store vs. MEMORY.md

This project uses a `context/` directory as a shared, version-controlled knowledge base. To avoid duplication with Claude Code's auto-memory (`~/.claude/projects/.../memory/MEMORY.md`), follow this boundary:

- **`context/` store** — Project knowledge: architecture decisions, domain context, specifications, requirements, and anything a collaborator or future agent would need to understand the project.
- **MEMORY.md** — Working patterns: how to run tests, common dev gotchas, preferred tooling shortcuts, and session-learned conventions.

Do not duplicate information between the two. If something belongs in the project record, it goes in `context/`. If it's an ephemeral working note about how to interact with this codebase, it goes in MEMORY.md.
```
