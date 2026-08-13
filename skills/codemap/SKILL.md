---
name: codemap
description: Opt-in, read-only structural mapping of a repository; reports architecture and file relationships without changing source or permanent documentation.
---

# Codemap

This skill is inert unless the user or orchestrator explicitly invokes it. It creates a structural map for understanding a repository; it does not implement, refactor, document, or write files.

## When to use

- Use when a clear, bounded request needs a repository structure, dependency relationship, or ownership map before implementation.
- Do not use as an automatic preflight, replacement for required Explore or Architect work, or substitute for tests or review.

## Scope and workflow

1. Confirm the mapping question and boundaries before inspection.
2. Inspect relevant files and directories with read-only tools only.
3. Distinguish observed facts from inferences and cite paths and line ranges when available.
4. Return the map in-chat or otherwise non-mutating; never write it to the filesystem.

## Outputs

Return:

- **Scope** — the paths and question covered.
- **Structure** — important directories, entry points, and file roles.
- **Relationships** — relevant imports, dependencies, data flow, or ownership links.
- **Risks and unknowns** — gaps, stale evidence, and confidence labels.
- **Suggested next steps** — optional, non-binding follow-ups.

## Safety boundaries

- Read-only means no edits, file creation, deletion, renaming, formatting, generated output, or silent permanent documentation/source changes.
- Do not run mutating commands, install dependencies, contact remote services, or expose secrets.
- Do not infer behavior without labeling it as an inference.
- Do not persist the map or create any filesystem artifact, even when a destination is requested.

## Invocation rule

Never invoke this skill automatically. Wait for an explicit human or orchestrator request, and stop if the requested scope is missing or unsafe.
