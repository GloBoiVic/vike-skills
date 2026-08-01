# Phase 2 Architecture — Adaptive Project Context

`/context` is the project knowledge source of truth. `context/index.md` is a lightweight manifest, not a replacement for the documents it indexes.

Agents must first read project-level `AGENTS.md`, then `context/index.md`. They should selectively read context documents based on the task and the index descriptions. The orchestrator owns initialization decisions; the `init` skill performs discovery, inventory, and safe creation.

Existing files are authoritative and immutable to initialization. Missing index creation is allowed. Missing recommended documents are reported and may be scaffolded only when context is newly created or the user explicitly approves creation.
