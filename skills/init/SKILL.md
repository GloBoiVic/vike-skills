---
name: init
description: Project context discovery and initialization. Detect context/, inventory files, create context/index.md only when missing, scaffold project-type starter docs when context/ is absent. Never overwrites or deletes existing files.
---

# Init — Adaptive Project Context

`context/` is the project's knowledge source of truth. `context/index.md` is the manifest that tells every agent what exists and which docs to load for which task.

This skill performs discovery and safe initialization only. It never overwrites, edits, or deletes existing files. Everything it generates is a starter draft for the developer and future sessions to refine.

## Core Principles

- **Existing files are authoritative and immutable.** Never modify, overwrite, or delete a file that already exists in `context/`.
- **Index is a manifest, not a replacement.** `context/index.md` summarizes what each doc contains so agents load selectively — it is not the content itself.
- **Only missing things are created.** When `context/` exists, the only thing created is `context/index.md` (and only if it is missing). Missing recommended docs are reported, not scaffolded.
- **Templates are starter drafts.** Generated docs are skeletons. They become authoritative only when the developer fills them in.

## Workflow

### Step 1 — Detect

Check whether `<project-root>/context/` exists.

### Step 2a — `context/` exists → Inventory mode

1. **Inventory** — list every file and subdirectory under `context/` (use glob/ls; do not read file contents yet).
2. **Create the index if missing** — if `context/index.md` does not exist, create it with one entry per inventory item (see Index Format below).
3. **Identify gaps** — compare the inventory against the recommended docs for the project type (see Recommended Docs). List the missing recommended docs. Do **not** create them; report them to the developer and let them decide.
4. **Preserve everything** — never overwrite, edit, or delete any existing file.

### Step 2b — `context/` does not exist → Create mode

1. **Identify the project type** — prefer inference from the codebase (`package.json`, `tsconfig.json`, framework configs, `prisma/schema.prisma`, `requirements.txt`, lock files, etc.). Ask the developer only when the type is genuinely ambiguous (e.g., a brand-new empty repo).
2. **Create `context/`**.
3. **Generate the core templates** (always):
   - `project-brief.md`
   - `tech-stack.md`
   - `architecture.md`
   - `coding-standards.md`
4. **Generate relevant optional templates** based on project type (see Optional Templates).
5. **Create `context/index.md`** indexing everything created.
6. **Tell the developer** these are starter drafts — refine them before treating them as authoritative.

## Recommended Docs

### Core (always)

| File | Purpose |
|------|---------|
| `project-brief.md` | What the project is, who it is for, goals and non-goals |
| `tech-stack.md` | Languages, frameworks, key libraries, versions |
| `architecture.md` | System boundaries, module layout, data flow |
| `coding-standards.md` | Conventions: style, testing, error handling, commit rules |

### Optional (project-type relevant)

| File | Include when |
|------|--------------|
| `database.md` | Backend, data-heavy, ORM/DB present |
| `design.md` | Any UI work or design system |
| `ui-registry.md` | UI work — component pattern registry |
| `ui-tokens/` (directory) | Design tokens warrant their own files (colors, spacing, type) |
| `library-docs.md` | Heavy external dependency usage, custom library notes |
| `security.md` | Auth, payments, PII, secrets handling |
| `api-contracts.md` | Client/server or service boundaries, external APIs |
| `domain-specific.md` | Industry knowledge that changes decisions (finance, healthcare, games, etc.) |

Map by project type, e.g.:

- **Web app (UI-heavy)** → `design.md`, `ui-registry.md`, `ui-tokens/`, `api-contracts.md`
- **Backend / API** → `database.md`, `security.md`, `api-contracts.md`
- **Library / package** → `library-docs.md`, `security.md`
- **Full-stack** → all of the above that apply

## Index Format

`context/index.md` is one line per doc. Keep descriptions tight — the index is scanned by every agent, so brevity saves tokens.

```markdown
# Context Index

> Source of truth for project knowledge. Read this first, then load only the docs relevant to your task.
> Last updated: [date]

## How to use

- Read `AGENTS.md` at the project root first, then this index.
- Load only the docs your task needs — do not bulk-read `context/`.
- Existing docs are authoritative; edits happen through normal project workflows, not initialization.

## Core

- **project-brief.md** — [one-line: what the project is]
- **tech-stack.md** — [one-line: stack and versions]
- **architecture.md** — [one-line: boundaries and layout]
- **coding-standards.md** — [one-line: key conventions]

## Optional

- **database.md** — [one-line: schema/ORM notes]
- **design.md** — [one-line: design system, tokens, spacing]
- **ui-registry.md** — [one-line: component pattern registry]
- **ui-tokens/** — [one-line: token files and what each holds]
- **security.md** — [one-line: trust boundaries, secrets, auth]

## Missing (reported, not scaffolded)

- [recommended doc that does not exist yet]
```

If a doc exists but has no entry, add an entry for it. If the index already covers everything, leave it unchanged — never rewrite it wholesale.

## Starter Templates

Use these skeletons for Create mode. Each is a starting point, not a finished document. Keep generated templates brief; the developer expands them.

### project-brief.md

```markdown
# Project Brief

## What we are building

[One paragraph — what the product/feature is]

## Who it is for

[Audience, users, stakeholders]

## Goals

- [measurable outcome]

## Non-goals

- [explicitly out of scope]

## Status

Draft — starter template. Refine with the team.
```

### tech-stack.md

```markdown
# Tech Stack

| Layer | Choice | Version | Notes |
|-------|--------|---------|-------|
| Language | | | |
| Framework | | | |
| Database | | | |
| Styling | | | |
| Testing | | | |
| Tooling | | | |

## Key libraries

- [library] — [purpose]

## Status

Draft — starter template. Confirm with the team.
```

### architecture.md

````markdown
# Architecture

## Boundaries

- [system/module boundary — responsibility]

## Layout

```
[high-level structure of the codebase]
```

## Data flow

- [how data moves across boundaries]

## Key decisions

- [decision] — [rationale]

## Status

Draft — starter template. Refine as design decisions are made.
````

### coding-standards.md

```markdown
# Coding Standards

## Style

- [formatting, naming, structure]

## Testing

- [what to test, how, coverage expectations]

## Error handling

- [how errors are surfaced and logged]

## Commits / PRs

- [convention]

## Status

Draft — starter template. Align with the team before enforcing.
```

## Token-Efficient Guidance

- **Read the index, not the directory.** Never bulk-read every file in `context/`.
- **Load only what the task needs.** Match task keywords against index descriptions, then read just those docs.
- **Keep inventory descriptions to one line.** Long descriptions defeat the purpose of the manifest.
- **Prefer excerpts.** When handing context to another agent, pass a short excerpt plus a pointer to the full file — not the whole file.
- **Report, don't copy.** The `remember` skill stores session state in `memory.md`; if the fact already lives in a context doc, reference it instead of duplicating it.

## Reporting

After running, report to the developer:

```
Context initialized.

- context/ existed / was created
- Files created: [list]
- Files preserved untouched: [list or "none existed"]
- Missing recommended docs: [list or "none"]
- Templates are starter drafts — refine before relying on them.
```
