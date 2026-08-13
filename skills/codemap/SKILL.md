---
name: codemap
description: Explicitly requested, read-only structural map of repository paths, relationships, ownership, risks, and unknowns; never mutates the repository.
---

# Codemap

This skill is inert until the user or orchestrator explicitly requests a clear, bounded mapping question. It is not automatic preflight, Explore/Architect, testing, review, or implementation.

## Contract

1. Confirm the question and exact boundaries. Stop if scope is missing or unsafe.
2. Inspect relevant files and directories with read-only tools only. Do not run mutating commands, install dependencies, contact remotes, or expose secrets.
3. Distinguish observed facts from inferences; cite paths and line ranges where available. Do not infer behavior without labeling it.
4. Return the map in chat or another non-mutating response. Never write, generate, format, persist, or silently document a filesystem artifact.

```markdown
## Scope
[paths and question]
## Structure
[directories, entry points, file roles]
## Relationships
[imports, dependencies, data flow, ownership]
## Risks and unknowns
[gaps with confidence labels]
## Suggested next steps
[optional, non-binding]
```
