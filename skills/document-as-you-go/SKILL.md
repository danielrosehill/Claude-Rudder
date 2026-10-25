---
name: document-as-you-go
description: Sweep this session for durable findings and undocumented changes, then file them where found again. "document as you go", "doc sweep".
argument-hint: "[--findings | --changes] [--check] [--since <ref>]"
---

<!-- Source: danielrosehill/Document-As-You-Go commands/document-as-you-go.md (canonical, generic copy). Keep the two in step. -->

Harvest the documentation debt built up in this session and pay it down.

Arguments: `$ARGUMENTS`

- no arguments — both passes below
- `--findings` — discoveries only
- `--changes` — code changes only
- `--since <ref>` — widen the change pass to everything since `<ref>`
- `--check` — report what is missing and stop; write nothing

## Pass 1 — Findings

Look back over the whole session, not the diff. The most valuable knowledge here
is usually not in any file you edited: it is what you learned about the systems
you were working against.

Collect candidates:

- Undocumented or internal API structure — endpoints, methods, parameters,
  pagination, rate limits, error shapes.
- Authentication and authorisation mechanics: what the flow actually requires,
  where credentials live, how tokens expire.
- The real shape of data — exports, schemas, field names and their true meanings,
  units, encodings, timezone handling.
- Mappings you had to work out, such as which UI action triggers which request.
- Platform quirks, bugs, and undocumented limits.
- The working sequence of steps, and the ones that failed before it.

Keep a candidate if you had to reverse-engineer it, if the vendor documentation is
absent or wrong, if it took several attempts, or if you would have to rediscover
it the same way next time. Discard anything a reader could get from the vendor's
own documentation in one search.

Then place each by reach:

| Reach | Home |
| --- | --- |
| Beyond this task — about a platform or system that will come up again | A dedicated knowledge repo for that subject. Check whether one already exists before proposing a new one. |
| This project only | `docs/` or `findings/` in the current repo |
| Right now only | Your reply. Nothing on disk. |

Write in-repo documentation without asking. For a new repository, state what you
found and why it deserves its own home, and create it only once the user agrees —
**private by default** unless they say otherwise.

Each finding should be usable cold: how the system is reached and authenticated
(credentials referenced, never pasted), concrete verified request and response
shapes with identifiers redacted, what failed and how, the absolute date of
verification, and a clear line between what you confirmed and what you inferred.

## Pass 2 — Changes

Read the diff — the working tree by default, or since `<ref>` if given. Look for
facts a competent reader could not recover from the code: a decision between
viable approaches and the constraint that settled it, a non-obvious constraint, a
workaround for an upstream bug, a new environment variable or command or migration
step, a changed public contract, an unverified assumption, and any documented
statement this change made untrue.

Route each one:

| Fact | Home |
| --- | --- |
| What the project is; how to install, configure, run it | `README.md` |
| A convention or gotcha an agent must follow here | `CLAUDE.md` / `AGENTS.md` |
| A decision with tradeoffs and rejected alternatives | `docs/decisions/NNNN-slug.md` |
| Why this particular line is strange | inline comment, at the line |
| What a function or endpoint promises and assumes | docstring / API reference |
| A user-visible behaviour change | `CHANGELOG.md` |
| How to recover when it breaks | `docs/runbooks/<service>.md` |

## Placement

Anything either pass produces goes in the repository, and the repository is a
workspace rather than a code project: alongside prose it can hold worked examples,
scripts worth running again, redacted fixtures showing the real shape of the data,
and diagrams. If the folder a given item needs does not exist — `docs/`,
`examples/`, `scripts/`, `docs/reference/` — create it as part of filing that item.
Match the repository's own names and layout where it has them; do not add a second
folder meaning what an existing one already means.

Create no folder you are not filling in this pass, and commit no credentials,
tokens, or unredacted data into any of them.

## Filtering, in both passes

Drop anything that restates the diff, narrates what you did, describes a routine
refactor with no behaviour change, or matters only to the current conversation.
Drop aspirational notes about work not yet done, and one-off scripts nobody will
run a second time.

Before creating anything, run `git ls-files` and grep for the subject: prefer
editing an existing document over starting a parallel one, and prefer a new
section in an existing file over a new file. Follow the repo's existing
conventions for structure, tone, and ADR numbering.

If nothing survives, say so plainly and stop. That is a normal and good outcome —
better than inventing documentation to look productive.

## Propose, then write

Present a table — the fact, its destination, and the exact text you intend to
write — before touching any file. Keep each entry to the smallest useful
increment. Stop here if `--check` was passed; otherwise wait for the user to
confirm or amend the plan, then apply it.

Anchor claims that will age — versions, pricing, external API behaviour — to an
absolute date. Do not create `SUMMARY.md`, `IMPLEMENTATION_NOTES.md`, or any other
record of this command having run.

## Report

One line per item written, with its destination. Then note separately: anything
you deliberately left undocumented and why, and any finding you could not place
because the right home does not exist and creating it was out of scope.
