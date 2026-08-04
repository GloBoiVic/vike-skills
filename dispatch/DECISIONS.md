# Decisions

## 2026-08-04 — Mandatory Architect Workflow + Config-Authoritative Models

- **Context:** The Vike Skills orchestrator workflow allowed Architect to be skipped for Feature tasks ("Architect if needed"), creating risk of implementation drift and missed architectural decisions. Separately, the README agent-model table had diverged from the actual `opencode.jsonc` config, listing models (e.g., `build: deepseek-v4-pro`, `frontend: kimi-k2.7-code`) that the config does not assign — making the README an unreliable reference.
- **Decision:** Enforce mandatory Explore → Architect → human confirmation for all Feature, Architecture, and Security-sensitive tasks before any implementation begins. Small tasks (typo, button, comment) remain exempt. The Architect's design blueprint becomes authoritative — implementation agents must follow it without deviation. Syncing the README model table to match `opencode.jsonc` exactly, removing all invented model assignments.
- **Rationale:** Mandatory Architect catches structural issues before code is written, reducing rework and technical debt. The README must be a faithful reflection of the config, not an independent source of truth — the config **is** the source of truth for agent model assignments. Keeping Small tasks exempt avoids unnecessary overhead for trivial changes. Making the Architect blueprint authoritative prevents implementation agents from re-interpreting or deviating from the agreed design.
- **Alternatives considered:**
  1. *Keep Architect optional for Features* — rejected because Feature tasks (new pages, APIs, components) have sufficient scope to benefit from architectural review, and the cost of a skipped Architect is disproportionate to the low overhead of running it.
  2. *Make all flows mandatory Architect (including Small)* — rejected as excessive overhead for trivial changes.
  3. *Update README models by re-running a discovery command* — rejected because the config is the definitive source; manual correction is more precise and avoids introducing additional disclosure structures.
  4. *Touch default_agent* — explicitly excluded per the implementation brief; no approval was given to assume this change.

## 2026-08-01 — Phase 2: Adaptive Project Context

- **Context:** [why this decision came up]
- **Decision:** [what was decided]
- **Rationale:** [why this was the right choice]
- **Alternatives considered:** [what was rejected and why]
