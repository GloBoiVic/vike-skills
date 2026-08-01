# Phase 2 Plan — Adaptive Project Context

## What we are building

An adaptive `/context` discovery and initialization workflow for OpenCode projects. The workflow treats `context/index.md` as the manifest and source-of-truth entry point, preserves existing project knowledge, and generates only project-type-relevant starter documents when no context exists.

## Tasks

1. Create an `init` skill for context discovery and initialization — assigned to documenter/skill specialist.
2. Update orchestrator, explorer, remember, and frontend guidance to use `AGENTS.md` plus `context/index.md`, then selectively load relevant context — assigned to documenter/skill specialist.
3. Add model/task tracking and validation notes — assigned to orchestrator.

## Complexity tier

Feature — configuration and workflow architecture, no application-code changes.

## Constraints

- Never overwrite or delete existing context files.
- If `context/` exists, inventory it and create `context/index.md` only when missing.
- If `context/` does not exist, ask or infer project type before generating recommended templates.
- Core context files: `project-brief.md`, `tech-stack.md`, `architecture.md`, `coding-standards.md`.
- Optional context is domain-specific: database, design, UI registry/tokens, library docs, security, API contracts, and other relevant files.
- Agents read `AGENTS.md` and `context/index.md` first, then load only task-relevant files.
- Do not split `impeccable` in this phase.
