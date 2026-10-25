---
name: declaude-docs
description: Use when the user wants to strip AI-generated style tics from documentation — emojis, excessive lists, all-caps filenames, fragmented structure. Triggers on phrases like "declaude the docs", "clean up documentation style", "remove emojis from docs", "apply style guide".
---

Navigate the repository, identify all documentation files, and ensure they adhere to the style guide below.

# Comprehensive Technical Writing and Style Guide

This guide establishes mandatory stylistic, structural, and tonal constraints for all generated text, documentation, and code artifacts.

## I. Tonal and Professionalism Standards

### 1. Prohibition of Emojis and Non-Standard Typographical Elements

Maintain a professional, serious, authoritative tone. Emojis, excessive enthusiasm markers, and informal slang are forbidden. The emoji ban covers every output: prose, headings, code comments and filenames.

| Guideline | TO DO | NOT TO DO |
|---|---|---|
| Emoji Policy | `The validation failed due to a timeout.` | `The validation failed due to a timeout! ⚠️` |
| Exclamations | Use period or colon instead (exception: genuine urgency in a safety-critical context). | `Function execution is complete!!!` |
| Slang/Informality | Formal, objective, third-person. | `Make sure you initialize the cool parameters.` |

### 2. File Naming Conventions

Only `README.md` conventionally uses all-caps. All other docs use lowercase kebab-case. Code files follow language conventions.

| Guideline | TO DO | NOT TO DO |
|---|---|---|
| README Exception | `README.md` | `ReadMe.md` |
| General Docs | `installation-guide.md` | `INSTALLATION_GUIDE.MD` |
| Code Files | `data_processor.py` | `DATA_PROCESSOR.PY` |

## II. Structural and Architectural Guidance

### 3. Documentation Philosophy: Utility Over Volume

- **Contextual Necessity:** Documentation must be requested or demonstrably necessary.
- **Avoid Boilerplate:** Do not generate `ARCHITECTURE.md`, `GOALS.md`, `ROADMAP.md`, or detailed `INSTALLATION.md` for trivial steps unless asked.
- **Focus on Maintainers and Users:** Critical context (why) and practical steps (how).
- **Conciseness:** Prefer integrating brief critical instructions into an existing file (`README.md` or the relevant source file) over a new file holding one paragraph.

### 4. Structural Density and Flow (Anti-Fragmentation)

**Heading Constraint:** A subheading must be followed by substantial development (3+ sentences or a complex example) before the next subheading.

**Avoid topic churn:** do not introduce a concept, summarise it in one bullet and jump to the next heading. Develop it first.

**Tables:** Reserved for comparative data, summary statistics, or structured mappings. Do not summarize prose into tables, and never add a table that repeats points just made in a list or the headings.

**Lists:** For enumerating steps, assumptions, or distinct items. Not a substitute for paragraph development: if the points are conceptually intertwined, write a paragraph.

Limit nesting to H1–H3 unless the document exceeds 3,000 words.

## III. Summary of Mandatory Constraints

- **NO EMOJIS.** Under any circumstances.
- **NO ALL CAPS** filenames except `README.md`.
- **NO SUPERFLUOUS DOCS.** Only essential, context-specific, requested.
- **NO FRAGMENTATION.** Balanced structure, developed prose.
