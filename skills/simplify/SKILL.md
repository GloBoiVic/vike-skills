---
name: simplify
description: Opt-in proposal for the smallest behavior-preserving simplification after a review finding or explicit request; never mutates without approval.
---

# Simplify

Invoke only after a relevant review finding or explicit simplification request. It is not automatic cleanup, design review, or permission to change behavior, APIs, performance, architecture, security controls, or observability.

## Contract

1. State the authorizing finding/request.
2. Inspect the affected implementation, tests, contracts, and architecture guidance.
3. Propose the smallest diff, explicit non-goals, affected files, behavior-preservation argument, and compatibility/performance/error/migration risks.
4. Wait for explicit human approval before editing. After approval, make only the agreed change and run applicable tests.
5. If preservation cannot be demonstrated, stop and report the uncertainty. Architecture-affecting work must complete Explore → Architect → explicit human confirmation and follow the authoritative blueprint.

Return rationale, diff scope, validation results, and remaining concerns. Never silently rewrite source, tests, configuration, documentation, or generated files; never hide an unapproved simplification in unrelated work or override ownership and contracts.
