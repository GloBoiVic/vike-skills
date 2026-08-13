---
name: simplify
description: Opt-in guidance for proposing behavior-preserving simplifications after review or an explicit request; never silently rewrites code or architecture.
---

# Simplify

This skill is inert unless explicitly invoked. It identifies and, only after approval, helps implement low-risk simplifications that preserve externally observable behavior.

## When to use

- Use after a review identifies unnecessary complexity or when the user explicitly requests simplification.
- Do not use as an automatic cleanup pass, a substitute for design review, or a reason to change behavior, APIs, performance characteristics, or architecture.

## Workflow and outputs

1. State the review finding or explicit request that authorizes the proposal.
2. Inspect the relevant implementation, tests, contracts, and architecture guidance.
3. Propose the smallest behavior-preserving change, including non-goals and affected files.
4. Identify compatibility, performance, error-handling, and migration risks.
5. Wait for explicit approval before editing; after approval, make only the agreed changes and run applicable tests.

Any architecture-affecting simplification must complete the Explore → Architect → explicit human confirmation flow before edits. It must follow the Architect's authoritative blueprint and cannot override or bypass that blueprint.

Return the rationale, proposed diff scope, behavior-preservation argument, validation plan/results, and remaining concerns.

## Safety boundaries

- Never silently rewrite source, tests, configuration, documentation, or generated files.
- Never override established architecture, contracts, ownership, or review findings without explicit approval from the responsible human.
- Architecture-affecting simplifications require Explore → Architect → explicit human confirmation before edits and must not override the authoritative blueprint.
- Never bypass confirmation by hiding a simplification inside an unrelated change.
- Preserve behavior, public interfaces, error semantics, security controls, and required observability unless a separately approved change says otherwise.
- If behavior cannot be demonstrated as preserved, stop and report the uncertainty.

## Invocation rule

Never invoke this skill automatically. It requires either a relevant review finding or an explicit simplification request, followed by explicit approval of the proposed change before mutation.
