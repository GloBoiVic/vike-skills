---
name: init
description: Project context discovery and initialization. Detect context/, inventory files, create context/index.md only when missing, scaffold project-type starter docs when context/ is absent. Never overwrites or deletes existing files.
---

# Init — Safe Context Bootstrap

`context/index.md` is a compact manifest; context files are the project's source of truth. This skill only discovers and initializes context. Never edit, overwrite, or delete an existing file.

## Procedure

1. Check whether `<project-root>/context/` exists.
2. If it exists, inventory every file and directory without reading contents. Create **only** a missing `context/index.md`, with one short entry per item. Report recommended docs that are absent; do not scaffold them.
3. If it does not exist, infer the project type from repository markers (ask only if genuinely ambiguous), create `context/`, and add brief starter drafts for `project-brief.md`, `tech-stack.md`, `architecture.md`, `coding-standards.md`, plus relevant optional docs such as database, design, security, API, library, domain, or token notes.
4. Create an index for everything created. Mark generated docs as **Draft** and tell the developer to refine them.

An existing index is never edited, even if it omits an item; report the omission for a normal documentation workflow. Keep descriptions terse. `ui-registry.md` is merely optional; this skill does not require or create it.

## Index contract

The index says to read root `AGENTS.md` first, then load only task-relevant docs. Include `## Core`, `## Optional`, and `## Missing (reported, not scaffolded)` as applicable. Entries use:

`- **path** — what it contains`

Starter drafts need only a heading, task-relevant placeholders, and `Status: Draft — refine with the team.` Do not treat placeholders as authoritative.

## Report

```text
Context initialized.
- context/: existed | created
- Files created: [list]
- Existing files preserved: [list or none]
- Missing recommended docs: [list or none]
- Generated docs are starter drafts; refine before relying on them.
```
